# Service

## Headless Service
서비스는 접근을 위해 ClusterIP 또는 ExternalIP 를 지정받는다. 

즉, 서비스를 통해 제공되는 기능들에 대한 엔드포인트를 쿠버네티스 서비스를 통해 통제하는 개념인데, MSA 에서는 기능 컴포넌트에 대한 엔드포인트를 찾는 기능을 Service Discovery 라 하고, 서비스의 위치를 등록해놓는 서비스 디스커버리 솔루션을 제공한다. (Etcd, consul 등)

이 경우, 쿠버네티스 서비스를 통해 컴포넌트를 관리하는 것이 아니라, 서비스 디스커버리 솔루션을 사용하기 때문에, 서비스에 대한 IP 주소가 필요없다. 

이러한 시나리오를 지원하기 위한 쿠버네티스의 서비스를 Headless Service 라 한다. Headless Service 생성 시, 클러스터 네트워크 내부에서 서비스에 속한 파드와 직접 접근 가능한 고유 IP와 DNS 가 생성된다. 

- Cluster IP 등의 주소를 가지지 않음 
- DNS 이름을 가지며, DNS를 nslookup 해보면 (서비스(로드밸런서)의 IP가 아닌) 서비스에 연결된 Pod 들의 IP 주소를 리턴한다


## Service Discovery 
- DNS 이름을 사용하여 생성된 서비스의 IP 를 확인할 수 있다. 
- 서비스 생성: DNS=[서비스명].[네임스페이스명].svc.cluster.local
- Pod: FQDN=IP.[네임스페이스명].pod.cluster.local



## References. 
https://bcho.tistory.com/1262
