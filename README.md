# Voice Summary

Kotlin • Jetpack Compose • MVVM • Hilt • Room  
Min SDK 24 • Target SDK 34

## Overview
Voice Summary is a foreground voice recorder that saves audio in **30-second chunks** (with a small overlap for continuity), survives common interruptions (phone calls, audio focus changes, low storage), and produces a **structured meeting summary** from the recorded session. The app stores data locally using Room and presents a simple, focused UI in Compose.

## Main Features

### Recording
- Foreground service with a persistent notification (shows timer and state).
- Real microphone capture using `AudioRecord` (16 kHz, mono, PCM 16-bit).
- **Chunking:** 30-second files with ~2-second overlap handling.
- **Interruptions handled:**
  - Phone calls → pause while ringing/in-call, resume when idle.
  - Audio focus loss (e.g., other apps playing audio) → pause; resume on focus gain.
- **Low storage:** check before starting; stop gracefully if space runs out.
- **Silence detection:** warn after ~10 seconds of near-silence.
- Files saved under the app’s external files directory: `…/Android/data/<package>/files/chunks`.

### Transcript & Summary
- Each chunk is considered for transcription in order.
- Stored in Room as the single source of truth (session → transcripts → summary).
- Summary screen shows four sections:
  - **Title**
  - **Summary**
  - **Action Items**
  - **Key Points**
- Loading and error states included; safe to reopen the app and continue.

## Architecture

```
UI (Compose)
   ↓
ViewModel (MVVM)
   ↓
Repository
   ↓
Room  •  Foreground Service  •  (WorkManager entry points available)
```

- **Compose**: Dashboard, Recording, Summary screens.
- **Hilt**: dependency injection for repository and database.
- **Room**: `Session`, `Transcript`, `Summary` entities; DAO exposes Flows.
- **Service**: `RecordingService` owns the audio pipeline and notifications.

## Folder Layout

```
app/src/main/java/com/example/voicesummaryapp/
  data/        (Room entities, DAO, database)
  di/          (Hilt module)
  repository/  (app data logic)
  service/     (RecordingService)
  ui/          (NavGraph, screens, theme)
  MainActivity.kt
  VoiceSummaryApp.kt
```

## Permissions
- `RECORD_AUDIO` — microphone capture  
- `FOREGROUND_SERVICE` / `FOREGROUND_SERVICE_MICROPHONE` — foreground recording  
- `POST_NOTIFICATIONS` — show the recording notification (Android 13+)  
- `READ_PHONE_STATE` — pause/resume on call state changes  
- Legacy storage permissions are declared for older devices; current devices write to the app’s scoped external files directory.

## How to Build & Run
1. Open the project in **Android Studio** and let Gradle sync.
2. Run on a device/emulator and grant the requested permissions.
3. To produce an APK: **Build → Build APK(s)**  
   Output: `app/build/outputs/apk/debug/app-debug.apk`

## Demo Guide (for your screen recording)
1. **Dashboard** → tap **+** to create a session (navigates to Recording).
2. **Recording**  
   - Watch the timer update in the app and in the notification.  
   - Demonstrate an interruption (e.g., start audio in another app to simulate focus loss); the state changes to “Paused – Audio focus lost”, then resumes on return.  
   - Tap **Stop & Generate Summary**.
3. **Summary**  
   - “Generating summary…” state appears, then the four sections render.
4. Return to **Dashboard** to show the session in the list.

## Notes & Trade-offs
- The 2-second overlap is handled at chunk boundaries; adding a small in-memory buffer can make it byte-accurate if needed.
- Headset/Bluetooth connect/disconnect does not show a dedicated “source changed” message yet; recording continues without user action.
- Process-death auto-resume can be extended with a small WorkManager task on boot. The codebase is ready for that addition.

## Submission Checklist
- Foreground recording with visible status and timer  
- 30-second chunks with overlap handling  
- Phone call and audio focus pause/resume  
- Low storage stop + message  
- Silence warning after ~10s  
- Room as single source of truth  
- Summary screen with Title, Summary, Action Items, Key Points  
- Compose UI, MVVM, Hilt, Coroutines/Flow  
- Buildable APK (debug) and short demo video
