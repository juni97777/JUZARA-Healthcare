# JUZARA Health — starter application

A mobile-first healthcare platform starter with a React/Vite patient UI, Express API, PostgreSQL and Prisma data model. The UI includes a Gen-Z-inspired dashboard and responsive navigation. The backend provides session authentication, role checks, facility/doctor/schedule reads, transactional appointment creation, queue status updates, articles, notifications and audit logging.

> **Important:** This is a development starter, not a certified or production-ready healthcare system. Do not enter real patient identifiers, diagnoses, prescriptions, laboratory results, or other sensitive health information until the deployment has undergone security, privacy, clinical, legal and operational review. Demo facility and schedule data are explicitly unverified. No real clinic address, doctor credential, laboratory result, email/SMS verification, push notification, video consultation, map provider, or PDF service is configured.

## Stack
- Client: React 18, TypeScript, Vite, Lucide icons, responsive CSS.
- API: Node.js, Express, TypeScript, session cookies, bcrypt password hashing, Zod validation, Helmet and rate limiting.
- Data: PostgreSQL + Prisma ORM.
- Time handling: initial schedule convention is Asia/Makassar (WITA); production should store timestamps in UTC and convert at the API/UI boundary.

## Requirements
- Node.js 20+
- PostgreSQL 15+

## Run locally

1. Create a PostgreSQL database named `juzara_health`.
2. Copy `server/.env.example` to `server/.env`, set `DATABASE_URL` and replace `SESSION_SECRET` with a long random secret.
3. From the repository root:

   ```bash
   npm install
   npm run db:generate
   npm run db:migrate -w server -- --name init
   npm run db:seed
   npm run dev
   ```

4. Open `http://localhost:5173` for the web client. API health endpoint: `http://localhost:4000/api/health`.

The UI opens in preview mode. Patient registration endpoint is `POST /api/auth/register`; login is `POST /api/auth/login` with `{ "email": "...", "password": "...", "portal": "patient" }`. Use only test data locally. A real doctor login requires an admin-controlled credential onboarding and verification workflow; public doctor self-registration is intentionally absent.

## Current API routes
- `GET /api/health`
- `POST /api/auth/register`
- `POST /api/auth/login`
- `POST /api/auth/logout`
- `GET /api/auth/me`
- `GET /api/facilities?q=&type=`
- `GET /api/doctors`
- `GET /api/schedules?date=&facilityId=&doctorId=`
- `POST /api/appointments` (patient session required)
- `GET /api/appointments/mine` (patient session required)
- `PATCH /api/queues/:id/status` (doctor/admin session required)
- `GET /api/articles`
- `GET /api/notifications` (authenticated)
- `PATCH /api/notifications/:id/read` (authenticated)

## Important implementation gaps before production
1. Add a PostgreSQL-backed session store (the default Express MemoryStore is development-only), CSRF protection, secure cookie/domain configuration and secret rotation.
2. Harden booking concurrency with database-level locking/retry and enforce that requested slotTime is an actual generated slot within the schedule. Add cancellation policy and idempotency keys.
3. Implement full doctor onboarding, credential review, admin RBAC, facility-scoped authorization and least-privilege access to patient records.
4. Add verified facility master data, doctor credential verification, holidays/leave/special schedules and schedule administration.
5. Implement consultation messaging with access checks, secure attachment storage/scanning and clinical record workflows.
6. Implement lab staff role permissions, signed/validated result provenance, immutable result history and patient access controls.
7. Add email/SMS verification and password reset provider, notifications worker, real-time queue transport, PDF generation, OpenStreetMap/Leaflet map, article CMS and push notifications.
8. Add automated unit, integration, concurrency, authorization, accessibility and end-to-end tests. No production QA certification is implied by this starter.
9. Complete Indonesian health-data/privacy and medical-service compliance review (including applicable PDP and health-sector rules), threat modeling, penetration testing, encrypted backups, monitoring and incident response.

## Demo seed
`npm run db:seed` creates one clearly-labelled demo facility and the five doctor display names requested. Doctor accounts are unverified and have unusable passwords; they cannot log in. The facility address and schedules are marked as demo/unverified.

## Design direction v0.2 — illustrated Gen-Z healthcare
The interface palette and dashboard illustration have been refreshed to follow the supplied visual reference: turquoise sky, warm peach/orange accents, soft green landscape cues, rounded cards, friendly tiger mascot motif, and responsive mobile layout. The reference image is used as a visual direction, not copied as a screen asset. The interface remains a web dashboard rather than a static phone mockup.

## Delivery status — do not treat as production complete
This archive is an implementation starter with working API routes for session login/register/logout, facility and verified-doctor reads, schedule reads, appointment creation, queue status, published articles, notifications and audit events. It is **not** a fully completed or production-deployed healthcare platform. The following work remains before real patient use:

- Patient registration UI, password reset and verified contact workflow.
- Separate verified doctor/admin login screens and complete facility-scoped authorization.
- Complete booking UX (date/slot picker, schedule validation, cancellation and idempotency).
- Live queue refresh / push notifications.
- Consultation chat, secure file attachments and clinical record workflow.
- Lab staff role, validated laboratory result entry, provenance and PDF report.
- Article CMS, map provider configuration, real facility directory and verified provider data.
- PostgreSQL-backed session store, CSRF defense, secure secret management, encrypted storage/backups and monitoring.
- Automated tests including concurrency, role isolation, privacy, accessibility and end-to-end workflows.
- Deployment to a managed host with HTTPS, operational runbooks and Indonesian healthcare/privacy compliance review.

