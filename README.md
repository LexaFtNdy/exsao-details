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
│  Client     │    (Capacitor)  │    Page              │
│  (Tauri)    │                 │    (Nuxt 3)          │
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

## 📋 Core Features

### Question Engine
- **5 question types:** Multiple Choice, Multiple Answer, Matching, True/False, and Essay
- All types support **image attachments** in both questions and answer options
- Questions can be created manually via a designed UI or **bulk-imported** from existing question banks
- **Question validation workflow** — questions must be reviewed and approved by a designated validator (assigned by school) before they can be scheduled for an exam

### Essay Grading
- Built-in grading interface for teachers — no external tools needed
- Before/after score tracking for fair and transparent grading

### Exam Lifecycle
- **Triple-token system:** Entry Token → Exam Token → Exit Token
- **Auto-generated exam cards** for student distribution
- **Auto-remedial:** If an exam is configured with remedial, eligible students are flagged automatically — teachers just request the school admin to schedule it

### Proctoring & Monitoring
- **Near-live monitoring dashboard** with intentional delay to prevent proctors from tipping off students
- **Proctor control panel:** Add/reduce exam time, force-submit answers, reset sessions for anomalies — every action requires a logged reason
- **Two teacher subtypes:** Subject Teacher (creates questions & grades) and Proctor (supervises exams) — with strict data visibility isolation between roles during active exams
- Proctors can be assigned manually, allowing external supervisors from outside the school

### Anti-Cheat & Violations
- Desktop client runs in **kiosk mode** (Tauri) — blocks Alt+Tab, screenshots, and external apps
- **Violation tracking system** — auto-logs tab switches, window blur events, and suspicious behavior
- Accumulated violations trigger **automatic student blocking** — enforced at the admin level
- Since the introduction of secure containers (Tauri/Capacitor), students have no way to bypass the lockdown

### Collaborative Exams (Cross-Tenant)
- **Room-based system** — one school creates a room (becomes Room Master), gets a room code, other schools join by entering the code
- Questions sync automatically from the Room Master's question bank
- **Cross-school leaderboard** to drive competition between participating schools
- IRT analysis works across collaborative exams

### Reports & Analytics
- **Export formats:** PDF and Excel — both simple summary and detailed per-question breakdown
- **IRT (Item Response Theory)** analysis for item-level diagnostics using standard psychometric parameters
- Classroom-integrated exports for teacher convenience

---

## ⚡ Key Technical Challenges Solved

### Concurrency at Scale
600+ students submitting auto-save answers every 30 seconds and clicking "Submit" at the exact same millisecond.

- **Pessimistic locking** (`SELECT ... FOR UPDATE`) on score calculation to prevent race conditions
- **Atomic database updates** for quota deduction: `UPDATE SET quota = quota - 1 WHERE quota > 0`
- **Database transaction wrapping** on all multi-table mutations — if one fails, everything rolls back
- **Connection pooling** and N+1 query elimination via eager loading

### Multi-Tenant Data Isolation
- School-level data isolation with tenant-scoped queries
- Centralized license management via Super Admin Dashboard
- Per-tenant configuration (exam rules, grading scales, report templates)

### Report Generation
- Multi-byte string safety (UTF-8 sanitization) for essay responses containing Unicode
- Efficient bulk exports with PhpSpreadsheet and DomPDF

---

## 👥 User Roles

| Role | Capabilities |
|---|---|
| **Super Admin** | Tenant CRUD, license management, school provisioning |
| **Admin (School)** | Manage teachers, students, classrooms, exam schedules, violation review |
| **Teacher** | Create/import questions, grade essays, export reports, question validation |
| **Proctor** | Monitor exams, control sessions, manage anomalies (assignable to external staff) |
| **Student** | Take exams, auto-save progress, view results |

---

## 🔧 Infrastructure

- **Server:** Ubuntu VPS, PHP 8.4-FPM, Nginx, MySQL 8
- **Deployment:** Git-based with custom `exsao-deploy` CLI utility
- **Backup:** Automated daily database backup to Google Drive via rclone
- **Process:** OPcache + FPM reload for zero-downtime deployments

---

## 📖 Origin Story

ExSAO wasn't born from a business plan — it was born from getting burned.

I was freelancing when a school hired me to build a basic local CBT app. Standard scope, standard deal. When it came time to pay, they refused to honor the agreement. I wasn't about to hand over my source code for free, so we negotiated: I'd host it myself on a VPS, set everything up, but the code stays mine.

After that first exam season ended, I looked at what I'd built and thought — *this could be something bigger*. That's when the SaaS idea clicked. I rebuilt the architecture from scratch: multi-tenant isolation, a super admin dashboard for school provisioning, and eventually wrapped the client into desktop (Tauri) and mobile (Capacitor) containers when schools started requesting dedicated apps.

### The Day Everything Broke

The real test came during a **cross-tenant collaborative exam** — multiple schools sharing the same exam simultaneously. I developed the feature, tested every scenario I could think of, and shipped it at 5 AM the night before the exam. Went to sleep.

At 7 AM, my phone exploded.

**580 students across 3 schools couldn't use the app.** Some got kicked out immediately after logging in. Some couldn't install the app — Google flagged it as a security threat. Some were mid-exam when the system crashed, and when they tried to resume, their progress was gone. One school had a power outage on top of everything.

I woke up on 1 hour of sleep and coded non-stop until 11 PM that night — live-patching a production system while teachers from multiple schools were sending complaints in real-time. The pressure was unlike anything I'd experienced: every decision I made could either save or permanently corrupt hundreds of student answer records.

The worst moment? After I finally stabilized the server, I discovered that **some student answers had vanished from the database entirely**. Auto-save had been running, but the data wasn't there. I spent that entire night forensically recovering answer data — piecing together fragments, rebuilding records, all while knowing there was a second collaborative exam scheduled for the next morning.

I got it done. Every student's data was recovered. The second exam ran flawlessly.

That week taught me more about production engineering than any course or tutorial ever could — the kind of lessons you only learn when 580 real students are counting on your code to not fail.

---

## Author

**LexaFtNdy** — Full-Stack Developer
