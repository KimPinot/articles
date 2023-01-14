---
title: firefox에서 '이 주소는 제한되어 있음' 이라며 포트 연결이 되지 않는 경우
date: 2021-01-21 11:10:00
tags: [troubleshoot, firefox]
categories:
- 개발에 도움이 되는 글
---

[Mozilla Port Blocking](https://developer.mozilla.org/en-US/docs/Mozilla/Mozilla_Port_Blocking)

firefox 주소창에 `about:config` 을 입력하고 설정창으로 들어가서...

`network.security.ports.banned.override` 라는 값에 문자열 `1-65535` 라는 값을 추가하고 껐다 켜면 된다
