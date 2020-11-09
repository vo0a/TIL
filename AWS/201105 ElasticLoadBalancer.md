## 201105 ElasticLoadBalancer

`201109 수정`

<img width="834" alt="스크린샷 2020-11-04 오후 5 02 41" src="https://user-images.githubusercontent.com/44199159/98096623-50f18080-1ecf-11eb-9365-f5bed65dcf8f.png">

[Mastering Chaos - A Netflix Guide to Microservices](https://youtu.be/CZ3wIuvmHeM)



## Load Balancing?? Load Balancer?? 
- **로드 밸런싱**은 쏟아지는 **트래픽을 여러 대의 서버로 분산해주는 기술**, 여러 대의 서버를 두고 서비스를 제공하는 분산 처리 시스템에서 필요한 기술

- **로드 밸런서**는 **서버에 가해지는 부하를 분산 해주는 장치**, 클라이언트와 서버풀 사이에 위치하며, 한대의 서버로 부하가 집중되지 않도록 트래픽을 관리해 각각의 서버가 최적의 퍼포먼스를 보일 수 있도록 한다. 

- 만약 **로드밸런싱을 쓰지 않으면** 각 EC2 인스턴스에 **고정된 아이피를 부여**해야 하는데, 문제는 하나의 **인스턴스에 하나의 도메인만 연결할 수 밖에 없다**. 이 경우 서버에 트래픽이 몰린다면 서버의 사양을 올리는 스케일 업 또는 서버의 개수를 늘리는 스케일 아웃을 고려해야한다. 

  - 스케일업은 인스턴스를 업데이트 하는 동안 서비스를 할 수 없고, 

  - 스케일 아웃의 경우 서버가 늘어날 때마다 도메인이 새로 필요하다. 

    → 로드 밸런싱이 필요한 이유



## L4(Layer 4) & L7(Layer 7) load balancing
> OSI 7 Layer에서 각각의 계층이 L1/L2/L3‥‥L7에 해당하고, 상위 계층에서 사용되는 장비는 하위 계층의 장비가 갖는 기능을 모두 가지고 있으며, 상위 계층으로 갈수록 더욱 정교한 로드밸런싱이 가능

[L2 , L3 , L4 , L7 스위치 구분 및 기본적인 설명](https://klero.tistory.com/entry/L2-L3-L4-L7-%EC%8A%A4%EC%9C%84%EC%B9%98-%EA%B5%AC%EB%B6%84-%EB%B0%8F-%EA%B8%B0%EB%B3%B8%EC%A0%81%EC%9D%B8-%EC%84%A4%EB%AA%85)

### L4 load balancer

- 네트워크 계층(IP, IPX)이나 **트랜스포트 계층(TCP, UDP)의 정보를 바탕으로 트래픽 분산**, IP 주소나 포트번호, MAC주소, 전송 프로토콜에 따라 트래픽을 나누는 것이 가능

### L7 load balancer
- 어플리케이선 계층(HTTP, FTP, SMTP)에서 로드를 분산하기 때문에 **HTTP 헤더, 쿠키** 등과 같은 **사용자 요청을 기준으로 특정 서버에 트래픽을 분산**하는 것이 가능
- 즉 패킷의 내용을 확인하고 그 내용에 따라 트래픽을 특정 서버에 분배하는 것이 가능
- 예를 들어 URL에 따라 부하를 분산시키거나, HTTP 헤더의 쿠키 값에 따라 부하를 분산하는 등 클라이언트 요청을 보다 세분화해서 서버에 전달한다. 
- 또한 L7 로드밸런서의 경우 **특정한 패턴을 지닌 바이러스를 감지해 네트워크를 보호**할 수 있으며, DoS/DDoS와 같은 비정상적인 트래픽을 필터링할 수 있어 네트워크 보안 분야에서도 활용된다. 

<img width="809" alt="스크린샷 2020-11-04 오후 7 37 59" src="https://user-images.githubusercontent.com/44199159/98101231-4639ea00-1ed5-11eb-9f7d-57d821d3dfb8.png">



## AWS Elastic Load Balancer

AWS에는 

- Classic Load Balancer (CLB)
- Application Load Balancer (ALB)
- Network Load Balancer (NLB)
    3개의 로드밸런서가 있고, 여기서 ALB, NLB에 대해서 알아보도록 하겠다.

![스크린샷 2020-11-04 오후 8 24 10](https://user-images.githubusercontent.com/44199159/98106250-88b2f500-1edc-11eb-83a5-709ac63749f8.png)



### Application Load Balancer(ALB)
<img width="766" alt="스크린샷 2020-11-04 오후 8 26 51" src="https://user-images.githubusercontent.com/44199159/98106040-2954e500-1edc-11eb-8349-56ac62d8bfa7.png">

- Reverse Proxy 대로 Client IP와 서버 사이에 들어오고 나가는 트래픽이 모두 로드 밸런서와 통신 
  - Reverse [Proxy](https://firework-ham.tistory.com/23)란,
    - Reverse Proxy는 클라이언트로부터의 요청을 받아서(필요하다면 주위에서 처리한 후) 적절한 웹 서버로 요청을 전송한다. 웹 서버는 요청을 받아서 평소처럼 처리를 하지만, 응답은 클라이언트로 보내지 않고 Reverse Proxy로 반환한다. 요청을 받은 Reverse Proxy는 그 응답을 클라이언트로 반환한다.
    - 통상의 프락시 서버는 LAN → WAN의 요청을 대리로 수행하지만 
      - 서버에게 클라이언트가 누구인지 감추는 역할
      - 서버가 응답받은 IP는 포워드 프록시 서버의 IP이기 때문에 클라이언트가 누군지 알 수 없다.
    - Reverse Proxy는 WAN → LAN의 요청을 대리한다.
      - 리버스 프록시는 서버가 누구인지 감추는 역할
      - 클라이언트는 리버스 프록시 서버를 먼저 호출하게 되기 때문에 실제 서버의 IP를 알 수 없다.
    - 클라이언트로부터의 요청이 웹서버로 전달되는 도중의 처리에 끼어들어서 다양한 전후처리를 시행할 수가 있게 된다.
- Security Group을 통한 보안이 가능 
- Client → Load Balancer의 Access 제한 기능
- ALB는 IP 주소가 변동되기 때문에 Client에서 Access 할 ELB의 DNS Name을 이용해야 한다. 
- Name Server 또는 Route53에서 CNAME을 사용해야 Domain Name 연동이 가능
- 패스나 포트 등에 따라 다른 대상 그룹으로 맵핑이 가능, 이러한 특징으로 도커 컨테이너 환경에서 아주 유용 또한 대상을 EC2 인스턴스, 람다, IP로도 연결 가능. 특정한 요청에 대해서는 서버 없이 직접 응답 메세지를 작성할 수 있기 때문에 마이크로아키텍처를 구성하기에 좋다. 



### Network Load Balancer(NLB)

<img width="769" alt="스크린샷 2020-11-04 오후 8 28 08" src="https://user-images.githubusercontent.com/44199159/98106103-45f11d00-1edc-11eb-838e-175dcd06dab3.png">

- Client IP와 서버 사이에 서버로 들어오는 트래픽은 Load balancer를 통하고 나가는 트래픽은 Client IP와 직접 통신한다. 
-  NLB는 Security Grop 적용이 되지 않아서 서버에 적용된 Security Group에서 보안이 가능
-  Client → Server에서 Access 제한 가능 
-  NLB는 할당한 Elastic IP로 사용이 가능하여 DNS Name과 IP 주소 모두 사용이 가능 
-  Name Server 또는 Route 53에서 A Record 사용 가능 



## 실습

> Load Balancer 로는 80포트로 요청, 스프링 부트에서는 8080포트로 요청을 보낸다.

### EC2 인스턴스 환경설정

#### 1.  git 설치 
- `sudo apt-get install git`

#### 2. Java11 설치 
- `sudo add-apt-repository ppa:openjdk-r/ppa`
- `sudo apt install openjdk-11-jdk`
- `java -version`

#### 3. SpringBoot 빌드 파일 설치 및 실행 
- `git clone https://github.com/Jeoungseungho/springboot.git`
- springboot 디렉토리로 이동 후 `java -jar hello-spring-0.0.1-SNAPSHOT.jar`

 

### ALB

1. `인스턴스 1`을 위의 방법으로 생성

   - 보안그룹 80, 8080, 22 포트 열어줌.

2. 스냅샷 찍어서 `인스턴스 1`와 환경이 같은 `인스턴스 2`를 생성

   - Image and templates → 이미지 생성
   - 이미지 - AMI - 생성한 이미지 선택 후 `시작하기`

3. 로드밸런서 생성

   - ALB 선택
   - 리스너 HTTP : 80
   - **대상 그룹** 설정
     - 대상 그룹 생성
     - 타겟 타입은 인스턴스
     - `인스턴스 1`와 `인스턴스 2`를 8080 포트로 연결
       - 혹시나 다른 포트로 연결되었다면, Deregister 누르고 Register targets로 다시 연결

4. 여기까지 모든 연결이 끝난 후 Load Balancer DNS주소로 접속

5. 회원 생성 후 회원 목록 확인

   - 새로고침을 누를 때마다 인스턴스가 바뀌는것을 알 수 있다.

   - 현재는 스프링부트가 인메모리에 회원 목록을 저장하기 때문에 인스턴스가 바뀌면서 내용이 다르게 보임을 알 수 있다.
   - 이렇게 로드 밸런싱이 잘 되는 것도 확인을 마쳤다.

