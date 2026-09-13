# Remote Vibe 完整代码审查 · 0.5.24

审查日期：2026-09-13。基线为这台开发 Mac 上的 0.5.23 工作副本；修复目标为 Mac / Android 0.5.24（Android 37）。仓库存在此前未提交和未跟踪的源码，本报告不虚构一个干净的 Git 基线。源码指纹另存于本机审查归档。

结论：本轮确认并修复 9 类问题。适合继续公开测试，仍不能宣布 1.0 已通过商店或 Apple 审核。P1 表示优先处理的安全/发布问题，P2 表示可复现的可靠性或数据保护问题。以下是代码审查和隔离复现，不表示已经发生了凭据泄露。

## P1：本机语音服务请求可能经过 HTTP 代理

位置：`switchpad/voice.py` 的 `_request`、`switchpad/remote.py` 的 `transcribe`，共用的新边界为 `switchpad/local_http.py`。

默认 urllib opener 读取系统/环境代理配置。配置代理且未绕过 localhost 时，发往本机 TypeWhisper 的 Bearer 凭据、转写响应乃至手机上传音频可能经过代理；默认重定向还会扩大凭据发送范围。隔离代理捕获器在修改前确实收到本机请求并被当作服务响应接受，复现未使用真实凭据和音频。

修复：本机请求只接受 loopback HTTP，禁止代理和所有重定向，拒绝 URL 用户信息；本机服务 JSON 响应有长度边界，日志仅记录异常类型。编辑器桥接也使用此边界。4 个真实 loopback HTTP 测试覆盖代理陷阱、音频上传、重定向和非法地址/响应大小。

## P1：官网下载站依赖包含已知安全漏洞

位置：独立官网项目的 `package.json` / `package-lock.json`。

升级前 npm audit 报告 11 个受影响包节点（8 high、2 moderate、1 low），涉及 React Server Components 拒绝服务、图像解析和网络/开发工具依赖。这个数字不是 11 个已证明能攻击当前网站的入口；其中 Windows 开发服务器问题不适用于当前部署环境。与服务器运行路径相关的依赖仍应更新。

