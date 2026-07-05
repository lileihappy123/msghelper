# MsgHelper OpenAPI（Client Credentials）文档

## 1. 目标

面向应用系统（非人工账号），通过 `clientId/clientSecret` 调用本地 Python API，并验证调用量扣减是否正常。

- Python API 基地址：`http://127.0.0.1:6003`
- 后端 API 基地址：`http://127.0.0.1:48080/admin-api`

---

## 2. 认证方式

### 2.1 推荐：通过 Python 登录封装接口

`POST /api/auth/login`

请求体：

```json
{
  "tenantId": "154",
  "clientId": "msghelper-client-154",
  "clientSecret": "<YOUR_CLIENT_SECRET>",
  "loginMode": "client_credentials",
  "scope": "msghelper.api",
  "forceRelogin": false
}
```

成功响应（示例）：

```json
{
  "code": 200,
  "msg": "执行成功",
  "data": {
    "code": 0,
    "msg": "成功",
    "data": {
      "accessToken": "...",
      "refreshToken": "...",
      "tenantId": "154"
    }
  }
}
```

> 说明：该接口内部做了 token 缓存与刷新，适合本地客户端统一接入。

---

## 3. 业务接口示例

### 3.1 获取客户端版本（不扣开放 API 套餐额度）

`GET /api/get_version`

请求头：

- `Authorization: Bearer <ACCESS_TOKEN>`
- `tenant-id: 154`
- `user-id: 0`

### 3.2 获取微信版本（会进入开放 API 配额拦截）

`GET /api/get_wx_version`

请求头同上。

> 该接口依赖本机微信窗口可被 UI 自动化识别。接口报错不影响“是否扣减”验证，只要请求经过 `/api/*` 且不在豁免列表，就会触发配额逻辑。

---

## 4. 配额扣减规则（当前实现）

Python 端使用本地批量回传策略：

- 累积阈值：`BATCH_CONSUME_SIZE = 3`
- 最长回传间隔：`SYNC_INTERVAL_SECONDS = 8`

行为：

1. 每次请求先本地累计 `pendingConsume`
2. 满足“累计达到 3”或“距离上次同步超过 8 秒”任一条件后，一次性回传本地累计值
3. 所以后端明细 `consumed` 可能是 2/3/6/7 等批量值，不一定是 1

全局闸门范围：

- 默认拦截 `/api/*`。
- 以下路径豁免：
  - `/api/ping`
  - `/api/get_version`
  - `/api/auth/login`
  - `/api/ui/update`
  - `/api/trial-quota/policy`
  - `/api/trial-quota/check-and-consume`
  - `/api/trial-quota/balance`
  - `/api/openapi-quota/balance`
  - `/api/mcp/tools`
- `/local/*` 前缀豁免。
- 请求需同时携带 `tenant-id` 与 `Authorization` 才会进入全局扣减流程。

---

## 5. 扣减验证步骤（推荐）

1. 记录基线：
   - `msghelper_openapi_quota_daily.consumed_count`
   - `msghelper_openapi_quota_detail_log` 最新 `id`
2. 在 8 秒内连续调用 3 次 `GET /api/get_wx_version`
3. 再查询基线：
   - 正常应看到 `consumed_count` 增加 3（若 Python 重启且 pending=0）
   - 明细新增一条 `consumed=3`（operation_id 形如 `openapi-batch-...`）

> 若出现增加不是 3，通常是重启前已有 pending 或跨过 8 秒触发了“超时回传”。

---

## 6. 常见问题

### Q1：`/api/get_wx_version` 返回窗口超时

这是微信窗口识别问题，不是认证问题。只要请求通过拦截器，配额仍会按规则累计/回传。

### Q2：为什么一次扣了 6 或 7？

因为是“批量回传”而非“单次立即落库”。

### Q3：如何改成每次都扣 1？

将 Python 端批量策略调整为每次立即回传（例如阈值改 1）。

---

## 7. 相关文件

- 本文档：`docs/OpenAPI/OPENAPI_CLIENT_CREDENTIALS_GUIDE.md`
- Postman 示例：`docs/OpenAPI/MsgHelper_OpenAPI_Quota_Test.postman_collection.json`
- 旧版认证测试总览：`docs/OpenAPI/API_AUTH_TESTING_README.md`
