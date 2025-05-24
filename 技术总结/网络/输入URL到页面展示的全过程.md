---
创建时间: 2025-04-15 22:08:32
作者: wangxiaoming
tags:
  - 网络
---
#### 一、核心流程（分步骤回答）
##### 1）`URL`解析与请求生成
- 浏览器解析`URL`，提取协议（`HTTP/HTTPS`）、主机名，端口，路径等信息。若用户输入非完整`URL`（如仅输入域名），浏览器自动补全协议（默认`HTTP/HTTPS`）或触发搜索引擎搜索。
- 检查本地缓存（强缓存/协商缓存），若命中则直接加载资源，无需后续步骤
##### 2）`DNS`解析
- 递归查询过程：浏览器缓存->系统hosts文件->本地`DNS`服务器->根`DNS`服务器->顶级域名服务器（如`.com`）->权威`DNS`服务器、最终获得目标IP
- 优化手段：`DNS`预解析（`dns-prefetch`）、`CDN`加速、本地hosts配置
##### 3）建立`TCP`连接
- 三次握手：客户端发送`SYN` -> 服务器返回`SYN-ACK` -> 客户端回复`ACK`,建立可靠传输通道
- `HTTPS`扩展：`TLS`握手（证书验证、密钥协商）加密通话
##### 4）发送HTTP请求
- 请求头包含方法（GET/POST）、协议版本、主机名、User-Agent、Cookie等
- 长连接优化：HTTP/1.1默认 `Connection:keep-alive`复用`TCP`连接，HTTP/2 支持多路复用
##### 5）服务器处理请求
- 静态资源：`Tomact`等服务器通过`DefaultServlet`直接返回文件（如HTML/CSS）
- 动态请求：`Servlet`容器(如`Tomact`)根据`web.xml`映射到对应`Servlet`，调用`service()`方法处理业务逻辑，生成响应数据
- 服务器缓存策略：`CDN`、反向代理缓存、数据库查询缓存
##### 6）浏览器解析与渲染
- 构建`DOM/CSSOM`：解析`HTML`生成`DOM`树，解析`CSS`生成`CSSOM`树，合并为渲染树
- 执行`JavaScript`：阻塞`DOM`解析（除非标记`async/defer`）,可能触发重排（`Reflow`）和重绘（`Repaint`）
- 渲染优化：减少`DOM`操作、使用`requestAnimationFrame`动画，GPU加速。
##### 7）断开连接
- HTTP/1.1默认保持连接复用，HTTP/2支持多路复用减少握手开销
- 四次挥手：客户端/服务端双向确认关闭`TCP`连接

