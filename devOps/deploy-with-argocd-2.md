# 혼자 해보는 CI/CD 구축기 with Argo CD (2) - 프로젝트 생성부터 kubernetes 배포까지

## k8s 리소스 생성

본 작업은 NHN Cloud 를 이용하여 진행되었다.

default 로 생성되는 VPC, Subnet 을 사용하여도 무방하지만, 네트워크를 공부하는 차원에서 처음부터 새로이 생성하였다.

| 순서                                  | 비고                                                                                                           |
| ------------------------------------- | -------------------------------------------------------------------------------------------------------------- |
| 1. Keypair 생성                       | - 생성된 pem 파일 관리에 유의하자.                                                                             |
| 2. VPC 생성                           | - CIDR 를 설정하여 생성한다.                                                                                   |
| 3. Subnet 생성                        | - CIDR 를 설정하여 생성한다.                                                                                   |
| 4. Internet Gateway 생성              | - Internet Gateway를 생성한다.                                                                                 |
| 5. Route Table 생성                   | - 2 에서 생성한 VPC 를 사용하는 라우팅 테이블을 생성한다.                                                      |
|                                       | - 4 에서 생성한 인터넷 게이트웨이를 연결한다. (다른 라우팅 테이블에 연결된 인터넷 게이트웨이는 연결할 수 없다) |
| 6. Floating IP 생성                   | - Floating IP 를 생성한다.                                                                                     |
| 7. 쿠버네티스 클러스터 생성           | - NHN Kubernetes Service(NKS) 를 생성한다. (CentOS 7.9 이미지를 사용, 노드 수는 2개)                           |
| 8. 인스턴스 생성                      | - CI/CD 구축에 사용할 인스턴스를 생성하고 Floating IP 를 연결한다.                                             |
| 9. 보안 그룹 설정                     | - SSH 접속을 위해 22 번 포트를 보안 그룹에 추가한다.                                                           |
| 10. SSH 접속                          | `$ ssh -i keypair.pem centos@<IP_ADDRESS>`                                                                     |
| 11. kubectl 설치 및 연결 확인         |                                                                                                                |
| 1) 공식 docs 를 참고하여 kubectl 설치 | https://docs.nhncloud.com/ko/Container/NKS/ko/user-guide/#_28                                                  |
| 2. kube_config 파일 원격 서버로 이동  | - NHN Kubernetes Service(NKS) 서비스 목록에서 kubeconfig 파일을 다운로드 받을 수 있다.                         |
|                                       | - 다운받은 파일을 로컬 -> 원격지로 이동한다.                                                                   |
|                                       | `$ scp -i keypair.pem kubeconfig.yaml centos@<IP_ADDRESS>:<복사할 경로>`                                       |

## Argo CD 설치

Argo CD 는 Kubernetes 애플리케이션으로 실행되도록 설계되어, kubernetes cluster 내부에 Argo CD 를 설치한다.

### 1. Argo CD 설치

https://argo-cd.readthedocs.io/en/stable/getting_started/

### 2. 포트포워딩 한 IP:Port 로 접속

쿠버네티스 클러스터 노드에서 작업중이라면, 로컬호스트에서 접속할 수 있도록 포트포워딩한다.

나는 별도 인스턴스를 띄워 진행하고 있어, argocd-server 의 external ip 주소를 확인하고 해당 주소로 접속하였다.

```shell
$ kubectl port-forward svc/argocd-server -n argocd 8080:443
Forwarding from 127.0.0.1:8080 -> 8080
Forwarding from [::1]:8080 -> 8080
...

$ k get svc -n argocd argocd-server -o wide
```

## 프로젝트 생성부터 kubernetes pod 배포까지

차근차근 해보기 위해... 아래 3가지 스텝으로 점진적으로 자동화를 구축해 보았다.

1. 타겟 리파지토리 내에서 수동으로 이미지 생성 & 쿠버네티스 배포
2. CI 파이프라인: Continuous Integration (with GitHub Actions)
3. CD 파이프라인: Continuous Deployment (with Argo CD)

