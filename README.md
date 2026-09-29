# Pressão Arterial — Blood Pressure Tracker

A local, self-hosted web app for logging hourly blood pressure readings (systolic, diastolic, pulse), built for family use. Single self-contained HTML file, no backend server required — data is stored in a free Firebase Firestore database and synced in real time between everyone who has access.

## Features

- **Hourly logging** with date/time defaulted to "now" but fully editable (for entries logged with a delay).
- **Multiple people**, each with their own independent history — switch between them from a dropdown.
- **Daily view**: averages for morning (05:00–11:59), afternoon (12:00–18:59), night (19:00–04:59), and the full day, plus a chart and a record table. Day-by-day navigation.
- **Weekly view**: same period breakdown, aggregated across the whole week, with navigation between weeks, a line chart, and bar charts comparing periods.
- **Total view**: same breakdown across every record ever logged for the selected person, plus a week-over-week trend chart for systolic pressure.
- **Automatic observation**: a local, rule-based summary (no external AI service, no data leaves the browser except to your own Firestore database) using user-configured reference values (120/80 mmHg, pulse 60–100 bpm) rather than a fixed clinical table. Flags low readings, out-of-range individual readings (not just averages), pulse pressure, variability, and small sample sizes. This is informational only — not a diagnosis.
- **Shared "spaces"**: anyone can create an account. New accounts get a private, randomly generated space code; sharing that code lets someone else join the same space during sign-up, so specific people (e.g. family members) see the same shared data, while unrelated users stay fully isolated.
- **Backup/restore**: export all data for the current space as JSON, and re-import it later.

## Setup

This app needs a free [Firebase](https://console.firebase.google.com) project to store data (Firestore) and handle logins (Authentication). No coding is required beyond pasting a config object.

1. Create a Firebase project.
2. Enable **Firestore Database** (production mode).
3. Enable **Authentication** → Sign-in method → **Email/Password**.
4. In Firestore → **Rules**, paste:

   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       function signedIn() { return request.auth != null; }
       function myGroup() {
         return get(/databases/$(database)/documents/users/$(request.auth.uid)).data.groupId;
       }
       match /users/{uid} {
         allow read, write: if signedIn() && request.auth.uid == uid;
       }
       match /pessoas/{id} {
         allow read, update, delete: if signedIn() && resource.data.groupId == myGroup();
         allow create: if signedIn() && request.resource.data.groupId == myGroup();
       }
       match /registros/{id} {
         allow read, update, delete: if signedIn() && resource.data.groupId == myGroup();
         allow create: if signedIn() && request.resource.data.groupId == myGroup();
       }
     }
   }
   ```

5. In Project settings → General → Your apps, register a **Web app** and copy the `firebaseConfig` object.
6. Open `index.html` and replace the placeholder `firebaseConfig` near the top of the `<script>` block with your own values.
7. Publish the file with **GitHub Pages** (Settings → Pages → deploy from the `main` branch, root folder).
8. In Firebase → Authentication → Settings → Authorized domains, add your GitHub Pages domain (e.g. `yourusername.github.io`).

## Usage

1. Open the published page and create an account (email/password). This generates a private space code.
2. Share that code with anyone you want to see the same data — they choose "join an existing space" when signing up.
3. Register at least one person, then start logging readings.

## Data model (Firestore collections)

- `users/{uid}` — `{ email, groupId }`
- `pessoas/{id}` — `{ nome, groupId }`
- `registros/{id}` — `{ pessoaId, groupId, date, time, sis, dia, pul, criadoPor, criadoEm }`

## Disclaimer

The observation feature classifies readings against user-configured reference values (120/80 mmHg, pulse 60–100 bpm) and highlights patterns (time-of-day differences, weekly trends, individual out-of-range readings, low pulse pressure, high variability). It is not a medical diagnosis and does not replace professional care — reported symptoms (dizziness, faintness, weakness) always matter more than a numeric classification; seek medical attention when they occur, regardless of what the table shows.
