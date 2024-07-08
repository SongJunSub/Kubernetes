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