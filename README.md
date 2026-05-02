# TaskFlow — Team Task Manager

A full-stack web app for managing projects, assigning tasks, and tracking team progress. Built with role-based access control so admins and members each have the right level of access.

## What it does

- **Auth** — signup and login with JWT-based sessions
- **Projects** — create projects, invite teammates by email, manage who's in
- **Tasks** — create tasks, assign them to team members, track status (To Do → In Progress → Done)
- **Dashboard** — see your task counts at a glance and catch anything overdue
- **RBAC** — admins manage the project and tasks, members update their own assigned work

## Tech stack

**Backend**
- Node.js + Express + TypeScript
- PostgreSQL with Prisma ORM
- JWT auth, bcrypt password hashing, Zod validation

**Frontend**
- React + Vite + TypeScript
- React Router for navigation
- Axios for API calls

**Deployment**
- Railway (single service — Express serves the React build)

## Project structure

```
├── backend/
│   ├── prisma/          schema + migrations
│   └── src/
│       ├── middleware/  JWT auth + RBAC guards
│       ├── routers/     auth, projects, tasks, dashboard
│       ├── services/    business logic
│       ├── schemas/     Zod validation
│       └── lib/         Prisma client, error helpers
├── frontend/
│   └── src/
│       ├── api/         typed API wrappers
│       ├── pages/       Login, Register, Dashboard, Projects, Tasks
│       ├── components/  Layout, sidebar
│       └── contexts/    AuthContext
├── railway.json
└── .env.example
```

## Running locally

**Prerequisites:** Node.js 20+, a PostgreSQL database

```bash
# 1. Clone and install
git clone <your-repo-url>
cd team-task-manager
npm install

# 2. Set up environment
cp .env.example .env
# Fill in DATABASE_URL and JWT_SECRET in .env

# 3. Run backend (port 3000)
npm run dev:backend

# 4. Run frontend in a separate terminal (port 5173)
npm run dev:frontend
```

Frontend proxies `/api` requests to the backend automatically via Vite config.

## Environment variables

```
DATABASE_URL=postgresql://user:password@localhost:5432/taskflow
JWT_SECRET=your-secret-key
PORT=3000
```

## Deploying to Railway

1. Push the repo to GitHub
2. Go to [railway.app](https://railway.app) → New Project → Deploy from GitHub
3. Add a PostgreSQL database service from the Railway dashboard
4. In your app service variables, set:
   - `DATABASE_URL` → `${{Postgres.DATABASE_URL}}`
   - `JWT_SECRET` → any random string
5. Railway handles the rest — it runs `npm run build` then starts the server

On startup the app automatically runs `prisma migrate deploy` so the database schema is always up to date.

## API overview

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| POST | `/api/auth/register` | — | Create account |
| POST | `/api/auth/login` | — | Login |
| GET | `/api/projects` | JWT | List your projects |
| POST | `/api/projects` | JWT | Create a project |
| GET | `/api/projects/:id` | JWT + member | Project details + team |
| POST | `/api/projects/:id/members` | JWT + admin | Add member by email |
| DELETE | `/api/projects/:id/members/:userId` | JWT + admin | Remove member |
| GET | `/api/projects/:id/tasks` | JWT + member | List tasks (filterable) |
| POST | `/api/projects/:id/tasks` | JWT + admin | Create task |
| PATCH | `/api/projects/:id/tasks/:taskId` | JWT + member | Update status or reassign |
| DELETE | `/api/projects/:id/tasks/:taskId` | JWT + admin | Delete task |
| GET | `/api/dashboard/me` | JWT | Personal task summary |
| GET | `/api/dashboard/projects/:id` | JWT + member | Project-level summary |

## Notes

- Passwords are hashed with bcrypt (12 rounds), never stored in plain text
- JWTs expire after 24 hours
- Deleting a project cascades and removes all its tasks
- A user can be admin in one project and a regular member in another — roles are per-project
