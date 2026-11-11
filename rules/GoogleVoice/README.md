# Google Voice 分流规则（Surge）

单文件 RULE-SET，导入后选择自己的节点或策略组即可。默认不需要 MITM、脚本或额外模块。

**版本：0.1.0 · 更新：2026-11-11 · 14 条规则。** 已做静态格式与匹配范围检查，尚未完成导入后的手机收发短信、双向通话和后台推送实测。它是基于已观察连接整理的初版，不宣称覆盖所有 GV 流量。

## 直接订阅

[GoogleVoice.list — 原始订阅链接](https://raw.githubusercontent.com/TEaPoT233/-/main/rules/GoogleVoice/GoogleVoice.list)

```text
https://raw.githubusercontent.com/TEaPoT233/-/main/rules/GoogleVoice/GoogleVoice.list
```

在 Surge 添加外部规则集，类型选择 **RULE-SET**，填入以上链接并选择已有策略组。把这一项放在 Google 总规则及其他会优先匹配这些请求的规则之前。

如果直接编辑配置，在已有的 `[Rule]` 段中添加以下一行。把 `你的GV策略组` 换成配置里真实存在的策略组名，不要重复添加 `[Rule]` 段：

```ini
RULE-SET,https://raw.githubusercontent.com/TEaPoT233/-/main/rules/GoogleVoice/GoogleVoice.list,你的GV策略组
```

文件内部没有固定策略，不会创建策略组，也不会替用户指定某个国家或节点。不要将本文件当成 DOMAIN-SET、完整配置或 sgmodule 导入。规则集下载更新不会自动重建已有连接；修改后退出并重新打开 GV，必要时断开旧连接再复测。

## 这份规则会影响什么

**这不是按 App 身份识别的规则。** GV 的 iOS 客户端使用共享 Google 域名；免解密的域名规则无法只挑选其中的 Voice 路径。其他应用请求相同域名时也会使用所选策略。

| 条目 | 纳入依据 | 范围与限制 |
| --- | --- | --- |
| `voice.google.com` | Google Voice 帮助文档 [S1] | 官方网页入口，精确匹配 |
| `www.googleapis.com` | iOS 观察 [O1] 中明确出现 `/voice/v1/voiceclient/…` | 覆盖账户、会话、消息读取和通知注册等观察到的 Voice API；其他路径也会一起分流 |
| `oauth2.googleapis.com`、`oauthaccountmanager.googleapis.com` | [O1] 的 token / issuetoken 路径 | 共享授权接口，不是 GV 独占 |
| `accountcapabilities-pa.googleapis.com` | [O1] 的 AccountCapabilitiesService 路径 | 共享账户接口 |
| `peoplestack-pa.googleapis.com` | [O1] 的 autocomplete / lookup 路径 | 共享联系人相关接口 |
| `people-pa.googleapis.com` | [O1] 中出现，但显示 MITM Failed | 未取得路径，无法确认业务归属；作为该使用时段观察到的共享候选域名纳入 |
| `notifications-pa.googleapis.com` | [O1] 的 updatethread / multilogin 相关路径 | 共享通知相关接口；不是 Apple APNs 通道 |
| `lh3.googleusercontent.com` | [O1] 的图片请求 | 共享头像及图片主机；没有收录整个 googleusercontent.com |
| `telephony.goog` 及子域 | Google 官方连接要求 [S2] | 通信基础设施，可能同时影响其他 Google 电话服务 |
| `stun.l.google.com` | [S2] | 共享 STUN 主机，域名规则不限制端口 |
| `74.125.39.0/24`、`2001:4860:4864:2::/64` | [S2] | 官方描述为 Workspace Voice 媒体范围，不能据此推定个人版全部通话地址 |
| `74.125.250.129/32` | [S2] 的 STUN 地址 | 不应当作 GV 独占地址 |

[O1] 是一次 iOS 使用 GV 时的近期请求观察，并非进程级归因结果。只有带 `/voice/…` 路径的请求能从现有证据直接确认属于 Voice；同时出现的共享接口不等于都由 GV 发起。公开文件只保留通用主机名和必要说明，不包含截图、令牌、账户标识、完整头像地址或请求正文。

未添加整个 `google.com`、`googleapis.com`、`googleusercontent.com`、Google ASN 或只按 UDP 端口匹配的规则，以免扩大到大量无关流量。其余 Google 依赖仍需由原配置中的 Google / 代理规则接管。

## 与参考规则的关系

本次核对了使用者提供的 [MagicRule GoogleVoice 产物](https://magicrule.pages.dev/data/_shared/artifacts/GoogleVoice_rules_7e2a5efa51b704e2.list) 及其 [生成清单](https://magicrule.pages.dev/data/rules/Surge/2b7e7ba8e631ed8fee1bcbcf/manifest.json)。

在 2026-11-11 的读取中，清单和文件均只有 1 条 DOMAIN 规则，内容为 `lens.l.google.com`；清单的 `providers` 为空，无法从该清单追溯具体上游。清单生成时间为 2026-11-10，规则更新时间则标为 2026-07-26。这说明“产物生成时间”不等于“业务覆盖已经更新”。

本文件沿用可直接订阅的 Surge RULE-SET 格式，条目根据 [O1] 和 Google 官方资料重新整理。现有证据不足以确认 `lens.l.google.com` 对 GV 的必要性，因此未纳入；这不等同于断言它永远不会被 GV 使用。

## 通话、推送与解密

- 规则命中只决定出口，不保证服务或节点可用。Google 官方说明通话音频依赖 UDP [S3]；所选节点及服务端需实际支持 UDP 转发 [S7]。TCP 测速通过不等于语音通话通过。
- [S2] 列出的出站端口包括 UDP 19302–19309、26500–26501 及 TCP 443。这里没有编写按这些端口匹配全网流量的规则；列出端口也不会自动开放防火墙。
- Workspace 媒体范围是补充，不是个人版媒体 IP 全表。个人版如连接其他媒体地址，会由原配置继续匹配；需要实际通话日志才能有依据地补充。
- iOS 后台来电或短信提醒还涉及系统通知权限及 APNs 等条件。此规则不改动 Apple 推送分流，不保证解决通知不弹出的问题。
- 本规则不需要解密，也不会关闭已有 MITM。如果 `people-pa.googleapis.com` 等出现 `MitM Failed`，先暂停 MITM 做对照，再检查需要排除的主机。不能仅凭这个提示认定节点故障或证书固定。[S6]
- 对 HTTPS 的 `/voice/` 路径做 URL-REGEX 精确分流需要成功解密 [S5]。为保持单文件、免解密的用法，本版本没有加入 URL-REGEX。

## 导入后的验证

先让 GV 策略组选用一个已知可用的节点，便于观察。分别测试：

1. 冷启动、刷新会话、打开旧短信：检查 `www.googleapis.com` 是否命中本规则及所选策略。
2. 实际收发短信、登录状态与联系人搜索：观察成功结果；“有下载字节”不能替代 HTTP / 业务成功验证。
3. 拨出、接听和双向音频：检查实际 UDP 连接与媒体目的地，确认没有单向无声。
4. 回到后台后接收短信或来电提醒：单独确认系统通知正常。

发布前检查仅覆盖文件结构、重复条目、CIDR 合法性及抽样匹配边界，不是 Surge 原生解析器运行结果。当前没有在维护环境中运行 Surge，也未替用户执行电话或短信测试。后续补规则应基于功能操作前后的连接对照，不要把整段背景流量直接全部归为 GV。

本文件是手动维护的订阅；固定链接便于后续更新，不代表存在自动抓包、自动追踪官方变化或定期维护任务。

## 来源

所有网页核对日期：2026-11-11。

- [S1 — Google Voice 使用与故障排查](https://support.google.com/voice/answer/10063441?hl=en)
- [S2 — Google Voice connectivity requirements](https://knowledge.workspace.google.com/admin/voice/voice-connectivity-requirements)
- [S3 — Optimize your network for Voice](https://knowledge.workspace.google.com/admin/voice/optimize-your-network-for-voice)
- [S4 — Surge Rule Set 格式](https://manual.nssurge.com/rules/ruleset.html)
- [S5 — Surge HTTP Rules](https://manual.nssurge.com/rules/http.html)
- [S6 — Surge MITM](https://manual.nssurge.com/http/mitm.html)
- [S7 — Surge UDP Relay](https://manual.nssurge.com/policies/udp.html)
- [S8 — Surge Domain Rules](https://manual.nssurge.com/rules/domain.html)
- [S9 — Surge IP Rules / no-resolve](https://manual.nssurge.com/rules/ip.html)
