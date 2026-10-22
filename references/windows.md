# Windows：Clash Verge + 系统 Tailscale

## 接手已有配置

先在 [个人环境快照](local-profile.md) 找已存在的维护目录。读取 settings、status、supervisor-status 和任务计划身份，核对更新时间与实际 PID/命令行归属。任务还在或 PID 被复用，都不能证明工作进程健康。

从当前核心 `/configs`、实际监听与 Verge 设置交叉确认 mixed-port、external-controller、mode、TUN。控制器 secret 仅在本机用于授权调用，不输出整个 config.yaml 或带密码的代理配置。端口曾从 7897 变到 7900；应查占用、启动日志和配置来源，不能仅凭变化归因用户修改或自动认定某种软件行为。

本机已有 `HomeAuto.ps1`、`TailnetHealth.ps1`、`HomeAutoSupervisor.ps1` 和 `Manage-HomeAuto.ps1`。在其所有者目录窄修改，备份前后状态；不复制另一套常驻程序或修改备份中的文件当作生效配置。现有实现含家庭地址假设，迁移网段不能只改 settings 后假定完成。

## 家庭判断与切换

以当前有效物理默认路由为基础，结合家庭网关地址、该接口上的网关 MAC 和源网段。SSID 可辅助，但别的 Wi-Fi 可能同名/同网段；仅经 Tailscale 访问到家里网关也不表示本机在家。网关 MAC 是拓扑识别线索，不是安全认证。家庭/外部同时存在或同优先级冲突时记录歧义，不把不确定状态当纯家庭或纯外出验收。

| 状态 | 本机接管 | Tailscale 子网接受 | 互联网出口 |
| --- | --- | --- | --- |
| 已确认家庭物理网络 | 系统代理关，TUN 关 | 本方案关，使用物理 LAN | 家庭网关/OpenClash |
| 已确认外部网络 | 规则模式；按需求启用系统代理、TUN | 开；仅已批准家庭网段 | 国内物理 DIRECT，国外家庭 SOCKS |
| 无网络/身份不明/服务故障 | 保留诊断并等待/有限恢复 | 不盲目翻转全局路由 | 不把国外目标悄悄兜底为成功直连 |

Windows 的全局 Tailscale exit node 应为空，否则国内 DIRECT 仍可能经家里。家里设备则可继续提供 exit node 给手机。

## 规则、接口与 DNS

家庭 SOCKS 使用家庭 Tailscale IP:端口和已有认证，必要时绑定系统 Tailscale 接口；不能把真实密码放入 skill 或报告。家庭子网与 tailnet 流量可用明确的直连策略绑定 Tailscale 或由 OS 路由承接，先于一般私网规则。`DIRECT` 的实际路径由路由/绑定决定，不自动等于物理网络。

给 Tailscale 服务进程、控制/协调连接及 peer 传输保留物理出站，避免 Clash → Tailscale → Clash 的依赖环。分别检查核心支持的进程规则、`tun.route-exclude-address`、tailnet/家庭 CIDR 以及 DNS 引导；`exclude-uid` 等 Linux 功能不能当作 Windows 进程排除。单纯排除 100.64/10 也不能覆盖所有公网 peer endpoint 和控制连接。

本机快照使用进程 DIRECT、tailnet 地址排除及家庭路径规则；它是起点，不能宣称解决所有 endpoint/IPv6 情况。不要在没有路由证据时一概排除所有私网；外部 Wi-Fi 与家庭网段重叠时需明确选路或改地址方案。避免 DNS 查询本身依赖尚未建立的家庭隧道；保留控制平面可启动的解析路径。

## 后台运行与有限修复

使用任务计划/服务和隐藏启动器，工作进程与用户 CMD 窗口脱离。守护进程检查 PID 身份、心跳新鲜度与卡死，异常后有退避和次数上限；只管理自己持有的工作进程，不批量杀 PowerShell/Clash。区分意外退出与用户明确 Pause/禁用，后者不能被守护进程反复拉起。

切网异常按物理连接 → Tailscale peer → 家庭 SOCKS → Clash 顺序探测。服务仍在线但底层连接过期时，一次受控重连可能有用；先记录原 prefs、确认用户仍希望运行并设冷却时间。不要每次失败都 down/up，更不要用 `--reset` 丢掉已设选项。恢复后重读 prefs 和链路；用户主动关闭不自动开启。旧案例中重连 Tailscale 后恢复，不等于所有故障根因已被证明。

验证异常退出恢复时只停止自有工作进程；准备恢复与本机记录后再测试，保持其他业务接口不变。真实网络切换/服务重启仅在当前任务已授权且需要时实施，不因 skill 回归擅自执行。

## 原生 Windows 验证

远程执行器或应用隔离可能看见不同的 WinINet 设置、SMB 会话/凭据。记录运行身份和上下文，不因为当前进程失败就删除所有凭据或网络映射。必要时在已授权的原生交互用户会话运行有限只读探针，保存结果并移除临时任务。

访问国外时观察本机到家庭 tailnet SOCKS 的连接；访问国内时观察 DIRECT 和物理源地址。读 NAS 只枚举必要信息，不创建测试文件、不输出私人文件名。失败先分 DNS、路由、TCP、代理认证和 SMB 权限，不全面关闭防火墙。

字段兼容性以安装版本和 [Mihomo TUN](https://wiki.metacubex.one/config/inbound/tun/)、[规则](https://wiki.metacubex.one/config/rules/)、[DNS](https://wiki.metacubex.one/config/dns/) 为准。
