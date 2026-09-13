# Testing and release gates

Candidate: 0.5.27 (40). Automated device tests use isolated Android emulators and `android/qa_receiver.py`; it renders synthetic screen data and never posts desktop input to the actual Mac. A dedicated purpose=qa route is used for public Cloudflare transport checks. No production pairing credentials are included in this package.

## Build validation

- compileSdk / targetSdk 36; minSdk 26. Same version source as the Mac receiver.
- The current candidate has 43 unit tests per distribution, 86 total (see the generated `verification.json` for exact suites/counts); both release lint tasks must have no errors. Warnings are retained in the reports, not suppressed to imply a clean audit.
- Official bundletool validates the AAB. AAB and APK signatures verified; direct/Play certificates must match. Compose now includes AndroidX graphics-path for four ABIs. The gate checks ELF LOAD alignment, 16 KB ZIP alignment in both APKs and AAB PAGE_ALIGNMENT_16K. RELRO offsets are recorded in native-alignment.json; the 0.5.27 ARM64 runtime check uses an actual 16 KB Android 16 kernel with both backcompat switches disabled, explicitly loads the packaged graphics library and runs the compatibility UI scenario. Other ABIs have static alignment checks only.
- Manifest gate rejects APK installer / direct battery exemption permissions in Play, exported updater provider, cleartext traffic, debugging or backup enabled.
- Public privacy and support routes must return HTTP 200 without account authentication. Some automated HTTP clients can trigger hosting anti-bot rules; anonymous normal browser/curl access was checked.

## Runtime scenarios and evidence scope

0.5.27 unifies Remote, Sessions, Screen, the shared composer and onboarding with the Material 3 settings design. Main navigation uses Compose with the AndroidX back dispatcher. Unchanged connection status notifications are no longer reposted on every UI observation; runtime language checks assert that 20 repeated observations keep the notification timestamp and that attention messages still update with locale changes. Current candidate evidence is limited to the exact APK hash and API/window/language cases recorded in verification.json; device manufacturer ROMs still need physical testing.

0.5.26 introduces the Compose / Material 3 settings screen. Current settings-mode evidence covers seven-language changes through the real new picker, draft/connection retention, saved and cancelled route changes, Activity recreation, privacy deletion cancellation and offline edits. Adaptive settings screenshots are retained separately. Earlier full-review scenarios below are historical baselines.

0.5.24 adds review-mode checks for stale Mac confirmation dialogs and pending-to-draft ownership, alongside boundary, outbox, background and composer checks. Mac full regression is 248 tests; editor bridge uses a real loopback HTTP test with fixture terminals. Dependency audit covers the installed Mac runtime, Android resolved Maven artifacts, relay and website npm lockfiles. See CODE-REVIEW.md for findings and limits. Failure checks use injected storage boundaries and the real Android Keystore; they do not simulate every physical disk or sudden-power-loss failure.

0.5.22 added Mac-side pairing revocation and terminal permission choices. Compact All windows actions and background conversation submission were introduced in 0.5.21. Current-build evidence is listed in verification.json; prior scenario descriptions remain a baseline, not new test claims.

0.5.20 added Japanese, Spanish, Italian and German to the existing English and Chinese interfaces, with seven-language real-picker, notification, draft/connection retention and first-use guide checks in `localization/`. Native button layout checks cover phone, large-font and unfolded sizes. The UI audit introduced in 0.5.18 and compatibility matrix introduced in 0.5.17 remain available; current-version reruns are exactly those listed in `verification.json`. The scenarios below describe the broader 0.5.16 release baseline and are not all claimed as newly rerun. Manufacturer ROM behavior still needs real devices.

Results are kept in the candidate's `runtime/` folder and summarized in `verification.json`.

- `store`: Play-only update intent, blocked Mac update APIs (also checked in relay scenario), normal battery settings, privacy link, cancel data deletion, open-source notices, actual UI screenshots.
- `setup`: one-time guide / existing-user migration, Android 16 Back, landscape, reopen, three connection modes and encrypted persistence.
- default screen: pinch, zoom reset, scroll mode, fullscreen hold-to-talk / send / cancel, return to the screen tab without reconnecting.
- `boundary`: malformed text followed by successful input/ESC, encrypted atomic staging and failure rollback, unrecoverable load does not overwrite ciphertext, no editable duplicate after receipt-save failure, Mac/session PendingIntent isolation and removed-Mac notification rejection.
- `background`: leave activity, retain connection, title/latest-message attention notification, outage detection, automatic recovery, explicit disconnect persists.
- `outbox`: lost receipt / retry, only one execution, delayed transcript reconciliation and reopened pending cleanup.
- `store-demo`: recorded user-visible Connect, background foreground-service notification, attention message and Disconnect.
- public relay: deployed WSS using system trust and DNS, LAN fallback/preference, wrong inner Mac certificate rejection, audio/screen transport, retry deduplication, no forbidden fallback in explicit local/Tailscale modes.

