# Keycloak Tutorials

# 1. Keycloak 설치 (using Docker)

설치를 위한 여러 방법 중, 도커를 사용한 키클락 설치 방법을 채택했다. 

https://www.keycloak.org/getting-started/getting-started-docker 

keycloak 어드민 유저/패스워드를 설정하여 도커 컨테이너를 실행한다. 

단, Apple M1 칩을 사용하는 경우 `--platform linux/amd64` 옵션을 추가하여준다. (자세한 내용은 https://www.lainyzine.com/ko/article/how-to-install-docker-for-m1-apple-silicon/)

도커 이미지는 `quay.io/keycloak/keycloak`, `jboss/keycloak` 두 가지를 사용할 수 있다. Keycloak 공식문서에는 quay.io 의 이미지를 사용하는 것으로 가이드 하고 있다. 

```
docker run -p 8080:8080 --platform linux/amd64 -e KEYCLOAK_ADMIN=shong91 -e KEYCLOAK_ADMIN_PASSWORD=admin quay.io/keycloak/keycloak:17.0.0 start-dev
```

`--platform linux/amd64` 옵션을 추가하여도 실행이 안되어 좀 더 찾아보니, M1칩 호환 문제를 질문하는 글이 많이 보였다. 

이 경우 오픈소스를 클론 받아 로컬에서 이미지를 직접 빌드하고, 해당 이미지를 사용하면 된다고 한다. 
(https://stackoverflow.com/questions/69492989/getting-error-while-installing-keycloack-in-docker-container-os-mac-m1)
```
git clone git@github.com:keycloak/keycloak-containers.git
cd keycloak-containers/server
git checkout ${VERSION}
docker build -t jboss/keycloak:${VERSION} .
docker run --rm -p 8080:8080 -e KEYCLOAK_USER=admin -e KEYCLOAK_PASSWORD=admin jboss/keycloak:${VERSION}
```

실행을 마쳤으면 `localhost:8080` 으로 접속하여, Keycloak Admin console 으로 들어간다.

# 2. Realm 생성

keycloak realm 은 tenant 와 같은 개념으로, 애플리케이션마다 고립된, 독립된 그룹으로 생성할 수 있다.

1. Keycloak Admin Console 접근
2. 좌측 상단 dropdown > Master > Add realm 클릭

# 3. User 생성

realm 마다 유저를 생성할 수 있다. 

1. Keycloak Admin Console 접근
2. 좌측 상단 dropdown > Master > Add User 클릭

# 4. Client 설정

애플리케이션의 보안을 위해, Client 설정을 등록할 수 있다. 

1. Keycloak Admin Console 접근
2. 좌측 Configure 메뉴 > Clients 클릭
3. key, value 설정 
  - Client ID: 애플리케이션 ID
  - Client Protocol: openid-connect
  - Root URL: 애플리케이션 root url