# 네트워크 커맨드

1. 네트워크 특정 포트 떠있나 확인

- 윈도우

```
  netstat -nab |findstr 포트번호
```

- linux

```
  netstat -nap |grep 포트번호
```

- 모니터링

```
  while true; do date ; netstat -nap |grep 포트번호 ; sleep 주기(초) ; done
```

2. 특정 디렉토리 공간사용량 조회

- 윈도우
- linux

```
  du -sh *
  (현재 기준 존재하는 폴더 당 사용량 조회)
```

- 모니터링

```
  while true; date ; do du -sh \* ; sleep 주기(초) ; done
```

3. nslookup: name server 관련한 조회. 서버의 네트워크가 제대로 설정되었는지 확인하는 용도로 주로 사용

```
nslookup google.com
Server:         172.18.208.1
Address:        172.18.208.1#53

Non-authoritative answer:
Name:   google.com
Address: 142.250.199.110
...
```
