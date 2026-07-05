# MsgHelper OpenAPI 详细说明（按 Postman 场景组织）

本文档按 `MsgHelper_OpenAPI_场景化集合.postman_collection.json` 的分组顺序整理，方便直接和 Postman 集合同步使用。

---

## 一、通用约定

### 1. 基础地址

- 默认示例：`http://127.0.0.1:6003`

### 2. 通用返回结构

```json
{
  "code": 200,
  "msg": "success",
  "data": {}
}
```

> 说明：大多数业务接口采用上面的包装结构；少量查询接口会直接返回对象/数组（不带 `code/msg/data`）。
> 文档中若某接口“返回值”章节已单列，则以单列内容为准。

### 3. 通用请求头

除非接口另有说明，业务接口建议统一携带：

| 字段 | 含义 | 必填 | 默认值 |
| --- | --- | --- | --- |
| `Authorization` | 访问令牌，格式 `Bearer <accessToken>` | 是 | 无 |
| `tenant-id` | 业务租户 ID | 是 | 无 |
| `user-id` | 业务用户 ID；`client_credentials` 场景建议固定 `0` | 是 | `0` |
| `Content-Type` | JSON 请求体类型 | POST/PUT 时是 | `application/json` |
| `client-id` | 客户端业务隔离字段，仅绑定接口使用 | 仅绑定接口必填 | 无 |

### 4. 关键字段词典

| 字段 | 含义 | 备注 |
| --- | --- | --- |
| `tenantId` | 业务租户 ID | 与请求头 `tenant-id` 对应 |
| `clientId` | 客户端业务标识 | 仅本地绑定接口使用 |
| `userId` | 业务用户 ID | `client_credentials` 场景默认 `0` |
| `wxInstanceId` | 业务微信实例 ID | 绑定表自增主键 `id` |
| `wxId` | 定时任务包装层中的微信实例 ID | 通常与 `wxInstanceId` 同值 |
| `wxName` / `userWxName` | 当前操作的微信名称 | 多数任务入口使用 |
| `wxNo` | 微信号 | 绑定去重优先字段 |
| `hwnd` | 微信窗口句柄 | 仅运行态使用，不是业务 ID |

---

## 1. 调用管理

### 1.1 ClientId 登录（client_credentials）

- **方法**：`POST`
- **路径**：`/api/auth/login`
- **用途**：获取 `accessToken` / `refreshToken`

#### Body 字段

| 字段 | 含义 | 必填 | 默认值 | 备注 |
| --- | --- | --- | --- | --- |
| `tenantId` | 租户 ID | 是 | 无 | 字符串形式传递 |
| `loginMode` | 登录模式 | 否 | `password` | `client_credentials` 表示客户端模式 |
| `clientId` | 客户端 ID | `loginMode=client_credentials` 时必填 | 无 | 与本地绑定接口的 `clientId` 建议保持一致 |
| `clientSecret` | 客户端密钥 | `loginMode=client_credentials` 时必填 | 无 | 由服务端签发 |
| `scope` | 权限范围 | 否 | `msghelper.api` | 客户端模式推荐固定该值 |
| `forceRelogin` | 是否强制重新登录 | 否 | `false` | 布尔值 |

#### 1.1 返回值示例

```json
{
  "code": 200,
  "data": {
    "code": 0,
    "data": {
      "access_token": "e83515cae8f84b5884a130895fb8b1b6",
      "expires_in": 7198,
      "refresh_token": "540fd16b1ce64121ba8b706cdb6a55b6",
      "scope": "msghelper.api",
      "token_type": "bearer"
    },
    "msg": ""
  },
  "msg": "执行成功"
}
```

#### 使用建议

- `client_credentials` 模式下，不要依赖登录响应返回 `userId`

#### 1.1 返回字段说明

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `code` | `number` | 外层业务状态码，`200` 表示成功 |
| `msg` | `string` | 外层返回消息 |
| `data.code` | `number` | 登录子状态码，`0` 表示成功 |
| `data.msg` | `string` | 登录子消息 |
| `data.data.access_token` | `string` | 访问令牌，后续接口放入 `Authorization` |
| `data.data.expires_in` | `number` | 令牌有效秒数 |
| `data.data.refresh_token` | `string` | 刷新令牌 |
| `data.data.scope` | `string` | 作用域 |
| `data.data.token_type` | `string` | 令牌类型 |


---

### 1.2 API 配额查询

- **方法**：`GET`
- **路径**：`/api/openapi-quota/balance`
- **用途**：查询开放 API 套餐配额余额

#### API 配额查询 Query 字段

| 字段 | 含义 | 必填 | 默认值 | 备注 |
| --- | --- | --- | --- | --- |
| `tenantId` | 租户 ID | 是 | 无 | 优先级高于请求头 |
| `accessToken` | 显式传入的访问令牌 | 否 | 无 | 不传时回退 `Authorization` |
| `forceRefresh` | 是否强制刷新缓存 | 否 | `false` | 支持 `true/false/1/0` |

#### 1.2 返回值示例

```json
{
  "code": 200,
  "data": {
    "dailyQuota": 10000,
    "dailyRemaining": 9996,
    "dailyUsed": 4,
    "expireTime": null,
    "resetTime": "2026-07-04T00:00",
    "unlimited": false
  },
  "msg": "成功"
}
```

#### 1.2 返回字段说明

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `code` | `number` | 外层业务状态码，`200` 表示成功 |
| `msg` | `string` | 外层返回消息 |
| `data.dailyQuota` | `number` | 当日总额度 |
| `data.dailyRemaining` | `number` | 当日剩余额度 |
| `data.dailyUsed` | `number` | 当日已用额度 |
| `data.expireTime` | `string\|null` | 套餐到期时间 |
| `data.resetTime` | `string` | 次日重置时间 |
| `data.unlimited` | `boolean` | 是否无限额度 |

---

## 2. 微信管理

### 2.1 获取登录的微信版本

- **方法**：`GET`
- **路径**：`/api/get_opened_wx_version`
- **用途**：返回当前系统已打开微信进程的版本列表

#### 微信版本接口字段说明

无额外 Query / Body 字段。

#### 微信版本接口响应字段

| 字段 | 含义 |
| --- | --- |
| `data[]` | 去重后的微信版本字符串列表 |

#### 2.1 返回值示例

```json
{
  "code": 200,
  "data": [
    "4.1.7"
  ],
  "msg": "success"
}
```

#### 2.1 返回字段说明

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `code` | `number` | 外层业务状态码，`200` 表示成功 |
| `msg` | `string` | 外层返回消息 |
| `data` | `string[]` | 已登录微信版本列表 |

---

### 2.2 获取登录的微信

- **方法**：`GET`
- **路径**：`/api/get_opened_wxs`
- **用途**：返回当前在线微信窗口运行态信息

#### 获取登录微信响应字段

| 字段 | 含义 | 备注 |
| --- | --- | --- |
| `index` | 当前返回项序号 | 运行态字段 |
| `nickName` | 微信昵称 | 运行态字段 |
| `wxNo` | 微信号 | 运行态字段 |
| `lang` | 语言 | 运行态字段 |
| `version` | 微信主版本 | 运行态字段 |
| `hwnd` | 窗口句柄 | 仅运行态使用 |
| `pid` | 进程 ID | 仅运行态使用 |
| `fullVersion` | 完整版本号 | 仅运行态使用 |

> 注意：此接口**不返回**业务 `wxInstanceId`。

> ⚠️ **`nickName` 返回"未知昵称"**：这是 UI 自动化无法读取微信界面文字导致的，通常在 Windows 无障碍服务未启用时出现。修复方法：
> 1. 退出微信（右键托盘图标 → 退出）
> 2. 打开 **Windows 讲述人**（快捷键 `Win + Ctrl + Enter`，或搜索"讲述人"）
> 3. 重新登录微信，等待主界面完全加载
> 4. 重新调用本接口，`nickName` 应能正常返回
>
> 讲述人开启后可保持运行，不影响后续所有接口的正常使用。

#### 2.2 返回值示例

```json
{
  "code": 200,
  "data": [
    {
      "fullVersion": "4.1.7",
      "hwnd": 196834,
      "index": 0,
      "lang": "cn",
      "nickName": "码农(如不回复,官网找联系方式)",
      "pid": 5916,
      "version": "4",
      "window": "",
      "wxNo": "ejiecrm"
    }
  ],
  "msg": "success"
}
```

#### 2.2 返回字段说明

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `code` | `number` | 外层业务状态码，`200` 表示成功 |
| `msg` | `string` | 外层返回消息 |
| `data` | `array` | 在线微信运行态列表 |
| `data[].fullVersion` | `string` | 完整版本号 |
| `data[].hwnd` | `number` | 窗口句柄 |
| `data[].index` | `number` | 列表序号 |
| `data[].lang` | `string` | 语言 |
| `data[].nickName` | `string` | 微信昵称 |
| `data[].pid` | `number` | 进程 ID |
| `data[].version` | `string` | 微信主版本 |
| `data[].window` | `string` | 窗口标题 |
| `data[].wxNo` | `string` | 微信号 |

---

### 2.3 绑定微信（生成 `WX_INSTANCE_ID`）

- **方法**：`POST`
- **路径**：`/api/wx-bindings/bind`
- **用途**：在客户端本地生成业务微信绑定记录

#### 绑定微信 Body 字段

| 字段 | 含义 | 必填 | 默认值 | 备注 |
| --- | --- | --- | --- | --- |
| `tenantId` | 租户 ID | 是 | 无 | 可由请求头透传，但建议显式传 |
| `clientId` | 客户端业务标识 | 是 | 无 | 本地绑定隔离维度 |
| `userId` | 业务用户 ID | 否 | `0` | `client_credentials` 场景建议固定 `0` |
| `wxName` | 微信名称 | `wxName` / `wxNo` 二选一至少传一个 | 无 | 常用主字段 |
| `wxNo` | 微信号 | `wxName` / `wxNo` 二选一至少传一个 | 无 | 去重优先字段 |

#### 重复绑定规则

- 优先按 `tenantId + clientId + userId + wxNo` 查重
- 若 `wxNo` 为空，则按 `tenantId + clientId + userId + wxName` 查重
- 命中重复时返回 `400`

#### 绑定微信响应字段

| 字段 | 含义 |
| --- | --- |
| `data.wxInstanceId` | 新生成的业务微信实例 ID |
| `data.tenantId` | 租户 ID |
| `data.clientId` | 客户端 ID |
| `data.userId` | 用户 ID |
| `data.wxName` | 微信名称 |
| `data.wxNo` | 微信号 |

#### 2.3 返回值示例

```json
{
  "code": 400,
  "data": {
    "clientId": "msghelper-client-154",
    "createdTime": "2026-06-27 03:51:00",
    "tenantId": "154",
    "updatedTime": "2026-06-27 03:51:00",
    "userId": "0",
    "wxInstanceId": 2,
    "wxName": "码农(如不回复,官网找联系方式)",
    "wxNo": "ejiecrm"
  },
  "msg": "该微信已存在绑定，请先解绑后再操作"
}
```

