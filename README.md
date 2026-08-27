# MYRA — Android AI Voice Assistant

Complete Kotlin/Android Studio project implementing the MYRA master build prompt.

## How to open

1. Unzip and open the `MyraApp` folder in Android Studio (Hedgehog+ recommended).
2. Let Gradle sync (it will pull OkHttp, coroutines, Material, etc. from the dependencies list).
3. Add a `local.properties` with your `sdk.dir` if Android Studio doesn't create it automatically.
4. Run on a device/emulator with **API 26+**.

## First run

1. Open the app → grant the permission prompts (mic, contacts, call, SMS, phone state, camera, overlay).
2. Go to **Settings** (gear icon) → paste your **Gemini API key**, set your name, pick model/voice/personality → **Save**.
3. Turn on the **Accessibility Service** (Settings → tap the accessibility status row) so app open/close voice commands work.
4. Restart the app — MYRA connects over the Gemini Live WebSocket and greets you in native audio.

## What's implemented

- `ai/GeminiLiveClient.kt` — WebSocket client for `BidiGenerateContent` (setup message, audio in/out, transcripts, 9-min session renewal, 8s keepalive, 3s auto-reconnect).
- `ai/AudioEngine.kt` — `AudioRecord` (16kHz mic) + `AudioTrack` (24kHz speaker) with a playback queue, RMS amplitude, and echo suppression while MYRA is speaking.
- `ai/CommandParser.kt` — Hinglish + English command parsing (open/close app, call, SMS, WhatsApp, prime contacts, volume, torch, WiFi/Bluetooth).
- `viewmodel/MainViewModel.kt` — executes parsed commands: launching apps (with a hardcoded package map + installed-app fallback), placing calls, sending SMS/WhatsApp, prime-contact storage/migration, torch/volume, call accept/reject.
- `service/AccessibilityHelperService.kt`, `CallMonitorService.kt`, `MyraOverlayService.kt`, `PowerButtonReceiver.kt`, `BootReceiver.kt` — background app-control, incoming-call detection + announcement flow, the draggable floating orb (double power-press), and boot auto-start.
- `ui/main/OrbAnimationView.kt` + `WaveformView` — custom Canvas orb (idle/listening/speaking/thinking/active) and a 20-bar amplitude-reactive waveform.
- `ui/main/MainActivity.kt`, `ui/settings/SettingsActivity.kt` — full lifecycle wiring, chat transcript, settings screen with prime-contacts RecyclerView.
- Dark red/purple themed layouts, drawables and vector icons matching the spec's UI theme.

## Notes / things to double-check before shipping

- The Gemini Live WebSocket schema (`setup`, `realtime_input`, `client_content`, `serverContent`) can change as the API evolves — verify field names against Google's current docs before a production build.
- `com.google.ai.client.generativeai:generativeai:0.9.0` is included per the spec's dependency list but isn't directly used by the WebSocket client (which talks to the API raw via OkHttp) — keep it only if you plan to also use the SDK's REST helpers.
- Runtime permission requests are batched in `MainActivity.checkPermissions()`; you may want a nicer onboarding flow before shipping.
- `SYSTEM_ALERT_WINDOW` (overlay) still needs the user to flip a system settings toggle on Android 8+; the app opens that screen automatically if not granted.
- Launcher icon is a minimal placeholder vector — swap in real artwork before publishing.
