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

**다중 컨테이너**

- 대부분 `1 Pod = 1 컨테이너` 이지만 여러 개의 컨테이너를 가진 경우도 꽤 흔하다.
- 하나의 Pod에 속한 컨테이너는 서로 네트워크를 localhost로 공유하고 동일한 디렉토리를 공유할 수 있다.
- counter-pod-redis.yml

    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: counter
      labels:
        app: counter
    spec:
      containers:
        - name: app
          image: ghcr.io/subicura/counter:latest
          env:
            - name: REDIS_HOST
              value: "localhost"
        - name: db
          image: redis
    ```

    - 요청 횟수를 Redis에 저장하는 간단한 웹 애플리케이션을 다중 컨테이너로 생성한다.
    - 환경변수(env) 설정
        - env는 `name`과 `value`를 별도로 정의한다.
- Pod 생성

    ```bash
    # Pod 생성
    kubectl apply -f counter-pod-redis.yml
    ```

- 같은 Pod에 컨테이너가 생성되었기 때문에 counter 앱은 redis를 localhost로 접근할 수 있다.
- 아직 Service를 배우지 않아 브라우저 테스트는 어려우니 직접 컨테이너에 접속하여 테스트를 진행해본다.

    ```bash
    # Pod 목록 조회
    kubtctl get pod
    
    # Pod 로그 확인
    kubectl logs counter # 오류 발생 (컨테이너 지정 필요)
    kubectl logs counter app
    kubectl logs counter db
    
    # Pod의 app 컨테이너 접속
    kubectl exec -it counter -c app -- sh
    # apk add curl busy-box-extras # install curl, telnet
    # curl localhost:3000
    # curl localhost:3000
    # telnet localhost 6379
    dbsize
    KEYS *
    GET count
    quit
    
    # Pod 제거
    kubectl delete -f counter-pod-redis.yml
    ```

- 멀티 컨테이너는 도커에선 볼 수 없던 개념이다. 쿠버네티스는 멀티 컨테이너를 이용한 다양한 패턴이 존재한다.
- 로그를 수집하는 별도의 컨테이너를 같은 Pod으로 배포한다던가, 서버가 실행되기 전 데이터베이스를 마이그레이션 하는 초기화 컨테이너를 만들 수도 있다.

**마무리**

- Pod은 쿠버네티스에서 굉장히 중요한 요소이지만 단독으로 사용하는 경우는 거의 없다. Pod이 컨테이너를 관리하듯이 다른 컨트롤러가 Pod을 관리한다.

**실습**

- MongoDB Pod 생성
    - 조건


        | 키 | 값 |
        | --- | --- |
        | Pod 이름 | mongodb |
        | Pod Label | app: mongo |
        | Container 이름 | mongodb |
        | Container 이미지 | mongo:4 |
    - mongodb-pod.yml
        
        ```yaml
        apiVersion: v1
        kind: Pod
        metadata:
          name: mongodb
          labels:
            app: mongo
        spec:
          containers:
            - name: mongodb
              image: mongo:4
        ```
        
    - Pod 생성
        
        ```bash
        kubectl apply -f mongodb-pod.yml
        ```

- MySQL Pod 생성
    - 조건


        | 키 | 값 |
        | --- | --- |
        | Pod 이름 | mysql |
        | Pod Label | app: mysql |
        | Container 이름 | mysql |
        | Container 이미지 | mysql:latest |
        | Container 환경변수 | MYSQL_ROOT_PASSWORD: 1234 |
    - mysql-pod.yml
        
        ```yaml
        apiVersion: v1
        kind: Pod
        metadata:
          name: mysql
          labels:
            app: mysql
        spec:
          containers:
            - name: mysql
              image: mysql:latest
              env:
                - name: MYSQL_ROOT_PASSWORD
                  value: "1234"
        ```
        
    - Pod 생성
        
        ```bash
        kubectl apply -f mysql-pod.yml
        ```

**ReplicaSet**

- Pod을 단독으로 만들면 Pod에 어떤 문제(서버가 죽어서 Pod이 사라졌다던가)가 생겼을 때 자동으로 복구되지 않는다. 이러한 Pod을 정해진 수만큼 복제하고 관리하는 것이 ReplicaSet이다.

**ReplicaSet 만들기**

- echo-replicaset.yml

    ```yaml
    apiVersion: apps/v1
    kind: ReplicaSet
    metadata:
      name: echo-replicaset
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: echo
          tier: app
      template:
        metadata:
          labels:
            app: echo
            tier: app
        spec:
          containers:
            - name: echo
              image: ghcr.io/subicura/echo:v1
    ```

- ReplicaSet 생성 및 리소스 확인

    ```bash
    # ReplicaSet 생성
    kubectl apply -f echo-replicaset.yml
    
    # 리소스 확인
    kubectl get po,rs
    ```

- 실행 결과

    ```bash
    NAME                        READY   STATUS    RESTARTS   AGE
    pod/echo-replicaset-cfbfp   1/1     Running   0          112s
    
    NAME                              DESIRED   CURRENT   READY   AGE
    replicaset.apps/echo-replicaset   1         1         1       112s
    ```

- ReplicaSet은 label을 체크해서 원하는 수의 Pod이 없으면 새로운 Pod을 생성한다. 이를 설정으로 표현하면 다음과 같다.


    | 정의 | 설명 |
    | --- | --- |
    | spec.selector | label 체크 조건 |
    | spec.replicas  | 원하는 Pod의 개수 |
    | spec.template  | 생성할 Pod의 명세 |
- 생성된 Pod의 label을 확인

    ```bash
    kubectl get pod --show-labels
    ```

- 실행 결과

    ```bash
    NAME                    READY   STATUS    RESTARTS   AGE   LABELS
    echo-replicaset-cfbfp   1/1     Running   0          9m    app=echo,tier=app
    ```

- 설정한대로 `app=echo,tier=app` label이 보인다. 임의로 label을 제거하면 어떻게 되는지 확인해보자.

    ```bash
    # app- 를 지정하면 app label을 제거
    kubectl label pod/echo-replicaset-cfbfp app-
    
    # 다시 Pod 확인
    kubectl get pod --show-labels
    ```

- 실행 결과

    ```bash
    NAME                    READY   STATUS    RESTARTS   AGE   LABELS
    echo-replicaset-cfbfp   1/1     Running   0          13m   tier=app
    echo-replicaset-wwzf5   1/1     Running   0          7s    app=echo,tier=app
    ```

- 다시 label 추가

    ```bash
    # app=echo를 지정하여 app label 추가
    kubectl label pod/echo-replicaset-cfbfp app=echo
    
    # 다시 Pod 확인
    kubectl get pod --show-labels
    ```

- 실행 결과

    ```bash
    NAME                    READY   STATUS    RESTARTS   AGE   LABELS
    echo-replicaset-cfbfp   1/1     Running   0          16m   app=echo,tier=app
    ```


**ReplicaSet의 흐름**

- `ReplicaSet Controller`는 ReplicaSet 조건을 감시하면서 현재 상태와 원하는 상태가 다른 것을 체크한다.
- `ReplicaSet Controller`가 원하는 상태가 되도록 `Pod`을 생성하거나 제거한다.
- `Scheduler`는 API Server를 감시하면서 할당되지 않은 `Pod`이 있는지 체크한다.
- `Scheduler`는 할당되지 않은 새로운 `Pod`을 감지하고 적절한 `Node`에 배치한다.
- 이후 `Node`는 기존대로 동작한다.
- 정리
  - ReplicaSet은 `ReplicaSet Controller`가 관리하고 `Pod`의 할당은 여전히 `Scheduler`가 관리한다.

**스케일 아웃**

- ReplicaSet을 이용하면 손쉽게 Pod을 여러 개로 복제할 수 있다.
- echo-replicaset-scaled.yml

    ```yaml
    apiVersion: apps/v1
    kind: ReplicaSet
    metadata:
      name: echo-replicaset
    spec:
      replicas: 4
      selector:
        matchLabels:
          app: echo
          tier: app
      template:
        metadata:
          labels:
            app: echo
            tier: app
        spec:
          containers:
            - name: echo
              image: ghcr.io/subicura/echo:v1
    ```

- ReplicaSet 생성 및 리소스 확인

    ```bash
    # ReplicaSet 생성
    kubectl apply -f echo-replicaset-scaled.yml
    
    # 리소스 확인
    kubectl get po,rs
    ```


**마무리**

- ReplicaSet은 원하는 개수의 Pod을 유지하는 역할을 담당한다. label을 이용하여 Pod을 체크하기 때문에 label이 겹치치 않게 신경써서 정의해야 한다.
- 실무에서 ReplicaSet을 단독으로 쓰는 경우는 거의 없다. Deployment가 ReplicaSet을 이용하고 주로 Deployment를 사용한다.

**실습**

- 다음 조건을 만족하는 ReplicaSet을 만들자.


    | 키 | 값 |
    | --- | --- |
    | ReplicaSet 이름 | nginx |
    | ReplicaSet selector | app: nginx |
    | ReplicaSet 복제 수 | 3 |
    | Container 이름 | nginx |
    | Container 이미지 | nginx:latest |
- nginx-replicaset.yml

    ```yaml
    apiVersion: apps/v1
    kind: ReplicaSet
    metadata:
      name: nginx
    spec:
      replicas: 3
      selector:
        matchLabels:
          app: nginx
      template:
        metadata:
          labels:
            app: nginx
        spec:
          containers:
            - name: nginx
              image: nginx:latest
    ```

- ReplicaSet 생성 및 리소스 확인

    ```bash
    # ReplicaSet 생성
    kubectl apply -f nginx-replicaset.yml
    
    # 리소스 확인
    kubectl get po,rs
    
    # ReplicaSet 제거
    kubectl delete -f nginx-replicaset.yml
    ```

**Deployment**

- Deployment는 쿠버네티스에서 가장 널리 사용되는 오브젝트이다. ReplicaSet을 이용하여 Pod을 업데이트하고 이력을 관리하여 롤백하거나 특정 버전으로 돌아갈 수 있다.

**Deployment 만들기**

- echo-deployment.yml

    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: echo-deploy
    spec:
      replicas: 4
      selector:
        matchLabels:
          app: echo
          tier: app
      template:
        metadata:
          labels:
            app: echo
            tier: app
        spec:
          containers:
            - name: echo
              image: ghcr.io/subicura/echo:v1
    ```

