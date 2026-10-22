# 架构与验收

## 四条数据路径

| 场景 | Windows 笔记本 | iPhone |
| --- | --- | --- |
| 外出访问国内规则目标 | Clash DIRECT → 当前 Wi-Fi/热点 → 目标 | Surge DIRECT → 当前蜂窝/Wi-Fi → 目标 |
| 外出访问命中国外规则目标 | Clash 家庭策略 → Tailscale → OpenClash SOCKS → 家庭规则/上游节点 → 目标 | Surge Home-Tailscale → 内置 Tailscale → 家庭 exit node → OpenClash → 目标 |
| 外出访问家庭内网 | 系统路由 → Tailscale → OpenWrt 子网路由 → NAS/路由器 | 家庭 CIDR 规则 → Home-Tailscale → OpenWrt → 内网服务 |
| 已在家庭局域网 | 自动关闭本地系统代理/TUN 接管，经家庭网关出站 | 匹配家庭网络后 Surge 暂停处理，经家庭网关出站 |

“国内/国外”是当前规则表、域名和 IP 分类结果，不是网页语言或服务器物理位置的保证。显式海外规则可先于 GEOIP/CN；新增业务域名应根据实际请求分析。

Tailscale 是加密传输，Clash/Surge 是手机或电脑的流量分类器，OpenClash 是家庭出站策略。Windows 本地 Clash 连接家庭 SOCKS 不一定再用一条独立代理订阅；不要只数进程就认定“两轮代理”。家庭旁路减少同一终端回环进家里的路径。

## 三种 Tailscale 能力

- **子网路由**发布指定家庭网段，让没有 Tailscale 的设备也可访问；还需要控制台批准、访问策略、目标服务和返回路径。
- **出口节点**承载被选择的互联网出站。Windows 的本方案用家庭 SOCKS，因此不需要全局选它；Surge 可把 exit node 限于指定策略的流量。
- **节点端口 / Serve**提供某个节点服务。爱快管理 VM 转发单一管理端口，不要求把爱快整个网段发布出去。

控制平面负责身份、密钥、peer/route 信息；业务数据由直连 UDP 或中继搬运。关闭节点密钥到期与永久注册用 Auth Key 不是一回事；即使禁用到期，仍可撤销设备。

## 验证顺序与证据强度

先找故障的第一层，避免同时重置所有组件：

1. 物理网络：当前默认路由、接口源地址和网关；热点测试确认家庭网线及家庭 Wi-Fi 已断开。
2. Tailnet：设备授权、peer 可见、子网批准、访问控制、当前连接/握手与必要端口。
3. 家庭服务：管理页/代理端口/SMB 445 连通，区分目标不监听与认证拒绝。
4. 规则：国内显示 DIRECT，国外显示家庭策略，家庭私网优先命中正确路径。
5. 应用：实际 HTTP/HTTPS、目录只读枚举或用户页面确认；受系统代理影响的应用和 TUN 捕获的应用分别检查。

| 要说明的结论 | 足够有用的证据 | 单独不足的证据 |
| --- | --- | --- |
| 国内直连当前网络 | 同次请求 DIRECT，加真实连接源地址/物理接口与目标 | 能打开百度、单个公网 IP 检测页 |
| 国外回家 OpenClash | 本地连接链选家庭 SOCKS，TCP 对端为家庭 tailnet IP:端口，应用成功；必要时核对家庭日志 | 只有节点测速成功或网页状态码 |
| NAS 可访问 | 目标 445 路由/握手，原生用户会话能只读列目录 | ping 成功、NAS Web 登录页正常 |
| Tailscale 直连 | 目标 peer 当前 direct endpoint、重复观测的 ping/status 与延迟 | 控制台 Preferred DERP 城市、Online=true |
| 自动恢复有效 | 网络状态变更及进程重启后的当前状态/动作日志和路径验证 | 任务计划存在、进程 PID 还在 |

HTTP/TCP 样例不自动证明 UDP/QUIC、IPv6、所有 App 或长期稳定性。记录测试时间与网络；旧成功用于定位回归，不冒充当前成功。

官方语义参考：[子网路由](https://tailscale.com/docs/features/subnet-routers)、[出口节点](https://tailscale.com/docs/features/exit-nodes)、[连接类型](https://tailscale.com/docs/reference/device-connectivity)。实施前核对当前版本与帮助。