#### 2.3 返回字段说明

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `code` | `number` | 外层业务状态码，`200` 或 `400` |
| `msg` | `string` | 外层返回消息 |
| `data.clientId` | `string` | 客户端 ID |
| `data.createdTime` | `string` | 创建时间 |
| `data.tenantId` | `string` | 租户 ID |
| `data.updatedTime` | `string` | 更新时间 |
| `data.userId` | `string` | 用户 ID |
| `data.wxInstanceId` | `number` | 业务微信实例 ID |
| `data.wxName` | `string` | 微信名称 |
| `data.wxNo` | `string` | 微信号 |

---

### 2.4 查询绑定微信列表

- **方法**：`GET`
- **路径**：`/api/wx-bindings/list`
- **用途**：查询某一业务上下文下的微信绑定列表

#### 查询绑定微信列表 Query 字段

| 字段 | 含义 | 必填 | 默认值 |
| --- | --- | --- | --- |
| `tenantId` | 租户 ID | 是 | 无 |
| `clientId` | 客户端业务标识 | 是 | 无 |
| `userId` | 业务用户 ID | 否 | `0` |

#### 查询绑定微信列表响应字段

| 字段 | 含义 |
| --- | --- |
| `data.list[]` | 绑定记录列表 |
| `data.list[].wxInstanceId` | 业务微信实例 ID |
| `data.list[].tenantId` | 租户 ID |
| `data.list[].clientId` | 客户端 ID |
| `data.list[].userId` | 用户 ID |
| `data.list[].wxName` | 微信名称 |
| `data.list[].wxNo` | 微信号 |
| `data.total` | 列表总数 |

#### 2.4 返回值示例

```json
{
  "code": 200,
  "data": {
    "list": [
      {
        "clientId": "msghelper-client-154",
        "createdTime": "2026-06-27 03:51:00",
        "tenantId": "154",
        "updatedTime": "2026-06-27 03:51:00",
        "userId": "0",
        "wxInstanceId": 2,
        "wxName": "码农(如不回复,官网找联系方式)",
        "wxNo": "ejiecrm"
      }
    ],
    "total": 1
  },
  "msg": "success"
}
```

#### 2.4 返回字段说明

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `code` | `number` | 外层业务状态码 |
| `msg` | `string` | 外层返回消息 |
| `data.list` | `array` | 绑定列表 |
| `data.total` | `number` | 总数 |
| `data.list[].clientId` | `string` | 客户端 ID |
| `data.list[].createdTime` | `string` | 创建时间 |
| `data.list[].tenantId` | `string` | 租户 ID |
| `data.list[].updatedTime` | `string` | 更新时间 |
| `data.list[].userId` | `string` | 用户 ID |
| `data.list[].wxInstanceId` | `number` | 业务微信实例 ID |
| `data.list[].wxName` | `string` | 微信名称 |
| `data.list[].wxNo` | `string` | 微信号 |

---

### 2.5 解绑微信

- **方法**：`DELETE`
- **路径**：`/api/wx-bindings/unbind`
- **用途**：物理删除本地绑定记录

#### 解绑微信 Query / Body 字段

| 字段 | 含义 | 必填 | 默认值 | 备注 |
| --- | --- | --- | --- | --- |
| `tenantId` | 租户 ID | 是 | 无 | Query / Body 均可 |
| `clientId` | 客户端业务标识 | 是 | 无 | Query / Body 均可 |
| `userId` | 业务用户 ID | 否 | `0` | Query / Body 均可 |
| `wxInstanceId` | 业务微信实例 ID | 是 | 无 | 必须为整数 |

#### 2.5 返回值说明

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `code` | `number` | 业务状态码，成功常见 `200` |
| `msg` | `string` | 返回消息 |
| `data` | `object\|null` | 删除结果（不同实现可能为空） |

---

## 3. 标签管理

> 标签是系统级资源，用于对联系人、群聊进行分类。可独立创建、编辑、删除标签，也可在批量修改联系人/群聊时自动创建。

### 3.1 创建标签

- **方法**：`POST`
- **路径**：`/api/tags`
- **用途**：创建新标签

#### 创建标签 Body 字段

| 字段 | 含义 | 必填 | 默认值 | 备注 |
| --- | --- | --- | --- | --- |
| `name` | 标签名称 | 是 | 空 | |
| `wxInstanceId` | 业务微信实例 ID | 是 | 无 | |
| `userId` | 业务用户 ID | 是 | 无 | 当前实现会直接入库 |
| `tenantId` | 业务租户 ID | 是 | 无 | 当前实现会直接入库 |
| `description` | 标签描述 | 否 | 空 | |
| `sort` | 排序值 | 否 | `0` | 数字越小越靠前 |

#### 创建标签响应

```json
{
  "code": 200,
  "msg": "success",
  "data": {
    "id": "tag_123abc",
    "name": "VIP客户2"
  }
}
```

#### 3.1 返回字段说明

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `code` | `number` | 外层业务状态码 |
| `msg` | `string` | 外层返回消息 |
| `data.id` | `string` | 标签 ID |
| `data.name` | `string` | 标签名称 |

---

### 3.2 更新标签

- **方法**：`PUT`
- **路径**：`/api/tags`
- **用途**：编辑已有标签

#### 更新标签 Body 字段

| 字段 | 含义 | 必填 | 默认值 | 备注 |
| --- | --- | --- | --- | --- |
| `id` | 标签 ID | 是 | 空 | |
| `name` | 标签名称 | 是 | 空 | |
| `description` | 标签描述 | 否 | 空 | |
| `color` | 标签颜色 | 否 | 空 | 十六进制格式 |

#### 3.2 返回值示例

```json
{
  "code": 200,
  "data": true,
  "msg": "success"
}
```

#### 3.2 返回字段说明

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `code` | `number` | 外层业务状态码 |
| `msg` | `string` | 外层返回消息 |
| `data` | `boolean` | 是否更新成功 |

---

### 3.3 查询标签列表

- **方法**：`GET`
- **路径**：`/api/tags`
- **用途**：按微信实例查询所有标签

#### 查询标签 Query 字段

| 字段 | 含义 | 必填 | 默认值 | 备注 |
| --- | --- | --- | --- | --- |
| `wxInstanceId` | 业务微信实例 ID | 是 | 无 | |

#### 查询标签列表响应

```json
{
  "code": 200,
  "msg": "success",
  "data": [
    {
      "createdTime": "2026-06-27 07:33:54",
      "friendCount": 1,
      "id": "709b20b81d40489cad832fca192c995b",
      "name": "Excel Helper用户",
      "sort": 0,
      "tenantId": 154,
      "updateTime": "2026-06-27 07:33:54",
      "userId": "0",
      "wxInstanceId": "2"
    },
    {
      "createdTime": "2026-06-27 07:33:54",
      "friendCount": 0,
      "id": "tag_456def",
      "name": "待跟进",
      "sort": 1,
      "tenantId": 154,
      "updateTime": "2026-06-27 07:33:54",
      "userId": "0",
      "wxInstanceId": "2"
    }
  ]
}
```

#### 3.3 返回字段说明

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `code` | `number` | 外层业务状态码 |
| `msg` | `string` | 外层返回消息 |
| `data` | `array` | 标签列表 |
| `data[].createdTime` | `string` | 创建时间 |
| `data[].friendCount` | `number` | 关联好友数量 |
| `data[].id` | `string` | 标签 ID |
| `data[].name` | `string` | 标签名称 |
| `data[].sort` | `number` | 排序值 |
| `data[].tenantId` | `number` | 租户 ID |
| `data[].updateTime` | `string` | 更新时间 |
| `data[].userId` | `string` | 用户 ID |
| `data[].wxInstanceId` | `string` | 微信实例 ID |

---

### 3.4 删除标签

- **方法**：`DELETE`
- **路径**：`/api/tags/{tagId}`
- **用途**：删除标签（同时清除所有关联）

#### 删除标签 Path 字段

| 字段 | 含义 | 必填 |
| --- | --- | --- |
| `tagId` | 标签 ID | 是 |

> **注意**：删除标签后，所有绑定该标签的联系人/群聊关联将被移除。标签名称字符串（如 `wx_contact.tags`）会同步清除相应标签名。
>
> 当前实现中，`userId` 仅在“创建标签”时需要传入；更新 / 查询 / 删除不强制要求 `userId`。

#### 3.4 返回值说明

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `code` | `number` | 业务状态码 |
| `msg` | `string` | 返回消息 |
| `data` | `object\|null` | 删除结果（实现可能为空） |

---

## 4. 好友管理

### 4.1 从微信导入好友

- **方法**：`POST`
- **路径**：`/api/get_contacts_v2`
- **用途**：从当前微信导入好友到本地联系人库

#### 导入好友 Body 字段

| 字段 | 含义 | 必填 | 默认值 | 备注 |
| --- | --- | --- | --- | --- |
| `userWxName` | 要导入的微信名称 | 是 | 无 | 用于定位微信实例；通常与当前选中微信昵称一致 |
| `userWxId` | 当前选中的微信绑定 ID | 是 | 无 | 来自 UI 的 `wx.id`，通常与 `wxInstanceId` 同值 |
| `wxVersion` | 微信大版本 | 是 | 无 | UI 中传入 `wx.version`，示例值 `4` |
| `hwnd` | 微信窗口句柄 | 是 | 无 | 运行态识别与启动导入必需 |
| `sleepStart` | 两次好友间隔开始秒数 | 否 | `0` | UI 默认值 |
| `sleepEnd` | 两次好友间隔结束秒数 | 否 | `0.5` | UI 默认值 |
| `specificName` | 指定好友名称 | 否 | 空 | 仅导入指定名称时使用 |
| `maxScrollTimes` | 最大同名滚动次数 | 否 | `3` | UI 字段名 `maxSameNameScroll` |
| `syncType` | 导入模式 | 否 | `full` | 常用值：`full` / `incremental` |
| `recentNum` | 增量模式最近聊天数量 | 否 | `10` | 仅 `incremental` 时有效 |
| `strategy` | 导入策略 | 否 | `merge` | 常用值：`merge` / `skip` / `overwrite` |
| `updateOptions.remark` | 是否更新备注 | 否 | `true` | 布尔值 |
| `updateOptions.tags` | 是否更新标签 | 否 | `true` | 布尔值 |
| `userId` | 业务用户 ID | 是 | `0` | |
| `tenantId` | 业务租户 ID | 是 | 无 | |
| `wxInstanceId` | 业务微信实例 ID | 否 | 无 | 由请求拦截器自动补充 |

#### 导入好友响应字段

| 字段 | 含义 |
| --- | --- |
| `data.taskId` | 任务 ID |
| `data.monitorEndpoint` | 任务监控地址 |

#### 说明

- 若当前微信未在线或 `hwnd` 无效，接口会直接返回错误
- Postman 调试时请手动补全 `hwnd`，并显式传 `userId`、`tenantId`（仅传请求头不足以覆盖当前入库路径）
- 建议同时带上 `userWxName`、`userWxId`、`wxInstanceId` 以便与 UI 行为保持一致