- Deployment 생성

    ```bash
    # Deployment 생성
    kubectl apply -f echo-deployment.yml
    
    # 상태 확인
    kubectl get all
    ```

- 이미지 버전 변경
  - echo-deployment-v2.yml

      ```yaml
      apiVersion: apps/v1
      kind: Deployment
      metadata:
        name: echo-deploy
      spec:
        replicas: 4
        selector:
          matchLabels:
            app: echo
            tier: app
        template:
          metadata:
            labels:
              app: echo
              tier: app
          spec:
            containers:
              - name: echo
                image: ghcr.io/subicura/echo:v2
      ```

- 변경된 버전 이미지 생성

    ```bash
    # 새로운 이미지 업데이트
    kubectl apply -f echo-deployment-v2.yml
    
    # 리소스 확인
    kubectl get all
    
    # 실행 결과
    NAME                               READY   STATUS    RESTARTS   AGE
    pod/echo-deploy-54c84dd47b-5cgjz   1/1     Running   0          8s
    pod/echo-deploy-54c84dd47b-5pxk4   1/1     Running   0          3s
    pod/echo-deploy-54c84dd47b-fzblz   1/1     Running   0          8s
    pod/echo-deploy-54c84dd47b-n5q5l   1/1     Running   0          4s
    
    NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
    service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   15d
    
    NAME                          READY   UP-TO-DATE   AVAILABLE   AGE
    deployment.apps/echo-deploy   4/4     4            4           5m53s
    
    NAME                                     DESIRED   CURRENT   READY   AGE
    replicaset.apps/echo-deploy-54c84dd47b   4         4         4       8s
    replicaset.apps/echo-deploy-5f9566c4d9   0         0         0       5m53s
    ```

