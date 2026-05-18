# MCP 安装指南（航司活动页面搭建）

本文档帮助你在 Claude Code 中安装以下三个 MCP 工具，安装完成后即可使用 AI 快速搭建航司活动页面。

---

## 安装前准备

**确认已安装 Claude Code**

打开 Terminal（Mac）或 PowerShell（Windows），输入以下命令并回车：

```
claude --version
```

如果显示版本号（如 `2.x.x`），说明已安装完成，可继续操作。
如果提示 `command not found`，请先安装 Claude Code，再回来执行本指南。

---

## 如何打开 Terminal / PowerShell

- **Mac：** 按 `Command + 空格`，输入 `Terminal`，回车打开。
- **Windows：** 按 `Win + R`，输入 `powershell`，回车打开。

---

## 安装步骤

将以下命令逐条复制到 Terminal / PowerShell 中执行。每条命令执行完后，等待出现新的输入提示符（`$` 或 `>`），再执行下一条。

### 第一步：安装 DSL 知识库（dsl-mcp）

```bash
claude mcp add --transport http \
--scope user dsl-mcp "http://xiaobang-mcp-function.faas.ctripcorp.com/mcp" \
--header "x-bbzai-mcp-token: YOUR token" \
--header "X-Knowledge-Id: kb-1774949209911-ns3zbv"
```

### 第二步：安装活动页面工具（campaign-page）

> ⚠️ **注意：** 以下命令中的 `YOUR_EMAIL@trip.com` 必须替换成你自己的公司邮箱（例如 `zhangsan@trip.com`），否则无法正常使用。

```bash
claude mcp add --transport http \
--scope user campaign-page "http://foxpage-page-mcp-hot-function.faas.ctripcorp.com/mcp" \
--header "user-email: YOUR_EMAIL@trip.com"
```

### 第三步：安装组件知识库（campaign-component-mcp）

```bash
claude mcp add --transport http \
--scope user campaign-component-mcp "http://xiaobang-mcp-function.faas.ctripcorp.com/mcp" \
--header "x-bbzai-mcp-token: YOUR token" \
--header "X-Knowledge-Id: kb-1778755975906-9iefs9"
```

---

## 验证安装是否成功

三条命令都执行完后，运行以下命令：

```
claude mcp list
```

如果看到以下三项，说明安装成功：

```
campaign-component-mcp
campaign-page
dsl-mcp
```

---

## 安装 Skill 文件

Skill 文件是 AI 的操作指南，告诉 AI 如何帮你搭建页面。

将 `build-airline-campaign-page.md` 文件下载后：

- **Mac：** 放入 `~/skills/` 文件夹（路径：`/Users/你的用户名/skills/`，如文件夹不存在请新建）
- **Windows：** 放入 `C:\Users\你的用户名\skills\` 文件夹（如不存在请新建）

---

## 开始使用

1. 在 Terminal / PowerShell 中输入 `claude` 并回车，打开 Claude Code
2. 告诉 AI：`我要搭建一个航司活动页面`
3. AI 会引导你逐步完成页面搭建

---

## 常见问题（Q&A）

**Q：我不知道自己的 token 是什么？**
A：进入 http://bbz-ai.ctripcorp.com/admin/personal/coding-plan 复制 token

---

**Q：执行命令时提示 `command not found: claude`？**
A：Claude Code 尚未安装，请先完成 Claude Code 的安装后，再回来执行本指南。

---

**Q：命令执行完没有任何输出，不知道有没有成功？**
A：执行 `claude mcp list`，如果能看到三个 MCP 名称，说明已安装成功。

---

**Q：`claude mcp list` 只看到部分 MCP，有几个缺失？**
A：缺少的几个未安装成功，重新执行对应步骤的命令，再次验证即可。

---

**Q：三个 MCP 都装好了，但 AI 说不认识这些工具？**
A：退出 Claude Code（输入 `/exit` 或关闭窗口），重新打开后再试。

---

**Q：第二步忘记替换邮箱，用的是 `YOUR_EMAIL@trip.com`？**
A：先删除错误配置，再重新安装：

```
claude mcp remove campaign-page
```

然后重新执行第二步，注意替换为自己的邮箱。

---

**Q：命令执行报错，不知道怎么处理？**
A：把报错信息截图，联系对应负责人获取支持。
