# 小程序 AccessToken 获取接口文档

## 一、接口地址

```
GET http://你的域名/api/miniprogram/getAccessToken
```

---

## 二、请求参数

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| appid | string | 是 | 小程序 AppID |
| timestamp | integer | 是 | 时间戳（秒级），必须在当前时间的 ±5 分钟内 |
| nonce | string | 是 | 随机字符串，建议 16-32 位 |
| signature | string | 是 | 签名，见下方算法 |

---

## 三、签名算法

```
signature = MD5(appid + CLIENT_SECRET + timestamp + nonce)
```

**参数拼接顺序：** `appid` → `CLIENT_SECRET` → `timestamp` → `nonce`（直接拼接，无分隔符）

### 重要：CLIENT_SECRET 值

```
CLIENT_SECRET = Xk9#mP2$vL7@nQ4wY8@Zj3!F6&Hs1Rt5
```

> 此密钥由平台统一分配，请勿泄露给外人！

---

## 四、签名示例（PHP）

```php
$appid = 'wx33d894b6aba827f7';
$client_secret = 'Xk9#mP2$vL7@nQ4wY8@Zj3!F6&Hs1Rt5';
$timestamp = time();
$nonce = bin2hex(random_bytes(16));  // 生成32位随机字符串
$signature = md5($appid . $client_secret . $timestamp . $nonce);

// 拼接完整请求URL
$url = "http://你的域名/api/miniprogram/getAccessToken?appid={$appid}&timestamp={$timestamp}&nonce={$nonce}&signature={$signature}";
```

---

## 五、签名示例（Python）

```python
import hashlib
import time
import secrets

appid = 'wx33d894b6aba827f7'
client_secret = 'Xk9#mP2$vL7@nQ4wY8@Zj3!F6&Hs1Rt5'
timestamp = int(time.time())
nonce = secrets.token_hex(16)

sign_str = appid + client_secret + str(timestamp) + nonce
signature = hashlib.md5(sign_str.encode()).hexdigest()

url = f"http://你的域名/api/miniprogram/getAccessToken?appid={appid}&timestamp={timestamp}&nonce={nonce}&signature={signature}"
```

---

## 六、签名示例（JavaScript/Node.js）

```javascript
const crypto = require('crypto');

const appid = 'wx33d894b6aba827f7';
const client_secret = 'Xk9#mP2$vL7@nQ4wY8@Zj3!F6&Hs1Rt5';
const timestamp = Math.floor(Date.now() / 1000);
const nonce = crypto.randomBytes(16).toString('hex');

const signature = crypto.createHash('md5')
    .update(appid + client_secret + timestamp + nonce)
    .digest('hex');

const url = `http://你的域名/api/miniprogram/getAccessToken?appid=${appid}&timestamp=${timestamp}&nonce=${nonce}&signature=${signature}`;
```

---

## 七、响应格式

### 成功响应

```json
{
    "status": 200,
    "msg": "获取成功",
    "data": {
        "access_token": "57_xxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
        "expires_in": 7200
    }
}
```

### 失败响应

```json
{
    "status": 400,
    "msg": "签名验证失败",
    "data": null
}
```

---

## 八、错误码说明

| status | msg | 解决方法 |
|--------|-----|----------|
| 400 | 参数不完整 | 检查是否缺少 appid、timestamp、nonce、signature |
| 400 | 小程序配置不存在 | 确认 appid 正确且已在平台配置 |
| 400 | 小程序Secret未填写 | 联系平台管理员配置小程序Secret |
| 400 | 请求已过期，timestamp必须在5分钟内 | 重新生成 timestamp，确保在 ±5 分钟内 |
| 400 | 签名验证失败 | 检查签名算法是否正确 |
| 500 | 获取AccessToken失败 | 检查网络或联系平台技术支持 |

---

## 九、注意事项

### 1. 时间戳有效期
- `timestamp` 必须与服务器时间误差在 **5 分钟以内**
- 建议每次请求前重新生成，不要复用旧的 timestamp

### 2. 签名安全
- `CLIENT_SECRET` 是平台级密钥，请妥善保管，**不要硬编码在前端代码中**
- 签名计算必须在后端完成，避免密钥泄露

### 3. AccessToken 缓存
- 接口会缓存 AccessToken，有效期约 2 小时
- 建议自行缓存返回的 `access_token`，避免频繁调用

### 4. 频率限制
- 每个 appid 每小时建议不超过 **200 次** 调用
- 超出限制可能导致接口被临时封禁

### 5. 特殊字符转义
- 如果使用 Java/Go 等语言，`CLIENT_SECRET` 中的特殊字符（`#`、`$`、`@`、`!`、`&`）无需额外转义，直接拼接字符串即可

---

## 十、快速测试

在浏览器地址栏直接访问（注意替换域名）：

```
http://你的域名/api/miniprogram/getAccessToken?appid=wx33d894b6aba827f7&timestamp=1745726400&nonce=test123456789012345678901234567890&signature=计算你的签名
```

> 由于签名需要实时计算，建议使用上面的示例代码生成真实有效的签名进行测试。
