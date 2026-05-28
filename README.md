# Applied Software Engineering — Group Project (SIT725)

This repository contains the team's **VolunteerHub** application for the group project.

## Repository layout

| Path                                | Description                                         |
| ----------------------------------- | --------------------------------------------------- |
| `volunteerhub/backend/`           | Node.js + Express API, MongoDB (Mongoose), JWT auth |
| `volunteerhub/frontend/`          | Static HTML/CSS/JS UI served by a small Express app |
| `volunteerhub/docker-compose.yml` | Full-stack Docker setup (MongoDB + API + UI)        |

## Docker (SIT725 8.2HD)

Run the **entire application** (database, backend API, frontend UI) with Docker. No local Node or MongoDB install is required.

### Prerequisites

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) (includes Docker Compose)

### Build and start

From the repository root:

```bash
cd volunteerhub
docker compose up --build
```

This starts all required services:

- `mongo` on `27017`
- `backend` on `5000`
- `frontend` on `3300`

Wait until you see log lines similar to:

- `Seed completed.`
- `Backend listening on http://localhost:5000`
- `Frontend running on 3300`

The first start may take a few minutes while images build.

### Access the application

| What                                  | URL                               |
| ------------------------------------- | --------------------------------- |
| **Web UI**                      | http://localhost:3300             |
| **API health**                  | http://localhost:5000/health      |
| **Student endpoint (required)** | http://localhost:5000/api/student |

### `/api/student` response

```json
{
  "name": "Om Jagtap",
  "studentId": "s225435163"
}
```

### Required setup steps (from scratch)

1. Install Docker Desktop.
2. Clone this repository.
3. Open a terminal at the repository root.
4. Run:

```bash
cd volunteerhub
docker compose up --build
```

### Verify database-backed features (login)

The stack includes MongoDB and auto-seeds demo data on startup. Use these accounts at http://localhost:3300:

| Role                 | Email                                 | Password       |
| -------------------- | ------------------------------------- | -------------- |
| Volunteer            | `demo.volunteer@volunteerhub.local` | `Pass@12345` |
| Organisation manager | `manager@volunteerhub.local`        | `Pass@12345` |
| Admin                | `admin@volunteerhub.local`          | `Pass@12345` |

After login you should reach the dashboard (`/dashboard`), which confirms JWT auth and MongoDB are working.

### Stop and reset

```bash
docker compose down
```

To remove the database volume and start fresh:

```bash
docker compose down -v
docker compose up --build
```

### Configuration and secrets

- **No `.env` file is required for Docker.** Runtime values are set in `volunteerhub/docker-compose.yml` (MongoDB URI, JWT settings, app ports, and student identity for `/api/student`).
- **Do not commit** `backend/.env` or `frontend/.env` — they are listed in `.gitignore`. Copy from `backend/.env.example` / `frontend/.env.example` only for non-Docker local development.
- For this submission, required runtime values for Docker verification are already provided in `docker-compose.yml`, so you do not need to source hidden secrets to run the app.

### Troubleshooting

- **Port already in use:** Stop other apps on ports `3300` and `5000`, or change the host ports in `docker-compose.yml`.
- **Backend exits immediately:** Run `docker compose logs backend` and ensure MongoDB became healthy first.
- **UI loads but login fails:** Wait for `Seed completed.` in backend logs, then refresh and try again.

---

## What the app does (current scope)

- **Authentication:** register, login (JWT), role-based access (Volunteer, OrganisationManager, Admin).
- **Organisations:** managers submit an organisation (Pending); admins approve or reject; managers can view their organisation and update details after approval.
- **Events & applications:** create events, apply, check-in, notifications (SSE).
- **UI:** auth pages and a post-login **dashboard** (`/dashboard`).

## Local development (without Docker)

### Prerequisites

- **Node.js** 18+
- **MongoDB** (local or Atlas) — set `MONGO_URI` in `volunteerhub/backend/.env`

### 1. Backend

```bash
cd volunteerhub/backend
npm install
```

Copy `backend/.env.example` to `backend/.env` and set your values (especially `MONGO_URI` and `JWT_SECRET`).

```bash
npm run seed
npm start
```

Default URL: **http://localhost:5000**
Health check: `GET /health`
Student endpoint: `GET /api/student`

### 2. Frontend

```bash
cd volunteerhub/frontend
npm install
```

Copy `frontend/.env.example` to `frontend/.env` if needed:

```env
PORT=3300
API_BASE_URL=http://localhost:5000
```

```bash
npm start
```

Open **http://localhost:3300**

## API reference

Main API prefixes (on the backend host):

- `GET /api/student` — HD submission identity (name, studentId)
- `POST /auth/register`, `POST /auth/login`
- Organisation routes under `/auth/organizations`, `/auth/organizations/me`, etc.
- `GET /events`, `POST /applications`, etc.

## Tech stack

- **Backend:** Express, Mongoose, bcrypt, jsonwebtoken, dotenv, CORS
- **Frontend:** Vanilla JS, Express (static files + HTML routes)
- **Docker:** MongoDB 7, Node 20 Alpine images via Compose
