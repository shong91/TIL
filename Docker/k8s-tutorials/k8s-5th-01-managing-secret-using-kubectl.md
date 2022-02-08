# kubectl을 사용한 시크릿 관리

https://kubernetes.io/ko/docs/tasks/configmap-secret/managing-secret-using-kubectl/# 제목

## what is secret ?

- 비밀번호, OAuth 토큰, ssh 키 등 민감 정보를 저장하는 용도로 사용
- key:value 형태로 구성하며, 값은 base64 로 인코딩됨
- 종류: 내장 시크릿(built-in), 사용자 시크릿
  - 내장 시크릿: 쿠버네티스 클러스터 내부에서 API에 접근할때 사용. 클러스터 내부에서 사용되는 계정인 `ServiceAccount`를 생성하면 자동으로 관련 시크릿이 만들어지며, 이를 통해 해당 `ServiceAccount`가 권한을 가지고 있는 API에 접근할 수 있다.
  - 사용자 시크릿: 사용자가 만든 시크릿. `kubectl create secret` 명령 또는 yaml 파일로 생성 가능.
- [and more... (공식문서 내용 보기)](https://kubernetes.io/ko/docs/concepts/configuration/secret/)

## Hands-on

### 목표:

- kubectl을 사용하여 시크릿 관리

### 실습

### 1. secret 생성하기

```
echo -n 'admin' > ./username.txt
echo -n '1f2d1e2e67df' > ./password.txt
```

`-n` 플래그는 생성된 파일의 텍스트 끝에 추가 개행 문자가 포함되지 않도록 해 준다.

이는 kubectl이 파일을 읽고 내용을 base64 문자열로 인코딩할 때, 개행 문자도 함께 인코딩될 수 있기 때문에 중요하다.

```
kubectl create secret generic db-user-pass \
  --from-file=./username.txt \
  --from-file=./password.txt
```

앞서 생성한 두 개의 파일로 secret을 생성한다.

별도 파일을 사용하지 않고, 리터럴로 시크릿을 생성할 수도 있다. (`--from-literal=<key>=<value>`)

특수문자가 포함될 경우 싱글 따옴표 ('') 를 사용하여 이스케이프 한다.

```
kubectl create secret generic db-user-pass \
  --from-literal=username=devuser \
  --from-literal=password='S!B\*d$zDsb='
```

### 2. secret 확인

```
kubectl get secrets
kubectl get secrets
```

위 명령어로는 시크릿의 전체 내용이 아닌, 간략한 개요만을 확인할 수 있는데,
kubectl get 및 kubectl describe 명령은 기본적으로 시크릿의 내용을 표시하지 않는다.

이는 시크릿이 실수로 노출되거나 터미널 로그에 저장되는 것을 방지하기 위한 것이다.

때문에, 생성한 시크릿을 보기 위해서는 디코딩을 활용한다.

### 3. secret 디코딩

```
kubectl get secret db-user-pass -o jsonpath='{.data}'
{"password":"MWYyZDFlMmU2N2Rm","username":"YWRtaW4="}
```

```
echo 'MWYyZDFlMmU2N2Rm' | base64 --decode
1f2d1e2e67df
```

## References.
