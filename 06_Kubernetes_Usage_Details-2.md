# 쿠버네티스 사용 상세 - 2

---

**웹 애플리케이션 배포**

- Pod, ReplicaSet, Deployment, Service를 이용하여 기본적인 웹 애플리케이션을 배포해본다.

**워드프레스 배포**

- MySQL


    | 키 | 값 |
    | --- | --- |
    | 컨테이너 이미지 | mysql:latest |
    | MySQL 포트 | 3306 |
    | 환경변수 | MYSQL_ROOT_PASSWORD: 1234 |
- Wordpress


    | 키 | 값 |
    | --- | --- |
    | 컨테이너 이미지 | wordpress:5.9.1-php8.1-apache |
    | Wordpress 포트 | 80 |
    | 환경변수 | WORDPRESS_DB_HOST: [wordpress host] |
    | 환경변수 | WORDPRESS_DB_NAME: wordpress |
    | 환경변수 | WORDPRESS_DB_USER: root |
    | 환경변수 | WORDPRESS_DB_PASSWORD: 1234 |
- wordpress.yml

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
            - name: mysql
              image: mysql:latest
              env:
                - name: MYSQL_DATABASE
                  value: wordpress
                - name: MYSQL_ROOT_PASSWORD
                  value: "1234"
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
            - name: wordpress
              image: wordpress:5.9.1-php8.1-apache
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
          nodePort: 30000
      selector:
        app: wordpress
        tier: frontend
    ```

- Deployment, Service 생성

    ```bash
    # Deployment, Service 생성
    kubectl apply -f wordpress.yml
    
    # 리소스 확인
    kubectl get all
    
    # 서비스 시작
    minikube service wordpress
    ```


**투표 애플리케이션 배포**

- Redis


    | 키 | 값 |
    | --- | --- |
    | 컨테이너 이미지 | redis:latest |
    | Redis 포트 | 6379 |
- Postgress


    | 키 | 값 |
    | --- | --- |
    | 컨테이너 이미지 | postgress:latest |
    | Postgress 포트 | 3306 |
    | 환경변수 | POSTGRESS_USER: postgress |
    | 환경변수 | POSTGRESS_PASSWORD: postgress |
- Worker


    | 키 | 값 |
    | --- | --- |
    | 컨테이너 이미지 | ghcr.io/subicura/voting/worker:latest |
    | 환경변수 | REDIS_HOST: [redis ip] |
    | 환경변수 | REDIS_PORT: [redis port] |
    | 환경변수 | POSTGRESS_HOST: [postgress ip] |
    | 환경변수 | POSTGRESS_PORT: [postgress port] |
- Vote (노드 31000으로 연결)


    | 키 | 값 |
    | --- | --- |
    | 컨테이너 이미지 | ghcr.io/subicura/voting/vote:latest |
    | Vote 포트 | 3306 |
    | 환경변수 | REDIS_HOST: [redis ip] |
    | 환경변수 | REDIS_PORT: [redis port] |
- Result (노드 31001로 연결)


    | 키 | 값 |
    | --- | --- |
    | 컨테이너 이미지 | ghcr.io/subicura/voting/result:latest |
    | Result 포트 | 80 |
    | 환경변수 | POSTGRESS_HOST: [postgress ip] |
    | 환경변수 | POSTGRESS_PORT: [postgress port] |
- vote.yml

    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: vote
    spec:
      selector:
        matchLabels:
          service: vote
      template:
        metadata:
          labels:
            service: vote
        spec:
          containers:
            - name: vote
              image: ghcr.io/subicura/voting/vote
              env:
                - name: REDIS_HOST
                  value: "redis"
                - name: REDIS_PORT
                  value: "6379"
              livenessProbe:
                httpGet:
                  path: /
                  port: 80
              readinessProbe:
                httpGet:
                  path: /
                  port: 80
              ports:
                - containerPort: 80
                  protocol: TCP
    
    ---
    apiVersion: v1
    kind: Service
    metadata:
      name: vote
    spec:
      type: NodePort
      ports:
        - port: 80
          nodePort: 31000
          protocol: TCP
      selector:
        service: vote
    
    ---
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: result
    spec:
      selector:
        matchLabels:
          service: result
      template:
        metadata:
          labels:
            service: result
        spec:
          containers:
            - name: result
              image: ghcr.io/subicura/voting/result
              env:
                - name: POSTGRES_HOST
                  value: "db"
                - name: POSTGRES_PORT
                  value: "5432"
              livenessProbe:
                httpGet:
                  path: /
                  port: 80
              readinessProbe:
                httpGet:
                  path: /
                  port: 80
              ports:
                - containerPort: 80
                  protocol: TCP
    
    ---
    apiVersion: v1
    kind: Service
    metadata:
      name: result
    spec:
      type: NodePort
      ports:
        - port: 80
          nodePort: 31001
          protocol: TCP
      selector:
        service: result
    
    ---
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: worker
    spec:
      selector:
        matchLabels:
          service: worker
      template:
        metadata:
          labels:
            service: worker
        spec:
          containers:
            - name: worker
              image: ghcr.io/subicura/voting/worker
              env:
                - name: REDIS_HOST
                  value: "redis"
                - name: REDIS_PORT
                  value: "6379"
                - name: POSTGRES_HOST
                  value: "db"
                - name: POSTGRES_PORT
                  value: "5432"
    
    ---
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: redis
    spec:
      selector:
        matchLabels:
          service: redis
      template:
        metadata:
          labels:
            service: redis
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
        service: redis
    
    ---
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: db
    spec:
      selector:
        matchLabels:
          service: db
      template:
        metadata:
          labels:
            service: db
        spec:
          containers:
            - name: db
              image: postgres:latest
              env:
                - name: POSTGRES_USER
                  value: "postgres"
                - name: POSTGRES_PASSWORD
                  value: "postgres"
              ports:
                - containerPort: 5432
                  protocol: TCP
    
    ---
    apiVersion: v1
    kind: Service
    metadata:
      name: db
    spec:
      ports:
        - port: 5432
          protocol: TCP
      selector:
        service: db
    ```

