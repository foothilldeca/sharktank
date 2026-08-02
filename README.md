# Foothill DECA — Shark Tank Challenge Portal

A static, multi-page site for the Shark Tank Challenge: a public site for student/mentor/judge
sign-up, and a password-gated officer dashboard for tracking registrations.

```
foothill-deca-shark-tank/
├── index.html        Overview / home page
├── register.html     Student registration form
├── mentor.html        Mentor application form
├── judge.html         Judge sign-up form
├── schedule.html       Three-day event schedule
├── officer.html        Officer dashboard (access-code gated)
├── assets/
│   ├── style.css      Shared styles
│   ├── app.js         Shared nav, ticker, CSV/export helpers
│   └── data.js        Storage layer (Firestore or local demo mode)
└── README.md
```

## 1. Put this on GitHub Pages

1. Create a new GitHub repository (public or private — Pages works with both on paid plans;
   public repos get Pages free).
2. Upload everything in this folder to the repo root (drag-and-drop on github.com works,
   or `git add . && git commit -m "Shark Tank portal" && git push`).
3. In the repo, go to **Settings → Pages**.
4. Under **Build and deployment**, set **Source** to "Deploy from a branch," branch `main`,
   folder `/ (root)`. Save.
5. GitHub gives you a URL like `https://<your-username>.github.io/<repo-name>/` within a
   minute or two. That's the live site — share it in your Google Classroom posts, Instagram
   bio, and the email blast to families.

## 2. Set up shared, live storage (Firebase — free tier)

Right now the site runs in **local demo mode**: registrations only save to the browser that
submitted them, so officers on a different device won't see them. You'll see a banner on
every page saying so. To make registrations shared and live for everyone, connect a free
Firebase project:

1. Go to [console.firebase.google.com](https://console.firebase.google.com) and create a
   new project (any name, e.g. "foothill-deca-shark-tank"). Google Analytics is optional —
   you can skip it.
2. In the project, click **Build → Firestore Database → Create database**. Choose
   **production mode** and pick a region close to you (e.g. `us-west1`).
3. Go to **Project settings → General**, scroll to "Your apps," click the `</>` (web) icon,
   register an app (any nickname), and skip the Firebase Hosting step — you're already
   using GitHub Pages.
4. Firebase shows you a config object like:
   ```js
   const firebaseConfig = {
     apiKey: "AIza...",
     authDomain: "foothill-deca-shark-tank.firebaseapp.com",
     projectId: "foothill-deca-shark-tank",
     storageBucket: "foothill-deca-shark-tank.appspot.com",
     messagingSenderId: "...",
     appId: "..."
   };
   ```
   Copy those six values into `FIREBASE_CONFIG` at the top of `assets/data.js`.
5. In Firestore, go to the **Rules** tab and set rules so the site can read and write while
   the event is being planned. A simple starting point:
   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /{document=**} {
         allow read, write: if true;
       }
     }
   }
   ```
   **This is wide open** — anyone with your Firebase project ID could read or write data.
   That's an acceptable tradeoff for a low-stakes school sign-up sheet with no sensitive
   data (no payment info is collected here), but two easy upgrades if you want more safety:
   - After registration closes, change `allow write: if true` to `allow write: if false` so
     the data becomes read-only.
   - Add [App Check](https://firebase.google.com/docs/app-check) to block traffic that isn't
     coming from your actual site.
6. Commit and push the updated `assets/data.js`. Reload the site — the demo-mode banner
   should disappear, and entries submitted from any device will now show up for everyone,
   including in the officer dashboard.

## 3. Change the officer access code

Open `officer.html`, find this line near the bottom:
```js
const OFFICER_CODE = "FOOTHILLDECA26";
```
Change it to whatever your officer team wants, then commit and push. This is a soft gate
(it's visible in the page source to anyone who inspects it) — good enough to keep casual
visitors out, not a substitute for the Firestore rules tightening in step 2 above.

## 4. What's already wired up

- **Student sign-up** (`register.html`) — name, email, school, grade, allergies, parent
  name/email/phone.
- **Mentor sign-up** (`mentor.html`) — name, email, grade, ICDC qualification, 1st/2nd
  choice of the six workshop/coaching topics.
- **Judge sign-up** (`judge.html`) — name, email, industry/role, Day 3 confirmation.
- **Officer dashboard** (`officer.html`) — live counts, searchable tables for all three
  groups, inline team assignment for students, CSV export per table, delete for cleaning
  up test entries.
- **Live ticker** on every page showing registration counts and a countdown to Day 1
  (Oct 5, 2026).

## 5. What's *not* wired up (by design — you'll want to handle these separately)

- **Email confirmations.** Submitting a form does not send an email. Officers should pull
  the CSV export and handle confirmations through your existing email workflow (or wire up
  a service like EmailJS or a Google Apps Script if you want it automated).
- **Payment collection.** The $59/$75 fee is shown as information only — no payment form
  is built in, matching the existing process of collecting checks separately.
- **Registration cutoffs / capacity limits.** The forms accept submissions any time the site
  is live. If you want registration to close automatically after September 22 or once a
  seat cap is hit, that logic would need to be added to `register.html`.
