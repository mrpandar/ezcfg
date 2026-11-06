# EZcfg

> **ez network cfg** — *configuration & cartography*
> 局域网拓扑发现 · 拖拽建图 · 隐患与故障定位
> LAN topology discovery · drag-and-drop mapping · risk & fault analysis

[![MIT License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Platform](https://img.shields.io/badge/platform-Windows%20%7C%20macOS%20%7C%20Linux-lightgrey.svg)](#installation)
[![Python](https://img.shields.io/badge/python-3.10%2B-blue.svg)](#installation)

**ezcfg** is an open-source LAN topology toolkit. It finds what is actually connected to your network, lets you refine the map by hand, and tells you where the risks and the faults are — in one place, in one file.

**ezcfg** 是一款开源局域网拓扑工具：发现真实连线的设备、允许手工修正拓扑、并定位隐患与故障点，三件事在同一份文档、同一个工具里完成。

> **Not affiliated.** The name `ezcfg` is reused independently. This project has no relation to any other project of the same name, and does not generate vendor configuration files.
> **项目无关声明：** 名称 `ezcfg` 为独立复用，与同名的其它项目无任何关联；本工具不生成厂商设备配置文件。

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
| 📊 | **Exports** — GraphML, JSON, PNG/SVG and a printable PDF report; re-run the same scan later and diff against the baseline | **导出** — GraphML、JSON、PNG/SVG 以及可打印的 PDF 报告；同一网段可反复扫描并与基线做差异对比 |
| 🔌 | **Flexible access** — desktop GUI, headless CLI for cron jobs, and a local web UI for teams; one scan result is shared across all three | **部署方式** — 桌面 GUI、用于定时任务的无头 CLI、面向团队的本地 Web UI，同一份扫描结果三端通用 |
| 🔓 | **Open and auditable** — every audit rule is a readable YAML file, including the exact MIB OIDs and the remediation command it checks for | **开源可审计** — 每条审计规则都是可读的 YAML 文件，含所查 MIB OID 与对应整改命令 |

---

## How discovery works · 探测原理

```
        Layer 2 discovery                    Layer 3 + inference
┌──────────────────────────────┐    ┌────────────────────────────┐
│ 1. ARP / NDP sweep           │    │ 5. Bridge-MIB dot1dTpFdb   │
│    → alive hosts + MAC table │    │    on every switch         │
├──────────────────────────────┤    ├────────────────────────────┤
│ 2. LLDP (lldpRemTable)       │───▶│ 6. IP-MIB / ARP on routers │
│    CDP (Cisco legacy)        │    │    → resolve unknown MACs   │
│    → neighbour pairs         │    │                            │
├──────────────────────────────┤    ├────────────────────────────┤
│ 3. System description        │    │ 7. Cross-check all sources  │
│    + ifName mapping          │    │    → confidence score       │
│    → vendor / model / role   │    │    → unknown L2 segments    │
├──────────────────────────────┤    ├────────────────────────────┤
│ 4. STP state                │───▶│ 8. Diff against baseline    │
│    → loops vs redundant link │    │    → added / moved / gone   │
└──────────────────────────────┘    └────────────────────────────┘
```

**Why two layers.** LLDP and CDP only reveal *direct* neighbours, so they cannot recover a topology across an unmanaged switch or a link where neighbour discovery is disabled. Layer 3 fills the gaps: switches expose their MAC forwarding table via the Bridge MIB, and routers expose their ARP cache. Cross-checking these sources produces a confidence score on every link; an unmanaged switch or a hub, which speaks neither LLDP nor SNMP, is rendered as an **unknown L2 segment** placeholder rather than silently merged into the wrong link.

**为什么分两层。** LLDP/CDP 只能看到直连邻居，无法穿透非网管交换机或关闭了邻居发现的链路。第三层负责补洞：交换机通过 Bridge-MIB 暴露 MAC 转发表，路由器暴露 ARP 缓存。多源交叉校验后每条链路都有可信度；既无 LLDP 也无 SNMP 的非网管交换机或 Hub，会显示为**未知二层网段**占位，而不是被悄悄合并到错误的链路上。

### Requirements · 采集前提

| Prerequisite · 前提 | Required for · 用于 | Optional · 是否可选 |
|:---|:---|:---:|
| ICMP reachable seed subnet | host sweep | no |
| LLDP or CDP enabled on switches | layer-2 links | no |
| SNMPv2c read community or SNMPv3 user | bridge tables, STP, interfaces | yes* |
| SSH read-only credentials | deep audit, config backup | yes |
| SSH write credentials | automated remediation | yes |

\* Without SNMP the result degrades to a host-and-gateway map; device-to-device links cannot be resolved.

\* 无 SNMP 时只能得到主机与网关图，设备之间的链路无法还原。

---

## Installation · 安装

| Platform | Command | 说明 |
|:---|:---|:---|
| Windows | download `ezcfg-win-x64.exe` from [Releases](../../releases) | 单文件，免安装，双击运行 |
| macOS | `brew install ezcfg` or download the universal binary | 通用二进制，支持 Apple Silicon |
| Linux | `curl -sSL https://ezcfg.dev/install.sh \| sh` | 安装到 `/usr/local/bin` |
| Python | `pipx install ezcfg` | 需要 Python 3.10 及以上 |
| Docker | `docker run --rm --net=host ezcfg/ezcfg scan 192.168.1.0/24` | 无头模式，仅用于扫描 |

> **Scanning is a sensitive operation.** Only run ezcfg on networks you own or are authorised to test. Discovery packets may be logged as reconnaissance by IDS/IPS, and some printers, cameras and industrial controllers misbehave under rapid ARP or SNMP polling. Start with a small /24 and observe before widening the scope.
>
> **扫描属于敏感操作。** 仅在你拥有或获得授权的网络上运行。发现报文可能被 IDS/IPS 记录为侦察行为；部分打印机、摄像头与工业控制器在高频 ARP 或 SNMP 轮询下会出现异常。建议先扫一个 /24 观察，再逐步扩大范围。

---

## Quick start · 快速开始

### GUI — one scan, one map · 界面 — 一次扫描，一张图

1. Launch the app and open **Discovery → New scan** · 打开 **发现 → 新建扫描**
2. Enter `192.168.1.0/24`, choose SNMP profile, set threads to `32` · 填入网段，选择 SNMP 模板，并发设为 `32`
3. Wait for `Reachable hosts` and `Layer-2 links` to stabilise · 等待「可达主机」与「二层链路」数量稳定
4. Click **Build map** — the canvas opens with devices auto-arranged · 点 **生成拓扑图**，画布自动布局
5. Save as `.ezcfg` (project, keeps history) or export GraphML / PDF · 存为 `.ezcfg`（含历史），或导出 GraphML / PDF

### GUI — drawing manually · 界面 — 手工建图

1. Start a blank canvas from **File → New** · 从 **文件 → 新建** 打开空白画布
2. Drag device icons from the left stencil: switch, router, firewall, AP, server, camera, printer, generic · 从左侧图元拖出设备
3. Connect ports by dragging from one port circle to another; the link label shows the inferred VLAN · 从一个端口圆点拖到另一个端口圆点完成连线，标签自动标注推断的 VLAN
4. Right-click a link for **Trace path** or **Add to audit scope** · 右键链路可执行「追踪路径」或「加入审计范围」
5. Import a previous scan as background layer to snap your drawing onto real data · 可导入历史扫描结果作为底图，让手绘图对齐真实数据

### CLI

```bash
# scan a subnet and write a GraphML file
# 扫描网段并输出 GraphML
ezcfg scan 192.168.1.0/24 --out topology.graphml

# same scan, JSON output, low verbosity
# 同一扫描，JSON 输出，低日志级别
ezcfg scan 192.168.1.0/24 --format json --vlan 10 --quiet

# run the audit rules against an existing topology
# 对已有拓扑运行审计规则
ezcfg audit topology.graphml --rules rules/core.yaml --severity medium

# render a scan result as a PNG without running the GUI
# 将扫描结果渲染为 PNG，无需启动界面
ezcfg render topology.graphml --out map.png --layout fdp

# trace the forwarding path between two endpoints
# 追踪两个端点之间的完整转发路径
ezcfg trace topology.graphml --from 192.168.1.20 --to 10.0.0.5

# diagnose a failed host and rank the possible causes
# 对一台失联主机做故障诊断并排序原因假设
ezcfg diagnose topology.graphml --host 192.168.1.20

# diff two scans to see what moved
# 对比两次扫描，查看变更
ezcfg diff scan-jan12.graphml scan-jan19.graphml
```

### Docker (headless) · 容器（无头模式，仅扫描）

```bash
docker run --rm --net=host \
  -v $PWD/out:/out \
  ezcfg/ezcfg scan 192.168.1.0/24 \
  --snmp-community-file /out/.snmp \
  --out /out/topology.graphml
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
| Security posture · 安全 | default SNMP community, Telnet/HTTP exposed, SSHv1, unused ports in administrative up | high / medium |
| Operations · 运维 | hostname mismatch across scans, config drift against baseline, no description on uplink ports | low |

Every rule carries the exact commands used to verify it, so the output is actionable rather than a list of warnings. Rules are versioned: re-running an old scan against a newer ruleset is supported.

每条规则都附带验证所用命令，结论可直接执行而非一堆告警。规则带版本号，可用新规则集重评历史扫描。

---

## Troubleshooting workflow · 故障定位流程

1. **Capture state** — run a scan, or open the most recent `.ezcfg` project · **采集状态**：执行扫描，或打开最近的 `.ezcfg` 工程
2. **Define the symptom** — failed host, unreachable subnet, slow link, or a flapping port · **定义症状**：失联主机、不可达网段、链路变慢、端口频繁震荡
3. **Choose the engine** — **Trace path** for two known endpoints, **Diagnose** for an observed symptom · **选择引擎**：已知两端用「追踪路径」，已知症状用「诊断推理」
4. **Read the ranked hypotheses** — each hypothesis names the device, interface and evidence, with the command to confirm · **查看排序假设**：每条假设注明设备、接口与证据，附确认命令
5. **Verify on the device** — run the printed command, then mark the hypothesis confirmed or dismissed · **上机验证**：执行打印出的命令，标记假设成立或排除
6. **Promote to a rule** — confirmed root causes can be saved as a custom audit rule · **沉淀为规则**：已确认的根因可保存为自定义审计规则

---

## Architecture · 架构

```
┌──────────────────────────────────────────────────────────┐
│                     GUI / Web UI                         │
│  canvas · stencil palette · inspector · report viewer     │
├──────────────────────────────────────────────────────────┤
│                   Audit & Diagnose                        │
│  rule engine (YAML) · path tracer · hypothesis ranker     │
├──────────────────────────────────────────────────────────┤
│                    Topology Core                          │
│  graph model · layout · diff · GraphML/JSON/PNG/SVG/PDF   │
├──────────────────────────────────────────────────────────┤
│                    Discovery Layer                        │
│  ARP/ND sweep · LLDP/CDP · SNMPv2c/v3 · SSH show parsing  │
├──────────────────────────────────────────────────────────┤
│                    Data Sources                           │
│  switches · routers · firewalls · WLC/APs · hosts         │
└──────────────────────────────────────────────────────────┘
```

| Component · 组件 | Language · 语言 | Responsibility · 职责 |
|:---|:---|:---|
| Discovery layer | Python | packet probing, SNMP walks, credential handling |
| Topology core | Python + Rust extension | graph model, layout, diff, exports |
| Audit engine | Python + YAML | rule evaluation, path tracing, hypothesis ranking |
| GUI | Tauri or Qt | canvas, stencil palette, inspector, report viewer |
| Web UI | React + FastAPI | shared scan result for a team |

---

## Roadmap · 路线图

- [x] GraphML / JSON export · GraphML / JSON 导出
- [x] Command-line scanner and headless Docker image · 命令行扫描器与无头容器镜像
- [ ] Desktop GUI with drag-and-drop canvas · 带拖拽画布的桌面端
- [ ] LLDP/CDP plus SNMP Bridge-MIB discovery · LLDP/CDP 与 SNMP Bridge-MIB 探测
- [ ] Built-in audit ruleset and baseline diff · 内置审计规则集与基线差异
- [ ] Path tracing and symptom-based diagnosis · 路径追踪与症状诊断
- [ ] Local web UI for team sharing · 面向团队协作的本地 Web UI

---

## Documentation · 文档

- `docs/discovery.md` — which MIBs and neighbour protocols each vendor exposes, and the exact switch-side commands to enable them
- `docs/discovery.md` — 各厂商设备暴露的 MIB 与邻居协议，以及设备上开启采集所需的命令
- `docs/rules.md` — how to write a rule, field reference, severity policy
- `docs/rules.md` — 规则编写方法、字段说明与严重级别策略
- `docs/architecture.md` — data model, graph schema, plugin interface
- `docs/architecture.md` — 数据模型、图结构与插件接口
- `docs/rfc5737.md` — documentation IP ranges used in examples
- `docs/rfc5737.md` — 示例中使用的文档保留地址段

---

## Contributing · 贡献

Vendor support lives or dies by the quality of its probes, so the most useful contributions are device tests: capture a real `lldpRemTable` walk, an SNMP walk and the output of `show lldp neighbor`, scrub the IPs and hostnames, and open a pull request under `tests/fixtures/<vendor>/`. New audit rules go in `rules/` as YAML and must include the verification command.

厂商支持的优劣取决于探测采集的准确性，因此最有价值的贡献是设备实测数据：抓取一份真实的 `lldpRemTable` 取值、SNMP walk 结果和 `show lldp neighbor` 输出，脱敏其中的 IP 与主机名，以 PR 形式提交到 `tests/fixtures/<厂商>/`。新增审计规则放在 `rules/` 目录下，使用 YAML 格式并须包含验证命令。

---

## License · 许可证

Released under the [MIT License](LICENSE).

以 [MIT 许可证](LICENSE) 发布。
