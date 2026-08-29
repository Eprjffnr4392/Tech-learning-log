# TCP / UDP

## TCP 세그먼트 구조

TCP는 전송 계층에서 사용하는 프로토콜로, 데이터를 **세그먼트(Segment)** 단위로 전송한다.

TCP 세그먼트는 크게 **헤더(Header)**와 **데이터(Data)**로 구성된다.

```text
┌─────────────────────────────────────────────────────────────┐
│ Source Port              │ Destination Port                │
├─────────────────────────────────────────────────────────────┤
│ Sequence Number                                             │
├─────────────────────────────────────────────────────────────┤
│ Acknowledgment Number                                       │
├───────┬───────┬───────────┬────────────────────────────────┤
│ HLEN  │ Res.  │  Flags    │ Window                         │
├─────────────────────────────────────────────────────────────┤
│ Checksum                  │ Urgent Pointer                  │
├─────────────────────────────────────────────────────────────┤
│ Options + Padding (선택 사항)                               │
├─────────────────────────────────────────────────────────────┤
│ Data                                                        │
└─────────────────────────────────────────────────────────────┘
```

### 1) Source Port / Destination Port

* **Source Port**: 송신 측 애플리케이션의 포트 번호
* **Destination Port**: 수신 측 애플리케이션의 포트 번호
* 어떤 애플리케이션 간에 통신하는지 구분하는 데 사용

---

### 2) Sequence Number

TCP에서 전송되는 데이터의 **순서를 나타내는 번호**

* 데이터가 순서대로 전달되었는지 확인하는 데 사용
* 데이터가 손실되거나 순서가 뒤바뀌었을 경우 이를 확인하고 재전송할 수 있음
* TCP 연결을 시작할 때 사용하는 초기 시퀀스 번호(ISN, Initial Sequence Number)는 일반적으로 임의의 값으로 설정

예:

```text
Sequence Number = 1000
데이터 = 500바이트

다음 데이터의 Sequence Number = 1500
```

---

### 3) Acknowledgment Number

수신한 데이터에 대한 **다음에 받기를 기대하는 Sequence Number**

```text
송신 측
Sequence Number = 1000
Data = 500바이트
        ↓
수신 측
ACK Number = 1500
```

즉, ACK Number가 `1500`이라는 것은

> "Sequence Number 1000부터 시작하는 500바이트를 정상적으로 받았고, 다음에는 1500번부터 보내줘."

라는 의미이다.

TCP는 Sequence Number와 Acknowledgment Number를 이용하여 데이터가 정상적으로 전달되었는지 확인한다.

---

### 4) 헤더 길이 (Header Length / Data Offset)

TCP 헤더의 길이를 **4바이트 단위의 워드(Word) 개수**로 나타낸다.

* 최소값: `5`
* 최대값: `15`
* 실제 크기:

  * `5 × 4 = 20바이트`
  * `15 × 4 = 60바이트`

따라서 TCP 헤더의 크기는 **20~60바이트**이다.

Options가 없는 경우 기본적으로 **20바이트**이다.

```text
Data Offset = 5

5 × 4 = 20바이트
```

---

### 5) Reserved

현재 사용하지 않는 **예약된 영역**

* 미래의 확장이나 새로운 기능을 위해 예약된 영역
* 일반적인 TCP 통신에서는 사용하지 않음

---

### 6) 플래그 (Flags)

TCP 연결의 상태나 제어 정보를 나타낸다.

#### SYN (Synchronize)

TCP 연결을 설정할 때 사용한다.

* 연결을 시작할 때 사용
* 초기 Sequence Number(ISN)를 전달
* 3-Way Handshake에서 사용

```text
Client → Server
SYN
Sequence Number = 1000
```

---

#### ACK (Acknowledgment)

수신한 데이터에 대한 확인 응답을 나타낸다.

* Acknowledgment Number가 유효함을 나타냄
* TCP 통신에서 데이터를 정상적으로 받았음을 확인하는 데 사용

```text
ACK Number = 1500
```

이는 다음에 `1500`번 Sequence Number의 데이터를 받기를 기대한다는 의미이다.

---

#### RST (Reset)

TCP 연결을 **즉시 재설정하거나 중단**할 때 사용한다.

* 존재하지 않는 연결에 접근하는 경우
* 비정상적인 연결 상태
* 연결을 더 이상 유지할 수 없는 경우

