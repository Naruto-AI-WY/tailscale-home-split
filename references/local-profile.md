# 个人环境快照

来源：本聊天在 2026-10-22 的配置与验收。此表帮助快速定位，不是实时状态、访问凭据或操作许可；新会话实施前先核对。迁移其他家庭时替换地址和身份，不盲目套用本表。

## 拓扑与节点

| 对象 | 已记录信息 | 角色 |
| --- | --- | --- |
| 家庭 OpenWrt ezopwrt-home | LAN 192.168.10.1/24；WAN 192.168.23.14；Tailscale 100.127.241.124 | LAN 子网入口、手机 exit node、OpenClash |
| OpenClash SOCKS | 100.127.241.124:7893，有认证 | 笔记本家庭出站 |
| 爱快 | 192.168.23.1 | OpenWrt 上级网关 |
| ikuai-mgmt Linux VM | LAN 192.168.23.5；Tailscale 100.100.172.75 | 独立爱快管理；入口 http://100.100.172.75/ |
| NAS_Naruto / nas-naruto | LAN 192.168.10.143；Tailscale 100.109.79.56；Download 共享 | 已验收通过家庭子网路由的 SMB |
| NAS_WY | LAN 192.168.10.189；Tailscale 100.101.130.14 | 第二台 NAS，不能与未知 LT_intelligence 名称等同 |
| 本笔记本 | Tailscale 100.105.49.73 | 系统 Tailscale + Clash Verge |
| iPhone | Surge 节点名 iphone-surge-home，Home-Split 配置 | 内置 Tailscale Home-Tailscale 策略 |

家庭 SSID：`Naruto`；测试热点：`iPhone_Yan`。家庭网关已记录 MAC `00-0C-29-D2-EB-62`。曾见 `Naruto_TP` / 192.168.23.1，不能仅凭相似 SSID 当作同一家庭接入。电脑有业务设备接口 192.168.1.198，网络分流维护不更改该接口。

两个路由侧节点 ezopwrt-home 和 ikuai-mgmt 均已按用户指定禁用节点密钥到期；使用前重新核对，不能因此给新手机/其他节点自动设置永久。ikuai-mgmt 不发布路由或 exit node。

爱快已有 UDP 映射 WAN1:41641 → 192.168.23.14:41641，服务于 OpenWrt Tailscale 直连候选路径；不要再添加重复规则。它未服务于新管理 VM 的 41642，也没有把管理 HTTP/SMB/SOCKS 暴露公网。

## 当前实现所有者

稳定目录：`C:/Users/wanpi/Documents/Codex/NetworkTools/ClashVergeHomeAuto`。

- 主要文件：`HomeAuto.ps1`、`TailnetHealth.ps1`、`HomeAutoSupervisor.ps1`、`Manage-HomeAuto.ps1`、`Launch-Supervisor.vbs`、`settings.json`。
- 状态证据：`status.json`、`supervisor-status.json`、lifecycle/events 日志；任务计划 `ClashVergeHomeAuto`。
- 控制入口：核验脚本后，`Manage-HomeAuto.ps1 -Action Status` 只读；`Pause` / `Resume` 会改状态，仅在当前维护范围使用。
- Clash 配置目录：`C:/Users/wanpi/AppData/Roaming/io.github.clash-verge-rev.clash-verge-rev`；配置 UID `HomeSplitTS20261022`。活动核心与用户 profile 都要核对，备份或缓存不是生效入口。
- 已记录控制器 127.0.0.1:9097；mixed-port 曾为 7897/7900，以当前核心与监听为准。`config.yaml` 含 secret，profile 含 SOCKS 认证，不整份打印或复制进 skill。
- 家庭出站策略名 `Home-OpenClash` / `家庭出口`；私网路径为绑定 Tailscale 的 `Home-LAN` direct 类型。它不等于走 OpenClash HTTP 代理。

家里模式记录为系统代理/TUN 关闭，accept-routes 关闭；外出为规则模式接管、accept-routes 开启，Windows 全局 exit node 为空。后台已从随 CMD 退出改为独立隐藏监督，不应再创建相同功能任务。

## 回溯位置（可选）

本轮原始工作目录：`C:/Users/wanpi/Documents/Codex/2026-10-22/new-chat`。以下是可追溯资料，缺失时重新发现/验证，不作为 skill 的硬依赖：

- `outputs/Home-Split.conf` 与 `outputs/Surge-Naruto-自动暂停设置.txt`：手机模板，用户端可能已另有修改。
- `outputs/手机热点-分流与NAS共享验证-20261022.txt`：此前外网样例；不能替代新一次外网测试。
- `outputs/爱快-Tailscale-部署记录-20261022.txt`、`work/ikuai-tailscale/deployment-result.json`：独立管理 VM、节点到期及验证范围。
- `work/ikuai-tailscale/guest-post-reboot-validation.json`：guest 重启检查；没有验证爱快宿主断电恢复。

`Tailscale_Download` 显示名保留，已验收实际目标为 `\\192.168.10.143\Download`。原生 Windows 凭据由用户输入并保存，不能从文档推定当前运行身份已能使用。

## 不携带的内容

本 skill 不收录 Auth Key、代理密码、控制器 secret、cookie、SSH 私钥、登录授权链接、VM 镜像或完整网络日志。旧临时 SSH 授权与注册 key 不复用。若要分享本 skill，先删除本个人环境参考及主文件中的对应入口，或替换为接收方经确认的资料。
