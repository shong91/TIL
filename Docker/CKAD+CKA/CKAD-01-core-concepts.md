# CKAD exercises

# 1. Core concepts (13%)

자주 쓰이는 플래그 정리

- `dry-run=client`: 서비스에 대한 구성을 생성할 시, 이를 쿠버네티스 API 서버에 전송하지 않음 (none(default)/server/client)
- `-k`: kustomization 파일/디렉토리를 실행한다.
- `-it`: 명령어의 output 을 바로 볼 수 있다. (-it will help in seeing the output or, just run it without -it)
- `-w`: --watch
- `-o wide`: IP, Node 정보 등 해당 리소스의 부가 정보를 보다 넓게 보여줌.
- `kubectl create/run/apply` 가 쿠버네티스 리소스를 생성하기 위함이라면, `kubectl exec` 는 실행중인 리소스(ex 파드) 의 쉘에 접근하기 위한 것.

* nginx 파드를 실행하고, 해당 파드를 별도 yaml 파일로 생성

```
kubectl run nginx --image=nginx --restart=Never --dry-run=client -n mynamespace -o yaml > pod.yaml
```

- busybox 파드를 생성하고, "env" 명령어를 실행하고 그 결과 보기

  `--command` 뒤에 작성한 스크립트가 Command 로 입력되므로, 쿠버네티스 실행을 위한 플래그들을 먼저 적어준 뒤 `--command` 를 실행할 수 있도록 해야한다.

```
kubectl run busybox --image=busybox -it --command -- env
```

```
kubectl run busybox --image=busybox --command -- env -it 라고 명령어 입력 시 pod description
Containers:
  busybox:
    Container ID:  docker://9613b7b6f6c8cfb8ab08c0fff0a0c20b43484a64ae82bcbfc398ff50db592f4b
    Image:         busybox
    Image ID:      docker-pullable://busybox@sha256:afcc7f1ac1b49db317a7196c902e61c6c3c4607d63599ee1a82d702d249a0ccb
    Port:          <none>
    Host Port:     <none>
    Command:
      env
      -it
```

- namespace 를 생성하지는 않고, 새로운 네임스페이스에 대한 yaml 출력

```
kubectl create namespace myns --dry-run=client -o yaml
```

- resourceQuota 생성

```
kubectl create quota myrq --hard=cpu=1,memory=1G,pods=2
```

- nginx 파드의 버전 변경하기

```
kubectl set image pod/nginx nginx=nginx:1.7.1
```

- 특정 파드의 로그 조회

  파드가 crashed and restarted 되었을 경우, 이전 로그를 확인하고 싶다면 `-p` 플래그를 추가한다.

```
kubectl logs nginx -p
```

- 파드 내에서 쉘 실행

```
kubectl exec -it nginx -- /bin/sh
```

- 파드 내에서 쉘 실행 후, 명령어 입력 후 쉘에서 나가기

  쉘에서 나가기 뿐 아니라, 파드도 함께 삭제를 원한다면 `--rm` 플래그를 추가하면 된다.

```
kubectl run busybox --image=busybox -it --restart=Never -- echo 'hello world'
kubectl run busybox --image=busybox -it --restart=Never -- /bin/sh -c 'echo hello world'
```

- env 값을 'var1=val1' 라고 설정하여 파드 생성 후 내용 확인

```
kubectl run nginx --image=nginx --env=var1=val1
# then
kubectl exec -it nginx -- env
# or
kubectl exec -it nginx -- sh -c 'echo $var1'
# or
kubectl describe po nginx | grep val1
# or
kubectl run nginx --restart=Never --image=nginx --env=var1=val1 -it --rm -- env
```