등에 사용될 수 있다.

FIN과 달리 **정상적인 연결 종료 과정이 아니라 강제적인 연결 종료 또는 재설정**에 가깝다.

---

#### FIN (Finish)

TCP 연결을 **정상적으로 종료**할 때 사용한다.

* 더 이상 보낼 데이터가 없음을 나타냄
* TCP 연결 종료 과정에서 사용
* 일반적으로 4-Way Handshake에서 사용

---

#### PSH (Push)

수신 측의 TCP 버퍼에 데이터가 쌓이기를 기다리지 않고 **애플리케이션으로 전달하도록 요청**하는 데 사용한다.

대화형 통신처럼 데이터를 빠르게 전달해야 하는 경우와 관련이 있다.

예:

```text
메신저에서 메시지를 입력
        ↓
Enter 입력
        ↓
데이터 전송
```

단, 실제 애플리케이션의 메시지 전송이 항상 PSH 플래그 하나로 결정되는 것은 아니며, 운영체제의 TCP 구현에 따라 동작이 달라질 수 있다.

---

#### URG (Urgent)

Urgent Pointer 필드가 유효함을 나타낸다.

* 긴급하게 처리해야 하는 데이터가 있을 경우 사용
* Urgent Pointer와 함께 사용

현대의 일반적인 애플리케이션에서는 자주 사용되지 않는다.

---

### 7) Window (Receive Window)

TCP에서 **수신 측이 한 번에 받아들일 수 있는 데이터의 양**을 나타낸다.

즉, 수신 측의 TCP 버퍼에 여유 공간이 얼마나 있는지를 송신 측에 알려주는 역할을 한다.

```text
수신 측
Window = 5000

        ↓

송신 측
"현재 5000바이트까지 추가로 보내도 된다."
```

Window 필드는 기본적으로 **16비트**이므로 표현할 수 있는 최대값은 다음과 같다.

```text
2^16 - 1 = 65,535
```

따라서 기본 Window 필드만 사용할 경우 최대 **65,535바이트**를 표현할 수 있다.

다만 TCP Window Scale 옵션을 사용하면 이보다 더 큰 수신 윈도우를 사용할 수 있다.

이러한 방식으로 TCP는 수신 측의 처리 속도를 고려하여 송신 속도를 조절할 수 있으며, 이를 **흐름 제어(Flow Control)**라고 한다.

---

### 8) Checksum

TCP 세그먼트가 전송 과정에서 손상되었는지 확인하기 위한 값이다.

Checksum을 계산할 때 TCP 헤더와 데이터뿐만 아니라 IP 계층의 일부 정보가 포함된 **Pseudo Header**도 함께 사용한다.

```text
Checksum 계산 대상

Pseudo Header
      +
TCP Header
      +
TCP Data
      +
Padding
```

Pseudo Header에는 일반적으로 다음과 같은 정보가 포함된다.

```text
Source IP Address
Destination IP Address
Protocol
TCP Length
```

Pseudo Header는 실제 TCP 세그먼트에 포함되어 전송되는 TCP 헤더가 아니라, **Checksum 계산을 위해 논리적으로 사용하는 정보**이다.

---

### 9) Urgent Pointer

URG 플래그가 설정된 경우 긴급 데이터의 위치를 나타내는 데 사용한다.

```text
URG = 1
        ↓
Urgent Pointer 사용
```

일반적인 TCP 통신에서는 거의 사용되지 않는다.

---

### 10) Options / Padding

TCP의 추가 기능을 사용할 때 사용한다.

대표적인 옵션:

* MSS (Maximum Segment Size)
* Window Scale
* SACK (Selective Acknowledgment)
* Timestamp

Options가 추가되면 TCP 헤더의 크기가 증가한다.

TCP 헤더의 크기는 최소 **20바이트**, Options를 포함하면 최대 **60바이트**이다.

Padding은 TCP 헤더 길이가 32비트 단위가 되도록 맞추기 위해 사용된다.

---

## TCP의 Sequence Number와 ACK Number

TCP는 Sequence Number와 Acknowledgment Number를 이용하여 데이터의 순서와 정상적인 수신 여부를 확인한다.

```text
Client
Sequence Number = 1000
Data = 500바이트
        │
        ▼
Server
ACK Number = 1500
```

ACK Number가 `1500`이라는 것은 `1000~1499`까지의 데이터를 정상적으로 수신했으며, 다음으로 `1500`번 데이터를 기대한다는 의미이다.

