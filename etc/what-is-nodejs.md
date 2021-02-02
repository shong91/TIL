---
layout: post
title: [etc] NodeJS 란? 
tags: [NodeJS]
author: shong91
excerpt_separator: 
---

# NodeJS 란

## 개념
- Chrome V8 자바스크립트 엔진으로 빌드 된 javascript 런타임
- 서버에서 javascript 를 동작할 수 있도록 하는 환경. 즉, 서버를 위해 설계된 플랫폼 (**NOT SERVER!**)
- 서버가 필요할 시 직접 구현해 주어야 함. (http 모듈을 import하여 서버 생성)

```
var http = require(‘http’);

//create a server object:
http.createServer(function (req, res) {
    res.write(‘Hello World!’); //write a response to the client
    res.end(); //end the response
}).listen(8080); //the server object listens on port 8080
```

## 특징
- 싱글 스레드 기반의 비동기 처리: 서버가 멈추지 않고 반응할 수 있음 => 확장성
- Event-loop 방식
- SPA, 데이터의 실시간 반영이 필요한 app 에 적합
- 하나의 작업 자체가 오랜 시간이 걸리는 경우에는 부적합함 (전체 시스템 저하 야기)
