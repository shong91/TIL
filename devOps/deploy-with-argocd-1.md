# 혼자 해보는 CI/CD 구축기 with argoCD (1) - ArgoCD 란

## What is Argo CD?

"선언적 GitOps 지속적 배포 도구로, Kubernetes를 위한 도구"

_Argo CD is a declarative, GitOps continuous delivery tool for Kubernetes._

GitOps 를 구현하기 위한 도구 중 하나로, Kubernetes 애플리케이션의 지속적인 배포를 위한 오픈소스 도구이다.

## Why Argo CD?

"애플리케이션의 정의, 구성, 환경은 선언적이어야 하며, 버전이 관리되어야 한다."

_Application deployment and lifecycle management should be automated, auditable, and easy to understand._

애플리케이션을 배포할 때에는, 명시적으로 원하는 상태를 정의하고, 원하는 상태를 유지하여야 한다.
기존의 배포 방식은 소프트웨어와 인프라를 분리하여 관리하였기 때문에, 소프트웨어와 인프라 간의 불일치가 발생하고, 버전 관리에 어려움이 있었다.

Argo CD 는 애플리케이션 배포와 수명 주기 관리는 자동화되고, 감사 가능하며, 이해하기 쉬워야한다는 관점에서 등장하였다.
Git Repository 를 이용한 소프트웨어 배포를 통해 버전의 일관성을 유지하고, 모든 코드와 인프라 변경 사항을 Git Repository 에서 관리함으로써 변경 내역을 추적하고 롤백을 쉽게 수행할 수 있다.

| 기존의 배포 방식                      | GitOps 배포 방식                            |
| ------------------------------------- | ------------------------------------------- |
| 인프라 환경을 수동으로 관리           | Git Repository에 소스코드, 인프라 환경 저장 |
| (소프트웨어와 인프라를 분리하여 관리) | (소프트웨어와 인프라를 통합하여 관리)       |
| => 소프트웨어와 인프라 간 불일치 발생 | => VCS(Git) 과 운영 환경 간의 일관성 유지   |

**_cf) Jenkins vs Argo CD ?_**

| 기준              | 젠킨스                                                               | 아르고 CD                                                        |
| ----------------- | -------------------------------------------------------------------- | ---------------------------------------------------------------- |
| 도구 유형         | 범용 자동화 서버                                                     | 쿠버네티스 배포 도구                                             |
| 사용자 인터페이스 | 플러그인으로 사용자 정의 가능한 웹 기반 UI                           | Kubernetes용으로 설계된 간소화된 UI                              |
| 아키텍처          | 일반적으로 전용 서버 또는 가상 머신에 설치                           | Kubernetes 애플리케이션으로 실행되도록 설계됨                    |
| 배포 방식         | 파이프라인 기반, 플러그인으로 사용자 정의 가능                       | 애플리케이션 배포를 위한 선언적 접근 방식                        |
| 통합              | 다른 도구 및 서비스와 통합하기 위해 사용할 수 있는 광범위한 플러그인 | Kubernetes 및 Git과의 Native Integration                         |
| 커뮤니티          | 크고 활발한 오픈소스 커뮤니티                                        | 소규모 커뮤니티가 있는 최신 프로젝트                             |
| 확장성            | 확장 가능하지만 대규모 배포를 위해서는 추가 구성이 필요할 수 있음    | 대규모의 복잡한 Kubernetes 환경에서 작동하도록 설계됨            |
| 보안              | 기본 내장 보안 기능, 플러그인을 통해 추가 보안 제공                  | RBAC 및 Git 기반 인증과 같은 CD 파이프라인 보안을 위한 기능 제공 |
| 사용 사례         | 다양한 자동화 작업에 적합                                            | Kubernetes 배포를 위해 특별히 설계됨                             |

[참고]

- [Jenkins & Argo CD](https://dev.to/ariefwara/jenkins-argo-cd-4ld5)
- [FluxCD, ArgoCD or Jenkins X: Which Is the Right GitOps Tool for You?](https://blog.container-solutions.com/fluxcd-argocd-jenkins-x-gitops-tools)

## Architecture

![](https://argo-cd.readthedocs.io/en/stable/assets/argocd_architecture.png)

1. [유저] Git Repository 에 push
2. [ArgoCD] Git Repository 의 변경 사항 감지 (webhook event)
3. [ArgoCD] kubernetes 배포, 반영

### 구성요소

**1. API Server**

- 웹 UI, CLI 및 CI/CD 시스템에서 소비하는 API를 노출하는 gRPC/REST 서버
- 애플리케이션 관리 및 상태 보고
- 애플리케이션 작업 호출(예: 동기화, 롤백, 사용자 정의 작업)
- 리포지토리 및 클러스터 자격 증명 관리(K8s 시크릿으로 저장)
- 외부 ID 공급자에 대한 인증 및 권한 위임
- RBAC 적용
- Git 웹훅 이벤트를 위한 리스너/포워더

**2. Repository Server**

- 애플리케이션 매니페스트를 보관하는 Git 리포지토리의 로컬 캐시를 유지하는 내부 서비스
- Repository URL, 리비전, 애플리케이션 경로, 특정 템플릿 설정(helm value.yaml)이 제공될 때 쿠버네티스 매니페스트를 생성 및 반환

**3. Application Controller (kubernetes controller)**

- 실행중인 애플리케이션을 지속적으로 모니터링하고, 현재 상태를 원하는 목표 상태와 비교
- 배포 상태(현재 상태)가 목표 상태와 다를 경우, 배포된 애플리케이션이 동기화 되지 않은(OutOfSync) 것으로 간주, 선택적으로 수정 조치 (자동/수동 동기화 가능)
- 수명 주기 이벤트(프리싱크, 동기화, 포스트싱크)에 대한 사용자 정의 hook 을 호출

## 특징

- 지정된 대상 환경에 애플리케이션 자동 배포
- multiple config 관리/템플레이트 도구(Kustomize, Helm, Jsonnet, 일반-YAML) 지원
- multiple cluster 에 관리 및 배포하는 기능
- SSO 통합(OIDC, OAuth2, LDAP, SAML 2.0, GitHub, GitLab, Microsoft, LinkedIn)
- 권한 부여를 위한 multi-tenancy 및 RBAC 정책
- Git 리포지토리에 커밋된 모든 애플리케이션 구성으로 어디서나 Rollback/Roll-anywhere
- 애플리케이션 리소스의 health status 분석
- 자동화된 configuration 드리프트 감지 및 시각화
- 애플리케이션을 원하는 상태로 자동 또는 수동 동기화
- 애플리케이션 활동을 실시간으로 볼 수 있는 웹 UI
- 자동화 및 CI 통합을 위한 CLI
- Webhook 통합(GitHub, BitBucket, GitLab)
- 자동화를 위한 액세스 토큰
- 복잡한 애플리케이션 롤아웃(예: 블루/그린 및 카나리아 업그레이드)을 지원하기 위한 사전 동기화, 동기화, 사후 동기화 후크
- 애플리케이션 이벤트 및 API 호출에 대한 감사 추적
- Prometheus 메트릭
- Git에서 helm 매개변수를 재정의하기 위한 매개변수 재정의

## References.

- https://argo-cd.readthedocs.io/en/stable
- https://wlsdn3004.tistory.com/37
