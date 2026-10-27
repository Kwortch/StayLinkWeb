# StayLink — Firebase setup

This build wires the app up to real Firebase Auth + Firestore. Do these steps
once before your booking flow and dashboard will actually work.

## 1. Enable Email/Password sign-in
Firebase console → **Build → Authentication → Sign-in method** →
enable **Email/Password**.

## 2. Create a Firestore database
Firebase console → **Build → Firestore Database → Create database**
→ Production mode → pick a region close to your users (e.g. `asia-southeast1`).

## 3. Set your admin account(s)
Two files need the same email address(es) — one controls the UI, the other
actually enforces access:

- `StayLink Hotel Reser/index.html` → near the top of the `<script>` block:
  ```js
  const ADMIN_EMAILS = ["admin@staylink.com"];
  ```
- `firestore.rules` → inside `isAdminEmail()`:
  ```
  request.auth.token.email in ["admin@staylink.com"]
  ```
  Replace with your real group/adviser email(s), e.g.
  `["staylink.admin@gmail.com", "yourname@gmail.com"]` (comma-separated list in both places).

Then simply **sign up** for a normal account using that same email through the
app's own "Sign in → Create an account" form — no separate admin creation step
needed. Once signed in with an allow-listed email, the "Dashboard" link
appears in the nav automatically.

## 4. Deploy
From the project folder (where `firebase.json` lives):
```bash
firebase deploy
```
This deploys both Hosting (the site) and the Firestore rules together.

To test locally with the same Firebase backend first:
```bash
firebase emulators:start
```
Opening `index.html` directly as a local file (`file://...`) will **not**
work — the app depends on `/__/firebase/init.js`, which only exists when
served by Firebase Hosting or the emulator.

## What's now real vs. still a demo simplification
- **Real:** email/password accounts, reservations written to Firestore,
  "My Stays" reads only the signed-in guest's own bookings, room
  Available/Occupied/Cleaning status is live and admin-editable, the admin
  dashboard reads real reservation + room data.
- **Simplified on purpose (documented, easy to extend later):** room
  *content* (name, photos, description, amenities) still lives in the
  `ROOMS` array in code rather than Firestore — only the parts that actually
  change at runtime (status, bookings) are in the database. Booking a room
  marks it "Occupied" outright rather than checking for date-range overlaps
  against other reservations. Admin access is a simple email allow-list
  rather than Firebase custom claims — fine for a capstone demo, but worth
  mentioning as a "future work" limitation in your documentation.