- Deployment는 새로운 이미지로 업데이트하기 위해 ReplicaSet을 이용한다. 버전을 업데이트하면 새로운 ReplicaSet을 생성하고 해당 ReplicaSet이 새로운 버전의 Pod을 생성한다.
- 엄밀히 말하면 “Pod을 새로운 버전으로 업데이트한다.”는 잘못된 표현이고, “새로운 버전의 Pod을 생성하고 기존 Pod을 제거한다.”가 정확한 표현이다.
- 흐름
  - 새로운 ReplicaSet을 0 → 1개로 조정하고 정상적으로 Pod이 동작하면 기존 ReplicaSet을 4 → 3개로 조정한다.
  - 새로운 ReplicaSet을 1 → 2개로 조정하고 정상적으로 Pod이 동작하면 기존 ReplicaSet을 3 → 2개로 조정한다.
  - 새로운 ReplicaSet을 2 → 3개로 조정하고 정상적으로 Pod이 동작하면 기존 ReplicaSet을 2 → 1개로 조정한다.
  - 최종적으로 새로운 ReplicaSet을 4개로 조정하고 정상적으로 Pod이 동작하면 기존 ReplicaSet을 0개로 조정하면 업데이트가 완료된다.
- 생성한 Deployment의 상세 상태를 보면 더 자세히 알 수 있다.

    ```bash
    # Deployment 상세 상태 확인
    kubectl describe deploy/echo-deploy
    
    # 실행 결과
    Events:
      Type    Reason             Age    From                   Message
      ----    ------             ----   ----                   -------
      Normal  ScalingReplicaSet  12m    deployment-controller  Scaled up replica set echo-deploy-5f9566c4d9 to 4
      Normal  ScalingReplicaSet  6m30s  deployment-controller  Scaled up replica set echo-deploy-54c84dd47b to 1
      Normal  ScalingReplicaSet  6m30s  deployment-controller  Scaled down replica set echo-deploy-5f9566c4d9 to 3 from 4
      Normal  ScalingReplicaSet  6m30s  deployment-controller  Scaled up replica set echo-deploy-54c84dd47b to 2 from 1
      Normal  ScalingReplicaSet  6m26s  deployment-controller  Scaled down replica set echo-deploy-5f9566c4d9 to 2 from 3
      Normal  ScalingReplicaSet  6m26s  deployment-controller  Scaled up replica set echo-deploy-54c84dd47b to 3 from 2
      Normal  ScalingReplicaSet  6m25s  deployment-controller  Scaled down replica set echo-deploy-5f9566c4d9 to 1 from 2
      Normal  ScalingReplicaSet  6m25s  deployment-controller  Scaled up replica set echo-deploy-54c84dd47b to 4 from 3
      Normal  ScalingReplicaSet  6m25s  deployment-controller  Scaled down replica set echo-deploy-5f9566c4d9 to 0 from 1
    ```


