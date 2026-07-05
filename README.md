# 5 分钟用 Postman 跑通群发

> **目标**：零基础，照着做，5 分钟内向一个微信好友发出第一条群发消息。
>
> **使用集合**：`MsgHelper_OpenAPI_场景化集合.postman_collection.json`（和本文档同目录）

---

## 前置条件（做完再打开 Postman）

### 1. 下载并安装 MsgHelper 与兼容微信

> 网盘地址由运营方提供，请联系运营方获取最新下载链接。
>
> - 百度网盘：`https://pan.baidu.com/s/1-yr5j0OLbCEKy6JU3iztXg?pwd=q1h8` 提取码: q1h8
> - 夸克网盘：`https://pan.quark.cn/s/a9c3fa726316`

下载后按以下顺序操作：

1. 安装**兼容版微信**（压缩包内含安装程序），安装完成后**先不要登录**。
2. 安装并运行 **MsgHelper**，等待主界面出现。
3. 在 MsgHelper 内完成初始配置后，再打开微信并登录。

### 2. 安装 Postman

官网免费下载：https://www.postman.com/downloads/

### 3. 确认就绪

| # | 检查项 | 说明 |
|---|---|---|
| 1 | MsgHelper 服务已运行 | MsgHelper 正常启动后会在本地开启 API 服务（默认端口 6003） |
| 2 | 微信 PC 版已登录 | 桌面上能看到微信窗口 |
| 3 | 手中有凭据 | MsgHelper提供的 `tenantId`、`clientId`、`clientSecret`, 测试帐号已经自动加到postman 变量中 |
| 4 | Postman 已安装 | 见上一步 |

---

## 第一步：导入集合

