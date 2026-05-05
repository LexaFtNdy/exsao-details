# ExSAO — Computer-Based Test Ecosystem

> Enterprise multi-tenant CBT (Computer-Based Test) platform serving **11+ schools** across East Java with **600+ concurrent exam sessions**. Source code is proprietary — this repository serves as an architectural showcase.

## ⚠️ Disclaimer
The source code for this project is **strictly private** under NDA. This repository exists solely for portfolio and architectural demonstration purposes. No source code is included.

---

## 🏗 Architecture

ExSAO is not a single application — it's a **5-component ecosystem** working in concert:

```
┌─────────────────────────────────────────────────────┐
│                  Super Admin Dashboard              │
│              (Multi-Tenant Management)              │
└──────────────────────┬──────────────────────────────┘
                       │
┌──────────────────────▼──────────────────────────────┐
│              ExSAO CBT Server (SaaS)                │
│         Laravel 11 · Vue 3 · Inertia.js             │
│     Multi-tenant · 600+ concurrent sessions         │
├─────────────┬─────────────────┬─────────────────────┤
│  Desktop    │    Mobile       │    Landing           │
│  Client     │    Client       │    Page              │
│  (Tauri)    │    (Capacitor)  │    (Nuxt 3)          │
└─────────────┴─────────────────┴─────────────────────┘
```

| Component | Stack | Purpose |
|---|---|---|
| **CBT Server** | Laravel 11, Vue 3, Inertia.js, MySQL | Core SaaS — exam engine, grading, reports |
| **Desktop Client** | Tauri (Rust), Vue 3 | Secure exam browser — anti-cheat, kiosk mode |
| **Mobile Client** | Capacitor, Vue 3 | Android exam app for BYOD environments |
| **Super Admin** | Laravel, Vue 3 | Tenant management, licensing, school provisioning |
| **Landing Page** | Nuxt 3 | Public-facing marketing site |

---

## ⚡ Key Technical Challenges Solved

### Concurrency at Scale
600+ students submitting auto-save answers every 30 seconds and clicking "Submit" at the exact same millisecond.

- **Pessimistic locking** (`SELECT ... FOR UPDATE`) on score calculation to prevent race conditions
- **Atomic database updates** for quota deduction: `UPDATE SET quota = quota - 1 WHERE quota > 0`
- **Database transaction wrapping** on all multi-table mutations — if one fails, everything rolls back
- **Connection pooling** and N+1 query elimination via eager loading

### Anti-Cheat & Exam Security
- Desktop client runs in **kiosk mode** (Tauri) — blocks Alt+Tab, screenshots, and external apps
- **Token-based session binding** — one student, one device, one active session
- **Violation tracking system** — auto-logs tab switches, window blur events, and suspicious behavior
- Real-time violation counter visible to teachers during active exams

### Multi-Tenant Data Isolation
- School-level data isolation with tenant-scoped queries
- Centralized license management via Super Admin Dashboard
- Per-tenant configuration (exam rules, grading scales, report templates)

### Report Generation
- **Excel & PDF exports** with per-student, per-question breakdown
- Multi-byte string safety (UTF-8 sanitization) for essay responses containing Unicode
- Classroom-integrated reports for teacher convenience

---

## 👥 User Roles

| Role | Capabilities |
|---|---|
| **Super Admin** | Tenant CRUD, license management, school provisioning |
| **Admin (School)** | Manage teachers, students, classrooms, exam schedules |
| **Teacher** | Create exams, import questions (DOCX/PDF), grade essays, export reports |
| **Student** | Take exams, auto-save progress, view results |

---

## 🔧 Infrastructure

- **Server:** Ubuntu VPS, PHP 8.4-FPM, Nginx, MySQL 8
- **Deployment:** Git-based with custom `exsao-deploy` CLI utility
- **Backup:** Automated daily database backup to Google Drive via rclone
- **Process:** OPcache + FPM reload for zero-downtime deployments

---

## Author

**LexaFtNdy** — Full-Stack Developer
