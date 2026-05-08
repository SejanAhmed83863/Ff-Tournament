# Jahanabad Tournament Club — Fixed Files

## What was fixed

### 1. Transactions & Tournament history not showing in profile  ✅
**Root cause:** The Firestore queries in `dashboard.html` used `where(...) + orderBy(...)` together, which requires a composite index. Without that index Firebase silently throws an error which was being swallowed by `try/catch`.

**Fix:** Removed the `orderBy` from the compound queries; results are now sorted client-side. Errors are now logged AND shown to the user instead of silently failing.

### 2. UID change — Google verification added  ✅
**Before:** Only email/password re-authentication worked. Users who signed up with Google could not change their UID a second time.

**Now:** When changing UID after the first time, users see a chooser:
- 🔑 **Password** → asks for current password (Firebase `EmailAuthProvider` reauth)
- **Google** → opens Google popup for re-authentication (`reauthenticateWithPopup`)
The default option is auto-selected based on the user's sign-in provider.

### 3. WhatsApp number + Full Name now collected  ✅
- **`index.html` (Register form):** Now requires Full Name + WhatsApp Number along with email/password.
- **Google sign-in:** First-time Google users see a small popup asking for Name + WhatsApp before continuing.
- **`dashboard.html` (Edit Profile):** Both Name and WhatsApp can be edited later.
- **`tournaments.html` (Join modal):** Pre-fills Name and WhatsApp from profile, but lets users override.
- All transactions and registrations now include `name` + `whatsapp` so admin sees them.

### 4. Logo added beside the app name  ✅
- A new `logo.png` is included (resized version of your logo).
- Shown in the **header of every page** (home, dashboard, tournaments) and on the **login screen**.

### 5. Red color of buttons changed  ✅
The aggressive red gradient (`#ec1c24 → #a01015`) was replaced with a premium **dark-gold gradient** (`#b8860b → #6b4f00`) on every action button:
- "Enter Arena" / "Join Tournament" (login & register)
- "Save", "Submit Request", "Withdraw", "Confirm & Join"
- Wallet badge in header

The gold accent border is kept for the eSports premium look.

### 6. Banner removed from `home.html`  ✅
The big "FREE FIRE TOURNAMENT" banner image at the top of the home page is gone. Replaced by:
- A clean welcome row ("Hello, *Name* 👋")
- A 3-up quick-stats row (Live tournaments / Total players / My plays)
- The Live Tournaments list

---

## Files in this folder

| File | Status |
|---|---|
| `index.html` | **Replaced** — login/register with Name + WhatsApp + Google extra-info modal |
| `home.html` | **Replaced** — banner removed, logo added, red → dark-gold |
| `dashboard.html` | **Replaced** — fixed transactions/history queries, Google reauth, name & whatsapp fields |
| `tournaments.html` | **Replaced** — logo added, dark-gold theme, name/whatsapp prefilled & saved |
| `admin.html` | **Unchanged** (you can keep your existing one) |
| `logo.png` | **NEW** — upload this to your repo root |

---

## How to deploy to GitHub Pages

1. Open your repo: <https://github.com/sejanahmed83863/Tournament-App->
2. Replace these files with the ones in this folder:
   - `index.html`
   - `home.html`
   - `dashboard.html`
   - `tournaments.html`
3. **Upload `logo.png`** to the repo root (same folder as the HTML files).
4. Commit & push. GitHub Pages will redeploy in ~1 min.

---

## ⚠ One Firestore step you may want to do

For the legacy tournaments query (`orderBy('time')`), Firebase usually creates the simple index automatically. **But** if you ever see errors loading the tournaments list, open the browser console — Firebase prints a *direct link* to create the missing index. Click it, that's it.

For the **transactions** & **registrations** queries we already removed `orderBy`, so no index is needed there.

---

## Recommended Firestore Security Rules

In Firebase Console → Firestore Database → Rules, paste:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /users/{userId} {
      allow read: if request.auth != null;
      allow write: if request.auth != null && request.auth.uid == userId;
    }
    match /tournaments/{id} {
      allow read: if request.auth != null;
      allow update: if request.auth != null;  // for joined counter
      allow create, delete: if false;         // admin only via console
    }
    match /transactions/{id} {
      allow read: if request.auth != null && request.auth.uid == resource.data.uid;
      allow create: if request.auth != null && request.auth.uid == request.resource.data.uid;
      allow update, delete: if false;         // admin only
    }
    match /registrations/{id} {
      allow read: if request.auth != null && request.auth.uid == resource.data.uid;
      allow create: if request.auth != null && request.auth.uid == request.resource.data.uid;
    }
  }
}
```
