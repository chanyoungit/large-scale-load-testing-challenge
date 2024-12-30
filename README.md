# KTB: 대규모 부하 테스트 대회

## 팀원 구성 및 역할

| <img width="122" alt="image" src="https://github.com/user-attachments/assets/c397adf8-767d-496d-b4d2-78be435704a1"> | <img width="122" alt="image" src="https://github.com/user-attachments/assets/3d885f38-52ad-44f5-b9ac-1bfa30c4b0f5"> | <img width="122" alt="image" src="https://github.com/user-attachments/assets/5af0ddb4-3cd0-4c71-a2f8-f7a7ff7c693e"> | <img width="122" alt="image" src="https://github.com/user-attachments/assets/8e5d9eeb-5049-42da-b6af-7b257db6f311"> | <img width="122" alt="image" src="https://github.com/user-attachments/assets/e78047d7-22d7-47a7-a34f-0eeb3283f05b"> | <img width="122" alt="image" src="https://github.com/user-attachments/assets/5f92b7ab-f887-4def-8266-e5c5c797c6f8"> |
|------------------------------------------------------|--------------------------------------------------|--------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------|
| [Jinho.Hong(홍진호)](https://github.com/mynameisjinhohong)                                            | [Ayaan.Park(박찬영)](https://github.com/chanyoungit)                                           | [Jully.Han(한주리)](https://github.com/Hanjuri)                                           | [Milo.Kim(김민제)](https://github.com/alswp006)                                                                                                                                               | [Winter.Park(박현혜)](https://github.com/hyeonhye126)                                                                                                                                                |[Dorothy.Kim(김도윤)](https://github.com/Do-yoon)                                           |
| 팀장<br>클라우드| 클라우드| 풀스택| 풀스택| AI     | AI                                   |


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

- 기존에 만들어져 있는 서비스를 각 팀들에게 제공하고,해당 서비스를 최적화하여 최대한의 부하를 견딜 수 있도록 설계

<img width="435" alt="스크린샷 2024-12-17 오후 3 04 30" src="https://github.com/user-attachments/assets/8d3b661a-fc21-4bb4-8e55-2d5d61f543b4" /><br>
- 토너먼트 형태로 대회 진행 <br>
- 각 팀에 대하여 부하테스트를 진행 후 보다 많은 접속자에게 서비스를 문제 없이 제공한 팀이 승리<br>

<br>

## 진행기간
24/12/09 ~ 24/12/13(총 5일)

<br>

## 진행 방식(부하 테스트)
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
- 프론트
  - 클라우드 프론트와 S3 버킷을 활용하여 호스팅
- 백엔드
  - 인스턴스 총 20개 생성
  - 오토스케일러를 활용해 각 인스턴스의 개수가 일정하게 유지되도록 설정
  - 각 인스턴스에 도커 컨테이너를 5개씩 띄우고 각각의 애플리케이션에 로드벨런싱
  - 총 100개의 서비스에 로드밸런싱
  - 파일 서비스와 같은 경우는 S3 버킷에 업로드하고 메타데이터를 레디스에 저장하는 방식
- DB
  - MongoDB로 구축되어 있던 서비스를 Redis로 변경
  - Redis Cluster을 통한 정합성 보장

<br>

# :question: 기술사용이유 <a name = "test"></a>

### Cloud
- 인스턴스의 성능과 개수 제한이 존재함
- 최대한 부가기능들은 인스턴스를 띄우지 않고 AWS의 서비스로 대체
  - 프론트 호스팅을 S3+CloudFront(CDN)을 활용
  - 부하 테스트의 최대 요청 수를 2~3만 단위로 잡고, 클라우드 프론트에서 감당 가능한지 확인 후 도입
  - 마찬가지로 로드밸런싱도 Nginx를 사용하지 않고 ELB 사용
- AutoScailig Group
  - 인스턴스 탬플릿 제작 후 등록
  - 최대 30개 인스턴스 제한이 있었기에 20개 이상 늘어나서 실격패 되지 않도록 정책 설정
- ELB
  - 각 인스턴스에 20개의 컨테이너를 띄워서 총 100개의 서비스 로드 밸런싱
  - 총 4개의 타깃 그룹을 제작 후 각각 연결 포트 변경으로 대응
- CloudWatch
  - 부하가 걸렸을 때, 병목지점을 파악하고 조치할 수 있도록 각 인스턴스에 CloudAgent 설치하여 메트릭 수집
  - 대시보드를 제작하여 대회진행 중에 지속적 모니터링
  - 각 인스턴스 뿐만 아닌 ELB 등등의 서비스들도 모니터링
- CloudFront
  - 클라우드 프론트를 활용한 프론트 호스팅
  - S3버킷을 연결
  - 만약 S3에서 에러를 반환할 경우 index.html 로 라우팅 되도록 설정
- Route 53
  - 주어진 도메인을 기반으로, 서브도메인을 설정하여 라우팅 설정

### Back
- 대용량 트래픽 처리를 해야 하기에 메세지 큐를 사용할까 논의
  - Kafka vs RabbitMQ 고려
    - RabbitMQ는 Push 모델 기반 설계이기 때문에 Kafka보단 레이턴시가 낮으며 메세지 전달 보장을 하기 떄문에 소켓 통신에선 RabbitMQ가 유리 
  - 하지만 RabbitMQ를 활용했을 때, t3.small 이라는 성능 한계에 의해서 병목지점이 될까 우려
    - 최대 3000명까지 WebSocket 연결이 이루어지고 메세지가 지속적으로 송수신되어 메세지가 RabbitMQ 큐에 쌓임
    - 큐에 쌓인 메세지가 많아질수록 소비자가 해당 메세지를 가져갈 때까지의 시간이 발생
    -> 실시간 통신 시스템에서 큰 문제가 우려가 되기 때문에 RabbitMQ는 사용안하기로 결정
  - 최종적으로 메시지큐를 활용하지않고 Redis Cluster을 통한 샤딩으로 대체
- Redis Cluster
  - t3.small은 싱글 코어기 때문에 한 서버 내에 여러 Redis를 두는 것은 그리 효율적이지 않을 것이라 생각
  - 저장 공간
    - 영어는 1바이트, 한글은 3바이트 정도로 두었을 때 한 채팅의 메시지나 유저의 토큰 값 등을 저장하는데 하나의 Key 당 100바이트를 사용한다고 가정
    - 인스턴스의 RAM은 2GB이지만 OS, 프로세스 등의 사용 공간을 가정하여 Redis가 사용할 수 있는 공간은 1.5GB라고 가정하여 데이터의 갯수를 1500만개 저장할 수 있고 넉넉하게 잡아도 500 ~ 1000만개를 저장할 수 있다는 결론을 도출
    -> 부하 테스트에서 아무리 작업을 많이 해도 1000만개를 넘길 일은 없을 것이라 생각하였고 데이터 저장 갯수는 클러스터 갯수 선정 기준에서 제외
  - 작업 처리 효율
    - 저장 공간을 제외하면 클러스터의 갯수를 선정하는 기준 중 가장 중요한 것은 Single Thread인 Redis에서의 작업 처리 효율이라고 생각하였습니다.
    -> 마스터 5 + 레플리카 5로 구성
    - 마스터 노드 5대로 구성하여 각 노드에 대한 해싱은 백엔드 서버에서 하고 마스터 노드는 작업 처리만 담당
    - 레플리카 노드는 Read 작업 수행
    - 부하 테스트 대회이기 때문에 레플리카 노드의 데이터 정합성은 고려 x
  - 고가용성
    - 마스터 노드 당 레플리카 노드를 한 개씩 두어 failover 발생 대비
    - 클러스터 모드에서 각 노드가 서로를 감시하며 HA를 보장
- 파일 시스템
  - 디스크에 저장하던 파일 시스템을 S3 버킷에 저장하도록 변경
  - Redis에서 메타파일을 저장하고 필요할 때 프론트에 URL 전달

<br>

# 🏆 결과물 <a name = "outputs"></a>
<p align="center">
<img width="435" alt="스크린샷" src="https://github.com/user-attachments/assets/ac423575-bc6a-4985-b59a-c863bd068f28" />
<img width="435" alt="스크린샷2" src="https://github.com/user-attachments/assets/e6beda3a-ec28-40d2-a2fe-c1dccf304162" /><br>
</p>

- 대상 수상
  - 카카오 대표이사 상장 수여
  - 빕스 30만원 상품권  
- 최종 3400명의 부하 테스트 중 150개 실패
  - 이외 수상팀(2,3등)에 비해서 약 400~500개 정도 테스트를 더 많이 통과
  - 이외에도 이벤트 전으로 우승자에게 도전으로 몇몇 팀들과 대결후 전부 압도적 승리

<br>

# 🤔 회고 <a name = "review"></a>
- 백엔드 서버 하나에 도커 컨테이너를 추가로 배포하여 처리량을 늘리려 했으나, t3.small 인스턴스의 CPU가 1개뿐이라 성능 향상이 미미했음
  - JS 코드를 Java로 리팩토링하고 쓰레드 풀을 조정하여 멀티 쓰레드의 이점을 활용했다면 더 좋았을 것
- S3 파일 업로드 로직을 백엔드에서 구현했으나, 프론트엔드에서 처리하면 더 효율적일 듯
