# Sakoon — complete source code (Phases 1, 2, 3 & bonus features)

## Please read this first

I can't compile or run Flutter/Android builds inside this chat's sandbox —
there's no Flutter SDK here, and the sandbox's network access doesn't
reach `pub.dev` (Dart packages) or Google's Android SDK servers, so even
installing the toolchain isn't possible in this environment. I checked
directly rather than assuming:

```
$ which flutter
flutter: not found
$ curl -I https://pub.dev
HTTP/2 403  (host not allowed)
```

So what's in this folder is **real, complete, hand-written Dart/Flutter
source code** — not a mockup and not pseudocode — that becomes a running
app once you build it on your own machine, where Flutter has normal
internet access. This is not a "can't be done" situation, just a "has to
be compiled outside this chat" one. If you'd rather I drive the build
myself interactively (creating the project, running it on a device,
fixing anything that comes up), **Claude Code** on your own computer gives
me that same file-editing ability plus a real terminal — I'd genuinely
recommend it for the rest of this project.

## What's implemented

- ✅ Salah (prayer) time alarms — on-device calculation, Karachi method +
  Hanafi Asr by default, exact/Doze-resistant Android alarms, full
  settings screen (method, madhab, location, pre-prayer reminder)
- ✅ **Per-prayer alarm type** (আজান / নোটিফিকেশন / সাইলেন্ট / বন্ধ) — pulled
  in from the "Noor Diary" spec you shared; a single global reminder
  setting didn't fit everyone's day
- ✅ **Prayer check-in + streak** — tap to mark each of the 5 daily
  prayers as prayed, with a running streak shown on the home screen
- ✅ **Triple Date Card** — Hijri, real Bengali calendar (San), and
  Gregorian dates together on the home screen, plus sunrise/sunset —
  the standout idea from the Noor Diary document. The Bengali date is a
  genuine implementation of Bangladesh's 2019-revised calendar rules
  (`lib/core/bengali_calendar.dart`), not a placeholder.
- ✅ **Alarm troubleshooting screen** — Xiaomi/Oppo/Vivo/Realme/Samsung-
  specific battery-optimization steps, reached from Settings, with an
  honest "best effort, not guaranteed" framing rather than overpromising
- ✅ Calendar task reminders — add/edit/delete, priority, repeat option,
  calendar view, exact alarm per task
- ✅ Home dashboard matching the earlier design mockup
- ✅ Tasbih — persisted per-phrase count, presets + custom dhikr phrases,
  haptic feedback, target-reached alert, bead-ring visual, 7-day history
  + streak screen
- ✅ Notes — search, pin-to-top, tags, checklist mode, autosave on exit
- ✅ Backup & recovery, two independent layers:
  - automatic Google Sign-In + Firestore sync (`backup_screen.dart`)
  - manual JSON export/import via the OS share sheet — works with zero
    account, zero Firebase setup
- ✅ Qibla compass (bonus) — device magnetometer + great-circle bearing,
  with a calibration prompt
- ✅ Hijri calendar (bonus) — month grid with Ramadan/Eid highlights,
  opened by tapping the date card on the home screen

Every feature from the original Sakoon brief is implemented, plus the
five Noor-Diary-inspired additions above. See "What I deliberately left
out" below for the parts of that document I did not bring in, and why.

## What I deliberately left out of the Noor Diary document

The document you shared is a genuinely thorough spec, but a lot of it —
AES-256-GCM encrypted export with password-based key derivation,
BiometricPrompt-locked notes, Room-FTS-style full-text search, per-record
sync-status/conflict-resolution UI, a full JUnit/MockK/Compose-test
suite, CI — is real engineering work that would take a professional team
weeks, not something to bolt onto a personal project in one sitting.
Rather than add half-working versions of all of it, I picked the five
ideas above that were high-value and realistically completable this
round, and left the rest out rather than pretend to deliver it. If any
specific one of these matters to you, say which and I'll build that one
properly next — not everything at once.

