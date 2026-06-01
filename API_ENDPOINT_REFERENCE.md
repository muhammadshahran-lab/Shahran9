# Riphah UMS - Complete API Endpoint Reference

Comprehensive list of all API endpoints across 17 modules.

---

## 📊 API Overview

**Base URL**: `http://localhost:4000/api`  
**Documentation**: `http://localhost:4000/api/docs` (Swagger UI)  
**Authentication**: Bearer Token (JWT)  
**Response Format**: JSON

---

## Authentication Endpoints

### POST /auth/login
Login with email and password
```
Request: { email: string, password: string }
Response: { access_token: string, refresh_token: string, user: User }
Auth: None
Roles: Public
```

### POST /auth/mfa/verify
Verify MFA code
```
Request: { code: string }
Response: { verified: boolean, token: string }
Auth: Required
Roles: All authenticated users
```

### POST /auth/refresh
Refresh access token
```
Request: { refresh_token: string }
Response: { access_token: string }
Auth: None
Roles: Public
```

### POST /auth/logout
Logout current session
```
Request: {}
Response: { success: boolean }
Auth: Required
Roles: All authenticated users
```

---

## User Management Endpoints

### GET /users
List all users with pagination and filtering
```
Query Params: skip=0&take=10&search=&role=
Response: { data: User[], total: number, page: number }
Auth: Required
Roles: Admin, Registrar, Dean
```

### GET /users/:id
Get user by ID
```
Response: User
Auth: Required
Roles: Admin, own user
```

### POST /users
Create new user
```
Request: CreateUserDto
Response: User
Auth: Required
Roles: Admin, Registrar
```

### PATCH /users/:id
Update user
```
Request: UpdateUserDto
Response: User
Auth: Required
Roles: Admin, own user
```

### DELETE /users/:id
Delete user
```
Response: { success: boolean }
Auth: Required
Roles: Admin
```

### POST /users/:id/roles
Assign role to user
```
Request: { roleId: string, expiresAt?: DateTime }
Response: UserRole
Auth: Required
Roles: Admin, Registrar
```

### DELETE /users/:id/roles/:roleId
Remove role from user
```
Response: { success: boolean }
Auth: Required
Roles: Admin
```

---

## Role & Permission Endpoints

### GET /roles
List all roles
```
Response: Role[]
Auth: Required
Roles: All authenticated users
```

### GET /roles/:id
Get role by ID
```
Response: Role with permissions
Auth: Required
Roles: All authenticated users
```

### POST /roles
Create new role
```
Request: CreateRoleDto
Response: Role
Auth: Required
Roles: Super Admin
```

### PATCH /roles/:id
Update role
```
Request: UpdateRoleDto
Response: Role
Auth: Required
Roles: Super Admin
```

### DELETE /roles/:id
Delete role
```
Response: { success: boolean }
Auth: Required
Roles: Super Admin
```

### POST /roles/:id/permissions
Add permission to role
```
Request: { permissionId: string }
Response: RolePermission
Auth: Required
Roles: Super Admin
```

### GET /permissions
List all permissions
```
Response: Permission[]
Auth: Required
Roles: All authenticated users
```

---

## Admission Module Endpoints

### Leads

```
GET    /admission/leads
POST   /admission/leads
GET    /admission/leads/:id
PATCH  /admission/leads/:id
DELETE /admission/leads/:id
```

### Applications

```
GET    /admission/applications
POST   /admission/applications
GET    /admission/applications/:id
PATCH  /admission/applications/:id/status
GET    /admission/applications/:id/documents
POST   /admission/applications/:id/documents
```

### Merit Lists

```
POST   /admission/merit-lists/generate
GET    /admission/merit-lists/:programId
GET    /admission/merit-lists/:programId/export
```

### Admission Offers

```
POST   /admission/offers/generate
GET    /admission/offers
GET    /admission/offers/:id
PATCH  /admission/offers/:id/accept
PATCH  /admission/offers/:id/reject
```

