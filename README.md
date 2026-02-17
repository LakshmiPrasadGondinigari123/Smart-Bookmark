# Smart Bookmark App

A real-time bookmark manager built with Supabase and Vanilla JS, deployed on Vercel.

## 🔗 Live URL
>  https://smart-bookmark-git-devcode-prasads-projects-272591f5.vercel.app

## 📦 GitHub Repo
> https://github.com/LakshmiPrasadGondinigari123/Smart-Bookmark/tree/dev_code

---

## ✅ Features
- **Google OAuth only** — no email/password, sign in with Google
- **Private bookmarks** — each user only sees their own bookmarks (Row Level Security)
- **Real-time updates** — add a bookmark in one tab, it appears instantly in another
- **Add bookmarks** — save any URL with a title
- **Delete bookmarks** — remove any bookmark instantly
- **Deployed on Vercel** — live and publicly accessible

---

## 🛠 Tech Stack
| Layer | Technology |
|---|---|
| Frontend | Vanilla HTML, CSS, JavaScript (single `.html` file) |
| Auth | Supabase Auth (Google OAuth 2.0) |
| Database | Supabase PostgreSQL |
| Realtime | Supabase Realtime (postgres_changes) |
| Styling | Custom CSS (dark theme, responsive) |
| Deployment | Vercel |

---

## 🚧 Problems I Ran Into & How I Solved Them

### 1. Google OAuth redirect not working locally
**Problem:** Clicking "Continue with Google" showed `redirect_uri_mismatch` error.

**Solution:** Added `http://127.0.0.1:5500` and `http://127.0.0.1:5500/smart.html` to both:
- Google Cloud Console → OAuth Client → Authorized Redirect URIs
- Supabase → Authentication → URL Configuration → Redirect URLs

Also added the Supabase callback URL: `https://<project>.supabase.co/auth/v1/callback`

---

### 2. After Google login, app returned to login screen instead of staying logged in
**Problem:** After completing Google OAuth, the app would redirect back but not detect the session, showing the login screen again.

**Solution:** 
- Added `detectSessionInUrl: true` and `persistSession: true` to the Supabase client config
- Used `await sb.auth.getSession()` at app boot to check for an existing session before showing any screen
- Added a splash screen to prevent UI flash while session loads

---

### 3. Client Secret missing from Supabase Google provider
**Problem:** Google OAuth was enabled in Supabase but the Client Secret field was empty, causing login to fail silently.

**Solution:** Generated a new OAuth Client Secret in Google Cloud Console → Credentials → clicked "Add Secret", then pasted it into Supabase → Authentication → Providers → Google → Client Secret field.

---

### 4. Real-time updates not working
**Problem:** Adding a bookmark didn't appear in the other tab automatically.

**Solution:** Used Supabase's `postgres_changes` realtime subscription filtered by `user_id` to listen for INSERT and DELETE events on the `bookmarks` table. Combined with Row Level Security policies, this ensures each user only receives their own events.

---

### 5. Row Level Security blocking all reads/writes
**Problem:** After enabling RLS on the bookmarks table, all database operations returned empty results.

**Solution:** Added three explicit RLS policies:
```sql
-- Allow users to read their own bookmarks
create policy "Users see own bookmarks"
  on bookmarks for select using (auth.uid() = user_id);

-- Allow users to insert their own bookmarks  
create policy "Users insert own bookmarks"
  on bookmarks for insert with check (auth.uid() = user_id);

-- Allow users to delete their own bookmarks
create policy "Users delete own bookmarks"
  on bookmarks for delete using (auth.uid() = user_id);
```

---

## 🗄️ Database Schema

```sql
create table bookmarks (
  id         uuid primary key default gen_random_uuid(),
  user_id    uuid references auth.users not null,
  title      text not null,
  url        text not null,
  created_at timestamptz default now()
);

alter table bookmarks enable row level security;
```

---

## 🚀 How to Run Locally

1. Clone the repo
2. Open `smart-bookmark-app.html` with VS Code Live Server
3. Navigate to `http://127.0.0.1:5500/smart-bookmark-app.html`
4. Click "Continue with Google" to sign in

---

## 📁 Project Structure

```
smart-bookmark-app/
├── smart-bookmark-app.html   # Complete app (HTML + CSS + JS in one file)
└── README.md                 # This file
```

---

## 👨‍💻 Built By
Gondinigari Lakshmi Prasad
