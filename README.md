# agent-browser 从零上手：两行命令，让 AI 帮你操作浏览器

> 本教程基于 2026 年 11 月 10 日在 InstaBox 云主机（Debian 13）上的真实安装实测写成，每一步命令都实际跑通过，报错和解法都是实测记录，不是翻译文档。

## 一句话介绍

agent-browser 是 Vercel Labs 出品的一个**命令行浏览器工具**：不用写代码，不用打开浏览器窗口，你在终端里敲几行命令，它就能替你打开网页、点链接、读页面内容——最适合想让 AI 帮自己上网查资料、抓页面信息的人。

**适合人群：**

- 听说过 AI Agent，但没写过代码、没用过 Playwright 之类工具的纯新手
- 想让 AI 帮自己批量看网页、盯 GitHub 热榜、读文章的效率党
- 想在自己的云服务器上跑一个“AI 浏览器”的折腾党

不适合谁：只想在手机上点一点就完成的人——这个工具全程在终端里操作，需要一台能上网的 Linux 或 macOS 电脑（Windows 可以用 WSL）。

## 它最近有多火

- GitHub 仓库 `vercel-labs/agent-browser`，截至 2026 年 11 月 10 日约 **4.8 万 star**，**本周新增约 3,200 个 star**
- 11 月 7 日、8 日均有快照显示登上 GitHub Trending **当日榜第一名**（不同时段的榜单快照排名有出入，稳妥说法是“11 月上旬多次登顶当日榜”）
- 实测当天（11 月 10 日）刚发布了 **v0.42.0** 新版本
- 实测中打开 `github.com/trending` 时，它自己就出现在当日热榜里

也就是说：这是眼下最受关注的 AI 浏览器工具之一，而中文世界里系统讲它怎么装、怎么用的教程还很少。

## 环境要求

实测环境（照着这个配，基本不会踩坑）：

| 项目 | 实测值 |
|---|---|
| 系统 | Debian 13（x86_64），一台能上网的 Linux 云主机即可 |
| Node.js | v22.23.3 |
| npm | 10.9.9 |
| 内存 | 8 GiB（实测空闲 7.2 GiB） |
| 磁盘 | 安装总共约占用 0.5G |

**装之前先检查一下：** 打开终端，依次运行这两条命令，能看到版本号就说明环境没问题：

```bash
node --version
npm --version
```

