---
layout: post
title: template page
categories: [cate1, cate2]
description: some word here
keywords: keyword1, keyword2
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

# 使用openssl制作CA证书

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
