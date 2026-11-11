# 让 WorkBuddy 自己上网干活：agent-browser 接入实测

> 本篇是《agent-browser 从零上手》系列第 2 篇。安装部分是 2026-11-10 在一台没有网络的 Windows 电脑上真机跑通的；WorkBuddy 里配 MCP、点“信任”这两步是按 WorkBuddy 官方 MCP 文档写的，**还没在作者的 WorkBuddy 上真连过**——照着走、卡住就按文末的坑逐条排。

## 一句话

WorkBuddy 是大脑，agent-browser 是手。接上之后，你在 WorkBuddy 里说一句人话，它自己打开浏览器、看页面、点按钮，把结果带回来——全程不用你敲命令，也不用花一分钱 key。

**适合谁：** 已经在用 WorkBuddy（或 Claude Code、Cursor）的人，手里有大脑、正缺一双能上网的手。
**不适合谁：** 还没装 WorkBuddy 的人——先装它，再回来接这篇。

## 先说清两件事，别混在一起

1. **装 agent-browser**：装在你自己的电脑上，不是在云主机上。WorkBuddy 是桌面端，它调的工具也跑在本机。
2. **在 WorkBuddy 里接 MCP**：告诉 WorkBuddy去哪儿找这双手，点一次“信任”启用。

两件事都通了，才算接通。第一件我们今天已经在无网的 Windows 电脑上一次装通了，第二件按官方文档走。

## 前置条件（都满足再往下走）

- 电脑上已装好 agent-browser（二选一）：
  - 有网：`npm install -g agent-browser`，再 `agent-browser install`
  - 没网：用我们做的 Windows 完全离线安装包，解压后双击 `install.bat` 就装好（包里自带 Chrome for Testing，不用再下载）
- 电脑上已装好 WorkBuddy
- 装完先验一下手是好的，在终端里跑：

```bash
agent-browser open baidu.com
```

看到 `✓ 百度一下，你就知道` 加网址，就是通了。**没弹出浏览器窗口是正常的**：它默认无头模式，在后台干活，本来就不露面。想看它在看什么，跑 `agent-browser snapshot -i` 看元素清单，或 `agent-browser screenshot baidu.png` 拍张照。

## 第 1 步：把 MCP 配置写进 WorkBuddy

WorkBuddy 用标准 MCP 格式，编辑它的配置文件（没有就新建一个）：

- Windows：`%USERPROFILE%\.workbuddy\mcp.json`
- macOS / Linux：`~/.workbuddy/mcp.json`

加进这一段：

```json
{
  "mcpServers": {
    "agent-browser": {
      "command": "agent-browser",
      "args": ["mcp"]
    }
  }
}
```

原理就一句：`agent-browser mcp` 会在本地起一个标准 MCP 服务，WorkBuddy 连上它，就多了打开网页、点元素、读页面这些工具。

**用离线包的人更省事**：包里已经生成了一份填好你电脑真实路径的 `workbuddy-mcp.json`，直接把它里面的内容合进上面这个文件就行，不用自己手写路径。

## 第 2 步：点一次“信任”启用

改完配置不会自动生效——这是 WorkBuddy 的硬性规矩，必须在连接器管理界面手动“信任”这个新服务一次。漏了这步，就会表现为“配了但调不到”。

路径：WorkBuddy 侧边栏「插件」→ 右上角「MCP 服务器」→ 找到 agent-browser → 点「信任」。

## 第 3 步：验证——说一句话，看它自己干活

新开一个对话，直接下自然语言任务：

> 打开 github.com/trending，把前三个仓库名告诉我。

它能调用浏览器工具、自己打开页面并返回结果，就通了。通了之后可以接着试：

- “打开百度，搜一下 agent-browser，把第一页的结果标题列出来”
- “打开 example.com，截图发我看一眼”

用完记得关浏览器（它归终端会话管，关它靠命令、不靠点叉）：

```bash
agent-browser close
```

忘了关也没事：后台进程闲置 1 小时后自己退出。开了好几个想一次全关：`agent-browser close --all`。

## 最可能踩的四个坑（都是我们真遇上或官方文档点名的）

| 坑 | 现象 | 解法 |
|---|---|---|
| WorkBuddy 报找不到 `agent-browser` 命令 | 配了但调不到 | 它启动环境的 PATH 和你终端的 PATH 不一致。把配置里 `command` 的 `agent-browser` 换成完整路径：终端里跑 `where agent-browser`（Windows）或 `which agent-browser`（Mac）查出来填进去 |
| 没弹出浏览器窗口，以为没装上 | 命令跑通了但看不见窗口 | 不是故障，是无头模式。`snapshot -i` 看清单、`screenshot` 拍照；非要看真窗口：先 `close`，再 `agent-browser open baidu.com --headed` |
| 杀毒软件 / SmartScreen 拦截 | “Windows 已保护你的电脑”，或被杀软隔离 | exe 是 Vercel 官方原版、没数字签名的新程序，拦截是预期内的。SmartScreen 弹框点“更多信息 → 仍要运行”；被隔离就从隔离区恢复并把安装目录加白名单。公司管控电脑加不了白名单就真装不了 |
| 以为又装了一个 Chrome，怕搞混 | 担心卸载时分不清 | 包里那份 Chrome for Testing 只是解压在 `%USERPROFILE%\.agent-browser\` 里躺着的一堆文件：不进开始菜单、不进应用列表、不抢默认浏览器、不碰你日常 Chrome 的书签密码。卸载就是删这一个文件夹 |

还有一句定心丸：它驱动的是独立的 Chrome for Testing，不是你日常用的 Chrome——你的登录状态、书签、密码它都读不到，也不会写进去。

## 它到底能帮你干什么

接通之后，WorkBuddy 不再是只能聊天的脑子，它能：

- 查资料：打开指定网页，把内容读回来、做总结
- 盯榜单：每天去 GitHub Trending 抓前几名（第一篇教程里实测过这条路）
- 跑流程：填表单、点按钮、翻页——按你一句话拆成的步骤自己走
- 出证据：截图、存 PDF，干完的活有图有真相

成本还是 0：agent-browser 本体不调任何模型、不连云端，没有 key、没有额度。真要花钱的地方只在 WorkBuddy 背后的模型那头，跟这双手无关。

## 和第一篇的关系

- 第一篇（仓库 README）：两行命令装好手、学会 open / snapshot / click 这一套
- 本篇：把手交给 WorkBuddy，开始说人话指挥它干活
- 下一篇预告：agent-browser vs browser-use——一个要花钱调模型，一个一分钱不花，差在哪

---

*功能以 agent-browser v0.42.0 为准；WorkBuddy 的 MCP 界面以你用的版本为准，入口名字可能略有出入。*