#### 3.1 返回值说明

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `code` | `number` | 业务状态码 |
| `msg` | `string` | 返回消息 |
| `data.taskId` | `string` | 导入任务 ID |
| `data.monitorEndpoint` | `string` | 导入进度查询地址 |

---

### 4.2 好友查询

- **方法**：`GET`
- **路径**：`/api/contacts`
- **用途**：分页查询本地好友数据

#### 好友查询 Query 字段

| 字段 | 含义 | 必填 | 默认值 |
| --- | --- | --- | --- |
| `wxInstanceId` | 业务微信实例 ID | 是 | 无 |
| `keyword` | 搜索关键词 | 否 | 空 |
| `limit` | 分页条数 | 否 | `50` |
| `offset` | 分页偏移量 | 否 | `0` |
| `tagIds` | 标签 ID 列表 | 否 | 空 |
| `tagMatchAll` | 是否必须命中全部标签 | 否 | `false` |
| `sortBy` | 排序字段 | 否 | 空 |
| `sortOrder` | 排序方向 | 否 | 空 |
| `duplicateName` | 是否筛选重名 | 否 | `false` |
| `unsynced` | 是否筛选未同步 | 否 | `false` |
| `status` | 状态筛选 | 否 | 空 |

#### 好友查询响应字段

| 字段 | 含义 |
| --- | --- |
| `data.list[]` | 联系人列表 |
| `data.total` | 总条数 |

#### 3.2 返回值说明

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `code` | `number` | 业务状态码 |
| `msg` | `string` | 返回消息 |
| `data.list` | `array` | 联系人列表 |
| `data.total` | `number` | 总条数 |

#### data.list[] 数组元素字段

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `id` | `string` | 联系人主键 ID |
| `wxNo` | `string` | 微信号 |
| `wxInstanceId` | `string` | 微信实例 ID |
| `nickName` | `string` | 好友昵称 |
| `name` | `string` | 好友显示名称（优先级：备注 > 昵称 > 微信号） |
| `remark` | `string` | 好友备注 |
| `callName` | `string` | 称呼名 |
| `tags` | `string` | 标签（以逗号分隔） |
| `status` | `number` | 联系人状态（`1`=正常，`0`=删除我，`-1`=拉黑我，`-2`=未知） |
| `tenantId` | `number` | 所属租户 ID |
| `userId` | `string` | 所属用户 ID |
| `createdTime` | `string` | 创建时间（ISO 8601 格式） |
| `updateTime` | `string` | 更新时间（ISO 8601 格式） |
| `source` | `string` | 数据来源（`wx`=微信导入，`manual`=手动添加） |
| `version` | `number` | 版本号（用于并发控制） |
| `needSyncRemark` | `number` | 是否需要同步备注（`0`=不需要，`1`=需要） |
| `needSyncTags` | `number` | 是否需要同步标签（`0`=不需要，`1`=需要） |
| `remarkSynced` | `number` | 备注是否已同步（`0`=未同步，`1`=已同步） |
| `tagsSynced` | `number` | 标签是否已同步（`0`=未同步，`1`=已同步） |

---

### 4.3 好友删除

#### 2.3.1 仅删除系统

- **方法**：`DELETE`
- **路径**：`/api/contacts/{contact_id}`
- **用途**：仅删除系统本地联系人数据，不操作微信客户端

#### 仅删除系统 Path 字段

| 字段 | 含义 | 必填 | 默认值 |
| --- | --- | --- | --- |
| `contact_id` | 本地联系人 ID | 是 | 无 |

#### 3.3.1 返回值说明

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `code` | `number` | 业务状态码 |
| `msg` | `string` | 删除结果说明 |
| `data` | `object\|null` | 删除结果 |

#### 2.3.2 删除系统 + 微信

- **方法**：`POST`
- **路径**：`/api/delete_contacts_v2`
- **用途**：在微信客户端删除联系人，并同步删除系统数据

#### 删除系统 + 微信 Body 字段

| 字段 | 含义 | 必填 | 默认值 | 备注 |
| --- | --- | --- | --- | --- |
| `wxName` | 当前操作微信名称 | 是 | 无 | 与当前登录微信一致 |
| `wxInstanceId` | 业务微信实例 ID | 是 | 无 | 任务关联 ID，不是 `hwnd` |
| `wxVersion` | 微信版本 | 是 | 无 | `3` / `4`；`wx4` 场景会调用窗口句柄 |
| `hwnd` | 微信窗口句柄 | 是 | 无 | 必须传，`wx4` 删除时必需 |
| `syncType` | 搜索方式 | 否 | `0` | `0=微信号`，`1=昵称`，`2=备注名`，`3=智能匹配（微信号+昵称+备注名）` |
| `contacts` | 待删除联系人列表 | 是 | 空 | 只删除匹配到的第一个联系人 |
| `contacts[].id` | 本地联系人 ID | 建议必填 | 空 | 用于系统删除成功后同步删除本地记录 |
| `contacts[].wxNo` | 联系人微信号 | 建议必填 | 空 | `syncType=0` 时最推荐 |
| `contacts[].nickName` | 联系人昵称 | 否 | 空 | `syncType=1/3` 时可用 |
| `contacts[].remark` | 联系人备注 | 否 | 空 | `syncType=2/3` 时可用 |
| `optSleepTimeStart` | 操作间隔起始秒数 | 否 | `0.5` | |
| `optSleepTimeEnd` | 操作间隔结束秒数 | 否 | `1` | |
| `delSleepTimeStart` | 删除后间隔起始秒数 | 否 | `1` | |
| `delSleepTimeEnd` | 删除后间隔结束秒数 | 否 | `1.5` | |

#### 删除系统 + 微信 使用说明

- `wx4` 场景下如果不传 `hwnd`，会在初始化微信窗口时失败
- 该接口会先在微信客户端执行删除，再同步删除系统本地数据

#### 3.3.2 返回值说明

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `code` | `number` | 业务状态码 |
| `msg` | `string` | 执行结果说明 |
| `data.taskId` | `string\|null` | 异步任务 ID（若异步执行） |
| `data.successCount` | `number\|null` | 成功数量 |
| `data.failedCount` | `number\|null` | 失败数量 |

#### 2.3.3 一键清空系统

- **方法**：`POST`
- **路径**：`/api/contacts/clear_all`
- **用途**：按 `wxInstanceId` 一次性清空该微信实例下的系统联系人数据，不操作微信客户端

#### 一键清空系统 Body 字段

| 字段 | 含义 | 必填 | 默认值 |
| --- | --- | --- | --- |
| `wxInstanceId` | 业务微信实例 ID | 是 | 无 |

#### 一键清空系统 使用说明

- 该接口只清理系统侧联系人记录，不会在微信客户端执行删除
- 一键清空通常用于重置本地联系人数据，执行前请确认微信实例选择正确

#### 3.3.3 返回值说明

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `code` | `number` | 业务状态码 |
| `msg` | `string` | 清空结果说明 |
| `data.cleared` | `number\|null` | 清空条数（若实现返回） |

---

### 4.4 修改备注 / 标签 / 同步好友（仅微信）

- **方法**：`POST`
- **路径**：`/api/update_contacts_v2`
- **用途**：更新联系人备注、标签，或同时同步两者

> **标签传值说明**：顶层 `tags` 是所有联系人共用的全局标签（`isSyncUp=false` 时生效）。若每个联系人标签不同，设 `isSyncUp: true`，标签改从各 `contacts[].tags` 读取。

#### update_contacts_v2 Body 字段

| 字段 | 含义 | 必填 | 默认值 | 备注 |
| --- | --- | --- | --- | --- |
| `wxName` | 当前操作微信名称 | 是 | 无 | |
| `wxVersion` | 微信版本 | 否 | `"3"` | 微信 4.x 传 `"4"` |
| `hwnd` | 微信窗口句柄 | `wxVersion="4"` 时必填 | `0` | 从"获取登录的微信"接口取 |
| `updateRemark` | 是否更新备注 | 否 | `false` | |
| `updateTags` | 是否更新标签 | 否 | `false` | |
| `tags` | 全局标签字符串 | `updateTags=true` 且 `isSyncUp=false` 时必填 | `""` | 英文逗号分隔；所有联系人共用 |
| `isSyncUp` | 使用每联系人独立标签 | 否 | `false` | `true` 时标签取各 `contacts[].tags`，忽略顶层 `tags` |
| `syncType` | 搜索方式 | 否 | `0` | `0`=按微信号；`1`=按昵称；`2`=按备注名；`3`=智能（依次尝试微信号→昵称→备注名） |
| `contacts` | 待更新联系人列表 | 是 | 空 | |
| `contacts[].id` | 本地联系人 ID | 建议必填 | 空 | 用于同步状态回写 |
| `contacts[].wxNo` | 联系人微信号 | `syncType=0` 时必填 | 空 | |
| `contacts[].nickName` | 联系人昵称 | `syncType=1/3` 时用于搜索 | 空 | |
| `contacts[].remark` | 新备注 | `updateRemark=true` 时必填 | 空 | |
| `contacts[].tags` | 每联系人标签 | `isSyncUp=true` 且 `updateTags=true` 时必填 | 空 | 英文逗号分隔；`isSyncUp=false` 时忽略 |
| `contacts[].updateRemark` | 行级备注开关 | 否 | 继承顶层 `updateRemark` | 可覆盖全局开关 |
| `contacts[].updateTags` | 行级标签开关 | 否 | 继承顶层 `updateTags` | 可覆盖全局开关 |
| `optSleepTimeStart` | 单步操作间隔起始秒数 | 否 | `0.5` | |
| `optSleepTimeEnd` | 单步操作间隔结束秒数 | 否 | `1` | |

#### 4.4 返回值说明

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `code` | `number` | 业务状态码 |
| `msg` | `string` | 执行结果说明 |
| `data.taskId` | `string\|null` | 异步任务 ID（如走任务执行） |
| `data.successCount` | `number\|null` | 更新成功数量 |
| `data.failedCount` | `number\|null` | 更新失败数量 |

---

### 4.5 修改单个联系人（仅系统）

- **方法**：`POST`
- **路径**：`/api/contacts`
- **用途**：新增或更新单个联系人的系统数据，**不操作微信客户端**

#### 修改单个联系人 Body 字段

| 字段 | 含义 | 必填 | 默认值 | 备注 |
| --- | --- | --- | --- | --- |
| `contact.id` | 本地联系人 ID | 条件必填 | 空 | `contact.source="manual"` 时可不传（后端自动生成 UUID）；其他场景缺失会报 400 |
| `contact.wxNo` | 微信号 | 否 | 保持原值 | |
| `contact.wxInstanceId` | 业务微信实例 ID | 是 | 无 | |
| `contact.userId` | 业务用户 ID | 是 | 无 | |
| `contact.tenantId` | 业务租户 ID | 是 | 无 | |
| `contact.remark` | 系统备注 | 否 | 保持原值 | |
| `contact.callName` | 称呼 | 否 | 保持原值 | |
| `contact.source` | 数据来源 | 否 | 空 | 仅手动创建场景使用；更新已有联系人通常不传 |
| `tagNames` | 标签名称列表 | 否 | 保持原值 | 与 `tagIds` 配合使用 |
| `tagIds` | 标签 ID 列表 | 否 | 保持原值 | 与 `tagNames` 配合使用 |

