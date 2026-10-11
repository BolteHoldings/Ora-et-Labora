# Ora et Labora — Android Build via GitHub Actions

## Status: What's Real vs. What Still Needs Setup

This project has been through a publishability pass. Here's exactly where things stand:

**Already real, working code:**
- ✅ Local Notifications (`@capacitor/local-notifications`) — daily reminders actually schedule and fire on-device
- ✅ Haptics (`@capacitor/haptics`) — real native haptic feedback, not the unreliable web Vibration API
- ✅ App Lock PIN — hashed with SHA-256 before storage, never stored in plain text
- ✅ Branded app icon and splash screen — no more Capacitor placeholder
- ✅ Android manifest permissions for notifications (`POST_NOTIFICATIONS`, `SCHEDULE_EXACT_ALARM`, `RECEIVE_BOOT_COMPLETED`)
- ✅ `PRIVACY_POLICY.md` and `TERMS_OF_SERVICE.md` — ready to host, need your contact email and a legal review before publishing

**Still simulated — needs your setup before launch:**
- ⚠️ **Subscriptions are NOT yet real Google Play Billing.** The `subscribe()` function still just sets a local flag. This is the one remaining hard blocker — Google Play requires all in-app digital subscriptions to go through Play Billing. See "Setting Up Real Billing" below for the recommended path.

---

