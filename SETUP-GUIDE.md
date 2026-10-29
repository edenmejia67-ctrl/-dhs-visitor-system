# Setup guide: from nothing to a working app (Android)

You need: a laptop with Chrome, one email address for yourself (GitHub and Supabase sign-ups need one; staff never need an email), and an Android phone for testing.
Do the parts in order: **1 Supabase, 2 GitHub, 3 Example accounts, 4 Install on phones, 5 Test.**

---
## Part 1. Supabase (the database)
1. Go to **supabase.com** > **Start your project** > sign up (GitHub login is easiest).
2. **New project**. Name: `dhs-visitors`. Set a **Database password** and save it somewhere safe. Region: **Southeast Asia (Singapore)**. Plan: **Free**. Press **Create new project** and wait about 2 minutes.
3. **Run the first database file.** Left menu **SQL Editor** > **New query**. Open `supabase/schema.sql` from the unzipped project (right-click > Open with Notepad), select all, copy, paste into the editor, press **Run**. It must say **Success. No rows returned.**
4. **Run the second file.** **New query** again, paste all of `supabase/phase3.sql`, **Run**. If a warning about "destructive operations" appears, press **Run this query**. Again: **Success**.
   - If either run shows a red error, stop and send me the exact message. Do not skip it.
5. **Turn on sign-ups without email.** Left menu **Authentication** > **Sign In / Providers** (may be called **Providers**) > **Email**:
   - **Allow new users to sign up** = ON
   - **Confirm email** = **OFF** (very important, we use no real emails)
   - Minimum password length = 8 (if the option exists). **Save**.
6. **Copy your two keys.** **Project Settings** (gear icon) > **API Keys**. Copy the **Project URL** (looks like `https://abcdxyz.supabase.co`) and the **Publishable key** (starts with `sb_publishable_`). If your project only shows an "anon public" key, copy that instead. Never copy the "secret" or "service_role" key. Paste both into a Notepad file for Part 2.
7. **Deploy the alert function.** Left menu **Edge Functions** > **Deploy a new function** > **Via Editor**. Delete the sample code, paste all of `supabase/functions/send-alert/index.ts`. Name the function exactly `send-alert`. Press **Deploy function**.
8. Open the `send-alert` function > **Settings** (or **Details**) and turn **OFF** "Enforce JWT verification" (also called "Verify JWT"). Save.

---
## Part 2. GitHub (puts the app online)
1. Go to **github.com** > **Sign up** (or sign in).
2. Top right **+** > **New repository**. Repository name: `dhs-visitor-system`. Choose **Public** (free GitHub Pages needs public; the files contain no secrets). **Do not** tick "Add a README". Press **Create repository**.
3. On your laptop, **unzip** `dhs-visitor-system-phase3.zip` (right-click > Extract All). Open the extracted folder. You should see: `site`, `supabase`, `archive`, `.github`, `README.md`, `.gitignore`, `SETUP-GUIDE.md`.
   - Mac: press **Cmd + Shift + .** to show hidden items like `.github` and `.gitignore`.
4. On the new repository page click **uploading an existing file**.
5. Select **everything inside the extracted folder** and **drag it all** into the GitHub page (Chrome or Edge). Drag the folders themselves, not the zip file and not the outer folder. Wait until the list shows every file.
6. Scroll down, leave "Commit directly to the main branch", press **Commit changes**.
7. **Check the workflow file.** On the repository page open `.github` > `workflows` > `deploy.yml`. If it is missing, press **Add file** > **Create new file**, type the name exactly `.github/workflows/deploy.yml` (typing / makes folders), paste this, then **Commit changes**:

```yaml
# Publishes the "site" folder to GitHub Pages every time you commit to main.
name: Deploy site to GitHub Pages

on:
  push:
    branches: [main]
  workflow_dispatch:   # lets you press "Run workflow" manually

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: pages
  cancel-in-progress: true

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - uses: actions/checkout@v4
      - uses: actions/configure-pages@v5
      - uses: actions/upload-pages-artifact@v3
        with:
          path: site
      - id: deployment
        uses: actions/deploy-pages@v4
```

8. **Add your keys.** Open `site` > `config.js` > pencil icon (Edit). Replace `https://YOUR-PROJECT.supabase.co` with your Project URL and `sb_publishable_YOUR_KEY` with your publishable key. Keep the quotation marks. **Commit changes**.
9. **Turn on the website.** Repository **Settings** > **Pages** (left menu) > **Build and deployment** > **Source: GitHub Actions**.
10. Open the **Actions** tab. A run called "Deploy site to GitHub Pages" starts. Wait for the green check (about 1 minute).
    - Red X or no run? Open **Actions** > "Deploy site to GitHub Pages" > **Run workflow** (or open the failed run > **Re-run all jobs**). It fails if step 9 was not done first.
11. Your app address: **Settings** > **Pages**, or `https://YOUR-GITHUB-NAME.github.io/dhs-visitor-system/`. Open it on the laptop. You should see the Welcome Back screen with the school crest.
- **To change anything later:** open the file on GitHub > pencil > **Commit changes**. Or **Add file > Upload files** to replace a file. The site updates in about a minute.