> 说明：`POST /api/contacts` 既用于手动创建，也用于系统内编辑更新（如改备注/称呼/标签）。

#### 3.5.1 手动添加好友（仅系统）

同样使用 `POST /api/contacts`。手动创建时建议传 `contact.source="manual"`；若未传且满足手动创建最小字段（`wxNo`、`nickName`、`userId`、`tenantId`、`wxInstanceId`）并且未传 `contact.id`，后端会按手动创建处理并自动补 `source=manual`。

##### 手动添加好友最小字段

| 字段 | 含义 | 必填 | 备注 |
| --- | --- | --- | --- |
| `contact.id` | 联系人主键 ID | 否 | 不传时后端自动生成 UUID；传入则按传入值落库 |
| `contact.source` | 数据来源 | 否 | 默认按 `manual` 处理（建议显式传 `manual` 便于可读性） |
| `contact.wxNo` | 微信号 | 是 | 同实例下唯一，重复会返回 409 |
| `contact.nickName` | 昵称 | 是 | |
| `contact.userId` | 业务用户 ID | 是 | |
| `contact.tenantId` | 业务租户 ID | 是 | |
| `contact.wxInstanceId` | 业务微信实例 ID | 是 | |

> 可选字段：`contact.remark`、`contact.callName`、`tagNames`、`tagIds`。

#### 3.5 返回值说明

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `code` | `number` | 业务状态码 |
| `msg` | `string` | 返回消息 |
| `data.contactId` | `string` | 联系人 ID |
| `data.wxNo` | `string\|null` | 微信号 |
| `data.wxInstanceId` | `string\|number` | 微信实例 ID |

---

### 4.6 批量改标签（仅系统）

- **方法**：`POST`
- **路径**：`/api/contacts/tags/batch`
- **用途**：批量修改系统中多个联系人的标签，**不操作微信客户端**

#### 批量改标签 Body 字段

| 字段 | 含义 | 必填 | 默认值 | 备注 |
| --- | --- | --- | --- | --- |
| `contactIds` | 联系人 ID 列表 | 是 | 空 | |
| `tagIds` | 标签 ID 列表 | 否 | 空 | **仅该字段会写入/删除 `wx_contact_tag` 关联表** |
| `tagNames` | 标签名称列表 | 否 | 空 | 主要用于更新 `wx_contact.tags` 展示字符串；不会按名称自动补 `tagId` |
| `mode` | 操作模式 | 否 | `"replace"` | `replace`=替换；`add`=追加；`remove`=移除（不传 `tagIds` 时为移除全部） |

> 实现说明：
>
> - 只传 `tagNames` 不传 `tagIds` 时，不会新增关联表记录。
> - `replace` 模式下若 `tagIds` 为空，会先清空关联，再按 `tagNames` 回写展示字符串。
> - 若希望“查询/筛选/群发按标签”正确生效，请始终传递有效 `tagIds`（可同时传对应 `tagNames` 保持展示一致）。

#### 3.6 返回值说明

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `code` | `number` | 业务状态码 |
| `msg` | `string` | 操作结果说明 |
| `data.updated` | `number\|null` | 受影响联系人数量（若实现返回） |

---

### 4.7 批量改称呼（仅系统）

- **方法**：`POST`
- **路径**：`/api/contacts/call_name/batch`
- **用途**：根据昵称/备注自动批量生成联系人称呼，**不操作微信客户端**

#### 批量改称呼 Body 字段

| 字段 | 含义 | 必填 | 默认值 | 备注 |
| --- | --- | --- | --- | --- |
| `contactIds` | 联系人 ID 列表 | 是 | 空 | |
| `source` | 称呼来源字段 | 否 | `"nickName"` | 可选 `"remark"` 等 |
| `index` | 取值位置（1-based） | 否 | `1` | 例：`1` 取名字第一个字 |
| `suffix` | 称呼后缀 | 否 | `""` | 例：`"老师"` |
| `onlyEmpty` | 仅填充空称呼 | 否 | `true` | `false` 时强制覆盖已有称呼 |

#### 3.7 返回值说明

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `code` | `number` | 业务状态码 |
| `msg` | `string` | 操作结果说明 |
| `data.updated` | `number\|null` | 受影响联系人数量（若实现返回） |

---

### 4.8 群管理

本组接口与好友管理类似，也分为 **仅微信** 与 **仅系统** 两类：

- `POST /api/get_groups`：从微信客户端采集群聊（仅微信）
- `POST /api/groups`：手动新增/更新本地群聊（仅系统）
- `GET /api/groups`：查询本地群聊（仅系统）
- `POST /api/groups/batch_tags`：批量改群标签（仅系统）
- `DELETE /api/groups/{groupId}`：删除本地群聊（仅系统）

#### 2.8.5 手动添加群聊（仅系统）

- **方法**：`POST`
- **路径**：`/api/groups`
- **用途**：手动新增或更新本地群聊数据，**不操作微信客户端**

##### 手动添加群聊 Body 字段

| 字段 | 含义 | 必填 | 默认值 | 备注 |
| --- | --- | --- | --- | --- |
| `group.groupName` | 群名称 | 是 | 无 | 当前接口唯一硬校验字段 |
| `group.wxInstanceId` | 业务微信实例 ID | 是 | 无 | 缺失会导致保存失败 |
| `group.userId` | 业务用户 ID | 是 | 无 | 缺失会导致保存失败 |
| `group.tenantId` | 业务租户 ID | 是 | 无 | 缺失会导致保存失败 |
| `group.memberCount` | 群人数 | 否 | 空 | |
| `group.remark` | 系统备注 | 否 | 空 | |
| `tagNames` | 标签名称列表 | 否 | 空 | 主要用于 `wx_group.tags` 展示字符串 |
| `tagIds` | 标签 ID 列表 | 否 | 空 | 仅该字段会写入 `wx_group_tag` 关联表 |

##### 2.8.5 返回值说明

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `code` | `number` | 业务状态码 |
| `msg` | `string` | 返回消息 |
| `data.groupId` | `string` | 群聊 ID |
| `data.groupName` | `string` | 群名称 |

#### 2.8.1 从微信导入群聊（仅微信）

- **方法**：`POST`
- **路径**：`/api/get_groups`
- **用途**：从微信客户端导入群聊信息

##### 导入群聊 Body 字段

| 字段 | 含义 | 必填 | 默认值 | 备注 |
| --- | --- | --- | --- | --- |
| `userWxName` | 要导入的微信名称 | 是 | 无 | |
| `wxVersion` | 微信版本 | 否 | `"3"` | 微信 4.x 请传 `"4"` |
| `hwnd` | 微信窗口句柄 | `wxVersion="4"` 时必填 | `0` | 从“获取登录的微信”接口取 |
| `wxInstanceId` | 业务微信实例 ID | 否 | 空 | 建议传，便于导入进度事件关联 |
| `userId` | 业务用户 ID | 否 | `` | 不传时会回落到请求头注入 |
| `tenantId` | 业务租户 ID | 否 | `` | 不传时最终回落到 `1` |
| `optSleepTimeStart` | 群聊遍历最小等待秒数 | 否 | `0` | |
| `optSleepTimeEnd` | 群聊遍历最大等待秒数 | 否 | `0` | 小于起始值时会自动对齐 |

##### 2.8.1 返回值说明

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `code` | `number` | 业务状态码 |
| `msg` | `string` | 返回消息 |
| `data.taskId` | `string\|null` | 导入任务 ID |
| `data.monitorEndpoint` | `string\|null` | 进度查询地址 |

#### 2.8.2 查询群聊（仅系统）

- **方法**：`GET`
- **路径**：`/api/groups`
- **用途**：分页查询本地群聊数据

##### 查询群聊 Query 字段

| 字段 | 含义 | 必填 | 默认值 | 备注 |
| --- | --- | --- | --- | --- |
| `wxInstanceId` | 业务微信实例 ID | 是 | 无 | |
| `userId` | 业务用户 ID | 是 | 无 | |
| `tenantId` | 业务租户 ID | 否 | 请求头 `tenant-id` | 代码中不传时回落到请求头 |
| `keyword` | 关键词 | 否 | 空 | |
| `limit` | 分页大小 | 否 | `50` | |
| `offset` | 分页偏移量 | 否 | `0` | |

##### 2.8.2 返回值说明

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `code` | `number` | 业务状态码 |
| `msg` | `string` | 返回消息 |
| `data.list` | `array` | 群聊列表 |
| `data.total` | `number` | 总条数 |

##### 2.8.2.1 群聊对象字段

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `id` | `string` | 群聊主键 ID |
| `groupName` | `string` | 群聊名称 |
| `wxNo` | `string` | 微信群 ID（群聊号） |
| `wxInstanceId` | `string` | 微信实例 ID |
| `tenantId` | `number` | 租户 ID |
| `userId` | `string` | 用户 ID |
| `memberCount` | `number` | 群成员数量 |
| `remark` | `string` | 备注 |
| `tags` | `string` | 标签（以逗号分隔） |
| `createdTime` | `string` | 创建时间（ISO 8601 格式） |
| `updateTime` | `string` | 更新时间（ISO 8601 格式） |
| `source` | `string` | 数据来源（`wx`=微信导入，`manual`=手动添加） |
| `version` | `number` | 版本号（用于并发控制） |

#### 2.8.3 更新群聊标签（仅系统）

- **方法**：`POST`
- **路径**：`/api/groups/batch_tags`
- **用途**：批量更新群聊标签

##### 更新群聊标签 Body 字段

| 字段 | 含义 | 必填 | 默认值 | 备注 |
| --- | --- | --- | --- | --- |
| `groupIds` | 群聊 ID 列表 | 是 | 空 | |
| `tagIds` | 标签 ID 列表 | 否 | 空 | **仅该字段会写入/删除 `wx_group_tag` 关联表** |
| `tagNames` | 标签名称列表 | 否 | 空 | 主要用于更新 `wx_group.tags` 展示字符串；不会按名称自动补 `tagId` |
| `mode` | 标签操作模式 | 否 | `"replace"` | `replace` / `add` / `remove` |
| `userId` | 业务用户 ID | 是 | 无 | |
| `tenantId` | 业务租户 ID | 否 | 请求头 `tenant-id` | 代码中不传时回落到请求头 |

> 与联系人批量改标签一致：仅传 `tagNames` 不会新增 `wx_group_tag` 关联；要保证标签筛选与按标签群发准确，请传 `tagIds`。

##### 2.8.3 返回值说明

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `code` | `number` | 业务状态码 |
| `msg` | `string` | 操作结果说明 |
| `data.updated` | `number\|null` | 受影响群聊数量（若实现返回） |

#### 2.8.4 删除群聊（仅系统）

- **方法**：`DELETE`
- **路径**：`/api/groups/{groupId}`

##### 删除群聊 Path 字段

| 字段 | 含义 | 必填 |
| --- | --- | --- |
| `groupId` | 群聊 ID | 是 |