This project wraps your Daily Office prototype as a real Android app using
[Capacitor](https://capacitorjs.com/), and uses **GitHub Actions** to build
the actual installable APK/AAB files in the cloud — so you never need to
install Android Studio, the Android SDK, or any build tools on your own
computer.

Read this whole file once before you start. Then follow the steps in order.

---

## How This Works, In Plain Terms

1. Your app's content (`www/index.html`) is the same prototype you already
   reviewed — same design, same content, same behaviour.
2. Capacitor wraps that HTML/CSS/JavaScript inside a real native Android
   project (the `android/` folder). This is what makes it installable from
   the Play Store rather than just a website.
3. Every time you push a change to GitHub, a robot (GitHub Actions) spins
   up a temporary computer in the cloud, installs the Android build tools,
   compiles your app, and hands you back two files:
   - a **debug APK** — install this directly on your own phone to test
   - a **release AAB** — this is the file you actually upload to the
     Google Play Console

You do not need to understand Gradle, Java, or Android Studio to use this.
You only need to follow the steps below.

---

## Step 1 — Create Your GitHub Account and Repository

1. If you don't already have one, sign up at [github.com](https://github.com).
2. Click the **+** icon in the top right → **New repository**.
3. Name it something like `ora-et-labora-app`.
4. Set it to **Private** (recommended, since your content library is your own
   creative work) or Public — your choice.
5. Do **not** initialise it with a README, since you already have one.
6. Click **Create repository**.

---

## Step 2 — Upload This Project to GitHub

On your own computer (or by using GitHub's website upload feature directly),
you need to get this entire folder into your new repository.

**Easiest method — no command line needed:**

1. On your new repository's GitHub page, click **uploading an existing file**.
2. Drag this entire project folder's contents into the browser window.
   GitHub will let you upload folders and files together in modern browsers.
3. Scroll down and click **Commit changes**.

**If you're comfortable with basic command line**, this is faster:

```bash
cd ora-et-labora-app
git init
git add .
git commit -m "Initial commit of Ora et Labora app"
git branch -M main
git remote add origin https://github.com/YOUR-USERNAME/ora-et-labora-app.git
git push -u origin main
```

---

## Step 3 — Watch Your First Build Run

As soon as you push, GitHub Actions will automatically start building your
app. Here's how to see it:

1. On your repository's GitHub page, click the **Actions** tab at the top.
2. You'll see a workflow run called "Build Android App" — click it.
3. It takes roughly 3–6 minutes to complete. You'll see each step run live:
   setting up Java, setting up the Android SDK, building the debug APK,
   building the release bundle.
4. When it finishes with a green checkmark, scroll down to the
   **Artifacts** section at the bottom of that run's page.
5. You'll see two downloadable files: `app-debug-apk` and `app-release-aab`.
   Click either to download a `.zip` containing your file.

**This first build will succeed for the debug APK, but the release AAB step
will produce an *unsigned* file** until you complete Step 4 below. An
unsigned AAB cannot be uploaded to the Play Store — so don't skip Step 4.

---

## Step 4 — Create Your Signing Key (Required for Play Store)

Every Android app must be signed with a private key before Google Play will
accept it. This key is yours alone — losing it means you can never update
your app again under the same listing, so treat it like a password you must
never lose.

You'll generate this using a tool called `keytool`, which comes bundled with
Java. If you don't have Java installed locally, the easiest route is to
generate it using an online interface or ask a technically-minded friend to
run one command for you — it takes 30 seconds. The command is:

```bash
keytool -genkeypair -v -keystore release-key.jks -keyalg RSA -keysize 2048 -validity 10000 -alias ora-et-labora
```

You'll be asked to set:
- A **keystore password** (write this down somewhere safe)
- A **key password** (can be the same as above)
- Your name, organisation, city, and country (this is just metadata, not
  verified by anyone)

This produces a file called `release-key.jks`. **Do not commit this file to
GitHub** — it must stay private. This is why it's already listed in
`.gitignore`.

---

## Step 5 — Add Your Signing Key to GitHub Secrets

GitHub Secrets let you store sensitive values (like your keystore) so your
automated build can use them without ever exposing them publicly.

1. Convert your keystore file to a text format so it can be safely stored:

   ```bash
   base64 -i release-key.jks | tr -d '\n' > keystore-base64.txt
   ```

   (On Windows, use `certutil -encode release-key.jks keystore-base64.txt`
   and then remove the header/footer lines it adds.)

2. Open the contents of `keystore-base64.txt` — it will be one long string
   of text. Copy all of it.

3. On your GitHub repository page, go to **Settings** → **Secrets and
   variables** → **Actions**.

4. Click **New repository secret** and add each of the following four
   secrets:

   | Secret Name | Value |
   |---|---|
   | `ANDROID_KEYSTORE_BASE64` | The long text you copied in step 2 |
   | `ANDROID_KEYSTORE_PASSWORD` | The keystore password you set in Step 4 |
   | `ANDROID_KEY_ALIAS` | `ora-et-labora` (or whatever alias you chose) |
   | `ANDROID_KEY_PASSWORD` | The key password you set in Step 4 |

5. Push any small change to your repository (or re-run the workflow from
   the Actions tab) to trigger a new build. This time, the release AAB
   artifact will be properly signed and ready for the Play Store.

---

## Step 6 — Test the App on Your Own Phone

Before submitting anywhere, install the debug APK on your own Android phone:

1. Download the `app-debug-apk` artifact from your latest successful
   GitHub Actions run (see Step 3).
2. Unzip it — you'll find `app-debug.apk` inside.
3. Transfer this file to your Android phone (email it to yourself, use
   Google Drive, or a USB cable).
4. On your phone, tap the file. Android will ask you to allow installs
   from this source the first time — approve it.
5. The app installs and opens like any other app on your phone.

Test everything: both offices, all days, saving entries, viewing history.

---

## Step 7 — Submit to the Google Play Store

1. Register a Google Play Developer account at
   [play.google.com/console](https://play.google.com/console) — this is a
   one-time $25 fee.
2. Click **Create app** and fill in the basic details (name, language,
   free or paid, declarations).
3. Under **Release → Production**, click **Create new release**.
4. Upload the `app-release.aab` file from your latest signed build
   (downloaded from GitHub Actions Artifacts as described in Step 5).
5. Complete the required sections in the left sidebar: Store listing
   (description, screenshots, icon), Content rating questionnaire,
   Privacy policy URL, and Target audience.
6. Submit for review. Google's review typically takes 1–7 days for a new
   app.

---

## Setting Up Real Billing

**The code side of this is done.** The app now calls real RevenueCat APIs (`configure`, `getOfferings`, `purchasePackage`, `restorePurchases`, `getCustomerInfo`) instead of faking a purchase — I verified every method name and parameter shape directly against the installed package's own type definitions rather than guessing, since getting this wrong is exactly the kind of mistake that only surfaces once real money is involved. The plugin (`@revenuecat/purchases-capacitor`) is installed and synced into the Android project already.

**What's left is entirely account setup on your end** — I can't create these accounts for you:

1. Create a free account at [revenuecat.com](https://www.revenuecat.com) and connect it to your Google Play Console app.
2. In Play Console, create two subscription products matching what's in the app: a monthly plan ($1.99) and an annual plan ($14.99).
3. In RevenueCat, create an **Entitlement** — the identifier must exactly match `ENTITLEMENT_ID` near the top of the `<script>` section in `www/index.html` (currently set to `"premium"` — change either side to match, doesn't matter which, they just have to agree).
4. In RevenueCat, create an **Offering** with your monthly and annual packages attached, and mark it as the **Current** offering — the app reads `offerings.current`, so an offering that exists but isn't marked current won't be found.
5. Copy your RevenueCat **public** API key (Project Settings → API Keys — use the public one, never a secret key, inside client-side app code).
6. In `www/index.html`, find the line `const REVENUECAT_API_KEY = 'YOUR_REVENUECAT_PUBLIC_API_KEY';` near the top of the script, and replace the placeholder with your real key.
7. Re-sync and rebuild: `npx cap sync android`, then push to GitHub so Actions builds a fresh APK/AAB with the real key baked in.

Until step 6 is done, the app detects the placeholder key and simply skips billing initialization — the paywall screen still displays correctly, but tapping Subscribe won't attempt a real purchase. This is intentional so the rest of the app remains testable before billing is fully wired up.

**One thing to test carefully once it's live:** RevenueCat's own dashboard has a sandbox/test mode — use it before ever taking a real payment, following RevenueCat's testing guide for Google Play. Do not test purchases with your real payment method.

## Making Changes Later

Whenever you want to update your app — new content, design tweaks, bug
fixes — the workflow is always the same:

1. Edit the files in the `www/` folder (this is your HTML/CSS/JavaScript).
2. In `android/app/build.gradle`, increase `versionCode` by 1 and update
   `versionName` (e.g., from `"1.0"` to `"1.1"`) — Google Play requires
   this for every new upload.
3. Push your changes to GitHub.
4. Wait for the Actions build to complete, download the new signed AAB,
   and upload it as a new release in the Play Console.

---

## A Note on What You Now Own

This project is now a real, standard Capacitor + Android project — the
same kind of codebase professional Android developers work with daily.
If at any point you want a developer's help (freelancer, friend, or an
agency), this is a completely ordinary, well-documented starting point for
them to pick up. Nothing about the automated GitHub build locks you in —
you can always open the `android/` folder in Android Studio directly if you
later decide to build locally instead.
