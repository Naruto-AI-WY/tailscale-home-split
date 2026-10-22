# iPhone：Surge 内置 Tailscale

## 登录与规则

iPhone 上独立 Tailscale App 与 Surge 的 VPN 接管通常互斥。本方案仅保持 Surge 运行；Tailscale 会话在 Surge 策略内部，不要求同时启动两个 VPN。

`[Proxy]` 中的 `tailscale, section-name=home` 与 `[Tailscale home]` 对应。交互登录在 Surge 策略编辑页完成，登录的是现有 tailnet 的身份提供商账号。`interactive-login = true` 仅引用此 App 本地状态，复制配置不能转移身份；不要为了整理文档改 section-name 后仍承诺旧登录有效。Auth Key 与交互登录二选一，真实密钥不嵌入示例。

按任务保留原有规则集，重点检查顺序：

1. Tailscale 控制 URL 的可用出站及解析先于兜底，不能递归指向依赖它的 Home-Tailscale。DERP/STUN/peer 底层同样不能绕回自身策略。
2. 家庭网段、所需 tailnet/MagicDNS 规则交给 Home-Tailscale，早于通用 LAN DIRECT。
3. 命中海外域名表交给家庭策略；国内域名/IP 交给 DIRECT；其余按用户指定兜底。

策略选择家庭 exit node 可承载海外互联网请求；它不把其他 DIRECT 请求自动变成全局回家。家庭子网仍须已被发布、批准且允许该 Surge 节点访问。

## 已遇到的登录故障

日志若为“交互登录但未找到有效的登录状态”，而私网规则已命中 Home-Tailscale，先修复 Surge 内部身份。打开首页代理服务器中 Home-Tailscale 的编辑页，在该页重新完成 Tailscale 登录，登录有效后点右上角保存并重新加载。独立 App 已登录不替代这一步。

需要取消旧登录时，随后在同一编辑流程重新登录再保存；不能让用户把未登录且空 Auth Key 的策略保存下来。出现“Auth Key 的值无效”先检查两种认证方式是否缺失/混用；不用公开粘贴密钥。重新登录仍失败时读控制阶段、时钟/DNS/门户及规则递归证据，别直接重装或删除全部配置。

## 家庭 Wi-Fi 自动暂停

当前家庭名为 Naruto；已核对官方语法：

```ini
[SSID Setting]
SSID:Naruto suspend=true
```

已有此节则只合并一行。实际家庭名称以用户当前设置为准，可分别添加多个已确认名称。其他 Wi-Fi/蜂窝不匹配时恢复规则处理；不要配置成所有 Wi-Fi 都暂停。`suspend=true` 暂停处理并不等于系统 VPN 图标消失。

**不要与 `include-all-networks=true` 合用**，该组合可让匹配网络断网。按当前版本核对相关依赖选项。Surge 读不到 SSID 时检查 App 显示的实际网络与系统权限，不自行猜导航名称。

保持 Surge VPN 启动及适当的始终开启/按需设置；SSID 条件只负责暂停/恢复流量处理，不负责把用户手动关闭的 VPN 打开。重启后、首次解锁、未登录热点等状态需要实际验证，不能仅因打开“始终开启”就保证所有情况。

## 验证与日常使用

家庭网络看暂停状态及原网络可用；关闭 Wi-Fi 用 5G，查看是否恢复，并分别发起国内、海外和 `http://家庭网关/` 请求。在 Surge 工具中的最近请求/请求查看对应记录：策略、命中规则、计时、错误；入口名称可能随版本变化，以用户屏幕为准。DIRECT 记录与当前网络连接信息支持国内直连；Home-Tailscale 记录与访问成功支持回家链路。VPN 图标、单个测速结果不够。

手机不能代操作时给最少步骤，要求必要文字或截图并标注用户确认；不要把电脑模板更新写成手机已安装。

官方参考：[Tailscale 策略](https://manual.nssurge.com/policies/tailscale.html)、[家庭网络暂停](https://manual.nssurge.com/features/subnet-settings.html)。模板及 UI 操作前检查版本。
