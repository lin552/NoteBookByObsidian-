---
创建时间: "2025-07-02 00:13:09"
作者: wangxiaoming
tags:
---
`SSO`（Single Sign-On，单点登录）是一种身份认证机制，允许用户通过**一次登录**访问多个关联系统（称为服务提供商，SP），避免重复输入账号密码。其核心是通过**身份认证中心（`IdP`）​**统一管理用户身份，各SP信任`IdP`的认证结果。以下是`SSO`的通用流程（以最经典的`SAML`协议为例），并补充其他主流协议（如`OAuth2.0/OpenID Connect`）的差异。

#### 一、`SSO`核心组件
- **身份认证中心（`IdP`, Identity Provider）​**​：负责用户身份验证（如企业AD域控、阿里云RAM、Google Identity），颁发认证票据（如`SAML`断言、`JWT`）。
- ​**服务提供商（SP, Service Provider）​**​：需要用户登录的业务系统（如`OA`、邮件、`CRM`），信任`IdP`的认证结果。
- ​**用户（User）​**​：发起登录请求的终端用户。
- ​**票据（Ticket）​**​：`IdP`颁发的临时凭证（如`SAML Assertion`、`OAuth2 Access Token`），用于SP验证用户身份。
#### 二、`SSO`通用流程（以`SAML` 2.0为例）
##### **场景**​：用户首次访问SP（如企业邮箱），未登录状态。
##### **步骤1：用户访问SP，触发未认证响应**​
用户通过浏览器访问SP的应用页面（如`https://mail.company.com`）。SP检查用户是否已登录（通过本地Cookie或Session）。若未登录，SP生成一个**认证请求（`SAML AuthnRequest`）​**，并重定向用户到`IdP`的登录页面。

​**关键操作**​：
- SP构造`SAML AuthnRequest`（包含SP的标识、请求的认证方式等）。
- 通过HTTP重定向（302）将用户导向`IdP`的登录URL（如`https://idp.company.com/saml/login?SAMLRequest=...`）。
##### ​**步骤2：用户在`IdP`完成登录**​
用户到达`IdP`的登录页面（如企业AD域控的登录界面），输入账号密码或其他认证方式（如MFA）。`IdP`验证用户身份：
- ​**验证成功**​：`IdP`生成`SAML`断言（`SAML Assertion`）​**​（包含用户身份信息、票据有效期、签名等），并将用户重定向回SP的回调URL（如`https://mail.company.com/saml/callback?SAMLResponse=...`）。
- ​**验证失败**​：`IdP`返回登录失败页面，流程终止。
##### ​**步骤3：SP验证`SAML`断言，完成登录**​
SP收到`IdP`重定向的请求后，提取`SAML`断言并进行验证：
- ​**签名校验**​：使用`IdP`的公钥验证断言的数字签名（确保断言未被篡改）。
- ​**有效期校验**​：检查断言中的`NotBefore`和`NotOnOrAfter`字段，确保票据未过期。
- ​**用户信息提取**​：从断言中提取用户标识（如`NameID`或`Subject`），建立本地Session（如写入Cookie）。
​**验证通过后**​：SP标记用户为已登录，允许访问受保护资源（如邮箱收件箱）。
##### ​**步骤4：访问其他SP，无需重复登录**​
当用户访问另一个信任同一`IdP`的SP（如企业`CRM`系统`https://crm.company.com`）时：
- `CRM`的SP检查本地Session（无）→ 生成新的`SAML AuthnRequest`，重定向用户到`IdP`。
- `IdP`发现用户已登录（通过`IdP`的Session Cookie），直接生成`SAML`断言并重定向回`CRM`。
- `CRM`验证断言后，建立本地Session，用户无需重复输入密码。
##### ​**步骤5：单点登出（`SLO`, Single Logout）​**​
用户主动登出时，`SSO`需同步终止所有关联SP的Session。流程如下：
- 用户访问`IdP`的登出页面（或SP触发登出请求）。
- `IdP`向所有已登录的SP发送**登出请求（`SAML LogoutRequest`）​**。
- 各SP收到请求后，销毁本地Session，并向`IdP`返回登出响应（`SAML LogoutResponse`）。
- `IdP`销毁自身Session，用户彻底退出所有系统。

#### 三、其他主流协议的`SSO`流程差异
不同`SSO`协议（如`OAuth2.0`、`OpenID Connect`）的流程类似，但票据形式和交互细节不同：
##### 1. `OAuth2.0 + OpenID Connect`**​
- ​**核心票据**​：`OAuth2`的`Access Token`（用于访问资源） + `OpenID Connect`的`ID Token`（包含用户身份信息，`JWT`格式）。
- ​**流程简化**​：
    1. 用户访问SP，SP重定向到`IdP`的授权端点（`/authorize`）。
    2. 用户登录`IdP`并授权后，`IdP`返回`Authorization Code`到SP的回调地址。
    3. SP用`Code`换取`Access Token`和`ID Token`（通过`/token`端点）。
    4. SP验证`ID Token`的签名和用户信息，完成登录。
##### ​2. `CAS`（中央认证服务）​**​
- ​**票据类型**​：`TGT（Ticket Granting Ticket）`和`ST（Service Ticket）`。
- ​**流程特点**​：
    1. 用户首次登录`CAS IdP`，获得`TGT`（存储在Cookie中）。
    2. 访问SP时，SP生成`Service Ticket`并重定向用户到`CAS`验证。
    3. `CAS`验证`Service Ticket`有效性后，返回用户信息给SP，SP建立本地Session。

#### 四、`SSO`流程的关键安全性设计
为防止票据被劫持或伪造，`SSO`流程需满足以下安全要求：
1. ​**票据签名/加密**​：`IdP`对票据（如`SAML`断言、`JWT`）进行数字签名（使用私钥），SP用公钥验证；敏感信息（如用户邮箱）可加密（使用SP的公钥）。
2. ​**票据一次性/短有效期**​：票据仅能使用一次（防止重放攻击），且设置短有效期（如10分钟）。
3. ​`HTTPS`强制传输**​：所有`SSO`交互（重定向、票据传输）必须通过`HTTPS`，防止中间人攻击。
4. ​**Session管理**​：`IdP`和`SP`的本地`Session`需设置合理的过期时间，并支持主动登出（`SLO`）。

#### **总结**​
`SSO`的核心是通过`IdP`统一认证，SP信任`IdP`的票据，实现“一次登录，多处访问”。其流程可概括为：​**用户触发认证→`IdP`验证身份→颁发票据→SP验证票据→建立本地Session**。不同协议（`SAML`、`OAuth2.0`等）的差异主要体现在票据形式和交互细节，但核心逻辑一致。理解`SSO`流程有助于设计安全的身份认证系统，或解决实际开发中的单点登录问题（如票据验证失败、跨域问题）。