**Deployment Controller의 동작 흐름**

- `Deployment Controller`는 Deployment 조건을 감시하면서 현재 상태와 원하는 상태가 다른 것을 체크한다.
- `Deployment Controller`가 원하는 상태가 되도록 `ReplicaSet`을 설정한다.
- `ReplicaSet Controller`는 ReplicaSet 조건을 감시하면서 현재 상태와 원하는 상태가 다른 것을 체크한다.
- `ReplicaSet Controller`가 원하는 상태가 되도록 `Pod`을 생성하거나 제거한다.
- `Scheduler`는 API Server를 감시하면서 할당되지 않은 Pod이 있는지 체크한다.
- `Scheduler`는 할당되지 않은 새로운 `Pod`을 감지하고 적절한 `Node`에 배치한다.
- 이후 `Node`는 기존대로 동작한다.

**버전 관리**

- Deployment는 변경된 상태를 기록한다.

    ```bash
    # 히스토리 확인
    kubectl rollout history deploy/echo-deploy
    
    # revision 1 히스토리 상세 확인
    kubectl rollout history deploy/echo-deploy --revision=1
    
    # 바로 전으로 롤백
    kubectl rollout undo deploy/echo-deploy
    
    # 특정 버전으로 롤백
    kubectl rollout undo deploy/echo-deploy --to-revision=2
    
    # 리소스 확인
    kubectl get all
    ```


**배포 전략 설정**

