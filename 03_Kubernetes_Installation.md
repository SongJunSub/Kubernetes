# 쿠버네티스 설치

**YAML**

- Kubernetes 사용에서 중요한 건 아키텍처, 그 다음이 **YAML**이다.
- “Blog를 만들 때 App 서버 3대, DB 서버 1대, 도메인은 blog.example.com으로 연결해달라.”를 쿠버네티스에 요청하려면 1. 어떤 오브젝트를 사용할지 정하고 2. 상세 설정을 YAML 형식으로 정의해야 한다.
- 들여쓰기 (Indent)
    - 들여쓰기는 기본적으로 2칸 또는 4칸을 지원하지만, 2칸 들여쓰기를 추천한다.
- 데이터 정의 (Map)
    - 데이터는 `Key: Value` 형식으로 정의한다.
- 배열 정의 (Array)
    - 배열은 `-`로 표시한다.
- 주석 (Comment)
    - 주석은 `#`으로 표시한다.
- 참/거짓
    - 참 거짓은 `true`, `false` 외에 `yes`, `no`를 지원하며, true, false는 대소문자를 가리지 않는다.
- 숫자
    - 정수 또는 실수를 따옴표(”) 없이 사용하면 숫자로 인식한다.
- 줄바꿈 (newline)
    - 여러 줄을 표현하는 방법이다.
    - `|`  지시어는 마지막 줄바꿈이 포함된다.
        
        ```yaml
        newlines_sample: |
        	number one line
        	
        	second line
        	
        	last line
        ```
        
        - JSON 형식으로 변환하면 다음과 같다.
        
        ```json
        {
        	"newlines_sample": "number one line\n\nsecond line\n\nlast line\n"
        }
        ```
        
    - `|-` 지시어는 마지막 줄바꿈을 제외한다.
        
        ```yaml
        newlines_sample: |-
        	number one line
        	
        	second line
        	
        	last line
        ```
        
        - JSON 형식으로 변환하면 다음과 같다.
        
        ```json
        {
        	"newlines_sample": "number one line\n\nsecond line\n\nlast line"
        }
        ```
        
    - `>` 지시어는 중간에 들어간 빈줄을 제외한다.
        
        ```yaml
        newlines_sample: >
        	number one line
        	
        	second line
        	
        	last line
        ```
        
        - JSON 형식으로 변환하면 다음과 같다.
        
        ```json
        {
        	"newlines_sample": "number one line\nsecond line\nlast line\n"
        }
        ```
        
- 주의 사항
    - 띄어쓰기
        - Key와 Value 사이에는 반드시 빈칸이 필요하다.
    - 문자열 따옴표
        - 대부분의 문자열을 따옴표 없이 사용할 수 있지만 `:` 가 들어간 경우는 반드시 따옴표가 필요하다.
- 참고 사이트
    - https://www.json2yaml.com
        - JSON 형식의 파일을 YAML 파일로 변환해주는 사이트이다.
    - https://yamllint.com
        - YAML 형식의 문법에 오류가 있는지 체크해주는 사이트이다.

**Kubernetes 설치 (MacOS) 개발 환경 vs 운영 환경**

- 쿠버네티스를 운영 환경에 설치하기 위해서는 최소 3대의 마스터 서버와 컨테이너 배포를 위한 n개의 노드 서버가 필요하다.
- 이러한 설치는 과정이 복잡하고 배포 환경(AWS, Google Cloud, Azure, Bare Metal 등)에 따라 방법이 다르기 때문에 처음 공부할 때 바포 구축하기는 적합하지 않다.
- 따라서, 개발 환경을 위해 마스터와 노드를 하나의 서버에 설치하여 손쉽게 관리하는 방법으로 사용할 것이다.
- 대표적인 개발 환경 구축 방법으로 minikube, k3s, docker for desktop, kind 등이 있는데, minikube를 사용할 것이다.
- 대부분의 환경에서 사용할 수 있고 간편하며, 무료인 minikube를 추천하지만 설치할 수 없거나 사양이 낮은 경우에는 저렴한 비용으로 테스트할 수 있는(1,000원 이하) k3s를 추천한다.
- **주의**
    - 개발 환경과 운영 환경의 가장 큰 차이점은 개발 환경은 단일 노드로 여러 노드에 스케줄링하는 테스트가 어렵고 LoadBalancer와 Persistent Local Storage 또한 가상으로 만들어야 한다. 이러한 사용을 정확하게 하려면 운영 환경(멀티 노드)에서 진행해야 한다.

**minikube**

- 쿠버네티스 클러스터를 실행하려면 최소한 Scheduler, Controller, API Server, Etcd, Kubelet, Kube Proxy를 설치해야 하고, 필요에 따라 DNS, Ingress Controller, Storage Class 등을 설치해야 한다.
- minikube는 Windows, MacOS, Linux에서 사용할 수 있으며 다양한 가상 환경(Hyperkit, Hyper-V, Docker, VirtualBox 등)을 지원하여 대부분의 환경에서 문제없이 동작한다.
- 명령어 (MacOS)
    - 설치
        - `brew install minikube`
    - 버전 확인
        - `minikube version`
    - 가상 머신 시작 (Docker가 실행 중이여야 한다.)
        - `minikube start --driver=docker`
    - 상태 확인
        - `minikube status`
    - 정지
        - `minikube stop`
    - 시작
        - `minikube start`
    - 삭제
        - `minikube delete`
    - SSH 접속
        - `minikube ssh`
    - IP 확인
        - `minikube ip`
    - 특정 k8s 버전 실행
        - `minikube start -—kubernetes-version=v1.30.0`
