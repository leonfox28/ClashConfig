# ClashConfig (Mihomo / Clash Meta / Loon 配置仓库)

个人自用的 **Mihomo (Clash Meta)** 配置文件与自定义分流规则仓库。采用二进制 `.mrs` 规则集、精细化 DNS 分流与防泄漏策略，以及按地区自动测速的策略组架构。

另提供 **Loon** 配置 `loon.lcf` 和转换后的个人规则，使用方式见下方「Loon 个人规则」。

---

## 📁 目录结构

```text
.
├── general.yml              # 通用配置模板（适用于常规订阅）
├── exflux.yml               # Exflux 专属配置模板
├── yyssr.yml                # YYSSR 专属配置模板
├── loon.lcf                 # Loon 配置模板
└── rule/                    # 个人自定义规则目录
    ├── Personal.list        # 个人直连规则（内网网段、私有域名、指定 IP 直连）
    ├── personal-proxy.list  # 个人代理规则（指定个人服务走「📲 个人服务」策略组）
    └── loon/
        ├── Personal.lsr       # Loon 个人直连规则
        └── personal-proxy.lsr # Loon 个人服务规则
```

---

## ✨ 核心特性

- **高性能 MRS 规则集**：全面接入 [MetaCubeX/meta-rules-dat](https://github.com/MetaCubeX/meta-rules-dat) 的 `.mrs` 二进制规则集，加载更快、内存占用更低。
- **防泄漏 DNS 架构**：
  - 启用 `fake-ip` 模式，内置完善的 `fake-ip-filter` 白名单（涵盖局域网、Apple/Android/Windows 网络连通性检测、NTP 授时、STUN 及主机游戏联机域名）。
  - **国内外解析隔离**：海外默认 DNS 通过代理隧道查询（`1.1.1.1` / `8.8.8.8` 绑定 `#🚀 节点选择`）；大陆及私有域名通过 `nameserver-policy` 强制走阿里 DoT（`tls://223.5.5.5:853`）及本地内网 DNS。
  - **DNS 劫持**：内置规则将发往 `8.8.8.8:53` / `8.8.4.4:53` 的明文 DNS 请求劫持至内部 `dns-out` 处理，防止客户端硬编码 DNS 绕过。
- **精准流量嗅探 (Sniffer)**：支持 HTTP / TLS / QUIC 端口嗅探与域名覆写，解决纯 IP 连接无法命中域名规则的问题，同时排除米家 (`+.io.mi.com`) 和苹果推送 (`+.push.apple.com`) 等长连接服务以免断连。
- **智能节点筛选**：使用 `include-all-proxies` 自动归类港、台、日、新、美、韩六大地区 `url-test` 自动测速组，并通过正则自动剔除「剩余流量、到期时间、官网」等无效信息节点。

---

## 🔄 配置文件差异对比

三份主配置文件共享完全一致的策略组与分流规则，仅针对不同机场/订阅源的节点解析需求对 **DNS 节点服务器 (`proxy-server-nameserver`)** 和 **`fake-ip-filter`** 做了区分：

| 配置文件 | 适用场景 | `proxy-server-nameserver` | 专属节点域名豁免 (`fake-ip-filter`) |
| :--- | :--- | :--- | :--- |
| **`general.yml`** | 通用订阅 | `223.5.5.5`、`119.29.29.29` | 无（保持通用纯净） |
| **`exflux.yml`** | Exflux 订阅 | 专属 DoH 节点解析器 | `proariaden927861.exzxwmrbnedyoldwod.com` |
| **`yyssr.yml`** | YYSSR 订阅 | 专属 DoH 节点解析器 | `+.tls.yytls.top`、`+.quic.yydzx.xyz` |

---

## 🧭 策略组与分流优先级

### 策略组一览

- **主控与手动**：`🚀 节点选择`、`🚀 手动切换`
- **业务分流组**：
  - `📲 个人服务`（默认直连，按需切换代理）
  - `🤖 AI 服务`（OpenAI / Claude / 海外 AI 平台）
  - `🐱 代码托管`（GitHub / GitLab / Atlassian）
  - `📹 油管视频`、`🎥 奈飞视频`、`📺 巴哈姆特`、`📺 哔哩哔哩`、`📺 哔哩港澳台`
  - `🔍 谷歌服务`、`📢 谷歌FCM`、`📲 电报消息`
  - `Ⓜ️ 微软服务`、`🍏 苹果服务`、`🎮 游戏平台`
- **基础与兜底**：`🔒 国内服务`、`🎯 全球直连`、`🛑 广告拦截`、`🐟 漏网之鱼`
- **地区自动测速组**：`🇭🇰 香港节点`、`🇨🇳 台湾节点`、`🇸🇬 新加坡节点`、`🇯🇵 日本节点`、`🇺🇸 美国节点`、`🇰🇷 韩国节点`

### 规则匹配顺序（自上而下）

1. **本地 DNS 劫持**：拦截 `8.8.8.8` / `8.8.4.4` 的 53 端口流量至 `dns-out`
2. **个人私有网络与穿透**：优先匹配 `personal-proxy`、`personal` 及局域网私有地址
3. **下载客户端直连**：前置匹配 BT / PT / 下载工具规则 (`download`)，防止跑代理流量或触发机场封禁
4. **广告拦截**：拦截 `category-ads-all`
5. **第一层国内直连**：国内大厂域名 (`geolocation-cn`)、中国大陆 IP (`cn-ip`) 及国内游戏 (`games-cn`) 优先直连
6. **专项海外服务**：依次匹配 AI、Google FCM、流媒体（YouTube / Netflix / 巴哈姆特 / Bilibili）、Google、Telegram、代码托管、微软、苹果、游戏平台
7. **海外常用域名**：`geolocation-!cn` 快速命中代理
8. **第二层国内兜底直连**：完整中国大陆域名 (`cn-domain`) 与 `GEOIP,CN` 直连
9. **最终兜底**：未命中规则的流量交由 `🐟 漏网之鱼` 处理

---

## 🚀 使用方法

### 1. 客户端要求
本仓库配置使用了 Mihomo 专属语法（如 `.mrs` 规则集、`rule-anchor`、`include-all-proxies` 等），请确保使用基于 **Mihomo (Clash Meta)** 内核的客户端：
- Clash Verge Rev / Mihomo Party / FlClash / Clash Nyanpasu
- Stash (iOS / macOS) / Clash Meta for Android (CMFA)
- OpenClash / Nikki (OpenWrt)

### 2. 引用链接 (GitHub Raw)

可在 **Sub-Store** 等订阅转换工具中将以下链接作为配置模板使用：

- **通用模板 (`general.yml`)**：
  ```text
  https://raw.githubusercontent.com/leonfox28/ClashConfig/main/general.yml
  ```
- **Exflux 模板 (`exflux.yml`)**：
  ```text
  https://raw.githubusercontent.com/leonfox28/ClashConfig/main/exflux.yml
  ```
- **YYSSR 模板 (`yyssr.yml`)**：
  ```text
  https://raw.githubusercontent.com/leonfox28/ClashConfig/main/yyssr.yml
  ```

### 3. 自定义规则维护
- 需要强制**直连**的个人域名或内网 IP，请添加至 `rule/Personal.list`。
- 需要单独控制走向（走 `📲 个人服务` 策略组）的域名或规则，请添加至 `rule/personal-proxy.list`。

`iroh.link` 按协议分流：`Personal.list` 仅匹配 UDP，`personal-proxy.list` 仅匹配 TCP，避免前置的个人直连规则覆盖 TCP 代理规则。需要代理 Relay 连接时，请在客户端将 `📲 个人服务` 切换到可用代理节点；`🎯 全球直连` 保持 `DIRECT`。三份模板均已将 `+.iroh.link` 排除在 fake-IP 外。域名规则不覆盖直接使用对端 IP 的 P2P UDP 连接。

### 4. Loon 个人规则

`rule/loon/Personal.lsr` 从 `rule/Personal.list` 转换，省略已由主配置 `10.0.0.0/8` 绕过设置及 `Lan.lsr` 覆盖的 `10.21.20.221/32`，保留 15 条规则，其余 IP 规则保留 `no-resolve`。其中 `iroh.link` 的直连规则限定为 UDP，并显式包含 Loon 的 QUIC、STUN 协议类型。`rule/loon/personal-proxy.lsr` 保留 2 条规则，将 Mihomo 的 `NETWORK,TCP` 转换为 Loon 的 TCP、HTTP、HTTPS 协议条件。逻辑规则和协议规则需要 Loon 3.1.7 或更新版本，语法见官方[逻辑规则](https://nsloon.app/docs/Rule/logic_rule/)和[协议规则](https://nsloon.app/docs/Rule/protocol_rule/)文档。

`loon.lcf` 已在 `[Remote Rule]` 最前面引用这两个文件：

```ini
[Remote Rule]
https://raw.githubusercontent.com/leonfox28/ClashConfig/main/rule/loon/Personal.lsr, policy=DIRECT, tag=Personal.lsr, enabled=true
https://raw.githubusercontent.com/leonfox28/ClashConfig/main/rule/loon/personal-proxy.lsr, policy=个人服务, tag=personal-proxy.lsr, enabled=true
```

新文件推送到 GitHub 的 `main` 分支后，以上订阅地址才可下载。将规则用于其他 Loon 配置时，需要保留名为 `个人服务` 的策略组，或同步修改引用行中的 `policy`。

这里保留个人直连规则在前、个人服务规则在后的顺序：`cpa.zenithconsulting.cn` 直连，其他 `zenithconsulting.cn` 子域名交给「个人服务」。Loon 版本将 `iroh.link` 及其子域名按协议拆分，避免整域直连覆盖 TCP 规则：

| Loon 识别的协议 | 策略 |
| --- | --- |
| TCP、HTTP、HTTPS | 个人服务 |
| UDP、QUIC、STUN | DIRECT |

这些域名规则只作用于 Loon 能识别为 `iroh.link` 或其子域名的请求，不会自动覆盖直接使用对端 IP 的 P2P 连接。主配置中的 `disable-stun = true` 仍可能阻止 STUN 请求；将其分流到 DIRECT 不等于解除全局 STUN 禁用，参见官方[通用配置](https://nsloon.app/docs/General/#disable-stun)。

`loon.lcf` 中原有的本地 `DOMAIN-SUFFIX,zenithconsulting.cn,个人服务` 已由远程规则接管，避免本地规则优先匹配后覆盖 `cpa.zenithconsulting.cn` 的直连例外。内网域名的 `[Host]` DNS 映射仍需配置在主配置中。

Loon 文件是独立副本；之后修改 Clash 个人规则时，请同步维护 `rule/loon/` 下的对应文件，并保留上述协议分流和冗余内网 IP 的省略。
