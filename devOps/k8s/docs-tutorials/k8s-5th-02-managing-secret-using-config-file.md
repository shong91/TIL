# 환경 설정 파일(Config)을 사용하여 시크릿을 관리

https://kubernetes.io/ko/docs/tasks/configmap-secret/managing-secret-using-config-file/

## what is secret-config ?

kubectl 을 사용하여 바로 secret 을 생성할 수 도 있지만, config 파일을 사용하여 보다 편리하게 secret 을 생성, 관리할 수 있다.

시크릿 리소스에는 `data` 와 `stringData` 의 두 가지 맵이 포함되어 있다. data 및 stringData은 영숫자, -, \_ 그리고 .로 구성되어야 한다.

- data: base64로 인코딩된 임의의 데이터 기입
- stringData: 편의를 위해 제공되는 필드. 시크릿 데이터를 인코딩되지 않은 문자열로 기입할 수 있다.

## Hands-on

### 목표:

- 환경 설정 파일을 사용하여 시크릿 관리

### 실습

### 1. config 파일 생성

```
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
data:
  username: YWRtaW4=
  password: MWYyZDFlMmU2N2Rm
```

data 필드의 uername, password 의 인코딩 항목은 각각 `echo -n 'admin' | base64`, `echo -n '1f2d1e2e67df' | base64` 으로 확인하여 인코딩 된 value 를 기입한다.

stringData 필드를 사용 시, base64로 인코딩되지 않은 문자열을 시크릿에 직접 넣을 수 있으며, 시크릿이 생성되거나 업데이트될 때 문자열이 인코딩된다.

실례로, 시크릿을 사용하여 구성 파일을 저장하는 애플리케이션을 배포하면서, 배포 프로세스 중에 해당 구성 파일의 일부를 채우려는 경우를 들 수 있다.

```
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
stringData:
  config.yaml: |
    apiUrl: "https://my.api.com/api/v1"
    username: <user>
    password: <password>
```

### 2. secret 생성

```
kubectl apply -f ./secret.yaml
```

stringData 필드는 쓰기 전용 필드로, 시크릿을 조회할 때 출력되지 않는다. (시크릿이 실수로 노출되거나 터미널 로그에 저장되는 것을 방지하기 위한!)

```
kubectl get secret mysecret -o yaml
```

### data와 stringData 혼용하여 secret 생성

하나의 필드(예: username)가 data와 stringData에 모두 명시되면, stringData에 명시된 값이 사용된다.

```
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
data:
  username: YWRtaW4=
stringData:
  username: administrator
```

위의 시크릿을 생성하여 `k get secret` 으로 내용을 출력하면, 아래와 같이 확인할 수 있다.

```
apiVersion: v1
data:
  username: YWRtaW5pc3RyYXRvcg==
kind: Secret
metadata:
  creationTimestamp: 2018-11-15T20:46:46Z
  name: mysecret
  namespace: default
  resourceVersion: "7579"
  uid: 91460ecb-e917-11e8-98f2-025000000001
type: Opaque
```

값을 디코딩해보면, `stringData` 필드의 값이 사용되었음을 확인할 수 있다.

```
echo 'YWRtaW5pc3RyYXRvcg==' | base64 --decode
administrator
```

## References.

https://kubernetes.io/ko/docs/concepts/storage/volumes/
