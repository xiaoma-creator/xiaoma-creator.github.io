---
layout: post
title: openssl 自签发CA证书
categories: [openssl, ca证书]
description:
keywords: openssl, ca证书
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

# 使用openssl制作CA证书

## 自建CA
1. 创建CA证书私钥

```shell
openssl genrsa -aes256 -out ca.key 2048
```

1. 创建请求证书

```shell
openssl req -new -sha256 -key ca.key -out ca.csr -subj "/C=CN/ST=SD/L=JN/O=QDZY/OU=www.test.com/CN=CA/emailAddress=admin@test.com"
```

参数含义：
- C-----国家（Country Name）
- ST----省份（State or Province Name）
- L----城市（Locality Name）
- O----公司（Organization Name）
- OU----部门（Organizational Unit Name）
- CN----产品名（Common Name）
- emailAddress----邮箱（Email Address）

1. 自签署证书

```shell
openssl x509 -req -days 36500 -sha256 -extensions v3_ca -signkey ca.key -in ca.csr -out ca.cer
```
## 生成服务器证书

1. 创建服务器密钥

```shell
openssl genrsa -aes256 -out server.key 2048
```

2. 请求证书

```shell
openssl req -new -sha256 -key server.key -out server.csr -subj "/C=CN/ST=SD/L=JN/O=QDZY/OU=www.test.com/CN=SERVER/emailAddress=admin@test.com"
```

3. 使用CA证书签署服务器证书

```shell
openssl x509 -req -days 36500 -sha256 -extensions v3_req  -CA  ca.cer -CAkey ca.key  -CAserial ca.srl  -CAcreateserial -in server.csr -out server.cer
```

## 生成客户端证书

1. 创建客户端密钥

```shell
openssl genrsa -aes256 -out client.key 2048
```

2. 请求证书

```shell
openssl req -new -sha256 -key client.key  -out client.csr -subj "/C=CN/ST=SD/L=JN/O=QDZY/OU=www.test.com/CN=CLIENT/emailAddress=admin@test.com"
```

3. 使用CA证书签署服务器证书

```shell
openssl x509 -req -days 36500 -sha256 -extensions v3_req  -CA  ca.cer -CAkey ca.key  -CAserial ca.srl  -CAcreateserial -in client.csr -out client.cer
```

## 测试

### 单向认证
服务器
```shell
openssl s_server -CAfile ca.cer -cert server.cer -key server.key -accept 22580
```

客户端
```shell
openssl s_client -CAfile ca.cer -cert client.cer -key client.key -connect 127.0.0.1 -port 22580
```

### 双向认证
服务器
```shell
openssl s_server -CAfile ca.cer -cert server.cer -key server.key -accept 22580 -Verify 1
```

客户端
```shell
openssl s_client -CAfile ca.cer -cert server.cer -key server.key -cert client.cer -key client.key -connect 127.0.0.1 -port 22580
```

# openssl 指令

## 格式转换

### pem转der

```shell
openssl x509 -in cert.crt -outform der -out cert.der
```
### der转pem

```shell
openssl x509 -in cert.crt -inform der -outform pem -out cert.pem
```

## 查看证书

### CA组织名hash值

```shell
openssl x509 -in ca.cer -noout -subject_hash
```

### 查看组织信息

```shell
openssl x509 -in server.cer -noout -subject
```

### 查看pem编码证书内容

```shell
openssl x509 -in server.cer -noout -text
```

### 查看der编码证书内容
```shell
openssl x509 -in ca.cer -inform der  -noout -text
```

### 查看linux根证书信任列表

```shell
awk -v cmd='openssl x509 -noout -subject' ' /BEGIN/{close(cmd)};{print | cmd}' < /etc/ssl/certs/ca-certificates.crt
```

参考链接：https://blog.csdn.net/qq153471503/article/details/109524764