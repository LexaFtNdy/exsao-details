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

tambahan aja disini gua mau cerita keluh kesah gua aja si ntar lu poles sendiri jadi bahasa teknikal, jadi exsao ini ga semena mena terbentuk, awalanya itu gua kan freelance fullstack developer, dari salah satu client gua ini dia pgn dibuatkan aplikasi cbt untuk ujian basic fitur la, buat deploy di local server mereka, nah udah gua buatin tu singkat waktu, basic ini masihan semua fitur gaada, ya cbt pada umumnya buat local gitu aja, nah pas di akhir pembayaran ternyata sekolah x ini dia gamau bayar sesuai perjanjian intinya underpaid banget la, bahkan bisa dibilang free jatuhnya, nah dari sana itu gua yakali njir nyerahin source codde, akhirnya gua disksui sama guru pihak sana buat yaudah gua pinjemin gua setupin tapi soruce code gaakan gua kasih, akhirnya ketemu solusi pake vps, yang mana setup juga gua, nah dari sana setelah ujian selesai gua mikir, ini bisa si di kembangin lagi, akhirnya keputusan ngebuat aplikasi saas tampil pada saat itu, dan sekarang gua juga harus prioritas mana dulu, berhubung ini cepet cepet dipake lagi beberapa minggu akhirnya gua memutuskan buat beta test free ke sekolah mereka mengenai fitur yang gua buat, awal awal gua fokus ngedevlop env saas super admin gua sama cbt gimana caranya mereka berdua komunikasi, terus seiring berjalannya waktu, ada masukan dari client gua kalau minta dibikinin aplikasi merka nyebutnya yaudah gua warp ke dalam container desktop sama mobile, untuk problem dan tanttangan jelas banyak si, gua sampe lupa apa aja, tapi yang paling gua inget pas producction, itu kan h-1 sebelum production clinet ngeyel minta update fitur apa ya pas itu, oiya mereka pgn ujian gabungan antar sekolah atau antar tenant la teknisnya, nah gua develop gua testing aman tu, semua secenario test udah gua lakuin, kelar jam 5 dini hari, akhirnya gua mutusin la buat tidur, jam 7 anj, gua di telp dapet laporan sekolah A, B, C sekolah yang pakek fitur ujian bersama atau kolaborasi gua ngebug brok gelok, gabisa masuk aplikasi, error semua, ada yang baru masuk langsung kepental, ada yang gabisa install aplikasi, ada yang aplikasi gua kedeteksi keamanan google, ada yang udah ngerjain tiba tiba error terus pas mau ngelanjut ngerjain gabisa, ada yang listrik mati, ada yang gilak banyak bet dah, listrik mati juga, pada saat itu bisa dikatakan nasib 580 siswa berada di tangan gua, dan aplikasi bener bener ancur gabisa dipakek siswa bener bener end user sebernya gabisa dipakek, akhirnya dari sana perwakilan guru guru pada complain sampein ke gua, gimana caranya gua mesti jalanin aplikasi ini bisa jalan pas lagi high concurrency, gimana caranya gua updatte tanpa bikin jawaban siswa ilang atau corrupt, dan banyak lagi, itu gua bener bener kebangun tidur cuma 1 jam terus benerin kode sampe jam 11 malem, dan lu tau apa? PUNCAKNYA kode udah stabil, sekarang jawaban siswa gaada di database, mampus kata gua, akhirnya gua pusing sambil di complain guru malemnya, gua barusan tau pas baru baca wa lagi soalnya nilai anak anaknya pada kaga muncul di laporan atau hasil akhir padahal udah ada autosave itu sistem gua, darisana gua mesti gilak nyariin data mereka, ngegabungin setiap puzzle, satu persatu ya pake ai si, tapi anj preassurenya tekanannya, soalnya itu besoknya ada ujian kedua gabungan juga, mampus kata gua mah, singkat cerita bisa dan habis tu gua langsung gilak, beban production real user sebrutall ini kah anj
