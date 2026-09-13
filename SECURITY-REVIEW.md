# 0.5.21 security review · 2026-09-13

This is a scoped developer review, not an independent penetration test or store approval.

## Changes in this release

- Submitted conversation messages target the selected CLI without activating its app/tab or using the shared clipboard. The receiver checks current UID, TTY, executable name, process start identity and foreground process group, rechecks lock state, and checks the connection lease/deadline before dispatch. Terminal also checks the target tab's process list. Stale or unsupported targets fail without falling back to the active window.
- Multiline messages use bracketed paste and one submission. Embedded Escape, Delete and other disallowed control characters cannot break out of the message. Prompts go through the subprocess standard input, not the process argument list. Lost acknowledgements retain the existing uncertain receipt instead of automatically typing twice.
- Existing Mac pairing credential files regain mode 0600 on startup. Pairing identity remains unchanged.
- Responses from the local editor bridge are capped at 1 MiB.
- A stale or incomplete cached phone update no longer hides a newer valid update bundled with the Mac app. Downloads still require pairing and verify the advertised digest; Android additionally checks package, version and signing certificate.

## Checked boundaries

126 Mac/receiver/transport regression tests passed, including invalid bearer credentials, browser-origin rejection, bounded input, expired/replaced sessions, lock-screen separation, delivery idempotency and local real-workerd relay authentication/stream isolation. Expected TLS rejection diagnostics in these negative tests are not test failures. Editor bridge contract tests passed.

An isolated native Terminal recorder received approximately 24 KB of Unicode, multiline text and shell-looking literal characters through the production submission method. Foreground application, selected tabs and clipboard change count stayed unchanged. Exactly one final Return arrived, and input after recorder exit was rejected. No input was sent to a real user CLI during this test. iTerm uses its documented exact-session `write text … newline NO` API; an installed iTerm runtime was not exercised in this release. Reference: https://iterm2.com/documentation-scripting.html

Reviewed existing protections: QR certificate pinning and inner TLS through relay, separate relay roles/tickets, Android Keystore storage, fresh strong-biometric unlock binding, no password in the durable outbox, private notifications, disabled application backup, and package-signature checks. Runtime evidence attached to the release identifies what was rerun on this exact build; review alone is not a new runtime test.

## Limits and follow-up

- Pairing is a long-lived shared control credential. Removing a phone entry does not revoke copies elsewhere. A dedicated Mac-side revoke/rotate flow with per-device credentials is still desirable. Currently a Mac-side pairing reset and separate relay-route revocation are required after credential loss; never publish pairing QR codes.
- Terminal's scripting API cannot atomically bind input to a specific running CLI. Identity/foreground checks and the immediate tab process check narrow, but cannot eliminate, a process-exit race. This does not replace OS user isolation or protect a compromised same-user Mac account.
- Normal remote keyboard actions, terminal option keys and model-menu navigation may intentionally activate a target. This release changes submitted conversation messages; explicit “view on Mac” still activates the selected session.
- Mac downloads remain ad-hoc signed and unnotarized. Developer ID signing/notarization and independent security review remain release work before a wider stable launch.
- Physical-device biometric/OEM background behavior, Play delivery and external network conditions require the separately documented device/store tests.
