# MySGC — Student Guidance Cell Portal

[![Deployed on Cloudflare](https://img.shields.io/badge/Deployed-Cloudflare-F38020?logo=cloudflare&logoColor=white)](https://sgc-cahcet.pages.dev)

A member portal for the **Student Guidance Cell (SGC)** at **CAHCET**. Members can track attendance, view sessions, submit feedback, propose topics, and admins can manage the full session lifecycle.

---

## Tech Stack

| Layer     | Technology                                                    |
| --------- | ------------------------------------------------------------- |
| Framework | [Next.js 15](https://nextjs.org/) (App Router, React 18)     |
| Language  | TypeScript (strict)                                           |
| Styling   | Tailwind CSS 3.4 + [shadcn/ui](https://ui.shadcn.com/)       |
| Database  | PostgreSQL via [Supabase](https://supabase.com)               |
| Auth      | Supabase Auth (PKCE flow, email/password)                     |
| Hosting   | [Cloudflare Pages](https://pages.cloudflare.com/)             |

---

## Features

### Member

- **Attendance** — Monthly summary with present/absent tracking and percentages.
- **Sessions** — View today's and upcoming sessions with handler & co-handler.
- **Feedback** — Rate and comment on today's sessions (1:30 PM – 11:59 PM IST).
- **Propose a Session** — Submit session interest with topic, type, date, and optional co-handler.
- **Session History** — Browse past sessions with feedback and ratings.
- **Password Management** — Change password and forgot password flow.
- **PWA** — Installable as a Progressive Web App.

### Admin

Role required: `President`, `Vice President`, `Administrator`, or `Session Incharge`.

- **Pending Approvals** — View, approve, reject, or reschedule session requests.
- **Approved Sessions** — Paginated list (10/page) with search by topic, handler, or date.
- **Feedback** — View average rating and individual comments per session.
- **Manage Sessions** — Delete sessions or add them manually.
- **Conflict Detection** — Prevents double-booking dates.

---

## Documentation

| Topic                           | Link                              |
| ------------------------------- | --------------------------------- |
| Setup & configuration           | [docs/SETUP.md](./docs/SETUP.md)                |
| Architecture & database schema  | [docs/ARCHITECTURE.md](./docs/ARCHITECTURE.md)  |
| Cloudflare deployment           | [docs/DEPLOYMENT.md](./docs/DEPLOYMENT.md)      |
| Contributing guide              | [CONTRIBUTING.md](./CONTRIBUTING.md)            |
| Code of conduct                 | [CODE_OF_CONDUCT.md](./CODE_OF_CONDUCT.md)      |

---

## Quick Start

```bash
npm install
cp .env.example .env.local   # fill in your Supabase credentials
npm run dev                   # → http://localhost:3000
```

See [docs/SETUP.md](./docs/SETUP.md) for full Supabase setup and configuration details.

---

## Meet the Contributors

[![Contributors](https://contrib.rocks/image?repo=sgc-cahcet/mysgc)](https://github.com/sgc-cahcet/mysgc/graphs/contributors)

Built and maintained by **Student Guidance Cell — CAHCET**.

---

## License

[MIT](./LICENSE)
