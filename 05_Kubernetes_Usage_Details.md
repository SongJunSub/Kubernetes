# 쿠버네티스 사용 상세

---

**Pod**

- Pod은 쿠버네티스에서 관리하는 가장 작은 배포 단위이다.
- 쿠버네티스와 도커의 차이점은 도커는 컨테이너를 만들지만, 쿠버네티스는 컨테이너 대신 Pod을 만든다. Pod은 한 개 또는 여러 개의 컨테이너를 포함한다.
- Pod 생성하기

    ```bash
    kubectl run echo --image ghcr.io/subicura/echo:v1
    ```

  > **주의**
  Kubernetes v1.18 이상은 `run` 명령어가 Pod을 만들지만 v1.17 이하는 Deployment를 만든다.
  >

    ```bash
    # Pod 목록 조회
    kubectl get pod
    
    # 단일 Pod 상세 확인
    kubectl describe pod/echo
    ```

- Pod 생성 분석
    - minikube 클러스터 안에 Pod이 있고 Pod 안에 컨테이너가 있다.
    - Pod 생성 과정
        1. `kubectl run` 명령어를 통해 API Server에 Pod을 생성한다.
        2. Scheduler는 API Server를 감시하면서 할당되지 않은 Pod이 있는지 체크한다.
        3. Node에 설치된 Kubelet은 자신의 Node에 할당된 Pod이 있는지 체크한다.
        4. Kubelet은 Scheduler에 의해 자신에게 할당된 Pod의 정볼르 확인하고 컨테이너를 생성한다.
        5. Kueblet은 자신에게 할당된 Pod의 상태를 API Server에 전달한다.
- Pod 제거하기

    ```bash
    kubectl delete pod/echo
    ```

- 실제로 kubectl run 명령어는 거의 사용하지 않는다.

**Pod 생성 분석**

- minikube 클러스터 안에 Pod이 있고 Pod 안에 컨테이너가 있다.
- Pod 생성 과정
    1. `kubectl run` 명령어를 통해 API Server에 Pod을 생성한다.
    2. Scheduler는 API Server를 감시하면서 할당되지 않은 Pod이 있는지 체크한다.
    3. Node에 설치된 Kubelet은 자신의 Node에 할당된 Pod이 있는지 체크한다.
    4. Kubelet은 Scheduler에 의해 자신에게 할당된 Pod의 정볼르 확인하고 컨테이너를 생성한다.
    5. Kueblet은 자신에게 할당된 Pod의 상태를 API Server에 전달한다.

**YAML로 설정 파일 작성하기**

- 원하는 리소스를 YAML 파일로 작성하면 복잡한 내용을 표현하기 좋고 변경된 내용을 버전 관리할 수 있다.
- Pod을 YAML 파일로 정의하기
- echo-pod.yml

    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: echo
      labels:
        app: echo
    spec:
      containers:
        - name: app
          image: ghcr.io/subicura/echo:v1
    ```

    - `run` 명령어로 생성할 때와 차이점은 `label`이 추가되었다. 쿠버네티스는 리소스를 관리할 때 `name`과 `label`을 이용한다.
- 필수 요소


    | 정의 | 설명 | 예 |
    | --- | --- | --- |
    | version | 오브젝트 버전 | v1, app/v1, http://networking.k8s.io/v1 등 |
    | kind | 종류 | Pod, ReplicaSet, Deployment, Service 등 |
    | metadata | 메타 데이터 | name과 label, annotation(주석)으로 구성 |
    | spec | 상세 명세 | 리소스 종류마다 다르다. |
- Pod 생성, 로그 확인, 제거

    ```bash
    # Pod 생성
    kubectl apply -f echo-pod.yml
    
    # Pod 목록 조회
    kubectl get pod
    
    # Pod 로그 확인
    kubectl logs echo
    kubectl logs -f echo
    
    # Pod 컨테이너 접속
    kubectl exec -it echo -- sh
    # ls
    # ps
    # exit
    
    # Pod 제거
    kubectl delete -f echo-pod.yml
    ```


**컨테이너 상태 모니터링**

- 컨테이너 생성과 실제 서비스 준비는 약간의 차이가 있다. 서버를 실행하면 바로 접속할 수 없고, 짧게는 수초, 길게는 수분의 초기화 시간이 필요한데 실제로 접속이 가능할 때 ‘서비스가 준비되었다’고 말할 수 있다.
- 쿠버네티스는 컨테이너가 생성되고 서비스가 준비되었다는 것을 체크하는 옵션을 제공하여 초기화하는 동안 서비스되는 것을 막을 수 있다.

**livenessProbe**

- 컨테이너가 정상적으로 동작하는지 체크하고 정상적으로 동작하지 않는다면 컨테이너를 재시작하여 문제를 해결한다.
- 정상이라는 것은 여러가지 방식으로 체크할 수 있는데, 여기서는 http get 요청을 보내 확인하는 방법을 사용한다.
- echo-lp.yml

    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: echo-lp
      labels:
        app: echo
    spec:
      containers:
        - name: app
          image: ghcr.io/subicura/echo:v1
          livenessProbe:
            httpGet:
              path: /not/exist
              port: 8080
            initialDelaySeconds: 5
            timeoutSeconds: 2 # Default 1
            periodSeconds: 5 # Default 10
            failureThreshold: 1 # Default 3
    ```

    - `initialDelaySeconds: 5`
        - 처음 5초 후에 HTTP GET 요청을 보낸다.
    - `periodSeconds: 5`
        - 5초마다 한 번씩 체크를 진행한다.
    - `failureThreshold: 1`
        - 한 번이라도 실패하면 컨테이너를 재시작한다.
