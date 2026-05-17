# Foxpage Page MCP

提供通过 MCP 来创建页面、更新页面、以及发布页面等能力。

目前只允许在 Foxpage Campaign 应用下的指定项目下来操作页面，具体可以和开发沟通修改项目配置。

1. 配置了指定的项目
2. 配置了指定的模板

## 在 Cursor 中使用

- MCP 测试环境地址：`http://foxpage-page-mcp-function.fws.faas.qa.nt.ctripcorp.com`
- MCP 生产环境地址：`http://foxpage-page-mcp-function.faas.ctripcorp.com`

cursor 中 url，serverUrl 要设置为对应的环境地址：

```json
{
  "mcpServers": {
    "foxpage-page-mcp": {
      "type": "http",
      "description": "Foxpage page Model Context Protocol server",
      "url": "http://foxpage-page-mcp-function.fws.faas.qa.nt.ctripcorp.com/mcp",
      "serverUrl": "http://foxpage-page-mcp-function.fws.faas.qa.nt.ctripcorp.com/mcp",
      "headers": {
        "user-email": "xxx@trip.com"
      }
    }
  }
}
```

## Claude Code 中配置

### 方式一：命令行添加

添加后重启 Claude，使用 `claude mcp list` 查看是否添加成功，注意测试/生产的地址：

```bash
claude mcp add foxpage-page-mcp \
  --transport http \
  --header "user-email: xxx@trip.com" \
  http://foxpage-page-mcp-function.fws.faas.qa.nt.ctripcorp.com/mcp
```

### 方式二：配置文件

在以下任一目录添加配置：

- `~/.claude/settings.json`
- `~/.config/claude/settings.json`
- 项目目录下的 `.claude/settings.json`

```json
{
  "mcpServers": {
    "foxpage-page-mcp": {
      "type": "http",
      "url": "http://foxpage-page-mcp-function.fws.faas.qa.nt.ctripcorp.com/mcp",
      "headers": {
        "user-email": "xxx@trip.com"
      }
    }
  }
}
```

## MCP 提供的 Tools

| 序号 | 名称 | 功能 | 备注 |
|------|------|------|------|
| 1 | `create-page` | 创建页面 | 不创建页面路由信息；如果页面 file 不存在则创建；如果 locale 已存在则返回提示 |
| 2 | `update-page` | 修改页面 | 修改草稿版本；如果不存在草稿则创建一个 |
| 3 | `publish-page` | 发布页面 | 将草稿版本发布为线上版本；如果不存在草稿则返回提示 |

### create-page 入参

```json
{
  "promoId": 1234,           // 可选，如果不传，默认使用配置的第一个活动号
  "fileId": "file_xxx",      // 可选，如果不传，则创建 file 信息
  "name": "test-promo-page", // 必填，页面内容名称
  "locale": "en-US",         // 必填，页面内容 locale，en-US|zh-HK|...
  "schemas": [{}]            // 必填，页面内容结构
}
```

### create-page / update-page 返回

```json
{
  "code": 200,
  "msg": "",
  "data": {
    "project": {
      "id": "fold_xxx",
      "promoId": 1234
    },
    "file": {
      "id": "file_xxx",
      "name": ""
    },
    "content": {
      "id": "cont_xxx",
      "name": ""
    },
    "version": {
      "id": "cver_xxx",
      "version": "",
      "schemas": [{}]  // update-page 返回中包含
    }
  }
}
```

### update-page 入参

```json
{
  "contentId": "cont_xxx",  // 必填，页面内容 id
  "schemas": [{}]           // 必填，页面内容结构
}
```

### publish-page 入参

```json
{
  "contentId": "cont_xxx"  // 必填，页面内容 id
}
```

### publish-page 返回

```json
{
  "code": 200,
  "msg": ""
}
```

## 配置的项目和模板

### 测试环境

- **项目：17172**
  - https://admin.foxpage.fat1.qa.nt.ctripcorp.com/#/workspace/projects/personal/detail?applicationId=appl_lrtAwm4oU6etnty&folderId=fold_pXUnhqUATj7g28T
- **模板：`cont_3FW5FimASJ5QRlc`**
  - https://admin.foxpage.fat1.qa.nt.ctripcorp.com/#/projects/content?applicationId=appl_lrtAwm4oU6etnty&fileId=file_wknh2TvugPLPJy2

### 生产环境

- **项目：41015**
  - https://admin.foxpage.ibu.ctripcorp.com/#/applications/appl_zrvXd1AE3LIf8a9/projects/detail?folderId=fold_l24xT6N1HJG4FRL
- **模板：`cont_E2qP26CsShge21i`**
  - https://admin.foxpage.ibu.ctripcorp.com/#/applications/appl_zrvXd1AE3LIf8a9/projects/content?fileId=file_29BQVvlCKquncgF&folderPage=1
