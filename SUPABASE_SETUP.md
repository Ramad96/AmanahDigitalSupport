# Supabase — AmanahDigital Support

Database backend for `support.amanahdigital.co.uk`.

---

## 1. Create a Supabase project

1. Go to [supabase.com](https://supabase.com) and sign in.
2. Click **New project**.
3. Give it a name (e.g. `amanahdigital-support`), choose a region close to your users, set a database password, then click **Create new project**.
4. Wait ~2 minutes for the project to finish provisioning.

---

## 2. Create the schema and table

1. In the left sidebar click **SQL Editor**, then **New query**.
2. Paste the entire block below and click **Run**:

```sql
-- 1. Create a separate schema to keep support data isolated
create schema if not exists support;

-- 2. Create the submissions table
create table support.support_feedback (
  id          uuid        primary key default gen_random_uuid(),
  created_at  timestamptz not null default now(),
  type        text        not null check (type in ('feedback', 'feature_request', 'bug_report')),
  product     text        not null,
  email       text        not null,
  subject     text        not null,
  message     text        not null
);

-- 3. Enable Row Level Security (required for anon key inserts to work)
alter table support.support_feedback enable row level security;

-- 4. Allow anyone with the anon key to INSERT (the website form)
create policy "allow anon insert"
  on support.support_feedback
  for insert
  to anon
  with check (true);

-- 5. Only authenticated users (you, in the dashboard) can SELECT
create policy "allow authenticated select"
  on support.support_feedback
  for select
  to authenticated
  using (true);
```

3. You should see "Success. No rows returned." — the table is ready.

---

## 3. Get your project URL and anon key

1. In the left sidebar go to **Project Settings → API**.
2. Copy the **Project URL** (looks like `https://xxxxxxxxxxxx.supabase.co`).
3. Copy the **anon / public** key under "Project API keys".

---

## 4. Connect the website

Open `index.html` and find these two lines near the bottom of the `<script>` block:

```js
const SUPABASE_URL  = 'YOUR_PROJECT_URL';
const SUPABASE_ANON = 'YOUR_ANON_KEY';
```

Replace the values with what you copied in Step 3. The section should look like:

```js
const SUPABASE_URL  = 'https://xxxxxxxxxxxx.supabase.co';
const SUPABASE_ANON = 'eyJhbGci...';
```

Save the file and commit/push to GitHub — the site will automatically redeploy via GitHub Pages.

---

## 5. Verify it works

Open [https://support.amanahdigital.co.uk](https://support.amanahdigital.co.uk), fill in the form, and submit. Then in Supabase go to **Table Editor → support schema → support_feedback** — your test submission should appear.

---

## Viewing submissions

In the Supabase dashboard, go to **Table Editor**, select the **support** schema from the schema dropdown, then open **support_feedback**.

### Useful queries

Run these in the **SQL Editor**:

```sql
-- Most recent 50 submissions
select created_at, type, product, email, subject, message
from support.support_feedback
order by created_at desc
limit 50;

-- Filter by product
select * from support.support_feedback
where product = 'Noor: A Better Ramadan';

-- Count by submission type
select type, count(*)
from support.support_feedback
group by type;

-- Count by product
select product, count(*)
from support.support_feedback
group by product
order by count desc;
```

---

## Table structure

| Column       | Type        | Notes                                          |
|--------------|-------------|------------------------------------------------|
| `id`         | uuid        | Auto-generated primary key                     |
| `created_at` | timestamptz | Set automatically on insert                    |
| `type`       | text        | `feedback`, `feature_request`, or `bug_report` |
| `product`    | text        | Product name or `General`                      |
| `email`      | text        | Submitter's email address                      |
| `subject`    | text        | Brief summary from the form                    |
| `message`    | text        | Full details from the form                     |

---

## Email notifications (optional)

To receive an email on every new submission, set up a Database Webhook:

1. Go to **Project Settings → Database → Webhooks**.
2. Create a new webhook on the `support.support_feedback` table, trigger: **INSERT**.
3. Point it at a service such as [Resend](https://resend.com), [Postmark](https://postmarkapp.com), or [Zapier](https://zapier.com).
