# Product Requirements Document (PRD)

## Goal

Build a **dockerized multi-tenant SaaS application** where organizations (tenants) can manage users, projects, and tasks.  
The system must enforce **strict tenant isolation**, **JWT-based authentication**, **role-based access control (RBAC)**, and **subscription limits**, and it must **automatically run database migrations and seed data on startup**.

---

## Personas & Roles

- **super_admin**
  - Platform-level administrator
  - Not associated with any tenant
  - Can list and manage tenants

- **tenant_admin**
  - Administrator within a single tenant
  - Can manage tenant users, projects, and tasks

- **user**
  - Regular tenant member
  - Can work only on projects and tasks within their tenant

---

## User Journeys

1. **Super Admin**
   - Logs in without a tenant context
   - Lists and inspects tenants on the platform

2. **Tenant Registration**
   - A new tenant registers
   - System creates the tenant and its first tenant admin

3. **Tenant Admin**
   - Logs in using tenant subdomain
   - Manages users, projects, and tasks within their tenant

4. **Tenant User**
   - Logs in under a tenant
   - Works only with tenant-scoped projects and tasks

---

## Functional Requirements

1. Support tenant login using `tenantSubdomain` or `tenantId`.
2. Support super admin login without tenant context.
3. Issue a JWT on successful authentication.
4. Protect secured endpoints using `Authorization: Bearer <token>`.
5. Enforce roles: `super_admin`, `tenant_admin`, `user`.
6. Enforce strict tenant isolation for all tenant-owned data.
7. Prevent cross-tenant access, even if resource IDs are guessed.
8. Allow tenant registration (create tenant + first tenant admin).
9. Allow `super_admin` to list all tenants.
10. Allow tenant admins to create users within their tenant.
11. Allow tenant admins to list users within their tenant.
12. Allow tenant admins to delete users (cannot delete self).
13. Allow tenants to create projects (respect `max_projects` limit).
14. Allow listing, updating, and deleting projects within a tenant.
15. Allow creating tasks under a project.
16. Allow listing tasks under a project with filters.
17. Allow updating task status:
    - `todo`
    - `in_progress`
    - `done`
    - `cancelled`
18. Enforce subscription limits (`max_users`, `max_projects`).
19. Record audit logs for important actions (login, logout, seed, etc.).
20. Provide a health check endpoint: `GET /api/health`.

---

## Non-Functional Requirements

1. One-command startup using:
2. Fixed ports:
- PostgreSQL: `5432`
- Backend API: `5000`
- Frontend: `3000`
3. Automatic database migrations on backend startup.
4. Automatic seed execution after migrations.
5. Idempotent startup (safe to restart with existing DB volume).
6. Security basics:
- Password hashing with bcrypt
- Parameterized SQL queries
- JWT secret required via environment variables

---

## Success Criteria

- `docker-compose up -d` completes successfully.
- `GET /api/health` returns ready status (`status=ok`).
- Seeded credentials from `submission.json` allow login.
- Cross-tenant access attempts are consistently rejected.

---

## Out of Scope

- Payments and billing
- Email notifications
- Background jobs
- Advanced or custom permission models

---