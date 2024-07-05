# 쿠버네티스 소개 및 아키텍쳐

**용어 및 발음 정리**

- master
- node
- k8s (쿠버네티스)
- kubectl (큐브 컨트롤)
- etcd (엣지디)
- pod
- istio
- helm (헬름)
- knative (케이 네이티브)

**Kubernetes**

- 컨테이너화된 애플리케이션을 자동으로 배포하고 스케일링하고 관리한다.
- 컨테이너를 쉽게 관리하고 연결하기 위해 논리적인 그룹 단위로 그룹화한다.
- Google에서 15년 간의 경험을 토대로 최상의 아이디어와 방법들을 결합하였다.

**Cloud Native**

- 클라우드 이전에는 리소스를 한 땀 한 땀 직접 관리하였다.
- 클라우드를 사용하기 시작한 후에는 수많은 리소스를 자유롭게 사용하고 추상적으로 관리할 수 있게 되었다.
- 이전에는 웹 서버, API 서버, DB 서버라고 했으면, 이제 서버 0번, 1번, 2번 이런 식으로 부르게 되고, 웹이 몇 번인지는 중요하지 않게 되었다.
    - 어딘가에만 웹이 떠 있으면 되는 환경이다.

**Kubernetes Architecture**

- **Desired State**
    - 현재 상태 (Current State) == 원하는 상태 (Desired State)
        - 처음에 현재 상태가 원하는 상태가 맞는지 상태 체크를 한다.
    - 현재 상태 (Current State) != 원하는 상태 (Desired State)
        - 차이점이 발견된다.
    - 현재 상태 (Current State) → 원하는 상태 (Desired State)
        - 현재 상태를 원하는 상태로 조치한다.
    - 결국 상태를 체크하고 차이점을 발견하고 차이점이 생기면 조치를 하는 루프를 계속해서 돌게 된다. 이 단순한 흐름만 잘 유지하면 서버 운영할 때 문제가 없어진다.
    - Desired State는 종류가 많아질 수 있으며, 다음 예시와 같다.
        - Replication Controller
            - 복제 셋을 체크하는 컨트롤러이며, 복제가 잘 됐는지만 체크를 하는 컨트롤러이다.
        - Endpoint Controller
            - 서비스, 로드 밸런싱을 관리하는 컨트롤러이며, 그것만 체크를 하는 컨트롤러이다.
        - Namespace Controller
        - Custom Controller
        - ML Controller
        - CI/CD Controller
- **Architecture**
    - Master
        - API Server
            - Scheduler, Controller들을 중간에서 제어해주는 영역이다.
        - etcd
            - 상태를 저장하고 조회하는 데이터베이스이다.
    - Node
        - 실제로 컨테이너가 실행되는 부분이다.

**Master**

- 체크하고 실행하는 영역이다.
- Master 상세 - **etcd**
    - 모든 상태와 데이터를 확실하게 관리하고 저장한다.
    - 분산 시스템으로 구성하여 안전성을 높인다. (고가용성)
    - 가볍고 빠르면서 정확하게 설계한다. (일관성)
    - Key(Directory)-Value 형태로 데이터를 저장한다.
    - TTL(Time To Live), Watch 같은 부가 기능을 제공한다.
    - 백업은 필수이다.
- Master 상세 - **API Server**
    - 상태를 바꾸거나 조회를 하는 역할을 한다.
    - etcd와 유일하게 통신하는 모듈이다.
    - RESTful API 형태로 제공된다.
    - 권한을 체크하여 적절한 권한이 없을 경우 요청을 차단한다.
    - 관리자 요청 뿐만 아니라 다양한 내부 모듈과 통신한다.
    - 수평으로 확장되도록 디자인되었다.
- Master 상세 - **Scheduler**
    - 새로 생성된 Pod을 감지하고 실행할 노드를 선택한다.
    - 노드의 현재 상태와 Pod의 요구사항을 체크한다.
        - 노드에 라벨을 부여한다.
        - Ex. a-zone, b-zone 또는 gpu-enabled 등
