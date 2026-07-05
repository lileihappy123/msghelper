# API 认证测试资源说明

> 状态说明（2026-06）：当前推荐阅读 [OPENAPI_CLIENT_CREDENTIALS_GUIDE.md](OPENAPI_CLIENT_CREDENTIALS_GUIDE.md) 和 [MsgHelper_OpenAPI_Quota_Test.postman_collection.json](MsgHelper_OpenAPI_Quota_Test.postman_collection.json)。

## 概览

本目录保留认证与回归相关资源，用于验证用户名/密码登录、client credentials 登录、Token 缓存和 OpenAPI 配额行为。

## 保留文件

- [OPENAPI_CLIENT_CREDENTIALS_GUIDE.md](OPENAPI_CLIENT_CREDENTIALS_GUIDE.md)：当前主文档
- [MsgHelper_OpenAPI_Quota_Test.postman_collection.json](MsgHelper_OpenAPI_Quota_Test.postman_collection.json)：OpenAPI 配额联调集合
- [MsgHelper_API_Auth_Tests.postman_collection.json](MsgHelper_API_Auth_Tests.postman_collection.json)：认证回归集合
- [API_AUTH_TESTING_README.md](API_AUTH_TESTING_README.md)：本文档

## 适用场景

- 验证登录封装接口 `POST /api/auth/login`
- 验证 `/api/*` 配额拦截与豁免路径
- 验证 Token 缓存、刷新和 client credentials 模式

## 使用顺序

1. 先读 [OPENAPI_CLIENT_CREDENTIALS_GUIDE.md](OPENAPI_CLIENT_CREDENTIALS_GUIDE.md)
2. 导入 [MsgHelper_OpenAPI_Quota_Test.postman_collection.json](MsgHelper_OpenAPI_Quota_Test.postman_collection.json)
3. 若需要补充认证回归，再参考 [MsgHelper_API_Auth_Tests.postman_collection.json](MsgHelper_API_Auth_Tests.postman_collection.json)

## 备注

- 以代码行为为准。
- `/api/*` 的全局配额规则和豁免清单见 [OPENAPI_CLIENT_CREDENTIALS_GUIDE.md](OPENAPI_CLIENT_CREDENTIALS_GUIDE.md)。
