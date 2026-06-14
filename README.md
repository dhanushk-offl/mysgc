# MySGC — Student Guidance Cell Portal

[![Deployed on Cloudflare](https://img.shields.io/badge/Deployed-Cloudflare-F38020?logo=cloudflare&logoColor=white)](https://sgc-cahcet.pages.dev)
[![Contributors](https://contrib.rocks/image?repo=sgc-cahcet/mysgc)](https://github.com/sgc-cahcet/mysgc/graphs/contributors)

A member portal for the **Student Guidance Cell (SGC)** at **CAHCET**. Members can track attendance, view sessions, submit feedback, propose session topics, and administrators can manage the entire lifecycle of sessions.

---

## Table of Contents

- [Tech Stack](#tech-stack)
- [Features](#features)
  - [Member Features](#member-features)
  - [Admin Features](#admin-features)
- [Database Schema](#database-schema)
  - [Tables](#tables)
  - [Row-Level Security](#row-level-security)
- [Supabase Setup](#supabase-setup)
  - [Step 1: Create a Supabase project](#step-1-create-a-supabase-project)
  - [Step 2: Run the schema & RLS script](#step-2-run-the-schema--rls-script)
  - [Step 3: Configure authentication](#step-3-configure-authentication)
  - [Step 4: Set environment variables](#step-4-set-environment-variables)
- [Environment Variables](#environment-variables)
- [Local Development](#local-development)
- [Deployment](#deployment)
  - [Cloudflare Pages](#cloudflare-pages)
- [Project Structure](#project-structure)
- [Data Handling & Security](#data-handling--security)
- [Contributing](#contributing)
- [Code of Conduct](#code-of-conduct)
- [License](#license)
- [Contributors](#contributors)

---

## Tech Stack

| Layer     | Technology                                                    |
| --------- | ------------------------------------------------------------- |
| Framework | [Next.js 15](https://nextjs.org/) (App Router, React 18)     |
| Language  | TypeScript (strict)                                           |
| Styling   | Tailwind CSS 3.4                                              |
| UI        | shadcn/ui (Radix UI primitives)                               |
| Database  | PostgreSQL via [Supabase](https://supabase.com)               |
| Auth      | Supabase Auth (PKCE flow, email/password)                     |
| Hosting   | Cloudflare Pages                                              |
| Icons     | Lucide React                                                  |
| Charts    | Recharts                                                      |
| Dates     | date-fns                                                      |

---

## Features

### Member Features

| Feature              | Description                                                                 |
| -------------------- | --------------------------------------------------------------------------- |
| **Attendance**       | View monthly attendance summary — present days, absent dates, percentage.   |
| **Upcoming Sessions**| See today's and upcoming sessions with handler, co-handler, type & time.    |
| **Session Feedback** | Rate and comment on today's sessions (available 1:30 PM – 11:59 PM IST).    |
| **Propose a Session**| Submit a session interest form with topic, type, preferred date, and description. Optionally select a co-handler. |
| **Session History**  | Browse past sessions with feedback and ratings.                             |
| **Password Mgmt**    | Change password, forgot password flow via email.                            |
| **PWA Support**      | Installable as a Progressive Web App on mobile/desktop.                     |

### Admin Features

Only users with role `President`, `Vice President`, `Administrator`, or `Session Incharge` can access `/admin`.

| Feature                     | Description                                                                        |
| --------------------------- | ---------------------------------------------------------------------------------- |
| **Pending Approvals**       | View, approve, reject, or reschedule session interest requests.                    |
| **Approved Sessions**       | Browse all approved sessions with pagination (10/page) and search (topic, handler, co-handler, date). |
| **View Feedback**           | Open a dialog to see average rating and individual feedback for any session.       |
| **Delete Sessions**         | Permanently remove a session and all associated feedback.                          |
| **Add Session Manually**    | Create a session directly without a prior interest request.                        |
| **Date Conflict Detection** | When approving, checks if the date is already booked by another approved session.  |

---

## Database Schema

### Tables

All tables live under the `public` schema in Supabase (PostgreSQL).

#### `members`
Stores registered SGC members. Pre-seeded by administrators — new users can only sign up if their email exists here.

| Column         | Type      | Notes                              |
| -------------- | --------- | ---------------------------------- |
| `id`           | `integer` | Primary key                        |
| `name`         | `text`    |                                    |
| `email`        | `text`    | Unique, matched against auth email |
| `department`   | `text`    |                                    |
| `role`         | `text`    | One of: `Member`, `Session Incharge`, `Administrator`, `Vice President`, `President` |
| `is_registered`| `boolean` | Set to `true` after first sign-up  |

#### `sessions`
Approved sessions (the "official" schedule).

| Column           | Type      | Notes                                |
| ---------------- | --------- | ------------------------------------ |
| `id`             | `uuid`    | Primary key                          |
| `title`          | `text`    |                                      |
| `date`           | `date`    |                                      |
| `time`           | `time`    | Default `01:00 PM`                   |
| `type`           | `text`    | e.g. Technical, Soft Skills, Career  |
| `handler`        | `text`    | Primary handler name                 |
| `handler_id`     | `integer` | FK to `members.id`                   |
| `handler_count`  | `integer` | 1 or 2                               |
| `co_handler`     | `text`    | Co-handler name (nullable)           |
| `co_handler_id`  | `integer` | FK to `members.id` (nullable)        |
| `description`    | `text`    |                                      |
| `is_approved`    | `boolean` | Always `true` for records here       |

#### `session_interests`
Proposed sessions submitted by members via the interest form.

| Column           | Type      | Notes                                |
| ---------------- | --------- | ------------------------------------ |
| `id`             | `uuid`    | Primary key                          |
| `member_id`      | `integer` | FK to `members.id`                   |
| `member_name`    | `text`    | Denormalized for display             |
| `topic`          | `text`    |                                      |
| `type`           | `text`    |                                      |
| `preferred_date` | `date`    |                                      |
| `description`    | `text`    |                                      |
| `handler_count`  | `integer` | 1 or 2                               |
| `co_handler_id`  | `integer` | FK to `members.id` (nullable)        |
| `co_handler_name`| `text`    | Denormalized (nullable)              |
| `is_approved`    | `boolean` | `false` = pending, `true` = approved |
| `created_at`     | `timestamptz` |                                  |

#### `session_feedback`
Ratings and comments left by members for completed sessions.

| Column       | Type      | Notes                               |
| ------------ | --------- | ----------------------------------- |
| `id`         | `uuid`    | Primary key                         |
| `session_id` | `uuid`    | FK to `sessions.id`                 |
| `member_id`  | `integer` | FK to `members.id`                  |
| `rating`     | `integer` | 1–5                                 |
| `comments`   | `text`    | (nullable)                          |
| `date`       | `date`    | Session date (denormalized)         |
| `created_at` | `timestamptz` |                                 |

#### `attendance`
Daily attendance records per member.

| Column       | Type      | Notes                               |
| ------------ | --------- | ----------------------------------- |
| `id`         | `uuid`    | Primary key                         |
| `member_id`  | `integer` | FK to `members.id`                  |
| `date`       | `date`    |                                     |
| `is_present` | `boolean` |                                     |

#### `feedback` (general)
Generic feedback (separate from session-specific feedback). Used by admins.

#### `notifications`, `push_subscriptions`
Support for push notifications (feature scaffolding).

### Indexes

```sql
attendance_member_date_idx        ON attendance (member_id, date)
attendance_date_idx               ON attendance (date)
sessions_handler_date_idx         ON sessions (handler_id, date desc)
sessions_co_handler_date_idx      ON sessions (co_handler_id, date desc)
sessions_lookup_idx               ON sessions (title, handler, date)
session_feedback_session_created  ON session_feedback (session_id, created_at desc)
session_feedback_member_date      ON session_feedback (member_id, date desc)
session_interests_member_created  ON session_interests (member_id, created_at desc)
-- plus notification and push subscription indexes
```

### Row-Level Security

Every table has RLS enabled. Policies are defined per operation:

| Table               | Select                              | Insert                                    | Update/Delete                           |
| ------------------- | ----------------------------------- | ----------------------------------------- | --------------------------------------- |
| `members`           | Self or admin                       | —                                         | —                                       |
| `attendance`        | Self or admin                       | —                                         | Admin only                              |
| `sessions`          | All authenticated                   | —                                         | Admin only                              |
| `session_interests` | Self or admin                       | Self or admin                             | Admin only (update & delete)            |
| `session_feedback`  | Self, admin, or session handler     | Self or admin                             | Admin delete only                       |
| `feedback`          | Admin only                          | All authenticated                         | Admin update only                       |
| `notifications`     | Self or admin                       | Admin only                                | Self or admin (update)                  |
| `push_subscriptions`| Self or admin                       | Self or admin                             | Self or admin (delete)                  |

Helper functions used in policies:
- `current_member_id()` — returns the `members.id` of the authenticated user based on JWT email
- `current_member_role()` — returns the role of the authenticated user
- `is_admin_member()` — returns `true` if the user has an admin-equivalent role

---

## Supabase Setup

### Step 1: Create a Supabase project

Go to [supabase.com](https://supabase.com) and create a new project. Note your **Project URL** and **anon key** from the API settings page.

### Step 2: Run the schema & RLS script

Open the **SQL Editor** in your Supabase dashboard and run the files in order:

1. **`supabase/queries-and-rls.sql`** — Creates all tables, helper functions, indexes, and RLS policies.
2. **`supabase/two-handler-session-migration.sql`** — Adds co-handler support columns (if the main script is already applied).

> These scripts assume the tables already exist. If starting from scratch, create the tables first (see [Database Schema](#database-schema) for column definitions).

### Step 3: Configure authentication

In the Supabase dashboard under **Authentication > Settings**:

- **Enable email/password sign-up** (if not already enabled).
- Under **Site URL**, set your production URL (e.g., `https://sgc-cahcet.pages.dev`).
- Under **Redirect URLs**, add both:
  - `http://localhost:3000/**` (local dev)
  - `https://sgc-cahcet.pages.dev/**` (production)

### Step 4: Set environment variables

Copy the following values from your Supabase project's **API settings**:

```
NEXT_PUBLIC_SUPABASE_URL=https://<project>.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=<your-anon-key>
```

Create a `.env.local` file in the project root with these values (see [Environment Variables](#environment-variables)).

---

## Environment Variables

Create a `.env.local` file in the project root:

```env
NEXT_PUBLIC_SUPABASE_URL=https://<your-project>.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=<your-supabase-anon-key>
```

These are the only two environment variables needed. They are exposed to the browser (Next.js public prefix `NEXT_PUBLIC_`) because the Supabase client runs on the client side.

---

## Local Development

```bash
# 1. Install dependencies
npm install

# 2. Create environment file
cp .env.local.example .env.local
# Then fill in your Supabase credentials

# 3. Run the dev server
npm run dev

# 4. Open http://localhost:3000
```

**Available scripts:**

| Command           | Description                  |
| ----------------- | ---------------------------- |
| `npm run dev`     | Start Next.js dev server     |
| `npm run build`   | Production build             |
| `npm run start`   | Start production server      |
| `npm run lint`    | Run Next.js lint             |

---

## Deployment

### Cloudflare Pages

The app is deployed on **Cloudflare Pages**. The build is triggered automatically from the `main` branch of the upstream repository.

| Setting           | Value                       |
| ----------------- | --------------------------- |
| Build command     | `npm run build`             |
| Build output dir  | `.next`                     |
| Node.js version   | 20+                         |
| Environment vars  | `NEXT_PUBLIC_SUPABASE_URL`, `NEXT_PUBLIC_SUPABASE_ANON_KEY` |

> **Note:** The `next.config.mjs` has `outputFileTracingRoot` set to support Cloudflare's build environment. Image optimization is disabled (`images.unoptimized: true`) since Cloudflare Pages uses its own image optimization.

---

## Project Structure

```
mysgc/
├── app/
│   ├── admin/
│   │   └── page.tsx              # Admin dashboard (pending + approved sessions)
│   ├── dashboard/
│   │   ├── page.tsx              # Member dashboard
│   │   └── session-history/
│   │       └── page.tsx          # Past sessions with feedback
│   ├── login/
│   │   └── page.tsx              # Login / sign-up page
│   ├── change-password/
│   │   └── page.tsx              # Change password
│   ├── layout.tsx                # Root layout (SessionGuard, Toaster)
│   └── page.tsx                  # Landing page
├── components/
│   ├── admin/
│   │   ├── session-interest-card.tsx  # Card for pending/approved session
│   │   ├── feedback-display.tsx       # Feedback dialog content
│   │   └── status-message.tsx         # Flash message component
│   ├── auth/
│   │   ├── SessionGuard.tsx           # Route protection wrapper
│   │   └── ForgotPasswordDialog.tsx   # Forgot password flow
│   ├── ui/                            # shadcn/ui primitives
│   │   ├── button.tsx
│   │   ├── pagination.tsx
│   │   ├── dialog.tsx
│   │   └── ... (30+ components)
│   ├── attendance-card.tsx            # Monthly attendance widget
│   ├── sessions-card.tsx              # Today + upcoming sessions
│   ├── feedback-form.tsx              # Session feedback form
│   ├── session-interest-form.tsx      # Propose a session form
│   ├── dashboard-header.tsx           # Top navigation bar
│   └── remove-service-worker.tsx      # Cleanup old SW registrations
├── lib/
│   ├── session-types.ts               # TypeScript interfaces
│   ├── date-utils.ts                  # Date formatting (IST timezone)
│   ├── auth-errors.ts                 # Supabase auth error messages
│   ├── utils.ts                       # Tailwind class merging (cn)
│   └── supabase.ts                    # (reserved for server client)
├── hooks/
│   └── use-toast.ts                   # Toast notification hook
├── supabase/
│   ├── queries-and-rls.sql            # Main schema + RLS setup
│   └── two-handler-session-migration.sql # Co-handler migration
├── public/
│   └── logo.png
├── next.config.mjs
├── tailwind.config.ts
└── package.json
```

---

## Data Handling & Security

### Authentication flow

1. User visits `/login` and enters their email.
2. The app calls `check_member_registration_status(email)` via Supabase RPC to verify the email exists in the `members` table.
3. If the email is not pre-registered, access is denied.
4. If the email exists but the member hasn't signed up yet, they can create a password.
5. On sign-up, the `members.is_registered` flag is set to `true`.
6. All subsequent requests are authenticated via Supabase Auth JWT (PKCE flow).

### Authorization

- **Row-Level Security (RLS)** on every table ensures members can only access their own data.
- **Admin roles** (`President`, `Vice President`, `Administrator`, `Session Incharge`) are checked via `is_admin_member()` in RLS policies.
- The admin dashboard enforces a secondary role check on the client side (routes to `/dashboard` if unauthorized).

### Client-side Supabase client

All components use `createBrowserClient()` from `@supabase/auth-helpers-nextjs` (v0.15+) with explicit `NEXT_PUBLIC_SUPABASE_URL` and `NEXT_PUBLIC_SUPABASE_ANON_KEY` passed as arguments. The singleton pattern is used to avoid creating multiple client instances.

### Data denormalization

Some fields are denormalized for performance:
- `session_interests.member_name` — copied from `members.name` at insert time (avoids joins on list views).
- `session_feedback.date` — copied from `sessions.date` at insert time.

---

## Contributing

We welcome contributions! Please follow these steps:

1. Fork the repository.
2. Create a feature branch (`git checkout -b feat/my-feature`).
3. Commit your changes (`git commit -m "feat: add my feature"`).
4. Push to the branch (`git push origin feat/my-feature`).
5. Open a Pull Request against `sgc-cahcet/main`.

**Guidelines:**

- Follow the existing code style (shadcn/ui conventions, Tailwind utility classes).
- Use TypeScript — avoid `any` where possible.
- Keep components focused and small.
- Test locally with `npm run dev` before submitting.
- For Supabase schema changes, add a migration script to `supabase/`.

---

## Code of Conduct

### Our Pledge

We are committed to providing a welcoming, inclusive, and harassment-free experience for everyone in the SGC community.

### Our Standards

- Be respectful and considerate of differing viewpoints.
- Use inclusive language.
- Accept constructive criticism gracefully.
- Focus on what is best for the community.

### Enforcement

Violations of this code of conduct may be reported to the SGC administration. All reports will be reviewed and handled appropriately.

---

## License

This project is licensed under the [MIT License](LICENSE).

```
MIT License

Copyright (c) 2025 Student Guidance Cell

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

---

## Contributors

[![Contributors](https://contrib.rocks/image?repo=sgc-cahcet/mysgc)](https://github.com/sgc-cahcet/mysgc/graphs/contributors)

- **Dhanush** — Core development, admin dashboard, session management, pagination, search, Supabase integration.
- Built for and maintained by **Student Guidance Cell, CAHCET**.

---

<p align="center">
  <sub>Built with ❤️ by the Student Guidance Cell — CAHCET</sub>
</p>
