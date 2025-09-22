TUT Clinics — README

TUT Clinics is a campus-aware digital health platform for Tshwane University of Technology (TUT).
It lets students book appointments, view visit history, receive awareness messages, and lets clinic staff manage schedules, referrals, and clinic records. The system integrates with the university API for student identity (student number is the canonical identifier).

Table of contents

Project overview

Features

Tech stack

Repository layout

Pre-requisites

Quick start — database setup

Environment variables

Running locally (development)

API examples (curl)

Database schema & SQL file

Security & privacy notes (POPIA)

Deployment notes

Testing

Contribution guide

License & contact

Project overview

TUT Clinics is a full-stack web application (web-first, mobile-ready) that improves access to campus healthcare services by digitizing appointment booking, referrals and clinic records while preserving student privacy and complying with local data protection rules. Students are identified via the university API and all app data references the student_number.

Features

Student

Secure login (university-backed)

Dashboard: upcoming appointments, messages, campus info

Book / reschedule / cancel appointments (campus-aware)

View clinic visit history & referral status

Update profile picture only

Staff (Nurse / Doctor / Counselor)

View/confirm/complete appointments

Create clinic visit records & notes

Create referrals (intra-campus; inter-campus within Gauteng)

Publish awareness messages

Admin

Manage campuses, staff, policies (e.g. referral rules)

Reports & analytics (appointments, referrals, campaign uptake)

Emergency contact management

Tech stack

Database: MySQL

Backend: Node.js + Express (recommended) or Django REST Framework

Frontend: React (web); design mobile-first, future React Native port possible

Auth: JWT for session tokens; integrate with University API for student verification

Notifications: SMTP (email) / SMS provider (configurable)

Hosting: Any cloud provider (AWS/GCP/Azure) or university DC — use managed MySQL for production

Repository layout (suggested)
/backend
  /src
    /controllers
    /models
    /routes
    /services    -- e.g., university API integration
  package.json
  .env.example

/frontend
  /src
    /components
    /pages
  package.json

/sql
  tut_clinics_schema.sql   -- full DB schema + seed data (run to create DB)
README.md

Pre-requisites

Node.js (v18+) and npm/yarn for running backend/frontend

MySQL 5.7+ or 8.x

Git

Optional: Postman for API testing

Quick start — database setup

Create database and import schema (run from terminal):

# replace /path/to/ with real path to the SQL file
mysql -u root -p < ./sql/tut_clinics_schema.sql


Alternatively, create DB and import manually:

-- in MySQL client
CREATE DATABASE tut_clinics;
USE tut_clinics;
SOURCE /full/path/to/sql/tut_clinics_schema.sql;


The SQL file creates all tables (students, staff, campuses, users, appointments, referrals, clinic_visits, awareness_messages, emergency_contacts) and inserts sample seed data.

Environment variables

Create a .env (backend) based on .env.example. Example variables:

# Database
DB_HOST=localhost
DB_PORT=3306
DB_USER=root
DB_PASSWORD=your_db_password
DB_NAME=tut_clinics

# JWT
JWT_SECRET=your_jwt_secret_here
JWT_EXPIRES_IN=7d

# University API (example)
UNI_API_BASE_URL=https://university.example/api
UNI_API_KEY=your_uni_api_key

# Email/SMS (optional)
SMTP_HOST=smtp.example.com
SMTP_PORT=587
SMTP_USER=mailer@example.com
SMTP_PASS=supersecret
SMS_API_KEY=your_sms_api_key

# App
PORT=4000


Important: Never commit .env to source control.

Running locally (development)
Backend (Node.js - example)
cd backend
npm install
# Ensure .env is set up
npm run dev   # assumes nodemon setup

Frontend (React - example)
cd frontend
npm install
npm start


You should now have frontend at http://localhost:3000 (or configured port) and backend at http://localhost:4000.

API examples (curl)

NOTE: The real system authenticates against the University API. For local development you can seed the users table or mock the university verification.

Login (POST)

curl -X POST http://localhost:4000/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"alice.khumalo@tut4life.ac.za","password":"password123"}'


Get student profile

curl -H "Authorization: Bearer <JWT_TOKEN>" \
  http://localhost:4000/students/20230001


List appointments for student

curl -H "Authorization: Bearer <JWT_TOKEN>" \
  "http://localhost:4000/appointments?student_number=20230001"


Create appointment

curl -X POST http://localhost:4000/appointments \
  -H "Authorization: Bearer <JWT_TOKEN>" \
  -H "Content-Type: application/json" \
  -d '{
    "student_number":"20230001",
    "staff_id": 1,
    "campus_id": 1,
    "appointment_type":"General Checkup",
    "appointment_date":"2025-10-18T14:30:00",
    "duration_minutes":30,
    "reason":"Routine check-up"
  }'


Create referral

curl -X POST http://localhost:4000/referrals \
  -H "Authorization: Bearer <JWT_TOKEN>" \
  -H "Content-Type: application/json" \
  -d '{
    "appointment_id": 12,
    "from_staff_id": 3,
    "to_campus_id": 2,
    "notes":"Refer to specialist counseling",
    "status":"Pending"
  }'

Database schema & SQL file

The full, ready-to-run SQL schema is in /sql/tut_clinics_schema.sql. It contains table definitions and sample seed data for quick testing.

Tables included: students, campuses, staff, users, appointments, referrals, clinic_visits, awareness_messages, emergency_contacts.

All student references use student_number as the canonical key. Staff use staff_id.

Security & privacy notes (POPIA)

Data minimisation: store only necessary student fields locally; authoritative data pulled from University API.

Transport encryption: TLS (HTTPS) required for all endpoints.

At-rest encryption: sensitive fields should be encrypted (database-level or application-level).

Access controls: RBAC enforced — staff only see students visited at their campus; admins have broader access.

Audit logs: every read/write to medical data must be logged with user id & timestamp.

Retention & backups: define retention policies aligned with university policy and POPIA. Keep encrypted backups.

Deployment notes

Use managed MySQL for production (Amazon RDS / Cloud SQL / Azure Database).

Use environment-specific secrets management (AWS Secrets Manager, Azure KeyVault).

Autoscale backend containers behind a load balancer.

Schedule DB backups and daily integrity checks.

Add health checks and monitoring (Prometheus/Grafana or cloud provider stack).

Testing

Seed DB with sample data from /sql and run integration tests against backend endpoints.

Add unit tests for business rules (e.g., referral province restrictions, capacity enforcement).

Include E2E tests (Cypress or Playwright) for common flows: login → book appointment → nurse confirm → referral.

Contribution guide

Fork the repo and create a feature branch feature/your-feature.

Keep commits small and focused.

Open a PR describing the change and link related ticket.

Include tests for any business logic added.

All PRs must be reviewed before merging.

Next steps & recommended additions

Integrate University SSO / OAuth2 (if available) for seamless login.

Add SMS gateway for appointment reminders.

Build a mobile app (React Native) reusing frontend components.

Implement role-based UI with stricter access rules for staff/admin.

Run a security review and POPIA compliance audit before pilot.

License & contact

License: Add a license file (e.g., MIT) — update here once chosen.

Contact / Maintainer: Add project owner / maintainer details here (email, Slack, phone).