## Option A — build in the cloud, no Flutter or Android Studio install

The easiest path if you don't want to install anything on your own
computer. This repo includes `.github/workflows/build-apk.yml`, which
builds a release APK entirely on GitHub's servers.

1. Make a free GitHub account at `github.com` if you don't have one.
2. Click **+ → New repository** (top right), give it any name, leave it
   **Public**, don't add a README/gitignore (this zip already has one),
   click **Create repository**.
3. On the new repo's page, click **uploading an existing file**. Drag
   the *entire extracted `sakoon_app` folder* (all of it — `lib`,
   `android`, `.github`, `pubspec.yaml`, everything) into the upload box.
   Modern browsers accept whole folders dropped this way. Scroll down
   and click **Commit changes** — no `git` command-line knowledge needed.
4. Click the **Actions** tab at the top of the repo. A run named "Build
   Sakoon APK" should already be in progress (it starts automatically on
   upload). It takes roughly 5–10 minutes the first time.
5. Once it finishes (green checkmark), click into that run, scroll to
   **Artifacts** at the bottom, and download **sakoon-release-apk** — a
   zip containing `app-release.apk`. Unzip it, then get it onto your
   phone (USB cable, or upload to Google Drive/email and download it on
   the phone) and open it there. Android will ask you to allow installs
   from that app (Files/Chrome/whichever you used) the first time — tap
   **Allow**, then **Install**.

**Honesty note:** I can't push to GitHub or trigger a run myself from
this sandbox, so this workflow file is written correctly to the best of
my knowledge (it uses the same standard, widely-used actions —
`actions/checkout`, `subosito/flutter-action`, `actions/upload-artifact`
— that most Flutter CI setups use) but hasn't been run end-to-end by me.
If the Actions tab shows a red ✗, open the failed step and paste me the
error — CI logs are usually very specific about what broke.

**Alternative:** Codemagic (`codemagic.io`) is another option built
specifically for Flutter, with a free tier and a guided web UI instead
of a YAML file, if you'd rather click through a wizard than use GitHub.

## Option B — build locally with Flutter + Android Studio

**1. Install tooling (one-time, on your own computer)**
- Flutter SDK: `flutter.dev/get-started/install`
- Android Studio (provides the Android SDK + an emulator): `developer.android.com/studio`
- Run `flutter doctor` and resolve anything it flags before continuing.

**2. Scaffold a fresh project, then drop these files in**
```bash
flutter create --org com.sakoon sakoon_app
cd sakoon_app
# Delete the generated placeholders:
rm -rf lib
rm pubspec.yaml
```
Now copy this chat's `lib/` folder and `pubspec.yaml` into `sakoon_app/`,
replacing what you just deleted.

**3. Replace the Android manifest**
Overwrite the freshly generated
`sakoon_app/android/app/src/main/AndroidManifest.xml` with the one
provided here. If you used a different `--org` value in step 2, update
the `package="com.sakoon.app"` line to match.

**4. Set the minimum Android version**
Open `android/app/build.gradle` (or `build.gradle.kts` on newer Flutter
versions) and set:
```
minSdk = 23
```
(23 comfortably covers everything needed for exact alarms and modern
notification handling.)

**5. Install packages and run**
```bash
flutter pub get
flutter run
```
Connect a physical Android phone (with USB debugging on) or start an
emulator from Android Studio first.

## Firebase setup (only needed for automatic cloud sync)

Skip this section entirely if you're fine relying on the manual
export/import backup layer — that one needs no account and no setup.
The automatic layer needs a few manual steps I genuinely cannot do for
you, since they require your own Google account and the Firebase web
console:

1. Go to `console.firebase.google.com` → **Add project** → name it
   anything (e.g. "Sakoon").
2. Inside the project, click **Add app → Android**. For the package
   name, enter exactly what's in your `AndroidManifest.xml`
   (`com.sakoon.app`, unless you changed it in step 2 of Setup above).
3. Download the `google-services.json` file it offers you, and place it
   at `android/app/google-services.json` in your project.
