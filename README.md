# 📞 CallTrack — Android Call Recording App
### ConnectionService API Implementation for Android 15

---

## 🏗️ Architecture Overview

```
┌─────────────────────────────────────────────────────────┐
│                    USER MAKES CALL                       │
└──────────────────────────┬──────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────┐
│           TelecomManager (Android System)                │
│   Routes through our app because we're Default Dialer   │
└──────────────────────────┬──────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────┐
│         CallConnectionService (our service)              │
│   onCreateOutgoingConnection / onCreateIncomingConnection│
└──────────────────────────┬──────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────┐
│           TeleCallConnection (our connection)            │
│   onStateChanged(STATE_ACTIVE) → triggers recording      │
└──────────────────────────┬──────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────┐
│        CallRecordingService (foreground service)         │
│   MediaRecorder(VOICE_COMMUNICATION) → saves .m4a file   │
└──────────────────────────┬──────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────┐
│          Room Database + File Storage                    │
│   CallRecord metadata + /CallRecordings/CALL_*.m4a       │
└─────────────────────────────────────────────────────────┘
```

---

## 📂 Project Structure

```
app/src/main/
├── AndroidManifest.xml              ← All permissions declared here
├── java/com/telecalling/callrecord/
│   ├── service/
│   │   ├── CallConnectionService.java   ← KEY: System-level call access
│   │   ├── TeleCallConnection.java      ← Per-call state callbacks
│   │   ├── CallRecordingService.java    ← MediaRecorder foreground service
│   │   └── CallReceiver.java            ← Broadcast receiver fallback
│   ├── ui/
│   │   ├── MainActivity.java            ← Call log + setup
│   │   ├── InCallActivity.java          ← In-call UI with recording controls
│   │   ├── DialerActivity.java          ← Dialer pad
│   │   ├── PlaybackActivity.java        ← Recording playback
│   │   └── CallDetailActivity.java      ← Call detail + notes
│   ├── model/
│   │   └── CallRecord.java              ← Room entity
│   └── utils/
│       ├── PhoneAccountHelper.java      ← Default dialer setup
│       ├── CallRecordDatabase.java      ← Room database
│       └── CallRecordDao.java           ← Database queries
└── res/
    └── xml/file_paths.xml               ← FileProvider paths
```

---

## ⚙️ Setup Steps (IMPORTANT — must do in order)

### Step 1: Permissions in AndroidManifest ✅
All permissions are already declared. Critical ones:
- `RECORD_AUDIO` — mic access
- `MANAGE_OWN_CALLS` — ConnectionService requirement
- `FOREGROUND_SERVICE_MICROPHONE` — Android 14+ required
- `FOREGROUND_SERVICE_PHONE_CALL` — Android 14+ required

### Step 2: Register as Default Dialer
The app prompts users on first launch via `RoleManager.ROLE_DIALER` (Android 10+).
Without this, ConnectionService calls won't be routed to our app.

```java
// In MainActivity.java — already implemented
RoleManager roleManager = getSystemService(RoleManager.class);
Intent intent = roleManager.createRequestRoleIntent(RoleManager.ROLE_DIALER);
startActivityForResult(intent, REQUEST_ROLE_DIALER);
```

### Step 3: Register PhoneAccount
```java
// In PhoneAccountHelper.java — automatically called after permissions
phoneAccountHelper.registerPhoneAccount();
```

---

## 🎙️ Recording Technical Details

### Audio Source
```java
mediaRecorder.setAudioSource(MediaRecorder.AudioSource.VOICE_COMMUNICATION);
```
- Best for call audio on non-system apps
- Applies echo cancellation & noise suppression
- Works reliably as Default Dialer

### Output Format
- Format: **MPEG-4 (.m4a)**
- Encoder: **AAC**
- Bitrate: **128 kbps**
- Sample Rate: **44,100 Hz**
- Storage: `Android/data/com.telecalling.callrecord/files/CallRecordings/`

### File Naming
```
CALL_+919876543210_20260312_143022.m4a
       └─ number ─┘ └── timestamp ──┘
```

---

## ⚠️ Android 15 Limitations & Notes

| Feature | Status | Notes |
|---------|--------|-------|
| Auto-record when call connects | ✅ Works | Via ConnectionService STATE_ACTIVE |
| Record mic audio (your side) | ✅ Works | VOICE_COMMUNICATION source |
| Record both sides of call | ⚠️ Partial | Requires OEM cooperation |
| Background recording | ✅ Works | Foreground service with notification |
| Pause on hold | ✅ Works | MediaRecorder.pause() (API 24+) |
| Store recordings | ✅ Works | App-specific storage, no permission |

### Why not both sides?
Android's `AudioSource.VOICE_CALL` (which captures call audio from both sides)
requires `CAPTURE_AUDIO_OUTPUT` — a **system-level permission** only granted to
system apps or apps pre-installed by OEM. As a user-installed default dialer,
we capture the microphone side clearly using `VOICE_COMMUNICATION`.

---

## 🚀 Installation

1. Clone project and open in Android Studio
2. Sync Gradle
3. Run on device (NOT emulator — needs real phone calls)
4. Grant all permissions when prompted
5. Set CallTrack as default dialer when prompted
6. Make a test call — recording starts automatically!

---

## 📱 Minimum Requirements
- Android 10 (API 29) minimum
- Android 15 (API 35) fully tested
- Real device required (emulator has no telephony)
- Must grant: Microphone, Phone, Contacts, Notifications

---

## 🔧 Key Dependencies
```gradle
androidx.room:room-runtime:2.6.1       // Call record storage
androidx.lifecycle:lifecycle-viewmodel  // MVVM pattern
androidx.work:work-runtime             // Background upload
```
