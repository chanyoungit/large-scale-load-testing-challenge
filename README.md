# KTB: 대규모 부하 테스트 대회

## 팀원 구성 및 역할

<table>
  <tr>
    <th>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 박찬영 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</th>
    <th>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 한주리 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</th>
    <th>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 김민제 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</th>
    <th>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 홍진호 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</th>
  </tr>
  <tr>
    <td>- Back-end <br> - DevOps</td>
    <td>- Front-end</td>
    <td>- Back-end</td>
    <td>- Cloud</td>
  </tr>
</table>                                                                                                                                

<br>

# 📝 목차 <a name = "index"></a>

- [대회설명](#idea)
- [아키텍처](#structure)
- [기술사용이유](#test)
- [결과](#outputs)
- [회고](#review)

<br>

# 🚩 대회설명 <a name = "idea"></a>
### 대규모 서비스 개발 및 운영을 위한 실전 엔지니어링 대회

<br>


## 기본 설명
### KakaoTechBootcamp AI Chat 서비스 소개

<img width="1176" alt="스크린샷 2024-12-17 오후 4 35 14" src="https://github.com/user-attachments/assets/f8c060ae-6ce5-4bf5-8362-3b21d2216270" />

#### 기존에 만들어져 있는 서비스를 각 팀들에게 제공하고,해당 서비스를 최적화하여 최대한의 부하를 견딜 수 있도록 설계

<img width="435" alt="스크린샷 2024-12-17 오후 3 04 30" src="https://github.com/user-attachments/assets/8d3b661a-fc21-4bb4-8e55-2d5d61f543b4" /><br>
- #### 각 팀에 대하여 부하테스트를 진행 후 보다 많은 접속자에게 서비스를 문제 없이 제공한 팀이 승리 <br>

<br>

## 진행기간
24/12/09 ~ 24/12/13(총 5일)

<br>

## 진행 방식(부하 테스트)
- 토너먼트 형태로 대회 진행
- 동시 접속자 수 증가 (변경될 수도 있음)
  - 초기 100명에서 시작하여 매 30초마다 100명씩 증가
  - 최대 3,000명까지 순차적 증가 목표
  - 각 사용자당 최소 1개 이상의 WebSocket 연결 유지
- 제한된 리소스로 최대한의 효율 달성
  - 최대 30대의 t3.small 인스턴스 사용
  - 다양한 아키텍쳐 시도
  - 사용 가능한 AWS 권한
    - EC2
    - Elastic Load Balancing
    - Auto Scaling
    - Route 53
    - ACM
    - S3
    - CloudFront
    - CloudWatch

<br>

# ☁️ 아키텍처 <a name = "structure"></a>

![카부캠AWS부하테스트 drawio (1)](https://github.com/user-attachments/assets/ccbc53d9-062b-4f85-b05a-befa49e3871c)
- 주어진 도메인에 CloudFront와 서브 도메인에 ELB 연결
- Front-end
  - CloudFront와 S3 버킷을 활용하여 호스팅
- Back-end
  - 인스턴스 총 20개 생성
  - 오토스케일링을 활용해 각 인스턴스의 개수가 일정하게 유지되도록 설정
  - 각 인스턴스에 도커 컨테이너를 5개씩 띄우고 각각의 애플리케이션에 로드벨런싱
  - 총 100개의 서비스에 로드밸런싱
  - 파일 서비스와 같은 경우는 S3 버킷에 업로드하고 메타데이터를 레디스에 저장하는 방식
- DB
  - MongoDB로 구축되어 있던 서비스를 Redis로 변경
  - Redis Cluster을 통한 정합성 보장

<br>

# :question: 기술사용이유 <a name = "test"></a>

### Cloud
- 인스턴스의 성능과 개수에 제한이 있음
- 최대한 부가 기능은 인스턴스를 사용하지 않고 AWS의 서비스로 대체
  - 프론트 호스팅을 S3+CloudFront(CDN) 활용
  - 부하 테스트의 최대 요청 수를 2~3만 단위로 설정 -> CloudFront에서 처리 가능한지 확인 후 도입
  - 로드밸런싱 : Nginx 대신 ELB 사용
- AutoScailig Group
  - 인스턴스 탬플릿 제작 후 등록
  - 최대 30개 인스턴스 제한, 20개 이상 늘어나지 않도록 정책 설정
- ELB
  - 각 인스턴스에 20개의 컨테이너를 배포하여 총 100개의 서비스 로드 밸런싱
  - 4개의 타깃 그룹을 제작 후 각 포트 변경으로 대응
- CloudWatch
  - 부하가 발생 시 병목 지점을 파악하고 조치할 수 있도록 각 인스턴스에 CloudAgent 설치하여 메트릭 수집
  - 대시보드를 제작히여 대회 진행 중에 지속적 모니터링
  - 인스턴스뿐만 아니라 ELB 등 서비스도 모니터링
- CloudFront
  - 클라우드 프론트를 활용한 프론트 호스팅
  - S3 버킷 연결
  - S3에서 에러를 반환 시 index.html 라우팅 설정
- Route 53
  - 주어진 도메인을 기반으로 서브도메인 설정 및 라우팅 설정

### Back
- Message Queue
  - Kafka와 RabbitMQ룰 밗
    - RabbitMQ는 Push 모델 기반으로 설계되어 Kafka보다 레이턴시가 낮고 메세지 전달 보장이 되어 소켓 통신에 유리 
  - 그러나 RabbitMQ를 사용할 경우, 대량의 메세지와 WebSocket 연결에 따라 병목 현상 우려
    - 최대 3000명까지 WebSocket 연결이 이루어지고, 메세지가 지속적으로 송수신되어 RabbitMQ 큐에 쌓임
    - 큐에 쌓인 메세지가 많아질수록 소비자가 메세지를 가져오는 시간이 지연될 수 있음
    - 실시간 통신 시스템에서 문제가 발생할 수 있어 RabbitMQ 사용을 지양
  - 최종적으로 메시지큐 대신 Redis Cluster을 통한 샤딩으로 대체
- Redis Cluster
  - t3.small은 싱글 코어이므로 한 서버 내에 여러 Redis를 두는 것은 그리 효율적이지 않을 것이라 생각
  - 저장 공간
    - 영어는 1바이트, 한글은 3바이트로 가정했을 때 한 채팅의 메시지나 유저의 토큰 값 등을 저장하는 하나의 Key 당 100바이트를 사용한다고 가정
    - 인스턴스의 RAM은 2GB이지만 OS, 프로세스 등의 사용을 고려하여 Redis가 사용할 수 있는 공간은 1.5GB로 가정
    - 데이터 개수는 1500만개 저장 가능, 넉넉하게 잡아도 500 ~ 1000만개를 저장 가능하다는 결론 도출
    - 부하 테스트에서 1000만 개를 넘길 일은 없을 것으로 예상, 데이터 저장 개수는 클러스터 개수 선정 기준에서 제외
  - 작업 처리 효율
    - 저장 공간을 제외하면 클러스터의 개수 선정 시 중요한 기준은 Redis의 Single Thread 작업 처리 효율이라고 생각
    - 마스터 5 + 레플리카 5로 구성
    - 마스터 노드 5대로 구성하여 각 노드에 대한 해싱은 백엔드 서버에서 하고 마스터 노드는 작업 처리만 담당
    - 레플리카 노드는 Read 작업 수행
    - 부하 테스트 대회이기 때문에 레플리카 노드의 데이터 정합성은 고려하지 않음
  - 고가용성
    - 마스터 노드마다 레플리카 노드를 하나씩 두어 failover 대비
    - 클러스터 모드에서 각 노드가 서로를 감시하며 HA 보장
- 파일 시스템
  - 디스크에 저장하던 파일 시스템을 S3 버킷에 저장으로 변경
  - Redis에서 메타파일 저장, 필요할 때 프론트에 URL 전달

<br>

# 🏆 결과물 <a name = "outputs"></a>
<p align="center">
<img width="435" alt="스크린샷" src="https://github.com/user-attachments/assets/ac423575-bc6a-4985-b59a-c863bd068f28" />
<img width="435" alt="스크린샷2" src="https://github.com/user-attachments/assets/e6beda3a-ec28-40d2-a2fe-c1dccf304162" /><br>
</p>

- 대상 수상
  - 카카오 대표이사 상장 수여  
- 실시간 3400명의 부하 테스트 중 150개 실패
  - 수상팀(2,3등)에 비해 약 400~500개 정도 더 많은 테스트 통과

<br>

# 🤔 회고 <a name = "review"></a>
- t3.small이 코어가 하나인 줄 알았지만 두개였음 (확신하지말고 다음부턴 꼭 확인하기..)
  - 하나의 서버에 Redis를 두개를 실행했으면 더 좋았을 것
- 백엔드 서버에 도커 컨테이너를 추가로 5개 배포하여 처리량을 늘리려 했으나, t3.small 인스턴스의 CPU가 2개뿐이라 성능 향상이 미미했음
  - JS 코드를 Java로 리팩토링하고 쓰레드 풀을 조정하여 멀티 쓰레드의 이점을 활용하거나 Node.js의 이벤트 루프와 워커 스레드를 통해 멀티코어를 활용했다면 좋았을 것
- S3 파일 업로드 로직을 백엔드에서 구현했으나, 프론트엔드에서 처리하면 더 효율적일 것으로 판단됨
- 무작정 메세지 큐를 배제하기보다는 어느 정도 구현한 후 테스트해보고, 요청 처리가 잘 안되면 그때 메세지 큐를 고려
