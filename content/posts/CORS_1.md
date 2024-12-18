---
title: "CORS跨域资源共享"
date: '2024-12-18T19:38:28+08:00'
# weight: 1
# aliases: ["/first"]
tags: ["Web", "HTTP", "前端"]
author: "Maayurri"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: true
description: "Desc Text."
# canonicalURL: "https://canonical.url/to/page"
disableHLJS: true # to disable highlightjs
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: true
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: true
UseHugoToc: true
# cover:
#     image: "<image path/url>" # image path/url
#     alt: "<alt text>" # alt text
#     caption: "<text>" # display caption under cover
#     relative: false # when using page bundles set this to true
#     hidden: true # only hide on current single page
# editPost:
#     URL: "https://github.com/<path_to_repo>/content"
#     Text: "Suggest Changes" # edit text
#     appendFilePath: true # to append file path to Edit link
---

# 理解CORS：跨域资源共享的全面指南

在现代Web开发中，跨域请求（Cross-Origin Requests）已成为不可避免的一部分。为了确保Web应用的安全性和灵活性，**跨域资源共享（CORS，Cross-Origin Resource Sharing）** 应运而生。本文将深入探讨CORS的概念、工作原理、配置方法以及常见问题，帮助开发者更好地理解和应用这一重要技术。

## 什么是CORS？

**跨域资源共享（CORS）** 是一种基于HTTP头的机制，允许服务器指示浏览器允许哪些来源（域、协议和端口）可以访问其资源。简单来说，CORS解决了浏览器的同源策略限制，允许在不同源之间进行安全的资源共享。

**同源策略** 是浏览器的一个安全机制，限制了从一个源加载的文档或脚本如何与来自不同源的资源进行交互。CORS通过在HTTP头中添加特定字段，放宽了这一限制，从而允许跨域请求。

## 为什么需要CORS？

随着Web应用的复杂化和微服务架构的普及，前端和后端往往部署在不同的域名或端口下。例如：

- 前端应用部署在 `https://www.example.com`
- 后端API服务部署在 `https://api.example.com`

在这种情况下，浏览器默认会阻止前端应用向后端API发送请求，因为它们属于不同的源。CORS提供了一种标准化的方法，让服务器声明哪些来源可以访问其资源，从而促进了前后端的分离和协作。

## CORS的工作原理

CORS通过在HTTP请求和响应中添加特定的头部信息，来控制跨域请求的权限。理解CORS的工作原理，关键在于了解两种类型的请求：**简单请求（Simple Requests）** 和 **预检请求（Preflight Requests）**。

### 简单请求

**简单请求** 是指满足以下条件的请求：

- 使用的HTTP方法是 `GET`、`POST` 或 `HEAD`
- 请求头仅限于以下字段：
  - `Accept`
  - `Accept-Language`
  - `Content-Language`
  - `Content-Type`（值为 `application/x-www-form-urlencoded`、`multipart/form-data` 或 `text/plain`）

对于简单请求，浏览器会直接发送请求，并在响应中检查 `Access-Control-Allow-Origin` 头，以确定是否允许跨域访问。

**示例：**

```
http复制代码GET /data HTTP/1.1
Host: api.example.com
Origin: https://www.example.com
```

响应：

```
http复制代码HTTP/1.1 200 OK
Access-Control-Allow-Origin: https://www.example.com
Content-Type: application/json

{
  "message": "成功获取数据"
}
```

### 预检请求

**预检请求** 是在某些条件下，为了确保实际请求是安全的，浏览器会先发送一个 `OPTIONS` 请求，以询问服务器是否允许实际请求。预检请求适用于：

- 使用了非简单方法（如 `PUT`、`DELETE`）
- 使用了自定义请求头
- `Content-Type` 不在简单类型范围内

预检请求的目的是避免在不被允许的情况下发送复杂请求，从而提升安全性。

**示例：**

```
http复制代码OPTIONS /data HTTP/1.1
Host: api.example.com
Origin: https://www.example.com
Access-Control-Request-Method: DELETE
Access-Control-Request-Headers: X-Custom-Header
```

响应：

```
http复制代码HTTP/1.1 204 No Content
Access-Control-Allow-Origin: https://www.example.com
Access-Control-Allow-Methods: GET, POST, DELETE
Access-Control-Allow-Headers: X-Custom-Header
```

如果服务器在响应中明确允许了实际请求的方法和头，浏览器才会继续发送实际请求。

## 如何配置CORS

CORS的配置主要在服务器端进行，通过设置适当的HTTP头来控制跨域访问权限。不同的服务器框架和平台有不同的配置方式，下面以常见的几种服务器为例介绍配置方法。

