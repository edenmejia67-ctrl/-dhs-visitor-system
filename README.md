# Dapdap High School Visitor System (Phase 3: the app)

**Starting from nothing (Phase 2 skipped)? Follow `SETUP-GUIDE.md`. It has every click for Supabase, GitHub, example accounts and installing on phones.**

An installable Android app (a web app you install from Chrome). Guards register visitors, teachers get an **in-app inbox** plus a **phone push alert**, and can answer with one tap. No ntfy and no extra app to install.

```
site/        index.html, config.js, sw.js, manifest.webmanifest, icon-192.png, icon-512.png, dhs-logo.jpg   <- the app (GitHub publishes this folder)
supabase/    schema.sql (Phase 2)  phase3.sql (new)  functions/send-alert/index.ts (new)  config.toml
.github/workflows/deploy.yml    archive/
```

## What is new
- **Installable app:** Chrome on Android shows **Install app**. It gets its own icon and opens full screen.
- **In-app inbox:** each teacher sees their visitors in the app, with the time, purpose and status. Unanswered ones show a red badge.
- **Push alerts:** the server sends them (no secrets in the browser), encrypted, even when the app is closed.
- **Urgent priority:** the guard picks Normal or Urgent. Urgent uses high-priority delivery and a strong vibration pattern.
- **Teacher replies:** **On my way**, **Please wait**, **Cannot meet**, or **Got it**. The guard sees the reply live (toast, sound, and in Notifications).
- **Auto-resend:** an urgent alert nobody has answered is sent again every 2 minutes, up to 3 more times, while the guard's page is open (optional cron below makes it work even when the page is closed).
- **Sound + vibration** when the app is open, for teachers (new visitor) and guards (teacher replied).

## Upgrade from Phase 2 (about 20 minutes)
### 1. Database
1. Supabase > **SQL Editor > New query**, paste all of `supabase/phase3.sql`, **Run**. (It removes the old ntfy columns. Fresh project: run `schema.sql` first, then `phase3.sql`.)
2. Supabase > **Edge Functions > Deploy a new function > Via Editor**. Name it exactly `send-alert`, paste `supabase/functions/send-alert/index.ts`, **Deploy**.
3. Open that function's settings and turn **off** "Enforce JWT verification" (the function checks the sign-in and role itself).
4. Optional secret (Edge Functions > Secrets): `VAPID_SUBJECT` = `mailto:` plus any real email you control. Not needed for Android to work.

### 2. GitHub
1. Upload these into the repo's `site` folder, replacing the old ones: `index.html`, `sw.js`, `manifest.webmanifest`, `icon-192.png`, `icon-512.png`.
2. **Keep your own `config.js`** (it holds your Project URL and key). The old `ntfy` line in it is no longer used and can stay or go.
3. Wait for the green run in the **Actions** tab, then open the site.

### 3. Each teacher's phone (once)
1. Open the site in **Chrome on Android**, sign in with your staff ID, then tap **Install app** on the Notifications page (or Chrome menu > Install app).
2. Open the **DHS Alerts** icon and sign in.
3. Tap **Enable alerts on this phone** and allow notifications.
4. Tap **Send test**. The phone should show "DHS Visitor Alert (TEST)". If it is silent: Android Settings > Apps > DHS Alerts (or Chrome) > Notifications > turn sound on, and set battery to "Unrestricted".

### 4. Full test
Guard: Register Visitor, a fictional name, pick the teacher, Priority **Urgent**, Register. The teacher's phone vibrates and shows "URGENT". The teacher taps **On my way**. The guard hears a beep and sees "SANTOS, MARIA: On my way".

## Daily workflow
1. **Guard:** Register Visitor > choose teacher > Normal or Urgent > Register. A note under the teacher shows whether a phone is registered.
2. **Teacher:** gets the push alert, opens the app, taps a reply.
3. **Guard:** watches **Notifications** (Delivery and Teacher response). Press **Alert again** or **Retry** if needed. Press **Check out** when the visitor leaves.
4. **Admin:** Accounts (approve people), Faculty (add names as `LAST NAME, FIRST NAME`, send a test to a person's phones), Records (calendar, Export month CSV).

## What the words mean
| Word | Meaning |
|---|---|
| Accepted by push service | Google's push service took the alert. This does **not** prove the phone showed it |
| Acknowledged / reply | The teacher answered. This is the real proof they saw it |
| No phone registered | That person has not tapped Enable alerts yet. The in-app inbox still has the visit |
| Request failed | The server could not send. Read the small text, then Retry |

## Optional: auto-resend even when the guard's page is closed
At the bottom of `supabase/phase3.sql` there is a commented block. Enable **pg_cron** and **pg_net** (Database > Extensions), add the Edge secret `CRON_SECRET`, then run that block with your project address and the same secret.

## Troubleshooting
| Symptom | Cause |
|---|---|
| "Could not reach the server function" | `send-alert` not deployed, or named differently |
| "This account is not active" | Account is not approved or is disabled |
| Enable alerts: "background service is not running" | Opened over http or a file. Use the https GitHub Pages address |
| Enable alerts: "not linked to a faculty record" | Admin must approve them as Teacher and link or create their faculty record |
| Install app button missing | Use Chrome menu > Install app. Not available in other browsers |
| Accepted, nothing on the phone | Notifications off or silent, battery saver, Do Not Disturb, or the phone is offline |
| Request failed: "subscription expired" | Tap Enable alerts again on that phone |
| Guard sees "No phone registered" | The teacher has not enabled alerts, or is not linked to that faculty record |

## Known limits
- Push alerts travel through Google's push service (Android Chrome). They cannot break through silent mode or Do Not Disturb, and cannot play a looping alarm. For a true emergency, also phone the teacher.
- The app is a Chrome-installed web app, not a Play Store app. Teachers on iPhone are not set up in this phase.
- Auto-resend by default needs the guard's page open; use the optional cron block for the rest.
- Before real student data: database backups, a Data Privacy Act review (visitor name and contact are personal data), a retention rule (`select public.purge_old_visitors(365);`), and check that free Supabase projects pause after about a week of no use.