### Fee Vouchers

```
GET    /admission/vouchers/:id
POST   /admission/vouchers/resend
GET    /admission/vouchers/student/:studentId
```

---

## Academic Module Endpoints

### Programs

```
GET    /academic/programs
POST   /academic/programs
GET    /academic/programs/:id
PATCH  /academic/programs/:id
DELETE /academic/programs/:id
GET    /academic/programs/:id/courses
GET    /academic/programs/:id/students
```

### Courses

```
GET    /academic/courses
POST   /academic/courses
GET    /academic/courses/:id
PATCH  /academic/courses/:id
DELETE /academic/courses/:id
```

### Sections

```
GET    /academic/sections
POST   /academic/sections
GET    /academic/sections/:id
PATCH  /academic/sections/:id
DELETE /academic/sections/:id
GET    /academic/sections/:id/students
```

### Registrations

```
POST   /academic/registrations
GET    /academic/registrations
GET    /academic/registrations/:id
PATCH  /academic/registrations/:id/status
DELETE /academic/registrations/:id
```

### Timetables

```
POST   /academic/timetables/generate
GET    /academic/timetables/:semesterId
PATCH  /academic/timetables/:id/publish
GET    /academic/timetables/:id/conflicts
```

### Grades

```
POST   /academic/grades
GET    /academic/grades/student/:studentId
GET    /academic/grades/section/:sectionId
PATCH  /academic/grades/:id
POST   /academic/grades/:id/appeal
```

---

## Examination Module Endpoints

### Date Sheets

```
POST   /examination/date-sheets/generate
GET    /examination/date-sheets/:semesterId
PATCH  /examination/date-sheets/:id/publish
GET    /examination/date-sheets/:id/export
```

### Seating Plans

```
POST   /examination/seating-plans/generate
GET    /examination/seating-plans/:examId
GET    /examination/seating-plans/room/:roomId
GET    /examination/seating-plans/:examId/export
```

### Exam Attendance

```
POST   /examination/attendance
GET    /examination/attendance/:examId
PATCH  /examination/attendance/:id
GET    /examination/attendance/report/:examId
```

### Results

```
POST   /examination/results
GET    /examination/results/:examId
PATCH  /examination/results/:id
GET    /examination/results/student/:studentId
POST   /examination/results/bulk-upload
```

### Transcripts

```
GET    /examination/transcripts/:studentId
GET    /examination/transcripts/:studentId/export
POST   /examination/transcripts/:studentId/verify
```

---

## Finance Module Endpoints

### Fee Structure

```
GET    /finance/fee-structure
POST   /finance/fee-structure
GET    /finance/fee-structure/:id
PATCH  /finance/fee-structure/:id
```

### Student Fees

```
GET    /finance/fees
GET    /finance/fees/:id
GET    /finance/fees/student/:studentId
PATCH  /finance/fees/:id/status
```

### Payments

```
POST   /finance/payments
GET    /finance/payments
GET    /finance/payments/:id
PATCH  /finance/payments/:id/verify
GET    /finance/payments/receipt/:id
```

### Scholarships

```
GET    /finance/scholarships
POST   /finance/scholarships
GET    /finance/scholarships/:id
PATCH  /finance/scholarships/:id
POST   /finance/scholarships/:id/assign
GET    /finance/scholarships/:id/recipients
```

### Reports

```
GET    /finance/reports/revenue
GET    /finance/reports/defaulters
GET    /finance/reports/scholarships
GET    /finance/reports/daily
GET    /finance/reports/monthly
```

---

## HR Module Endpoints

### Staff & Faculty

```
GET    /hr/staff
POST   /hr/staff
GET    /hr/staff/:id
PATCH  /hr/staff/:id
GET    /hr/faculty
POST   /hr/faculty
GET    /hr/faculty/:id
PATCH  /hr/faculty/:id
```

### Leave Management

