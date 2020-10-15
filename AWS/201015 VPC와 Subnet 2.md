## 201015 VPC와 Subnet 2

> 같은 VPC 내에 두 가지(public/private) 인스턴스가 있다. 실습에서는 public 인스턴스에서 private 인스턴스로 접속을 해본다.

[민태님 설명](https://www.notion.so/ausgga/2020-10-15-VPC-Subnet-b06ae4626dc3449ab8a12c4c3e77fbc9)



## 이론

VPC 에 이어서 설명

여러 인스턴스를 격리하기 위함

- VPN = 보안이 포인트
  - 모든 네트워크를 감청할 수 있기 때문에 만약에 써야된다면 유료를 쓰세요..

- 프록시 = 중개서버
  - 프록시 캐시
  - 공개 프록시
  - 포워드 프록시(일반적인 프록시)
    - 요청자의 ip를 모름. 
  - 리버스 프록시 - Nginx
    - 프록시 서버가 다른 서버로 중개를 해준다.
    - 포트! 잘 쓰면 443
    - 부하가 걸리는데 리버스 프록시 서버에서는 
    - 리버스 돌려줄 때는 웹서버에서는 80포트로  
    - 서버도메인 따라서 할당도 되고,  LB도 이런 역할을 함. 
    - OSI 7 레이어에서 어플리케이션 단에 가깝다.
    - 스위치, LB는 더 상위계층 
    - 계층에따라 사용이 나뉠 뿐 포워드는 전통적이다.
- NAT
  - NAT를 이용하는 이유는 대개 [사설 네트워크](https://ko.wikipedia.org/wiki/사설_네트워크)에 속한 여러 개의 호스트가 하나의 공인 IP 주소를 사용하여 [인터넷](https://ko.wikipedia.org/wiki/인터넷)에 접속하기 위함이다.
  - 중개 최상단 - Gateway가 NAT역할을 함
  - 게이트웨이 단에서 돌아감
  - 1:1  거의 사용 안함 → 1:N 
  - NAT 인스턴스 → NAT 게이트웨이
  - 게이트웨이가 어떻게 범위를 지정하냐에 따라 할 수 있는 역할이 다르다.
- Bastion Host
  - Public과 Private 서버를 이어주는 중간 통로의 서버



### 참고자료

[프록시 VPN 차이](https://m.blog.naver.com/PostView.nhn?blogId=reductionist101&logNo=221567693949&proxyReferer=https:%2F%2Fwww.google.com%2F)

[가상 데이터 센터 만들기 - VPC 기본 및 연결 옵션](https://www.youtube.com/watch?v=R1UWYQYTPKo)





## 실습

### 1. AWS setting

이전 시간 실습했던 내용과 거의 유사

1. VPC생성
   - VPN - 173.32.0.0/16
2. 서브넷 생성
   1. public
      - 173.32.0.0/17
      - 가용영역 a
   2. private
      - 173.32.128.0/16
      - 가용영역 c (왜인지는 모르겠지만 서울리전인 경우에 a, c로 하라고 들음)
3. 인터넷 게이트웨이 생성
   - VPC 연결
4. **NAT 게이트웨이 생성**
   - **public 서브넷**과 연결
   - 탄력적 IP 할당 클릭
5. 라우팅 테이블 생성
   - public
     - 서브넷 연결 - public 서브넷과 연결
     - 라우팅 편집 - Internet Gateway `0.0.0.0/0` 추가
   - private
     - 서브넷 연결 - private 서브넷과 연결
     - 라우팅 편집 - **NAT Gateway** `0.0.0.0/0` 추가
       - 이 부분을 해야 인터넷 연결이 됨.
6. EC2 인스턴스 생성
   - public
     - public 서브넷과 연결
     - 퍼블릭 IP 할당
     - 보안그룹 ssh 22포트 열어두기
   - private
     - private 서브넷과 연결
     - 퍼블릭 IP 할당받지 않아도 됨
     - 보안그룹 ssh 22포트 열어두기
     - public 과 동일한 pem키 사용



### 2. 접속하기

1. public 인스턴스 SSH 접속하기

2. private 인스턴스 접속

   1. `touch a.pem` 
      - pem 키 생성
   2. `vi a.pem`
      - 편집기가 뜨면, 인스턴스 생성시 생성했던 pem 키 내용을 복사하여 붙여넣기
      - `:wq` or `Shift + ZZ` 로 저장 후 종료
   3. `ssh -i "a.pem" ubuntu@private-인스턴스-프라이빗-주소`
      - 퍼블릭 IP를 할당받았더라도, 퍼블릭 IP를 쓰면 접속 실패
      - 반드시 프라이빗 IPv4 주소로 연결할 것 

3. 인터넷 접속이 되는지 확인

   - `curl https://www.naver.com`

   - 아래와 같이 나온다면 성공!
     

     ![image](https://user-images.githubusercontent.com/44438366/96143828-775e7480-0f3e-11eb-95cb-6fbb106a69bf.png)



### 3. 코멘트

NAT를 생성해서 NAT 게이트웨이를 연결하면, 인터넷 통신이 NAT 한테 간다. 그러면 그 주소변환기가 인터넷 요청을 보고 소스 IP를 NAT IP로 바꿔서 public 인스턴스(서버)에게 전달, public 서버는 다시 NAT IP로 응답을 주는데 그거를 NAT가 다시 private 으로바꿔서 요청자한테 넘겨주는 것이다.



틀린 부분이 있다면, 이슈로 남겨주세요.😀