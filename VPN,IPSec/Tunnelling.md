````markdown
# 터널링의 이해

## 터널링(Tunneling)

- 터널링(Tunneling): 하나의 네트워크 프로토콜 데이터를 다른 프로토콜의 패킷 내부에 캡슐화(Encapsulation)하여 네트워크를 통해 전달하는 기술
- 인터넷과 같은 공용 네트워크 위에 가상의 통신 경로인 터널(Tunnel)을 생성하여 데이터를 전달한다.
- 서로 떨어진 두 호스트 또는 두 네트워크를 논리적으로 연결할 때 사용한다.
- VPN(Virtual Private Network), 원격 접속, 서로 다른 프로토콜 간 데이터 전달 등에 사용된다.
- 터널링 자체가 반드시 암호화를 의미하는 것은 아니다.
  - GRE, L2TP 등은 자체적인 데이터 암호화 기능을 제공하지 않는다.
  - 보안이 필요한 경우 IPsec 등의 암호화 기술과 함께 사용할 수 있다.

### 터널링의 기본 동작

1) 원본 패킷 생성
   - 실제로 전달하려는 데이터를 포함한 원본 패킷을 생성한다.

2) 캡슐화(Encapsulation)
   - 원본 패킷 전체를 다른 프로토콜의 데이터 영역에 포함시킨다.
   - 새로운 헤더를 추가하여 터널을 통해 전달할 수 있도록 한다.

3) 터널을 통한 전송
   - 캡슐화된 패킷이 인터넷 또는 중간 네트워크를 통해 터널의 반대편 끝점까지 전달된다.

4) 역캡슐화(Decapsulation)
   - 터널의 끝점에서 추가된 헤더를 제거한다.
   - 원래의 패킷을 복원하여 최종 목적지로 전달한다.

### 터널링의 특징

- 두 노드 또는 두 네트워크 사이에 논리적인 가상 링크를 형성할 수 있다.
- 서로 다른 네트워크 프로토콜을 다른 프로토콜 내부에 캡슐화하여 전달할 수 있다.
- VPN을 구성할 때 많이 사용된다.
- 원래 패킷을 다른 패킷 내부에 캡슐화하므로 중간 네트워크에서는 내부 패킷의 구조를 직접 확인하기 어려울 수 있다.
- 암호화 기술과 결합하면 데이터의 기밀성과 무결성을 보호할 수 있다.
- 터널링 자체는 암호화를 보장하지 않으므로 프로토콜별 보안 기능을 구분해야 한다.

---

# 터널링 방식

터널링은 전달하려는 데이터와 동작 계층에 따라 여러 방식으로 구분할 수 있다.

## 1) 2계층 기반 터널링 방식

- 데이터 링크 계층의 프레임이나 PPP 데이터를 터널링하는 방식
- 대표적인 프로토콜
  - PPTP
  - L2TP
  - L2TPv3

### PPTP

- PPTP(Point-to-Point Tunneling Protocol)
- PPP(Point-to-Point Protocol) 패킷을 IP 네트워크를 통해 전달하기 위한 터널링 프로토콜
- Microsoft 등이 참여한 벤더 컨소시엄에서 개발하였다.
- RFC 2637에 정의되어 있다.
- GRE(Generic Routing Encapsulation)를 확장한 방식으로 PPP 패킷을 전달한다.
- PPTP 자체의 GRE 터널은 암호화 기능을 제공하지 않는다.
- 기존에는 PPP 인증 및 MPPE(Microsoft Point-to-Point Encryption)와 함께 사용되었다.
- 현재는 보안 취약성으로 인해 VPN 환경에서 사용하는 것을 권장하지 않는다.
- 현대 VPN에서는 IPsec, SSL/TLS 기반 VPN, WireGuard 등의 방식이 주로 사용된다.

### L2TP

- L2TP(Layer Two Tunneling Protocol)
- PPP 패킷을 IP 네트워크를 통해 터널링하기 위한 프로토콜
- PPTP와 Cisco의 L2F(Layer 2 Forwarding) 기술을 기반으로 개발되었다.
- RFC 2661에 정의되어 있다.
- L2TP 자체는 암호화 기능을 제공하지 않는다.
- 보안이 필요한 경우 일반적으로 IPsec과 결합하여 사용한다.

```text
L2TP + IPsec
```

- L2TP는 터널링을 담당하고 IPsec은 암호화, 인증, 무결성 보호 등을 담당한다.

### L2TPv3

- L2TP Version 3
- RFC 3931에 정의되어 있다.
- PPP뿐만 아니라 Ethernet, Frame Relay 등 다양한 2계층 데이터를 IP 네트워크를 통해 전달할 수 있다.
- 자체적인 암호화 기능은 제공하지 않는다.
- 필요한 경우 IPsec과 함께 사용하여 보안을 강화할 수 있다.

---

## MPLS

