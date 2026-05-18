# Campaign2.0 选品 MCP 使用文档

## 一、安装

安装 Campaign Selection MCP：在支持 MCP 的客户端中新增 MCP Server，传输方式必须是 streamable-http。

- 服务地址：`http://mcp.gateway.ctripcorp.com/mcp/campaign/selection`
- 认证信息：请求头中添加 `Authorization: Bearer 60566e6f-f8dc-41e5-ac58-2c7860304bcc`

## 二、各产线选品池接口说明（提供给 Agent）

### 机票产线

接口名称：`upsertSelectionPoolForApi`

详细入参说明请参考：[flight-upsert-selection-pool-agent-guide.md](./flight-upsert-selection-pool-agent-guide.md)

### TNT 产线

接口名称：`upsertTntSelectPool`

接口传参说明：（参见飞书文档内嵌附件）

### 酒店产线

接口名称：`upsertHotelSelectionSchema`

### 酒店 & 机票选品池完整 Skill

详细使用说明请参考：[skill-hotel-flight-selection.md](./skill-hotel-flight-selection.md)
