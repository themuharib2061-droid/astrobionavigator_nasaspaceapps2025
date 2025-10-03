# AstroBio Navigator (Flutter)

Discover, filter, and **save space/astrobiology research papers** with a clean Flutter app. Built for NASA Space Apps 2025 project work, this frontend talks to a simple Node/Express backend (deployed on Vercel in our setup) and uses **Supabase** for auth/profile. It runs on **Android/iOS/Web**.

> **TL;DR**  
> - Search papers by keyword, see curated trending results by category/keywords, open PDFs, and generate AI summaries.  
> - Save favorites locally (persistent) and manage your profile (name, country, avatar, preferred categories).  
> - Backend endpoints (search/trending/summarize) are configurable; Supabase URL/keys must be set.  
> - Works on Flutter 3.x, Dart 3.x.

---

## ✨ Features

- **Explore**: search papers (`/api/search-papers`) and show **trending by keywords** (`/api/trending-papers-by-keywords`).  
- **Study Cards**: title, authors, year, quick **Open** (prefers direct PDF), **Summarize** (AI summary modal), and **Save** (heart).  
- **Summarize**: bottom-sheet chat using `POST /api/summarize` with the paper URL.  
- **Saved Articles**: persistent favorites (SharedPreferences).  
- **Auth (Supabase)**: email/password signup (2-step), login, logout, profile info + avatar upload.  
- **Profile**: display name, country, **preferred categories (up to 3)**, and avatar.  
- **Dark UI** with a consistent theme.  
- **Flutter Web** support (with appropriate CORS on backend).

---

## 🧱 Tech Stack

- **Flutter 3.x** / **Dart 3.x**
- **Supabase** (`supabase_flutter`) for auth + `profiles` table
- **HTTP** (`http`) for backend APIs
- **SharedPreferences** for client-side saved articles
- **url_launcher** for opening paper/PDF links

> The repo already contains the standard `web/` folder for Flutter Web builds.

---

## 📁 Project Structure (high level)

```
lib/
  main.dart
  models/
    study_model.dart
    study_compat.dart
  screens/
    explore_page.dart
    saved_articles_page.dart
    profile_page.dart
    edit_profile_page.dart
    login_page.dart
    signup_step1_page.dart
    signup_step2_page.dart
    main_screen.dart
  services/
    auth_service.dart
    supabase_service.dart
    paper_service.dart        # calls /search-papers and /trending-papers-by-keywords
    summary_chat_api.dart     # calls /summarize
    favorites_service.dart    # local saved items
  theme/
    app_colors.dart
  widgets/
    study_card.dart
    category_picker.dart
    app_bars.dart
    summary_chat_sheet.dart
web/
  index.html, manifest.json, icons/...
```

---

## 🔧 Configuration

### 1) Supabase (required)
Edit **`lib/main.dart`** and replace:
```dart
const supabaseUrl = 'https://YOUR-PROJECT.supabase.co';
const supabaseAnonKey = 'YOUR-ANON-KEY';
```
These are used by `supabase_flutter` during `main()` initialization.  
> Tip: For production, consider reading from `--dart-define`:
```dart
const supabaseUrl = String.fromEnvironment('SUPABASE_URL');
const supabaseAnonKey = String.fromEnvironment('SUPABASE_ANON_KEY');
```
Then run with:
```bash
flutter run --dart-define SUPABASE_URL=https://xxx.supabase.co            --dart-define SUPABASE_ANON_KEY=eyJhbGciOi...
```

### 2) Backend API base URLs
- **Search/Trending**: `lib/services/paper_service.dart`
  ```dart
  static const String baseUrl = 'https://astro-bio-navigator-server.vercel.app/api';
  ```
- **Summarize**: `lib/services/summary_chat_api.dart`
  ```dart
  static const String baseUrl = 'https://astro-bio-navigator-server.vercel.app/api';
  ```

If you fork or self-host the backend, change these base URLs.

> **CORS note (for Web)**: the backend must include appropriate `Access-Control-Allow-Origin` (e.g., `*` during dev or your web origin in prod).

### 3) Android & iOS
- **Internet permission** is already included via Flutter templates.  
- For **`url_launcher`**, ensure your links are valid `http(s)` URLs. No additional native setup is required for modern Flutter.

---

## 📡 API Contracts (expected by the frontend)

> Exact shapes are enforced in `lib/services/paper_service.dart` and used by `widgets/study_card.dart` & `screens/explore_page.dart`.

### `POST /api/search-papers`
**Body**
```json
{"keyword":"microgravity", "limit": 10}
```
**Response**
```json
{
  "success": true,
  "papers": [
    {
      "title": "Paper title",
      "link": "https://paper-page",
      "pdfLink": "https://direct.pdf",           // optional
      "abstract": "…",
      "authors": ["A", "B"],                     // or comma-separated string
      "publishYear": 2024,
      "publicationInfo": "Nature, 2024"
    }
  ]
}
```