升级 React / React DOM / RSC 至 19.2.8，vinext 至 beta.9，配套升级 Vite、RSC 插件和 Cloudflare 工具及类型；保留同一架构并解决真实 peer dependency 约束，没有使用 force/legacy-peer-deps。升级后 npm audit 为 0 项；生产构建通过。RSC 官方上游告警：[CVE-2026-44907](https://github.com/advisories/GHSA-wx67-qw84-cm4g)。网站发布结果单独记录，不把本地升级当作线上已经生效。

## P2：剪贴板恢复覆盖新内容，编辑器读取可能误用旧剪贴板

位置：`switchpad/synth.py`、`switchpad/terminal_view.py`、`switchpad/clipboard.py`，编辑器扩展 `editor-bridge/extension.js`。

原逻辑只保存文本且无条件恢复，可能覆盖用户刚复制的新内容、丢失图片等格式；编辑器复制没有改变剪贴板时，还可能把用户之前的剪贴板误认作终端输出。

修复：保存全部可读取的 pasteboard item/type 数据；仅在写入计数和当前内容仍属于本次操作时恢复。编辑器读取要求发生真实复制，扩展返回完整剪贴板摘要，同时限制显示文本长度，避免把截断文本用于错误恢复。7 项隔离测试覆盖多格式、空剪贴板、新复制优先、锁屏/取消、无复制及长输出摘要。扩展还统一 Unicode 码点计数，拒绝 DEL 和孤立代理项；Node 测试使用真实 HTTP 和虚构终端，不操作桌面。

剩余边界：系统剪贴板没有可供本程序使用的原子 compare-and-swap。已缩短检查/写入窗口，不能声称消灭了所有与其他应用并发写入的瞬间竞态。桥接请求在复制之后失败时，也不会为了恢复而覆盖未知来源的新内容。

## P2：暂停或退出后，迟到的语音转写仍可能输入

位置：`switchpad/voice.py`、`switchpad/control.py`、`switchpad/synth.py`。

松开语音键之后的转写线程不再属于 active recording；暂停/断开时仅 stop 无法撤销它。修复增加 cancel 状态，在返回文本和实际粘贴按键之前再次检查。取消中的转写回归测试通过。此保证针对本程序控制的录音/转写路径；通过热键开启的第三方输入法，其自身后台行为仍由第三方控制。

## P2：通知钩子安装可能删除其他用户钩子

位置：`switchpad/install_attention.py`。

以前发现一个自己安装的 hook 就删除整个分组，同组的其他 hook、matcher 或字段会丢失；直接重写配置还可能在写入失败时留下半文件。

修复仅移除明确属于本程序的命令，保留同组其他 hook 和字段；配置用同目录 0600 临时文件、fsync、原子替换写入，保留备份。3 项测试覆盖混合分组、幂等、误判 helper 字符串和替换失败时原文件完整。Claude 与 Codex 配置各自原子替换，并非两个文件的跨文件事务。

## P2：旧确认框可能将操作发到刚切换的新 Mac

位置：Android `MainActivity.renameConversation` / `createConversation` / `launchConversation`。

打开确认框后切换 Mac，再确认，旧代码读取当时的全局 client。修复绑定开框时的客户端实例；新会话还绑定当时选择的上下文。客户端变更即拒绝旧操作。Android 运行测试用内存传输陷阱确认不会向新 Mac 发出改名/新建请求。

## P2：自动重试中的未知消息可以恢复成第二份草稿

位置：Android `MessageProjection` 与 `MainActivity.recoverPending`。

重启加载时 sending 可能显示为 unknown，但 automatic 重试仍在运行；恢复按钮会制造第二份可编辑文本。修复禁止恢复自动重试的 unknown，确认时重新检查消息状态和草稿上下文；允许恢复时先完成持久化移除，再交给草稿。单元测试及真实 Android 对话框测试覆盖自动重试、恢复单一所有者、切换上下文和确认前状态变化。旧版未知消息保留核对提示；删除本机记录不撤销已进入 Mac/CLI 的任务。

## P2：回执存储失败可能中断后续输入处理

位置：`switchpad/control.py` 的 `_remote_tick`。

输入完成后，finally 中写回执若遇到磁盘/SQLite 失败，后续清理和 task.done 都不会执行。现在错误会返回清晰失败状态并完成队列清理，下一条输入继续执行；保持原消息编号及输入前保存的 delivering 状态，不用新编号盲目重发不确定的粘贴。故障注入确认两条任务完成、队列无残留，第一条错误、第二条成功。

## P2：超长日志记录绕过未读统计的工作预算

位置：`switchpad/conversation_activity.py`。

跳过超大 JSONL 行的内部循环原本不计入 16 MiB 扫描预算，会在数据库锁内一直扫描。现在分段保存跳过进度和 skip 状态，重启后继续；预算在正常记录中间用完时保留完整记录供下一轮读取。20 MiB 单行、重启恢复和正常记录跨预算边界均通过测试。

## 审查范围与验证

人工阅读重点覆盖 Mac HTTP/TLS 认证、配对撤销、连接租约、输入白名单、持久回执、锁屏/密码、剪贴板、语音、录屏、会话日志和模型选择；Android 连接生命周期、通知、存储、后台服务、消息合并、输入、二维码、指纹凭据与更新安装；Cloudflare/fallback 中转、编辑器扩展、Mac/Android 打包、公证和官网依赖。完整 Python 测试、Android lint 和 Bandit 补充覆盖外围模块，不代表逐行形式化证明或第三方渗透测试。

- Python 全量 248 项测试通过，包含公网中转协议/负面用例、认证与配对撤销、锁屏、模型控制、回执和本次新增边界。
- Android direct / Play 两渠道共 86 项单元测试通过，release lint 无 Error/Fatal。保留警告供复核。
- Android API 36 当前构建运行 review、boundary、outbox、background、composer 和 header 场景；精确 APK 哈希与最终通过记录见随包 `verification.json`。本轮新增常驻右上角设置齿轮，并检查连接收起/展开、离线和横屏时入口可用。
- Mac DMG 隔离自检、挂载、签名完整性、共享版本、内嵌 APK 哈希和生产凭据排除检查；AAB 由 bundletool 验证，核对权限和已有签名连续性。
- 依赖查询：Mac 已安装的 10 个运行依赖（pip-audit）、Android 已解析的 30 个 Maven 坐标（OSV）、公网中转及官网 npm lockfile。查询时均未命中已知漏洞；不涵盖未公开漏洞、操作系统、Python 解释器或所有第三方 CLI/输入法。
- Bandit 的 shell=True 位于所有者自行配置的本地手柄动作；远程协议不接受 shell 事件。0.0.0.0 监听是 LAN 直连功能，并非未认证公开控制端口。固定公网健康检查和隔离构建自检的 urllib 告警单独核查，不将静态告警数量等同漏洞数量。

## 上线前仍需完成

1. Mac 目前没有有效 Developer ID 证书。真实签名、公证票据和第二台 Mac 的隔离下载/离线首次打开尚未完成。步骤见 MAC-DISTRIBUTION.md。
2. 当前公网中转是管理员配置的测试服务；单个 Durable Object 有全局连接额度。现有认证、并发与流量边界不能证明能抵御大规模公网滥用。大规模公开服务前需要独立容量/滥用测试及可运营的配额、监控和注册流程。
3. Android 厂商省电策略、真实指纹、蜂窝网络切换和 Play 生成包必须实机验证；本轮模拟器不能替代 Samsung/荣耀/小米等设备。
4. 开发者身份、支持邮箱、Play Console 账号和适用的真实封闭测试及商店审批仍需所有者完成。现有报告是同一开发者工具的内部复核，不能冒充独立安全审计。

UI 自动化仅连接隔离接收端，屏幕/会话为虚构数据。未输入真实解锁密码，未更改使用中的会话模型，也未向用户的终端发送测试指令。公网最终更新校验只读取状态及下载 APK，不获取桌面画面或控制租约。

## 0.5.27 current session, allowance and resend changes

- Selected-session metadata refresh is independent of transcript paging. Confirmed model choices display immediately. Passive Terminal/iTerm reads never activate a tab; background polling does not invoke the editor clipboard bridge. Claude’s status-line adapter preserves an existing renderer and stores only model, effort, quota windows and timestamps in private files.
- Codex allowance is read through the installed CLI app-server’s read-only account endpoint over stdio; no thread or model turn is started, no auth material reaches the phone, and refresh is bounded to once per minute. Claude quota is reported by the official status line after a supported subscription session receives a response. Missing windows and stale observations are explicit.
- Both new metadata/allowance endpoints require paired authentication, a live lease and an unlocked Mac. Existing role/control boundaries are retained.
- Every unconfirmed outbox state remains actionable. Manual resend validates the original receipt digest, checks transcript confirmation, refuses to duplicate queued/in-flight work, and records a separate retry ID durably. Transport replay of that retry ID cannot trigger another resend. An ambiguous CLI acceptance can still produce a duplicate after a deliberate new resend; the phone explains this before confirmation. Local deletion stops that phone’s retry record; it does not recall already delivered text.
- Scoped evidence: current-feature UI scenario on the final APK; 254-test Mac regression plus additional metadata/auth boundary checks; see verification-0.5.27.md and the attached logs. Physical OEM phones, independent security testing and store-generated installs remain open.
