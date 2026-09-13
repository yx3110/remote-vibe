# Mac distribution

The default build is an ad-hoc signed test DMG. It is **not** an Apple-notarized release.

For public distribution, the account owner must enroll in Apple Developer and install a **Developer ID Application** certificate and its private key in the Mac keychain. Create a notarytool keychain profile interactively; do not put passwords or private keys in this repository, command history or a release asset.

```sh
xcrun notarytool store-credentials remote-vibe-release
export REMOTE_VIBE_SIGN_IDENTITY='Developer ID Application: YOUR LEGAL NAME (TEAMID)'
.venv/bin/python install/mac/build.py --notarize --keychain-profile remote-vibe-release
```

The identity is public certificate metadata, not a password. The build checks that it exists before rebuilding. PyInstaller signs with hardened runtime; the only extra entitlement is Apple Events automation. The build submits an app ZIP, requires Apple's `Accepted` result, staples and validates the app ticket, and checks Gatekeeper. It then builds and signs the DMG, submits it, staples and validates its ticket, and checks Gatekeeper again. The final SHA-256 is calculated **after** stapling. Failure aborts the build. `dist/mac-distribution.json` records the exact final DMG digest and ticket status.

A missing identity, missing profile, rejected or unfinished notarization must never be described as a completed release. Real signed and notarized execution has not yet been tested on this project: the current development Mac has no Developer ID identity. Once credentials are supplied, verify a quarantined download and offline first launch on a second Mac, including Accessibility, Screen Recording and Automation prompts.

Apple references: [Notarizing macOS software](https://developer.apple.com/documentation/security/notarizing-macos-software-before-distribution), [Resolving notarization issues](https://developer.apple.com/documentation/security/resolving-common-notarization-issues). PyInstaller documents [code signing and hardened runtime](https://pyinstaller.org/en/stable/feature-notes.html#macos-binary-code-signing).