- Master 상세 - **Controller**
    - 논리적으로 다양한 컨트롤러가 존재한다.
        - 복제 컨트롤러, 노드 컨트롤러, 엔드포인트 컨트롤러 등
    - 끊임 없이 상태를 체크하고 원하는 상태를 유지한다.
    - 복잡성을 낮추기 위해 하나의 프로세스로 실행한다.
- Master 상세 - **조회 흐름**
    - Controller가 API Server에 현재 상태 정보를 조회한다.
    - API Server는 Controller가 리소스를 볼 수 있는지 권한 체크 후 권한이 있다고 판단이 되면 etcd에 정보 조회를 하여 알려주게 된다.
    - etcd는 원하는 상태가 저장이 되어 있으면 원하는 상태 변경 여부를 Controller한테 알려주게 된다.
    - Controller는 현재 상태와 원하는 상태가 맞지 않기 때문에 조치하여 리소스 변경을 하고 그 리소스 변경된 결과를 API Server에 전달해준다.
    - API Server는 정보 갱신 권한 체크 후 권한이 있다고 판단이 되면 etcd에 정보를 갱신한다.

**Node**

- Node에는 크게 두 개의 컴포넌트가 떠 있는데, 하나는 Proxy, 하나는 Kubelet이다.
- Proxy와 Kubelet도 Master와 통신을 할 때 API 서버만 바라보게 된다.
- Proxy는 iptables이나 IPVS 방식을 사용하고, Kubelet이 Pod과 직접 통신을 하게 된다.
- Node 상세 - **Kubelet**
    - 각 노드에서 실행된다.
    - Pod을 실행/중지하고 상태를 체크한다.
    - CRI (Container Runtime Interface)
        - Docker, Containerd, CRI-O 등
- Node 상세 - **Proxy**
    - 네트워크 프록시와 부하 분산 역할을 한다.
    - 성능상의 이유로 별도의 프록시 프로그램 대신 iptables 또는 IPVS를 사용한다. (설정만 관리)

**Pod이 생성되는 과정**

- 관리자가 API Server에 Pod을 하나 추가해달라는 요청을 한다.
- API Server는 etcd에 Pod 생성 요청 정보를 넣는다.
- Controller는 API Server를 통해 새로 생긴 Pod이 있는지 계속 체크를 하게 된다.
    - 이 때, Pod 생성 요청을 발견하게 되면 API Server에 실제 Pod을 할당하는 요청을 다시 하게 된다.
- API Server는 Pod 정보를 할당 요청으로 etcd에 다시 넣게 된다.
- Scheduler는 계속 할당 요청이 들어온 Pod이 있는지 체크를 하며, 특정 Node에 Pod을 할당하게 된다.
- API Server는 이 정보를 받아서 이 Pod을 특정 Node에 할당하는데 상태는 아직 실행되기 전 상태로 etcd로 보내 정보를 업데이트하게 된다.
- Kubelet은 Node에 할당된 Pod 중에서 아직 실행이 안된 Pod이 있는지 계속 체크를 한다.
    - 미실행 Pod이 확인 되면, 그 정보로 Pod을 생성하고 그 정보를 다시 API Server가 업데이트를 하게 된다.
- API Server는 이 정보를 받아서 이 Pod을 특정 Node에 할당하였으며, 실행 중 상태로 etcd로 보내 정보를 최종 업데이트하게 된다.
- 이 상태에서도 Scheduler, Controller, Kubelet은 계속 상태 체크를 하게 된다.

**Addons**

- CNI (네트워크)
- DNS (도메인, 서비스 디스커버리)
- 대시보드 (시각화)

**Kubernetes Objects**

