# 0.5.23 security review · 2026-09-13

This is a scoped engineering review and automated verification record, not an independent penetration test or a claim that every security defect has been eliminated.

## New protections in 0.5.23

- Notification slots and immutable PendingIntents use a full SHA-256 identity scoped to Mac plus session, rather than Java session hashCode. Same IDs on different Macs and colliding session hashes no longer overwrite destinations. A notification for a removed Mac cannot open that session ID on the currently connected Mac.
- Literal text validation occurs before either queue accepts it: 1–8000 Unicode code points, no unsupported control characters or broken surrogate pairs. Transient events have a serialized byte cap and owned snapshots; a defensive flush guard rejects an oversized head rather than blocking all later input.
- Pending prompt, transcript baseline exclusions and automatic retry metadata commit together. Failed saves roll back the in-memory queue; failed loads prevent overwriting existing unreadable ciphertext. The existing Android Keystore AES-GCM protection remains. Once staging succeeds, the queue owns the message and the editor clears; a later receipt-save failure retries the original ID without retaining a second editable draft.

## Protections retained from 0.5.22

- Session reorder gestures stay inside this app: no ClipData or global drag flag exposes session content to another app. Only session identifiers and order are saved in app-private preferences, separated by Mac identity and active/history scope. App data clearing removes this preference; no reorder request or shell input is sent to the Mac.

- Mac menu bar → Security & permissions → Revoke all pairings. The local UI requires confirmation, atomically rotates the private bearer token, invalidates the active control lease and cancels queued phone input. The Mac identity and certificate pin remain stable. Disk-write failure leaves the previous working pairing intact. Previously displayed QR codes become invalid; phones must scan a new code.
- Authorization is checked again after reading a POST body and inside the locked lease-creation operation. A request authenticated before revocation cannot finish uploading later and acquire a new control lease. Long-running transcription also rechecks the lease before returning its result.
- New terminal sessions use the Mac owner's selected policy. An unconfigured Mac defaults to standard: Codex workspace-write sandbox with on-request approval; Claude inherits its CLI permission settings. Full access is a local explicit choice with confirmation, including a description of command/file access. Existing sessions, models and shell aliases are not edited. Users upgrading from a previous version should review this new preference.
- The distribution build requires a real Developer ID identity before notarizing. It requires Accepted responses, staples both app and DMG, checks their tickets and Gatekeeper, and hashes the final artifact. Ad-hoc builds explicitly record that they are not notarized. Credentials stay in the owner's keychain.

## Evidence for 0.5.23

Android two-channel 84 unit tests and lint pass. API 36 fault injection exercises the real Android Keystore, real PendingIntent identity and isolated QA transport. Current-build boundary, outbox, background and composer results are included in verification.json. Tests model a rejected storage write, not every filesystem failure or power-loss scenario. Android framework identity and persistence behavior are documented in [PendingIntent](https://developer.android.com/reference/android/app/PendingIntent) and [SharedPreferences](https://developer.android.com/reference/android/content/SharedPreferences).

16 targeted Python localization, update and delivery tests passed. The Mac receiver implementation is unchanged apart from the shared release version; the rebuilt bundle and public APK update are checked separately.

## Previous 0.5.22 baseline (not all rerun)

122 targeted Python tests passed, covering bearer rejection, origin restrictions, slow-upload/reset races, stable identity/pin, credential file permissions, failed disk replacement, queued password clearing, terminal policies, notarization failure handling, session controls, background input, receipts, lock-screen input, updates and real local workerd relay authentication/isolation. Negative TLS/auth tests intentionally reject connections.

Native Mac controls were exercised with isolated credentials: cancel leaves pairing unchanged; confirming reset revokes it; full-access cancel leaves standard mode; confirmed preferences persist. The real user's pairing was not reset. Seven-language setup and security windows passed text-bound checks and produced native screenshots. Chinese and German security screenshots were visually reviewed. Android and public-update checks are bound to the current build in verification.json.

The real Developer ID and Apple notarization workflow could not be executed: this Mac has zero usable signing identities. The tests cover failure paths and command orchestration with mocked Apple results. A second Mac must verify downloaded, quarantined and offline first launch after real signing becomes available.

## Boundaries and remaining work

- Pairing still uses one shared long-lived bearer credential per Mac. Reset revokes **all** phones, not a selected individual phone. Per-device identities, expiring invitations and granular permissions remain future improvements. Do not publish pairing QR codes.
- Pairing reset revokes application access, but does not rotate the separate Cloudflare route credentials. A copied route key may still reach the encrypted transport without being able to authenticate to the Mac. If a route key is exposed, the relay operator must separately rotate/revoke that route; consider denial-of-service and reconnect contention in the independent review.
- Revocation cannot undo operations already executed or already dispatched. Memory clearing of queued password fields reduces retention, but does not promise cryptographic erasure of immutable Python objects or system buffers. Durable password storage remains disabled.
- Terminal scripting cannot atomically bind the entire input operation to a running CLI. Process identity and foreground-group checks narrow but do not eliminate a process-exit race. This does not protect a compromised same-user Mac account. iTerm's documented path was not tested against a live installed iTerm runtime.
- Submitted conversation text uses the background route introduced in 0.5.21. Ordinary keyboard controls and model-menu navigation may activate the target; explicit view-on-Mac still activates it.
- Public QR authorization grants extensive Mac control. Screen, terminal and password capabilities need focused independent review. Physical device biometrics/OEM background behavior, Play-delivered installation, network changes and multi-user abuse testing remain open gates in TESTING.md.

Codex flags were checked against the installed CLI and [official CLI reference](https://learn.chatgpt.com/docs/developer-commands?surface=cli). Mac distribution references and the exact command are in install/mac/DISTRIBUTION.md.
