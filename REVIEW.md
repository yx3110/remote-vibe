# App review and foreground-service declaration draft

App: Remote Vibe · package dev.switchpad.remote · candidate 0.5.27 (40).

## Reviewer access

There is no Remote Vibe account, subscription, paywall, invitation or geographic gate in the Android app. Its core purpose requires an external Mac; an Android device alone cannot perform Mac control. The interface supports English, Simplified Chinese, Traditional Chinese, Japanese, Spanish, Italian and German. It follows the system language by default; Settings → Language and the first-use guide language button allow an independent choice. User conversation content remains in its original language. Do not enter any developer personal pairing credentials into Play Console.

1. On an Apple Silicon Mac running macOS 14+, download the receiver from https://remote-vibe.eclipse-myworld.chatgpt.site/. Drag the app into Applications, open it, and follow Mac setup. The candidate receiver is available in the candidate GitHub release. Current Mac builds are ad-hoc signed, not Apple notarized; this is disclosed to users.
2. Grant Mac Accessibility for input, and Screen Recording for viewing. The app provides settings links. Screen access can be denied while other functions remain usable.
3. Put Mac and Android on the same reachable Wi-Fi. Open the Mac pairing page/QR code. On Android use Add Mac (添加 Mac), scan the code, and connect. The QR contains sensitive pairing credentials; do not publish it in review videos or screenshots.
4. Remote (遥控) provides pointer, keyboard and window actions. Screen (画面) views the Mac screen with zoom and fullscreen. Sessions (会话) lists compatible local CLI sessions when those tools are installed on the Mac; the app itself does not provide an AI account or model entitlement.
5. Hold “Hold to talk” (按住说话) for microphone recording; release to transcribe on the paired Mac. This requires TypeWhisper on the Mac, configured with the user's chosen transcription backend. Use ordinary text input when unavailable.
6. To reproduce session notifications, start a Codex or Claude Code session on the test Mac and allow its local session/hooks integration during Mac setup. Keep Android connected, send a request, and put the Android UI in the background. When the session needs input, a notification contains its conversation title and latest response.
7. Expand the connected Mac header and choose Disconnect (断开). The app cancels the persistent connection service and automatic reconnect intent. Returning to the app does not silently override this choice.
8. Settings → Privacy and data management (隐私与数据管理) provides the public privacy page, sent history and confirmed clearing of this phone's app data. No remote account deletion is necessary because there is no Remote Vibe user account.

Public relay access is optional and separately provisioned per Mac. It is not required for local testing. Tailscale is also optional and separately installed/configured. Do not advertise a demonstration fixture as a live Mac or as evidence of real-device long-duration testing.

## FGS declaration: connectedDevice

Permission: android.permission.FOREGROUND_SERVICE_CONNECTED_DEVICE.
Type: connectedDevice. Use case: Continuous Data Transfer to an External Device.

Suggested description:
“After the user connects to a paired Mac, Remote Vibe keeps an authenticated, encrypted network connection to that external device for remote input and session-status notifications. The persistent foreground-service notification identifies the connected Mac and provides a Disconnect action. The user can leave the activity while continuing to receive session attention alerts. Disconnect explicitly stops the connection and its reconnect behavior.”

Impact if interrupted/deferred:
“Closing or deferring the connection would interrupt user-requested live control and delay notifications from the paired computer. Network loss causes retries only while the user's connection intent remains enabled. Android force-stop and device power restrictions can still stop the service.”

Demonstration video: `assets/foreground-service-demo.mp4`. Public asset URL: https://github.com/yx3110/remote-vibe/releases/download/v0.5.16/foreground-service-demo.mp4 . This video was recorded on the previous 0.5.16 candidate and demonstrates the unchanged service flow; it is not a 0.5.24 test recording. The video shows the actual app UI and service: Connect → view a session → leave activity → persistent connection and attention notification → return and Disconnect. The external data source is an isolated synthetic Mac receiver so no private screen or conversation is recorded. Its notification and networking execute through the production service. A public video URL is listed with the candidate release; include it in the Console declaration after verifying reviewer access.

No full-screen-intent, background location, VPN service, accessibility service, SMS or call permissions are declared. Foreground microphone recording is user-held and stops on release/cancel; no microphone foreground service is used. The screen shown is the paired Mac screen, not background capture of the phone screen.

[Google foreground service declaration requirements](https://support.google.com/googleplay/android-developer/answer/13392821)

## Other app-content answers to verify in Console

- Contains ads: No (no ad SDK or ad inventory).
- App access: Explain the external Mac requirement above; do not claim all features work without hardware.
- Category proposal: Tools. Target audience proposal: adults using their own Mac / development tools; publisher must confirm actual intended audience before submission.
- Content rating: Complete the questionnaire based on remote display of user-owned content, CLI conversations and control features; do not invent a rating before IARC returns it.
- No financial, medical, gambling, dating, social-network publishing or app-provided generative AI service. The app can remotely interact with the user's separately installed CLI tools.
- App signing: retain the existing certificate; no claims of independent security audit, ISO certification or guaranteed manufacturer scan clearance.