### 1. 타겟 리파지토리 내에서 수동으로 이미지 생성 & 배포

**1. 프로젝트 생성**

java 17 / spring boot 3.0.4 버전의 스프링 부트 프로젝트를 생성하였다.

**2. 도커 이미지 생성**

**방법1. Dockerfile 생성**

```shell
# Dockerfile

FROM openjdk:17-alpine
EXPOSE 8080
WORKDIR /data
COPY ./target/*.jar /data/my-app-test.jar
ENTRYPOINT exec java -jar /data/my-app-test.jar
```

프로젝트를 생성한 경로에서 아래 커맨드를 실행, 해당 경로의 Dockerfile 으로 도커 이미지를 생성한다.

```shell
$ docker build -t my-app-test .
```

**방법2. Spring Boot 빌드팩 이용**

```shell
$ ./mvnw spring-boot:build-image
```

도커 이미지 목록을 아래와 같이 확인할 수 있다.

```shell
$ docker images

-----
REPOSITORY                                                TAG                                                                          IMAGE ID       CREATED             SIZE
my-app-test                                               latest                                                                       265fbdedf80a   About an hour ago   418MB
...
```

**4. 도커 컨테이너 실행**

이미지가 정상적으로 만들어졌다면, 도커 컨테이너에 올려 프로젝트를 실행해보자.

```shell
# docker run -d -i -t --name [생성할 컨테이너 name 설정] -p [host port : container port] [image name or ID]
$ docker run -d -p 8080:9090 my-app-test
```

컨테이너 목록을 아래와 같이 확인할 수 있다.

```shell
$ docker ps

-----
CONTAINER ID   IMAGE                     COMMAND                  CREATED             STATUS             PORTS                    NAMES
952645b6ca7d   my-app-test               "/bin/sh -c 'exec ja…"   About an hour ago   Up About an hour   0.0.0.0:9090->8080/tcp   brave_albattani
...
```

Dockerfile 작성 시 `EXPOSE 8080` 으로 8080 포트로 노출되도록 하였으므로 호스트 포트는 8080이며,
9090으로 포트 포워딩 하기 위해 위와 같이 커맨드를 입력하엿다.

`localhost:9090` 으로 접속 시 정상적으로 애플리케이션이 실행되는 것을 확인할 수 있다.

**5. 도커 이미지 배포하기**

5-1. 도커 이미지에 태그 생성

```shell
# docker tag [ image name or Tag ] [ docker hub ID 혹은 private registry ip:port ] / [ push image이름 ]
$ docker tag my-app-test <DOCKERHUB_ID>/my-app-test
```

5-2. 도커 허브에 이미지 배포

사전에 로그인 하지 않았다면, 로그인 후 진행한다.

```shell
$ docker login
Username: <DOCKERHUB_ID>
Password: <DOCKERHUB_PASSWORD>
...
Login Succeeded
```

```shell
# docker push [ tag ]
$ docker push <DOCKERHUB_ID>/my-app-test
```

**6. 배포 대상 프로젝트에 helm chart 생성**

helm chart 를 이용하여 pod, service, deployment, sa 등을 한 번에 생성 및 관리할 수 있다.

```shell
$ helm create my-app-test
```

- Chart.yaml: 차트의 기본적인 정보가 담겨있다.
- values.yaml: Template 파일들과 결합하여 실제 Kubernetes Manifest를 만들게 된다.
- /template/\*\*: Kubernetes Manifest 로 변환할 템플릿 파일들이 위치하여 있다. 이 파일들은 Chart 를 통해 생성될 서비스의 전체 오브젝트에 해당한다.

위 파일들의 설정값을 변경하여 원하는 버전으로 애플리케이션을 배포할 수 있다.