##### 2.8.4 返回值说明

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `code` | `number` | 业务状态码 |
| `msg` | `string` | 删除结果说明 |
| `data` | `object\|null` | 删除结果 |

---

## 5. 群发消息

### 5.1 立即群发

- **方法**：`POST`
- **路径**：`/api/sendmsg_v2`
- **用途**：立即执行好友/群聊群发

#### 立即群发 Body 字段

| 字段 | 含义 | 必填 | 默认值 | 备注 |
| --- | --- | --- | --- | --- |
| `userWxName` | 当前发送微信名称 | 是 | 无 | |
| `wxInstanceId` | 业务微信实例 ID | 建议必填 | 空 | 强烈建议传入，便于链路闭环 |
| `wxVersion` | 微信版本 | 否 | `"3"` | 微信 4.x 建议显式传 `"4"` |
| `hwnd` | 微信窗口句柄 | `wx4` 建议必填 | 空 | 建议与 `userWxName` 同时传，减少运行态定位失败 |
| `selectMode` | 选择模式 | 是 | 无 | `contacts` / `groups` / `tags` / `group_tags` |
| `receiverMode` | 发送范围模式 | 否 | `include` | `include` / `exclude` |
| `selectedContacts` | 联系人对象列表 | `selectMode=contacts` 时必填 | 空 | 详情见 `selectedContacts[]` |
| `selectedGroups` | 群聊选择源 | `selectMode=groups` 时必填 | 空 | 详情见 `selectedGroups[]` |
| `selectedTags` | 标签 ID 列表 | `selectMode=tags/group_tags` 时必填 | 空 | |
| `tagMatchAll` | 标签是否全匹配 | 否 | `false` | 标签模式使用 |
| `messages` | 消息数组 | 是 | 空 | 至少一条 |
| `messages[].msg` | 文本内容 | 否 | 空 | 可与图片/文件组合 |
| `messages[].msgImgUrl` | 图片远程 URL（http/https） | 否 | 空 | 必须是服务端可访问地址，不支持本地路径（如 `C:/xx/a.png`） |
| `messages[].msgFileUrl` | 文件远程 URL（http/https） | 否 | 空 | 必须是服务端可访问地址，不支持本地路径（如 `D:/docs/a.pdf`） |
| `sendSleepTimeStart` | 联系人之间最小等待秒数 | 否 | 业务默认值 | |
| `sendSleepTimeEnd` | 联系人之间最大等待秒数 | 否 | 业务默认值 | |
| `optSleepTimeStart` | 单步操作最小等待秒数 | 否 | 业务默认值 | |
| `optSleepTimeEnd` | 单步操作最大等待秒数 | 否 | 业务默认值 | |
| `checkDelete` | 是否顺带检测删好友 | 否 | `0` | `1` 表示开启 |
| `relaxedMode` | 是否启用宽松模式 | 否 | `false` | 为 `true` 时，定时任务将启用30秒倒计时后自动发送，同时每500人一次的二次确认提示将不再弹出 |

#### `selectedContacts[]` 对象字段（`selectMode=contacts`）

| 字段 | 含义 | 必填 | 备注 |
| --- | --- | --- | --- |
| `id` | 联系人 ID | 建议必填 | 用于去重、成功记录查询；代码里会优先用它做 key |
| `wxNo` | 微信号 | 建议必填 | `id` 为空时会退回用它做 key；两者至少传一个，否则会被过滤掉 |
| `nickName` | 昵称 | 否 | 用于发送时的展示名/称呼回退：`name` 为空时会优先取它，`{昵称}` 变量会被替换，如果是群发群，nickName 会被用作群名称 |
| `callName` | 称呼 | 否 | 会原样传给发送引擎，用于 `{称呼}` 变量替换 |
| `remark` | 备注 | 否 | 会原样传给发送引擎和日志,`{备注}` 变量会被替换 |
| `type` | 目标类型 | 否 | 好友传 `0`；群聊用 `1` |

#### `selectedGroups[]` 对象字段（`selectMode=groups`）

| 字段 | 含义 | 必填 | 备注 |
| --- | --- | --- | --- |
| `id` | 群聊 ID | 建议必填 | 用于去重和成功记录查询 |
| `name` | 群名 | 否 | 群聊名称 |
| `remark` | 群备注 | 否 | 会传给发送引擎和日志 |
| `type` | 目标类型 | 否 | 群聊固定为 `1` |

#### 4.1 返回值说明

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `code` | `number` | 业务状态码 |
| `msg` | `string` | 执行结果说明 |
| `data.logId` | `string\|number\|null` | 群发日志 ID |
| `data.taskId` | `string\|null` | 执行任务 ID（若异步） |
| `data.successCount` | `number\|null` | 成功数量 |
| `data.failedCount` | `number\|null` | 失败数量 |

---

### 5.2 定时群发

- **方法**：`POST`
- **路径**：`/api/scheduled_task/create`
- **用途**：创建 `taskType=message_sending` 的定时群发任务

#### 定时群发包装层字段

| 字段 | 含义 | 必填 | 默认值 |
| --- | --- | --- | --- |
| `wxName` | 当前任务归属微信名称 | 是 | 无 |
| `wxId` | 业务微信实例 ID | 是 | 无 |
| `taskName` | 任务名称 | 是 | 无 |
| `taskType` | 任务类型 | 是 | `message_sending` |
| `triggerType` | 触发方式 | 是 | 无 |
| `triggerConf` | 触发配置 | 是 | `{}` |
| `executeTime` | 定时执行时间 | 条件必填 | `date` 触发时必填；`cron` 可为 `null` |
| `relaxedMode` | 是否启用宽松模式 | 否 | `false` |
| `payload` | 实际业务载荷 | 是 | 无 |
| `tenantId` | 租户 ID（包装层） | 建议传 | 无 |
| `userId` | 用户 ID（包装层） | 建议传 | 无 |
| `wxInstanceId` | 微信实例 ID（包装层） | 建议传 | 无 |
| `userWxName` | 微信昵称（包装层） | 建议传 | 无 |

#### 定时群发 `triggerConf` 字段

| 触发方式 | 字段 |
| --- | --- |
| `date` | 可传空对象 `{}` |
| `cron` | `second`、`minute`、`hour`、`day`、`month`、`day_of_week` |

> 注意：当前调度执行器仅处理 `date` 与 `cron`。
>
> 使用 APS Trigger 结构：
>
> - 每 N 分钟：`{"second":"0","minute":"*/N","hour":"*","day":"*","month":"*","day_of_week":"*"}`
> - 每 N 小时：`{"second":"0","minute":"0","hour":"*/N","day":"*","month":"*","day_of_week":"*"}`
> - 每天固定时刻：`{"second":"0","minute":"30","hour":"9","day":"*","month":"*","day_of_week":"*"}`

#### 定时群发 `payload` 字段

与 **4.1 立即群发** 基本一致，至少建议包含：

- `userWxName`
- `wxInstanceId`
- `wxVersion`（微信 4.x 建议传 `"4"`）
- `hwnd`（`wx4` 建议必填）
- `selectMode`
- `receiverMode`
- `selectedContacts` / `selectedGroups` / `selectedTags`
- `messages`
- `tenantId`
- `userId`
- `scopeSummary`
- `plannedUsage`
- `relaxedMode`

#### `scopeSummary` 说明（可选）

`scopeSummary` 是前端用于展示“发送范围概要”的对象，不参与后端调度判定，属于可选增强字段。

##### 建议结构

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `targetType` | `string` | 目标类型：`好友` / `群` |
| `selectBy` | `string` | 选择方式：`按选择` / `按标签` |
| `receiverMode` | `string` | 发送范围：`include` / `exclude` |
| `tags` | `string[]` | 选中的标签名称列表 |
| `total` | `number` | 预计发送数量 |
| `excluded` | `number` | 排除数量（`exclude` 模式） |
| `cooldownFiltered` | `number` | 群冷却过滤数量（群相关模式） |
| `unit` | `string` | 数量单位：`位好友` / `个群` |
| `text` | `string` | 最终展示文案（UI 直接显示） |

##### 示例

```json
{
  "scopeSummary": {
    "targetType": "好友",
    "selectBy": "按选择",
    "receiverMode": "include",
    "tags": [],
    "total": 1,
    "excluded": 0,
    "cooldownFiltered": 0,
    "unit": "位好友",
    "text": "发给好友｜按选择｜标签：无｜预计发送 1位好友"
  }
}
```

#### 4.2 返回值说明

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `code` | `number` | 业务状态码，成功为 `200` |
| `msg` | `string` | 返回消息 |
| `data.taskId` | `string` | 定时任务 ID |
| `data.status` | `string` | 初始状态，常见 `WAITING` |

---

### 5.3 群发历史查询

- **方法**：`GET`
- **路径**：`/api/mass_send/tasks`

#### 群发历史查询 Query 字段

| 字段 | 含义 | 必填 | 默认值 |
| --- | --- | --- | --- |
| `page` | 页码 | 否 | `1` |
| `pageSize` | 每页条数 | 否 | `20` |
| `userId` | 业务用户 ID | 是 | 无 |
| `tenantId` | 业务租户 ID | 是 | 无 |

#### 4.3 返回值说明

**直出型返回，无 `code/msg` 外层包装**

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `items` | `array` | 群发历史列表 |
| `total` | `number` | 总条数 |

#### 4.3.1 群发历史对象字段

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `logId` | `string` | 群发日志 ID |
| `taskName` | `string` | 任务名称 |
| `wxName` | `string` | 微信昵称 |
| `wxInstanceId` | `string` | 微信实例 ID |
| `status` | `string` | 任务状态（`success`=成功，`failed`=失败，等） |
| `createdTime` | `string` | 创建时间（ISO 8601 格式） |
| `plannedTotal` | `number` | 计划发送数量 |
| `success` | `number` | 成功数量 |
| `failed` | `number` | 失败数量 |
| `total` | `number` | 实际处理总数 |

---

### 5.4 群发历史详情

- **方法**：`GET`
- **路径**：`/api/mass_send/{logId}/detail`

| 字段 | 含义 | 必填 |
| --- | --- | --- |
| `logId` | 群发日志 ID | 是 |

#### 5.4 返回值说明

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `code` | `number` | 业务状态码 |
| `msg` | `string` | 返回消息 |
| `data` | `object` | 群发历史详情 |

---

### 5.5 失败补发 / 断点续发

- **方法**：`POST`
- **路径**：`/api/mass_send/execute`

#### 失败补发或断点续发 Body 字段

| 字段 | 含义 | 必填 | 默认值 |
| --- | --- | --- | --- |
| `logId` | 群发日志 ID | 是 | 无 |
| `mode` | 执行模式 | 是 | 无 |
| `wxInfo` | 运行时微信信息 | 是 | 无 |
| `wxInfo.wxName` | 当前微信名称 | 是 | 无 |
| `wxInfo.hwnd` | 当前微信句柄 | 是 | 无 |

`mode` 常用值：

- `retry_failed`
- `resume`

#### 4.5 返回值说明

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `code` | `number` | 业务状态码 |
| `msg` | `string` | 执行结果说明 |
| `data` | `object\|null` | 补发/续发执行结果 |

---

### 5.6 删除群发历史