---
## Part 3. Example accounts (pick ONE method)
### Method A: register inside the app (recommended, about 5 minutes)
1. Open your app address > **Create account**. ID: `admin01`. Name: `ADMIN, ALEX`. Role: any. Password: `Admin#2026dhs` twice. **Request account**. You will see "Waiting for approval".
2. Supabase > **SQL Editor** > **New query**, run: `select public.make_admin('admin01');`
3. Back in the app press **Check again**. You are now the administrator.
4. **Log out.** Create two more accounts the same way: `guard01` (REYES, PEDRO, "security guard") and `teacher01` (SANTOS, MARIA, "teacher or office staff"). Log out after each.
5. Log in as `admin01` > **Accounts**. For each waiting person press **Approve**. For `guard01` choose **Guard**. For `teacher01` choose **Teacher** and leave "Create a new record from their name".
6. Add offices: **Faculty** > **Add faculty** > `REGISTRAR` (type Office).

### Method B: one SQL paste (fast, less checked)
Supabase > **SQL Editor** > **New query** > paste all of `supabase/example-accounts.sql` > **Run**. It must end with a table of 6 rows. If you then cannot log in ("Database error querying schema"), use Method A instead.

### The example accounts (Method B creates all of these)
| Login ID | Password | Role | Faculty record |
|---|---|---|---|
| admin01 | Admin#2026dhs | Administrator | none |
| guard01 | Guard#2026dhs | Guard | none |
| teacher01 | Teacher#2026a | Teacher | SANTOS, MARIA |
| teacher02 | Teacher#2026b | Teacher | GARCIA, JOSE |
| registrar | Registrar#2026 | Office | REGISTRAR |
| headteacher | HeadTeacher#2026 | Office | HEAD TEACHER'S OFFICE |

**These passwords are public in this document. Change them (Settings > Change my password, or Accounts > key button) or delete the accounts before real use.** Real staff register themselves with their own staff ID and you approve them in **Accounts**.

---
## Part 4. Make it a downloadable app on the phones
### Normal way: install from Chrome (recommended)
1. Send the app address to the teacher (Messenger, Viber, SMS).
2. On the **Android phone** open it in **Chrome** and sign in.
3. On the **Notifications** page tap **Install app**. If the button is missing: Chrome menu (three dots) > **Install app** (or **Add to Home screen**). Press **Install**.
4. A **DHS Alerts** icon with the school crest appears with the other apps. Open the app from that icon, from now on.
5. Tap **Enable alerts on this phone** > **Allow**. Tap **Send test**: the phone must show "DHS Visitor Alert (TEST)".
6. If it is silent: Android **Settings > Apps > DHS Alerts** (or Chrome) > **Notifications** > turn on sound. Also **Battery > Unrestricted**.
Chrome builds a real Android app entry for it. Updates arrive by themselves when you edit the files on GitHub.

### Optional: a real APK file (not needed for alerts to work)
1. Go to **pwabuilder.com**, paste your app address, press **Start**.
2. Press **Package for stores** > **Android** > fill the form (package ID like `io.github.YOURNAME.dhs`) > **Generate**.
3. Download the zip, send the `.apk` file to the phone, open it, and allow "Install unknown apps" for that source.
Notes: I have not tested this step. Keep the signing key file in the zip safe. Because the site lives under `github.io/dhs-visitor-system/`, the APK will show a small browser bar. Alerts still work. For a bar-free app you would need your own domain.

---
## Part 5. First test (10 minutes)
1. Phone: log in as `teacher01`, install the app, **Enable alerts**, **Send test**.
2. Laptop: log in as `guard01` > **Register Visitor** > name "Test Bisita", purpose **Other** (type "Pickup"), choose **SANTOS, MARIA**, Priority **Urgent** > **Register**.
3. Phone vibrates and shows "URGENT". Open the app, tap **On my way**.
4. Laptop: you see "SANTOS, MARIA: On my way". **Visitor Records** shows today on the calendar. **Check out** the visitor.

## If something fails
| What you see | What to do |
|---|---|
| Red error when running a SQL file | Send me the exact message and which file |
| Page says "Not connected yet" | `site/config.js` still has the placeholders. Redo Part 2 step 8 |
| Create account says sign-up refused | Change `emailDomain` in `config.js` to another normal-looking domain (and in `example-accounts.sql` if used) |
| "Account created, but sign-in is blocked" | Confirm email is still ON in Supabase (Part 1 step 5) |
| Stays on "Waiting for approval" | Admin must approve in Accounts, then press **Check again** |
| "Could not reach the server function" | `send-alert` not deployed or misspelled (Part 1 step 7) |
| Enable alerts: "background service is not running" | You opened the file or an http address. Use the https github.io address |
| "Accepted" but no alert on the phone | Notifications off or silent, battery saver, or Do Not Disturb |
| GitHub Actions shows red X | Settings > Pages > Source must be "GitHub Actions", then re-run |
