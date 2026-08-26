<h1 align="center">
  clashctl
</h1>

<p align="center">mihomo / clash 一键部署与管理工具</p>

<p align="center">
  <img alt="GitHub License" src="https://img.shields.io/github/license/nelvko/clash-for-linux-install" />
  <img alt="GitHub top language" src="https://img.shields.io/github/languages/top/nelvko/clash-for-linux-install" />
  <img alt="GitHub Repo stars" src="https://img.shields.io/github/stars/nelvko/clash-for-linux-install" />
  <a href="https://deepwiki.com/nelvko/clash-for-linux-install"><img src="https://deepwiki.com/badge.svg" alt="Ask DeepWiki"></a>
</p>

## 📸 Preview

![preview](preview.png)

## ✨ Features

- **开箱即用**：一键部署 `mihomo` / `clash` 内核、Web 面板及运行依赖。
- **广泛兼容**：支持 `root` / 普通用户，适配主流 `Linux` 发行版、容器环境及 `systemd` / `OpenRC` 等 `init` 系统。
- **统一管理**：通过 `clashctl` 管理代理启停、状态查看、日志追踪、Web 面板、TUN 模式、访问密钥与内核升级等。
- **订阅管理**：支持多订阅源配置、一键新增、切换、更新等，并集成 [subconverter](https://github.com/tindy2013/subconverter) 实现订阅格式转换。

## 🚀 Installation

在终端中执行以下命令即可完成安装：

```bash
git clone --branch master --depth 1 https://gh-proxy.org/https://github.com/nelvko/clash-for-linux-install.git \
  && cd clash-for-linux-install \
  && bash install.sh
```

- 上述命令使用了[加速前缀](https://gh-proxy.org/)，如失效请更换其他[可用链接](https://ghproxy.link/)。
- 可通过 `.env.install` 文件自定义安装选项。
- 没有订阅？[click me](https://次元.net/auth/register?code=oUbI)

## 🎯 Quick Start

安装完成后，即可使用 `clashctl` 管理代理：

```bash
clashctl on              # 开启代理
clashctl off             # 关闭代理
clashctl status          # 查看内核状态
clashctl ui              # 查看 Web 面板地址

clashctl sub add <url>   # 添加订阅
clashctl sub update      # 更新订阅
clashctl node            # 切换节点

clashctl -h              # 查看全部命令
```

## 🧹 Uninstall

在项目目录下执行以下命令即可干净卸载（清除内核、配置及服务）：

```bash
bash uninstall.sh
```

## 📖 Documentation

- [Usage](https://github.com/nelvko/clash-for-linux-install/wiki) — 命令用法与示例。
- [FAQ](https://github.com/nelvko/clash-for-linux-install/wiki/FAQ) — 常见问题。

## 🧭 Custom Split-Routing: Direct Access for Specific Domains (e.g. Company Intranet)

`Mixin` applies **globally to all subscriptions** — switching or updating subscriptions always re-merges the base config with your Mixin. Add a rule to `rules.prepend` to bypass the proxy for specific domains:

```yaml
proxies:
  prepend:
    - {name: MIDEA-DIRECT, type: direct, udp: true, interface-name: enp130s0} # bind to the company NIC

rules:
  prepend:
    - DOMAIN-KEYWORD,midea,MIDEA-DIRECT # any domain containing "midea" goes direct via the company NIC
```

- `DOMAIN-KEYWORD` matches a keyword in the domain name (hits `aimp.midea.com`, `xxx.midea.com.cn`, etc.), but does **not** match URL paths.
- `prepend` rules/proxies are placed before the subscription's own entries and are matched top-down, so they take priority.
- `interface-name` forces the outbound's traffic out of the given NIC (`SO_BINDTODEVICE`), independent of the system default route. If you don't need NIC binding, just use the built-in `DIRECT` instead.

If intranet domains can only be resolved by the company DNS (symptom: intranet sites unreachable while the proxy is on, working again after `clashctl off`), also add a `dns` section to `Mixin` pointing those domains at the internal DNS servers:

```yaml
dns:
  nameserver-policy:
    "+.midea.com": ["10.156.20.35#MIDEA-DIRECT", "10.156.20.36#MIDEA-DIRECT"] # replace with your company DNS
    "+.midea.com.cn": ["10.156.20.35#MIDEA-DIRECT", "10.156.20.36#MIDEA-DIRECT"]
```

- Why: in `fake-ip` mode public DNS servers can't resolve intranet domains; `nameserver-policy` routes those lookups to the internal DNS instead.
- The `#outbound-name` suffix on a DNS server sends the query itself through that outbound. With Tun enabled, the kernel binds default egress to the default-route NIC — without this, internal DNS queries leave via the wrong interface and time out.
- Find your internal DNS servers with `resolvectl status` or `nmcli dev show <iface> | grep -i dns`.
- Editing via `clashctl mixin -e` re-merges and restarts automatically on save. Verify with: `curl -x http://127.0.0.1:7890 -I https://aimp.midea.com` — a response means split-routing works.

### Advanced: NIC-Level Split (company traffic via company network, everything else via an outside network)

On a dual-NIC setup (e.g. company ethernet + personal hotspot), make the outside network the default route and pin only the intranet prefixes to the company gateway; combined with the `interface-name` outbound above, this gives physical separation:

```bash
sudo nmcli con mod "Wired connection 1" ipv4.route-metric 700 ipv4.routes "10.0.0.0/8 10.156.64.1" # company prefixes via company gateway
sudo nmcli con mod "MIFI_2926" ipv4.route-metric 100  # personal hotspot becomes the default egress
sudo nmcli con up "MIFI_2926" && sudo nmcli con up "Wired connection 1"
```

- Result: proxy uplinks and all ordinary traffic leave via the personal hotspot — the company network sees no VPN traffic. Only midea domains (via the `MIDEA-DIRECT` bound NIC) and company intranet prefixes (static route) use the company network.
- Verify with `ss -tn`: connections from mihomo to the airport servers should show the hotspot IP as source; connections to intranet sites should show the company IP.
- Rollback: `sudo nmcli con mod "Wired connection 1" ipv4.route-metric 100 ipv4.routes ""`, set the hotspot metric back to 600, then reconnect both.

## 💖 Support

### <img alt="Maru Code" src="https://cdn.nodeimage.com/i/hc6anADTcLP0P2CTOoqUMkKcHER4KeYY.webp" width="20" height="20"> [Maru Code —— 稳定可靠的 API 中转服务](https://api.muteki.site/register?aff=NELVKO&promo=nelvko)

- ⚡ 模型能力完整，`Claude` 系列满血可用。
- 📊 计费倍率透明公开，成本更容易预估。
- 🔑 自营号池保障可用性，日常调用更稳定。
- 🎁 新用户注册赠送 `$2` 额度：👉[立即注册](https://api.muteki.site/register?aff=NELVKO&promo=nelvko)

## ⭐ Star History

<a href="https://star-history.dera.page/#nelvko/clash-for-linux-install&Date">
 <picture>
   <source media="(prefers-color-scheme: dark)" srcset="https://star-history.dera.page/svg?repos=nelvko/clash-for-linux-install&type=Date&theme=dark" />
   <source media="(prefers-color-scheme: light)" srcset="https://star-history.dera.page/svg?repos=nelvko/clash-for-linux-install&type=Date" />
   <img alt="Star History Chart" src="https://star-history.dera.page/svg?repos=nelvko/clash-for-linux-install&type=Date" />
 </picture>
</a>

## ⚠️ Disclaimer

- 编写本项目主要目的为学习和研究 `Shell` 编程，不得将本项目中任何内容用于违反国家/地区/组织等的法律法规或相关规定的其他用途。
- 本项目保留随时对免责声明进行补充或更改的权利，直接或间接使用本项目内容的个人或组织，视为接受本项目的特别声明。