- Pod 생성

    ```bash
    kubectl apply -f echo-lp.yml
    ```

- 실행 결과

    ```bash
    # Pod 목록 조회
    kubectl get pod
    
    # 결과
    NAME      READY   STATUS             RESTARTS      AGE
    echo-lp   0/1     CrashLoopBackOff   5 (16s ago)   107s
    ```

    - 정상적으로 응답하지 않았기 때문에 Pod이 여러 번 재시작되고 CrashLoopBackOff 상태로 변경되었다.

**readnessProbe**

- 컨테이너가 준비되었는지 체크하고 정상적으로 준비되지 않았다면 Pod으로 들어오는 요청을 제외한다.
- livenessProbe와 차이점은 문제가 있어도 Pod을 재시작하지 않고 요청만 제외한다는 점이다.
- echo-rp.yml

    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: echo-rp
      labels:
        app: echo
    spec:
      containers:
        - name: app
          image: ghcr.io/subicura/echo:v1
          readinessProbe:
            httpGet:
              port: 8080
              path: /not/exist
            initialDelaySeconds: 5
            timeoutSeconds: 2 # Default 1
            periodSeconds: 5 # Default 10
            failureThreshold: 1 # Default 3
    ```

- Pod 생성

    ```bash
    kubectl apply -f echo-rp.yml
    ```

- 실행 결과

    ```bash
    # Pod 목록 조회
    kubectl get pod
    
    # 결과
    NAME      READY   STATUS             RESTARTS        AGE
    echo-rp   0/1     Running            0               2m24s
    ```

    - 재시작하지는 않고 내부적으로 서비스 준비가 안되었으므로 접속을 막는 준비 상태로 체크하는 것으로 끝나게 된다.

**livenessProbe + readnessProbe**

- 보통 livenessProbe와 readnessProbe를 같이 적용한다. 상세한 설정은 애플리케이션 환경에 따라 적절하게 조정한다.
- echo-pod-health.yml

    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: echo-health
      labels:
        app: echo
    spec:
      containers:
        - name: app
          image: ghcr.io/subicura/echo:v1
          livenessProbe:
            httpGet:
              port: 3000
              path: /
          readinessProbe:
            httpGet:
              port: 3000
              path: /
    ```

- Pod 생성

    ```bash
    kubectl apply -f echo-pod-health.yml
    ```

- 실행 결과

    ```bash
    # Pod 목록 조회
    kubectl get pod
    
    # 결과
    NAME          READY   STATUS             RESTARTS         AGE
    echo-health   1/1     Running            0                9s
    ```

    - `3000`번 포트와 `/`경로는 정상적이기 때문에 Pod이 오류없이 생성된 것을 확인할 수 있다.
    - 서버가 살아있는지 체크하기 위해 주기적으로 Heath Check를 하게 된다. 어떤 내부적인 이유로 접속이 안 되면 차단을 끊거나 재시작을 하게 된다.