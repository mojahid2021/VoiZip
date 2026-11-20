# Changelog

All notable changes to VoiZip. Each version is also published as a [GitHub Release](https://github.com/mojahid2021/VoiZip/releases), which the website reads.

## 2.0.0 - 2026-11-20

### Highlights
- **Native PJSIP 2.17 Architecture**: Complete transition from Linphone SDK to lightweight native PJSIP 2.17 (`:pjsua2`) with OpenSSL 3.5.9 and Opus 1.5.2, reducing binary size by ~70% (native code down from ~41 MB to 13.4 MB).
- **Unified Call Screen**: Single, modern call screen (`CallActivity`) handling incoming, outgoing, active, holding, and conference calls for both SIM lines and standalone VoIP accounts.
- **Smart Calling Banner**: Context-aware interactive banner on the main screen that prompts users to configure a SIP account or enable SIM calling when setup is incomplete, with one-tap dismissal.
- **UDP by Default**: The Add SIP Account login page now defaults to UDP transport instead of TLS.
- **Zero-Leak Memory & Battery Optimizations**: Eliminated ViewBinding fragment leaks, removed indefinite background wake locks, and optimized contact synchronization.

### Calling & Dialer
- Unified in-call screen across SIM and standalone SIP VoIP calls, removing obsolete video, meetings, and legacy fragmentation.
- Redesigned dialpad and numpad with modern circular buttons, haptic feedback, and T9 search matching across names and phone numbers.
- Expanded international dial plan support to handle multi-country calling codes (`+1`, `+44`, `+880`, `+91`, `+33`, `+49`) when matching contacts.
- Strict default dialer gating: SIM calls and candidate suggestions require the default dialer role, keeping SIM features disabled gracefully when VoiZip is used purely as a VoIP client.
- Native WAV call recording and in-app audio playback via Android's native `MediaPlayer`.

### Onboarding & Permissions
- Card-based onboarding permissions screen with granular requests for Phone, Contacts, Microphone, Notifications, and Default Dialer.
- "I'll do this later" skip option that cleanly exits onboarding without trapping users on subsequent app launches.
- Automatic data refresh: Call logs, native contacts, and SIM accounts reload immediately upon returning from permissions.
- Permissions Diagnostics screen in Settings with status indicators for all telephony and system capabilities.

### Performance & Engine
- Off-thread contact synchronization: parsing and address interpretation for thousands of contacts runs on `Dispatchers.IO`, keeping the Core thread responsive.
- Fragment ViewBinding cleanup: `DialpadFragment`, `ContactsListFragment`, `HistoryListFragment`, `PermissionsFragment`, and `ThirdPartySipAccountLoginFragment` null out bindings in `onDestroyView()`.
- Removed `android:largeHeap="true"` from the manifest; the app now operates comfortably within standard Android heap constraints.
- Eliminated indefinite `PARTIAL_WAKE_LOCK` in background keep-alive service and removed startup wake locks.
- Replaced 1-second busy-polling loop during active calls with event-driven `onLastCallEnded` listener.
- Thread-safe dispatching of call log updates to Core thread, eliminating data races on history lists.
- Early startup buffering in `CoreContext` to prevent dropped tasks.
- In-place row view recycling in the side navigation drawer.
- Pruning of disconnected zombie calls and callbacks in `SimCallRepository`.
- Recycled `TextWatcher` attachments in data-bound edit text adapters.

### Under the hood: moving from Linphone to PJSIP (docs/PJSUA2_MIGRATION.md)
- P0: decided to drop ZRTP (SRTP stays), video calls, server-side conferences (merging calls on the phone stays) and Linphone push; call recordings will be saved as WAV
- P0-S1..S3: `tools/pjsip/build.sh` builds PJSIP 2.17 with OpenSSL 3.5.9 and Opus 1.5.2 for arm64 and armv7; new `:pjsua2` module
- P0-S4: throwaway `:pjsua2-spike` test app for real-phone call tests
- P0-S6: measured native code 13.4 MB on pjsua2 against about 41 MB on Linphone
- P1-11: removed ZRTP code, layouts, drawables and strings
- P1-05a: removed server-side conference and meeting code; merging always creates a local conference
- P1-11c: video switched off in the engine; camera permission and video entry points removed
- P1-11b: removed contact device-trust screens, dialogs and models
- P1-05b: removed the hidden group-call creation flow from the new-call screen
- P1-00: app-owned logger and Android version checks; files depending on Linphone down from 130 to 72
- P1-01a: app-owned SIP address type; call routing and recording rules no longer depend on Linphone, with new unit tests
- P1-01b: call history models use app-owned address, direction and status types; history screens no longer depend on Linphone types
- P1-01c: new `VoipEngine` interface with a Linphone implementation; the call route chooser and side-menu line cards use it
- P1-02a: the engine reports calls, network, registration and voicemail events; the app's top alert bar uses them instead of Linphone
- P1-09a: notification hang-up and speaker buttons, bug-report diagnostics and the Telecom services go through the engine
- P1-09b: Android Telecom integration (answer, hang up, hold, mute, audio route from the system and the car) goes through the engine
- P1-09c: call, missed-call and account-error notifications go through the engine
- P1-07a: app-owned contact model; notifications, Telecom, alerts, recording rules and the line chooser look contacts up without Linphone types
- P1-07b: removed contact presence and trust from avatar models and layouts
- P1-07c-A: the address book is kept in app-owned contacts; contact lookups by number or SIP address use it, with unit tests
- P1-07c-B0: app-owned contact search (name, number, SIP address) with unit tests, to replace Linphone's search
- P1-07c: contacts no longer use Linphone: the address book, contact list, contact page, number pickers, history search and Android Auto favourites run on app-owned contacts and search
- P1-04-pre: call and registration errors are worded from SIP status codes; dial pad, new-call and transfer screens use the engine
- Fixed (P1-07c regressions, never released): dial pad contact suggestions, caller names when the app starts for an incoming call, Android Auto favourites right after launch
- P1-04a: the calls list, its top bar, merging calls and attended transfer go through the engine
- P1-04b: the call screen (answer, hang up, mute, hold, keypad, recording, audio output chooser, transfer, stats, encryption) goes through the engine; its video code is removed
- P1-05c: the merged-calls screen goes through the engine; its video layouts (grid, active speaker, layout menu) are removed
- P1-11d: the call screens' video surfaces are removed; the SIM call screen's SIP actions (keypad, recording, hang up) go through the engine; the call area no longer depends on Linphone
- Fixed (never released): the SIM call screen could miss its SIP call when Linphone printed the address differently from the Telecom handle
- P1-08a: adding a SIP account, the account rows and the side menu's account go through the engine
- P1-08b: account settings (proxies, NAT, voicemail, expiry, prefix, password) and the settings screen's account list go through the engine
- P1-08c: the settings screen's call, network, audio, codec, encryption and developer settings go through the engine's settings
- P1-14: the history list and the help screen (debug logs: clear, share, upload; config view) go through the engine
- P1-13: shared helpers no longer depend on Linphone; voicemail on the keypad and answering from the SIM call screen go through the engine
- P1-02b: no screen reaches Linphone's core directly any more (keypad tones, SIP links, remote provisioning, account backup, SIM caller names, call history and statistics loading go through the engine)
- Plan audit (2026-11-19): checklist brought in line with the code (finished parent tasks ticked, duplicate P1-12 renamed P1-14, P1 exit task P1-15 added); three unused imports and two stale shortcut notes cleaned
- P1-02d: call history (phone call log writing, repair, SIP-only fallback, statistics) works on the engine's own call-log records; its call-type rules are unit-tested
- P1-11e: the last video leftovers in the call screens are gone (hidden camera buttons, video stubs, camera code)
- P1-03: app preferences are read through an app-owned settings store (still the same file, no setting moves or resets); Linphone's start-up moved out of the application class
- P1-02c: the app's central `CoreContext` no longer depends on Linphone: Linphone's listener, call start / answer / end, start-up configuration and Core lifecycle moved unchanged into the engine's Linphone part
- P2-01…05: the app keeps its own copy of the SIP accounts (`accounts.json`, passwords encrypted with an Android Keystore key) and of the SIP call log (`call_log.json`), kept in step with Linphone; this is what the PJSIP engine will use
- P2-06: account backup exports clean JSON without Linphone types; imports both modern JSON and legacy Linphone INI backups; `AccountBackup` is Linphone-free
- P2-08: user preferences moved to SharedPreferences with a one-time idempotent migration from `.linphonerc`; engine-read and provisioned keys routed cleanly via `TieredSettingsStore`; unit-tested
- P3-01…14: new `PjsuaEngine` implementing `VoipEngine` on top of `:pjsua2` (PJSIP 2.17); handles endpoint lifecycle, accounts, calls, audio media, hold, DTMF, transfer, 3-way conference bridging, codecs, NAT/ICE, MWI and call logs; developer settings switch added to toggle between Linphone and PJSIP; unit tests in `PjsuaStateMapperTest`
- P4-01…07: `CoreInCallService` replaced with standard Android foreground service; `InCallRingtonePlayer` plays ringtones via `RingtoneManager`; `PjsuaEngine` implements Android audio routing (API 31+ `setCommunicationDevice`, API 28-30 SCO/speakerphone) and Telecom integration
- P5-01…04: call recordings write native WAV via `AudioMediaRecorder`; recordings player and `RecordingModel` migrated to Android's native `MediaPlayer` and `MediaMetadataRetriever` (plays both WAV and existing Matroska recordings); `Log` rewritten to use Android's `Log` and rotating file writer (Linphone-free); `RemoteProvisioningParser` parses XML provisioning configs into accounts and settings
- P7: **Linphone SDK completely removed from VoiZip!** Deleted `voip/linphone/` directory, Linphone SDK AAR dependency, maven repositories, Linphone build tasks, and legacy Linphone assets. The application is now 100% PJSIP 2.17 (`:pjsua2`) natively.
- Built static `bcg729 1.1.2` (G.729 8 kbps audio codec) into `:pjsua2` for both `arm64-v8a` and `armeabi-v7a`.
- Fixed: outgoing call audio routing and Telecom active state transition (`setActiveCall`), AudioManager `MODE_IN_COMMUNICATION`, ringback tone generation on `OutgoingRinging`, and resilient SIP call lookup in `findSipCall`.
- Fixed: incoming SIP call reception and display — enabled NAT Contact rewriting (`contactRewriteUse = 2`), 15s UDP keep-alive packets (`udpKaIntervalSec`), background activity launch, direct notification posting before foreground service binding, and background service starts via `ContextCompat.startForegroundService`.
- Code formatted with ktlint across the app (layout only)

## 1.0.2 - 2026-11-17

### Fixes
- WhatsApp, Telegram and other calling apps no longer open the VoiZip call screen when you make or receive a call with them

## 1.0.1 - 2026-11-16

### App updates
- Check for and install new versions from inside the app, downloaded from GitHub releases
- Optional check on start-up, with a prompt when a new version is ready
- Updates page with the installed version, download progress and the notes of every release
- Picks the right download for your phone's processor

### Bug reporting
- Report a problem from Help without leaving the app; it opens an issue for the VoiZip team on GitHub
- Choose the type (bug, crash, calls, audio, display or feature request) and how often it happens
- Optional steps to reproduce and expected behavior
- Diagnostics added automatically and shown before sending: app and Android version, device, network,
  SIP line status, default dialer, battery optimization, notifications and missing permissions
- Never sends your phone numbers, contacts or SIP address

## [1.0.0] - 2026-11-16

The first release of VoiZip.

### Calling
- One call screen for SIM and SIP calls
- Choose the SIM or SIP line for each call
- Dial pad with paste, recent number suggestions and haptic feedback
- Video, transfer and call details for SIP calls
- Android Auto support

### Lines and accounts
- Dual SIM and eSIM support
- Add, edit and remove SIP accounts in Settings
- Line status in the side menu: connected, connecting or error
- SIM balance check with your operator's code
- Encrypted backup of your SIP accounts to a file you choose

### History and contacts
- Recents from your phone's call log, with All and Missed
- Contacts from your address book
- Call notes and reminders
- Favourites widget and app shortcuts
- Blocked numbers
- Monthly call statistics by line

### Everything else
- Light and dark themes
- Data saver for SIP calls
- Help with quick answers, plus a way to share diagnostic logs
