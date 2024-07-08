# 쿠버네티스 사용

---

**워드프레스 배포**

- 1차 목표는 PHP와 MySQL로 구성된 워드프레스라는 샘플 프로젝트를 쿠버네티스로 배포하는 것이다.
- Docker Compose를 이용한 배포와 차이점을 함께 확인해본다.
- **Docker Compose를 이용한 배포**
    - docker-compose.yml

        ```yaml
        version: "3"
        
        services:
          wordpress:
            image: wordpress:5.5.3-apache
            platform: linux/amd64
            environment:
              WORDPRESS_DB_HOST: mysql:3306
              WORDPRESS_DB_NAME: wordpress
              WORDPRESS_DB_USER: root
              WORDPRESS_DB_PASSWORD: password
            ports:
              - "30000:80"
        
          mysql:
            image: mysql:5.6
            platform: linux/amd64
            environment:
              MYSQL_ROOT_PASSWORD: password
              MYSQL_DATABASE: wordpress
        ```

    - 해당 디렉토리로 이동하여 `docker-compose up -d` 명령어를 실행하여 컨테이너를 생성한다.
    - [localhost:30000](http://localhost:30000) 으로 접속하여 정상 접속 여부를 확인한다.
    - `docker-compose down` 명령어를 실행하여 중단시킨다.
- **쿠퍼네티스를 이용한 배포**
    - 쿠버네티스는 조금 더 다양한 컴포넌트로 구성된다.
    - wordpress-k8s.yml

        ```yaml
        apiVersion: apps/v1
        kind: Deployment
        metadata:
          name: wordpress-mysql
          labels:
            app: wordpress
        spec:
          selector:
            matchLabels:
              app: wordpress
              tier: mysql
          template:
            metadata:
              labels:
                app: wordpress
                tier: mysql
            spec:
              containers:
                - image: mysql:latest
                  name: mysql
                  env:
                    - name: MYSQL_DATABASE
                      value: wordpress
                    - name: MYSQL_ROOT_PASSWORD
                      value: password
                  ports:
                    - containerPort: 3306
                      name: mysql
        
        ---
        apiVersion: v1
        kind: Service
        metadata:
          name: wordpress-mysql
          labels:
            app: wordpress
        spec:
          ports:
            - port: 3306
          selector:
            app: wordpress
            tier: mysql
        
        ---
        apiVersion: apps/v1
        kind: Deployment
        metadata:
          name: wordpress
          labels:
            app: wordpress
        spec:
            replicas: 2
          selector:
            matchLabels:
              app: wordpress
              tier: frontend
          template:
            metadata:
              labels:
                app: wordpress
                tier: frontend
            spec:
              containers:
                - image: arm64v8/wordpress:5.9.1-php8.1-apache
                  name: wordpress
                  env:
                    - name: WORDPRESS_DB_HOST
                      value: wordpress-mysql
                    - name: WORDPRESS_DB_NAME
                      value: wordpress
                    - name: WORDPRESS_DB_USER
                      value: root
                    - name: WORDPRESS_DB_PASSWORD
                      value: password
                  ports:
                    - containerPort: 80
                      name: wordpress
        
        ---
        apiVersion: v1
        kind: Service
        metadata:
          name: wordpress
          labels:
            app: wordpress
        spec:
          type: NodePort
          ports:
            - port: 80
          selector:
            app: wordpress
            tier: frontend
        ```

    - `kubectl apply -f wordpress-k8s.yml` 명령어를 실행하여 쿠버네티스에 적용시킨다.
    - `kubectl get all`
        - 현재 Default Namespace에 배포되어 있는 리소스들을 확인할 수 있다.
        - service/wordpress         NodePort    10.97.250.36   <none>        80:31782/TCP   80s
        - 이 내용의 31782 포트로 접속을 하게 된다.
    - `kubectl get pods` / `kubectl get services`
        - 현재 Default Namespace에 배포되어 있는 Pod이나 Service들을 확인할 수 있다.
    - `kubectl logs <pod-name>`
        - 특정 Pod의 로그를 확인할 수 있다.
    - `minikube ip`
        - minikube의 IP가 192.168.49.2이므로 192.168.49.2:31782로 접속하여 정상 접속이 되는지 확인한다.
    - ❗️ARM64 아키텍처 기반의 맥북에서는 `minikube service wordpress` 명령어를 실행하여 터널링을 활성화 한 상태에서 접속해야 한다.
    - `kubectl delete pod/<pod-name>`
        - 특정 Pod을 삭제한다.
    - `kubectl delete -f wordpress-k8s.yml`
        - wordpress-k8s.yml로 만들었던 것을 제거한다.
- Docker와 Kubernetes의 큰 차이점은 Kubernetes는 서비스를 중단시켰을 때 쿠버네티스가 자동으로 새로 생성을 해준다.
- Kubernetes는 컨테이너 추가 시 내부적으로 자동으로 로드 밸런서를 설정해준다.

**kubectl 기본 명령어**

- 쿠버네티스는 대표적인 GUI 프로그램으로 Lens가 있지만 직접 작성할 수 있는 역량을 가지고 있어야 한다.
- kubectl의 역할은 쿠버네티스의 상태를 확인하고 원하는 상태를 요청하는 것이다. 추가로 컨테이너 로그도 확인하고 원격으로 접속할 수 있다.
- `apply`
    - 원하는 상태를 적용한다. 보통 -f 옵션으로 파일과 함께 사용한다.
- `get`
    - 리소스 목록을 보여준다.
- `describe`
    - 리소스 상태를 자세하게 보여준다.
- `delete`
    - 리소스를 제거한다.
- `logs`
    - 컨테이너의 로그를 확인한다.
- `exec`
    - 컨테이너에 명령어를 전달한다. 컨테이너에 접근할 때 주로 사용한다.
- `config`
    - kubectl 설정을 관리한다.
- alias 이용
    - alias로 kubectl 명령어를 k로 줄여쓸 수 있다.

        ```bash
        # alias 설정
        alias k='kubectl'
        
        # shell 설정 추가
        echo "alias k='kubectl'" >> ~/.bashrc
        source ~/.bashrc
        ```

- 상태 설정하기 (apply)
    - 원하는 리소스의 상태를 YAML로 작성하고 apply 명령어로 선언한다.
    - `kubectl apply -f [파일명 또는 URL]`
    - 워드프레스를 URL로 배포해보도록 한다.
        - `kubectl apply -f https://subicura.com/k8s/code/guide/index/wordpress-k8s.yml`
- 리소스 목록보기 (get)
    - `kubectl get [TYPE]`
    - `-o`
        - 출력 형태를 변경할 수 있는 옵션이다.
    - `—-show-labels`
        - 레이블을 확인할 수 있는 옵션이다.

    ```bash
    # Pod 조회
    kubectl get pod
    
    # 줄임말(Shortname)과 복수형 사용 가능
    kubectl get pods
    kubectl get po
    
    # 여러 TYPE 입력
    kubectl get pod,service
    kubectl get po,svc
    
    # Pod, ReplicaSet, Deployment, Service, Job 조회 => all
    kubectl get all
    
    # 결과 포맷 변경
    kubectl get pod -o wide
    kubectl get pod -o yaml
    kubectl get pod -o json
    
    # Label 조회
    kubectl get pod --show-labels
    ```

- 리소스 상세 상태보기 (describe)
    - `kubectl describe [TYPE]/[NAME] 또는 [TYPE] [NAME]`
    - 특정 리소스의 상태가 궁금하거나 생성이 실패한 이유를 확인할 때 주로 사용한다.

    ```bash
    # Pod 조회로 이름 검색
    kubectl get pod
    
    # 조회한 이름으로 상세 확인
    kubectl describe pod/wordpress-544b66f64c-dstgr
    ```

- 리소스 제거 (delete)
    - `kubectl delete [TYPE]/[NAME] 또는 [TYPE] [NAME]`

    ```bash
    # Pod 조회로 이름 검색
    kubectl get pod
    
    # 조회한 Pod 제거
    kubectl delete pod/wordpress-544b66f64c-dstgr
    ```

    - Pod을 제거해도 자꾸 살아나는 것은 정상적인 결과이다. ReplicaSet이 Pod의 개수를 유지해준다.
- 컨테이너 로그 조회 (logs)
    - `kubectl logs [POD_NAME]`
    - 실시간 로그를 보고 싶다면 `-f` 옵션을 이용하고 하나의 Pod에 여러 개의 컨테이너가 있는 경우는 `-c` 옵션으로 컨테이너를 지정해야 한다.

    ```bash
    # Pod 조회로 이름 검색
    kubectl get pod
    
    # 조회한 Pod 로그 조회
    kubectl logs wordpress-544b66f64c-65mjs
    
    # 실시간 로그 보기
    kubectl logs -f wordpress-544b66f64c-65mjs
    ```

- 컨테이너 명령어 전달 (exec)
    - `kubectl exec [-it] [POD_NAME] -- [COMMAND]`
    - Shell로 접속하여 컨테이너 상태를 확인하는 경우에 `-it` 옵션을 사용하고 여러 개의 컨테이너가 있는 경우엔 `-c` 옵션으로 컨테이너를 지정한다.

    ```bash
    # Pod 조회로 이름 검색
    kubectl get pod
    
    # 조회한 Pod의 컨테이너에 접속
    kubectl exec -it wordpress-544b66f64c-65mjs -- bash
    ```

- 설정 관리 (config)
    - kubectl은 여러 개의 쿠버네티스 클러스터를 Context로 설정하고 필요에 따라 선택할 수 있다. 현재 어떤 Context로 설정되어 있는지 확인하고 원하는 Context를 지정한다.

    ```bash
    # 현재 Context 확인
    kubectl config current-context
    
    # Context 설명
    kubectl config use-context minikube
    ```

- ETC 명령어

    ```bash
    # 전체 오브젝트 종류 확인
    kubectl api-resources
    
    # 특정 오브젝트 설명 보기
    kubectl explain pod
    ```

- 테스트 후 워드프레스 리소스를 제거한다.

    ```bash
    kubectl delete -f wordpress-k8s.yml
    ```