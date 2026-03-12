# 🚀 How to Get Your APK via GitHub Actions
## Step-by-step guide (No Android Studio needed!)

---

## STEP 1 — Create a GitHub Account
Go to https://github.com and sign up (free).

---

## STEP 2 — Create a New Repository

1. Click the **"+"** button (top right) → **"New repository"**
2. Name it: `CallTrackApp`
3. Set to **Private** (your code is private)
4. Click **"Create repository"**

---

## STEP 3 — Upload the Project Files

### Option A: Using GitHub Website (Easiest)
1. In your new repository, click **"uploading an existing file"**
2. Drag and drop ALL files from the `CallRecordApp` folder
3. Click **"Commit changes"**

### Option B: Using Git (Terminal/Command Prompt)
```bash
# Install Git from https://git-scm.com if not installed
git init
git add .
git commit -m "Initial commit - CallTrack app"
git remote add origin https://github.com/YOUR_USERNAME/CallTrackApp.git
git push -u origin main
```

---

## STEP 4 — GitHub Actions Builds Automatically! 🎉

Once you push/upload files:
1. Go to your repository on GitHub
2. Click the **"Actions"** tab
3. You will see **"Build CallTrack APK"** workflow running
4. Wait 3–5 minutes for it to finish ✅

---

## STEP 5 — Download Your APK

1. Click on the completed workflow run
2. Scroll down to **"Artifacts"** section
3. Click **"CallTrack-Debug-APK"** to download
4. You get a `.zip` file — extract it to get `app-debug.apk`

---

## STEP 6 — Install on Your Android Device

1. Transfer `app-debug.apk` to your phone (WhatsApp, email, USB, Google Drive)
2. On your Android 15 phone, go to:
   **Settings → Apps → Special app access → Install unknown apps**
3. Allow your file manager or browser to install
4. Tap the APK file to install
5. Open **CallTrack** and follow setup:
   - Grant all permissions (mic, phone, contacts)
   - Tap **"Set as Default Dialer"** when prompted ← CRITICAL!
6. Make a test call — recording starts automatically! 🎙

---

## OPTIONAL: Set Up Release Signing (for production APK)

For a signed release APK (not required for personal use):

### Generate a keystore:
```bash
keytool -genkey -v -keystore calltrack.jks -alias calltrack -keyalg RSA -keysize 2048 -validity 10000
```

### Add secrets to GitHub:
Go to: Repository → Settings → Secrets → Actions → New secret

Add these 4 secrets:
| Secret Name | Value |
|-------------|-------|
| `SIGNING_KEY` | Base64 of your .jks file (`base64 calltrack.jks`) |
| `KEY_ALIAS` | `calltrack` |
| `KEY_STORE_PASSWORD` | Your keystore password |
| `KEY_PASSWORD` | Your key password |

---

## 📂 Required Folder Structure When Uploading

```
CallTrackApp/                          ← Upload everything inside here
├── .github/
│   └── workflows/
│       └── build.yml                 ← GitHub Actions (auto-build)
├── app/
│   ├── build.gradle
│   ├── proguard-rules.pro
│   └── src/main/
│       ├── AndroidManifest.xml
│       ├── java/...                  ← All .java files
│       └── res/...                   ← All layout/values/drawable files
├── gradle/wrapper/
│   └── gradle-wrapper.properties
├── build.gradle
├── settings.gradle
└── gradlew
```

---

## ❓ Common Problems

**Build fails with "Permission denied on gradlew"**
→ The workflow already runs `chmod +x gradlew` — this is handled.

**Build fails with "SDK not found"**
→ The workflow uses `android-actions/setup-android@v3` — this is automatic.

**APK installs but recording doesn't work**
→ Make sure you set CallTrack as the **Default Dialer** — this is mandatory on Android 15.

**"Install blocked" on phone**
→ Go to Settings → Security → Enable "Install from unknown sources" for your file manager.