### `POST /api/trending-papers-by-keywords`
**Body**
```json
{"keywords": ["origin of life","microgravity biology"], "limit": 3, "yearFrom": 2020}
```
**Response**
```json
{
  "success": true,
  "results": [ 
    {
      "keyword": "origin of life",
      "papers": [ /* same paper shape as above */ ]
    }
  ]
}
```

### `POST /api/summarize`
**Body**
```json
{"url":"https://paper-or-pdf-link"}
```
**Response**
```json
{ "success": true, "summary": "Short AI summary text…" }
```

---

## ▶️ Run Locally

### Prerequisites
- Flutter SDK 3.x installed (`flutter doctor` clean)
- Android Studio or VS Code (optional) for devices/emulators
- A Supabase project + anon key
- A running backend (or the provided Vercel URL)

### Install & run
```bash
git clone https://github.com/yourname/astrobio-navigator-frontend.git
cd astrobio-navigator-frontend

flutter pub get

# (Option A) Hardcode keys in lib/main.dart then:
flutter run -d chrome     # Web
# or
flutter run               # Android/iOS if a device/emulator is connected

# (Option B) Use dart-define
flutter run -d chrome   --dart-define SUPABASE_URL=https://xxx.supabase.co   --dart-define SUPABASE_ANON_KEY=eyJhbGciOi... 
```

### Build
```bash
# Web
flutter build web
# Android (debug apk)
flutter build apk --debug
# iOS
flutter build ios
```

Deploy the `build/web` folder to any static host (Vercel/Netlify/GitHub Pages).  
Remember to set CORS on your backend to your deployed domain.

---

## 👤 Supabase Setup (minimal)

Create a table **`profiles`** with `id` (UUID, PK), `email` (text), `display_name` (text), `avatar_url` (text, nullable), `country` (text, nullable), `preferred_categories` (text[]). The `id` must equal `auth.users.id`.

Example SQL:
```sql
create table if not exists public.profiles (
  id uuid primary key references auth.users(id) on delete cascade,
  email text not null,
  display_name text not null,
  avatar_url text,
  country text,
  preferred_categories text[] default '{}'
);

alter table public.profiles enable row level security;

-- Select/update own profile
create policy "profiles_select_own"
  on public.profiles for select
  using ( auth.uid() = id );

create policy "profiles_update_own"
  on public.profiles for update
  using ( auth.uid() = id );
```

The app uses:
- `AuthService.signUpStep1` → creates auth user and inserts a profile row.  
- `AuthService.setPreferredCategories` → saves top-3 categories.  
- Streaming/reading profile is done via `supabase.from('profiles')` in services.

---

## 🧭 How the UI Flows

- **Login/Signup** → `MainScreen` (tabs for Explore, Saved, Profile).
- **Explore**:
  - Type a keyword and hit **Enter** to call `PaperService.searchPapers`.
  - Each **StudyCard** shows actions:
    - **Open**: `url_launcher` → prefers `pdfLink`, falls back to `link`.
    - **Summarize**: opens `summary_chat_sheet` and calls `SummaryChatApi.summarizeUrl`.
    - **Heart**: toggles saved status via `FavoritesService`.
- **Saved Articles**: reads from SharedPreferences and lists saved papers.  
- **Profile**: edit display name, country, avatar, and **pick up to 3 preferred categories**.

---

## 🛠️ Troubleshooting

- **CORS errors in Web**: configure your backend to return `Access-Control-Allow-Origin` (e.g., `*` for dev).  
- **Nothing opens on “Open”**: ensure the paper has a valid `pdfLink` or `link` and the device has a browser.  
- **Unauthorized with Supabase**: double-check URL/Anon key and that RLS policies exist.  
- **Search returns 0 items**: verify backend is reachable and `keyword` is non-empty; inspect backend logs.  
- **Summarize fails**: make sure the URL is accessible publicly and your summarize endpoint is up.  
- **Build errors about packages**: run `flutter pub get`; ensure your Flutter SDK matches the repo’s `.metadata` channel (`stable`).

---

## 🗺️ Roadmap Ideas (optional)

- Offline cache for last results
- Filters (year, journal), paging/infinite scroll
- Share sheet for papers
- Light theme toggle
- Error reporting (Sentry) and analytics (optional)

---

## 🤝 Contributing

PRs welcome! Please:
1. Create a feature branch
2. Keep commits scoped and descriptive
3. Add/adjust comments where logic is non-obvious

---

## 📄 License

MIT © Your Name
