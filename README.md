 # Linux 一键安装 Clash

> **Fork 说明**：本仓库 fork 自 [nelvko/clash-for-linux-install](https://github.com/nelvko/clash-for-linux-install)，在原有功能上增加了：端口/Mixin 覆盖说明、**节点列表与延迟/质量显示**（`clash node list` / `clash node test`）、节点切换与自动选择（`clash node use` / `clash node auto`）、节点名称标识说明（流量倍率、家宽）等。上游更新可随时通过 `git fetch origin` 合并。

![GitHub License](https://img.shields.io/github/license/nelvko/clash-for-linux-install)
![GitHub top language](https://img.shields.io/github/languages/top/nelvko/clash-for-linux-install)
![GitHub Repo stars](https://img.shields.io/github/stars/nelvko/clash-for-linux-install)

![preview](resources/preview.png)

- 默认安装 `mihomo` 内核，[可选安装](https://github.com/nelvko/clash-for-linux-install/wiki/FAQ#%E5%AE%89%E8%A3%85-clash-%E5%86%85%E6%A0%B8) `clash`。
- 自动使用 [subconverter](https://github.com/tindy2013/subconverter) 进行本地订阅转换。
- 多架构支持，适配主流 `Linux` 发行版：`CentOS 7.6`、`Debian 12`、`Ubuntu 24.04.1 LTS`。

## 快速开始

### 环境要求

- 用户权限：`root`、`sudo`。（无权限可参考：[#91](https://github.com/nelvko/clash-for-linux-install/issues/91)）
- `shell` 支持：`bash`、`zsh`、`fish`。

### 一键安装

下述命令适用于 `x86_64` 架构，其他架构请戳：[一键安装-多架构](https://github.com/nelvko/clash-for-linux-install/wiki#%E4%B8%80%E9%94%AE%E5%AE%89%E8%A3%85-%E5%A4%9A%E6%9E%B6%E6%9E%84)

```bash
git clone --branch master --depth 1 https://gh-proxy.com/https://github.com/nelvko/clash-for-linux-install.git \
  && cd clash-for-linux-install \
  && sudo bash install.sh
```

> 如遇问题，请在查阅[常见问题](https://github.com/nelvko/clash-for-linux-install/wiki/FAQ)及 [issue](https://github.com/nelvko/clash-for-linux-install/issues?q=is%3Aissue) 未果后进行反馈。

- 上述克隆命令使用了[加速前缀](https://gh-proxy.com/)，如失效请更换其他[可用链接](https://ghproxy.link/)。
- 默认通过远程订阅获取配置进行安装，本地配置安装详见：[#39](https://github.com/nelvko/clash-for-linux-install/issues/39)
- 没有订阅？[click me](https://次元.net/auth/register?code=oUbI)

### 源码更新应用
Everytime when the source code is updated, have to run the following code to apply the change.
```bash
# Copy the APP to OS root 
sudo cp /media/zoujd4/DATA1/Users/zoujd4/Zoujd_IMI/f0_Environment/clash-for-linux-install/script/clashctl.sh /opt/clash/script/clashctl.sh
# Configure the APP entry point
source /opt/clash/script/clashctl.sh
```

### 命令一览

执行 `clashctl` 列出开箱即用的快捷命令。

> 同 `clash`、`mihomo`、`mihomoctl`

```bash
$ clashctl
Usage:
    clash     COMMAND [OPTION]
    
Commands:
    on                   开启代理
    off                  关闭代理
    ui                   面板地址
    status               内核状况
    proxy    [on|off]    系统代理
    tun      [on|off]    Tun 模式
    mixin    [-e|-r]     Mixin 配置
    secret   [SECRET]    Web 密钥
    update   [auto|log]  更新订阅
    node     list|test|use|auto  节点列表 / 测速 / 切换
```

### 优雅启停

```bash
$ clashon
😼 已开启代理环境

$ clashoff
😼 已关闭代理环境
```
- 启停代理内核的同时，设置系统代理。
- 亦可通过 `clashproxy` 单独控制系统代理。

### Web 控制台

```bash
$ clashui
╔═══════════════════════════════════════════════╗
║                😼 Web 控制台                  ║
║═══════════════════════════════════════════════║
║                                               ║
║     🔓 注意放行端口：9090                      ║
║     🏠 内网：http://192.168.0.1:9090/ui       ║
║     🌏 公网：http://255.255.255.255:9090/ui   ║
║     ☁️ 公共：http://board.zash.run.place      ║
║                                               ║
╚═══════════════════════════════════════════════╝

$ clashsecret 666
😼 密钥更新成功，已重启生效

$ clashsecret
😼 当前密钥：666
```

- 通过浏览器打开 Web 控制台，实现可视化操作：切换节点、查看日志等。
- 控制台密钥默认为空，若暴露到公网使用建议更新密钥。

### 更新订阅

```bash
$ clashupdate https://example.com
👌 正在下载：原配置已备份...
🍃 下载成功：内核验证配置...
🍃 订阅更新成功

$ clashupdate auto [url]
😼 已设置定时更新订阅

$ clashupdate log
✅ [2025-02-23 22:45:23] 订阅更新成功：https://example.com
```

- `clashupdate` 会记住上次更新成功的订阅链接，后续执行无需再指定。
- 可通过 `crontab -e` 修改定时更新频率及订阅链接。
- 通过配置文件进行更新：[pr#24](https://github.com/nelvko/clash-for-linux-install/pull/24#issuecomment-2565054701)

### `Tun` 模式

```bash
$ clashtun
😾 Tun 状态：关闭

$ clashtun on
😼 Tun 模式已开启
```

- 作用：实现本机及 `Docker` 等容器的所有流量路由到 `clash` 代理、DNS 劫持等。
- 原理：[clash-verge-rev](https://www.clashverge.dev/guide/term.html#tun)、 [clash.wiki](https://clash.wiki/premium/tun-device.html)。
- 注意事项：[#100](https://github.com/nelvko/clash-for-linux-install/issues/100#issuecomment-2782680205)

### `Mixin` 配置

```bash
$ clashmixin
😼 less 查看 mixin 配置

$ clashmixin -e
😼 vim 编辑 mixin 配置

$ clashmixin -r
😼 less 查看 运行时 配置
```

- 持久化：将自定义配置写在 `Mixin` 而不是原配置中，可避免更新订阅后丢失自定义配置。
- 运行时配置是订阅配置和 `Mixin` 配置的并集。
- 相同配置项优先级：`Mixin` 配置 > 订阅配置。

### 修改订阅自带的端口 / 控制台

订阅里自带的 **代理端口**（终端里 `http_proxy` 用的）和 **Web 控制台端口** 会被 **Mixin** 覆盖，因此要“切换”成你想要的端口，只需改 Mixin 再重启即可。

1. **编辑 Mixin**（安装后实际生效的是 `/opt/clash/mixin.yaml`）：
   ```bash
   clashmixin -e
   ```
2. **在 Mixin 里设置（覆盖订阅）：**
   - **代理端口**：增加或取消注释 `mixed-port: 7890`，把 `7890` 改成你想要的端口（终端代理用这个端口）。
   - **Web 控制台端口**：修改 `external-controller: "0.0.0.0:9090"` 里的 `9090` 为你要的端口。
3. **保存并退出**，脚本会自动合并配置并重启；也可手动执行 `clashrestart`。

若希望以后重装也使用同一端口，可修改本仓库里的 `resources/mixin.yaml` 再执行安装/更新。

### 切换订阅节点（地区/“线路”）

订阅里通常包含多地区节点（如香港、新加坡、日本、美国等）。

1. **列出所有节点及延迟/质量**
   ```bash
   clash node list
   ```
   输出节点列表、当前选中节点（带 ◀ 标记）、延迟（ms）和质量星级：

   | 延迟 | 质量 |
   |------|------|
   | < 100ms | ★★★★★ |
   | 100–200ms | ★★★★☆ |
   | 200–350ms | ★★★☆☆ |
   | 350–600ms | ★★☆☆☆ |
   | > 600ms | ★☆☆☆☆ |
   | 未测试 | `-` |

2. **测试所有节点延迟（刷新）**
   ```bash
   clash node test
   ```
   向所有节点发起延迟测试，完成后自动显示更新后的列表和延迟。首次 `list` 显示 `-` 时先跑一次 `test`。

3. **切换到指定节点（如新加坡）**
   ```bash
   clash node use "新加坡 01"
   ```
   节点名必须与 `clash node list` 里显示的完全一致（含空格、数字），建议用双引号包住。

4. **切回自动选择（按延迟自动选）**
   ```bash
   clash node auto
   ```
   会选回列表里的第一项（一般为「♻️ 自动选择」或订阅里的自动/测速项）。

也可在 Web 控制台（`clashui`）里用界面点选节点；上述命令与界面操作等价。

**节点名称里常见标识说明：**

- **流量倍率（如 0.3X、1X）**  
  - **0.3X**：该节点流量按 **30%** 计入套餐用量。即你实际用 100GB，只扣 30GB 额度，更省套餐。  
  - **1X**：按实际流量 1:1 计费。  
  - 倍率小于 1 的节点对用户更划算；大于 1 则更耗额度。

- **家宽（家庭宽带 / 住宅 IP）**  
  - 表示该节点出口是**家庭宽带**（住宅 IP），而不是机房/数据中心 IP。  
  - 部分流媒体、网站对住宅 IP 限制更松或只认住宅 IP，家宽节点有时更适合看剧、解锁地区内容等；速度与稳定性因线路而异。

### 卸载

```bash
sudo bash uninstall.sh
```

## 常见问题

[wiki](https://github.com/nelvko/clash-for-linux-install/wiki/FAQ)

### 快速故障排除

**问题：VPN 不工作，IP 地址没有变化**
```bash
# 1. 检查服务状态
clashstatus

# 2. 如果服务未运行，启动服务
clashon

# 3. 检查代理环境变量
echo "http_proxy: $http_proxy"
echo "https_proxy: $https_proxy"

# 4. 如果变量为空，重启服务
clashoff && clashon

# 5. 测试 VPN 连接
curl -s --proxy http://127.0.0.1:7890 https://api.ipify.org
```

**问题：订阅更新失败**
```bash
# 1. 检查订阅链接是否可访问
curl -I "your-subscription-url"

# 2. 手动更新订阅
clashupdate "your-subscription-url"

# 3. 更新后重启服务
clashrestart

# 4. 检查配置是否正确加载
clashui
```

**问题：clashstatus 命令卡住**
- 已修复：`clashstatus` 现在使用 `--no-pager` 参数，不会卡在分页器中
- 如果仍有问题，可以使用：`sudo systemctl status mihomo --no-pager`

## 引用

- [Clash 知识库](https://clash.wiki/)
- [Clash 家族下载](https://www.clash.la/releases/)
- [Clash Premium 2023.08.17](https://downloads.clash.wiki/ClashPremium/)
- [mihomo v1.19.2](https://github.com/MetaCubeX/mihomo)
- [subconverter v0.9.0：本地订阅转换](https://github.com/tindy2013/subconverter)
- [yacd v0.3.8：Web 控制台](https://github.com/haishanh/yacd)
- [yq v4.45.1：处理 yaml](https://github.com/mikefarah/yq)

## Star History

<a href="https://www.star-history.com/#nelvko/clash-for-linux-install&Date">
 <picture>
   <source media="(prefers-color-scheme: dark)" srcset="https://api.star-history.com/svg?repos=nelvko/clash-for-linux-install&type=Date&theme=dark" />
   <source media="(prefers-color-scheme: light)" srcset="https://api.star-history.com/svg?repos=nelvko/clash-for-linux-install&type=Date" />
   <img alt="Star History Chart" src="https://api.star-history.com/svg?repos=nelvko/clash-for-linux-install&type=Date" />
 </picture>
</a>

## Thanks

[@鑫哥](https://github.com/TrackRay)

## 特别声明

1. 编写本项目主要目的为学习和研究 `Shell` 编程，不得将本项目中任何内容用于违反国家/地区/组织等的法律法规或相关规定的其他用途。
2. 本项目保留随时对免责声明进行补充或更改的权利，直接或间接使用本项目内容的个人或组织，视为接受本项目的特别声明。

## 主要特性

- ✅ **一键安装**：支持多种 Linux 发行版，自动配置系统代理
- ✅ **智能订阅转换**：自动处理 Base64 编码和多种代理协议格式
- ✅ **Web 控制台**：可视化界面管理代理节点和配置
- ✅ **Tun 模式**：支持全局代理，包括 Docker 容器
- ✅ **自动更新**：支持定时更新订阅配置
- ✅ **故障排除**：内置诊断工具和详细错误提示
- ✅ **多架构支持**：x86_64、ARM64 等主流架构

## 更新日志

### Base64 订阅支持增强 (2025-09-09)

**问题描述：**
部分订阅链接返回 Base64 编码的代理服务器列表而非标准 Clash YAML 配置，导致订阅更新失败并出现以下错误：
```
yaml: unmarshal errors: line 1: cannot unmarshal !!str `<html> ...` into config.RawConfig
```

**解决方案：**
增强了 `_download_raw_config()` 函数，新增以下功能：

1. **Base64 内容检测**：自动识别 Base64 编码的订阅内容
2. **自动解码**：检测到 Base64 内容时自动进行解码
3. **代理 URL 转换**：将解码后的代理 URL 列表转换为标准 Clash YAML 配置
4. **多级回退机制**：
   - 首先尝试直接使用下载的配置
   - 失败时尝试 Base64 解码
   - 再失败时尝试代理 URL 转换
   - 最后回退到 subconverter 转换

**技术实现：**
- 使用正则表达式检测 Base64 内容：`^[A-Za-z0-9+/]*={0,2}$`
- 支持多种代理协议：`trojan://`、`ss://`、`vmess://`、`vless://`、`ssr://`
- 自动生成 Clash 配置结构：端口设置、代理列表、代理组、路由规则
- 保持与现有 subconverter 的兼容性

**使用效果：**
- ✅ 支持 Base64 编码的订阅链接
- ✅ 自动处理代理 URL 列表格式
- ✅ 保持原有 subconverter 功能
- ✅ 提供更友好的错误提示和状态反馈

**示例订阅格式支持：**
```
trojan://password@server:port?allowInsecure=1#节点名称
ss://method:password@server:port#节点名称
vmess://base64-encoded-config#节点名称
```

### 运行时配置重载问题修复 (2025-09-09)

**问题描述：**
`clashupdate` 命令成功更新订阅后，代理服务器已添加到配置文件中，但 Clash 服务仍使用旧的运行时配置，导致：
- 代理服务器列表为空或未正确加载
- 流量无法通过 VPN 服务器路由
- IP 地址保持不变，VPN 功能失效

**根本原因：**
Clash 服务在订阅更新后未自动重载新的配置文件，导致运行时配置与更新后的配置文件不同步。

**解决方案：**
在 `clashupdate` 命令执行成功后，自动执行 `clashrestart` 重载配置：

```bash
# 更新订阅后自动重载配置
clashupdate [subscription-url]
clashrestart  # 自动执行，确保新配置生效
```

**验证方法：**
1. **检查 IP 地址变化**：
   ```bash
   curl -s --proxy http://127.0.0.1:7890 https://api.ipify.org
   ```

2. **测试区域限制服务**：
   ```bash
   curl -s --proxy http://127.0.0.1:7890 https://api.openai.com/v1/models
   ```

**技术细节：**
- 配置文件路径：`/opt/clash/config.yaml` (更新后的配置)
- 运行时配置：`/opt/clash/runtime.yaml` (Clash 实际使用的配置)
- 重载机制：通过 `systemctl restart mihomo.service` 实现配置同步

**使用建议：**
- 每次执行 `clashupdate` 后，建议手动执行 `clashrestart` 确保配置生效
- 可通过 `clashstatus` 检查服务状态
- 使用 `clashui` 打开 Web 界面验证代理服务器是否正确加载

### 命令优化和用户体验改进 (2025-09-09)

**clashstatus 命令优化：**
- **问题**：`clashstatus` 命令会卡在分页器中，需要手动退出
- **解决方案**：添加 `--no-pager --lines=20` 参数，直接输出到终端
- **效果**：命令执行更流畅，无需手动干预

**临时文件权限修复：**
- **问题**：`mktemp` 创建的文件在 `sudo` 环境下权限不足
- **解决方案**：使用固定路径 `/tmp/clash_temp_$$` 替代 `mktemp`
- **效果**：避免权限错误，提高订阅更新成功率

**错误处理和用户反馈：**
- 增强了错误提示的详细程度
- 添加了更多诊断信息帮助用户排查问题
- 改进了订阅更新过程的用户反馈

### 兼容性改进

**支持的订阅格式：**
- ✅ 标准 Clash YAML 配置
- ✅ Base64 编码的代理列表
- ✅ 原始代理 URL 格式（trojan、ss、vmess、vless、ssr）
- ✅ HTML 重定向页面自动处理

**系统兼容性：**
- ✅ Ubuntu 24.04 LTS
- ✅ Debian 12
- ✅ CentOS 7.6+
- ✅ 其他主流 Linux 发行版
