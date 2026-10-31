# CampusCare: Smart Campus Service Management System

**KL University CODESPARK Hackathon**

CampusCare lets students report campus issues (electrical, plumbing, Wi-Fi, maintenance and so on), routes each request to the right department and technician, and tracks it through to resolution with a full audit timeline, overdue alerts, feedback, and an FAQ chatbot.

---

## Table of Contents

- [Tech Stack](#tech-stack)
- [Features](#features)
- [Roles and Workflow](#roles-and-workflow)
- [Quick Start](#quick-start)
- [Configuration](#configuration)
- [Demo Accounts](#demo-accounts)
- [API Reference](#api-reference)
- [Architecture](#architecture)
- [Project Structure](#project-structure)
- [Security Notes](#security-notes)
- [Known Limitations](#known-limitations)

---

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | React 18, Vite 5, Recharts |
| Backend | Java 17, Spring Boot 3.3.5 |
| Database | MySQL 8 |
| ORM | Spring Data JPA / Hibernate |
| Security | Spring Security, JWT (jjwt 0.12.6), BCrypt |
| Build | Maven |

---

## Features

- **Authentication**: JWT login, student self-registration, BCrypt password hashing.
- **Role-based dashboards**: separate views for students, technicians and admins.
- **Request submission**: title, description, building / floor / room, category and priority.
- **Auto category suggestion**: a keyword-scoring classifier (`ClassifierService`) suggests a category from the description.
- **Workflow and timeline**: every status change is recorded and viewable as a history.
- **Assignment**: admins assign requests to departments and technicians.
- **Overdue alerts**: requests unresolved for more than 72 hours are flagged on the admin dashboard.
- **Feedback**: star rating and comment after resolution.
- **FAQ chatbot**: keyword-scored answers, seeded with 7 common campus questions; FAQs can be added or removed through the API.
- **Admin analytics**: dashboard stats with charts, plus CRUD for categories, departments and users.
- **Search and filter**: status and priority filters on request tables.

---

## Roles and Workflow

Roles: `STUDENT`, `STAFF`, `TECHNICIAN`, `DEPT_HEAD`, `ADMIN`

Priorities: `LOW`, `MEDIUM` (default), `HIGH`

```
OPEN  ->  ASSIGNED  ->  IN_PROGRESS  ->  RESOLVED  ->  CLOSED
```

| Role | Can do |
|---|---|
| Student | Submit requests, track status and history, give feedback |
| Technician | View assigned requests, move them to In Progress / Resolved with remarks |
| Admin / Dept Head | Dashboard, assignment, status changes, manage users, categories, departments |

---

## Quick Start

### Prerequisites

- Java 17+
- Maven 3.8+
- MySQL 8+
- Node.js 18+

### 1. Database

```bash
mysql -u root -p < database/schema.sql
mysql -u root -p smart_campus < database/seed.sql
```

`schema.sql` creates the `smart_campus` database and its 7 tables. Hibernate is also set to `ddl-auto=update`, so missing tables are created on startup. On first start the backend creates the demo accounts and seeds the chatbot FAQs if none exist.

### 2. Backend

```bash
cd backend
mvn spring-boot:run
```

The API starts at **http://localhost:8080**.

Spring Boot does not read a `.env` file by itself. Set the variables in your shell or IDE run configuration instead:

```bash
# macOS / Linux
export DB_PASS=yourpassword
mvn spring-boot:run

# Windows PowerShell
$env:DB_PASS="yourpassword"; mvn spring-boot:run
```

Note the default MySQL port in the config is **3307**. If your MySQL runs on 3306, set `DB_URL` accordingly.

### 3. Frontend

```bash
cd frontend
npm install
npm run dev
```

Open **http://localhost:5173**. Vite proxies `/api/*` to `http://localhost:8080`.

Production build: `npm run build` (output in `frontend/dist`).

---

## Configuration

Backend settings come from `backend/src/main/resources/application.properties`, overridable with environment variables. `backend/.env.example` lists them.

| Variable | Default | Description |
|---|---|---|
| `DB_URL` | `jdbc:mysql://localhost:3307/smart_campus?useSSL=false&serverTimezone=UTC&allowPublicKeyRetrieval=true` | JDBC URL |
| `DB_USER` | `root` | MySQL user |
| `DB_PASS` | none (set this) | MySQL password |
| `JWT_SECRET` | sample value in `.env.example` | Token signing secret; change for any real deployment |
| `ADMIN_EMAIL` | `admin@campus.edu` | Admin account email |
| `ADMIN_PASSWORD` | `Admin@123` | Admin account password |

Other settings: server port `8080`, JWT lifetime 24 hours (`app.jwt.expiration-ms=86400000`).

---

## Demo Accounts

Created automatically on first start.

| Role | Email | Password |
|---|---|---|
| Admin | admin@campus.edu | Admin@123 |
| Technician | tech@campus.edu | Tech@123 |
| Student | student@campus.edu | Student@123 |

These are for demo use only. Change or remove them before any real deployment.

---

## API Reference

Base URL: `http://localhost:8080`. Send `Authorization: Bearer <token>` on protected routes.

### Auth (public)

| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/auth/register` | Student self-registration |
| POST | `/api/auth/login` | Login, returns a JWT |
| GET | `/api/auth/me` | Current user profile |

### Lookups

| Method | Endpoint | Access |
|---|---|---|
| GET | `/api/categories` | Public |
| GET | `/api/departments` | Public |

### Requests (authenticated)

| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/requests` | Create a request |
| GET | `/api/requests` | List requests (scoped by role) |
| GET | `/api/requests/{id}` | Get one request |
| GET | `/api/requests/{id}/history` | Status timeline |
| PUT | `/api/requests/{id}/status` | Change status |
| PUT | `/api/requests/{id}/assign-department` | Assign department |
| PUT | `/api/requests/{id}/assign-technician` | Assign technician (`/assign` is also mapped) |
| POST | `/api/requests/suggest-category` | Suggest a category (public) |
| POST | `/api/requests/{id}/feedback` | Submit feedback |
| GET | `/api/requests/{id}/feedback` | Read feedback |

### Admin (ADMIN, DEPT_HEAD)

| Method | Endpoint | Description |
|---|---|---|
| GET | `/api/admin/stats` | Dashboard analytics |
| GET | `/api/admin/overdue` | Requests unresolved over 72 hours |
| POST / PUT / DELETE | `/api/admin/categories[/{id}]` | Category management |
| POST / PUT / DELETE | `/api/admin/departments[/{id}]` | Department management |

### Users (ADMIN, DEPT_HEAD)

| Method | Endpoint | Description |
|---|---|---|
| GET | `/api/users` | List users |
| GET | `/api/users/role/{role}` | List users by role |
| GET | `/api/users/technicians` | List technicians |
| POST | `/api/users` | Create staff / technician |
| PUT | `/api/users/{id}` | Update user |
| DELETE | `/api/users/{id}` | Delete user |

### Chatbot

| Method | Endpoint | Description |
|---|---|---|
| GET | `/api/chatbot/ask?q=...` | Ask the FAQ bot (public) |
| GET | `/api/chatbot/faqs` | List FAQs (public) |
| POST | `/api/chatbot/faqs` | Add an FAQ |
| DELETE | `/api/chatbot/faqs/{id}` | Remove an FAQ |

---

## Architecture

```
Browser (React 18 + Vite, :5173)
   Student / Technician / Admin dashboards, chatbot widget
        |  HTTP + JWT Bearer token, /api/**
        v
Spring Boot 3 backend (:8080)
   JwtFilter -> Controllers -> Services -> JPA repositories
   BCrypt, ClassifierService, chatbot FAQ matching
        |  JDBC
        v
MySQL 8
   users, departments, categories, service_requests,
   request_updates, feedback, chatbot_faqs
```

---

## Project Structure

```
smart-campus/
├── backend/
│   ├── pom.xml
│   ├── .env.example
│   └── src/main/
│       ├── resources/application.properties
│       └── java/com/campus/smartcampus/
│           ├── SmartCampusApplication.java
│           ├── config/     AppConfig (seeding), SecurityConfig
│           ├── model/      User, ServiceRequest, Category, Department,
│           │               RequestUpdate, Feedback, ChatbotFaq, Enums
│           ├── repo/       JPA repositories
│           ├── security/   JwtUtil, JwtFilter
│           ├── service/    RequestService, ClassifierService
│           └── web/        Auth, Request, Admin, User, Chatbot controllers
├── frontend/
│   ├── index.html, package.json, vite.config.js
│   └── src/                main.jsx, App.jsx, api.js, styles.css
├── database/
│   ├── schema.sql
│   ├── seed.sql
│   └── useful_queries.sql
└── README.md
```

---

## Security Notes

Items to fix before any non-demo deployment:

- **Hardcoded database password.** `application.properties` contains a real-looking fallback password for `DB_PASS`. Remove the fallback, rotate that password, and supply it only through the environment.
- **Default secrets.** Replace the sample `JWT_SECRET` and the default admin and demo passwords.
- **CORS** currently allows all origins (`*`). Restrict it to your frontend's origin.
- **CSRF** is disabled, which is normal for a stateless JWT API, but keep tokens out of cookies unless you re-enable it.
- Do not commit `.env` or the `backend/target/` build output (both are in `.gitignore`).

---

## Known Limitations

- No pagination on request lists.
- No email notifications (Spring Mail would be the natural addition).
- Location picking uses dropdowns; an interactive map is a stretch goal.
- The category classifier is keyword-based; an ML classifier could replace it behind `ClassifierService`.

---

*CampusCare, KL University CODESPARK Hackathon. Built with Spring Boot and React.*