따라서 TCP에서는 단순히 패킷 자체만 확인하는 것이 아니라 **Sequence Number를 기반으로 데이터의 순서와 손실 여부를 확인**한다.

---

# 3-Way Handshake

TCP는 연결 지향(Connection-oriented) 프로토콜이기 때문에 데이터를 전송하기 전에 먼저 연결을 설정한다.

TCP 연결 설정에는 일반적으로 **3-Way Handshake**가 사용된다.

```text
Client                                      Server
   │                                           │
   │  SYN                                      │
   │  Seq = 1000                               │
   │ ─────────────────────────────────────────>│
   │                                           │
   │  SYN + ACK                                │
   │  Seq = 5000                               │
   │  Ack = 1001                               │
   │ <─────────────────────────────────────────│
   │                                           │
   │  ACK                                      │
   │  Seq = 1001                               │
   │  Ack = 5001                               │
   │ ─────────────────────────────────────────>│
   │                                           │
   │              연결 완료                    │
```

### 1단계: Client → Server

클라이언트가 `SYN` 플래그를 설정하여 연결을 요청한다.

```text
SYN = 1
Sequence Number = 1000
```

이때 클라이언트는 자신의 초기 Sequence Number(ISN)를 서버에 전달한다.

---

### 2단계: Server → Client

서버는 클라이언트의 SYN을 확인하고 **SYN + ACK**를 전송한다.

```text
SYN = 1
ACK = 1

Sequence Number = 5000
Acknowledgment Number = 1001
```

`Acknowledgment Number = 1001`인 이유는 SYN 자체가 Sequence Number를 **1만큼 소비하기 때문**이다.

```text
Client SYN
Seq = 1000

1000 + 1 = 1001
```

동시에 서버 자신의 초기 Sequence Number인 `5000`도 전달한다.

---

### 3단계: Client → Server

클라이언트는 서버의 SYN을 확인하고 `ACK`를 전송한다.

```text
ACK = 1

Sequence Number = 1001
Acknowledgment Number = 5001
```

서버의 SYN Sequence Number가 `5000`이므로

```text
5000 + 1 = 5001
```

이 된다.

이 과정을 통해 양쪽에서 서로의 초기 Sequence Number를 확인하고 TCP 연결을 설정한다.

---

## TCP 데이터 전송과 ACK

TCP 연결이 설정된 이후에는 Sequence Number와 ACK Number를 이용하여 데이터를 주고받는다.

```text
Client                                      Server
   │                                           │
   │ Seq = 1001                                │
   │ Data = 500 bytes                          │
   │ ─────────────────────────────────────────>│
   │                                           │
   │                         ACK = 1501        │
   │ <─────────────────────────────────────────│
   │                                           │
```

클라이언트가 500바이트를 전송했으므로 서버는 다음에 받을 Sequence Number인 `1501`을 ACK로 전달한다.

---

## TCP 재전송

TCP는 데이터가 정상적으로 전달되지 않은 경우 **재전송(Retransmission)**을 수행하여 신뢰성 있는 통신을 제공한다.

예:

```text
Client                                      Server
   │                                           │
   │ Seq = 1001                                │
   │ Data = 500 bytes                          │
   │ ─────────────────────────X                │
   │              데이터 손실                  │
   │                                           │
   │        ACK가 도착하지 않음                │
   │                                           │
   │ 재전송                                    │
   │ ─────────────────────────────────────────>│
```

송신 측이 일정 시간 동안 ACK를 받지 못하면 해당 데이터를 다시 전송할 수 있다.

또한 TCP는 **중복 ACK** 등을 이용하여 패킷 손실을 판단하고 재전송하는 방식도 사용한다.

이러한 기능을 통해 TCP는 데이터의 신뢰성 있는 전달을 지원한다.

---

# 4-Way Handshake

TCP 연결을 정상적으로 종료할 때는 일반적으로 **4-Way Handshake**가 사용된다.

TCP는 양방향 통신이므로 연결 종료도 각각의 방향을 독립적으로 종료할 수 있다.

```text
Client                                      Server
   │                                           │
   │ FIN                                       │
   │ ─────────────────────────────────────────>│
   │                                           │
   │ ACK                                       │
   │ <─────────────────────────────────────────│
   │                                           │
   │                         FIN               │
   │ <─────────────────────────────────────────│
   │                                           │
   │ ACK                                       │
   │ ─────────────────────────────────────────>│
   │                                           │
   │              연결 종료                    │
```

