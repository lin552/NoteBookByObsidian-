---
创建时间: 2025-04-15 22:31:02
作者: wangxiaoming
tags:
  - HTTP
  - 状态码
  - 缓存头
---
#### 一、HTTP状态码分类与常见状态码
`HTTP`状态码用于表示服务器对请求的处理结果，分为5类，以三位数字的第一位表示类别

| **类别**​   | ​**含义**​  | ​**常见状态码**​                                          |
| --------- | --------- | ---------------------------------------------------- |
| ​**1xx**​ | 信息类（临时响应） | 100（继续发送请求）<br>                                      |
| ​**2xx**​ | 成功类       | 200（请求成功）<br>204（无返回内容）                              |
| ​**3xx**​ | 重定向类      | 301（永久重定向）<br>302（临时重定向）<br>304（资源未修改，使用缓存）          |
| ​**4xx**​ | 客户端错误     | 400（请求语法错误）<br>401（需身份验证）<br>403（禁止访问）<br>404（资源不存在） |
| ​**5xx**​ | 服务器错误     | 500（服务器内部错误）<br>503（服务不可用/过载）<br>504（网关超时）           |
##### 状态码解释：
- 200 OK：请求成功，响应中包含所需资源
- 301/302：资源重定向，301为永久，302为临时
- 304 Not Modified：协商缓存生效，资源为修改，客户端使用本地缓存
- 404 Not Found:：请求资源不存在，需检查URL正确性
- 500 Internal Error：服务器处理请求时发生未知错误

#### 二、HTTP头部内容
`HTTP`头部由客户端和服务端的请求/响应报文组成，根据用于分为四类
##### 1）通用头（General Headers）
**功能**：适用于请求和响应，控制通信的通用行为
**核心字段**：
- `Cache-Control`：缓存策略（如 `max-age=3600`强缓存、`no-cache`协商缓存）
- `Connection`：管理`TCP`连接（如 `keep-alive`复用连接）
- `Date`：报文生成时间（GMT格式）
**场景**：所有`HTTP`通信的基础控制，如连接复用、全局缓存策略

##### 2）请求头（`Request Headers`）
**功能**：客户端向服务器传递请求的附加信息
**核心字段：**
- `Host`：目标域名和端口（HTTP/1.1必需字段）
- `User-Agent`：客户端标识（浏览器/设备类型
- `Authorization`：携带认证凭证（如`Bearer Token`）
- `Accept`：声明可接受的响应类型（如`application/json`）
​**场景**​：API调用、跨域请求、内容协商（如适配不同设备）

##### 3）响应头（Response Headers）
**功能**​：服务器返回响应的元数据及控制指令
**核心字段**​：
- `Server`：服务器软件信息（如`Nginx/1.18`）
- `Set-Cookie`：设置客户端Cookie（含`HttpOnly`、`Secure`属性）
- `Location`：重定向目标URL（配合`3xx`状态码）
​**场景**​：会话管理、重定向、服务器信息传递。

##### 实体头（Entity Headers）
**功能**​：描述消息体（Body）的属性
**核心字段**​：
`Content-Type`：消息体类型（如`text/html; charset=utf-8`）
`Content-Length`：消息体字节长度（用于非分块传输）
`Content-Encoding`：压缩方式（如`gzip`优化传输效率）
​**场景**​：文件上传、API数据传输、资源类型声明。

#### **二、核心场景与使用建议**​
##### 1. ​**缓存控制**​
- ​**强缓存**​：`Cache-Control: max-age=31536000`（静态资源如图片、`CSS/JS`）。
- ​**协商缓存**​：`ETag`+`If-None-Match`（动态资源如用户数据）。
- ​**禁用缓存**​：`Cache-Control: no-store`（敏感数据如支付接口）。
##### 2. ​**安全防护**​
- ​**防`XSS`攻击**​：`Content-Security-Policy: default-src 'self'`
- ​**防点击劫持**​：`X-Frame-Options: DENY`
- ​**强制`HTTPS`**​：`Strict-Transport-Security: max-age=31536000`
##### 3. ​**内容协商与优化**​
- ​**压缩传输**​：`Accept-Encoding: gzip` + `Content-Encoding: gzip`
- ​**跨域支持**​：`Access-Control-Allow-Origin: *`（需配合`CORS`策略）。
##### 4. ​**会话管理**​
- ​**Cookie安全**​：`Set-Cookie: token=abc; HttpOnly; Secure`
- ​**Token传递**​：`Authorization: Bearer <token>`

#### ​**三、注意事项**​

1. ​**字段命名规范**​：
    - 自定义头部需以`X-`前缀开头（如`X-API-Version`）
    - 避免使用保留字段名（如`Host`、`Content-Type`）。
2. ​**性能优化**​：
    - 减少冗余头部（如不必要的`Cookie`）以降低请求体积
    - 分块传输时使用`Transfer-Encoding: chunked`替代`Content-Length`
3. ​**安全配置**​：
    - 敏感字段（如`Authorization`）需通过`HTTPS`传输
    - 定期更新安全策略（如`CSP`白名单）
4. ​**兼容性处理**​：
    - 旧版浏览器可能不支持某些字段（如`Content-Security-Policy`）
    - 移动端需关注`User-Agent`适配
