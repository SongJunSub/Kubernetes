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
