# ERROR: docker command not found in Jenkins pipeline 

1. 젠킨스, 도커 모두 로컬에 설치 
2. 젠킨스 플러그인 설치: 도커, 쿠버네티스 관련 플러그인 설치 
3. 파이프라인 스크립트 작성 

에서... checkout 하여 받아온 내용을 우여곡절 끝에 mvn build 까지 마쳤으며.. 이미지를 빌드하려는데 오류가 났다. 


## 1. Using docker plugin syntax: 
```
docker.build(IMAGE_NAME) 
```

**error: docker command not found**

Jenkins 를 도커 컨테이너로 받아왔을 경우, 해당 컨테이너 안에는 docker 가 설치되어 있지 않기 때문에 이와 같은 문제가 발생한다고 하나, 
내 케이스는 도커와 젠킨스 모두 로컬에 설치하였었다. 

### ref) Jenkins 를 도커 컨테이너로 받아왔을 경우에는, 

- Docker in Docker  (DinD)

  도커 컨테이너 안에 도커를 설치하는 방법 (비권장)
  Docker 측에서는 DinD 방식을 권장하지 않고 있다.

  DinD 방식은 도커 컨테이너 안에 도커를 또 설치, 즉 도커 데몬이 2개가 뜨는 것이다. 
  CI 측면에서는 Task를 수행하는 Agent가 Docker Client와 Docker Daemon역할까지 하게되어 도커 명령들을 수행하는데 문제가 없어진다.

  하지만, 이 방식을 사용하기 위해서는 호스트 도커 컨테이너가 privilieged mode로 실행되어야 하는데, 
  `--privileged` 플래그를 사용하면 호스트컨테이너가 호스트머신에서 할 수 있는 거의 모든 작업을 할 수 있게 되어, 보안 상의 큰 문제를 초래할 수 있다. 

- Docker out of Docker (DooD)

  DinD 방식 보다 권장되는 방식이 이 DooD 방식이다. 
  도커 외부의 도커는 호스트의 docker socket을 에이전트 컨테이너에 볼륨 세팅을 통해 공유하고
  호스트의 도커 데몬을 이용해 CI의 도커 명령을 실행한다.

  -v옵션으로 호스트의 docker socket을 빌려사용할 수 있다. 소켓통신을 통해 에이전트 컨테이너는 호스트의 daemon에 docker 명령을 전달한다.

  in macOS: 
  > find . -name "docker.sock"

  > ./.docker/run/docker.sock

  이 방식을 취할 때, docker daemon 은 설치하지 않아도 docker cli 는 설치하여야 한다고 한다.

  (https://stackoverflow.com/questions/55315528/docker-not-found-after-mounting-var-run-docker-sock)

  ```
  $ sudo apt-get install -y docker-ce-cli
  $ docker run -v /var/run/docker.sock:/var/run/docker.sock ...
  ```

## 2. Using a dockerfile with Jenkins Scripted Pipeline Syntax
도커 mvn 이미지를 가져온 파이프라인 문법을 참고하여 (`agent { docker { image 'maven:3.6.3' } }`),

젠킨스 공식문서에서 dockerfile 도 이와 같은 방식으로 읽어와 빌드할 수 있다는 것을 알게 되었다. 

```
stage("Build Image") {
  agent { dockerfile { true } }
  steps {
    sh 'echo Build Image '
  }
}
```

error: exec: "docker": executable file not found in $PATH: unknown

말 그대로 PATH 에 실행 가능한 `docker` 파일을 찾지 못하였다는 것인데... 결국 첫 번째 오류와 같은 결인듯 하다. 

**1. 원인(추정) 1**

 arguments 의 순서가 달라서 생기는 오류로, 아래와 같이 수정해주면 된다는 스택오버플로우의 글이 있었으나, 

(https://stackoverflow.com/questions/27158840/docker-executable-file-not-found-in-path?rq=1)
dockerfile { } 에 별도의 인자 없이 `true` 로 값을 설정하였을 때에도 동일한 오류가 발생하였다. 

```
This was the first result on google when I pasted my error message, and it's because my arguments were out of order.
The container name has to be after all of the arguments.

Bad:
docker run <container_name> -v $(pwd):/src -it
Good:
docker run -v $(pwd):/src -it <container_name>
```

**원인(추정) 2** 

Dockerfile의 WORKDIR 경로가 유효하지 않아 생기는 문제라는 케이스도 있었으나, 
(https://stackoverflow.com/questions/69260507/dockers-executable-file-not-found-in-path-unknown-trying-to-run-cd)

동일한 도커파일로 로컬에서는 정상 빌드 되었는데.. 이 문제도 아닌 것 같다. 



일단... 1이든 2이든 docker 를 찾지 못하는 것이 key 이니, 환경변수 (PATH 설정 등?) 이 해결책이 될 수 있을 듯 하여 좀 더 찾아보는 중. 

1. Jenkins 관리 > Status Information > 시스템 정보 : 시스템 속성 및 환경변수 확인 

환경변수 `PATH` 가 로컬의 `/etc/paths` 와 동일함을 확인. 

여기에 도커 경로가 들어가야 하나 싶어.. `which docker` 로 확인한 경로를 추가한 후 restart > rebuild 해 보았으나 여전히 동일한 오류. 

(애초에 /usr/local/bin 이 추가 되어 있으니 /usr/local/bin/docker 는 의미 없는 행동이지만..)


PATH	/Users/shong/.nvm/versions/node/v17.2.0/bin:/opt/homebrew/bin:/opt/homebrew/sbin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/usr/local/bin/docker


### 해결!
1) 절대경로 사용 

terminal
```
$ which docker
/usr/local/bin/docker
```

jenkins script 
```
$ /usr/local/bin/docker build -t ${IMAGE_NAME} .
```

2) Docker plugin 의 PATH 설정

Global Tool Configuration > Docker > installiation root 에 로컬에 설치한 도커 경로를 입력해주어야 

jenkins 에서 설치한 docker plugin 이 경로를 인식할 수 있게 된다. 

/usr/local/bin/docker 는 바로가기 같은? 경로라 실제 경로로 적어주는 것이 좋다. 

```
$ ls -l /usr/local/bin/docker
lrwxr-xr-x  1 root  wheel  54 12 26 15:04 /usr/local/bin/docker -> /Applications/Docker.app/Contents/Resources/bin/docker
```

docker installation root
```
/Applications/Docker.app/Contents/Resources
```

**반성**
1. 가이드의 기본은 공식문서..
2. 트러블슈팅의 기본은 에러 로그.. 



### Reference
1. 젠킨스 기본 파이프라인 신택스 https://www.jenkins.io/doc/book/pipeline/syntax/
2. DinD, DooD 에 대하여 
https://aidanbae.github.io/code/docker/dinddood/
http://jpetazzo.github.io/2015/09/03/do-not-use-docker-in-docker-for-ci/
