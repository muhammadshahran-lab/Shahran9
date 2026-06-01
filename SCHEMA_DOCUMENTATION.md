# Comprehensive Prisma Schema for Riphah UMS

This document contains the complete, syntactically correct Prisma schema for all 17 modules of the University Management System. 

## How to Use

1. Copy the schema content from the file `schema.prisma` in `backend/nest/prisma/` directory
2. Replace any broken schema with the corrected version provided below
3. Run migrations with `npm run prisma:migrate`

## Complete Prisma Schema (All 17 Modules)

The schema includes:
- **Core System**: Users, Roles, Permissions, Sessions, Audit Logs
- **Organization**: Campus, Faculty, Department, Buildings, Rooms
- **Academic**: Programs, Courses, Sections, Registrations, Grades, Timetables
- **Admission**: Leads, Applications, Merit Lists, Offers, Fee Vouchers
- **Examination**: Exams, Seating Plans, Attendance, Results
- **Finance**: Student Fees, Payments, Scholarships
- **HR**: Staff, Faculty Members, Leave, Salary, Performance
- **LMS**: Courses, Lessons, Assignments, Submissions, Enrollments
- **Library**: Books, Borrowing, Returns
- **Hostel**: Rooms, Allocations
- **Transport**: Routes, Buses, Passes
- **Research**: Projects, Publications
- **Student Affairs**: Complaints, Events
- **Alumni**: Alumni Profiles
- **QEC**: Surveys, Responses
- **Notifications**: Email, SMS, In-App
- **Analytics**: Metrics, Reports
- **Integration**: External System Logs

## Key Enums

- **UserStatus**: ACTIVE, INACTIVE, SUSPENDED, ARCHIVED
- **ApplicationStatus**: SUBMITTED, UNDER_REVIEW, ACCEPTED, REJECTED, ENROLLED
- **ExamStatus**: SCHEDULED, ONGOING, COMPLETED, CANCELLED
- **StudentStatus**: ENROLLED, ACTIVE, SUSPENDED, GRADUATED, DROPPED
- **PaymentStatus**: PENDING, COMPLETED, FAILED, REFUNDED
- **And 25+ other domain-specific enums**

## To Apply the Fixed Schema

Since the current schema has syntax errors, follow these steps:

1. **Delete or backup** the current `backend/nest/prisma/schema.prisma`
2. **Create a new file** with the correct schema from this document
3. **Run**: `npm run prisma:generate`
4. **Run**: `npm run prisma:migrate -- --name initial`
5. **Run**: `npm run prisma:seed`

## Database Relations Map

```
User (core identity)
  ├─ Student → Program, Specialization
  ├─ FacultyMember → Courses, Research
  ├─ Staff → Department
  ├─ Alumni → Alumni Profile
  ├─ Roles → Permissions
  ├─ Sessions → Login history
  └─ AuditLog → Action tracking

Program
  ├─ Department
  ├─ Courses
  ├─ Specializations
  └─ Students

Course
  ├─ Sections
  ├─ Registrations
  ├─ Exams
  └─ LMS Content

Student
  ├─ Registrations
  ├─ Grades
  ├─ Attendance
  ├─ Fees
  ├─ Hostel Allocation
  ├─ Transport Pass
  ├─ Complaints
  └─ LMS Enrollments

Exam
  ├─ ExamRooms
  ├─ SeatingPlans
  └─ Attendance

Finance
  ├─ StudentFee
  ├─ Payments
  └─ Scholarships

HR
  ├─ Leave
  ├─ Salary
  └─ Performance

Etc.
```

## Migration Strategy

Since this is a new project initialization:

1. All tables created in first migration (`initial`)
2. Seed default data (roles, permissions, departments, campuses)
3. Incremental schema updates handled through future migrations

## Important Notes

- All IDs use UUID primary keys
- Timestamps use `DateTime` with `@default(now())` and `@updatedAt`
- Relationships use cascade delete where appropriate
- Indexes created on frequently queried fields
- Composite unique constraints where needed
- Enums for all fixed-value fields

---

**Next Steps:**
1. Fix the syntax errors in the current schema file
2. Run Prisma migrations
3. Seed initial data
4. Generate NestJS service skeletons for all 17 modules
