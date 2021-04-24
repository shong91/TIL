# WSL 2 설치

​
윈도우에 리눅스를 설치하면 클라우드 작업과 개발 환경 설정이 훨씬 간편해 집니다.
​

## 1. Microsoft Store 에서 Windows Terminal 설치

​

```
- Win+S -> store 검색
- terminal 검색
```

​

## 2. Windows Terminal 관리자 권한으로 실행

​

```
- Win+S -> terminal 검색 -> 마우스 우 클릭 -> 관리자 권한으로 실행
```

​

## 3. WSL 활성화

​

```shell
> dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
> dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```

​

## 4. 윈도우 재부팅

​

## 5. Microsoft Store 에서 Ubuntu 설치

​

## 6. Linux 커널 업데이트 패키지 설치

​

- [Linux 커널 업데이트 패키지 다운로드](https://docs.microsoft.com/ko-kr/windows/wsl/install-win10#step-4---download-the-linux-kernel-update-package)

​

## 7. WSL 2를 기본 버전으로 설정

```shell
> wsl --set-default-version 2
> wsl --set-version Ubuntu 2
```

## 8. Ubuntu 터미널 실행

Windows Terminal 을 실행하고 탭 오른쪽 아래 꺽쇠 클릭
​
​

## 9. Font setup

1. Ctrl + ,
2. Ubuntu 선택
3. 모양 탭
4. 글꼴 : D2Coding
5. 저장

## 참고 자료

​

- [Windows 10에 Linux용 Windows 하위 시스템 설치 가이드](https://docs.microsoft.com/ko-kr/windows/wsl/install-win10)
- [Windows 터미널 설치 및 설정](https://docs.microsoft.com/ko-kr/windows/terminal/get-started)