4. In `android/settings.gradle` (or `.kts`), add the Google Services
   plugin to the `plugins` block:
   ```
   id("com.google.gms.google-services") version "4.4.2" apply false
   ```
5. In `android/app/build.gradle` (or `.kts`), add near the top:
   ```
   id("com.google.gms.google-services")
   ```
6. In the Firebase console, go to **Build → Authentication → Sign-in
   method** and enable **Google** as a sign-in provider.
7. Go to **Build → Firestore Database → Create database**, and start in
   test mode (you can tighten security rules later — for a single-user
   personal app, restricting each `users/{uid}/...` path to
   `request.auth.uid == uid` is enough; the Firestore console's Rules
   tab has a template for exactly this).
8. `flutter clean && flutter pub get && flutter run`.

Until you do this, the app runs perfectly well — `Firebase.initializeApp()`
fails silently into local-only mode (see the comment in `main.dart`), and
every screen already reads from local Hive storage either way.

## Six spots flagged for verification

I don't have live access to `pub.dev` from this sandbox, so these six
API surfaces are written from my best knowledge of each package rather
than a live check against the exact version `pub get` resolves to. Each
is called out with a comment in the code too, and each is a small,
one-file fix if it doesn't match — `flutter run`'s error message will
point at the exact line:

1. **`lib/services/prayer_service.dart`** — the `adhan_dart` calculation-method
   factory calls (e.g. `CalculationMethod.karachi()`) and the `Madhab`
   constant. If this errors, open the adhan_dart package page on pub.dev
   for the current method names.
2. **`add_edit_task_screen.dart`** / **`settings_screen.dart`** — `DropdownButtonFormField`'s
   `initialValue:` parameter — some Flutter versions expect `value:` instead.
3. **`lib/services/notification_service.dart`** — the three `zonedSchedule(...)`
   calls pass `uiLocalNotificationDateInterpretation:`, required in older
   `flutter_local_notifications` releases and removed in some newer ones.
   Delete that one argument from all three calls if it errors.
4. **`lib/screens/hijri_calendar_screen.dart`** — the `hijri` package's
   `HijriCalendar.fromDate()` / `.hYear` / `.hMonth` / `.hDay` fields.
5. **`lib/services/auth_service.dart`** — deliberately pinned to
   `google_sign_in: ^6.x` in pubspec.yaml (see the comment at the top of
   that file) rather than the newer 7.x line, which redesigned the
   package's API around a singleton/async-init pattern.
6. **Firestore data types** — `sync_service.dart` reads Firestore
   documents with `.data() as Map<dynamic, dynamic>`; if a future
   `cloud_firestore` version tightens this cast, change it to
   `Map<String, dynamic>` at that call site.
7. **`lib/core/bengali_calendar.dart`** — this one isn't a package-version
   risk, it's arithmetic I derived by hand from the 2019 Bangladesh
   calendar reform rules (fixed 14 April new year; 31/31/31/31/31/30×6/
   29-or-30 day month lengths). I traced through the day-count math
   carefully, but a date converter is exactly the kind of code where a
   silent off-by-one is easy to miss — worth checking a few dates
   near month boundaries (mid-April, mid-March) against
   `calendar.gov.bd` or another official source before trusting it.

Everything else follows standard, stable Flutter/Dart APIs — these six
are the only spots where a package's current API might have moved since
my last verified knowledge of it.

## Why these specific technical choices

Full reasoning is in the project brief document from earlier in this
chat (`sakoon-app-development-prompt.md`) — in short: Hive instead of a
typed database (no `build_runner` code-generation step required, so
`flutter pub get` is the only setup command needed); exact
`setExactAndAllowWhileIdle` alarms with a full-screen intent on Android
so prayer alarms actually ring instead of sitting as a silent
notification; and a 7-day rolling notification window for prayer times
specifically because the times shift by a few minutes daily, so a naive
"repeat at the same clock time" notification would drift out of sync.