- **方法**：`DELETE`
- **路径**：`/api/mass_send/{logId}`

| 字段 | 含义 | 必填 |
| --- | --- | --- |
| `logId` | 群发日志 ID | 是 |

#### 4.6 返回值说明

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `code` | `number` | 业务状态码 |
| `msg` | `string` | 删除结果说明 |
| `data` | `object\|null` | 删除结果 |

---

## 6. 自动加好友

### 工作流程说明

加好友任务支持两种使用方式：

- **仅创建任务（用于准备阶段）**：调用 `/api/friend_apply/create`，设置 `autoStart: false`，任务创建后状态为 `pending`，微信无动作。
- **创建并立即执行（推荐）**：调用 `/api/friend_apply/create`，设置 `autoStart: true`，任务创建后状态为 `running`，微信立即开始加好友。

### 6.1 创建加好友任务

- **方法**：`POST`
- **路径**：`/api/friend_apply/create`
- **说明**：创建加好友任务，根据 `autoStart` 参数决定是否立即执行

#### autoStart 参数说明

| 值 | 行为 | 用途 |
| --- | --- | --- |
| `false` | 只创建任务，任务状态为 `pending`，微信不会立即有动作 | 先创建任务配置，稍后再手动执行 |
| `true` | 创建任务后立即执行，任务状态变为 `running`，微信立即开始加好友 | 创建并立即执行，适合自动流程 |

#### 创建加好友任务 Body 字段

| 字段 | 含义 | 必填 | 默认值 | 备注 |
| --- | --- | --- | --- | --- |
| `taskName` | 任务名称 | 否 | 空 | |
| `userWxName` | 当前操作微信名称 | 是 | 无 | |
| `userId` | 用户 ID | 是 | 无 | |
| `tenantId` | 租户 ID | 是 | 无 | |
| `wxInstanceId` | 业务微信实例 ID | 建议必填 | 空 | 建议始终传入 |
| `wxVersion` | 微信版本 | 是 | `3` | wx4 时传 `4` |
| `hwnd` | 窗口句柄 | 是 | `0` | 从 wx 对象获取 |
| `selectedResources` | 资源列表 | 是 | 空 | 至少一条 |
| `selectedResources[].id` | 资源 ID | 是 | 无 | |
| `selectedResources[].keyword` | 搜索关键词 / 目标微信号 | 是 | 无 | |
| `selectedResources[].nickname` | 资源昵称 | 否 | 空 | |
| `applyMessage` | 好友申请内容 | 是 | 无 | 最长 60 字 |
| `tags` | 新好友标签列表 | 否 | `[]` | |
| `permission` | 权限申请文案 | 否 | `朋友圈` | |
| `sleepTimeMin` | 联系人间最小等待秒数 | 否 | `5` | |
| `sleepTimeMax` | 联系人间最大等待秒数 | 否 | `8` | |
| `optSleepTimeStart` | 单步最小等待秒数 | 否 | `0.5` | |
| `optSleepTimeEnd` | 单步最大等待秒数 | 否 | `1` | |
| `stopIfTooManyRequests` | 是否遇到频繁提示后停止 | 否 | `true` | |
| `remark` | 备注信息 | 否 | 空 | |
| `templateId` | 模板 ID | 否 | 空 | |
| `autoStart` | 创建后是否立即执行 | 否 | `false` | 见上表说明 |

#### 创建任务响应

```json
{
  "code": 200,
  "data": {
    "taskId": "runtime-task-id",
    "logId": "friend-apply-log-id",
    "status": "pending"
  }
}
```

#### 7.1 返回值说明

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `code` | `number` | 业务状态码 |
| `msg` | `string\|null` | 返回消息 |
| `data.taskId` | `string` | 运行任务 ID |
| `data.logId` | `string` | 日志 ID |
| `data.status` | `string` | 初始状态：`pending` / `running` |

---

### 6.2 定时加好友

- **方法**：`POST`
- **路径**：`/api/scheduled_task/create`
- **用途**：创建 `taskType=friend_apply` 的定时任务

#### 定时加好友包装层字段

同 **4.2 定时群发**，但：

- `taskType` 固定为 `friend_apply`

#### 定时加好友 `payload` 字段

| 字段 | 含义 | 必填 | 默认值 | 备注 |
| --- | --- | --- | --- | --- |
| `userWxName` | 当前微信名称 | 是 | 无 | |
| `userId` | 用户 ID | 是 | 无 | |
| `tenantId` | 租户 ID | 是 | 无 | |
| `wxInstanceId` | 业务微信实例 ID | 是 | 无 | |
| `wxVersion` | 微信版本 | 是 | `3` | wx4 时传 `4` |
| `hwnd` | 窗口句柄 | 是 | `0` | 从 wx 对象获取 |
| `selectedResources` | 资源列表 | 是 | 空 | 至少一条 |
| `applyMessage` | 申请内容 | 是 | 无 | |
| `optSleepTimeStart` | 单步最小等待秒数 | 否 | `0.5` | |
| `optSleepTimeEnd` | 单步最大等待秒数 | 否 | `1` | |
| `sleepTimeMin` | 联系人间最小等待秒数 | 否 | `5` | |
| `sleepTimeMax` | 联系人间最大等待秒数 | 否 | `8` | |
| `autoStart` | 是否立即启动 | 否 | `false` | |

> 本次已检查并补齐 Postman 中 3 个定时加好友示例的 `payload.wxInstanceId`。

#### 6.2 返回值说明

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `code` | `number` | 业务状态码 |
| `msg` | `string` | 返回消息 |
| `data.taskId` | `string` | 定时任务 ID |
| `data.status` | `string` | 初始状态，常见 `WAITING` |

---

### 6.3 加好友历史查询

- **方法**：`GET`
- **路径**：`/api/friend_apply/tasks`

#### 加好友历史查询 Query 字段

| 字段 | 含义 | 必填 | 默认值 |
| --- | --- | --- | --- |
| `page` | 页码 | 否 | `1` |
| `pageSize` | 每页条数 | 否 | `20` |
| `userId` | 业务用户 ID | 是 | 无 |
| `tenantId` | 业务租户 ID | 是 | 无 |

#### 6.3 返回值说明

**直出型返回，无 `code/msg` 外层包装**

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `items` | `array` | 加好友任务历史列表 |
| `total` | `number` | 总条数 |

#### 6.3.1 任务历史对象字段

> **注意**：当前测试环境中无加好友历史任务数据，以下字段基于代码实现推导，建议实际调用接口时验证

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `logId` | `string` | 任务日志 ID |
| `taskName` | `string` | 任务名称 |
| `wxName` | `string` | 微信昵称 |
| `wxInstanceId` | `string` | 微信实例 ID |
| `status` | `number` | 任务状态 |
| `createdTime` | `string` | 创建时间（ISO 8601 格式） |
| `runtimeTaskId` | `string` | 运行时任务 ID |

---

### 6.4 加好友历史详情

- **方法**：`GET`
- **路径**：`/api/friend_apply/{logId}/detail`

| 字段 | 含义 | 必填 |
| --- | --- | --- |
| `logId` | 加好友日志 ID | 是 |

#### 6.4 返回值说明

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `code` | `number` | 业务状态码 |
| `msg` | `string` | 返回消息 |
| `data` | `object` | 加好友历史详情 |

---

### 6.5 资源上传（批量导入）

- **方法**：`POST`
- **路径**：`/api/friend-apply-resources/import`

#### 资源上传 Body 字段

| 字段 | 含义 | 必填 | 默认值 |
| --- | --- | --- | --- |
| `rows` | 资源行数组 | 是 | 空 |
| `rows[].keyword` | 资源关键词 | 是 | 无 |
| `rows[].nickname` | 资源昵称 | 否 | 空 |
| `userId` | 业务用户 ID | 是 | 无 |
| `tenantId` | 业务租户 ID | 是 | 无 |

#### 6.5 返回值说明

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `code` | `number` | 业务状态码 |
| `msg` | `string` | 导入结果说明 |
| `data.successCount` | `number\|null` | 导入成功条数 |
| `data.failedCount` | `number\|null` | 导入失败条数 |

---

### 6.6 资源添加

- **方法**：`POST`
- **路径**：`/api/friend-apply-resources`

#### 资源添加 Body 字段

| 字段 | 含义 | 必填 | 默认值 |
| --- | --- | --- | --- |
| `keyword` | 资源关键词 | 是 | 无 |
| `nickname` | 资源昵称 | 否 | 空 |
| `userId` | 业务用户 ID | 是 | 无 |
| `tenantId` | 业务租户 ID | 是 | 无 |

#### 6.6 返回值说明

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `code` | `number` | 业务状态码 |
| `msg` | `string` | 返回消息 |
| `data.resourceId` | `string\|number\|null` | 新增资源 ID |

---

### 6.7 资源查询

- **方法**：`GET`
- **路径**：`/api/friend-apply-resources`

#### 资源查询 Query 字段

| 字段 | 含义 | 必填 | 默认值 |
| --- | --- | --- | --- |
| `page` | 页码 | 否 | `1` |
| `pageSize` | 每页条数 | 否 | `20` |
| `userId` | 业务用户 ID | 是 | 无 |
| `tenantId` | 业务租户 ID | 是 | 无 |

#### 6.7 返回值说明

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `code` | `number` | 业务状态码 |
| `msg` | `string` | 返回消息 |
| `data.items` | `array` | 资源列表 |
| `data.total` | `number` | 总条数 |
| `data.page` | `number` | 当前页码 |
| `data.pageSize` | `number` | 每页条数 |

#### 6.7.1 资源对象字段

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `id` | `number` | 资源 ID |
| `keyword` | `string` | 资源关键词（搜索词） |
| `nickname` | `string` | 资源昵称（展示名） |
| `remark` | `string` | 资源备注 |
| `userId` | `string` | 用户 ID |
| `tenantId` | `number` | 租户 ID |
| `finalStatus` | `number` | 最终处理状态（`0`=待处理，`1`=成功，`-1`=失败） |
| `createdTime` | `string` | 创建时间（ISO 8601 格式） |
| `updateTime` | `string` | 更新时间（ISO 8601 格式） |

---

### 6.8 资源删除

- **方法**：`DELETE`
- **路径**：`/api/friend-apply-resources/{resourceId}`

#### 资源删除 Path / Query 字段

| 字段 | 含义 | 必填 |
| --- | --- | --- |
| `resourceId` | 资源 ID | 是 |
| `userId` | 业务用户 ID | 是 |
| `tenantId` | 业务租户 ID | 是 |

#### 6.8 返回值说明

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `code` | `number` | 业务状态码 |
| `msg` | `string` | 删除结果说明 |
| `data` | `object\|null` | 删除结果 |

---

## 7. 发朋友圈

### 7.1 发朋友圈（立即）

- **方法**：`POST`
- **路径**：`/api/post_moments`

#### 发朋友圈 Body 顶层

请求体是数组，每个元素代表一组朋友圈发送任务。

