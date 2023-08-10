# kubernetes (k8s)

### 쿠버네티스 = 컨테이너 오케스트레이션 도구

- 컨테이너를 쉽고 빠르게 배포/확장하고 관리를 자동화해주는 오픈소스 플랫폼
- 여러 개의 서버에 컨테이너를 배포/운영 가능 (한 번에 하나의 서비스만 배포 가능한 도커 대비 강점)
- MSA, Cloud platform 지향

### WHY k8s?

작은 수의 컨테이너라면 수동으로 VM이나 하드웨어에 직접 배포하면 되지만,
VM이나 하드웨어의 수가 많아지고 컨테이너의 수가 많아지면, 이 컨테이너를 어디에 배포해야 하는지에 대한 결정이 필요하다.

자원을 최대한 최적으로 사용하기 위해서 적절한 위치에 배포해야 하고,
애플리케이션 특성들에 따라서, 같은 물리 서버에 배포가 되어야 하거나 또는 가용성을 위해서 일부러 다른 물리서버에 배포되어야 하는 일이 있다.

쿠버네티스는 이러한 스케쥴링 + 컨테이너에 대한 종합적인 관리를 해주는 컨테이너 운영환경을 제공한다.

### 장점

- 배포 자동화 (Rolling update, Blue-Green, Canary, ..)
- 리소스의 효율적 활용
- Monitoring & Auto Healing
- Auto Scaling & Load Balancing

### Reference

https://subicura.com/2019/05/19/kubernetes-basic-1.html
https://bcho.tistory.com/1255
