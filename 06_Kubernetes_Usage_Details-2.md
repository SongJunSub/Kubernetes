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