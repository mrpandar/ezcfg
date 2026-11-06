# EZcfg

> **EZ network cfg** — *configuration & cartography*
> 局域网拓扑发现 · 拖拽建图 · 隐患审计 · 故障定位 · AI 辅助处置
> LAN topology discovery · drag-and-drop mapping · risk audit · fault diagnosis · AI-assisted remediation

[![MIT License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Platform](https://img.shields.io/badge/platform-Windows%20%7C%20macOS%20%7C%20Linux-lightgrey.svg)](#installation--部署)
[![Python](https://img.shields.io/badge/python-3.10%2B-blue.svg)](#installation--部署)
[![AI](https://img.shields.io/badge/AI-built--in%20%26%20BYO--API-8a2be2.svg)](#ai-assistant--ai-助手)

**EZcfg** is an open-source LAN operations toolkit: it discovers what is actually connected to your network, lets you refine the map by hand, audits risks and faults, and uses AI to turn each finding into a concrete remediation plan. The same scan can be inspected in a desktop GUI, a local web UI, or a headless CLI.

**EZcfg** 是一款开源局域网运维工具：发现真实连线的设备、允许手工修正拓扑、审计隐患与故障，并由 AI 把每一条发现转化为可直接执行的处理建议。同一次扫描结果可在桌面 GUI、本地 Web UI 与无头 CLI 之间共享。

> **Not affiliated.** The name `EZcfg` is reused independently. This project has no relation to any other project of the same name, and does not generate vendor configuration files on its own.
>
> **项目无关声明：** 名称 `EZcfg` 为独立复用，与同名的其它项目无任何关联；本工具本身不生成厂商设备配置文件。

---

## Features · 功能特性

| Icon | English | 中文 |
|---:|:---|:---|
| 🔍 | **Auto-discovery** — scan a subnet and recover the live topology from LLDP/CDP neighbours, ARP/ND tables, SNMP bridge tables and STP state, then rank every link by confidence | **自动发现** — 扫描网段，从 LLDP/CDP 邻居、ARP/ND、SNMP 转发表与 STP 状态还原实时拓扑，并为每条链路标注可信度 |
| 🧭 | **Path tracing** — pick two endpoints and see the exact forwarding path: every switch, VLAN, trunk, ACL and NAT translation in between | **路径追踪** — 选两个端点即给出完整转发路径，逐跳列出经过的交换机、VLAN、Trunk、ACL 与 NAT 转换 |
| 🖱️ | **Drag-and-drop canvas** — place devices and links freely, import a scan as the starting layer, then edit without losing the discovery data underneath | **拖拽画布** — 自由摆放设备与链路，可把扫描结果作为底图导入，手工修改不会覆盖原始发现数据 |
| 🧩 | **Multi-vendor** — 10+ vendors: Huawei, Cisco, H3C, Ruijie, Aruba, Ruiyuan, Palo Alto, Fortinet and generic SNMP-capable boxes | **多厂商** — 覆盖华为、思科、新华三、锐捷、Aruba、锐捷（通用）、Palo Alto、Fortinet 及通用 SNMP 设备 |
| 🧰 | **Object types** — switches, routers, firewalls, wireless controllers and APs, servers and virtual machines, printers, IP cameras, NAS and unmanaged switches | **设备类型** — 交换机、路由器、防火墙、无线控制器与 AP、服务器与虚拟机、打印机、IP 摄像头、NAS、非网管交换机 |
| ⚠️ | **Risk audit** — loops and blocked STP ports, VLAN leaks, duplicate IPs and overlapping DHCP scopes, single points of failure, weak SNMP community and plaintext protocols | **隐患审计** — 环路与阻塞端口、VLAN 泄漏、IP 冲突与 DHCP 地址池重叠、单点故障、弱 SNMP Community、明文管理协议 |
| 🩺 | **Fault diagnosis** — symptom-to-cause engine: start from a failed host or a down link, get a ranked hypothesis list with the commands to verify each one | **故障诊断** — 症状到原因的推理引擎：从一台失联主机或一条断链出发，给出排序后的原因假设及逐条验证命令 |
| 📡 | **Config & alert collection** — pull `running-config` / `startup-config` over SSH and SFTP, and collect syslog and SNMP trap messages into a single evidence timeline | **配置与告警采集** — 通过 SSH 与 SFTP 抓取 `running-config` / `startup-config`，并收集 syslog 与 SNMP Trap，统一合并为一条证据时间线 |
| 🤖 | **AI-assisted analysis** — the built-in AI explains each finding, ranks causes, drafts vendor-specific remediation commands and predicts impact before you apply them | **AI 辅助分析** — 内置 AI 解释每条发现、排序原因、起草对应厂商的处理命令，并在执行前预判影响范围 |
| 💬 | **Configuration copilot** — describe the goal in plain language and get a config draft for unfamiliar devices, then validate it against the device model before pushing | **配置助手** — 用自然语言描述目标，为不熟悉的机型生成配置草稿，下发前按设备型号做语法校验 |
| 🔌 | **Flexible access** — desktop GUI, local web UI for teams, and a headless CLI for cron jobs; one scan result is shared across all three | **部署方式** — 桌面 GUI、面向团队的本地 Web UI、用于定时任务的无头 CLI，同一份扫描结果三端通用 |
| 🔓 | **Open and auditable** — every audit rule and every AI prompt is a readable file; AI output is shown as a *suggestion* and is never applied automatically | **开源可审计** — 每条审计规则与每一套 AI 提示词均为可读文件；AI 输出一律以「建议」呈现，绝不自动执行 |

---

## How discovery works · 探测原理

```
          Layer 2 discovery · 二层发现                Layer 3 + inference · 三层与推理
┌──────────────────────────────────────┐    ┌──────────────────────────────────────┐
│ 1. ARP / NDP sweep                   │    │ 5. Bridge-MIB dot1dTpFdb             │
│    → alive hosts + MAC table         │    │    on every switch                   │
│    → 存活主机与 MAC 表                 │    │    → 逐台交换机的 MAC 转发表            │
├──────────────────────────────────────┤    ├──────────────────────────────────────┤
│ 2. LLDP (lldpRemTable)               │───▶│ 6. IP-MIB / ARP on routers           │
│    CDP (Cisco legacy)                │    │    → resolve unknown MACs            │
│    → neighbour pairs                 │    │    → 在路由器上解析未知 MAC             │
│    → 直连邻居配对                      │    │                                      │
├──────────────────────────────────────┤    ├──────────────────────────────────────┤
│ 3. System description                │    │ 7. Cross-check all sources           │
│    + ifName mapping                  │    │    → confidence score                │
│    → vendor / model / role           │    │    → unknown L2 segments             │
│    → 厂商 / 型号 / 角色                │    │    → 多源交叉校验并给出可信度与未知       │
├──────────────────────────────────────┤    │      二层网段                         │
│ 4. STP state                         │───▶├──────────────────────────────────────┤
│    → loops vs redundant link         │    │ 8. Diff against baseline             │
│    → 环路与冗余链路区分                 │    │    → added / moved / gone            │
└──────────────────────────────────────┘    │    → 与基线比对，识别新增 / 移动 /       │
                                            │      消失                             │
                                            └──────────────────────────────────────┘
```

**Why two layers.** LLDP and CDP only reveal *direct* neighbours, so they cannot recover a topology across an unmanaged switch or a link where neighbour discovery is disabled. Layer 3 fills the gaps: switches expose their MAC forwarding table via the Bridge MIB, and routers expose their ARP cache. Cross-checking these sources produces a confidence score on every link; an unmanaged switch or a hub, which speaks neither LLDP nor SNMP, is rendered as an **unknown L2 segment** placeholder rather than silently merged into the wrong link.

**为什么分两层。** LLDP/CDP 只能看到直连邻居，无法穿透非网管交换机或关闭了邻居发现的链路。第三层负责补洞：交换机通过 Bridge-MIB 暴露 MAC 转发表，路由器暴露 ARP 缓存。多源交叉校验后每条链路都有可信度；既无 LLDP 也无 SNMP 的非网管交换机或 Hub，会显示为**未知二层网段**占位，而不是被悄悄合并到错误的链路上。

### Requirements · 采集前提

| Prerequisite · 采集前提 | Required for · 采集能力 | Optional · 是否可选 |
|:---|:---|:---:|
| ICMP reachable seed subnet<br>可 Ping 通的种子子网 | host sweep<br>主机存活扫描 | no · 否 |
| LLDP or CDP enabled on switches<br>交换机已开启 LLDP 或 CDP | layer-2 links<br>二层链路发现 | no · 否 |
| SNMPv2c read community or SNMPv3 user<br>SNMPv2c 只读团体名或 SNMPv3 用户 | bridge tables, STP, interfaces<br>转发表、STP 状态、接口信息 | yes* · 是* |
| SSH read-only credentials<br>SSH 只读凭据 | config backup, deep audit, AI evidence<br>配置备份、深度审计、AI 分析证据 | yes · 是 |
| Syslog / SNMP trap destination set to EZcfg<br>Syslog 与 SNMP Trap 指向 EZcfg | alert and event correlation<br>告警与事件关联分析 | yes · 是 |
| SSH write credentials or an API token<br>SSH 写入凭据或 API 令牌 | AI-assisted config push (manual confirmation required)<br>AI 辅助配置下发（须人工确认） | yes · 是 |

\* Without SNMP the result degrades to a host-and-gateway map; device-to-device links cannot be resolved.

\* 无 SNMP 时只能得到主机与网关图，设备之间的链路无法还原。

---

## AI Assistant · AI 助手

### Two channels · 两种通道

| Channel · 通道 | English | 中文 |
|:---|:---|:---|
| ☁️ **Cloud AI** (default) | hosted by the EZcfg project, works with no setup, trained on the bundled vendor knowledge base and updated continuously | **云端 AI**（默认）由 EZcfg 项目托管，开箱即用，基于随版本发布的厂商知识库训练，持续更新 |
| 🔑 **Bring your own API** | point EZcfg at any OpenAI-compatible endpoint — OpenAI, Azure OpenAI, a self-hosted model, or a local Ollama instance | **自带 API** 指向任意 OpenAI 兼容端点：OpenAI、Azure OpenAI、自建模型或本地 Ollama |

```bash
# use the cloud AI (sign in once, free tier available)
# 使用云端 AI（一次性登录，提供免费额度）
EZcfg ai login

# bring your own API key
# 使用自带的 API Key
EZcfg ai config --provider openai-compatible \
    --base-url https://api.example.com/v1 \
    --model gpt-4o-mini \
    --api-key-file ~/.EZcfg/ai.key

# local model, completely offline
# 使用本地模型，全程离线
EZcfg ai config --provider openai-compatible \
    --base-url http://127.0.0.1:11434/v1 \
    --model qwen2.5:32b
```

### What the AI sees · AI 能看到什么

Only the minimum context needed for the task is sent, and you approve it before it leaves the machine:

仅发送完成任务所必需的最小上下文，且发送前须经你确认：

1. **Discovery evidence** — the relevant slice of the topology graph, not the whole network · **探测证据**：拓扑图中与本次问题相关的片段，而非整网数据
2. **Device model and OS version** — to pick the right CLI syntax · **设备型号与软件版本**：用于匹配正确的命令行语法
3. **Running configuration** — only the sections related to the finding · **当前运行配置**：仅涉及该问题的相关段落
4. **Syslog and trap messages** — the correlated event window · **Syslog 与 Trap 消息**：已关联到该问题的事件时间窗

The device list, MAC table and unrelated config blocks stay on the local machine unless you explicitly attach them. All prompts and response schemas are stored under `prompts/` as plain text files that can be reviewed or edited.

设备清单、MAC 表及无关的配置段落始终保留在本地，除非你显式附加。所有提示词与响应结构均以纯文本形式存放在 `prompts/` 目录下，可随时审阅或修改。

### AI workflows · AI 能做什么

| Workflow · 场景 | Trigger · 触发方式 | AI output · AI 输出 |
|:---|:---|:---|
| Finding explainer · 发现解读 | click any audit item | a one-screen explanation of the risk in plain language · 用通俗语言解释该风险的一屏摘要 |
| Fault root cause · 故障根因 | **Diagnose** or a red device on the canvas | ranked causes with evidence, verification commands and confidence · 排序后的原因、证据、验证命令与置信度 |
| Remediation planner · 处置方案 | **Fix it for me** | a step-by-step plan, vendor-specific commands, rollback steps and blast-radius estimate · 分步处置方案、对应厂商命令、回退步骤与影响范围评估 |
| Config copilot · 配置助手 | **New config** or paste a partial config | a config draft for the target model, a diff view, and a pre-check against the device schema · 针对目标型号的草稿、差异视图及下发前语法校验 |
| Alert triage · 告警分诊 | incoming syslog or trap | deduplicated incidents, related historical events, and a suggested first check · 去重后的事件、历史相似事件及建议首先排查的方向 |
| Baseline advisor · 基线建议 | a completed scan | a hardening checklist ordered by impact, with commands you can copy · 按影响排序的加固清单及可直接复制的命令 |

### Scope of responsibility · 责任范围

The cloud AI receives the vendor knowledge base updates and is tuned against real device outputs, so its suggestions are generally more accurate than a generic model asked the same question.

云端 AI 随版本获得厂商知识库更新，并针对真实设备输出做过调优，因此其建议通常比使用通用模型回答同一问题更精准。

**Customer-provided AI.** When you supply your own API key or run a local model, EZcfg is a transport layer only: it formats the prompt and displays the answer. The EZcfg project does not control the model, cannot guarantee its output, and **takes no responsibility for any suggestion produced by a customer-provided AI**. Review every command before it touches a production device.

**客户自带 AI。** 使用你自己的 API Key 或本地模型时，EZcfg 仅作为传输层，负责组织提示词并展示回答。EZcfg 项目不控制该模型、不保证其输出，且**对客户自带 AI 产生的任何建议不承担任何责任**。命令下发到生产设备前，须逐条复核。

This distinction is enforced in the UI: cloud suggestions carry a **Cloud-verified** tag, while customer-model suggestions carry an **Unverified model** tag and a mandatory confirmation checkbox.

这一区分在界面上有强制体现：云端建议标注 **Cloud-verified**，客户模型建议标注 **Unverified model** 并显示强制确认勾选框。

> **AI suggestions are not commands.** Every AI-generated command is shown in a diff-style preview and requires a deliberate confirm action. AI never logs in to a device, never writes to `startup-config`, and never schedules a change window on its own. The audit log records who confirmed each command.
>
> **AI 建议不是命令。** 每条 AI 生成的命令均以差异预览形式展示，须主动确认才会执行。AI 不会自行登录设备、不会写入 `startup-config`，也不会自行排定变更窗口。审计日志记录每一条命令的确认人。

---

## One-click diagnosis · 一键定位

The GUI is designed so that the most common operation — *the network is broken, find out why* — takes three clicks. No CLI knowledge is required.

界面围绕最常见的场景设计：网络出故障了，找出原因。全程无需命令行知识，三步完成。

```
   ┌────────────────────────────────────────────────────────────────────┐
   │  ①  Select symptom · 选择症状                                       │
   │  ─────────────────────────────                                     │
   │  🔴 Red device          🐢 Slow segment      📴 Unreachable host   │
   │     红色设备/异常            网段缓慢              主机不可达           │
   │  🆕 New device          ➕ Custom alert                            │
   │     新出现设备               自定义告警                               │
   │                              │                                     │
   │                              ▼                                     │
   │  ②  EZcfg investigates · EZcfg 自动排查                             │
   │  ────────────────────────────────────                              │
   │  ▸ 3 scan passes                       三轮扫描                     │
   │  ▸ log + config collection             日志与配置采集                │
   │  ▸ path + STP + FDB cross-check        路径 / STP / FDB 交叉校验     │
   │  ▸ AI ranks 4 hypotheses               AI 排序四个假设               │
   │  ▸ evidence copied into prompt         证据片段送入提示词             │
   │                              │                                     │
   │                              ▼                                     │
   │  ③  Apply suggestion · 应用建议                                     │
   │  ─────────────────────────────                                     │
   │  [Verify]  ping …         先验证：执行 ping 等确认命令                 │
   │  [Fix it]  system …       再处置：修复系统或下发配置                    │
   │  [Rollback] config …      可回滚：回退到变更前配置                     │
   └────────────────────────────────────────────────────────────────────┘
```

**Step 1 — pick the symptom.** Click any red device on the map, choose from a preset list (unreachable host, slow segment, flapping port, new unknown device), or paste a free-form alert such as `VLAN 20 unreachable 10 minitues`. Voice input is available for field work.

**第 1 步 — 选择症状。** 点击图中任意红色设备，从预设列表选择（主机不可达、网段变慢、端口频繁震荡、出现未知设备），或粘贴一条原始告警，例如 `VLAN 20 断网 10 分钟`。现场作业支持语音输入。

**Step 2 — EZcfg investigates.** Three scan passes run automatically; syslog and trap messages are pulled for the same window; the path engine, STP state, FDB and running configuration are cross-checked; the AI receives only the evidence slice and returns a ranked hypothesis list.

**第 2 步 — EZcfg 自动排查。** 自动执行三轮扫描，同步拉取同一时间窗内的 syslog 与 trap 消息，由路径引擎、STP 状态、FDB 与运行配置交叉校验；AI 仅接收相关证据片段，返回排序后的假设列表。

**Step 3 — read, verify, fix.** Each hypothesis shows the device, port, evidence and a command to confirm. A green badge means the command was already run and matches; a red badge means it contradicts the theory. The **Fix it** button drafts the remediation; the **Rollback** button snapshots the current config first.

**第 3 步 — 阅读、验证、处理。** 每条假设显示设备、端口、证据及验证命令。绿色标记表示已实际执行且结果吻合，红色标记表示与假设矛盾。**Fix it** 起草处置方案，**Rollback** 会先对当前配置做快照。

A finished diagnosis can be saved as a **case**, exported as a PDF for the changelog, or promoted into a reusable **playbook** that runs automatically the next time the same pattern appears.

一次完成的诊断可保存为**案例**，导出 PDF 归入变更记录，也可沉淀为可复用的**预案**，下次出现相同模式时自动执行。

---

## Installation & Deployment · 安装与部署

### Four deployment shapes · 四种部署形态

| Shape · 形态 | Best for · 适合谁 | One-liner · 一行命令 | What you get · 得到什么 |
|:---|:---|:---|:---|
| 🪟 **Desktop GUI** | office admins, one site, a single LAN | download the installer from [Releases](../../releases) | one-click scan, map, diagnosis, no server needed · 一键扫描、出图、诊断，无需服务器 |
| 🐳 **Single-box appliance** | small IT team, shared scan history | `docker compose up -d` | web UI at port 8080, SQLite, scheduled scans · 8080 端口 Web UI、SQLite、定时扫描 |
| 🖥️ **Team server** | multiple sites, role-based access | `apt install EZcfg-server` or the compose file | multi-user, projects, audit log, optional cloud AI relay · 多用户、多项目、审计日志、可选云端 AI 中继 |
| ⌨️ **Headless CLI** | cron, scripts, CI pipelines | `pipx install EZcfg` | scan, audit, trace, diagnose — no GUI required · 扫描、审计、追踪、诊断，无需界面 |

```bash
# Desktop GUI — Windows, macOS, Linux
# 桌面 GUI：Windows、macOS、Linux
# download EZcfg-desktop from Releases, then
# 从 Releases 下载 EZcfg-desktop，然后：
EZcfg desktop

# Single-box appliance — web UI on http://localhost:8080
# 单机部署：Web UI 位于 http://localhost:8080
curl -sSL https://EZcfg.dev/install.sh | sh
EZcfg serve --port 8080 --data-dir /var/lib/EZcfg

# Small team — Compose stack with scheduling and history
# 小团队：含调度与历史记录的 Compose 组合
git clone https://github.com/mrpandar/EZcfg.git
cd EZcfg/deploy
docker compose up -d

# Headless — scan from a script or a cron job
# 无头模式：从脚本或定时任务中扫描
pipx install EZcfg
EZcfg scan 192.168.1.0/24 --out tonight.graphml
```

### Deployment recommendations · 部署建议

| Situation · 场景 | Recommended shape · 建议形态 | Why · 原因 | AI channel · AI 通道 |
|:---|:---|:---|:---|
| One office, one part-time admin | Desktop GUI on a laptop | zero infrastructure, USB stick is enough for field work | Cloud AI, sign in once · 云端 AI，一次登录 |
| 2–10 person IT team, one building | Single-box appliance on an old PC or a VM | shared history and scheduled scans, no cloud server needed | Cloud AI relay, one API key for the team · 云端 AI 中继，团队共用一个 Key |
| Multi-site managed service | Team server plus one agent per site | central view, per-site credentials, audit log per customer | Cloud AI, or BYO key per tenant · 云端 AI，或按租户自带 Key |
| Air-gapped or classified network | Desktop GUI + local model | no traffic leaves the site | local Ollama / vLLM endpoint · 本地 Ollama / vLLM 端点 |
| Already has Grafana / Loki | Team server with syslog and trap receivers enabled | reuse existing alert pipeline | either · 两种通道均可 |

### Sizing · 容量参考

| LAN size · 局域网规模 | CPU | RAM | Storage for history · 历史存储 | Scan duration · 扫描耗时 |
|:---|:---:|:---:|:---:|:---:|
| up to 100 endpoints<br>100 台以内 | any modern x64<br>任一款现代 x64 | 2 GB | 5 GB / year<br>5 GB / 年 | 30–90 s |
| 100–1,000 endpoints<br>100–1,000 台 | 4 cores<br>4 核 | 8 GB | 20 GB / year<br>20 GB / 年 | 2–6 min |
| 1,000–5,000 endpoints<br>1,000–5,000 台 | 8 cores<br>8 核 | 16 GB | 60 GB / year<br>60 GB / 年 | 6–15 min |
| larger<br>更大规模 | split by site, one agent each<br>按站点拆分，每站一个 Agent | — | — | parallel<br>并行 |

These are planning figures, not guarantees. Scan time depends mainly on SNMP responsiveness and the polling concurrency setting.

以上为规划参考值，非性能保证。扫描耗时主要取决于 SNMP 响应速度与并发轮询设置。

> **Scanning is a sensitive operation.** Only run EZcfg on networks you own or are authorised to test. Discovery packets may be logged as reconnaissance by IDS/IPS, and some printers, cameras and industrial controllers misbehave under rapid ARP or SNMP polling. Start with a small /24 and observe before widening the scope. When AI is enabled, the data minimisation rules in the AI Assistant section apply: review the evidence packet before it is sent.
>
> **扫描属于敏感操作。** 仅在你拥有或获得授权的网络上运行。发现报文可能被 IDS/IPS 记录为侦察行为；部分打印机、摄像头与工业控制器在高频 ARP 或 SNMP 轮询下会出现异常。建议先扫一个 /24 观察，再逐步扩大范围。启用 AI 时，须遵循 AI 助手章节中的数据最小化规则：发送前审阅证据包。

---

## Quick start · 快速开始

### GUI — one scan, one map, one diagnosis · 界面 — 一次扫描，一张图，一次定位

1. Launch the app and open **Discovery → New scan** · 打开 **发现 → 新建扫描**
2. Enter `192.168.1.0/24`, choose SNMP profile, set threads to `32` · 填入网段，选择 SNMP 模板，并发设为 `32`
3. Wait for `Reachable hosts` and `Layer-2 links` to stabilise · 等待「可达主机」与「二层链路」数量稳定
4. Click **Build map** — the canvas opens with devices auto-arranged · 点 **生成拓扑图**，画布自动布局
5. Right-click any red device and choose **Diagnose** → the AI panel opens with ranked hypotheses · 右键任意红色设备选 **诊断**，AI 面板即显示排序后的假设
6. Click **Fix it** to draft the remediation, then **Apply** after reviewing the diff · 点 **Fix it** 起草处置方案，审阅差异后点 **Apply**
7. Save as `.EZcfg` (project, keeps history and rollback points) or export GraphML / PDF · 存为 `.EZcfg`（含历史与回退点），或导出 GraphML / PDF

### GUI — drawing manually · 界面 — 手工建图

1. Start a blank canvas from **File → New** · 从 **文件 → 新建** 打开空白画布
2. Drag device icons from the left stencil: switch, router, firewall, AP, server, camera, printer, generic · 从左侧图元拖出设备
3. Connect ports by dragging from one port circle to another; the link label shows the inferred VLAN · 从一个端口圆点拖到另一个端口圆点完成连线，标签自动标注推断的 VLAN
4. Right-click a link for **Trace path** or **Add to audit scope** · 右键链路可执行「追踪路径」或「加入审计范围」
5. Import a previous scan as background layer to snap your drawing onto real data · 可导入历史扫描结果作为底图，让手绘图对齐真实数据

### Writing a config with the AI copilot · 用 AI 助手写配置

```
1. Canvas → right-click device → "Configure"
   画布中右键设备 →「配置」
2. Describe the goal: "把 1-8 口划到 VLAN 120，网关 10.12.0.1/24"
   描述目标
3. AI drafts a diff for the detected model and OS version
   AI 按识别出的型号与版本起草差异
4. EZcfg parses the diff and checks syntax against the device schema
   EZcfg 解析差异并按设备模式校验语法
5. Confirm to push; a rollback snapshot is taken first
   确认后下发，先自动做回退快照
```

The copilot works without an internet connection when a local model endpoint is configured, and falls back to manual editing if neither AI channel is available.

已配置本地模型端点时，配置助手可离线工作；两种 AI 通道均不可用时，自动退化为手工编辑模式。

### CLI

```bash
# scan a subnet and write a GraphML file
# 扫描网段并输出 GraphML
EZcfg scan 192.168.1.0/24 --out topology.graphml

# same scan, JSON output, low verbosity
# 同一扫描，JSON 输出，低日志级别
EZcfg scan 192.168.1.0/24 --format json --vlan 10 --quiet

# run the audit rules against an existing topology
# 对已有拓扑运行审计规则
EZcfg audit topology.graphml --rules rules/core.yaml --severity medium

# render a scan result as a PNG without running the GUI
# 将扫描结果渲染为 PNG，无需启动界面
EZcfg render topology.graphml --out map.png --layout fdp

# trace the forwarding path between two endpoints
# 追踪两个端点之间的完整转发路径
EZcfg trace topology.graphml --from 192.168.1.20 --to 10.0.0.5

# diagnose a failed host and rank the possible causes
# 对一台失联主机做故障诊断并排序原因假设
EZcfg diagnose topology.graphml --host 192.168.1.20

# ask the AI to explain a finding and draft a fix
# 让 AI 解释一条发现并起草处置方案
EZcfg ai explain topology.graphml --finding dup-ip-12
EZcfg ai fix topology.graphml --finding dup-ip-12 --dry-run

# pull running config and syslog from a device, then ask the AI
# 抓取设备运行配置与日志，再交由 AI 分析
EZcfg collect 192.168.1.1 --config --syslog --hours 2
EZcfg ai case ./cases/2026-01-19-vlan20-outage.yaml

# diff two scans to see what moved
# 对比两次扫描，查看变更
EZcfg diff scan-jan12.graphml scan-jan19.graphml
```

### Docker (headless) · 容器（无头模式，仅扫描）

```bash
docker run --rm --net=host \
  -v $PWD/out:/out \
  EZcfg/EZcfg scan 192.168.1.0/24 \
  --snmp-community-file /out/.snmp \
  --out /out/topology.graphml

# collect configs and alerts, then analyse with the AI
# 采集配置与告警，再交由 AI 分析
docker run --rm \
  -v $PWD/out:/out \
  EZcfg/EZcfg collect 192.168.1.1 \
  --config --syslog --hours 2 \
  --out /out/device01.yaml
```

---

## What the audit engine checks · 审计规则库

| Category · 类别 | Rules · 检查项 | Severity · 级别 |
|:---|:---|:---:|
| Topology · 拓扑 | no redundant path for a core switch, single upstream gateway, orphan switch | high |
| Loops & STP · 环路 | blocked STP port in a topology without a planned blocked link, root bridge priority below 4096 | high / medium |
| Addressing · 地址 | duplicate IP, overlapping DHCP scopes, IP outside subnet mask, gateway not in subnet | high / medium |
| VLAN & trunk · VLAN | VLAN allowed on a trunk but absent from the VLAN database, native VLAN mismatch, voice VLAN on an access port | medium |
| Availability · 可用性 | SNMP unreachable, interface error counter growth, CPU or memory above threshold | medium |
| Config drift · 配置漂移 | running-config differs from startup-config, unexpected change against baseline | high / medium |
| Security posture · 安全 | default SNMP community, Telnet/HTTP exposed, SSHv1, unused ports in administrative up | high / medium |
| Operations · 运维 | hostname mismatch across scans, no description on uplink ports | low |

Every rule carries the exact commands used to verify it, so the output is actionable rather than a list of warnings. Rules are versioned: re-running an old scan against a newer ruleset is supported. AI suggestions are layered on top of the ruleset and never replace it.

每条规则都附带验证所用命令，结论可直接执行而非一堆告警。规则带版本号，可用新规则集重评历史扫描。AI 建议叠加在规则之上，不替代规则引擎。

---

## Troubleshooting workflow · 故障定位流程

1. **Capture state** — run a scan, or open the most recent `.EZcfg` project · **采集状态**：执行扫描，或打开最近的 `.EZcfg` 工程
2. **Define the symptom** — failed host, unreachable subnet, slow link, a flapping port, or a pasted alert · **定义症状**：失联主机、不可达网段、链路变慢、端口频繁震荡，或粘贴一条原始告警
3. **Choose the engine** — **Trace path** for two known endpoints, **Diagnose** for an observed symptom, **Alert triage** for incoming events · **选择引擎**：已知两端用「追踪路径」，已知症状用「诊断推理」，实时事件用「告警分诊」
4. **Read the ranked hypotheses** — each hypothesis names the device, interface and evidence, with the command to confirm · **查看排序假设**：每条假设注明设备、接口与证据，附确认命令
5. **Hand off to AI** — click **Explain** for a plain-language summary, **Fix it** for a remediation draft, or **Draft config** when the device is unfamiliar · **交予 AI**：点 **Explain** 看通俗解释，**Fix it** 看处置草稿，设备不熟时点 **Draft config**
6. **Verify on the device** — run the printed command, then mark the hypothesis confirmed or dismissed · **上机验证**：执行打印出的命令，标记假设成立或排除
7. **Rollback if needed** — every confirmed change records the pre-change snapshot and the commands applied · **必要时回退**：每次确认的变更都记录变更前快照与已执行命令
8. **Promote to a rule** — confirmed root causes and working fixes can be saved as a custom audit rule or a reusable playbook · **沉淀为规则**：已确认的根因与有效处置可保存为自定义审计规则或可复用预案

---

## Architecture · 架构

```
┌──────────────────────────────────────────────────────────────────┐
│                         GUI / Web UI /界面                        │
│   画布 · 一键诊断 · AI 面板 · 案例与报告                              │
│  canvas · one-click diagnosis · AI panel · case & report         │
├──────────────────────────────────────────────────────────────────┤
│                        AI Assistant / AI助手                      │
│   上下文构建 · 自带 API + 云端中继 · 提示词库                         │
│  context builder · BYO-API + cloud relay · prompt library        │
├──────────────────────────────────────────────────────────────────┤
│                Audit & Diagnose & Config copilot                 │
│  规则引擎 · 路径追踪 · 假设排序 · 配置校验 · 回滚管理                   │
│  rule engine · path tracer · hypothesis ranker · config          │
│  validator · rollback manager                                    │
├──────────────────────────────────────────────────────────────────┤
│                      Topology Core                               │
│  图模型 · 布局 · 差异对比 · GraphML/JSON/PNG/SVG/PDF 导出            │
│  graph model · layout · diff · GraphML/JSON/PNG/SVG/PDF          │
├──────────────────────────────────────────────────────────────────┤
│                     Discovery Layer                              │
│  ARP/ND 扫描 · LLDP/CDP · SNMPv2c/v3 · SSH/SFTP ·                 │
│  syslog 与 trap 接收 · 配置与告警采集                                │
│  ARP/ND sweep · LLDP/CDP · SNMPv2c/v3 · SSH/SFTP ·               │
│  syslog & trap receiver · config & alert collection              │
├──────────────────────────────────────────────────────────────────┤
│                       Data Sources                               │
│  交换机 · 路由器 · 防火墙 · 无线控制器与 AP · 主机                     │
│  switches · routers · firewalls · WLC/APs · hosts                │
└──────────────────────────────────────────────────────────────────┘
```

| Component · 组件 | Language · 语言 | Responsibility · 职责 |
|:---|:---|:---|
| Discovery layer<br>探测层 | Python | 报文探测、SNMP Walk、SSH/SFTP、日志采集<br>packet probing, SNMP walks, SSH/SFTP, log collection |
| Topology core<br>拓扑核心 | Python + Rust extension<br>Python + Rust 扩展 | 图模型、布局、差异对比、导出<br>graph model, layout, diff, exports |
| Audit & diagnose<br>审计与诊断 | Python + YAML | 规则求值、路径追踪、假设排序<br>rule evaluation, path tracing, hypothesis ranking |
| Config copilot<br>配置助手 | Python + Jinja2 | 厂商模板、语法校验、回滚<br>vendor templates, syntax validation, rollback |
| AI assistant<br>AI 助手 | Python + pluggable providers<br>Python + 可插拔 Provider | 上下文提取、提示词组装、响应解析<br>context extraction, prompt assembly, response parsing |
| GUI | Tauri or Qt | 画布、一键流程、AI 面板、报告查看<br>canvas, one-click flow, AI panel, report viewer |
| Web UI | React + FastAPI | 团队共享扫描结果、基于角色的访问控制<br>shared scan result for a team, role-based access |

---

## Roadmap · 路线图

- [x] GraphML / JSON export · GraphML / JSON 导出
- [x] Command-line scanner and headless Docker image · 命令行扫描器与无头容器镜像
- [ ] Desktop GUI with drag-and-drop canvas and one-click diagnosis · 带拖拽画布与一键诊断的桌面端
- [ ] LLDP/CDP plus SNMP Bridge-MIB discovery · LLDP/CDP 与 SNMP Bridge-MIB 探测
- [ ] Built-in audit ruleset and baseline diff · 内置审计规则集与基线差异
- [ ] Path tracing and symptom-based diagnosis · 路径追踪与症状诊断
- [ ] Config and alert collection (SSH/SFTP, syslog, SNMP trap) · 配置与告警采集
- [ ] AI assistant: cloud AI and BYO-API channels · AI 助手：云端 AI 与自带 API 双通道
- [ ] Configuration copilot with rollback snapshots · 带回退快照的配置助手
- [ ] Local web UI for team sharing · 面向团队协作的本地 Web UI

---

## Documentation · 文档

- `docs/discovery.md` — which MIBs and neighbour protocols each vendor exposes, and the exact switch-side commands to enable them
- `docs/discovery.md` — 各厂商设备暴露的 MIB 与邻居协议，以及设备上开启采集所需的命令
- `docs/ai.md` — AI channels, data minimisation rules, prompt library, supported self-hosted endpoints
- `docs/ai.md` — AI 通道、数据最小化规则、提示词库、支持的私有化部署端点
- `docs/copilot.md` — configuration copilot workflow, vendor templates, rollback behaviour
- `docs/copilot.md` — 配置助手工作流、厂商模板、回退机制
- `docs/rules.md` — how to write a rule, field reference, severity policy
- `docs/rules.md` — 规则编写方法、字段说明与严重级别策略
- `docs/architecture.md` — data model, graph schema, plugin interface
- `docs/architecture.md` — 数据模型、图结构与插件接口
- `docs/rfc5737.md` — documentation IP ranges used in examples
- `docs/rfc5737.md` — 示例中使用的文档保留地址段

---

## Contributing · 贡献

Vendor support lives or dies by the quality of its probes, so the most useful contributions are device tests: capture a real `lldpRemTable` walk, an SNMP walk and the output of `show lldp neighbor`, scrub the IPs and hostnames, and open a pull request under `tests/fixtures/<vendor>/`. New audit rules go in `rules/` as YAML and must include the verification command. Prompt improvements go in `prompts/` and must state which model they were tested against.

厂商支持的优劣取决于探测采集的准确性，因此最有价值的贡献是设备实测数据：抓取一份真实的 `lldpRemTable` 取值、SNMP walk 结果和 `show lldp neighbor` 输出，脱敏其中的 IP 与主机名，以 PR 形式提交到 `tests/fixtures/<厂商>/`。新增审计规则放在 `rules/` 目录下，使用 YAML 格式并须包含验证命令。提示词改进放在 `prompts/` 目录下，并须注明测试所用的模型。

---

## Security and privacy · 安全与隐私

Credentials are stored in the system keyring where available, and otherwise in an encrypted local file. Scan packets, topology graphs and configuration snippets never leave the machine except through the channels you configure (cloud AI, remote syslog, or a team server you operate). The audit log records every AI-suggested command and who confirmed it.

凭据优先存入系统密钥环，否则存入本地加密文件。扫描报文、拓扑图与配置片段不会离开本机，除非经由你配置的通道（云端 AI、远程 syslog 或你自行运营的团队服务器）。审计日志记录每条 AI 建议命令及其确认人。

---

## Disclaimer · 免责声明

EZcfg is a diagnostic and operational aid. It does not replace a qualified network engineer. Configuration changes carry risk; always test in a lab or on a staging device first, maintain a current backup and a rollback path, and follow your organisation's change management process. The EZcfg project is not liable for any damage arising from the use of this software, whether AI-generated or not.

EZcfg 是诊断与运维辅助工具，不替代具备资质的网络工程师。配置变更存在风险，请先在实验室或预发设备上验证，保留最新备份与回退路径，并遵循所在组织的变更管理流程。无论是否由 AI 生成，EZcfg 项目均不对使用本软件所造成的任何损失承担责任。

---

## License · 许可证

Released under the [MIT License](LICENSE).

以 [MIT 许可证](LICENSE) 发布。
