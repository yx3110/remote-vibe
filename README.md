# Remote Vibe

用手机遥控 Mac、查看实时画面、语音输入，并浏览和操作 Codex / Claude Code 会话。

## 下载

- [Mac · Apple 芯片（M 系列）](https://github.com/yx3110/remote-vibe/releases/latest/download/Remote-Vibe-Mac-Apple-Silicon.dmg) · macOS 14 或更新版本
- [Android APK](https://github.com/yx3110/remote-vibe/releases/latest/download/Remote-Vibe.apk) · Android 8 或更新版本
- [最新版本、安装说明和 SHA-256 校验文件](https://github.com/yx3110/remote-vibe/releases/latest)

当前是 **0.5.36 测试版**。Mac 版尚未完成 Apple Developer ID 签名与公证，也未上架 Mac App Store；Android 尚未上架 Google Play。Intel Mac 和 iOS 版本暂未提供。

界面支持简体中文、繁體中文、English、日本語、Español、Italiano 和 Deutsch，可跟随系统或手动选择。手机「设置 → 语言」与 Mac 菜单栏的语言选择互不影响，离线也可切换。

## 首次连接

1. 在 Mac 下载 DMG，打开后将 Remote Vibe 拖入 Applications，再从“应用程序”启动。不需要 Python、Homebrew 或终端命令。
2. 如果 macOS 阻止打开，在“系统设置 → 隐私与安全性”找到本次拦截，确认下载来源后点“仍要打开”。
3. 在 App 设置窗口开启辅助功能、屏幕录制权限。需要登录后自动启动时勾选“登录 Mac 时启动”。
4. 手机安装 Android APK，与 Mac 接入同一 Wi-Fi；在 Mac 点“显示配对二维码”，在手机点“添加 Mac”扫码。
5. 按住说话需另装 TypeWhisper，下载语音模型并开启 API Server；App 设置提供下载入口。手机键盘自带的语音输入也可以使用。

App 首次运行显示操作引导，以后可以从手机“设置”重新查看。Mac 设置中可以配置会话提醒与 Cursor/VS Code 终端扩展；需先自行安装和登录 Codex/Claude Code。

0.5.27 将遥控、会话、画面和语音输入栏统一到 Material 3 设计，保留独立设置齿轮，并补测 Android 8/11/13/15/16 与 16 KB ARM64 系统。同时修复模型刷新与卡住消息，加入画面方向键和使用额度。原有配对可继续使用，手机内检查更新即可。0.5.24 完整代码审查与修复记录见 CODE-REVIEW.md；Mac 公证仍待所有者安装证书。

## 安全与授权

Mac 菜单栏可打开“安全与授权”。手机丢失或二维码泄露时，确认“撤销全部配对”，旧手机断开后重新扫码。Cloudflare 路由凭据如泄露还需管理员单独撤销。

新建终端会话默认使用标准授权；如确有需要，可在此明确选择完全访问（YOLO）。该选项不修改现有会话、模型或 shell 别名。旧版本升级后请检查此偏好。

会话列表支持长按拖动排序，顺序按每台 Mac 和运行/历史筛选分别保存；新消息不会打乱排列。

正式发布待办见 [上线状态](LAUNCH-READINESS.md)。

## 三种连接方式

在手机点连接卡片右侧的独立齿轮 → 连接方式，为每台 Mac 分别保存：

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


## 0.5.28 会话命名与多语言网站

新会话可由对应的已登录 Codex / Claude Code 根据多轮对话生成具体名称，成功后固定保存。原会话及其模型/effort 不变；生成使用该服务商账号的额度。失败保留已有名称。网站的下载、支持和隐私页面提供七种语言与语言切换。

如需为这台 Mac 上已有的会话重新生成名称，安装 0.5.28 后，在该 Mac 的终端运行：

```sh
"/Applications/Remote Vibe.app/Contents/MacOS/Remote Vibe" --helper refresh_titles --refresh-all
```

这会先备份 Remote Vibe 名称库，再刷新自动名称；手动名称和原始 CLI 会话记录会保留。只影响执行命令的 Mac，不会刷新其他电脑。较多会话需要数分钟，需对应 CLI 已登录且有可用额度。详见 [本版验证范围](https://github.com/yx3110/remote-vibe/releases/download/v0.5.28/verification-0.5.28.md)。

## 0.5.29 连接面板

顶部连接状态保持紧凑，点开独立的 Material 3 底部面板，页面不会再被向下挤。连接操作、设备管理和版本信息重新分组，独立设置齿轮保留。适配手机、折叠屏、大字体与横屏；返回、旋转和切换设备入口保留草稿与连接。详见 [本版验证范围](https://github.com/yx3110/remote-vibe/releases/download/v0.5.29/verification-0.5.29.md)。

## 0.5.30 回车键

普通画面和全屏的回车键统一放在方向键中间，使用清晰居中的矢量图标和强调底色；遥控页中央回车使用同样样式。无需在工具条里寻找小号字符。详见 [本版验证范围](https://github.com/yx3110/remote-vibe/releases/download/v0.5.30/verification-0.5.30.md)。

## 0.5.31 多台 Mac 切换

设备切换改为独立卡片列表，突出当前电脑和连接状态，各台电脑分别提供修改地址、指纹解锁设置和确认移除；适配窄屏、大屏和大字体，保留各设备草稿和稳定排序。详见 [本版验证范围](https://github.com/yx3110/remote-vibe/releases/download/v0.5.31/verification-0.5.31.md)。

## 0.5.32 额度与手机语音

修复 Codex 后台启动的额度读取，补充 Claude 账户订阅额度与清晰的用量上限提醒。按住说话改用手机系统语音服务，识别结果先进入草稿，确认发送后才传文字到 Mac；无公开识别服务时可使用键盘语音，不需要 Mac 转写软件。详见 [本版验证范围](https://github.com/yx3110/remote-vibe/releases/download/v0.5.32/verification-0.5.32.md)。

## 0.5.33 统一界面与会话问题

新建会话使用 Codex / Claude Code 卡片；配对、模型、额度、历史与确认窗口统一主界面样式。修复横屏输入按钮遮挡和对话框软键盘焦点。Claude/Codex 的交互提问与选项显示在会话中，并提供临时回答和确认入口。详见 [本版验证范围](https://github.com/yx3110/remote-vibe/releases/download/v0.5.33/verification-0.5.33.md)。

## 0.5.34 直接切换 Mac

连接状态面板直接列出其他已配对的 Mac，点名称即可连接。保留各 Mac 的草稿和独立设备管理入口。详见 [本版验证范围](https://github.com/yx3110/remote-vibe/releases/download/v0.5.34/verification-0.5.34.md)。

## 0.5.35 修复新建会话

修复源码安装下 Codex 启动脚本模块加载失败，以及 Terminal 复用已结束标签编号后无法识别新标签的问题。详见 [本版验证范围](https://github.com/yx3110/remote-vibe/releases/download/v0.5.35/verification-0.5.35.md)。

## 0.5.36 每台 Mac 独立连接方式

可分别为每台 Mac 保存本地 Wi-Fi、公网中转或 Tailscale；设置其他 Mac 不会切换当前连接，设备列表可查看已保存的连接方式。修复旧状态覆盖其他设备设置和重新扫码重置连接方式的问题。详见 [本版验证范围](https://github.com/yx3110/remote-vibe/releases/download/v0.5.36/verification-0.5.36.md)。