### 1단계

클라이언트가 더 이상 전송할 데이터가 없음을 나타내기 위해 `FIN`을 전송한다.

### 2단계

서버는 클라이언트의 FIN을 확인하고 `ACK`를 전송한다.

이후 서버는 아직 클라이언트에게 전송할 데이터가 있다면 계속 전송할 수 있다.

### 3단계

서버가 모든 데이터를 전송한 후 `FIN`을 전송한다.

### 4단계

클라이언트가 서버의 FIN을 확인하고 `ACK`를 전송한다.

이 과정을 통해 양쪽의 TCP 연결이 정상적으로 종료된다.

---

# TCP의 주요 특징

TCP는 다음과 같은 기능을 제공한다.

* 연결 지향(Connection-oriented)
* 신뢰성 있는 데이터 전송
* 데이터 순서 보장
* 데이터 손실 시 재전송
* 흐름 제어(Flow Control)
* 혼잡 제어(Congestion Control)
* 오류 검출
* 3-Way Handshake를 통한 연결 설정
* 4-Way Handshake를 통한 정상적인 연결 종료

---

# UDP

UDP(User Datagram Protocol)는 TCP와 달리 **비연결형(Connectionless)** 프로토콜이다.

TCP처럼 연결을 설정하기 위한 3-Way Handshake 과정이 없으며, 데이터 전달의 신뢰성을 TCP가 보장하는 방식으로 제공하지 않는다.

UDP 데이터 단위는 **Datagram**이라고 한다.

## UDP 헤더 구조

UDP 헤더는 TCP보다 단순하며 크기가 **8바이트**로 고정되어 있다.

```text
┌─────────────────────────────────────────────────────────────┐
│ Source Port              │ Destination Port                │
├─────────────────────────────────────────────────────────────┤
│ Length                   │ Checksum                         │
├─────────────────────────────────────────────────────────────┤
│ Data                                                        │
└─────────────────────────────────────────────────────────────┘
```

### 1) Source Port

송신 측 애플리케이션의 포트 번호

### 2) Destination Port

수신 측 애플리케이션의 포트 번호

### 3) Length

UDP 헤더와 데이터를 포함한 전체 길이를 나타낸다.

최소 크기는 UDP 헤더만 존재하는 경우인 **8바이트**이다.

### 4) Checksum

UDP 데이터그램의 오류를 검출하기 위한 값이다.

TCP와 마찬가지로 IP 계층의 일부 정보가 포함된 Pseudo Header를 사용하여 Checksum을 계산한다.

---

# TCP와 UDP 비교

| 구분      | TCP             | UDP           |
| ------- | --------------- | ------------- |
| 연결 방식   | 연결 지향           | 비연결형          |
| 연결 설정   | 3-Way Handshake | 없음            |
| 연결 종료   | 4-Way Handshake | 없음            |
| 신뢰성     | 보장              | 보장하지 않음       |
| 데이터 순서  | 보장              | 보장하지 않음       |
| 재전송     | 지원              | 기본적으로 지원하지 않음 |
| 흐름 제어   | 지원              | 없음            |
| 혼잡 제어   | 지원              | 없음            |
| 헤더 크기   | 20~60바이트        | 8바이트          |
| 데이터 단위  | Segment         | Datagram      |
| 속도/오버헤드 | 상대적으로 큼         | 상대적으로 작음      |
| 주요 특징   | 신뢰성 있는 통신       | 빠르고 단순한 통신    |

## TCP를 사용하는 대표적인 경우

* HTTP / HTTPS
* SSH
* FTP
* SMTP
* 데이터베이스 통신 등

TCP는 데이터가 정확한 순서로 전달되어야 하고 손실된 데이터를 재전송해야 하는 경우 적합하다.

## UDP를 사용하는 대표적인 경우

* DNS
* DHCP
* 실시간 음성/영상 통신
* 온라인 게임
* 스트리밍 등

UDP는 일부 데이터가 손실되더라도 **빠른 전송과 낮은 지연시간**이 중요한 경우 적합하다.

# QUIC

QUIC(Quick UDP Internet Connections)은 **UDP를 기반으로 설계된 전송 계층 프로토콜**

