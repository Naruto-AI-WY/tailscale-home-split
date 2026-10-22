# 路由器、独立管理和 NAS

## OpenWrt 家庭入口

先确认固件、架构、Tailscale 安装方式/服务、当前 LAN/WAN、转发/防火墙、OpenClash 监听与认证、路由批准和访问策略。优先既有软件包/服务，按实际版本操作；不要猜命令、全放行 tailscale0 或公开未认证代理。记录家庭 SOCKS 和互联网出站实测，避免透明代理捕获 Tailscale 自身流量产生依赖环。

家庭 subnet 与 exit-node 分别配置、批准和验证，不自动添加其他网段。临时 SSH 公钥只在已授权范围使用，任务结束核对撤销；历史已移除的 key 不代表现在仍有 SSH 权限。管理页登录由用户完成，不索要聊天明文密码。

## 直连、DERP 与端口映射

先用 `tailscale status`、目标 `tailscale ping` 和 `netcheck` 的当前输出判断，再查家庭 WAN 与外部公网是否一致、上级 NAT/CGNAT、IPv6、UDP 限制和实际监听端口。控制台 Preferred DERP 是候选中继偏好，不证明业务流量一直经该城市。

在可达公网边界上为实际 Tailscale UDP 监听端口做精确映射，可能提高直连成功率；基本访问原本可通过穿透/DERP，所以映射不是“以前为什么能访问”的前提。本案例为爱快 WAN1 UDP 41641 → OpenWrt WAN 192.168.23.14:41641；它不是 OpenClash 端口，也不是所有节点都必须开放的固定规则。

只有当前任务已授权所需防火墙/端口变更才保存；已有明确授权可沿用，不再次询问同一保存。不能用历史案例给新环境开公网端口；不开放路由器管理、SMB 或 SOCKS 到 WAN。CGNAT 或双端限制下映射仍不保证直连。测试要来自真正外网，记录连接从 DERP 到 direct 是否发生、实际端点和延迟；保留正常中继回退。

## 爱快作为独立管理入口

先核对当前爱快版本、设备架构、虚拟机功能、存储与现有能力，不假定有可用的原生 Tailscale 插件或 Docker。已有 VM 时维护现有 VM；无原生能力时可选官方支持的 Linux 管理 VM。

本例使用 Linux VM 的 userspace Tailscale 加尾网内 TCP Serve：

```text
tailnet 客户端 → ikuai-mgmt 的 TCP 80
  → Tailscale Serve → VM 127.0.0.1:8080
  → 本地端口转发 → 爱快 192.168.23.1:80
```

在目标版本确认后，示例 Serve 命令为 `tailscale serve --bg --tcp=80 tcp://127.0.0.1:8080`。后端转发只绑定 loopback，限制目的管理地址/端口；不监听 VM 的所有 LAN 地址。客户端入口须 tailnet 访问策略允许，爱快页面仍使用原爱快后台凭据；Tailscale 身份提供商账号和 Linux 控制台账号用途不同。

仅管理入口的目标：不 advertise-routes、不 advertise-exit-node、不接管 DNS/路由、不开 IP forwarding、不启用 Funnel，不覆盖原 OpenWrt 路由或增加 WAN 管理映射。若当前版本对 Serve/userspace 的组合不支持，先验证替代设计，不扩大成全网段代理。

存储先核对分区身份、容量、用途和实际占用，优先现有可用空间。只有确实需初始化/格式化且已有针对该分区的许可时实施；本例只授权 sda7，最终未发生格式化。上传后核对实际目录和文件，不把进度 99% 当成功。VM 修改后验证同一节点身份、自启服务、Serve、管理页及既有网络；宿主机重启/断电属于不同验证范围。

Linux guest 软重启挂起时查启动参数、控制台/getty、服务启停日志。过去同时调整串口启动与重复终端后恢复，但没有独立 A/B 证明单一原因，不照搬成所有 VM 的通用修复。

## 节点密钥到期

“设备永不过期”应针对指定节点禁用 node key expiry，不是把注册用 Auth Key 改成永久。核对名称和 tailnet IP，按当前请求处理指定设备；已授权同一节点就执行，不额外要求重复确认。保存后检查控制台到期状态并可用受支持接口交叉验证。新节点不继承旧节点的永久授权；不要给整个 tailnet 无关设备关闭到期。

## NAS：网络与凭据分层

区分 NAS 的 tailnet 地址和家庭 LAN 地址两条路径。一条故障不等于整台 NAS 离线。子网路由可用时可访问 `\\家庭LAN地址\共享名`；NetBIOS 广播/旧快捷方式名称不保证跨子网工作，入口显示名也不证明实际目标。

先确认目标身份、共享名、路由和 TCP 445，再在原生 Windows 用户会话验证 SMB 认证。需要密码时让用户在 Windows 凭据窗口输入，不写聊天/报告。不批量断开其他 SMB 会话、清空凭据、重置 NAS 账号、关闭签名或打开 SMB1。

验证以只读目录枚举为主，少量统计即可，不输出私密文件名。已知工作共享可保留显示名，修正它的实际目标并验收。其他未知 NAS 名称不能凭猜测映射。NAS 网页成功不代表 SMB 共享权限通过。

官方参考：[Serve](https://tailscale.com/docs/reference/tailscale-cli/serve)、[用户态网络](https://tailscale.com/docs/concepts/userspace-networking)、[节点到期](https://tailscale.com/docs/features/access-control/key-expiry)、[爱快虚拟机说明](https://www.ikuai8.com/support/ymgn/lyym/2019-12-13-06-35-05/2019-05-13-02-02-08.html)。
