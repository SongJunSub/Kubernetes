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
