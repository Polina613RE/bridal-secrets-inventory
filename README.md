# Bridal Secrets Inventory Dashboard

A shared, web-based inventory tracker for Bridal Secrets (Cedarhurst + Lakewood),
so Rochella's team can enter counts and cost prices directly, and River Edge
gets a clean live view of completion status and inventory value.

It's a single file (`index.html`) hosted free on GitHub Pages, backed by a
free Supabase database — no Google account needed, sign up with your existing
GitHub account or any work email.

## One-time setup (about 15 minutes)

### 1. Create a Supabase account and project
1. Go to https://supabase.com and click **Start your project**.
2. Sign up using **Continue with GitHub** (the same GitHub account you used
   for the 1st Alliance project) — no new password to remember. If you'd
   rather not use GitHub, you can sign up with any email address instead.
3. Click **New project**.
4. Name it `bridal-secrets-inventory`.
5. Set a database password (write it down somewhere safe — you likely won't
   need it again, but keep it just in case).
6. Pick any region close to you, click **Create new project**. It takes
   about a minute to spin up.

### 2. Turn on anonymous sign-in
This lets Rochella's team use the app without creating individual logins,
the same way the Google Sheet works today.
1. In the left sidebar, click **Authentication**.
2. Click **Providers**.
3. Find **Anonymous Sign-Ins** in the list and toggle it **on**.

### 3. Create the two data tables
1. In the left sidebar, click **SQL Editor**.
2. Click **New query**.
3. Paste this in and click **Run**:
   ```sql
   create table inventory_items (
     id uuid primary key default gen_random_uuid(),
     location text,
     item_number text,
     description text,
     bust text,
     waist text,
     height text,
     retail_price numeric,
     cost_price numeric,
     status text default 'active',
     counted boolean default false,
     created_at timestamptz default now()
   );

   create table transactions (
     id uuid primary key default gen_random_uuid(),
     date date,
     trx_number text,
     contact text,
     associate text,
     item_number text,
     type text,
     amount numeric,
     fee numeric,
     net numeric,
     created_at timestamptz default now()
   );

   alter table inventory_items enable row level security;
   alter table transactions enable row level security;

   create policy "Allow signed-in users" on inventory_items
     for all using (auth.uid() is not null) with check (auth.uid() is not null);
   create policy "Allow signed-in users" on transactions
     for all using (auth.uid() is not null) with check (auth.uid() is not null);
   ```
   This creates the two tables and locks them so only people using the app
   (who get signed in automatically and anonymously) can read or write —
   random visitors can't touch the data.

### 4. Get your API keys
1. In the left sidebar, click the gear icon > **Project Settings**.
2. Click **API** in the settings menu.
3. You'll see a **Project URL** and a **Project API keys** section with an
   **anon / public** key. Copy both.

### 5. Paste the keys into index.html
Open `index.html` in this repo, find this block near the top of the
`<script>` section:

```js
const SUPABASE_URL = "REPLACE_ME";
const SUPABASE_ANON_KEY = "REPLACE_ME";
```

Replace the two `"REPLACE_ME"` values with your Project URL and anon key.
Save the file.

### 6. Put it on GitHub Pages
1. Create a new GitHub repo (e.g. `bridal-secrets-inventory`).
2. Upload `index.html` (and this `README.md`) via the GitHub web UI
   (Add file > Upload files).
3. Go to the repo's **Settings > Pages**, set **Source** to the `main`
   branch, root folder, and save.
4. GitHub will give you a URL like
   `https://<your-github-username>.github.io/bridal-secrets-inventory/`.
   That's the link to send Rochella.

No server, no hosting bill, nothing else to maintain.

## What Rochella's team can do
- **Cedarhurst / Lakewood tabs** — add a row per gown: item #, description,
  measurements, retail price, cost price, active/dead stock, and a checkbox
  once physically counted.
- **Rentals & sales tab** — log each rental or sale (date, transaction #,
  contact, associate, amount, fee) — net calculates automatically.

## What you see on the Dashboard tab
- Total gowns tracked, how many are missing a cost price, and estimated
  inventory value at both retail and cost.
- Completion percentage per location.
- A flagged list of every counted item still missing a cost price — your
  live follow-up list instead of combing through a spreadsheet.

## Notes / next steps to discuss
- Right now anyone with the link can enter data (no individual logins) —
  matches how the Google Sheet works today.
- The "Rentals & sales" log is manual entry for now. If you want it to pull
  directly from Bridal Live exports, that's a good follow-on (an import
  button reading a CSV) — let me know if that's worth building next.
- Once populated, "current inventory value" for the balance sheet becomes a
  read of the Dashboard tab instead of a manual reconciliation.