**_[Trouble Shooting#1]_** `ErrImagePull` 오류 발생

- 원인: private docker image 을 받아오기 위한 docker login 정보 미기재
- 해결: 배포된 이미지를 public 으로 변경하거나, 파드 생성 시 docker registry 의 로그인 정보 추가

**_[Trouble Shooting#2]_** Liveness probe failed: Get "http://10.100.2.20:8080/health": dial tcp 10.100.2.20:8080: connect: connection refused

- 원인: initialDelaySeconds 미설정
- 해결: deployment.yaml 의 livenessProbe, readinessProbe 에 initialDelaySeconds 프로퍼티 추가

**_[Trouble Shooting#3]_** Readiness probe failed: HTTP probe failed with statuscode: 401

- 원인: probe path 에 대한 권한 부족
- 해결: health check path 에 대하여 permit all 설정하거나, spring-actuator 설정 추가

**_cf) spring-actuator_**

- 스프링부트의 health check library. 애플리케이션에 대한 모니터링 정보를 엔드포인트를 통해 제공한다.
- db, matrix 등의 상태 정보를 제공하며, actuator/health(기본값) 을 통해 접근하면 애플리케이션의 건강 상태를 제공 받을 수 있다.

```shell
livenessProbe:
  httpGet:
    path: /actuator/health/liveness
    port: 8080
    scheme: HTTP
  initialDelaySeconds: 30
  periodSeconds: 5
  failureThreshold: 3
  successThreshold: 1
```

**8. 실행**

아래 명령어로 chart 에 있는 k8s manifest 파일들을 k8s resource 로 생성한다.

```shell
$ helm install my-app-test
```

도커 이미지 생성부터 쿠버네티스에 배포하는 하나의 사이클을 성공적으로 마쳤다 !

하지만 매번 수동으로 이미지를 생성하고 배포하는 것은 번거로울 뿐 아니라 버전 관리에도 어려움이 있다.
이번 실습의 목표에 맞추어, CI/CD 를 순차적으로 진행해보자.

### 2. Continuous Integration

![](/z.images/cicd-process.png)

Argo CD 는 continuous delivery/deployment 를 담당하는 것이고, continuous integration 을 위한 pipeline 이 별도로 필요하다.

익숙한 Jenkins 를 이용할까 고민하였는데, 많은 기업에서 GitHub Actions 를 사용한 CI/CD 를 채택하고 있다고 하여 이번 기회에 한 번 익혀두는 게 좋겠다 싶어 GitHub Actions 를 사용하기로 결정하였다.
(GitHub - GitHub Issues - GitHub Actions 를 통한 시너지 효과를 기대하는 것도 한 몫 했다.)

**_cf) Jenkins vs GitHub Actions ?_**
| Jenkins | GitHub Actions |
| --------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------ |
| 서버 설치가 필요 | 클라우드에서 동작하므로 어떤 설치도 필요 없음 |
| 작업이 동기적으로 일어나므로, 제품을 시장까지 배포하는 데에 더 많은 시간이 소요됨 | 비동기 CI/CD 가능함 |
| 계정과 트리거에 기반하고 있으며 GitHub 이벤트를 처리할 수 없음 | 모든 GitHub 이벤트에 대해 GitHub Actions를 제공하고 있으며 많은 언어와 프레임워크도 지원함 |
| 환경 호환성을 위해 Docker 이미지에서 동작해야 함 | 모든 환경에 호환됨 |
| 캐싱 기법을 위해 플러그인을 제공하고 있음 | 캐싱이 필요하다면 직접 캐싱 메커니즘을 작성해야 함 |
| 공유할 수 있는 기능을 제공하고 있지 않음 | GitHub 마켓플레이스를 통해 공유할 수 있음 |

https://blog.bitsrc.io/github-actions-or-jenkins-making-the-right-choice-for-you-9ac774684c8

**1. 타겟 리파지토리에 CI integration yaml 파일 생성**

`.github/workflows` 디렉토리를 신규 생성하여, Actions 를 수행할 yaml 파일을 생성한다.

아래는 `main` 브랜치에 `push` 이벤트가 발생하였을 시 Actions 를 수행하도록 하는 스크립트이다.

```shell
name: CI with maven

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v2

      # 배포된 버전에 해당하는 Release / Tag 를 만든다.
      - name: Extract Version Id
        id: extract_version
        run: echo "::set-output name=version::$(cut -c 1-4 <<< '${{ github.sha }}' )"  # commit id 를 잘라 version 명으로 사용

      - name: Create Release
        uses: actions/create-release@v1
        env:
          GITHUB_TOKEN: ${{ secrets.PAT  }}   # public repository 의 경우 ${{ secrets.GITHUB_TOKEN }} 을, private repository 의 경우 personal access token 을 리파지토리 secret 에 등록하여야 한다.
        with:
          tag_name: ${{ steps.extract_version.outputs.version }}
          release_name: ${{ steps.extract_version.outputs.version }}

      # 배포할 프로젝트의 JDK, maven 을 설정한다.
      - name: Set up JDK
        uses: actions/setup-java@v1
        with:
          java-version: 17

      - name: Grant execute permission for maven
        run: chmod +x mvnw

      # maven build & test & create docker image 를 수행
      - name: build with maven & docker image
        run: ./mvnw spring-boot:build-image

      # 생성한 docker image 를 docker registry 에 등록한다. (DockerHub 를 사용)
      - name: Login to Docker Hub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}  # 리파지토리 secret 에 등록하여야 한다.
          password: ${{ secrets.DOCKERHUB_TOKEN }}     # 리파지토리 secret 에 등록하여야 한다.

      - name: Build and push
        run: |
          docker tag sample:0.0.1-SNAPSHOT <DOCERHUB_USERNAME>/sample:${{ steps.extract_version.outputs.version }}
          docker push <DOCERHUB_USERNAME>/sample:${{ steps.extract_version.outputs.version }}

      # kubenetes manifest 파일을 관리하는 리파지토리에 접근한다
      - name: Check out to k8s-manifest repository
        uses: actions/checkout@v2
        with:
          repository: <GITHUB_USERNAME/ORG>/<REPOSITORY_NAME> #  k8s yaml 파일이 있는 repo
          ref: main                                           # branch name
          token: ${{ secrets.PAT }}
          path: <REPOSITORY_NAME>                             # repo 명과 동일

      # 배포할 image tag 를 신규 버전으로 변경하고, 변경 사항을 반영한다
      - name: Update Kubernetes resources
        run: |
          pwd
          cd <REPOSITORY_NAME>/sample
           sed -i 's/<DOCERHUB_USERNAME>.*$/<DOCERHUB_USERNAME>\/sample:${{ steps.extract_version.outputs.version }}/g' ./deployment.yaml
          cat deployment.yaml

      - name: Commit manifest files
        env:
          GITHUB_TOKEN: ${{ secrets.PAT }}
        run: |
          cd <REPOSITORY_NAME>
          git config --global user.email <GITHUB_USER_EMAIL>
          git config --global user.name <GITHUB_USERNAME>
          git config --global github.token ${{ secrets.PAT }}
          git commit -am "Update image tag ${{ steps.extract_version.outputs.version }}"
          git push -u origin main

```

**2. git commit & push**

```shell
$ git add .
$ git commit -m "add .github/workflows/ci.yaml"
$ git push
```

**3. Actions 탭에서 확인**

workflow 에 작성한 job의 step 이 순차적으로 진행되는 모습을 확인할 수 있다.

![](/z.images/github-action-workflow.png)

앞서 살펴본 바와 같이, CI 파이프라인은 [Test -> Build -> Docker image push -> k8s manifest update] 의 순서로 이루어진다.
마지막 스텝인 kubenetes manifest 리파지토리에 push 이벤트가 발생하고, 이를 argocd 에서 잡아내어 자동으로 변경 사항을 deploy 한다.

kubenetes manifest 리파지토리 구성과 CD 파이프라인에 대하여 아래 장에서 계속 이어가도록 하겠다.

### 3. Automated continuous deployment

**[참고]**
https://www.youtube.com/watch?v=MeU5_k9ssrs

1번 실습에서는 배포하고자 하는 타겟 리파지토리에서 Dockerfile 과 helm chart 를 모두 관리하였다.

이 경우 리파지토리에 접근 가능한 개발자라면 누구든 배포에 관한 파일을 수정할 수 있기 때문에, 버전 관리에 위험이 있을 수 있다.
개발자가 통합/배포를 함께 관리하는 조직이라면 이점이 있겠지만, application source 와 deploy config 가 명확히 분리되고, 독립적으로 존재하는 것이 더욱 효율적으로 프로젝트를 관리할 수 있다.

하여, k8s manifest 를 관리하는 리파지토리를 분리하고, Argo CD 를 사용하여 지속적인 배포가 이루어지도록 파이프라인을 구성해보겠다.

**1. kubenetes manifest 리파지토리 생성**

kubenetes manifest 파일을 관리하기 위한 리파지토리를 생성한다.

**2. Application.yaml 작성**

```shell
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: sample-application
  namespace: argocd
spec:
  project: default

  source:
    repoURL: <K8S_MANIFEST_GITHUB_REPO>    #  kubenetes manifest 파일을 관리하는 리파지토리
    targetRevision: HEAD
    path: sample  # 배포하고자 하는 manifest 파일이 위치하는 디렉토리
  destination:
    server: https://kubernetes.default.svc  # default
    namespace: default  # 배포하고자 하는 네임스페이스

  syncPolicy:
    syncOptions:
      - CreateNamespace=true

    automated:
      selfHeal: true
      prune: true
```

타겟 리파지토리가 private repository 일 경우, Credential 을 설정해주어야 한다.

https://argo-cd.readthedocs.io/en/stable/user-guide/private-repositories/

**_[Trouble Shooting#1]_** fata[0000] argo cd server address unspecified

- 원인: 클러스터 노드가 아닌 별도 인스턴스를 띄워 진행하고 있어 argocd-server 의 주소를 찾지 못함
- 해결: argocd-server 가 떠있는 external-ip 를 찾아, 해당 IP 에 위치한 argocd 에 로그인 하는 것으로 커맨드를 수정, 실행

```shell
fata[0000] argo cd server address unspecified
...

$ k get svc -n argocd argocd-server
$ argocd --insecure login <EXTERNAL_IP>:443
Username: admin
Password:
'admin:login' logged in successfully
Context '<EXTERNAL_IP>:443' updated
$ argocd repo add <K8S_MANIFEST_GITHUB_REPO> --username <username> --password <password>
Repository <K8S_MANIFEST_GITHUB_REPO> added
```

**3. git commit & push**

```shell
$ git add .
$ git commit -m "add Application.yml for /sample"
$ git push
```

**4. Argo CD 확인**

위와 같이 진행하였을 시, 최초 commit & push 시 애플리케이션을 자동으로 생성하지 않는 이슈가 있었다. 최초 실행은 트리거가 잡히지 않는 것인지 조금 더 확인이 필요할 듯 하다.

Argo CD 는 UI 가 사용하기 편리하게 되어 있어, web 에서 로그인 한 뒤 Application 을 생성하여 쉽게 사용할 수 있다.
Web 으로 생성하였을 경우 최초에는 `OutOfSync` 상태이므로, Sync 버튼을 클릭하여 동기화 시킬 수 있도록 한다.

혹은, `k apply -f application.yaml` 명령어를 통해서 애플리케이션을 생성할 수 있다.

최초 Sync 를 마치고, 그 이후 변경사항에 대하여 commit & push 하면 자동으로 deploy 가 이루어지는 모습을 확인할 수 있다. (약 3~5분 소요)

![](/z.images/argocd-automated-integration.png)

## 보완해야 할 점은....

1. 빌드 속도 개선: 프로젝트 빌드~도커 이미지 배포까지 1~2분 => 파이프라인 전체 완료까지 약 3분 소요
2. 이미지 태그 관리 개선: v.0.0.1 와 같이 이력을 관리할 수 있도록 태그를 생성하는 것이 더욱 효율적임