```
POST   /hr/leave/request
GET    /hr/leave/requests
GET    /hr/leave/requests/:id
PATCH  /hr/leave/requests/:id/approve
PATCH  /hr/leave/requests/:id/reject
GET    /hr/leave/balance/:employeeId
```

### Attendance

```
POST   /hr/attendance/mark
GET    /hr/attendance/:employeeId
GET    /hr/attendance/report/:employeeId
PATCH  /hr/attendance/:id
```

### Payroll

```
GET    /hr/payroll/:month/:year
POST   /hr/payroll/process
GET    /hr/payroll/:id/slip
PATCH  /hr/payroll/:id/verify
```

### Performance

```
POST   /hr/performance
GET    /hr/performance/:employeeId
PATCH  /hr/performance/:id
GET    /hr/performance/report/:employeeId
```

---

## LMS Module Endpoints

### Courses

```
GET    /lms/courses
POST   /lms/courses
GET    /lms/courses/:id
PATCH  /lms/courses/:id
DELETE /lms/courses/:id
POST   /lms/courses/:id/enroll
```

### Lessons

```
GET    /lms/courses/:courseId/lessons
POST   /lms/courses/:courseId/lessons
GET    /lms/lessons/:id
PATCH  /lms/lessons/:id
DELETE /lms/lessons/:id
```

### Assignments

```
GET    /lms/assignments
POST   /lms/assignments
GET    /lms/assignments/:id
PATCH  /lms/assignments/:id
DELETE /lms/assignments/:id
POST   /lms/assignments/:id/submissions
```

### Submissions

```
GET    /lms/assignments/:assignmentId/submissions
GET    /lms/submissions/:id
POST   /lms/submissions/:id/grade
```

### Gradebook

```
GET    /lms/gradebook/course/:courseId
GET    /lms/gradebook/student/:studentId
GET    /lms/gradebook/student/:studentId/export
```

---

## Library Module Endpoints

### Catalog

```
GET    /library/books
POST   /library/books
GET    /library/books/:id
PATCH  /library/books/:id
DELETE /library/books/:id
GET    /library/books/search
```

### Borrowing

```
POST   /library/borrow
GET    /library/borrow/:id
GET    /library/borrow/student/:studentId
PATCH  /library/borrow/:id/return
GET    /library/borrow/:id/extend
GET    /library/overdue
POST   /library/fines/:borrowId/pay
```

---

## Hostel Module Endpoints

### Rooms

```
GET    /hostel/rooms
POST   /hostel/rooms
GET    /hostel/rooms/:id
PATCH  /hostel/rooms/:id
```

### Allocations

```
POST   /hostel/allocations
GET    /hostel/allocations
GET    /hostel/allocations/:id
PATCH  /hostel/allocations/:id/vacate
GET    /hostel/allocations/student/:studentId
```

### Fees

```
GET    /hostel/fees/:id
POST   /hostel/fees/:id/pay
```

---

## Transport Module Endpoints

### Routes

```
GET    /transport/routes
POST   /transport/routes
GET    /transport/routes/:id
PATCH  /transport/routes/:id
```

### Buses

```
GET    /transport/buses
POST   /transport/buses
GET    /transport/buses/:id
PATCH  /transport/buses/:id
```

### Passes

```
POST   /transport/passes
GET    /transport/passes/:id
PATCH  /transport/passes/:id/renew
GET    /transport/passes/student/:studentId
```

---

## Research Module Endpoints

### Projects

```
GET    /research/projects
POST   /research/projects
GET    /research/projects/:id
PATCH  /research/projects/:id
```

### Publications

```
GET    /research/publications
POST   /research/publications
GET    /research/publications/:id
PATCH  /research/publications/:id
```

---

## Student Affairs Module Endpoints

### Complaints

```
POST   /student-affairs/complaints
GET    /student-affairs/complaints
GET    /student-affairs/complaints/:id
PATCH  /student-affairs/complaints/:id/status
GET    /student-affairs/complaints/:id/history
```

### Events

