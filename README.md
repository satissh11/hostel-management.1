# HostelCare

HostelCare is a responsive hostel complaint and incident tracking system. It has a React dashboard, Express API, PostgreSQL database, and Prisma data model. Emergency reports are persisted first and then create in-app alerts for active wardens and administrators assigned to the reporter's hostel. External email is optional and its delivery attempt is recorded separately from authority acknowledgement.

## Start locally

1. Install Node.js 20+ and PostgreSQL 14+.
2. Create a PostgreSQL database named `hostelcare`.
3. Copy `.env.example` to `.env` and set `DATABASE_URL` and a random `JWT_SECRET` of at least 32 characters.
4. Run `npm install`.
5. Run `npm run db:generate`, then `npm run db:migrate`.
6. Run `npm run db:seed` to create a Maple House demo hostel, staff, and categories.
7. Run `npm run dev` and open http://localhost:5173.

Seeded accounts use the password `HostelCare2026!`: `student@hostelcare.local`, `warden@hostelcare.local`, `admin@hostelcare.local`, and `maintenance@hostelcare.local`. Change these demo credentials before any shared or production deployment. The seed script takes admin/warden email overrides from the environment.

## Roles and flow

- Students, staff, and guards can create and track their own complaints and serious incidents.
- Students can create accounts from the sign-in screen. If `STUDENT_SIGNUP_CODE` is set, the registration form requires that code. Staff, warden, administration, and maintenance accounts are provisioned by an administrator.
- Wardens see complaints and incidents for their hostel; administration can see incidents across hostels.
- Maintenance staff see their assigned complaint work.
- Emergency submission directly notifies active wardens and administrators associated with the hostel. The stored `Alert Sent` status records the creation of in-app alerts; it does not mean an authority acknowledged the incident.
- In-app notifications and attempt logs are database backed. Configure SMTP values to enable email attempts. SMS and WhatsApp require a provider integration and are intentionally not reported as delivered when unconfigured.
- A server worker checks overdue incident deadlines and records an automatic escalation to administration. Keep the API process running or move this worker to a durable queue for multi-instance production deployments.

## Structure

- `src/`: React/Vite application and mobile-first styles.
- `server/index.js`: Express API, authorization, validation, notification creation, and escalation worker.
- `prisma/schema.prisma`: PostgreSQL schema and indexes.
- `prisma/seed.js`: local development seed data.

## API

All protected routes use `Authorization: Bearer <token>`. Authentication: `POST /api/auth/login`, `POST /api/auth/logout`, `GET /api/auth/me`, `POST /api/auth/forgot-password`, `POST /api/auth/reset-password`. Complaint routes: `POST/GET /api/complaints`, `GET /api/complaints/:id`, `PATCH /api/complaints/:id/assign`, `PATCH /api/complaints/:id/status`, `POST /api/complaints/:id/reopen`, `POST /api/complaints/:id/feedback`. Incident routes: `POST/GET /api/incidents`, `GET /api/incidents/:id`, `PATCH /api/incidents/:id/acknowledge`, `PATCH /api/incidents/:id/assign`, `PATCH /api/incidents/:id/status`, `POST /api/incidents/:id/escalate`, and incident history/notification endpoints. `/api/notifications`, `/api/admin/dashboard`, `/api/admin/incidents`, `/api/admin/notification-logs`, `/api/admin/reports`, `/api/users`, `/api/hostels`, `/api/blocks`, `/api/rooms`, and `/api/categories` support dashboards. List endpoints paginate complaints/incidents with `page` and `limit`.

## Operational notes

Use HTTPS, a managed PostgreSQL service with backups, a secrets manager, and a durable queue/worker in production. Configure SMTP and verify provider delivery receipts for stronger external delivery guarantees. File upload storage is not enabled in this starter; attachment URL fields are reserved in the schema and URL input is validated. Add private object storage and malware/type validation before enabling attachments. Password reset links are emailed only when SMTP is configured. Account provisioning is administrator-managed outside this initial API. Configure real hostel emergency numbers in deployment-specific content; the interface clearly instructs users to contact emergency services directly and never waits for online acknowledgement.