- MPLS(Multiprotocol Label Switching)
- 패킷의 IP 주소만을 이용하여 라우팅하는 대신 Label을 이용하여 데이터를 전달하는 기술
- OSI 계층으로 명확하게 하나의 계층에 속하지 않으며 일반적으로 2계층과 3계층 사이에서 동작한다고 설명한다.
  - 흔히 Layer 2.5라고 표현한다.
- 통신 사업자 네트워크에서 VPN, 트래픽 엔지니어링 등에 사용된다.
- MPLS 자체가 일반적인 의미의 암호화 터널링 프로토콜은 아니다.
- MPLS VPN을 사용하여 서로 떨어진 사설 네트워크를 논리적으로 연결할 수 있다.

---

# 3계층 기반 터널링 방식

- 네트워크 계층에서 IP 패킷 등을 캡슐화하여 전달하는 방식
- 대표적인 기술
  - GRE
  - IPsec

## GRE

- GRE(Generic Routing Encapsulation)
- RFC 2784에 정의되어 있다.
- 하나의 네트워크 계층 프로토콜 패킷을 다른 네트워크 계층 프로토콜 내부에 캡슐화하는 기술
- 다양한 프로토콜을 IP 네트워크를 통해 전달할 수 있다.
- 멀티캐스트 및 다양한 네트워크 프로토콜을 터널링하는 데 활용할 수 있다.
- GRE 자체는 암호화 기능이나 강력한 인증 기능을 제공하지 않는다.

```text
GRE = 터널링 O
      암호화 X
```

- 데이터 보호가 필요한 경우 GRE와 IPsec을 함께 사용할 수 있다.

```text
GRE + IPsec
```

- GRE
  - 터널 생성 및 다양한 프로토콜 캡슐화 담당

- IPsec
  - 암호화
  - 인증
  - 무결성 보호 담당

## IPsec

- IPsec(Internet Protocol Security)
- IP 계층에서 데이터의 보안을 제공하는 기술 집합
- IP 패킷을 암호화하고 인증하여 안전하게 전달한다.
- VPN 구축에 널리 사용된다.

### IPsec의 주요 보안 기능

- 기밀성(Confidentiality)
  - 데이터를 암호화하여 제3자가 내용을 확인하지 못하도록 한다.

- 무결성(Integrity)
  - 데이터가 전송 과정에서 변조되었는지 확인한다.

- 인증(Authentication)
  - 통신 상대방의 신원을 확인한다.

- 재전송 공격 방지(Anti-Replay)
  - 공격자가 기존 패킷을 다시 전송하는 공격을 방지한다.

### IPsec의 동작 모드

#### 전송 모드(Transport Mode)

- 원본 IP 패킷의 Payload 부분을 보호한다.
- 일반적으로 호스트 간 통신에서 사용한다.

```text
[IP Header][Encrypted Payload]
```

#### 터널 모드(Tunnel Mode)

- 원본 IP 패킷 전체를 보호한다.
- 원본 패킷 전체를 새로운 IP 패킷 내부에 캡슐화한다.
- VPN Gateway 간 연결 등에 많이 사용된다.

```text
[New IP Header][Encrypted Original IP Packet]
```

---

# 상위 계층 기반 터널링 방식

- TCP 연결이나 애플리케이션 세션을 이용하여 터널을 형성하는 방식
- 대표적인 기술
  - TLS
  - SSH
  - SOCKS

> SSL/TLS, SSH, SOCKS를 단순히 모두 "4계층 프로토콜"이라고 분류하는 것은 정확하지 않다.
> 이들은 TCP 등의 전송 계층 위에서 동작하며 애플리케이션의 통신을 보호하거나 중계하는 방식으로 보는 것이 더 정확하다.

## SSL/TLS

- SSL(Secure Sockets Layer) / TLS(Transport Layer Security)
- TCP 기반 애플리케이션 통신에 암호화와 인증 기능을 제공한다.
- SSL은 과거 기술이며 현재는 TLS가 사용된다.
- HTTPS, SSL VPN 등의 보안 통신에 사용된다.

### 주요 기능

- 데이터 암호화
- 데이터 무결성 보호
- 서버 인증
- 필요에 따라 클라이언트 인증

### SSL/TLS VPN

- 웹 브라우저 또는 전용 VPN 클라이언트를 통해 VPN에 접속하는 방식
- HTTPS와 동일하게 TLS 기반의 암호화 통신을 사용할 수 있다.
- 원격 근무나 사내 시스템 접속 등에 사용된다.

---

## SSH

- SSH(Secure Shell)
- 원격 시스템에 안전하게 접속하기 위한 프로토콜
- TCP 22번 포트를 기본적으로 사용한다.
- 통신 내용을 암호화한다.
- 원격 명령 실행뿐만 아니라 터널링 기능도 제공한다.

### SSH 터널링

- Local Port Forwarding
- Remote Port Forwarding
- Dynamic Port Forwarding

예:

```text
Client → SSH Tunnel → SSH Server → Destination Server
```

