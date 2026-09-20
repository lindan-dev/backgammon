# Backgammon — Daniel & Lina (self-hosted)

A single-page backgammon app for the two of you, synced live through your
own Supabase project. No server code — just `index.html`, served as a
static file (GitHub Pages, Netlify, Vercel, anywhere).

## 1. Create the table in Supabase

In your Supabase project, open the **SQL Editor** and run:

```sql
create table if not exists bg_matches (
  id text primary key,
  state jsonb not null,
  updated_at timestamptz not null default now()
);

alter table bg_matches enable row level security;

-- Just the two of you use this table for one shared game row, so a
-- permissive policy is fine here. Anyone with your anon key AND your
-- project URL could read/write this table — keep the repo private if
-- that matters to you, or tighten these policies later (e.g. require
-- a shared secret column) if you want extra safety.
create policy "allow all reads" on bg_matches
  for select using (true);
create policy "allow all writes" on bg_matches
  for insert with check (true);
create policy "allow all updates" on bg_matches
  for update using (true);
```

## 2. Turn on Realtime for the table

Dashboard → **Database → Replication** → find `bg_matches` → toggle it on.

(Or via SQL: `alter publication supabase_realtime add table bg_matches;`)

## 3. Grab your project keys

Dashboard → **Project Settings → API**:
- **Project URL** — looks like `https://xxxxxxxx.supabase.co`
- **anon / public key** — a long JWT-looking string

This anon key is *meant* to be public/client-side — it's not a secret,
your RLS policies above are what actually controls access.

## 4. Fill in `index.html`

Near the top of the `<script>` block:

```js
const SUPABASE_URL = "https://xxxxxxxx.supabase.co";
const SUPABASE_ANON_KEY = "eyJ...your anon key...";
```

## 5. Put it online

Simplest option — GitHub Pages:

1. Create a repo, add `index.html` to it.
2. Push it.
3. Repo → Settings → Pages → Deploy from branch → pick `main` / root.
4. Your game is live at `https://<you>.github.io/<repo>/`.

Any other static host (Netlify, Vercel, Cloudflare Pages, a plain
`python -m http.server` on your own box) works exactly the same way —
it's just one HTML file.

## Playing

- Open the link on your own device, tap **I'm Daniel** or **I'm Lina**
  (remembered locally after that, per device).
- First person to open it sets the week's challenge and best-of.
- Roll, tap a checker then a highlighted destination to move, bear off
  once all your checkers are home. Score and match winner are tracked
  automatically. The gear icon lets you edit the challenge/best-of or
  start a fresh match anytime.

## Notes

- Everything lives in one Supabase row (`id = 'current'`), so this is
  built for exactly one match at a time between the two of you — not a
  general multi-user app.
- If you ever want to reset from scratch, just delete the row in the
  Supabase table editor and reload the page.