- **Pod**
    - Kubernetes는 컨테이너를 직접 관리하지 않고 Pod으로 감싸 관리를 한다. 컨테이너를 배포하는 것이 아닌 Pod을 배포하는 것이다.
    - Pod은 가장 작은 배포 단위이다.
    - 각 Pod은 전체 클러스터에서 고유한 IP를 할당 받는다.
    - 여러 개의 컨테이너가 하나의 Pod에 존재할 수 있다.
- **ReplicaSet**
    - 여러 개의 Pod을 관리한다. 몇 개의 Pod을 관리할 지를 결정하게 된다.
- **Deployment**
    - ReplicaSet을 감싸며 배포 버전을 관리하는 오브젝트이다.
    - 무중단 배포를 진행할 때, Deployment의 version = 1 → version = 2로 배포될 때 ReplicaSet의 replicas = 3이라 가정하면 version = 1의 replicas = 1 / version = 2의 replicas = 1로 자동으로 분리 된다. 배포가 정상적으로 이루어졌을 때 version과 replicas가 변경되면서 자연스럽게 업데이트가 된다.
- Service - **ClusterIP**
    - 클러스터 내부에서 사용하는 프록시이다.
    - Pod을 로드밸런스하는 별도의 서비스이다.
    - 클러스터 내부에서 서비스 연결은 DNS를 이용한다.
- Service - **NodePort**
    - 노드(Host)에 노출되어 외부에서 접근 가능한 서비스이다.
- Service - **LoadBalancer**
    - 하나의 IP 주소를 외부에 노출해준다.
- 즉, 사용자(Web Browser)는 Load Balancer로 요청을 보내고 그 요청이 다시 Node Port로 가고 그 Node Port가 다시 Cluster IP로 가고 그 Cluster IP가 다시 Pod으로 전달되는 구조이다.
- **Ingress**
    - 도메인 이름, 도메인 뒤에 붙는 경로별로 내부에 있는 Cluster IP 연결에 대해 라우팅을 해준다.
        - Nginx, HAProxy, ALB 등
- ETC
    - Volume
        - Storage (EBS, NFS 등)
    - Namespace
        - 논리적인 리소스 구분
    - ConfigMap/Secret
        - 설정
    - ServiceAccount
        - 권한 계정
    - Role/ClusterRole
        - 권한 설정 (Get, List, Watch, Create 등)

**Kubernetes API 호출**

- Object Spec - **YAML**
- Pod의 YAML 예시
    
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
        name: example
    spec:
        containers:
        - name: exampleContainer
            image: exampleContainer:1.25
    ```
    
- ReplicaSet의 YAML 예시
    
    ```yaml
    apiVersion: apps/v1
    kind: ReplicaSet
    metadata:
        name: backend
    spec:
        replicas: 3
        selector:
            matchLabels:
                app: backend
        template:
            metadata:
                labels:
                    app: backend
            spec:
                containers:
                - name: web
                    image: image:v1
    ```
    
- ArgoCD(Custom Resource)의 YAML 예시
    
    ```yaml
    apiVersion: argoproj.io/v1alpha1
    kind: Application
    metadata:
        name: guestbook
        namespace: argocd
    spec:
        project: default
        source:
            repoURL: https://github.com/argoproj/argocd-example-apps.git
            targetRevision: HEAD
            path: guestbook
        destination:
            server: https://kubernetes.default.svc
        namespace: guestbook
    ```
    
- **Object Spec**
    - apiVersion
        - apps/v1, v1, batch/v1, networking.k8s.io/v1 등
    - kind
        - Pod, Deployment, Service, Ingress 등
    - metadata
        - name, label, namespace 등
    - spec
        - 각종 설정 (https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.18)
    - status (read-only)
        - 시스템에서 관리하는 최신 상태
- API 호출하기를 정리하자면, 원하는 상태(Desired State)를 다양한 오브젝트(Object)로 정의(Spec)하고 API 서버에 YAML 형식으로 전달한다.
