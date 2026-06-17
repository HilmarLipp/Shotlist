# Shotlist

Single-file HTML shot-list app for film/video production. No build step — just
`index.html`. Deployed via GitHub Pages: <https://hilmarlipp.github.io/Shotlist/>

## Features

- Multiple shotlists, each with its own shareable URL (`?id=<slug>`)
- Project picker (create, open, delete with automatic backup)
- 14-column shot table: shot #, INT/EXT, location, size, angle, audio, subject,
  description, dialogue, 9:16 reference image, VFX, done
- Drag-to-reorder (mouse + touch), per-shot reference image upload (drag/drop or click)
- Sort (script / location / status) + location filter, multiple sequences
- Landscape A4 PDF export, JSON import/export
- Link sharing across devices via Supabase (optional — see below)

## Storage layout

State lives in `localStorage`:

- `shotlist_v3:index` → `[{id, title, updatedAt, shots}]`
- `shotlist_v3:project:<id>` → the full state object for one project
- `shotlist_v3:backup:<id>:<timestamp>` → pre-delete backup, without image blobs (max 2 per project)

On first load the app migrates a legacy `shotlist_v2` single-state into a v3 project
automatically. The old `shotlist_v2` key is left in place as a safety net.

## Sharing (Supabase setup)

Sharing is **optional**. Without it the app works fully offline; the "Share" button
just explains it isn't configured. To enable cross-device share links:

1. Create a free project at <https://supabase.com>.
2. In the SQL editor, run:

   ```sql
   create table if not exists public.shotlists (
     id text primary key,
     data jsonb not null,
     updated_at timestamptz not null default now()
   );

   alter table public.shotlists enable row level security;

   -- Public app with unguessable slugs: allow anonymous read + upsert.
   create policy "public read"   on public.shotlists for select using (true);
   create policy "public insert" on public.shotlists for insert with check (true);
   create policy "public update" on public.shotlists for update using (true);
   ```

3. In **Project Settings → API**, copy the **Project URL** and the **anon public** key.
4. In `index.html`, fill the config block near the top of the `<script>`:

   ```js
   var SUPABASE_URL = 'https://YOURPROJECT.supabase.co';
   var SUPABASE_ANON_KEY = 'eyJ...';   // anon public key
   ```

The anon key is a public client key and is safe to ship in a static page **because**
the table has Row Level Security enabled with the policies above.

### How sharing works

- **Share**: uploads the current project (by its slug) to Supabase and gives you a
  `?id=<slug>` link to copy. Edits after sharing are **not** auto-synced — click
  Share again to push an updated snapshot.
- **Receiving**: opening a `?id=<slug>` link that isn't on your device fetches the
  shared copy from Supabase and saves it locally after you confirm. From then on it's
  a normal local project; your edits stay on your device.

## Local development

Serve the folder with any static server, e.g.:

```bash
python3 -m http.server 8000
```

then open <http://localhost:8000/>.
