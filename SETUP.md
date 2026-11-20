# Umbrella Collectibles Inventory — starter build

## What works in this build
- Installable PWA shell (after HTTPS hosting)
- Local inventory database using IndexedDB
- UPC camera scanning where the browser supports `BarcodeDetector`
- Existing UPC scans increment quantity
- Manual product entry/editing, brand/category/variant/condition/location/cost/price/notes
- Search, inventory totals, CSV export and import
- Optional Supabase email sign-in and cloud sync code

## Important current limitations
1. This is a starter build, not yet a published app. It must be hosted on an HTTPS address before Android installation and camera scanning will work.
2. Camera scanning depends on Chrome's `BarcodeDetector` support. Manual UPC entry is the fallback.
3. UPC product identification is not connected yet. The form's “Look up UPC” button is deliberately honest and reports that. A product-data provider/API must be selected and configured; Funko/Thrilljoy variant details may still need manual verification.
4. Cloud sync needs a Supabase project and the SQL below. The cloud sync currently uploads local records and then downloads newer records. For production multi-device conflict handling, use a more robust merge strategy and test it before relying on it.
5. Back up with CSV regularly. Clearing browser data can remove local records.

## Deploy
Upload `index.html`, `app.js`, `manifest.webmanifest`, `sw.js`, and `icon.svg` to a static HTTPS host (for example, a free static-site host). Do not open `index.html` as a local file; PWA install and camera features require HTTPS or localhost.

## Supabase setup (optional cloud sync)
1. Create a Supabase project and confirm email authentication settings.
2. In SQL Editor, run:

```sql
create table if not exists public.inventory_items (
  id text primary key,
  owner uuid not null references auth.users(id) on delete cascade,
  data jsonb not null,
  updated_at timestamptz not null default now()
);

alter table public.inventory_items enable row level security;

create policy "Users can read their own inventory"
on public.inventory_items for select
using (auth.uid() = owner);

create policy "Users can insert their own inventory"
on public.inventory_items for insert
with check (auth.uid() = owner);

create policy "Users can update their own inventory"
on public.inventory_items for update
using (auth.uid() = owner)
with check (auth.uid() = owner);
```

3. In the app Settings, enter the Supabase Project URL and public anon/publishable key, then create an account or sign in.
4. Never put a Supabase service-role key in this app. Only use the public anon/publishable key with Row Level Security enabled.

## Android install
After deployment, open the HTTPS site in Chrome on Android → menu (⋮) → Install app / Add to Home screen.

## Before real inventory use
Test on a small sample first: scan a barcode, edit the record, scan it again, export CSV, import CSV, sign in, sync, and test on a second device. Confirm all rows are present before loading your full inventory.
