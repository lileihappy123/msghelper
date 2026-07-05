# OpenAPI 文档入口

本目录已按“精简可维护”原则整理，仅保留当前有价值的文档与 Postman 集合。

## 保留文件

- `QUICKSTART_群发5分钟.md`：**新手入口**，照着做 5 分钟跑通群发（推荐先看这个）
- `OPENAPI_CLIENT_CREDENTIALS_GUIDE.md`：认证原理与配额说明（进阶）
- `API_DETAILED_DOCUMENTATION.md`：所有接口字段详细说明（查字典用）
- `API_AUTH_TESTING_README.md`：认证与联调补充说明
- `MsgHelper_OpenAPI_场景化集合.postman_collection.json`：主 Postman 集合（配合快速上手指南使用）
- `MsgHelper_OpenAPI_Quota_Test.postman_collection.json`：配额专项联调集合

## 快速使用

1. 启动本地服务：`python app.py`，确认微信 PC 版已登录
2. 阅读 `QUICKSTART_群发5分钟.md`，照步骤操作
3. 需要了解字段细节时，查 `API_DETAILED_DOCUMENTATION.md`

## 当前实现要点（与代码一致）

- `/api/*` 请求默认会进入开放 API 配额拦截。
- 豁免路径（不扣开放 API 配额）：`/api/ping`、`/api/get_version`、`/api/auth/login`、`/api/ui/update`、`/api/trial-quota/policy`、`/api/trial-quota/check-and-consume`、`/api/trial-quota/balance`、`/api/openapi-quota/balance`、`/api/mcp/tools`
- `/local/*` 前缀豁免。
- 若请求未携带 `tenant-id` 或 `Authorization`，不在全局配额闸门内扣减，仍由各业务接口按原有逻辑处理。
