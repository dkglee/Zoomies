# Zoomies

## <mark>프로젝트 설명</mark>
직업별로 지정된 동물을 사냥해 점수를 얻고, 다른 플레이어의 행동을 관찰해 정체를 추리하는 **소셜 추론형 TPS**입니다.  
대규모 동물 개체와 다수 플레이어가 동시에 상호작용하는 환경을 목표로 **저지연·고안정 네트워크 통신 모듈**을 설계·구현했습니다.

### 대표 영상

---

## <mark>역할 및 목표</mark>

**역할**
- 네트워크 통신 모듈 설계/구현(클라이언트)
- 멀티 스레드 패킷 파이프라인 및 동기화 구조
- 패킷 포맷/직렬화(Protocol Buffers), Steam 네트워킹 연동

**목표**
- 대량 객체 처리 환경에서 **낮은 지연**과 **높은 안정성**
- **Boss–Worker 멀티 스레드**와 **Double Buffering**으로 Lock 최소화
- <b>캐시 정렬(64B)</b>과 데이터 레이아웃 개선으로 스루풋 향상
- **Steam Datagram Relay(SDR)** + **PollGroup IO Multiplexing**으로 회선 품질 편차 대응

---

## 주요 기능

### <mark>01. 멀티 스레드 통신 관리</mark>
패킷 입출력을 분리한 파이프라인으로 병목과 캐시 충돌을 완화합니다.
- **Boss–Worker 모델**: IO 작업 분산, 메인 스레드와 독립 처리
- **Double Buffering**: Front/Back 버퍼 스왑으로 **Mutex Lock 1회**로 수집/소비 전환
- **64B 정렬**: 캐시 라인 경계에 맞춘 구조체/큐 배치로 캐시 일관성 문제(MESI) 완화

**아키텍쳐**
<img width="990" height="291" alt="image" src="https://github.com/user-attachments/assets/9ab47cc2-749e-467b-8e7e-9ec3ecdb1687" />

**설명**
- 읽기 시 스왑 후 **Lock 없이** 읽기 버퍼 접근(즉시 소비), 레이스는 스왑 단일 구간에서만 관리
- 버퍼 포인터 스왑으로 동기화 비용을 줄였습니다.

**소스 코드**  

[네트워크 IO Worker 스레드 소스 코드 바로가기](https://github.com/dkglee/Zoomies/blob/dev/proj_a/Source/proj_a/Network/NetworkWorker/Private/NetworkWorker.cpp) <br>
[Double Buffer를 활용한 Race Condition 관리 소스 코드 바로가기](https://github.com/dkglee/Zoomies/blob/dev/proj_a/Source/proj_a/Utility/Private/DoubleBuffer.cpp) <br>
[캐시 최적화 메모리 정렬 소스 코드 바로가기](https://github.com/dkglee/Zoomies/blob/dev/proj_a/Source/proj_a/Utility/Public/DoubleBuffer.h#L28C1-L29C49)

---

### <mark>02. 소켓 통신 모듈</mark>
다양한 이벤트·세션을 효율적으로 관리합니다.
- **Protocol Buffers** 기반 경량 패킷(필드 확장 용이)
- **Steam Datagram Relay(SDR)** 사용으로 NAT/품질 이슈 대응
- **Steam PollGroup** 기반 **IO Multiplexing(Non-blocking, Event-driven)**

**아키텍쳐**  
<img width="611" height="315" alt="image" src="https://github.com/user-attachments/assets/c068d678-98ab-4c37-b388-018e27f18308" />

**설명**
- 레벨 트리거 이벤트로 **불필요한 폴링 비용** 감소, 컨텍스트 스위치 최소화

**소스 코드**  

[SDR 기반 멀티플렉싱 소스 코드 바로가기](https://github.com/dkglee/Zoomies/blob/dev/proj_a/Source/proj_a/Network/SocketInterface/Private/SteamSocket.cpp#L12C1-L71C2) <br>
[Protobuf 패킷 바로가기](https://github.com/dkglee/Zoomies/blob/dev/proj_a/Source/proj_a/Protobuf/Proto_file/message.proto)

---

## <mark>성과/메모</mark>
- **스팀 출시**  
- 멀티 스레드 + 캐시 최적화로 **Lock 지연/캐시 충돌 감소**, 대량 객체 처리 시 **네트워크 부하 절감**을 확인