```
GET    /student-affairs/events
POST   /student-affairs/events
GET    /student-affairs/events/:id
PATCH  /student-affairs/events/:id
POST   /student-affairs/events/:id/register
```

---

## QEC Module Endpoints

### Surveys

```
GET    /qec/surveys
POST   /qec/surveys
GET    /qec/surveys/:id
POST   /qec/surveys/:id/responses
GET    /qec/surveys/:id/results
GET    /qec/surveys/:id/report
```

---

## Alumni Module Endpoints

### Profiles

```
GET    /alumni/profiles
POST   /alumni/profiles
GET    /alumni/profiles/:id
PATCH  /alumni/profiles/:id
```

### Networking

```
GET    /alumni/events
POST   /alumni/events
GET    /alumni/networking
POST   /alumni/jobs
```

---

## Analytics Module Endpoints

### Dashboards

```
GET    /analytics/dashboard/executive
GET    /analytics/dashboard/faculty
GET    /analytics/dashboard/student
GET    /analytics/dashboard/admin
```

### Reports

```
GET    /analytics/reports/enrollment
GET    /analytics/reports/performance
GET    /analytics/reports/finance
GET    /analytics/reports/attendance
POST   /analytics/reports/custom
```

### Predictions

```
GET    /analytics/predictions/enrollment
GET    /analytics/predictions/dropout-risk
GET    /analytics/predictions/performance
```

---

## Notification Module Endpoints

### Send Notifications

```
POST   /notifications/send
POST   /notifications/broadcast
GET    /notifications/user/:userId
PATCH  /notifications/:id/read
DELETE /notifications/:id
```

---

## Integration Module Endpoints

### External Systems

```
GET    /integration/logs
GET    /integration/logs/:id
POST   /integration/test/:service
```

---

## System Endpoints

### Health Check

```
GET    /health
Response: { status: "ok", timestamp: DateTime }
Auth: None
```

### Documentation

```
GET    /api/docs              # Swagger UI
GET    /api/docs-json         # OpenAPI JSON
```

### Audit Logs

```
GET    /audit-logs
GET    /audit-logs/:id
Query: entityId=&action=&startDate=&endDate=
Auth: Required
Roles: Admin, Super Admin
```

---

## Response Format

### Success Response (200)
```json
{
  "success": true,
  "data": { ... },
  "message": "Operation successful",
  "timestamp": "2026-06-01T10:30:00Z"
}
```

### Paginated Response (200)
```json
{
  "data": [...],
  "pagination": {
    "total": 100,
    "page": 1,
    "pageSize": 10,
    "totalPages": 10
  }
}
```

### Error Response (4xx/5xx)
```json
{
  "success": false,
  "error": "Error Code",
  "message": "Human-readable error message",
  "statusCode": 400,
  "timestamp": "2026-06-01T10:30:00Z"
}
```

---

## Authentication Header

```
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

Or use Swagger UI "Authorize" button.

---

## Rate Limiting

```
Rate Limit: 1000 requests/hour per user
Burst Limit: 100 requests/minute per user
```

---

## CORS Policy

```
Allowed Origins: configured in backend/nest/.env
Allowed Methods: GET, POST, PUT, PATCH, DELETE, OPTIONS
Allowed Headers: Content-Type, Authorization
Credentials: true
```

---

## Pagination

Default pagination: `skip=0&take=10`

```
GET /api/admission/applications?skip=0&take=20&sort=createdAt&order=desc
```

---

## Search & Filter

```
GET /api/users?search=john&role=Faculty&status=ACTIVE&skip=0&take=10
```

---

## Sorting

```
GET /api/courses?sort=name&order=asc
GET /api/courses?sort=-createdAt  # Descending with minus
```

---

**Total API Endpoints**: 150+  
**Modules**: 17 + Core  
**Documentation**: Auto-generated via Swagger

Generate OpenAPI spec: `http://localhost:4000/api/docs-json`

---

**Last Updated**: June 1, 2026  
**Version**: 1.0.0
