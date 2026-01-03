# Backend API Documentation

This document describes all backend APIs implemented under `backend/src/routes/*`.

---

## Base URLs

- Backend (Docker): http://localhost:5000/api  
- Frontend (Nginx proxy): http://localhost:3000/api

---

## Authentication

Most endpoints require a JWT.

Header:
Authorization: Bearer <token>

JWT payload contains:
- userId
- tenantId (null for super_admin)
- role

---

## Roles

- super_admin → Platform-level
- tenant_admin → Admin within a tenant
- user → Regular tenant member

Important:
- super_admin is platform-scoped and cannot access tenant resources (projects, tasks, users).
- Tenant endpoints always enforce tenant isolation using tenantId from JWT.

---

## Response Format

Success:
{ "success": true, "data": ..., "message": "optional" }

Error:
{ "success": false, "message": "error message", "data": "optional" }

---

## Constraints

Task status:
- todo
- in_progress
- done
- cancelled

Task priority:
- low
- medium
- high
- urgent

---

## Endpoint Count

- Core APIs: 19  
  (Auth + Tenants + Users + Projects + Tasks)
- Operational APIs: 2  
  (/health, /)

---

# Operational Endpoints

## GET /api/health

Readiness probe.

Responses:
- 200 → { "status": "ok", "database": "connected" }
- 503 → { "status": "initializing", "database": "connected" }
- 500 → { "status": "error", "database": "disconnected" }

---

## GET /api

API metadata.

Auth: Not required

Response:
{ "success": true, "data": { "version": "1.0.0" } }

---

# Auth (4)

## POST /api/auth/register-tenant

Creates a tenant and first tenant admin.

Auth: Not required

Body:
- tenantName (string, required)
- subdomain (string, required, unique)
- adminEmail (string, required)
- adminPassword (string, required)
- adminFullName (string, required)

Responses:
- 201 success
- 400 validation error
- 409 conflict
- 500 server error

---

## POST /api/auth/login

Authenticate and return JWT.

Auth: Not required

Tenant login requires:
- email
- password
- tenantSubdomain OR tenantId

Super admin login:
- email
- password

Responses:
- 200 success
- 400 invalid input
- 401 invalid credentials
- 403 inactive/suspended
- 404 tenant not found

---

## GET /api/auth/me

Returns current user and tenant info.

Auth: Required

Responses:
- 200 success
- 401 invalid token
- 404 user not found

---

## POST /api/auth/logout

Logs logout event.

Auth: Required

Response:
- 200 success

---

# Tenants (3)

## GET /api/tenants

List tenants (paginated).

Auth: Required  
Role: super_admin only

Query:
- page (default 1)
- limit (default 10, max 100)
- status
- subscriptionPlan

---

## GET /api/tenants/:tenantId

Get tenant details and stats.

Auth: Required

Access:
- super_admin → any tenant
- others → own tenant only

---

## PUT /api/tenants/:tenantId

Update tenant fields.

Auth: Required

Permissions:
- super_admin → all fields
- others → name only (own tenant)

Fields:
- name
- status
- subscriptionPlan
- maxUsers
- maxProjects

---

# Users (4)

## POST /api/users/:tenantId/users

Create user inside tenant.

Auth: Required  
Role: tenant_admin

Limits:
- Enforces max_users

---

## GET /api/users/:tenantId/users

List users in tenant.

Auth: Required

Access:
- super_admin OR same tenant

Query:
- search
- role
- page
- limit

---

## PUT /api/users/:userId

Update user.

Auth: Required

Rules:
- fullName → self / tenant_admin / super_admin
- role, isActive → tenant_admin only

---

## DELETE /api/users/:userId

Delete user.

Auth: Required  
Role: tenant_admin

Rules:
- Cannot delete self
- Tasks are auto-unassigned

---

# Projects (4)

## POST /api/projects

Create project.

Auth: Required

Limits:
- Enforces max_projects

---

## GET /api/projects

List tenant projects.

Auth: Required

Query:
- status
- search
- page
- limit

---

## PUT /api/projects/:projectId

Update project.

Auth: Required

Allowed:
- tenant_admin
- project creator

---

## DELETE /api/projects/:projectId

Delete project and tasks.

Auth: Required

---

# Tasks (4)

## POST /api/tasks/projects/:projectId/tasks

Create task.

Auth: Required

Default:
- status = todo

---

## GET /api/tasks/projects/:projectId/tasks

List tasks for project.

Auth: Required

Query:
- status
- assignedTo
- priority
- search
- page
- limit

---

## PATCH /api/tasks/:taskId/status

Update task status only.

Auth: Required

Body:
- status (todo | in_progress | done | cancelled)

---

## PUT /api/tasks/:taskId

Update task fields.

Auth: Required

Fields:
- title
- description
- status
- priority
- assignedTo
- dueDate

---

# Quick cURL Examples

Tenant login:
curl http://localhost:5000/api/auth/login \
-H "Content-Type: application/json" \
-d '{"email":"admin@demo.com","password":"Demo@123","tenantSubdomain":"demo"}'

Get current user:
curl http://localhost:5000/api/auth/me \
-H "Authorization: Bearer <token>"

Update task status:
curl -X PATCH http://localhost:5000/api/tasks/<taskId>/status \
-H "Authorization: Bearer <token>" \
-d '{"status":"done"}