### 在服务器端配置CORS

#### 使用Node.js和Express

```
javascript复制代码const express = require('express');
const cors = require('cors');
const app = express();

// 配置CORS
app.use(cors({
  origin: 'https://www.example.com', // 允许的来源
  methods: ['GET', 'POST', 'DELETE'], // 允许的方法
  allowedHeaders: ['Content-Type', 'Authorization'] // 允许的头
}));

app.get('/data', (req, res) => {
  res.json({ message: '成功获取数据' });
});

app.listen(3000, () => {
  console.log('服务器运行在端口3000');
});
```

#### 使用Nginx

在Nginx配置文件中添加以下内容：

```
nginx复制代码server {
    listen 80;
    server_name api.example.com;

    location / {
        if ($request_method = 'OPTIONS') {
            add_header 'Access-Control-Allow-Origin' 'https://www.example.com';
            add_header 'Access-Control-Allow-Methods' 'GET, POST, DELETE';
            add_header 'Access-Control-Allow-Headers' 'Content-Type, Authorization';
            return 204;
        }
        
        add_header 'Access-Control-Allow-Origin' 'https://www.example.com';
        add_header 'Access-Control-Allow-Methods' 'GET, POST, DELETE';
        add_header 'Access-Control-Allow-Headers' 'Content-Type, Authorization';

        proxy_pass http://localhost:3000;
    }
}
```

#### 使用Apache

在Apache的配置文件或 `.htaccess` 文件中添加：

```
apache复制代码<IfModule mod_headers.c>
    Header set Access-Control-Allow-Origin "https://www.example.com"
    Header set Access-Control-Allow-Methods "GET, POST, DELETE"
    Header set Access-Control-Allow-Headers "Content-Type, Authorization"
</IfModule>
```

### 在客户端处理CORS

虽然CORS主要由服务器控制，但客户端也需要正确处理跨域请求。例如，使用 `fetch` API 时，可以设置 `credentials` 选项以包含凭证（如Cookies）：

```
javascript复制代码fetch('https://api.example.com/data', {
  method: 'GET',
  credentials: 'include' // 允许发送凭证
})
.then(response => response.json())
.then(data => console.log(data))
.catch(error => console.error('错误:', error));
```

## CORS的常见问题及解决方案

### 1. **No 'Access-Control-Allow-Origin' Header**

**问题描述：** 在跨域请求时，浏览器控制台报错 `No 'Access-Control-Allow-Origin' header is present on the requested resource.`

**解决方案：** 确保服务器在响应中包含 `Access-Control-Allow-Origin` 头，并设置为允许的来源或 `*`（不推荐用于包含凭证的请求）。

### 2. **CORS预检请求失败**

**问题描述：** 复杂请求的预检请求返回错误，导致实际请求未发送。

**解决方案：**

- 确认服务器正确处理 `OPTIONS` 请求，并返回适当的CORS头。
- 检查 `Access-Control-Allow-Methods` 和 `Access-Control-Allow-Headers` 是否包含实际请求所需的方法和头。

### 3. **凭证请求被阻止**

**问题描述：** 发送带有凭证（如Cookies）的跨域请求时，响应被阻止。

**解决方案：**

- 在客户端请求中设置 `credentials: 'include'`。
- 服务器必须将 `Access-Control-Allow-Origin` 设置为具体的域，而不能使用 `*`。
- 服务器需要设置 `Access-Control-Allow-Credentials: true`。

### 4. **使用通配符导致的安全问题**

**问题描述：** 服务器使用 `Access-Control-Allow-Origin: *` 允许所有来源访问，可能引发安全隐患。

**解决方案：** 尽量指定允许的具体来源，避免使用 `*`，特别是在处理敏感数据或需要凭证的请求时。

## CORS的安全性考虑

尽管CORS提供了灵活的跨域资源访问机制，但不当的配置可能导致安全漏洞。以下是一些安全性建议：

1. **最小化允许的来源**：仅允许信任的域名访问资源，避免使用 `*`。
2. **限制允许的方法和头**：仅允许必要的HTTP方法和自定义头，减少潜在攻击面。
3. **处理预检请求**：确保服务器正确处理预检请求，并在需要时进行额外的验证。
4. **使用HTTPS**：确保所有跨域通信通过安全的HTTPS协议，防止中间人攻击。
5. **监控和日志记录**：监控跨域请求，记录异常行为，及时发现和响应潜在威胁。

## 总结

CORS作为解决跨域请求限制的标准机制，极大地促进了Web应用的灵活性和扩展性。然而，正确理解其工作原理和安全性考量，对于开发者来说至关重要。通过合理配置CORS，既能满足业务需求，又能保障应用的安全性。