## Still requires real distribution / users

1. Upload the signed AAB to internal testing with the existing App Signing key imported. Install the Google-generated app via the Play opt-in link and review its automated pre-launch report. Exercise update from the currently installed direct release and confirm pairing, history, biometric credential and pending queue retention.
2. Repeat on the user's actual Honor Magic V6, folded/unfolded, using microphone, manufacturer biometric prompt, Android Back gestures and the manufacturer's application-start/power controls. Emulator results do not certify that hardware or its safety scanner.
3. Use mobile data with Wi-Fi disabled and a Mac on a different network; verify public relay, switch Wi-Fi/cellular and VPN configuration, overnight reconnect and notifications. Public WSS synthetic-receiver testing does not replace carrier-specific testing or a real Mac workload.
4. New personal Play accounts require at least 12 real testers continuously opted into closed testing for 14 days before requesting production access. Keep actual dates, activity and feedback; do not fabricate them or count emulator instances as testers.
5. Test a clean Mac installation, including unsigned/unnotarized Gatekeeper flow. Obtain Developer ID signing/notarization for the intended polished 1.0 experience. Confirm microphone provider setup is comprehensible for a new user.
6. Confirm public publisher identity/support email, app-signing recovery backup, intended audience/regions, final Data Safety form and content rating. Check the chosen product name before a paid launch; no legal trademark clearance is claimed.

## Closed-test log template

Track participant identifier privately, opt-in date/time, Android model/version, receiver version, connection mode, scenario, result and issue link. Do not place tester emails or pairing QR codes in public release assets.

Suggested sequence: initial setup and permissions → ordinary remote input → session/voice/queue → screen and fold → background/day-to-day use → network interruption and reconnect → upgrades → feedback and fixes. This is a usage plan, not evidence that 14 days have elapsed.

Official requirements: [target API](https://support.google.com/googleplay/android-developer/answer/11926878), [new personal account testing](https://support.google.com/googleplay/android-developer/answer/14151465), [foreground service review](https://support.google.com/googleplay/android-developer/answer/13392821).

## 招募测试者

可以通过朋友、同事、目标用户社群或付费众测招募，不需要线下见面。Remote Vibe 的完整体验需要 Android 手机和 Mac；当前独立 DMG 为 Apple Silicon / macOS 14+，应在招募时说明。

建议招募 15–20 人，为中途退出留余量，尽量覆盖三星、Pixel、小米/Redmi、OPPO/一加、vivo 和荣耀。参与者需要能够使用 Google Play，通过该应用的封闭测试链接加入并安装；单独发 APK 不等于加入 Play 的封闭测试。连续加入至少 14 天的人数要求见 Google 页面；使用频率应反映实际需求，不能把“每天启动一次”当成保证通过的规则。

[Google 的招募与测试说明](https://support.google.com/googleplay/android-developer/answer/14151465)明确允许个人关系网和线上社群。[Testlio](https://www.testlio.com/)提供商业众测；委托前需确认是否能满足 Android + Mac 双设备、Play 封闭测试参与时长、真实反馈及具体机型要求。尚未向任何服务询价、付款或招募，不承诺审核结果。

## Manual ordering (0.5.22)

Long-press a session card and drag vertically, including the viewport edges. Dropping saves; Back, app backgrounding and dropping outside the list cancel. Check both active and history filters, search results, new-message refresh, reconnect, app recreation and another Mac. Ordering is local to the phone and applies to the loaded page; hidden search/page slots are retained. TalkBack offers move-up/down actions.

## 0.5.26 scoped UI update

Only the settings button/card hierarchy and shared release version changed. The current APK ran header checks on API 36 phone and compatibility checks on the fold-open profile, plus both channels' 86 unit tests and release lint. The 0.5.24 full-review scenarios above remain historical baseline, not rerun results for this APK.

Current-feature checks cover live model/effort updates while browsing history, quota labels, all four arrow events in normal/fullscreen views, receipt-safe manual resend and durable local deletion. See the exact build evidence; previous artifacts are not claimed as current tests.