- Deployment는 다양한 방식의 배포 전략이 있다. 여기서는 롤링업데이트(RollingUpdate) 방식을 사용할 때 동시에 업데이트하는 Pod의 개수를 변경해본다.
- echo-deployment-rollingupdate.yml

    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: echo-deploy
    spec:
      replicas: 4
      minReadySeconds: 5
      strategy:
        type: RollingUpdate
        rollingUpdate:
          maxSurge: 3
          maxUnavailable: 3
      selector:
        matchLabels:
          app: echo
          tier: app
      template:
        metadata:
          labels:
            app: echo
            tier: app
        spec:
          containers:
            - name: echo
              image: ghcr.io/subicura/echo:v1
    ```

  - maxSurge와 maxUnavailable의 기본값은 25%이다. 대부분의 상황에서 적당하지만 상황에 따라 적절하게 조정이 필요하다.
- Deployment 생성

    ```bash
    # Deployment 생성
    kubectl apply -f echo-deployment-rollingupdate.yml
    
    # 리소스 확인
    kubectl get all
    
    # Deployment 상세 확인
    kubectl describe deploy/echo-deploy
    
    # 실행 결과
    Events:
      Type    Reason             Age                  From                   Message
      ----    ------             ----                 ----                   -------
      Normal  ScalingReplicaSet  51m                  deployment-controller  Scaled up replica set echo-deploy-5f9566c4d9 to 4
      Normal  ScalingReplicaSet  45m                  deployment-controller  Scaled up replica set echo-deploy-54c84dd47b to 1
      Normal  ScalingReplicaSet  45m                  deployment-controller  Scaled down replica set echo-deploy-5f9566c4d9 to 3 from 4
      Normal  ScalingReplicaSet  45m                  deployment-controller  Scaled up replica set echo-deploy-54c84dd47b to 2 from 1
      Normal  ScalingReplicaSet  45m                  deployment-controller  Scaled down replica set echo-deploy-5f9566c4d9 to 2 from 3
      Normal  ScalingReplicaSet  45m                  deployment-controller  Scaled up replica set echo-deploy-54c84dd47b to 3 from 2
      Normal  ScalingReplicaSet  45m                  deployment-controller  Scaled down replica set echo-deploy-5f9566c4d9 to 1 from 2
      Normal  ScalingReplicaSet  45m                  deployment-controller  Scaled up replica set echo-deploy-54c84dd47b to 4 from 3
      Normal  ScalingReplicaSet  45m                  deployment-controller  Scaled down replica set echo-deploy-5f9566c4d9 to 0 from 1
      Normal  ScalingReplicaSet  20m                  deployment-controller  Scaled down replica set echo-deploy-54c84dd47b to 3 from 4
      Normal  ScalingReplicaSet  20m                  deployment-controller  Scaled up replica set echo-deploy-5f9566c4d9 to 1 from 0
      Normal  ScalingReplicaSet  20m                  deployment-controller  Scaled up replica set echo-deploy-5f9566c4d9 to 2 from 1
      Normal  ScalingReplicaSet  20m                  deployment-controller  Scaled down replica set echo-deploy-54c84dd47b to 2 from 3
      Normal  ScalingReplicaSet  20m                  deployment-controller  Scaled up replica set echo-deploy-5f9566c4d9 to 3 from 2
      Normal  ScalingReplicaSet  20m                  deployment-controller  Scaled down replica set echo-deploy-54c84dd47b to 1 from 2
      Normal  ScalingReplicaSet  16m                  deployment-controller  Scaled up replica set echo-deploy-54c84dd47b to 1 from 0
      Normal  ScalingReplicaSet  16m (x7 over 16m)    deployment-controller  (combined from similar events): Scaled down replica set echo-deploy-5f9566c4d9 to 0 from 1
      Normal  ScalingReplicaSet  2m26s (x2 over 20m)  deployment-controller  Scaled up replica set echo-deploy-5f9566c4d9 to 4 from 3
      Normal  ScalingReplicaSet  2m26s                deployment-controller  Scaled up replica set echo-deploy-5f9566c4d9 to 3 from 0
      Normal  ScalingReplicaSet  2m26s                deployment-controller  Scaled down replica set echo-deploy-54c84dd47b to 1 from 4
      Normal  ScalingReplicaSet  2m19s (x2 over 20m)  deployment-controller  Scaled down replica set echo-deploy-54c84dd47b to 0 from 1
    ```

  - Pod을 하나씩 생성하지 않고 한 번에 3개가 생성된 것을 확인할 수 있다.
- Deployment는 가장 흔하게 사용하는 배포 방식이다. 이외에 StatefulSet, DaemonSet, CronJob, Job 등이 있지만 사용법은 크게 다르지 않다.

**실습**

- #1 다음 조건을 만족하는 Deployment를 만들자.


    | 키 | 값 |
    | --- | --- |
    | Deployment 이름 | nginx |
    | Deployment Label | app: nginx |
    | Deployment 복제 수 | 3 |
    | Container 이름 | nginx |
    | Container 이미지 | nginx:1.14.2 |
- nginx-deployment.yml

    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: nginx
    spec:
      replicas: 3
      selector:
        matchLabels:
          app: nginx
      template:
        metadata:
          labels:
            app: nginx
        spec:
          containers:
            - name: nginx
              image: nginx:1.14.2
    ```

