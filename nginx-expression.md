---
title: nginx에서 자주 사용하는 문법
date: 2020-02-26 03:34:00
tags: [nginx]
categories:
- 개발에 도움이 되는 글
---

### 기본적인 라우팅

```text
# Common Routing
server {
    listen 80;
    listen [::]:80;
    server_name _;

    location / {
    	root /home/username/public_html/;
    	index index.html;
    	try_files $uri $uri/ =404;
    }
}
```

### 리액트 라우팅

```text
# React Routing
server {
    listen 80;
    listen [::]:80;
    server_name _;

    location / {
    	root /home/username/public_html/;
    	index index.html;
    	try_files $uri $uri/ /index.html;
    }
}
```

### 리버스 프록시

```text
# Reverse Proxy
server {
    listen 80;
    listen [::]:80;
    server_name _;

    location / {
        proxy_pass http://localhost:3000;
    }
}
```
