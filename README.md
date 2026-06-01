
# AI Timetable Generator

A modern full-stack university web application foundation built with Next.js, Material UI, Express.js, and PostgreSQL.

## Project Structure

- `client/` — Next.js frontend with Material UI and responsive dashboard pages.
- `server/` — Express backend with API routing, database setup, and auth-ready architecture.
- `.env.example` — Example environment variable settings.

## Technologies

- Frontend: Next.js, React, Material UI, Axios
- Backend: Node.js, Express.js, PostgreSQL, pg, JWT-ready auth workflow
- Tooling: npm workspaces, concurrently, dotenv, nodemon

## Setup

1. Install dependencies:

   ```bash
   npm install
   ```

2. Create environment files:

   - Copy `.env.example` to `server/.env`
   - Copy `.env.example` to `client/.env.example`

3. Update values in `server/.env`:

   ```bash
   DATABASE_URL=postgresql://username:password@localhost:5432/ai_timetable
   JWT_SECRET=your_jwt_secret
   PORT=5000
   ```

4. Update values in `client/.env.example` if needed:

   ```bash
   NEXT_PUBLIC_API_URL=http://localhost:5000/api
   ```

## Development

Run both client and server concurrently:

```bash
npm run dev
```

Individual startup commands:

```bash
npm run client
npm run server
```

- Frontend: `http://localhost:3000`
- Backend API: `http://localhost:5000/api`

## Backend API

- `GET /api` — health check endpoint.
- `POST /api/auth/register` — create a new user account with role support.
- `POST /api/auth/login` — authenticate and receive JWT access and refresh tokens.
- `POST /api/auth/refresh` — refresh an access token using a valid refresh token.
- `POST /api/auth/logout` — invalidate the current session token version.
- `GET /api/protected/profile` — example protected route (requires Bearer token).
- `GET /api/admin/summary` — counts for users, departments, courses, and instructors.
- `GET /api/admin/roles` — list supported roles.
- `GET /api/admin/departments` — list departments.
- `POST /api/admin/departments` — create a department.
- `GET /api/admin/courses` — list courses.
- `POST /api/admin/courses` — create a course.
- `GET /api/admin/instructors` — list instructors.
- `POST /api/admin/instructors` — create an instructor.

## Database schema

- `server/src/db/schema.sql` — PostgreSQL schema for users, departments, courses, and instructors.

## Notes

- This repository contains the project foundation only. Timetable generation logic will be added later.
- Database connection is configured and ready for future migrations, models, and queries.
