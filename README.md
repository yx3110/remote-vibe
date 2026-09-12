# Remote Vibe

用手机遥控 Mac、查看实时画面、语音输入，并浏览和操作 Codex / Claude Code 会话。

## 下载

- [Mac · Apple 芯片（M 系列）](https://github.com/yx3110/remote-vibe/releases/latest/download/Remote-Vibe-Mac-Apple-Silicon.dmg) · macOS 14 或更新版本
- [Android APK](https://github.com/yx3110/remote-vibe/releases/latest/download/Remote-Vibe.apk) · Android 8 或更新版本
- [最新版本、安装说明和 SHA-256 校验文件](https://github.com/yx3110/remote-vibe/releases/latest)

当前是 **0.5.15 测试版**。Mac 版尚未完成 Apple Developer ID 签名与公证，也未上架 Mac App Store；Android 尚未上架 Google Play。Intel Mac 和 iOS 版本暂未提供。

## 首次连接

1. 在 Mac 下载 DMG，打开后将 Remote Vibe 拖入 Applications，再从“应用程序”启动。不需要 Python、Homebrew 或终端命令。
2. 如果 macOS 阻止打开，在“系统设置 → 隐私与安全性”找到本次拦截，确认下载来源后点“仍要打开”。
3. 在 App 设置窗口开启辅助功能、屏幕录制权限。需要登录后自动启动时勾选“登录 Mac 时启动”。
4. 手机安装 Android APK，与 Mac 接入同一 Wi-Fi；在 Mac 点“显示配对二维码”，在手机点“添加 Mac”扫码。
5. 按住说话需另装 TypeWhisper，下载语音模型并开启 API Server；App 设置提供下载入口。手机键盘自带的语音输入也可以使用。

App 首次运行显示操作引导，以后可以从手机“设置”重新查看。Mac 设置中可以配置会话提醒与 Cursor/VS Code 终端扩展；需先自行安装和登录 Codex/Claude Code。

## 三种连接方式

在手机展开顶部连接卡片 → 设置 → 连接方式，为每台 Mac 分别保存：

| 方式 | 使用条件 |
| --- | --- |
| 仅本地 Wi-Fi | 手机和 Mac 在同一局域网，不需要 VPN 或公网中转；Mac 有线接同一路由器也可以。 |
| Cloudflare | 本地直连优先，离开后走公网中转。每台 Mac 需要部署管理员单独开通并导入专属配置，普通下载不会自动开通。 |
| Tailscale | 本地直连优先，离开后使用两端已经安装并加入同一网络的 Tailscale。 |

局域网需允许设备间互通，访客网络的隔离设置可能阻止连接。合盖、关机、完全断网以及重启后首次 FileVault 登录不能保证远程唤醒或解锁。

## 更新与迁移

手机可以在连接 Mac 后点“检查 App 更新”，也可以下载安装新版 APK。安装仍需 Android 系统确认，现有配对会保留。

Mac 更新：从菜单栏退出旧版，以新版 App 替换应用程序中的旧版，再打开。每次更新可能需要重新确认系统权限。

已有 SwitchPad 脚本版接收端的用户，请保留原安装继续使用，或者先退出并停用旧版自动启动后再迁移；不要同时运行两个接收端。此 Mac App 是独立手机接收端，不含 Switch 手柄驱动。

本仓库用于分发安装包与说明，不包含开发者的配对凭证、私人配置或会话记录。
