# k8s helm

- helm 은 k8s package manager 로, 쿠버네티스 리소스를 보다 효율적으로 관리하기 위한 툴 (python 의 pip, nodejs의 npm 과 같은 역할)
- helm chart 는 helm 의 package format 으로, k8s 를 설명하는 파일들을 집합 

helm chart 로 간단한 쿠버네티스 환경 구성 & 배포하는 내용을 정리해 보도록 하겠다. 

## helm 설치 

Homebrew 가 설치되어 있는 mac 환경을 기반으로 했다. 

```
$ brew install helm
```

## helm 생성 

test 이라는 이름으로 helm 환경을 만들었다. 이렇게 생성된 helm 환경은 helm chart 를 담고 있으며, 아래와 같은 기본 디렉토리 구조를 가진다. 

```
$ helm create test
```

```
test/
  Chart.yaml          # helm chart 에 대한 정보 가 담긴 YAML file
  LICENSE             # OPTIONAL: A plain text file containing the license for the chart
  README.md           # OPTIONAL: A human-readable README file
  values.yaml         # helm chart 에서 사용하는 configuration values 에 대한 정의를 하는 YAML 
  values.schema.json  # OPTIONAL: A JSON Schema for imposing a structure on the values.yaml file
  charts/             # 의존하는 chart 에 대한 정보가 담긴 디렉토리
  crds/               # Custom Resource Definitions
  templates/          # k8s를 정의하는 manifest files이 정의된 디렉토리
  templates/NOTES.txt # OPTIONAL: A plain text file containing short usage notes
```

### 1. chart.yaml 파일 수정 
Chart.yaml 파일에는 helm chart 에 대한 간단한 정보들이 담겨있다. 버전정보, 이름, description 등을 수정한다. 
```
apiVersion: v2
name: test
description: A Helm chart for Kubernetes

# A chart can be either an 'application' or a 'library' chart.
#
# Application charts are a collection of templates that can be packaged into versioned archives
# to be deployed.
#
# Library charts provide useful utilities or functions for the chart developer. They're included as
# a dependency of application charts to inject those utilities and functions into the rendering
# pipeline. Library charts do not define any templates and therefore cannot be deployed.
type: application

# This is the chart version. This version number should be incremented each time you make changes
# to the chart and its templates, including the app version.
# Versions are expected to follow Semantic Versioning (https://semver.org/)
version: 0.1.0

# This is the version number of the application being deployed. This version number should be
# incremented each time you make changes to the application. Versions are not expected to
# follow Semantic Versioning. They should reflect the version the application is using.
# It is recommended to use it with quotes.
appVersion: "1.16.0"

```

### 2. values.yaml 파일 수정 
helm 에서 사용되는 각종 value 들을 정의할 수 있다. default 로는 nginx 이미지를 생성하도록 되어 있는데, 해당 이미지 대신 내가 배포할 dockerhub의 이미지를 입력하면 된다. 

```
# Default values for test.
# This is a YAML-formatted file.
# Declare variables to be passed into your templates.

replicaCount: 1

image:
  repository: nginx
  pullPolicy: IfNotPresent
  # Overrides the image tag whose default is the chart appVersion.
  tag: ""

imagePullSecrets: []
nameOverride: ""
fullnameOverride: ""

serviceAccount:
  # Specifies whether a service account should be created
  create: true
  # Annotations to add to the service account
  annotations: {}
  # The name of the service account to use.
  # If not set and create is true, a name is generated using the fullname template
  name: ""

podAnnotations: {}

podSecurityContext: {}
  # fsGroup: 2000

securityContext: {}
  # capabilities:
  #   drop:
  #   - ALL
  # readOnlyRootFilesystem: true
  # runAsNonRoot: true
  # runAsUser: 1000

service:
  type: ClusterIP
  port: 80

ingress:
  enabled: false
  className: ""
  annotations: {}
    # kubernetes.io/ingress.class: nginx
    # kubernetes.io/tls-acme: "true"
  hosts:
    - host: chart-example.local
      paths:
        - path: /
          pathType: ImplementationSpecific
  tls: []
  #  - secretName: chart-example-tls
  #    hosts:
  #      - chart-example.local

resources: {}
  # We usually recommend not to specify default resources and to leave this as a conscious
  # choice for the user. This also increases chances charts run on environments with little
  # resources, such as Minikube. If you do want to specify resources, uncomment the following
  # lines, adjust them as necessary, and remove the curly braces after 'resources:'.
  # limits:
  #   cpu: 100m
  #   memory: 128Mi
  # requests:
  #   cpu: 100m
  #   memory: 128Mi

autoscaling:
  enabled: false
  minReplicas: 1
  maxReplicas: 100
  targetCPUUtilizationPercentage: 80
  # targetMemoryUtilizationPercentage: 80

nodeSelector: {}

tolerations: []

affinity: {}

```

### 3. helm chart 를 적용하여 쿠버네티스 환경 구축 

helm 을 생성한 경로에서, 아래 커맨드를 입력하여 test 환경을 기반으로 한 helm 기반 k8s 를 생성한다. 

```
$ helm install test .
```

### 4. 정상 적용 여부 확인 

helm 이 생성되고, k8s pods 역시 생성된 것을 확인할 수 있다. 

```
$ helm ls 
$ kubectl get pods --all-namespaces
```

배포한 환경에 정상 접속 여부를 확인하기 위해, `kubectl get svc` 명령어를 입력하여 쿠버네티스 클러스터 정보를 확인한다. 

values.yaml 파일에서 설정한 ClusterIP:port(80) 로 떠있는 것을 확인할 수 있다. 

<br>

현재 80번 포트에 떠 있는 서비스를 로컬 환경의 8080 포트로 포트 포워딩하여 nginx 서버에 접속해보도록 한다. 

```
$ kubectl port-forward svc/test 8080:80
```

localhost:8080 으로 접속 시 nginx 서버가 정상적으로 뜨는 것을 확인할 수 있다! 


## helm 삭제

`helm install` 으로 환경을 생성했다면, `helm uninstall` 으로 환경을 삭제할 수 있다. 
```
$ helm uninstall test
```

생성했을 때와 마찬가지로, 아래 명렁어를 통해 정삭적으로 삭제 되었는지 확인할 수 있다. 

```
$ helm ls 
$ kubectl get pods --all-namespaces
```


### References. 
https://lsjsj92.tistory.com/583?category=891065