| 字段 | 含义 | 必填 | 默认值 |
| --- | --- | --- | --- |
| `[]` | 朋友圈任务数组 | 是 | 空 |
| `[].userWxName` | 当前微信运行时对象 | 是 | 无 |
| `[].userWxName.wxName` | 微信名称 | 是 | 无 |
| `[].userWxName.wxVersion` | 微信版本 | 是 | 无 |
| `[].userWxName.hwnd` | 微信句柄 | 是 | 无 |
| `[].userId` | 业务用户 ID | 是 | 无 |
| `[].tenantId` | 业务租户 ID | 是 | 无 |
| `[].moments` | 朋友圈内容数组 | 是 | 空 |
| `[].deleteFirst` | 是否先删除草稿/旧内容 | 否 | `false` |
| `[].optStartTime` | 单步最小等待秒数 | 否 | `0` |
| `[].optEndTime` | 单步最大等待秒数 | 否 | `1` |

#### 发朋友圈 `moments[]` 字段

| 字段 | 含义 | 必填 | 默认值 |
| --- | --- | --- | --- |
| `momentType` | 朋友圈类型 | 是 | 无 |
| `momentContent` | 文本内容 | 视类型而定 | 空 |
| `momentImages` | 图片列表 | 否 | `[]` |
| `whoCanSee` | 可见范围配置 | 是 | 无 |
| `whoCanSee.type` | 可见类型 | 是 | 无 |
| `whoCanSee.list.tags` | 标签可见名单 | 否 | `[]` |
| `whoCanSee.list.friends` | 好友可见名单 | 否 | `[]` |

> 说明：`momentImages` 中的图片必须使用**本地文件路径**（例如 `C:/images/a.jpg`），远程 URL 无法成功发布。

#### 6.1 返回值说明

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `code` | `number` | 业务状态码 |
| `msg` | `string` | 执行结果说明 |
| `data.logId` | `string\|number\|null` | 日志 ID |
| `data.taskId` | `string\|null` | 异步任务 ID（若异步） |

---

### 7.2 定时发朋友圈

- **方法**：`POST`
- **路径**：`/api/scheduled_task/create`
- **用途**：创建 `taskType=post_moment_batch` 的定时任务

#### 定时发朋友圈包装层字段

同 **4.2 定时群发**，但：

- `taskType` 固定为 `post_moment_batch`

#### 定时发朋友圈 `payload` 字段

`payload` 为数组，字段与 **7.1 发朋友圈（立即）** 相同。

#### 7.2 返回值说明

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `code` | `number` | 业务状态码 |
| `msg` | `string` | 返回消息 |
| `data.taskId` | `string` | 定时任务 ID |
| `data.status` | `string` | 初始状态 |

---

### 7.3 历史查询

- **方法**：`GET`
- **路径**：`/api/moment/tasks`

#### 朋友圈历史查询 Query 字段

| 字段 | 含义 | 必填 | 默认值 |
| --- | --- | --- | --- |
| `page` | 页码 | 否 | `1` |
| `pageSize` | 每页条数 | 否 | `20` |
| `userId` | 业务用户 ID | 是 | 无 |
| `tenantId` | 业务租户 ID | 是 | 无 |

#### 7.3 返回值说明

**直出型返回，无 `code/msg` 外层包装**

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `items` | `array` | 朋友圈历史列表 |
| `total` | `number` | 总条数 |

#### 7.3.1 朋友圈历史对象字段

> **注意**：当前测试环境中无朋友圈历史任务数据，以下字段基于代码实现推导，建议实际调用接口时验证

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `id` | `string` | 任务日志 ID |
| `name` | `string` | 任务名称 |
| `wxInstanceName` | `string` | 微信昵称 |
| `wxInstanceId` | `string` | 微信实例 ID |
| `userId` | `string` | 用户 ID |
| `tenantId` | `number` | 租户 ID |
| `runtimeTaskId` | `string` | 运行时任务 ID |
| `mode` | `string` | 执行模式 |
| `status` | `string` | 任务状态 |
| `plannedTotal` | `number` | 计划发送数量 |
| `processed` | `number` | 已处理数量 |
| `success` | `number` | 成功数量 |
| `failed` | `number` | 失败数量 |
| `createdTime` | `string` | 创建时间（ISO 8601 格式） |
| `updateTime` | `string` | 更新时间（ISO 8601 格式） |

---

### 7.4 历史详情查询

- **方法**：`GET`
- **路径**：`/api/moment/tasks/{logId}/detail`

| 字段 | 含义 | 必填 |
| --- | --- | --- |
| `logId` | 朋友圈日志 ID | 是 |

#### 7.4 返回值说明

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `code` | `number` | 业务状态码 |
| `msg` | `string` | 返回消息 |
| `data` | `object` | 历史详情 |

---

### 7.5 失败补发 / 断点续发

- **方法**：`POST`
- **路径**：`/api/moment/execute`

#### 朋友圈重试执行 Body 字段

| 字段 | 含义 | 必填 | 默认值 |
| --- | --- | --- | --- |
| `logId` | 朋友圈日志 ID | 是 | 无 |
| `mode` | 执行模式 | 是 | 无 |
| `wxInfo` | 运行时微信信息 | 是 | 无 |
| `wxInfo.wxName` | 微信名称 | 是 | 无 |
| `wxInfo.hwnd` | 微信句柄 | 是 | 无 |

#### 7.5 返回值说明

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `code` | `number` | 业务状态码 |
| `msg` | `string` | 执行结果说明 |
| `data` | `object\|null` | 重试执行结果 |

---

## 8. 快捷回复

### 8.1 快捷回复消息

- **方法**：`POST`
- **路径**：`/api/mcp/send_current_chat_reply`
- **用途**：向当前激活聊天窗口直接发送文本（无需先识别当前微信）

#### 快捷回复 Body 字段

| 字段 | 含义 | 必填 | 默认值 | 备注 |
| --- | --- | --- | --- | --- |
| `replyText` | 要发送的文本内容 | 是 | 无 | |
| `clear` | 发送前是否先清空输入框 | 否 | `true` | |

> 兼容说明：接口仍兼容 `hwnd` / `userWxName` 入参；不传时默认自动识别当前前台微信窗口。

#### 8.1 返回值说明

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `code` | `number` | 业务状态码 |
| `msg` | `string` | 发送结果说明 |
| `data.success` | `boolean\|null` | 是否发送成功 |
| `data.error` | `string\|null` | 失败原因（失败时） |

---

## 9. 自动通过好友申请

### 9.1 通过好友申请（立即）

- **方法**：`POST`
- **路径**：`/api/friend_accept/create`

#### 通过好友申请 Body 字段

| 字段 | 含义 | 必填 | 默认值 | 备注 |
| --- | --- | --- | --- | --- |
| `taskName` | 任务名称 | 否 | 空 | |
| `userWxName` | 当前微信名称 | 是 | 无 | |
| `wxInstanceId` | 业务微信实例 ID | 建议必填 | 空 | |
| `greetEnabled` | 是否启用自动打招呼 | 否 | `true` | |
| `greetMessages` | 打招呼消息数组 | `greetEnabled=true` 且未传模板时必填 | `[]` | |
| `greetMessages[].msg` | 文本内容 | 否 | 空 | |
| `greetMessages[].msgImgUrl` | 图片远程 URL（http/https） | 否 | 空 | 必须是服务端可访问地址，不支持本地路径 |
| `greetMessages[].msgFileUrl` | 文件远程 URL（http/https） | 否 | 空 | 必须是服务端可访问地址，不支持本地路径 |
| `greetTemplateId` | 打招呼模板 ID | 否 | 空 | 与 `greetMessages` 二选一可满足 |
| `greetWaitSeconds` | 通过后等待几秒再打招呼 | 否 | `3` | |
| `tags` | 标签列表 | 否 | `[]` | |
| `permission` | 权限文案 | 否 | `朋友圈` | |
| `autoStart` | 创建后是否立即执行 | 否 | `false` | |

#### 9.1 返回值说明

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `code` | `number` | 业务状态码 |
| `msg` | `string\|null` | 返回消息 |
| `data.taskId` | `string` | 运行任务 ID |
| `data.logId` | `string` | 日志 ID |
| `data.status` | `string` | 初始状态 |

---

### 9.2 定时通过好友申请

- **方法**：`POST`
- **路径**：`/api/scheduled_task/create`
- **用途**：创建 `taskType=friend_accept` 的定时任务

#### 定时通过好友申请包装层字段

同 **4.2 定时群发**，但：

- `taskType` 固定为 `friend_accept`

#### 定时通过好友申请 `payload` 字段

| 字段 | 含义 | 必填 | 默认值 |
| --- | --- | --- | --- |
| `userWxName` | 当前微信名称 | 是 | 无 |
| `wxInstanceId` | 业务微信实例 ID | 是 | 无 |
| `greetEnabled` | 是否启用打招呼 | 否 | `true` |
| `greetMessages` | 打招呼消息数组 | `greetEnabled=true` 时建议传 | `[]` |
| `greetWaitSeconds` | 打招呼等待秒数 | 否 | `3` |

#### 9.2 返回值说明

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `code` | `number` | 业务状态码 |
| `msg` | `string` | 返回消息 |
| `data.taskId` | `string` | 定时任务 ID |
| `data.status` | `string` | 初始状态 |

---

## 10. 快捷回复

- **方法**：`POST`
- **路径**：`/api/scheduled_task/create`
- **用途**：创建 `taskType=quick_reply` 的定时任务

### 10.1 快捷回复包装层字段

同 **4.2 定时群发**。

### 10.2 返回值说明

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `code` | `number` | 业务状态码 |
| `msg` | `string` | 返回消息 |
| `data.taskId` | `string` | 定时任务 ID |
| `data.status` | `string` | 初始状态 |

---

## 11. 定时任务管理

> 本章只收录定时任务管理接口（除创建 `POST /api/scheduled_task/create` 之外）。

### 11.1 定时任务查询

- **方法**：`GET`
- **路径**：`/api/scheduled_task/list`

| Query 字段 | 含义 | 必填 | 默认值 |
| --- | --- | --- | --- |
| `page` | 页码 | 否 | `1` |
| `pageSize` | 每页条数 | 否 | `10` |

#### 11.1 返回值

```json
{
  "code": 200,
  "data": {
    "items": [
      {
        "id": "9fd3...",
        "wxName": "码农",
        "taskName": "群发-07-02 23:17",
        "wxId": "local_132250",
        "taskType": "message_sending",
        "triggerType": "cron",
        "triggerConf": {"second":"0","minute":"*/6","hour":"*","day":"*","month":"*","day_of_week":"*"},
        "executeTime": null,
        "payload": {},
        "relaxedMode": false,
        "status": "WAITING",
        "createdAt": "2026-07-02T23:17:09"
      }
    ],
    "total": 1,
    "page": 1,
    "pageSize": 10,
    "pages": 1
  }
}
```

