# A Lovely Touch of Clean, LLC — website

A press-release-ready, single-page site for **A Lovely Touch of Clean, LLC**
(Phoenix, AZ). Magazine-style layout, light + dark theme, and a "free estimate"
booking form wired to **Supabase**. Ships as one `index.html` — no build step.

The site works immediately in **demo mode** (the form shows a confirmation but
doesn't save). Add your Supabase keys to make requests save for real.

---

## 1. Files

```
index.html      the whole site (self-contained)
README.md       this file
images/
  hero.png             Dawntaya's Maid Service hero (top of page)
  living-room-01.png   sunlit living room (Why we clean section)
  group.png            the cleaning team (About us section)
  sandy-d.png          Sandy D. review photo
  michelle-m.png       Michelle M. review photo
  denny-w.png          Denny W. review photo
  living-room-02.png   spare living-room shot (not currently placed)
```

Every photo is wired in. To swap any of them, drop a new file in `images/` and
update the matching `<img src="images/...">` in `index.html`. Photos use
`object-fit:cover`, so any reasonable aspect ratio fills its frame cleanly.
`living-room-02.png` is an unused spare — drop it anywhere you'd like a second
interior shot.

**Theme colors:** the light theme is **lavender & white** (soft white background,
royal-purple headings, lavender accents, pale-lavender CTA band). The dark theme
is unchanged from the previous version.

---

## 2. Content baked in (from the old site)

- Business: **A Lovely Touch of Clean, LLC**
- Tagline: *Freshly organized spaces. Lovely touches of clean. Peace of mind.*
- Services: **House Cleaning · Ironing · Office Cleaning · Car Washing**
- Reviews: Sandy D., Michelle M., Denny W.
- Address: **3411 N 16th Street, Phoenix, AZ 85016**
- Phone: **602-816-0202** · Email: **service@lovelytouchcleaning.com**

> Note: business **hours** weren't shown on the old site, so I used sensible
> defaults (Mon–Fri 8–5, Sat 9–2, Sun closed). Edit the `hours` array near the
> bottom of `index.html` to match your real schedule.

---

## 3. Supabase setup (estimate-request backend)

1. Create a project at supabase.com.
2. Open **SQL Editor** and run:

```sql
create table public.bookings (
  id             bigint generated always as identity primary key,
  name           text not null,
  phone          text not null,
  email          text not null,
  service        text not null,
  property_type  text not null,
  preferred_date date,
  notes          text,
  status         text default 'new',
  created_at     timestamptz default now()
);

-- Let the public site INSERT requests, but not read others' data.
alter table public.bookings enable row level security;

create policy "anyone can request an estimate"
  on public.bookings for insert
  to anon
  with check (true);
```

3. In **Project Settings → API**, copy your **Project URL** and **anon public** key.

4. In `index.html`, near the top, replace the two placeholders:

```js
window.SUPABASE_URL = "https://YOURPROJECT.supabase.co";
window.SUPABASE_ANON_KEY = "eyJhbGciOi...your-anon-key...";
```

Requests now land in the `bookings` table — view them in the Supabase **Table
Editor**. The `anon` key is safe in the browser; RLS above only allows inserts,
so visitors can't read or edit other people's requests.

---

## 4. Deploy: GitHub → Vercel

1. **Push to GitHub**
   ```bash
   cd clean
   git init
   git add .
   git commit -m "A Lovely Touch of Clean site"
   git branch -M main
   git remote add origin https://github.com/YOU/lovely-touch-clean.git
   git push -u origin main
   ```

2. **Vercel** → vercel.com → **Add New → Project** → import the repo.
   - Framework preset: **Other** (static file)
   - Build command: *(leave empty)* · Output directory: `.` (root)
   - Deploy. Every push to `main` redeploys.

No environment variables needed — the Supabase URL + anon key live in the HTML.

---

## 5. Editing content

- **Services** — the `#services` section (`.svc` cards).
- **Reviews** — the `#reviews` section (`.quote` blocks).
- **Hours** — the `hours` array in the script near the bottom.
- **Address / phone / email** — the `#visit` section and the footer.
- **Colors** — `:root` (light) and `[data-theme="dark"]` at the top of the CSS.
- **Real photos** — replace the placeholder `.ph` blocks with `<img>` tags.

---

## 6. Theme toggle

- Respects the visitor's OS light/dark setting on first load.
- Remembers their manual choice in `localStorage`.
- Toggle button is top-right in the nav (◐ / ☀).
