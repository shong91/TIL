# MacOS 적응기 

## nvm install ${특정 node version} failed 


nodejs 를 사용하기 위해 우선 nvm 을 설치하고,
(nvm은 Node.js 여러 버전을 설치해두고 편하게 관리할 수 있게 해주는 도구!)

현 프로젝트에 사용중인 v.10.15.0 을 설치하려고 하니 오류가 발생했다. 

```
$ curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.38.0/install.sh | bash
$ nvm install 10.15.0
```

```
Downloading and installing node v10.15.0...
Computing checksum with shasum -a 256
Provided checksum to compare to is empty.
Checksum check failed!
...
nvm: install v10.15.0 failed!
```

구글링 해보니 M1 chip 을 사용하는 맥북 유저들의 글을 찾아볼 수 있었는데, M1에서 >=v.15 버전 체크를 하는 듯 했다. 

What would be ideal, then, is for nvm to detect that you're on Apple Silicon, and trying to install a version that's not supported (< 15, at least), and fail with a helpful message. I'll leave this open to track that. (https://github.com/nvm-sh/nvm/issues/2350)


version 정보를 변경, 혹은 아래 명령어로 latest version 의 node 를 성공적으로 인스톨 할 수 있었다. 

```
$ nvm install node
```

