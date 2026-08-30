# Finance-Data-Processing-and-Access-Control-Backend

*Part of [[Projects]] → [[Real Projects]]*

**Repo:** [Finance-Data-Processing-and-Access-Control-Backend](https://github.com/prakharDvedi/Finance-Data-Processing-and-Access-Control-Backend)
**Visibility:** Public
**Language:** TypeScript

---

#Financial Records Management API with RBAC

API Documentation (Swagger UI): https://petstore.swagger.io/?url=https://raw.githubusercontent.com/prakharDvedi/Finance-Data-Processing-and-Access-Control-Backend/main/finance-data/public/openapi.json

Roadmap: [ROADMAP.md](ROADMAP.md)

  
Backend API for managing financial records with role-based access control, dashboard analytics, and JWT authentication.

Built as part of an intern assignment.

---

## Tech Stack

- **Runtime**: Next.js (App Router, API routes only)
- **Language**: TypeScript
- **Database**: PostgreSQL (via Docker)
- **ORM**: Prisma
- **Auth**: JWT (jsonwebtoken + bcryptjs)
- **Validation**: Zod
- **Testing**: Jest + Supertest

---

## Design Decisions

**Register always creates VIEWER** — accepting a role in the register payload would let anyone make themselves admin. Role changes go through the admin endpoint only.

**Soft delete instead of hard delete** — financial data shouldn't disappear. `isDeleted` flag hides records from all reads but keeps them in the DB for audit purposes.

**Access token only, no refresh token** — kept it simple for the assignment scope. Adding refresh tokens later would mean a `/auth/refresh` endpoint and a token rotation strategy.

**Decimal for money** — JavaScript floats have precision issues (`0.1 + 0.2 !== 0.3`). Prisma Decimal maps to Postgres `numeric` which handles money correctly.

**Integration tests over unit tests** — with limited time, testing the full request flow (HTTP → DB → response) catches more real bugs than mocking individual layers.

---

## Project Structure

```
src/
├── app/api/          # Route handlers (thin, just call controllers)
├── controllers/      # Parse input, call services, shape responses
├── services/         # Business logic, RBAC enforcement
├── repositories/     # Database queries only (Prisma)
├── schema/           # Zod validation schemas
├── lib/              # JWT, password hashing, auth helpers, error wrapper
└── middleware/        # (reserved for future use)
```

The app follows a strict layered architecture:

```
Route → Controller → Service → Repository → DB
```

Each layer only talks to the one below it. Controllers never touch the database. Repositories never know about HTTP.

---

## Getting Started

### Prerequisites

- Node.js 18+
- Docker (for Postgres)

### 1. Clone and install

```bash
git clone
cd finance-data
npm install
```

### 2. Start Postgres

```bash
docker compose up -d
```

### 3. Set up environment

Create a `.env` file in the project root:

```env
DATABASE_URL="postgresql://zorvyn:zorvyn123@localhost:5432/zorvyn_db"
JWT_SECRET="pick-any-long-random-string"
JWT_EXPIRES_IN="1h"
```

### 4. Run migrations and seed

```bash
npx prisma migrate dev --name init
npx prisma db seed
```

### 5. Start the server

```bash
npm run dev
```

Runs at `http://localhost:3000`.

---

## Test Credentials

| Role    | Email               | Password    |
| ------- | ------------------- | ----------- |
| Admin   | admin@example.com   | password123 |
| Analyst | analyst@example.com | password123 |
| Viewer  | viewer@example.com  | password123 |

---

## API Endpoints

### Auth (public)

| Method | Endpoint             | Description              |
| ------ | -------------------- | ------------------------ |
| POST   | `/api/auth/register` | Register (always VIEWER) |
| POST   | `/api/auth/login`    | Login, returns JWT       |

### Records

| Method | Endpoint           | Roles          | Notes          |
| ------ | ------------------ | -------------- | -------------- |
| POST   | `/api/records`     | ADMIN          | Create record  |
| GET    | `/api/records`     | ANALYST, ADMIN | List + filters |
| GET    | `/api/records/:id` | ANALYST, ADMIN | Get single     |
| PATCH  | `/api/records/:id` | ADMIN          | Partial update |
| DELETE | `/api/records/:id` | ADMIN          | Soft delete    |

**Query params for GET /api/records**: `type`, `category`, `dateFrom`, `dateTo`, `page`, `limit`

### Dashboard

| Method | Endpoint                 | Roles                  |
| ------ | ------------------------ | ---------------------- |
| GET    | `/api/dashboard/summary` | VIEWER, ANALYST, ADMIN |
| GET    | `/api/dashboard/recent`  | VIEWER, ANALYST, ADMIN |
| GET    | `/api/dashboard/trends`  | VIEWER, ANALYST, ADMIN |

`recent` accepts `?limit=5` (default 5, max 50).
`trends` accepts `?months=6` (default 6, max 24).

### User Management

| Method | Endpoint                | Roles |
| ------ | ----------------------- | ----- |
| GET    | `/api/users`            | ADMIN |
| PATCH  | `/api/users/:id/role`   | ADMIN |
| PATCH  | `/api/users/:id/status` | ADMIN |

---

## RBAC Matrix

|           | VIEWER | ANALYST | ADMIN     |
| --------- | ------ | ------- | --------- |
| Records   | —      | Read    | Full CRUD |
| Dashboard | Read   | Read    | Read      |
| Users     | —      | —       | Full      |

---

## Error Handling

All routes are wrapped in a global error handler. Errors always return this shape:

```json
{
  "error": {
    "message": "Validation failed",
    "code": "VALIDATION_ERROR"
  }
}
```

Zod validation errors return 400 with field-level details. Auth errors return 401/403. Not found returns 404. No stack traces are ever exposed.

---

## Running Tests

Make sure the server is running first:

```bash
npm run dev          # terminal 1
npm test             # terminal 2
```

The integration suite covers:

1. Register + login returns JWT
2. Viewer blocked from record creation
3. Analyst can read but not create records
4. Admin full CRUD on records
5. Dashboard summary returns correct aggregation shape
6. Soft-deleted records hidden from GET and list

---

## Assumptions

- Single-tenant system (no multi-org support)
- Dates are accepted and returned as ISO 8601 strings
- Pagination defaults to page 1, limit 10
- All monetary amounts are in a single currency (no currency field)

---

## What I'd Add With More Time

- Refresh token rotation
- Rate limiting on auth endpoints
- OpenAPI/Swagger documentation
- Record restore endpoint (undo soft delete)
- Audit log for admin actions
- CI pipeline with automated test runs