- #2 복제 개수를 5로 조정하자.
- nginx-deployment.yml

    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: nginx
    spec:
      replicas: 5
      selector:
        matchLabels:
          app: nginx
      template:
        metadata:
          labels:
            app: nginx
        spec:
          containers:
            - name: nginx
              image: nginx:1.14.2
    ```

- #3 이미지를 nginx:1.19.5로 변경하자.
- nginx-deployment.yml

    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: nginx
    spec:
      replicas: 5
      selector:
        matchLabels:
          app: nginx
      template:
        metadata:
          labels:
            app: nginx
        spec:
          containers:
            - name: nginx
              image: nginx:1.19.5
    ```

**Service**

- Pod은 자체 IP를 가지고 다른 Pod과 통신할 수 있지만, 쉽게 사라지고 생성되는 특징 때문에 직접 통신하는 방법은 권장하지 않는다. 쿠버네티스는 Pod과 직접 통신하는 방법 대신, 별도의 고정 IP를 가진 서비스를 만들고, 그 서비스를 통해 Pod에 접근하는 방식을 사용한다.

**Service(ClusterIP) 만들기**

- ClusterIP는 클러스터 내부에 새로운 IP를 할당하고 여러 개의 Pod을 바라보는 로드밸런서 기능을 제공한다. 그리고 서비스 이름을 내부 도메인 서버에 등록하여 Pod 간에 서비스 이름으로 통신할 수 있다.
- counter 앱 중에 redis를 서비스로 노출해보도록 하자.
- counter-redis-service.yml

    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: redis
    spec:
      selector:
        matchLabels:
          app: counter
          tier: db
      template:
        metadata:
          labels:
            app: counter
            tier: db
        spec:
          containers:
            - name: redis
              image: redis
              ports:
                - containerPort: 6379
                  protocol: TCP
    
    ---
    apiVersion: v1
    kind: Service
    metadata:
      name: redis
    spec:
      ports:
        - port: 6379
          protocol: TCP
      selector:
        app: counter
        tier: db
    ```

- 하나의 YAML 파일에 여러 개의 리소스를 정의할 땐 “—-”를 구분자로 사용한다.
- Service 생성

    ```bash
    # Service 생성
    kubectl apply -f counter-redis-service.yml
    
    # 리소스 확인
    kubectl get all
    ```

- 같은 클러스터에서 생성된 Pod이라면 `redis`라는 도메인으로 redis Pod에 접근할 수 있다. (`redis.default.svc.cluster.local` 로도 접근이 가능하다. 서로 다른 namespace와 cluster를 구분할 수 있다.)
- ClusterIP 서비스의 설정


    | 정의 | 설명 |
    | --- | --- |
    | spec.ports.port | 서비스가 생성할 Port |
    | spec.ports.targetPort | 서비스가 접근할 Pod의 Port (기본: port랑 동일) |
    | spec.selector | 서비스가 접근할 Pod의 label 조건 |
- redis Service의 selector는 redis Deployment에 정의한 label을 사용했기 때문에 해당 Pod을 가리킨다. 그리고 해당 Pod의 6379 포트로 연결하였다.
- 이제 redis에 접근할 counter 앱을 Deployment로 만들자.
- counter-app.yml

    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: counter
    spec:
      selector:
        matchLabels:
          app: counter
          tier: app
      template:
        metadata:
          labels:
            app: counter
            tier: app
        spec:
          containers:
            - name: counter
              image: ghcr.io/subicura/counter:latest
              env:
                - name: REDIS_HOST
                  value: "redis"
                - name: REDIS_PORT
                  value: "6379"
    ```

