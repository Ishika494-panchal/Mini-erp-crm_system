# Mini ERP-CRM System

A full-stack Mini ERP + CRM application for managing customers, products/inventory, and delivery challans, with JWT authentication and role-based access control (RBAC).

## 🚀 Live Demo

- **Frontend App:** https://mini-erp-crm-system-1.onrender.com
- **Backend API Base:** https://mini-erp-crm-system-oul3.onrender.com/api
- **Health Check:** https://mini-erp-crm-system-oul3.onrender.com/health

> Both services are hosted on [Render](https://render.com/). The backend is on Render's free tier, so it may take 30–60 seconds to wake up after a period of inactivity.

Use one of the seeded demo accounts below (password `Password123`) to log in.

## Tech Stack

- **Backend:** Node.js, Express, TypeScript, Prisma ORM, PostgreSQL, JWT
- **Frontend:** React, TypeScript, Vite, React Router
- **Database:** PostgreSQL (via Supabase in production, Docker locally)

## Features

- JWT-based authentication with 4 roles: `ADMIN`, `SALES`, `WAREHOUSE`, `ACCOUNTS`
- Customer management with follow-ups (leads, active, inactive)
- Product & inventory management with stock movement tracking
- Delivery challan creation and status tracking
- Role-protected API routes and module access

## Architecture

The system follows a clean N-Tier Client-Server Architecture:

- **Frontend:** Single Page Application built with React 18, TypeScript, React Router DOM, and vanilla CSS. State management uses React Context API (`AuthContext`) and custom HTTP handlers (`client.ts`).
- **Backend:** RESTful API server built with Node.js, Express, and TypeScript, organized using Routes ➔ Controllers ➔ Services ➔ Prisma ORM.
- **Authentication & RBAC:** Stateless JWT (JSON Web Token) authentication paired with role-authorization middleware (`authorize('ADMIN', 'SALES', ...)`).
- **Database & Data Integrity:** Hosted PostgreSQL (Supabase) managed via Prisma ORM. Delivery challan generation and stock deductions are protected inside atomic `prisma.$transaction` blocks to prevent stock going negative under concurrency.

In production, the frontend (static build on Render) and backend (Node service on Render) are deployed separately and communicate over HTTPS.

## Known Limitations & Future Scope

- **PDF Export:** Delivery challans can be printed/exported via standard browser print dialogues (`Ctrl + P`); native server-side PDF generation can be added.
- **Graphical Analytics:** Metrics and stock alerts are displayed in numeric summary cards; graphical charts (e.g. Chart.js/Recharts) can be added for monthly revenue visualization.
- No automated tests (unit/integration) are included yet.
- Password reset / "forgot password" flow is not implemented.
- No email notifications (e.g. for follow-ups or low stock alerts).
- File/image uploads (e.g. product images) are not supported.
- CORS currently allows all origins by default — should be restricted before production use (see [Deployment checklist](#post-deployment-checklist)).
- Render free tier causes cold-start delays on the backend after inactivity.
- No CI/CD pipeline configured — deploys are manual via Render's dashboard.

## Project Structure

```
Mini-erp-crm_system/
├── Backend/                # Express + TypeScript API
│   ├── prisma/              # Prisma schema & seed script
│   └── src/
│       ├── config/            # env & db config
│       ├── controllers/       # route handlers
│       ├── middleware/        # auth, role guard, error handler
│       ├── routes/            # API route definitions
│       ├── services/          # business logic
│       ├── utils/             # helpers (jwt, response, errors)
│       └── validators/        # Zod request validation
├── Frontend/                # React + Vite client
│   └── src/
│       ├── api/                # API client
│       ├── components/         # shared components (Layout, ProtectedRoute)
│       ├── context/            # AuthContext
│       └── pages/              # Dashboard, Customers, Products, Challans, Login
├── docker-compose.yml       # PostgreSQL container for local dev
└── postman_collection.json  # Postman collection for the API
```

## Prerequisites

- [Node.js](https://nodejs.org/) v18 or higher
- [npm](https://www.npmjs.com/)
- [Docker](https://www.docker.com/) (recommended, for PostgreSQL) **or** a local PostgreSQL 15+ instance
- [Git](https://git-scm.com/)

## 1. Clone the Repository

```bash
git clone https://github.com/Ishika494-panchal/Mini-erp-crm_system.git
cd Mini-erp-crm_system
```

## 2. Database Setup

The project ships with a `docker-compose.yml` that spins up PostgreSQL.

```bash
docker compose up -d
```

This starts a Postgres container with:

- **User:** `postgres`
- **Password:** `postgrespassword`
- **Database:** `mini_erp_db`
- **Port:** `5432`

## 3. Backend Setup

```bash
cd Backend
npm install
```

**Configure environment variables** — copy the example file and fill in your values:

```bash
cp .env.example .env
```

`.env`:

```env
PORT=5000
NODE_ENV=development
JWT_SECRET=your_jwt_secret_key_here
JWT_EXPIRES_IN=24h
DATABASE_URL="postgresql://postgres:postgrespassword@localhost:5432/mini_erp_db?schema=public"
```

**Run database migrations & generate Prisma client**

```bash
npm run prisma:generate
npm run prisma:migrate
```

**Seed the database** (creates 4 default role-based users, all with password `Password123`):

```bash
npm run prisma:seed
```

```
Role         Email                    Password
Admin        admin@minierp.com        Password123
Sales        sales@minierp.com        Password123
Warehouse    warehouse@minierp.com    Password123
Accounts     accounts@minierp.com     Password123
```

> ⚠️ Change these credentials before deploying to production.

**Start the backend server**

```bash
npm run dev
```

The API will be available at `http://localhost:5000`:
- Health check: `GET /health`
- API base / docs: `GET /api`

For production:

```bash
npm run build
npm start
```

## 4. Frontend Setup

Open a new terminal:

```bash
cd Frontend
npm install
```

**Configure environment variables** — create a `.env` file in `Frontend/`:

```env
VITE_API_URL=http://localhost:5000
```

**Start the frontend dev server**

```bash
npm run dev
```

The app will be available at `http://localhost:5173`.

For production build:

```bash
npm run build
npm run preview
```

## Deployment

This project is currently deployed with the database on **[Supabase](https://supabase.com/)** (PostgreSQL) and both the backend and frontend on **[Render](https://render.com/)**. To deploy your own instance:

### 1. Database (Supabase PostgreSQL)

1. Create a new project on [Supabase](https://supabase.com/).
2. Go to **Project Settings → Database → Connection String** and copy the **URI** (choose the **Connection Pooler** URL if available — recommended for Render deployments).
3. Replace `[YOUR-PASSWORD]` in the string with your database password. This full string is your `DATABASE_URL`:

```
postgresql://postgres.xxxxxxxxxxxx:[YOUR-PASSWORD]@aws-0-region.pooler.supabase.com:6543/postgres
```

4. If using the pooled connection (port `6543`), append `?pgbouncer=true` so Prisma handles it correctly:

```
postgresql://postgres.xxxxxxxxxxxx:[YOUR-PASSWORD]@aws-0-region.pooler.supabase.com:6543/postgres?pgbouncer=true
```

### 2. Backend (Render Web Service)

1. Create a new **Web Service**, pointing at this repo with **Root Directory** set to `Backend`.
2. Build command: `npm install && npm run build && npx prisma generate`
3. Start command: `npm start`
4. Add environment variables in the Render dashboard:

```
NODE_ENV         production
DATABASE_URL     (the Supabase connection string from step 1)
JWT_SECRET       (a strong, unique secret)
JWT_EXPIRES_IN   24h
```

5. After the first deploy, run migrations and seed data against the production DB (via Render's **Shell** tab, or locally with `DATABASE_URL` pointed at Supabase):

```bash
npx prisma migrate deploy
npx prisma db seed
```

6. Confirm it's live: `GET /health` should return `{"success":true,...}`.

### 3. Frontend (Render Static Site)

1. Create a new **Static Site**, pointing at this repo with **Root Directory** set to `Frontend`.
2. Build command: `npm install && npm run build`
3. Publish directory: `dist`
4. Environment variable:

```
VITE_API_URL    Your backend's Render URL, e.g. https://mini-erp-crm-system-oul3.onrender.com
```

> Note: Vite env vars are baked in at **build time** — if you change `VITE_API_URL` later, trigger a new deploy/rebuild for it to take effect.

### Post-deployment checklist

- [x] Backend deployed and healthy — see [Live Demo](#-live-demo)
- [x] Frontend deployed — see [Live Demo](#-live-demo)
- [ ] Replace the default `JWT_SECRET` with a strong, unique value.
- [ ] Change or remove the seeded default user passwords before real use.
- [ ] Restrict CORS (`Backend/src/app.ts`) to the frontend's production domain instead of allowing all origins.
- [ ] Consider upgrading off Render's free tier if cold-start delays are a problem.
- [ ] In Supabase, restrict database network access / rotate the password if it was ever exposed publicly.

## API Reference

Key endpoint groups (see `GET /api` for the full live list, or import `postman_collection.json` into Postman):

- **Auth:** `POST /api/auth/register`, `POST /api/auth/login`, `GET /api/auth/me`
- **Customers:** `GET/POST /api/customers`, `GET/PUT/DELETE /api/customers/:id`, `POST /api/customers/:id/followup`
- **Products:** `GET/POST /api/products`, `GET/PUT/DELETE /api/products/:id`, `POST /api/products/:id/stock`
- **Challans:** `GET/POST /api/challans`, `GET /api/challans/:id`, `PATCH /api/challans/:id/status`, `DELETE /api/challans/:id`
- **Modules:** Role-gated — `GET /api/modules/admin`, `/sales`, `/warehouse`, `/accounts`

All protected routes require an `Authorization: Bearer <token>` header, obtained from `POST /api/auth/login`.


## Architecture

The system follows a clean N-Tier Client-Server Architecture:

- **Frontend:** Single Page Application built with React 18, TypeScript, React Router DOM, and vanilla CSS. State management uses React Context API (`AuthContext`) and custom HTTP handlers (`client.ts`).
- **Backend:** RESTful API server built with Node.js, Express, and TypeScript, organized using Routes ➔ Controllers ➔ Services ➔ Prisma ORM.
- **Authentication & RBAC:** Stateless JWT (JSON Web Token) authentication paired with role-authorization middleware (`authorize('ADMIN', 'SALES', ...)`).
- **Database & Data Integrity:** Hosted PostgreSQL (Supabase) managed via Prisma ORM. Delivery challan generation and stock deductions are protected inside atomic `prisma.$transaction` blocks to prevent stock going negative under concurrency.

## Known Limitations & Future Scope

- **PDF Export:** Delivery challans can be printed/exported via standard browser print dialogues (`Ctrl + P`); native server-side PDF generation can be added.
- **Graphical Analytics:** Metrics and stock alerts are displayed in numeric summary cards; graphical charts (e.g. Chart.js/Recharts) can be added for monthly revenue visualization.


## License

ISC


