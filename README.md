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

## 🧭 自定义分流：指定域名直连（如公司内网）

`Mixin` 对**所有订阅全局生效**（切换/更新订阅时都会与 Mixin 重新合并），在 `rules.prepend` 中添加规则即可让指定域名绕过代理直连：

```yaml
proxies:
  prepend:
    - {name: MIDEA-DIRECT, type: direct, udp: true, interface-name: enp130s0} # 绑定公司网卡

rules:
  prepend:
    - DOMAIN-KEYWORD,midea,MIDEA-DIRECT # 域名含 midea 的一律从公司网卡直连
```

- `DOMAIN-KEYWORD` 按域名关键字匹配（`aimp.midea.com`、`xxx.midea.com.cn` 均可命中），但不匹配 URL 路径。
- `prepend` 的规则/节点会排在订阅内容之前，自上而下匹配，优先生效。
- `interface-name` 让该出站的流量物理上从指定网卡发出（`SO_BINDTODEVICE`），与系统默认路由无关；不需要网卡绑定时直接用内置的 `DIRECT` 即可。

若内网域名只有公司 DNS 能解析（现象：开启代理后内网站点无法访问，`clashctl off` 后恢复），还需在 `Mixin` 中补充 `dns` 配置，指定内网 DNS：

```yaml
dns:
  nameserver-policy:
    "+.midea.com": ["10.156.20.35#MIDEA-DIRECT", "10.156.20.36#MIDEA-DIRECT"] # 替换为公司内网 DNS
    "+.midea.com.cn": ["10.156.20.35#MIDEA-DIRECT", "10.156.20.36#MIDEA-DIRECT"]
```

- 原因：`fake-ip` 模式下公共 DNS 解析不到内网域名，`nameserver-policy` 让指定域名改走内网 DNS 解析。
- DNS 服务器后的 `#出站名` 表示该 DNS 查询也从公司网卡发出；Tun 开启时内核会把默认出口绑定到默认路由网卡，不指定会导致内网 DNS 查询走错网卡超时。
- 内网 DNS 地址可通过 `resolvectl status` 或 `nmcli dev show <网卡> | grep -i dns` 查看。
- 用 `clashctl mixin -e` 编辑保存后会自动合并配置并重启生效；验证：`curl -x http://127.0.0.1:7890 -I https://aimp.midea.com` 能通即分流成功。

### 进阶：网卡级分流（公司流量走公司网，其余一切走外部网络）

双网卡场景（如公司有线 + 个人热点）下，让默认路由走外部网络、仅内网网段走公司网关，配合上面的 `interface-name` 出站即可实现物理隔离：

```bash
sudo nmcli con mod "Wired connection 1" ipv4.route-metric 700 ipv4.routes "10.0.0.0/8 10.156.64.1" # 公司网段走公司网关
sudo nmcli con mod "MIFI_2926" ipv4.route-metric 100  # 个人热点成为默认出口
sudo nmcli con up "MIFI_2926" && sudo nmcli con up "Wired connection 1"
```

- 效果：代理上行与普通流量经个人热点发出，公司看不到翻墙流量；仅 midea 域名（经 `MIDEA-DIRECT` 绑定的网卡）与公司内网网段（静态路由）走公司网络。
- 验证：`ss -tn` 观察 mihomo 到机场服务器的连接源地址应为热点 IP，到内网站点的连接源地址应为公司 IP。
- 回退：`sudo nmcli con mod "Wired connection 1" ipv4.route-metric 100 ipv4.routes ""`，并将热点 metric 调回 600 后重连。

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