TCP의 신뢰성 있는 데이터 전송 기능과 TLS의 보안 기능을 결합하면서, TCP에서 발생하는 연결 설정 및 데이터 전송의 지연을 줄이도록 설계되었다.

현재 HTTP/3는 QUIC을 전송 프로토콜로 사용한다.

## QUIC의 특징

### 1) UDP 기반

QUIC은 TCP가 아닌 **UDP를 기반으로 동작**한다.

```text
기존 HTTPS

HTTP
 ↓
TLS
 ↓
TCP
 ↓
IP
```

```text
HTTP/3

HTTP/3
 ↓
QUIC
 ↓
UDP
 ↓
IP
```

UDP 자체는 신뢰성 있는 데이터 전송을 제공하지 않지만, QUIC이 UDP 위에서 다음과 같은 기능을 구현한다.

* 데이터의 순서 보장
* 손실된 데이터의 재전송
* 흐름 제어
* 혼잡 제어
* 연결 관리
* TLS를 이용한 암호화

따라서 QUIC은 **UDP의 단순한 전송 방식 위에 TCP와 유사한 신뢰성 기능을 구현한 프로토콜**이라고 이해할 수 있다.

---

### 2) TLS 1.3 기본 적용

QUIC은 **TLS 1.3을 기본적으로 사용**하여 통신을 암호화한다.

TCP에서는 일반적으로 다음과 같은 구조를 사용한다.

```text
TCP 연결 설정
    ↓
TLS Handshake
    ↓
암호화된 데이터 통신
```

반면 QUIC은 TLS가 QUIC 연결 설정 과정에 통합되어 있다.

```text
QUIC + TLS 1.3
        ↓
암호화된 통신
```

이를 통해 연결 설정 과정에서 발생하는 지연을 줄일 수 있다.

---

### 3) 빠른 연결 설정

TCP는 일반적으로 데이터를 전송하기 전에 **3-Way Handshake**를 수행하고, HTTPS에서는 이후 TLS Handshake 과정도 필요하다.

```text
TCP

Client → Server : SYN
Client ← Server : SYN + ACK
Client → Server : ACK

        ↓

TLS Handshake

        ↓

데이터 전송
```

QUIC은 QUIC과 TLS 1.3을 통합하여 연결 설정에 필요한 왕복 횟수를 줄일 수 있다.

따라서 TCP + TLS 기반의 HTTPS보다 **연결 설정에 필요한 지연시간을 줄이는 데 유리하다.**

---

### 4) Stream 기반 통신

QUIC은 하나의 연결(Connection) 안에서 여러 개의 **Stream**을 사용할 수 있다.

```text
QUIC Connection
│
├── Stream 1
│   └── 데이터
│
├── Stream 2
│   └── 데이터
│
└── Stream 3
    └── 데이터
```

각 Stream은 독립적으로 데이터를 전달할 수 있다.

이러한 구조를 통해 TCP에서 발생하는 **Head-of-Line Blocking(HOL Blocking)** 문제를 완화할 수 있다.

---

### 5) Head-of-Line Blocking 완화

TCP는 하나의 연결에서 데이터를 순서대로 처리하기 때문에 특정 데이터가 손실되면 뒤에 도착한 데이터도 해당 데이터가 복구될 때까지 기다려야 한다.

```text
TCP

Packet 1 → 정상 수신
Packet 2 → 손실
Packet 3 → 수신
Packet 4 → 수신

Packet 2가 복구될 때까지
Packet 3, Packet 4 처리 지연
```

QUIC은 하나의 연결 안에서 여러 Stream을 사용할 수 있기 때문에 특정 Stream에서 패킷 손실이 발생하더라도 **다른 Stream의 데이터 처리를 계속할 수 있다.**

```text
QUIC

Stream 1 → 데이터 손실
Stream 2 → 정상 처리
Stream 3 → 정상 처리
```

따라서 TCP의 연결 단위 Head-of-Line Blocking 문제를 완화할 수 있다.

단, 하나의 QUIC Stream 내부에서는 데이터 순서가 보장되므로 같은 Stream에서 발생한 손실은 해당 Stream의 데이터 처리에 영향을 줄 수 있다.

---

### 6) Connection Migration

QUIC은 **Connection ID**를 이용하여 연결을 식별할 수 있다.

TCP 연결은 일반적으로 다음과 같은 IP와 Port의 조합에 크게 의존한다.

