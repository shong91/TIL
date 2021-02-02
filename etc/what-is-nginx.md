---
layout: post
title: [etc] Nginx 란? 
tags: [nginx]
author: shong91
excerpt_separator: 
---

# Nginx 란

## 개념
- 경량 웹 서버 
- Http web server: 클라이언트의 요청을 받아 정적 응답을 하는 역할
- Reverse Proxy server: WAS 서버의 부하를 줄이는 로드밸런서 역할

## 특징
- 싱글 스레드 기반의 비동기 처리: 새로운 요청이 오더라도, 한 개/고정된 프로세스만 생성, 사용한다. 
- Event-driven 방식
- Thread 생성 비용이 없음 => 적은 자원으로 효율적 운용 가능, 단일 서버로 많은 연결 처리 가능 
- 하나의 Master process 에 여러 개의 Worker process 가 존재하며, 모든 요청은 worker process에서 처리한다. (wocker <-> client)

### cf) vs. Apache
Apache server 는 요청 하나 당 하나의 Thread (멀티 스레드)
-> Thread 생성으로 인한 메모리 및 CPU 낭비 문제 
