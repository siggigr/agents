---
name: android-app-bootstrap
description: Use when starting milestone 0 of a new Android app: sets up the GitHub repo, Firebase project, Firestore, release keystore, CI secrets and test emulators by CLI, with the owner approving every step and no secrets in chat.
---

# Android app bootstrap (milestone 0 setup)

Sets up everything outside the code that a new Kotlin / Jetpack Compose app
needs before the first feature milestone: repository, Firebase, signing, CI
secrets and emulators. The owner approves every command. Code scaffolding
(Gradle modules, CI workflows, lint config) is the developer's milestone 0
work and is not part of this skill.

## Security rules (always)

- The owner signs in to every tool personally (`gh auth login`,
  `firebase login`). Never ask for, paste or print tokens, passwords, keys
  or the contents of secret files.
- Run one command at a time and only after the owner approves it. Never
  suggest skipping permission prompts.
- Move secrets from file to destination directly (`gh secret set NAME <
  file`). When a password is needed, let the tool prompt the owner.
- Keystores and their base64 copies live outside the repository.
- `.gitignore` is committed before any secret or config file exists in the
  working tree.
- Stop on the first error. Explain it and wait. Do not retry with broader
  permissions or a different tool.
- Confirm permanent choices with the owner before running them: package
  name, Firebase project ID, Firestore location, keystore alias.

## Before starting

1. Read `docs/architecture.md` (build and release section, milestone 0) and
   `docs/requirements.md` if they exist. Take the values below from them.
2. Confirm these values with the owner in one list:
   - App display name, repository name (owner/repo), visibility (private by
     default)
   - Application ID (package name), for example `is.example.app`
   - Whether Firebase is used. If so: project ID, Firestore location (for
     EEA data, `europe-north1` or another EU region), sign-in providers
   - Emulator API levels needed (for example minSdk and the latest)
3. Check tools and sign-ins, and tell the owner what is missing:
   `git --version`, `gh --version`, `gh auth status`, `firebase --version`,
   `firebase login:list`, `keytool -help`, `sdkmanager --version`.
   The owner runs any `login` command themselves.

## Phase 1: GitHub repository

1. Create the repo (skip if it exists):
   `gh repo create OWNER/REPO --private --clone`
2. First commit: `.gitignore` (Android, plus `local.properties`, `*.jks`,
   `*.keystore`, `*.b64`, `app/google-services.json`, `.idea/`, `.gradle/`,
   `build/`) and `.gitattributes` (`* text=auto`, `*.bat text eol=crlf`,
   `gradlew text eol=lf`). Commit through a branch and pull request if main
   already has content.
3. Merge settings:
   `gh repo edit OWNER/REPO --enable-squash-merge --enable-merge-commit=false --enable-rebase-merge=false --delete-branch-on-merge`
4. Dependabot alerts:
   `gh api -X PUT repos/OWNER/REPO/vulnerability-alerts`
   The developer adds `.github/dependabot.yml` (weekly, grouped per
   ecosystem) in milestone 0.
5. Branch protection: on a free plan with a private repo it is not
   enforced. Tell the owner, and record the manual rule ("never push to
   main, merge only through pull requests") in the architecture. On a
   public or paid repo, offer a ruleset requiring pull requests and green
   checks.

## Phase 2: Firebase (skip if not used)

1. Create the project:
   `firebase projects:create PROJECT_ID --display-name "APP NAME"`
2. Register the Android app:
   `firebase apps:create ANDROID "APP NAME" --package-name APPLICATION_ID --project PROJECT_ID`
   Note the app ID it prints.
3. Create Firestore. The location cannot be changed later, so confirm it
   first:
   `firebase firestore:databases:create "(default)" --location LOCATION --project PROJECT_ID`
4. Commit `firebase.json`, `.firebaserc` and a deny-all `firestore.rules`,
   then deploy them:
   `firebase deploy --only firestore:rules --project PROJECT_ID`
5. Debug SHA-1 for Google sign-in: run `./gradlew signingReport` (or
   `gradlew.bat` on Windows), take the debug SHA-1, then:
   `firebase apps:android:sha:create APP_ID SHA1 --project PROJECT_ID`
6. Download the config without printing it:
   `firebase apps:sdkconfig ANDROID APP_ID --project PROJECT_ID --out app/google-services.json`
   Confirm it is ignored: `git check-ignore app/google-services.json`.
7. Owner by hand in the Firebase console: enable each sign-in provider
   (for example Google), and set the support email.

## Phase 3: Release signing

1. Agree on a folder outside the repo, for example
   `%USERPROFILE%\keystores\` on Windows or `~/keystores/`.
2. Create the keystore. The owner types both passwords when prompted:
   `keytool -genkeypair -v -keystore PATH/APP-release.jks -alias APP -keyalg RSA -keysize 4096 -validity 10000`
3. Tell the owner to store the keystore file and passwords in a password
   manager or another safe backup now. A lost keystore means the app can
   never be updated under the same signature.
4. Base64 copy, outside the repo:
   - PowerShell: `[Convert]::ToBase64String([IO.File]::ReadAllBytes("PATH\APP-release.jks")) | Out-File -Encoding ascii PATH\APP-release.b64`
   - bash: `base64 -w0 PATH/APP-release.jks > PATH/APP-release.b64`
5. If Firebase sign-in is used, add the release SHA-1 as well:
   `keytool -list -v -keystore PATH/APP-release.jks -alias APP` (owner
   types the password), then `firebase apps:android:sha:create` as above.

## Phase 4: CI secrets

Use the secret names the architecture names. Typical set:

- `gh secret set KEYSTORE_BASE64 < PATH/APP-release.b64`
- `gh secret set KEYSTORE_PASSWORD` (gh prompts, owner types)
- `gh secret set KEY_ALIAS` (gh prompts, owner types)
- `gh secret set KEY_PASSWORD` (gh prompts, owner types)
- If CI builds need it: `gh secret set GOOGLE_SERVICES_JSON < app/google-services.json`

Then delete the `.b64` file and check with `gh secret list` (names only).

## Phase 5: Emulators

For each API level, use a Google Play image so Google sign-in works:

1. `sdkmanager "system-images;android-API;google_apis_playstore;x86_64"`
2. `avdmanager create avd -n "Pixel_API_API" -k "system-images;android-API;google_apis_playstore;x86_64" -d pixel_4`
   (older API levels need an older device profile such as `pixel_4`).
3. Owner by hand: start each emulator and sign in to Google, using a
   separate test account where a second user is needed.

## Phase 6: Verify and record

1. `git ls-files` shows no `.jks`, `.keystore`, `.b64`, `local.properties`
   or `google-services.json`.
2. `gh secret list`, `firebase apps:list --project PROJECT_ID` and
   `firebase apps:android:sha:list APP_ID --project PROJECT_ID` show the
   expected entries.
3. Add a short entry to `docs/status.md` (create it if missing): date, what
   was set up, what the owner did by hand, and anything still open. Never
   include secret values, only names.
4. Report to the owner in under 150 words: done, done by hand, still open.
   Then stop. The developer continues with the code part of milestone 0.