- Vote 관련 Deployment, Service 생성

    ```bash
    # Deployment, Service 생성
    kubectl apply -f vote.yml
    
    # 리소스 확인
    kubectl get all
    ```

**Ingress**

- `Ingress`는 여러 도메인을 각각에 맞는 서비스에 분기 시켜준다. 예를 들어, `example.com`, `subicura.com/blog`, `subicura.com/help` 라는 도메인이 있을 때 각각의 주소로 서로 다른 서비스에 접근할 수 있다. `80(http)` 또는 `443(https)` 포트로 여러 개의 서비스를 연결해야 하는데 이럴 때 `Ingress`를 사용한다.

**Ingress 만들기**

- echo 웹 애플리케이션을 버전별로 도메인을 다르게 만들어 보자.
- `minikube ip`로 테스트 클러스터의 노드 IP를 구하고 도메인 주소로 사용한다. 결과 IP는 `192.168.49.2`이며, 사용할 도메인은 다음과 같다.
  - v1.echo.192.168.49.2.sslip.io
  - v2.echo.192.168.49.2.sslip.io
- 도메인을 테스트하려면 여러가지 설정이 필요하다. 여기서는 별도의 설정없이 IP 주소를 도메인에 넣어 바로 사용할 수 있는 [sslip.io](http://sslip.io) 서비스를 이용하도록 한다.

**minikube에 Ingress 활성화하기**

- Ingress는 Pod, ReplicaSet, Deployment, Service와 달리 별도의 컨트롤러를 설치해야 한다. 여러가지 컨트롤러 중에 입맛에 맞게 고를 수 있는데 여기서는 nginx ingress controller를 사용한다.
- Ingress Controller
  - nginx를 제외한 대표적인 컨트롤러로 haproxy, traefik, alb 등이 있다.
- 명령어

    ```bash
    # Ingress 활성화
    minikube addons enable ingress
    
    # 설치된 Namespace 확인
    kubectl get pod -A
    
    # Ingress 컨트롤러 확인
    kubectl -n ingress-nginx get pod
    
    # 실행 결과
    NAME                                        READY   STATUS      RESTARTS   AGE
    ingress-nginx-admission-create-7x72c        0/1     Completed   0          2m42s
    ingress-nginx-admission-patch-m25g6         0/1     Completed   1          2m42s
    ingress-nginx-controller-768f948f8f-sbzgt   1/1     Running     0          2m42s
    ```

- Docker Driver를 사용 중이라면 `minikube service ingress-nginx-controller -n ingress-nginx -—url` 명령어를 이용하여 접속 주소를 확인한다.
- 실행 결과

    ```bash
    # 접속 주소 확인
    minikube service ingress-nginx-controller -n ingress-nginx --url
    
    # 실행 결과 (첫번째 주소 사용)
    http://127.0.0.1:49732
    http://127.0.0.1:49733
    ❗  darwin 에서 Docker 드라이버를 사용하고 있기 때문에, 터미널을 열어야 실행할 수 있습니다
    
    # 테스트 주소 접속
    curl -I http://127.0.0.1:49732/healthz
    
    # 실행 결과
    HTTP/1.1 200 OK
    Date: Tue, 23 Jul 2024 14:23:59 GMT
    Content-Type: text/html
    Content-Length: 0
    Connection: keep-alive
    ```