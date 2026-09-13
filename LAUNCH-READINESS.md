# Remote Vibe 0.5.25 · 上线就绪状态

**可以公开测试；还不能宣布已经正式上架。** 0.5.25 将设置按钮移到连接卡片外侧，形成独立控件；本次验证范围见 verification.json。 本轮完整代码审查确认并修复本机服务代理泄露、剪贴板恢复、迟到的语音输入、通知钩子覆盖、旧确认框串到新 Mac、消息恢复重复、回执写入故障、超长日志阻塞及官网依赖漏洞。具体触发条件、证据和验证范围见 CODE-REVIEW.md。配对撤销、终端授权选择与签名公证构建流程已在上一版提供。Android AAB、直装 APK、Mac DMG 和版本一致的验证材料一并提供。

## 尚未完成的外部事项

| 事项 | 当前状态 | 接下来需要什么 |
| --- | --- | --- |
| 开发者公开名称、支持邮箱 | 待所有者提供 | 用于商店与隐私/支持页面，不能虚构身份 |
| Google Play 开发者账号 | 本轮未确认完成验证 | 所有者注册、身份/设备验证和账号权限 |
| Mac Developer ID 与公证 | 本机没有可用签名证书 | Apple Developer 账号、Keychain 中的 Developer ID Application 证书和 notarytool profile |
| 商店生成的安装包 | 未分发 | 上传内部测试，检查签名连续性、安装升级及预发布报告 |
| 真实安卓设备试测 | 自动化不能替代 | 至少覆盖荣耀/华为生态差异、Samsung、Pixel 或小米等，重点测后台通知、指纹、重连和网络切换 |
| 第二台 Mac 的首次安装 | 待签名后验证 | 下载隔离标记、离线 Gatekeeper、控制/录屏/自动化权限、扫码配对 |
| 独立安全复核 | 未完成 | 按 SECURITY-REVIEW.md 复核配对/中转/会话输入/密码边界，解决阻断问题 |
| Google Play 生产权限 | 未取得 | 适用的新个人账号需至少 12 位真实测试者连续加入封闭测试 14 天，然后申请生产访问 |

账号、真实测试时长和商店审核不能由脚本代办或伪造。Android 当前 targetSdk 36；Google 的当前目标 API 规则与新个人账号测试要求分别见 [目标 API](https://developer.android.com/google/play/requirements/target-sdk)、[封闭测试要求](https://support.google.com/googleplay/android-developer/answer/14151465)。本项目把独立安全复核和第二台 Mac 实测作为自己的稳定版质量门槛，并非声称它们都是商店统一硬性要求。

## 可重复检查

```sh
.venv/bin/python install/release_readiness.py
.venv/bin/python install/release_readiness.py --strict
```

报告输出至 dist/launch-readiness.json。严格模式在任一门槛未完成时返回失败；不会把旧版本或不同哈希的包当成本版证据。账号与测试结论来自所有者/测试者的明确记录，不由脚本臆测；商店批准始终以 Console 为准。

手工结论可放在本机私有 ~/.config/switchpad/release-profile.json。字段定义见 install/release_readiness.py；不需要密码、私钥或支付信息，也不随发布包上传。真实公证命令与证书准备见 install/mac/DISTRIBUTION.md。

公开 Wi-Fi 直连与用户自己的 Tailscale 已保留；Cloudflare 公网中转目前仍需管理员为每台 Mac 开通配置。此限制需在推广和首用说明中披露，不能宣传为无需配置的自助云服务。iOS 和 Windows 不在本次 Android + Mac 上线范围内。
