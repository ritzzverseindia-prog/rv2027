# Ritzzverse India — Office ERP (HTML edition)

**No React, Vite, npm, or build step.** Static HTML + CSS + JavaScript (Supabase client loaded as an ES module from jsDelivr). Suitable for GitHub → Vercel static deployment. The prior Vite `/src/main.jsx` build failure is eliminated because `index.html` now loads `app.js` directly. This does **not** mean the database is automatically configured.

## 1. Replace your GitHub files

Best: create a **new private GitHub repository** for this HTML version to avoid stale `package.json`, `src/`, and Vite settings. Upload **the contents of this extracted folder** (not the ZIP and not a nested folder). Required root files: `index.html`, `app.js`, `style.css`, `config.js`, `vercel.json`, `README.md`, and `supabase/schema.sql`.

To reuse the existing repo: delete the old `package.json`, `package-lock.json`, `vite.config.js`, and `src/` directory; upload the files above to the repository root. Make sure `index.html` contains `<script type="module" src="app.js"></script>` and **no** `/src/main.jsx` reference.

## 2. Configure Supabase

1. Create a Supabase project at https://supabase.com/dashboard.
2. In SQL Editor, run `supabase/schema.sql` **once on a fresh project**. If you previously ran it, do not run it again.
3. In Authentication settings, disable public self-service signups. Keep Email authentication enabled. Invite users from Authentication → Users. The app has no signup screen.
4. In Authentication → Users, copy each invited user's UUID. In SQL Editor, create a profile for every user (including your admin):

```sql
insert into public.profiles (id,full_name,email,role,annual_leave_allowance)
values ('AUTH_USER_UUID','Admin Name','admin@yourcompany.com','admin',18);
```

For employees, use role `employee`; for supervisors, `manager`. Profiles are manually provisioned. Restrict invited emails to your company domain in your onboarding process. **Email/password sign-in alone does not enforce a domain**; strict enforcement requires server-side auth rules/SSO.

5. Open `config.js` and replace the placeholders with your Supabase Project URL and **public publishable/anon key** (Project Settings → API Keys). These values are intentionally visible in the browser. **NEVER paste a service_role or secret key.**

## 3. Deploy to Vercel

1. Push the files to GitHub.
2. In Vercel, import the repository.
3. Choose **Framework Preset: Other**.
4. Set **Root Directory: `./`**.
5. **Override Build Command: enabled, leave the command empty** (or choose no build command). **Output Directory: `.`** (repository root), if Vercel asks. For a new static project, Vercel generally detects `index.html` automatically.
6. Deploy. Changes to `config.js` require a new Git commit/deployment.

If Vercel still says `vite build` or tries `/src/main.jsx`, the old files or project build settings are still in use. Clear the build-command override and confirm the new `index.html` is on the connected branch.

## Features

- Supabase email/password login and logout, manager/admin/employee roles
- Assigned tasks, priorities, deadlines, status updates, daily schedule
- Attendance clock-in/out, leave requests and estimated leave balances
- Client projects, poster/video/file submission (50 MB limit), manager review and revisions
- Manager team directory (onboarding done in Supabase)

## Security and limits

- This is a functional starter, **not production-audited**. Test with separate employee and manager accounts before using real staff data.
- The provided schema enables row-level security and a private Storage bucket. Do not remove RLS policies.
- Video uploads are limited to 50 MB; use resumable uploads or external storage for large footage.
- Leave counter counts calendar days, not holidays/workdays. The estimated yearly balance counts requests starting in the current year; cross-year requests require more careful logic.
- No employee performance scoring, payroll, notification engine, automatic content posting, client portal, or Google Workspace SSO.
- The JavaScript Supabase library is loaded from a CDN, so the user's browser needs internet access to that CDN.
- The app's `config.js` is public. The publishable key is safe to expose *only when RLS is correctly configured*.
- Attendance SQL uses database `current_date`; test behavior around midnight in your local timezone.