- SSH 터널을 통해 일반 TCP 애플리케이션의 트래픽을 암호화하여 전달할 수 있다.

---

## SOCKS

- SOCKS는 클라이언트와 서버 사이에서 네트워크 연결을 중계하는 프록시 프로토콜이다.
- SOCKS5가 널리 사용된다.
- TCP 연결 및 UDP 전달을 지원할 수 있다.
- 애플리케이션의 트래픽을 프록시 서버를 통해 전달한다.

```text
Client → SOCKS Proxy → Destination Server
```

- SOCKS 자체는 일반적인 VPN처럼 모든 네트워크 트래픽을 자동으로 암호화하지 않는다.
- SOCKS를 SSH Dynamic Port Forwarding과 함께 사용하면 암호화된 프록시 터널을 구성할 수 있다.

---

# VPN

## VPN(Virtual Private Network)

- 인터넷과 같은 공용 네트워크를 이용하여 논리적인 사설 네트워크를 구성하는 기술
- 서로 떨어진 사용자 또는 네트워크를 가상의 사설망으로 연결한다.
- 터널링과 암호화 기술을 이용하여 데이터를 안전하게 전달할 수 있다.

```text
사용자 또는 사설망
        ↓
   암호화 / 캡슐화
        ↓
========================
     Internet
     VPN Tunnel
========================
        ↓
   복호화 / 역캡슐화
        ↓
      사설망
```

## VPN의 주요 기능

- 터널링
- 데이터 암호화
- 사용자 및 장치 인증
- 데이터 무결성 보호
- 원격 네트워크 접속
- 서로 다른 사설 네트워크 간 연결

---

# VPN의 구성 방식

## 1) Site-to-Site VPN

- 서로 떨어진 두 개 이상의 네트워크를 VPN으로 연결하는 방식
- 기업 본사와 지사 간 연결 등에 사용된다.

```text
본사 LAN
   │
VPN Gateway
   │
=== Internet ===
   │
VPN Gateway
   │
지사 LAN
```

- 주로 IPsec VPN 등이 사용된다.

## 2) Remote Access VPN

- 외부 사용자가 인터넷을 통해 내부 사설망에 접속하는 방식
- 재택근무, 외부 직원의 사내 시스템 접속 등에 사용된다.

```text
Remote User
     │
  Internet
     │
 VPN Tunnel
     │
VPN Gateway
     │
Corporate LAN
```

- 주로 다음과 같은 방식이 사용된다.
  - IPsec VPN
  - SSL/TLS VPN

---

# 터널링 프로토콜 비교

| 기술 | 주요 동작 영역 | 터널링 | 자체 암호화 | 주요 용도 |
|---|---|---|---|---|
| PPTP | 2계층 기반 | O | 제한적/별도 방식 필요 | 과거 원격접속 VPN |
| L2TP | 2계층 기반 | O | X | L2TP/IPsec VPN |
| L2TPv3 | 2계층 기반 | O | X | L2 네트워크 연결 |
| MPLS | Layer 2.5 | 논리적 경로 제공 | X | 통신사업자망, MPLS VPN |
| GRE | 3계층 기반 | O | X | 네트워크 간 터널 |
| IPsec | 3계층 | O | O | Site-to-Site VPN, Remote VPN |
| TLS | TCP 상위 | O | O | HTTPS, SSL/TLS VPN |
| SSH | TCP 상위 | O | O | 원격 접속, Port Forwarding |
| SOCKS5 | 애플리케이션 계층 | 프록시 방식 | X | 프록시, 트래픽 중계 |

---

# 정리

- 터널링
  - 원본 데이터를 다른 프로토콜 내부에 캡슐화하여 전달하는 기술
  - 서로 떨어진 네트워크나 호스트 사이에 가상의 통신 경로를 구성한다.

- 터널링과 암호화는 서로 다른 개념이다.

```text
터널링 = 데이터를 캡슐화하여 전달
암호화 = 데이터 내용을 알아볼 수 없도록 보호
```

- 따라서 모든 터널링 프로토콜이 암호화를 제공하는 것은 아니다.

```text
GRE
→ 터널링 O
→ 암호화 X

L2TP
→ 터널링 O
→ 암호화 X

IPsec Tunnel Mode
→ 터널링 O
→ 암호화 O

GRE + IPsec
→ GRE로 터널링
→ IPsec으로 암호화 및 보안 제공

L2TP + IPsec
→ L2TP로 터널링
→ IPsec으로 암호화 및 보안 제공
```

- VPN
  - 터널링 기술을 이용하여 인터넷 위에 가상의 사설 네트워크를 구성한다.
  - 보통 터널링과 함께 암호화, 인증, 무결성 보호 기능을 사용한다.

- 대표적인 VPN 구성

```text
Site-to-Site VPN
→ 네트워크 ↔ 네트워크 연결

Remote Access VPN
→ 사용자 ↔ 사설 네트워크 연결
```
````