如果提示 `command not found`，说明你的机器还没装 Node.js，先去 [Node.js 官网](https://nodejs.org/)装一个 LTS（长期支持）版本，再回来继续。

## 一键安装（就两条命令）

```bash
npm install -g agent-browser
agent-browser install
```

第一条装命令行工具本身，第二条下载它要用的 Chrome 浏览器。实测全程约 1 分钟，其中下载 Chrome 占了大部分时间（194 MB）。

## 完整步骤

### 第 1 步：装命令行工具

```bash
npm install -g agent-browser
```

成功的话会看到类似这样的输出：

```
added 1 package in 3s
```

3 秒就装完了。如果这一步提示权限不足（`EACCES` 之类的字样），说明你的 npm 全局目录需要管理员权限，改成 `sudo npm install -g agent-browser` 再试一次。实测的云主机已经配好了全局目录，不用 sudo 也能装。

### 第 2 步：装 Chrome 浏览器

```bash
agent-browser install
```

成功的话会看到：

```
Installing Chrome...
  Downloading Chrome 157.0.8103.50 for linux64
✓ Chrome 157.0.8103.50 installed successfully
Location: /data/home/.agent-browser/browsers/chrome-157.0.8103.50
Download 194/194 MB (100%)
```

注意两点：

1. 这一步会先冒出一行以 `⚠` 开头的黄色提示（见下面“实测报错库”），**那不是报错**，不用管它，继续等下载完成就行。
2. Chrome 默认装在你用户目录下的 `.agent-browser/browsers/` 里，实测占用 **409 MB**；命令行工具本身占 **128 MB**。两样加起来约 0.5G 磁盘。

### 第 3 步：打开第一个网页

```bash
agent-browser open example.com
```

成功输出：

```
[agent-browser] launched browser
✓ Example Domain
https://example.com/
```

看到 `✓ Example Domain`，说明浏览器已经在后台启动、页面也打开了。你不需要、也看不到浏览器窗口——它是在后台无头运行的。

### 第 4 步：看页面上有什么（snapshot）

```bash
agent-browser snapshot -i
```

这是 agent-browser 最核心的命令：它把当前页面变成一张“元素清单”，每个能点的东西都有一个编号（`ref=e1`、`ref=e2`……），之后点哪个就报哪个编号。

example.com 页面很简单，只返回 1 个元素：

```
- link "Learn more" [ref=e1]
```

别慌，元素少不是出问题了——这个页面本来就只有一个链接。

### 第 5 步：确认当前网址，关掉浏览器

```bash
agent-browser get url
agent-browser close
```

输出：

```
https://example.com/
✓ Browser closed
```

到这里，最小的一套流程就跑通了：打开 → 看元素 → 关掉。

### 第 6 步：来个真实的——打开 GitHub Trending

```bash
agent-browser open https://github.com/trending
agent-browser snapshot -i
agent-browser close
```

这次页面内容丰富，`snapshot -i` 一口气返回了约 **150 个可交互元素**，包括每个仓库的名字、star 数、fork 数，例如：

```
- heading "vercel-labs / agent-browser" [level=2, ref=e37]
- heading "ZhuLinsen / daily_stock_analysis" [level=2, ref=e44]
- link "star 69,103" [ref=e47]
```

看到这份清单，就说明 agent-browser 已经能正常读取真实网站的完整页面结构了。接下来你可以用 `click @e37` 之类的命令点进某个仓库——玩法就从这里展开。

## 实测报错库

本次实测在 InstaBox 云主机上**全程零报错**，一次就装通了。但下面这个提示最容易让新手误判，特意记下来：

### 唯一的“坑”：安装时那行黄色警告不是报错

**报错（提示）原文：**

```
⚠ Linux detected. If browser fails to launch, run:
  agent-browser install --with-deps
```

以及安装成功后还会再出现一次：

```
Note: If you see "shared library" errors when running, use:
agent-browser install --with-deps
```

**原因：** 这是安装程序在 Linux 系统上的预防性提示，不是安装失败。意思是：某些特别精简的 Linux 系统缺少 Chrome 需要的共享库（shared library），到时候浏览器可能启动不了。

**解法：** 实测的 Debian 13 上什么都不用做，浏览器直接就能启动。**只有当你后面运行 `agent-browser open` 时真的看到 "shared library" 相关的错误**，才需要补跑这条命令：

```bash
agent-browser install --with-deps
```

这条命令会顺带装上系统缺的依赖库，个别系统上可能需要 `sudo`。

**新手最容易犯的错：** 看到 `⚠` 就以为装失败了，跑去重装、换源、删目录——其实完全没必要，等下载跑完 `✓ Chrome … installed successfully` 就是成功了。

### 如果 snapshot 只返回 1 个元素，是出问题了吗？

不是。`snapshot -i` 返回多少元素，完全取决于页面本身有多少可交互内容。example.com 只有一个链接，就只返回 1 个；GitHub Trending 内容多，就返回约 150 个。元素少≠安装失败。

## 冒烟验证方法（装完照着跑一遍）

装完后用下面 5 条命令自测，全部出现 `✓` 就说明装通了：

```bash
agent-browser open example.com        # 应看到 ✓ Example Domain
agent-browser snapshot -i             # 应看到 - link "Learn more" [ref=e1]
agent-browser get url                 # 应看到 https://example.com/
agent-browser close                   # 应看到 ✓ Browser closed
agent-browser open https://github.com/trending   # 再开一个真实页面验证
agent-browser snapshot -i             # 应看到约 150 个元素
agent-browser close
```

## 卸载清理

如果以后不想用了，两步清干净：

```bash
npm uninstall -g agent-browser
rm -rf ~/.agent-browser
```

第一条卸掉命令行工具，第二条删掉它下载的 Chrome（约 409 MB）。实测中 Chrome 装在 `~/.agent-browser/browsers/` 下（云主机上实际路径是 `/data/home/.agent-browser/`），删这个目录就够了，不用去动 `~/.cache` 里的东西——实测确认 agent-browser 没有往 `~/.cache/puppeteer` 或 `~/.cache/ms-playwright` 里写任何文件。

> 说明：卸载步骤按常规方式给出，本次实测没有实际执行卸载。

## 成本与账号说明

- **不用 GitHub 账号。** 安装和上面的全部操作都不需要登录 GitHub；GitHub Trending、仓库页面都是公开页面，不登录也能打开。实测全程没有登录任何账号。
- **不用付费、不用额度。** agent-browser 本身是开源免费工具（Apache-2.0 协议），下载的 Chrome for Testing 也是免费的，没有任何按量计费或 API 额度。
- **唯一的成本是机器和流量：** 一台能跑 Node.js 的 Linux 主机（实测用的免费云沙箱，4 核 8G），加上安装时下载约 194 MB 的 Chrome，以及约 0.5G 磁盘空间。

## 实测数据卡片

| 项目 | 实测值（2026-11-10） |
|---|---|
| 安装耗时 | 约 1 分钟（npm 3 秒 + 下载 Chrome 约 1 分钟） |
| 全程耗时 | 约 8 分钟（含冒烟测试和等待浏览器启动） |
| 遇到报错 | 0 个（1 条预防性警告，见报错库） |
| Chrome 版本 | 157.0.8103.50（194 MB，解压后 409 MB） |
| agent-browser 包体积 | 128 MB |
| 磁盘总占用 | 约 0.5G（`/` +0.1G，`/data` +0.4G） |
| 系统 | Debian 13，Node v22.23.3，npm 10.9.9 |
