# Mac 签名、公证与分发

## 所有者首次设置（2026-09-13 核对）

当前这台 Mac 的 `security find-identity -v -p codesigning` 返回 **0 个有效身份**。`notarytool` 和 `stapler` 已安装；公证流程已编写并用模拟结果测试，但没有完成真实 Apple 提交。

1. 加入 [Apple Developer Program](https://developer.apple.com/programs/enroll/)，年费 99 美元或当地货币价格。官网 DMG 分发使用 Developer ID 和公证，不需要先上架 Mac App Store。
2. 在“钥匙串访问 → 证书助理 → 从证书颁发机构请求证书”中填写自己的邮箱和密钥名称，选择存储到磁盘，生成 CSR。私钥留在本机钥匙串。[Apple CSR 指引](https://developer.apple.com/help/account/certificates/create-a-certificate-signing-request/)
3. 由账号持有人打开 [Certificates, Identifiers & Profiles](https://developer.apple.com/account/resources/certificates/list)，创建 **Developer ID Application**，上传 CSR，下载 `.cer`，双击导入生成 CSR 的这台 Mac。“我的证书”中必须能展开看到配套私钥。当前分发 app/DMG，不需要 Developer ID Installer；后者用于 `.pkg`。[Apple 证书指引](https://developer.apple.com/help/account/certificates/create-developer-id-certificates/)
4. 在自己的终端运行下面的 `store-credentials`，按交互提示设置公证凭据并保存在钥匙串。不要把 Apple 密码、应用专用密码或私钥发到聊天里。
5. 复制本机实际证书的完整名称到 `REMOTE_VIBE_SIGN_IDENTITY`，运行下方公证构建命令。成功后查看 `dist/mac-distribution.json` 的 app、DMG 票据和最终哈希。
6. 从下载网址在第二台 Mac 下载最终 DMG，保留系统隔离标记，验证离线首次打开与辅助功能、录屏、自动化授权。公证不代替这些系统授权，也不等于 App Store 审核。

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