1. 打开 Postman，点击左上角 **Import**。
![导入集合](https://github.com/lileihappy123/msghelper/blob/main/images/1.import.png)
2. 选择文件，找到本目录下的  
   `MsgHelper_OpenAPI_场景化集合.postman_collection.json`，点击打开。
3. 左侧 Collections 面板出现 **MsgHelper OpenAPI 场景化测试集合** 即成功。
![MsgHelper OpenAPI 场景化测试集合](https://github.com/lileihappy123/msghelper/blob/main/images/2.imported.png)

---

## 第二步：填写 4 个变量（必做，不然请求全报错）

1. 在左侧集合名上单击，右侧打开集合详情，切换到 **Variables** 标签页。
2. 找到下列变量，在 **CURRENT VALUE** 列填入你的值：
![填写变量](https://github.com/lileihappy123/msghelper/blob/main/images/3.variables.png)

| 变量名 | 填什么 | 示例 |
|---|---|---|
| `BASE_URL` | 本地服务地址 | `http://127.0.0.1:6003` |
| `TENANT_ID` | 运营方给的租户 ID | `154` |
| `CLIENT_ID` | 运营方给的客户端 ID | `msghelper-client-154` |
| `CLIENT_SECRET` | 运营方给的客户端密钥 | `3334d486910e48e1837b05a46a187189` |

> 其余变量（`ACCESS_TOKEN`、`WX_NAME`、`WX_HWND` 等）**不用手填**，后面的接口运行后会自动写入。

3. 点击右上角 **Save** 保存。

---

## 第三步：按顺序点击发送，共 6 个请求

### 请求 1｜登录，获取 token

**路径**：`1. 调用管理` → `ClientId 登录（client_credentials）`

点击 **Send**，看到右侧响应：

```json
{
  "code": 200,
  "data": { "data": { "accessToken": "abc123..." } }
}
```

![登录成功](https://github.com/lileihappy123/msghelper/blob/main/images/4.login.png)

---

### 请求 2｜获取已登录的微信

**路径**：`1. 调用管理` → `获取登录的微信`

点击 **Send**，响应里能看到你的微信昵称和 hwnd（窗口句柄）。

![获取已登录的微信](https://github.com/lileihappy123/msghelper/blob/main/images/5.get_opened_wx.png)

> ❗ 如果返回空数组，说明微信窗口未被识别。检查微信是否已登录并且窗口可见（不要最小化到托盘）。
>
> ⚠️ **昵称显示为"未知昵称"**：这是 UI 自动化无法读取微信界面文字导致的。按以下步骤修复：
> 1. 退出微信（右键托盘图标 → 退出）
> 2. 打开 **Windows 讲述人**（快捷键 `Win + Ctrl + Enter`，或搜索"讲述人"）
> 3. 重新登录微信，等待主界面完全加载
> 4. 重新发送本请求，昵称应能正常识别
>
> 修复后讲述人可以保持开启，不影响正常使用。

---

### 请求 3｜绑定微信，获取实例 ID

**路径**：`1. 调用管理` → `绑定微信（生成 WX_INSTANCE_ID）`

点击 **Send**。

![绑定微信](https://github.com/lileihappy123/msghelper/blob/main/images/6.bind_wx.png)

✅ 成功标志：响应 `code: 200`，变量 `WX_INSTANCE_ID` 已自动填入。

> ℹ️ 如果返回 400 且 msg 含"已绑定"字样，也没关系，说明之前绑过了，`WX_INSTANCE_ID` 已存在，继续下一步即可。

---

### 请求 4｜手动创建一个好友

**路径**：`4. 好友管理` → `手动加好友（仅系统）`

找一个你想发送消息的微信好友，填入他的微信号（`wxNo`）和昵称（`nickName`），点击 **Send**。

![手动创建好友](https://github.com/lileihappy123/msghelper/blob/main/images/7.add_friend.png)

---

### 请求 5｜查询好友列表，选一个发送目标

**路径**：`4. 好友管理` → `好友查询`

点击 **Send**，响应里能看到你的微信好友列表（最多返回 20 条）。

✅ 变量 `CONTACT_ID`、`CONTACT_NICK_NAME`、`CONTACT_WX_NO`、`CONTACT_CALL_NAME` 自动填入（取的是列表第一条）。

---

### 请求 6｜发出群发消息 🎉

**路径**：`6. 群发消息` → `群发消息-带称呼（文字+图片+文件）`

默认消息内容是：

```
{称呼}您好，这是一条场景化群发测试消息
```

其中 `{称呼}` 会被系统替换为联系人的 `callName`（称呼字段）。

点击 **Send**。

✅ 成功标志：响应 `code: 200`，微信窗口会自动操作，你可以看到消息被发出。

---

## 流程图一览

```
启动服务 & 打开微信
       ↓
  [1] 登录 → 获得 ACCESS_TOKEN
       ↓
  [2] 获取微信 → 获得 WX_NAME / WX_HWND
       ↓
  [3] 绑定微信 → 获得 WX_INSTANCE_ID
       ↓
  [4] 手动创建好友 → 生成 CONTACT_ID 等
       ↓
  [5] 查询好友 → 获得 CONTACT_ID 等
       ↓
  [6] 发送群发 → 消息发出 🎉
```

---

## 常见问题

### Q：发送返回 401 Unauthorized
重新执行请求 1（登录），token 可能过期（默认 2 小时），重新获取后再试。

### Q：请求 2 返回空列表（微信未找到）
- 确认微信主窗口可见，不要缩到系统托盘。

### Q：请求 3 返回 400
正常，说明已绑定过。重新查询一次绑定列表（`1. 调用管理` → `查询绑定微信列表`）来获取 `WX_INSTANCE_ID`，或者直接看 Variables 里是否已有值。

### Q：群发后对方收到的消息 `{称呼}` 没有被替换
联系人的 `callName`（称呼）字段为空，系统会原样输出 `{称呼}`。可先给该联系人设置称呼，或者直接把消息里的 `{称呼}` 删掉。

### Q：想发给多个好友怎么做？
在请求 5 的 Body 里，`selectedContacts` 是数组，可以手动添加多条联系人对象。更推荐的方式是使用 **按标签群发**（`6. 群发消息` → `按标签群发好友`），给目标好友打上同一个标签后批量发送。

---

## 下一步

- 想了解每个字段的详细含义 → 查看 [API_DETAILED_DOCUMENTATION.md](API_DETAILED_DOCUMENTATION.md)
- 完整场景列表（定时任务、加好友、发朋友圈等）→ 直接在 Postman 集合里按分组逐一探索

---

## 技术沟通

![联系微信](https://github.com/lileihappy123/msghelper/blob/main/images/8.wx.png)