### Seed data warning
All seeded facilities and schedules are explicitly demonstrational and unverified. The five doctor names are seed display labels only; no professional identity, credential, facility affiliation, address or clinical availability is asserted. Do not use these seed records to make a real appointment.

### Required production configuration
Set a strong random `SESSION_SECRET`, a private PostgreSQL `DATABASE_URL`, production `CLIENT_ORIGIN`, HTTPS and secure cookie settings. Replace the development session store with a persistent store. Configure verified facility/provider records and third-party services before enabling corresponding features. Never enter real NIK, diagnoses, prescriptions, laboratory results or other sensitive health information into this development build.

## Doctor portal, digital consultation, laboratory and notifications (v0.3)

### Doctor-only access
- The login dialog has a separate **Portal dokter** mode and sends `portal: "doctor"` to the API.
- Doctor login is accepted only when the backend session role is `DOCTOR` and `Doctor.verifiedAt` is set. Patient role selection in the browser cannot grant doctor access.
- There is no public doctor self-registration. A signed-in `ADMIN` account must provision a doctor through `POST /api/admin/doctors`.
- The seed doctor display names are intentionally unverified and have unusable passwords. They cannot sign in.

### First administrator and doctor onboarding
1. Copy `server/.env.example` to `server/.env`.
2. Set a private `ADMIN_EMAIL` and a unique `ADMIN_PASSWORD` of at least 12 characters, plus `DATABASE_URL` and a strong `SESSION_SECRET`.
3. Run `npm run db:generate`, `npm run db:migrate -w server -- --name init`, then `npm run db:seed`.
4. Log in to the API using `POST /api/auth/login` with `{ "email": "admin-email", "password": "admin-password", "portal": "admin" }` and retain the session cookie securely.
5. The admin creates an account only after independently checking the doctor's professional credentials. Example request (use the same authenticated admin session):

   ```bash
   curl -X POST http://localhost:4000/api/admin/doctors \\
     -H 'Content-Type: application/json' \\
     -b admin-cookie.txt -c admin-cookie.txt \\
     -d '{"email":"doctor@example.com","password":"use-a-unique-strong-password","displayName":"Nama Dokter","specialty":"Dokter umum","licenseNumber":"ID-KREDENSIAL-TERVERIFIKASI","facilityId":"FACILITY_ID","weekday":1,"startTime":"08:00","endTime":"12:00","quota":20}'
   ```

   `weekday` follows JavaScript numbering (0 Sunday ... 6 Saturday). `facilityId` must be an existing facility. The administrator is responsible for checking the identity/license against authoritative professional records before calling this endpoint; setting `verifiedAt` is an administrative attestation, not an external credential verification service.
6. The doctor can now use **Portal dokter** with the provisioned email and password. Passwords are hashed with bcrypt; never share a common doctor password.

### Implemented modules
- Doctor dashboard: daily appointments, counts, schedules and assigned patients.
- Patient/queue workflow: doctor can update queue status only for appointments assigned to that doctor; patient receives an in-app notification.
- Digital consultation: patient and assigned verified doctor can exchange text messages; doctor can start/complete consultation and save structured consultation notes.
- Laboratory: doctor can create a result only for a patient with an appointment relationship to that doctor; result stores source reference, validating account and timestamp. Patients see validated results only.
- Notifications: authenticated user-specific list, unread marker, mark one read and mark all read. Appointment, queue, consultation and laboratory events create in-app notifications.
- Patient views: consultation conversation, validated lab results and notification list.

### Added API routes
- `POST /api/admin/doctors` (ADMIN only; account provisioning)
- `GET /api/doctor/dashboard` (verified DOCTOR only)
- `GET /api/doctor/appointments`
- `GET /api/doctor/consultations`
- `POST /api/doctor/consultations/:id/start`
- `PUT /api/doctor/consultations/:id/record`
- `POST /api/doctor/consultations/:id/complete`
- `POST /api/consultations/:id/messages` (patient owner or assigned verified doctor only)
- `GET /api/consultations/mine` (PATIENT only)
- `GET /api/doctor/laboratory/:patientId` (assigned verified doctor only)
- `POST /api/doctor/laboratory` (assigned verified doctor only)
- `GET /api/laboratory/mine` (PATIENT only; validated results only)
- `GET /api/notifications`, `PATCH /api/notifications/:id/read`, `PATCH /api/notifications/read-all`

### Remaining production requirements
This is still a development build. Dependencies must be installed and the app must be built, run against a configured PostgreSQL database, and tested in the target environment. The current consultation is text-based; no video call or secure attachment upload is configured. Notifications are in-app only; WebSocket/push/email/SMS delivery is not configured. Laboratory result entry is a clinician workflow, not a connection to a real analyzer/LIS. The default Express MemoryStore is development-only and must be replaced with a persistent session store; CSRF protection, encrypted file storage, secure backups, monitoring, penetration testing and healthcare/privacy compliance review remain mandatory before any real patient data is used.


## Visual design v0.4
The frontend styling now follows the supplied illustrated healthcare reference: turquoise sky, warm cream cards, orange actions, playful tiger-doctor mascot accents, clinic/landscape motifs, and matching styling for patient and doctor portals. See `docs/DESIGN-SYSTEM.md` and `docs/juzara-visual-reference.png`.
