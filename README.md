# EZcfg
EZcfg – Make Network Device Configuration Effortless.​
EZcfg —— 让网络设备配置生成变得轻而易举。

**EZcfg** is a free and open-source **network device configuration generator** designed for network engineers, integrators, and O&M teams.  
It helps you automatically generate standardized CLI configurations for multi-vendor network devices (switches, routers, firewalls, wireless controllers, etc.) through simple form inputs or templates — **no more hand-crafting repetitive commands**.

**EZcfg** 是一个免费开源的**网络设备配置生成工具**，面向网络工程师、集成商和运维团队。  
通过简单的表单填写或模板导入，自动生成多厂商网络设备（交换机、路由器、防火墙、无线控制器等）的标准化 CLI 配置——**告别手工敲重复命令的时代**。

> ⚠️ **Note / 声明**  
> This project is **independent** and focuses on **network device configuration generation**.  
> It is **not affiliated with** any existing `ezcfg` system-config or dotfile-management tools on GitHub/PyPI/npm.  
> 本项目专注于**网络设备配置生成**，与 GitHub/PyPI/npm 上已有的系统配置管理类 `ezcfg` 项目**无任何关联**。

---

## ✨ Supported Devices / 支持的设备类型

| Device Type | Examples |
|-------------|----------|
| 🖥️ Switches / 交换机 | Huawei VRP, H3C Comware, Cisco IOS/IOS-XE, Ruijie OS |
| 🌐 Routers / 路由器 | Huawei AR/NE, Cisco ISR/ASR, MikroTik RouterOS |
| 🔥 Firewalls / 防火墙 | Huawei USG, H3C SecPath, Cisco ASA/FTD, Palo Alto, Fortinet |
| 📡 Wireless Controllers / 无线控制器 | Huawei AC, H3C WX, Cisco WLC, Aruba |
| 🔌 APs / 无线接入点 | Fit AP, Cloud-managed AP (basic profile) |

---

## ✨ Features / 功能特性

| English | 中文 |
|---------|------|
| 🏢 Multi-vendor Support – 10+ vendors, 20+ device families | 🏢 多厂商支持 – 10+ 厂商，20+ 设备系列 |
| ⚙️ Core Scenarios – VLAN, routing, NAT, ACL, VPN, HA, SSID | ⚙️ 核心场景 – VLAN、路由、NAT、ACL、VPN、高可用、SSID |
| 📦 Batch Generation – Generate configs for dozens of devices from Excel/JSON in minutes | 📦 批量生成 – 从 Excel/JSON 几分钟生成几十台设备配置 |
| 🧩 Template Engine – Jinja2/YAML-based, easy to extend your own templates | 🧩 模板引擎 – 基于 Jinja2/YAML，轻松扩展自定义模板 |
| 🌐 Web / CLI / GUI – Flexible deployment for personal use or team SaaS | 🌐 Web / CLI / GUI – 灵活部署，支持个人使用或团队 SaaS |
| 🔒 Safe & Transparent – Open-source, auditable, no black-box automation on live devices | 🔒 安全透明 – 开源可审计，不对在线设备进行黑盒自动化操作 |

---

## 🎯 Philosophy / 理念

> Network configuration should be **EZ (Easy)**, **Consistent**, and **Error-free**.  
> EZcfg is built to eliminate copy-paste mistakes and speed up network deployment.  
> 网络配置应该**简单、一致、零错误**。EZcfg 致力于消除复制粘贴错误，加速网络部署。

---

## 🚀 Quick Start / 快速开始