- Service 생성

    ```bash
    # Service 생성
    kubectl apply -f counter-app.yml
    
    # 리소스 확인
    kubectl get all
    
    # 정상 작동 테스트
    kubectl exec -it counter-59b58897d6-zg4pj -- sh
    
    # curl 및 telnet 설치 후 테스트
    apk add curl busybox-extras # install telnet
    curl localhost:3000
    curl localhost:3000
    telnet redis 6379
    dbsize
    KEYS *
    GET count
    quit
    ```

- Service를 통해 Pod과 성공적으로 연결된 것을 확인할 수 있다.

**Service 생성 흐름**

- Service는 각 Pod을 바라보는 로드밸런서 역할을 하면서 내부 도메인 서버에 새로운 도메인을 생성한다. Service가 어떻게 동작하는지 살펴보자.
- `Endpoint Controller`는 `Service`와 `Pod`을 감시하면서 조건에 맞는 Pod의 IP를 수집한다.
- `Endpoint Controller`가 수집한 IP를 가지고 `Endpoint`를 생성한다.
- `Kube-Proxy`는 `Endpoint` 변화를 감시하고 노드의 `iptables`를 설정한다.
- `CoreDNS`는 `Service`를 감시하고 서비스 이름과 IP를 `CoreDNS`에 추가한다.
- `iptables`는 커널 레벨의 네트워크 도구이고 `CoreDNS`는 빠르고 편리하게 사용할 수 있는 클러스터 내부용 도메인 네임 서버이다. 각각의 역할은 `iptables` 설정으로 여러 IP에 트래픽을 전달하고 `CoreDNS`를 이용하여 IP 대신 도메인 이름을 사용한다.
- iptables는 규칙이 많아지면 성능이 느려지는 이슈가 있어, `ipvs`를 사용하는 옵션도 있다.
- CoreDNS는 클러스터에서 호환성을 위해 `kube-dns`라는 이름으로 생성된다.
- Endpoint는 서비스의 접속 정보를 가지고 있다. Endpoint 상태를 확인해보자.

    ```bash
    # Endpoint 확인
    kubectl get endpoints
    kubectl get ep # 줄여서
    
    # redis Endpoint 상세 정보 확인
    kubectl describe ep/redis
    
    # Pod의 IP 확인
    kubectl get po -o wide
    ```


**Service(NodePort) 만들기**

- ClusterIP는 클러스터 내부에서만 접근할 수 있다. 클러스터 외부(노드)에서 접근할 수 있도록 NodePort 서비스를 만들어보자.
- counter-nodeport.yml

    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: counter-np
    spec:
      type: NodePort
      ports:
        - port: 3000
          protocol: TCP
          nodePort: 31000
      selector:
        app: counter
        tier: app
    ```

- NodePort 서비스의 설정


    | 정의 | 설명 |
    | --- | --- |
    | spec.ports.nodePort | 노드에 오픈할 Port (미지정 시 30000-32768 중에 자동 할당) |
- Service 생성

    ```bash
    # Service 생성
    kubectl apply -f counter-nodeport.yml
    
    # Service 확인
    kubectl get svc
    
    # minikube ip 확인
    minikube ip # 192.168.49.2
    
    # counter-np Service 터널링 활성화
    minikube service counter-np
    
    # 브라우저에서 192.168.49.2:31000으로 접속되는지 확인
    ```

- NodePort는 클러스터의 모든 노드에 포트를 오픈한다. 지금은 테스트라서 하나의 노드밖에 없지만 여러 개의 노드가 있다면 아무 노드로 접근해도 지정한 Pod으로 접근할 수 있다.
- NodePort는 ClusterIP의 기능을 기본으로 포함한다.