```text
Client IP
+
Client Port
+
Server IP
+
Server Port
```

따라서 클라이언트의 네트워크가 변경되면 기존 TCP 연결을 유지하기 어려울 수 있다.

예:

```text
Wi-Fi
  ↓
LTE / 5G
```

QUIC은 Connection ID를 사용하기 때문에 네트워크 환경이 변경되어 IP 주소가 변경되더라도 **기존 연결을 유지하면서 통신을 계속할 수 있도록 설계되어 있다.**

이를 **Connection Migration**이라고 한다.

---

## QUIC의 구조

QUIC은 UDP 위에서 동작하며, QUIC 패킷 내부에 여러 종류의 프레임을 포함할 수 있다.

```text
┌─────────────────────────────────────────┐
│ UDP Header                              │
├─────────────────────────────────────────┤
│ QUIC Header                             │
├─────────────────────────────────────────┤
│ QUIC Packet                             │
│                                         │
│ ┌─────────────────────────────────────┐ │
│ │ Frame                               │ │
│ │ - STREAM                            │ │
│ │ - ACK                               │ │
│ │ - CRYPTO                            │ │
│ │ - CONNECTION_CLOSE                  │ │
│ │ - 기타                              │ │
│ └─────────────────────────────────────┘ │
└─────────────────────────────────────────┘
```

QUIC에서는 데이터 전송, ACK, 연결 관리 등의 정보를 **Frame** 단위로 처리한다.

---

## TCP / UDP / QUIC 비교

| 구분                   | TCP                     | UDP                 | QUIC                 |
| -------------------- | ----------------------- | ------------------- | -------------------- |
| 기반 프로토콜              | IP 위에서 직접 동작            | IP 위에서 직접 동작        | UDP 위에서 동작           |
| 연결 방식                | 연결 지향                   | 비연결형                | 연결 지향                |
| 연결 설정                | 3-Way Handshake         | 없음                  | QUIC + TLS Handshake |
| 암호화                  | 기본 제공하지 않음              | 기본 제공하지 않음          | TLS 1.3 기본 사용        |
| 신뢰성                  | 지원                      | 기본적으로 미지원           | 지원                   |
| 재전송                  | 지원                      | 기본적으로 미지원           | 지원                   |
| 순서 보장                | 지원                      | 기본적으로 미지원           | Stream별 지원           |
| 흐름 제어                | 지원                      | 없음                  | 지원                   |
| 혼잡 제어                | 지원                      | 없음                  | 지원                   |
| 다중 Stream            | 없음                      | 없음                  | 지원                   |
| HOL Blocking         | 연결 단위로 발생 가능            | 해당 개념이 상대적으로 다름     | Stream 단위로 완화        |
| Connection Migration | 어려움                     | 별도 구현 필요            | 지원                   |
| 주요 사용                | HTTP/1.1, HTTP/2, SSH 등 | DNS, DHCP, 실시간 통신 등 | HTTP/3               |

## QUIC과 HTTP/3

HTTP/3는 QUIC을 전송 프로토콜로 사용한다.

```text
HTTP/1.1
    ↓
TCP
    ↓
IP
```

```text
HTTP/2
    ↓
TCP
    ↓
IP
```

```text
HTTP/3
    ↓
QUIC
    ↓
UDP
    ↓
IP
```

따라서 HTTP/3에서는 TCP 대신 QUIC을 사용하여 **빠른 연결 설정, TLS 1.3 기반 암호화, Stream 기반 통신, Connection Migration** 등의 장점을 활용할 수 있다.

---

## 정리

QUIC은 단순히 **UDP를 사용하는 TCP**라고 이해하기보다는,

> **UDP를 기반으로 TCP의 신뢰성·흐름 제어·혼잡 제어 기능 등을 구현하고, TLS 1.3과 Stream 및 Connection Migration 등의 기능을 결합한 전송 프로토콜**

이라고 이해하는 것이 좋다.

```text
TCP
→ 신뢰성은 높지만 연결 설정과 HOL Blocking 등의 단점 존재

UDP
→ 빠르고 단순하지만 신뢰성 및 연결 관리 기능이 부족

QUIC
→ UDP 기반
→ 신뢰성 있는 데이터 전송
→ TLS 1.3 기본 사용
→ 다중 Stream
→ Connection Migration
→ HTTP/3의 전송 프로토콜
```

