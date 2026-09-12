# Quick Notes

A tiny full-stack demo:

- **Frontend**: single static `index.html` page (no build step) with a form to submit a note and a list showing every saved note.
- **Database**: Supabase Postgres table `notes` (`id`, `content`, `created_at`), with row-level security allowing anonymous insert + select via the public anon key.
- **Hosting**: deployed as a static site on Vercel.

## How it works

The page loads the `@supabase/supabase-js` client from a CDN and talks directly to Supabase's REST API using the project's public anon key (safe to expose client-side by design — access is governed by the table's Row Level Security policies, not by keeping the key secret).

- Submitting the form inserts a row into `public.notes`.
- On load (and after every insert), the page re-fetches the latest 100 notes ordered by newest first.

## Local development

Just open `index.html` in a browser, or serve the folder with any static file server:

```bash
npx serve .
```

## Deploying

This is a zero-config static site — Vercel deploys it with no build command needed.

## Database schema

```sql
create table public.notes (
  id uuid primary key default gen_random_uuid(),
  content text not null check (char_length(content) between 1 and 500),
  created_at timestamptz not null default now()
);

alter table public.notes enable row level security;

create policy "Anyone can read notes" on public.notes
  for select to anon using (true);

create policy "Anyone can insert notes" on public.notes
  for insert to anon with check (true);
```