#### 11.1 返回字段说明

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `code` | `number` | 业务状态码，`200` 表示成功 |
| `data.items` | `array` | 定时任务列表 |
| `data.items[].id` | `string` | 定时任务 ID |
| `data.items[].wxName` | `string` | 微信昵称 |
| `data.items[].taskName` | `string` | 任务名称 |
| `data.items[].wxId` | `string` | 微信实例 ID |
| `data.items[].taskType` | `string` | 任务类型，如 `message_sending` |
| `data.items[].triggerType` | `string` | 触发类型：`date` / `cron` |
| `data.items[].triggerConf` | `object\|null` | 触发配置 |
| `data.items[].executeTime` | `string\|null` | 首次执行时间；`cron` 可能为 `null` |
| `data.items[].payload` | `object\|array` | 业务载荷 |
| `data.items[].relaxedMode` | `boolean` | 是否宽松模式 |
| `data.items[].status` | `string` | 任务状态：`WAITING` / `PAUSED` / `CANCELLED` / `FINISHED` / `WAITING_CONFIRM` |
| `data.items[].createdAt` | `string` | 创建时间 |
| `data.total` | `number` | 总条数 |
| `data.page` | `number` | 当前页码 |
| `data.pageSize` | `number` | 每页条数 |
| `data.pages` | `number` | 总页数 |

### 11.2 定时任务修改

- **方法**：`POST`
- **路径**：`/api/scheduled_task/edit/{taskId}`
- **说明**：请求体结构与创建定时任务一致。

#### 11.2 路径参数

| 字段 | 含义 | 必填 |
| --- | --- | --- |
| `taskId` | 定时任务 ID | 是 |

#### 11.2 返回值

```json
{ "code": 200, "msg": "定时任务已更新" }
```

#### 11.2~11.10 返回字段说明（通用）

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `code` | `number` | 业务状态码，`200` 成功，`400/404` 失败 |
| `msg` | `string` | 结果说明或错误信息 |

> 适用接口：`edit/delete/cancel/pause/resume/batch_delete/confirm/delay/confirm_popup`。

### 11.3 定时任务删除

- **方法**：`POST`
- **路径**：`/api/scheduled_task/delete/{taskId}`

#### 11.3 路径参数

| 字段 | 含义 | 必填 |
| --- | --- | --- |
| `taskId` | 定时任务 ID | 是 |

#### 11.3 返回值

- 成功：`{"code": 200, "msg": "已删除"}`
- 不存在：`{"code": 404, "msg": "任务不存在"}`

### 11.4 定时任务停止（取消）

- **方法**：`POST`
- **路径**：`/api/scheduled_task/cancel/{taskId}`

#### 11.4 路径参数

| 字段 | 含义 | 必填 |
| --- | --- | --- |
| `taskId` | 定时任务 ID | 是 |

#### 11.4 返回值

```json
{ "code": 200, "msg": "已取消" }
```

### 11.5 定时任务暂停

- **方法**：`POST`
- **路径**：`/api/scheduled_task/pause/{taskId}`

#### 11.5 路径参数

| 字段 | 含义 | 必填 |
| --- | --- | --- |
| `taskId` | 定时任务 ID | 是 |

#### 11.5 返回值

```json
{ "code": 200, "msg": "已暂停" }
```

### 11.6 定时任务恢复

- **方法**：`POST`
- **路径**：`/api/scheduled_task/resume/{taskId}`

#### 11.6 路径参数

| 字段 | 含义 | 必填 |
| --- | --- | --- |
| `taskId` | 定时任务 ID | 是 |

#### 11.6 返回值

```json
{ "code": 200, "msg": "已恢复" }
```

### 11.7 定时任务批量删除

- **方法**：`POST`
- **路径**：`/api/scheduled_task/batch_delete`

#### 11.7 Body 字段

| 字段 | 含义 | 必填 | 默认值 |
| --- | --- | --- | --- |
| `ids` | 任务 ID 列表 | 是 | `[]` |

#### 11.7 返回值

- 成功：`{"code": 200, "msg": "批量删除成功"}`
- 参数为空：`{"code": 400, "msg": "请选择要删除的任务"}`

### 11.8 定时任务确认执行

- **方法**：`POST`
- **路径**：`/api/scheduled_task/confirm/{taskId}`

#### 11.8 路径参数

| 字段 | 含义 | 必填 |
| --- | --- | --- |
| `taskId` | 定时任务 ID | 是 |

#### 11.8 返回值

- 成功：`{"code": 200, "msg": "..."}`
- 失败：`{"code": 400, "msg": "..."}`

### 11.9 定时任务延迟执行

- **方法**：`POST`
- **路径**：`/api/scheduled_task/delay/{taskId}`

#### 11.9 路径参数

| 字段 | 含义 | 必填 |
| --- | --- | --- |
| `taskId` | 定时任务 ID | 是 |

#### 11.9 Body 字段

| 字段 | 含义 | 必填 | 默认值 |
| --- | --- | --- | --- |
| `delayMinutes` | 延迟分钟数 | 否 | `5` |

#### 11.9 返回值

```json
{ "code": 200, "msg": "已延迟 5 分钟" }
```

### 11.10 定时任务确认弹窗重开

- **方法**：`GET`
- **路径**：`/api/scheduled_task/confirm_popup/{taskId}`

#### 11.10 路径参数

| 字段 | 含义 | 必填 |
| --- | --- | --- |
| `taskId` | 定时任务 ID | 是 |

#### 11.10 返回值

- 成功：`{"code": 200}`
- 非待确认状态：`{"code": 400, "msg": "任务状态不支持确认"}`

### 11.11 定时任务执行记录列表

- **方法**：`GET`
- **路径**：`/api/scheduled_task/executions`

#### 执行记录 Query 字段

| 字段 | 含义 | 必填 | 默认值 |
| --- | --- | --- | --- |
| `taskId` | 任务 ID | 否 | 空 |
| `wxName` | 微信名称 | 否 | 空 |
| `status` | 执行状态 | 否 | 空 |
| `limit` | 条数 | 否 | `50` |
| `offset` | 偏移量 | 否 | `0` |

#### 11.11 返回值（直出结构）

```json
{
  "list": [
    {
      "executionId": "e-1",
      "taskId": "task-123",
      "fireTime": "2026-02-16T09:00:00",
      "actualRunTime": "2026-02-16T09:00:01",
      "endTime": "2026-02-16T09:00:20",
      "status": "SUCCESS",
      "skipReason": null,
      "errorMsg": null,
      "wxName": "张三",
      "taskType": "message_sending",
      "durationMs": 19000
    }
  ],
  "limit": 50,
  "offset": 0
}
```

#### 11.11 返回字段说明

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `list` | `array` | 执行记录列表 |
| `list[].executionId` | `string` | 执行记录 ID |
| `list[].taskId` | `string` | 对应的定时任务 ID |
| `list[].fireTime` | `string` | 计划触发时间 |
| `list[].actualRunTime` | `string\|null` | 实际开始执行时间 |
| `list[].endTime` | `string\|null` | 执行结束时间 |
| `list[].status` | `string` | 执行状态：`RUNNING` / `SUCCESS` / `FAILED` / `SKIPPED` |
| `list[].skipReason` | `string\|null` | 跳过原因（仅 `SKIPPED`） |
| `list[].errorMsg` | `string\|null` | 失败原因（仅 `FAILED`） |
| `list[].wxName` | `string` | 微信昵称 |
| `list[].taskType` | `string` | 任务类型 |
| `list[].durationMs` | `number\|null` | 执行耗时（毫秒） |
| `limit` | `number` | 本次查询条数上限 |
| `offset` | `number` | 本次查询偏移量 |

`status` 常见值：`RUNNING` / `SUCCESS` / `FAILED` / `SKIPPED`。

### 11.12 定时任务执行记录详情

- **方法**：`GET`
- **路径**：`/api/scheduled_task/execution/{executionId}`

#### 11.12 路径参数

| 字段 | 含义 | 必填 |
| --- | --- | --- |
| `executionId` | 执行记录 ID | 是 |

#### 11.12 返回值（直出结构）

```json
{
  "executionId": "e-1",
  "taskId": "task-123",
  "fireTime": "2026-02-16T09:00:00",
  "actualRunTime": "2026-02-16T09:00:01",
  "endTime": "2026-02-16T09:00:20",
  "status": "SUCCESS",
  "skipReason": null,
  "errorMsg": null,
  "wxName": "张三",
  "taskType": "message_sending",
  "durationMs": 19000
}
```

#### 11.12 返回字段说明

字段含义与 **11.11** 的 `list[]` 单项一致。

未找到返回：HTTP `404`，`{"error":"NOT_FOUND"}`。

### 11.13 定时任务未来触发时间

- **方法**：`GET`
- **路径**：`/api/scheduled_task/next_fire_times`

#### 未来触发时间 Query 字段

| 字段 | 含义 | 必填 | 默认值 |
| --- | --- | --- | --- |
| `taskId` | 任务 ID | 是 | 无 |
| `n` | 返回未来触发次数 | 否 | `5` |

#### 11.13 返回值（直出结构）

```json
{
  "taskId": "task-123",
  "triggerType": "cron",
  "nextFireTimes": [
    "2026-07-03 09:00:00",
    "2026-07-03 09:06:00"
  ]
}
```

#### 11.13 返回字段说明

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `taskId` | `string` | 定时任务 ID |
| `triggerType` | `string` | 触发类型；仅 `cron` 会返回未来时间 |
| `nextFireTimes` | `string[]` | 未来触发时间列表（格式：`YYYY-MM-DD HH:mm:ss`） |

错误场景：

- 缺少 `taskId`：HTTP `400`，`{"error":"taskId required"}`
- `n` 超范围（1~50）：HTTP `400`，`{"error":"n must be between 1 and 50"}`
- 任务不存在：HTTP `404`，`{"error":"TASK_NOT_FOUND"}`

---

## 12. 实时任务

### 12.1 实时任务查询

- **方法**：`GET`
- **路径**：`/api/get_running_tasks`

#### 12.1 返回值说明

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `code` | `number` | 业务状态码（包装型实现） |
| `msg` | `string` | 返回消息（包装型实现） |
| `data` | `array\|object` | 实时运行任务列表（包装型） |
| `[]` | `array` | 若为直出结构，则直接返回任务数组 |

### 12.2 实时任务停止

- **方法**：`GET`
- **路径**：`/api/stop_task/{taskId}`

#### 12.2 返回值说明

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `code` | `number` | 业务状态码 |
| `msg` | `string` | 停止结果说明 |
| `data` | `object\|null` | 停止结果 |

### 12.3 实时任务删除（若有）

- 当前没有“按任务 ID 的独立删除接口”。
- 可用批量停止/清理接口：
  - **方法**：`DELETE`
  - **路径**：`/api/wx/{wxName}/tasks`

#### 12.3 返回值说明

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `code` | `number` | 业务状态码 |
| `msg` | `string` | 清理结果说明 |
| `data.stopped` | `number\|null` | 已停止任务数量（若实现返回） |

---

## 13. 推荐调用顺序（客户端闭环）

1. `POST /api/auth/login` 获取 `accessToken`
2. 固定业务上下文：`tenantId`、`clientId`、`userId=0`
3. `GET /api/get_opened_wxs` 获取在线微信运行态信息
4. `POST /api/wx-bindings/bind` 生成业务 `wxInstanceId`
5. 后续联系人 / 群组 / 任务接口统一使用该 `wxInstanceId`

---

