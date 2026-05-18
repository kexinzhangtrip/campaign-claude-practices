# Foxpage 组件知识库 MCP

## Claude Code 中配置

Foxpage 相关知识库

### DSL 结构知识库（dsl-mcp）

```json
"dsl-mcp": {
  "type": "http",
  "description": "foxpage页面dsl结构知识库",
  "url": "http://xiaobang-mcp-function.faas.ctripcorp.com/mcp",
  "headers": {
    "x-bbzai-mcp-token": "",
    "X-Knowledge-Id": "kb-1774949209911-ns3zbv"
  }
}
```

### 活动组件知识库（campaign-component-mcp）

```json
"campaign-component-mcp": {
  "type": "http",
  "description": "foxpage活动组件知识库",
  "url": "http://xiaobang-mcp-function.faas.ctripcorp.com/mcp",
  "headers": {
    "x-bbzai-mcp-token": "",
    "X-Knowledge-Id": "kb-1778755975906-9iefs9"
  }
}
```

> 🔷 注意：上面的 `x-bbzai-mcp-token` 要复制自己 coding-plan 中的 token，填写进去，才可以正确访问。

**coding-plan 中的 token**

http://bbz-ai.ctripcorp.com/admin/personal/coding-plan
