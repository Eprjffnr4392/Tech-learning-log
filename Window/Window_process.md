## 윈도우 주요 프로세스

### 프로그램, 프로세스, 스레드

- 프로그램(Program)
  - 하드 디스크 등의 저장 장치에 저장되어 있는 실행 가능한 코드와 관련 데이터
  - 실행되기 전에는 단순히 파일 형태로 존재한다.

- 프로세스(Process)
  - 실행 중인 프로그램의 인스턴스
  - 운영체제로부터 메모리 공간, 핸들, 실행 환경 등의 자원을 할당받아 실행된다.
  - 하나의 프로세스에는 하나 이상의 스레드가 존재할 수 있다.

- 스레드(Thread)
  - 프로세스 내부에서 실제 작업을 수행하는 실행 단위
  - CPU에서 실행되는 기본적인 스케줄링 단위
  - 하나의 프로세스는 여러 개의 스레드를 가질 수 있으며, 여러 스레드가 프로세스의 자원을 공유할 수 있다.

```text
프로그램
  ↓ 실행
프로세스
  ├── 스레드
  ├── 스레드
  └── 스레드
```

---

# 세션(Session)

- Windows에서 세션(Session)은 사용자 로그온과 프로세스 및 데스크톱 환경을 구분하는 논리적인 실행 공간이다.
- 하나의 세션에는 해당 세션에서 실행되는 프로세스와 사용자 환경 등이 포함된다.
- Windows는 여러 사용자가 동시에 로그인하거나 원격 데스크톱을 사용할 수 있도록 여러 세션을 관리할 수 있다.

### Session 0

- Windows Vista 이전의 Windows에서는 첫 번째 대화형 사용자 세션이 Session 0으로 사용되었다.
- Windows Vista 이후부터는 **Session 0을 서비스와 시스템 프로세스가 사용하는 비대화형 세션**으로 분리하였다.
- 사용자가 로그인하여 사용하는 대화형 세션은 Session 0과 분리된 별도의 세션 번호를 사용한다.
- 이러한 구조는 서비스와 사용자 응용 프로그램을 분리하여 **Session 0 Isolation**을 구현하기 위한 것이다.

```text
Windows Vista 이후

Session 0
└── 시스템 프로세스 및 서비스
    └── 비대화형 세션

Session 1 이상
└── 대화형 사용자 세션
    ├── explorer.exe
    ├── 사용자 응용 프로그램
    └── 기타 사용자 프로세스
```

---

# 프로세스 종류

## Windows 7 주요 프로세스

Windows 7의 주요 시스템 프로세스는 다음과 같다.

### 1) System

- Windows 커널에서 사용하는 시스템 프로세스
- 일반적인 사용자 응용 프로그램이 실행되는 프로세스와는 다르다.
- 커널 모드에서 동작하는 일부 시스템 스레드를 담당한다.
- 프로세스 ID(PID)가 일반적으로 4로 표시된다.

---

### 2) smss.exe

- **Session Manager Subsystem**
- Windows 부팅 과정에서 세션을 초기화하는 역할을 담당한다.
- 시스템 세션과 사용자 세션을 초기화한다.
- 부팅 과정에서 필요한 환경을 구성하고 이후 필요한 프로세스를 실행한다.

### 주요 역할

- Session 0 초기화
- 사용자 세션 초기화
- csrss.exe 및 wininit.exe 등의 초기화 과정 지원
- 세션별 환경 구성

Windows의 세션이 생성될 때 각 세션을 초기화하기 위해 SMSS가 관여한다.

```text
smss.exe
   │
   ├── Session 0 초기화
   │      └── wininit.exe
   │
   └── 사용자 세션 초기화
          └── csrss.exe
          └── winlogon.exe
```

> 참고: 세션이 생성된 이후 해당 세션의 SMSS 인스턴스는 초기화 작업을 마치면 종료될 수 있다. 따라서 항상 여러 개의 smss.exe가 실행되고 있는 것은 아니다.

<img width="728" height="181" alt="Image" src="https://github.com/user-attachments/assets/fe3a6549-b132-477b-a85b-0590b78eed29" />

### 프로세스의 부모-자식 관계

Windows의 프로세스를 분석할 때는 프로세스 이름만 확인하는 것이 아니라 **부모 프로세스와 자식 프로세스의 관계(Process Tree)**를 함께 확인하는 것이 중요하다.

예를 들어 정상적인 시스템 프로세스와 이름이 동일한 악성 프로그램이 존재할 수 있다.

```text
의심스러운 프로세스 발견
        ↓
프로세스 이름 확인
        ↓
실행 경로 확인
        ↓
부모 프로세스 확인
        ↓
자식 프로세스 확인
        ↓
서명 및 해시 등의 추가 정보 확인
```

- 동일한 이름의 프로세스라도 실행 위치나 부모 프로세스가 정상적인 Windows 프로세스와 다를 수 있다.
- 따라서 **프로세스 이름만으로 악성 여부를 판단해서는 안 된다.**

---

### 3) wininit.exe

- **Windows Initialization Process**
- Session 0의 초기화 과정에서 실행되는 핵심 시스템 프로세스
- Windows가 부팅된 후 백그라운드에서 실행되는 여러 시스템 구성요소의 초기화를 지원한다.
- 주요 하위 프로세스로 다음과 같은 프로세스가 연결될 수 있다.

```text
wininit.exe
├── services.exe
├── lsass.exe
└── 기타 시스템 프로세스
```

- services.exe
  - Windows 서비스 관리

- lsass.exe
  - 로컬 보안 인증 및 보안 정책 관련 기능

> 참고: lsm.exe는 Windows 7에서 일반적으로 wininit.exe의 직접적인 자식 프로세스로 설명하기보다는 별도의 시스템 구성요소로 구분하는 것이 적절하다.

---

### 4) taskhost.exe

- Windows 7에서 사용되는 **Task Host** 프로세스
- DLL 기반 작업을 호스팅하기 위한 프로세스
- 작업 스케줄러에 의해 실행되는 일부 DLL 기반 작업을 처리한다.
- 하나의 프로세스에서 여러 작업을 호스팅할 수 있다.

> 참고: "모든 DLL 기반 서비스나 그룹 서비스의 호스트"라고 설명하는 것은 부정확하다.
> Windows 서비스의 DLL 호스팅은 주로 svchost.exe가 담당한다.

---

### 5) lsass.exe

- **Local Security Authority Subsystem Service**
- Windows의 보안 정책 및 사용자 인증과 관련된 핵심 시스템 프로세스
- 사용자 로그온 및 인증 과정에 관여한다.
- 보안 정책 적용
- 사용자 인증
- 보안 토큰 생성 및 관리
- 비밀번호 변경 등의 보안 관련 기능을 담당한다.

```text
사용자 로그온
      ↓
winlogon.exe
      ↓
lsass.exe
      ↓
인증 및 보안 정책 확인
```

- 정상적인 Windows 시스템에서는 `C:\Windows\System32\lsass.exe`에 위치한다.
- 이름이 같은 악성 프로그램이 존재할 수 있으므로 실행 경로와 디지털 서명 등을 함께 확인해야 한다.

---

### 6) csrss.exe

- **Client/Server Runtime Subsystem**
- Windows의 핵심 사용자 모드 시스템 프로세스 중 하나
- 프로세스 및 스레드 생성과 종료, 콘솔 관련 기능 등 Windows의 중요한 시스템 기능을 담당한다.
- 초기 Windows NT에서는 더 많은 Win32 서브시스템 기능을 담당했지만, 현재 Windows에서는 역할이 많이 축소되었다.

### 세션과 CSRSS

- 각 Windows 세션에는 일반적으로 해당 세션을 위한 `csrss.exe` 인스턴스가 존재한다.
- 따라서 작업 관리자에서 여러 개의 `csrss.exe`가 관찰될 수 있다.
- 이것은 정상적인 동작일 수 있다.

<img width="753" height="185" alt="Image" src="https://github.com/user-attachments/assets/7d684fb0-73ff-400e-a619-6212a6a7b749" />

<img width="741" height="252" alt="Image" src="https://github.com/user-attachments/assets/826db41f-afd1-424a-92b3-e493f1992293" />

- 위와 같이 System 프로세스 wininit.exe 등과 같은 깊이에서 실행되는 것을 알 수 있음.

---

### 7) services.exe

- **Service Control Manager(SCM)**의 프로세스
- Windows 서비스를 생성, 시작, 중지 및 관리한다.
- 시스템이 시작될 때 자동으로 실행되어야 하는 서비스들을 관리한다.
- 여러 Windows 서비스의 시작 및 종료를 담당한다.

```text
services.exe
├── svchost.exe
├── svchost.exe
├── 기타 서비스 프로세스
└── ...
```

> 참고: 작업 스케줄러 자체를 services.exe가 직접 관리한다고 표현하기보다는, Windows 서비스의 관리가 주요 역할이라고 이해하는 것이 정확하다.

---

### 8) svchost.exe

- **Service Host**
- Windows 서비스를 호스팅하는 시스템 프로세스
- 실행 파일(.exe)이 아닌 **DLL 형태로 구현된 Windows 서비스**를 실행할 수 있도록 호스팅한다.
- 여러 Windows 서비스가 하나의 svchost.exe 프로세스에서 실행될 수 있다.
- 시스템 구성에 따라 여러 개의 svchost.exe 프로세스가 동시에 실행될 수 있다.

```text
services.exe
      │
      ├── svchost.exe
      │      ├── Service A
      │      └── Service B
      │
      ├── svchost.exe
      │      ├── Service C
      │      └── Service D
      │
      └── svchost.exe
             └── Service E
```

<img width="718" height="327" alt="image" src="https://github.com/user-attachments/assets/51e1841c-d422-44ae-b51f-bffe2ee6332e" />

<img width="700" height="308" alt="image" src="https://github.com/user-attachments/assets/839b3706-0d3b-4131-82cb-e953e4b3f9cd" />

- 위와 같이 각각의 `svchost.exe`가 어떤 서비스를 담당하고 있는지 확인할 수 있다.

### svchost.exe의 리소스 사용

- Windows에서 특정 svchost.exe 프로세스가 CPU나 메모리를 많이 사용하는 경우가 있다.
- 이때 무조건 svchost.exe 자체를 종료하기보다는 **해당 프로세스가 어떤 서비스를 호스팅하고 있는지 먼저 확인**해야 한다.
- 서비스를 종료하면 Windows의 다른 기능에 영향을 줄 수 있다.
- 서비스에 문제가 있는 경우 해당 서비스의 상태, 이벤트 로그, 관련 프로세스 등을 확인하여 원인을 분석하는 것이 좋다.
- 서비스의 복구 옵션 등에 의해 중지된 서비스가 다시 시작될 수도 있다.

```text
svchost.exe 리소스 사용량 증가
          ↓
호스팅 중인 서비스 확인
          ↓
어떤 서비스가 리소스를 사용하는지 확인
          ↓
이벤트 로그 및 서비스 상태 확인
          ↓
원인 분석 및 문제 해결
```

---

### 9) lsm.exe

- **Local Session Manager**
- Windows 7에서 사용자 세션 및 터미널 서비스와 관련된 기능을 담당하는 시스템 프로세스
- 로컬 및 원격 세션의 생성과 관리에 관여한다.
- 터미널 서비스 및 원격 데스크톱 환경의 세션 관리와 관련된 기능을 수행한다.

> 참고: "smss.exe와 같이 새 세션이 생성될 때마다 실행된다"라고 단순하게 이해하기보다는, LSM은 시스템에서 세션 관리 기능을 제공하는 프로세스라고 이해하는 것이 적절하다.

---

### 10) winlogon.exe

- **Windows Logon Process**
- Windows 사용자 로그온 및 로그오프 과정과 관련된 핵심 시스템 프로세스
- 사용자 로그온 과정에서 인증 및 사용자 환경 초기화에 관여한다.
- Ctrl + Alt + Delete 등의 보안 관련 사용자 입력 처리에도 관여한다.
- 사용자 세션의 로그온 및 로그오프 과정에서 중요한 역할을 한다.

```text
사용자 로그온
     ↓
winlogon.exe
     ↓
사용자 인증 과정
     ↓
사용자 세션 초기화
     ↓
explorer.exe 등 사용자 환경 실행
```

---

### 11) explorer.exe

- Windows의 기본 셸(Shell) 프로세스
- 파일 탐색기(File Explorer) 기능을 제공한다.
- 파일 및 폴더에 접근하고 관리할 수 있는 GUI 환경을 제공한다.
- Windows 바탕 화면, 작업 표시줄, 시작 메뉴 등의 사용자 셸 기능도 담당한다.

```text
explorer.exe
├── 바탕 화면
├── 작업 표시줄
├── 시작 메뉴
└── 파일 탐색기
```

---

### 12) iexplore.exe

- **Internet Explorer**의 실행 파일
- Windows 7 시절 Internet Explorer 웹 브라우저를 실행할 때 사용되었다.
- `explorer.exe`에서 실행된 프로세스라고 단순하게 설명하는 것은 정확하지 않다.
- Internet Explorer는 독립적인 응용 프로그램이며, explorer.exe가 반드시 부모 프로세스인 것은 아니다.

> 참고: `iexplore.exe`는 Internet Explorer의 프로세스 이름이며, 현재 Internet Explorer는 지원 종료되었기 때문에 최신 Windows 환경에서는 일반적인 웹 브라우저 프로세스로 사용되지 않는다.

---

# 프로세스 종류 - Windows 10

Windows 10의 주요 시스템 프로세스는 Windows 7과 유사하지만 새로운 Windows 기능과 보안 기능을 지원하기 위한 프로세스가 추가되었다.

## 1) RuntimeBroker.exe

- **Runtime Broker**
- Microsoft Store 앱 및 UWP(Universal Windows Platform) 앱의 권한과 관련된 기능을 관리하는 Windows 시스템 프로세스
- 앱이 카메라, 마이크, 위치 정보 등의 특정 시스템 리소스에 접근할 때 권한과 관련된 작업을 중개한다.
- 앱의 권한 및 시스템 API 접근을 관리하는 역할을 수행한다.

### UWP

- **Universal Windows Platform**
- Windows 10에서 다양한 Windows 장치에서 실행할 수 있는 앱을 개발하기 위한 플랫폼
- PC, Xbox 등의 Windows 기반 장치를 대상으로 동일한 앱 모델을 사용할 수 있도록 설계되었다.

> 참고: RuntimeBroker.exe를 단순히 "UWP 앱과 전체 Windows API 사이의 프록시"라고 표현하기보다는 **Windows 앱의 권한 및 리소스 접근을 중개하는 시스템 프로세스**라고 이해하는 것이 적절하다.

---

## 2) taskhostw.exe

- Windows에서 DLL 기반 작업을 호스팅하는 프로세스
- Windows 7의 `taskhost.exe`와 유사한 역할을 수행한다.
- `taskhostw.exe`의 `w`는 Windows의 64비트 환경 및 Windows 버전에서 사용되는 실행 파일 이름의 차이를 나타내는 것으로 이해할 수 있다.
- 작업 스케줄러 등을 통해 실행되는 일부 DLL 기반 작업을 호스팅한다.

```text
Windows 7
→ taskhost.exe

Windows 10
→ taskhostw.exe
```

---

## 3) lsaiso.exe

- **LSA Isolated Process**
- Windows 10에서 보안 기능을 강화하기 위해 사용되는 프로세스
- Credential Guard가 활성화된 환경에서는 LSA 프로세스의 중요한 자격 증명 및 보안 비밀을 격리된 환경에서 보호하는 데 사용된다.
- 가상화 기반 보안(Virtualization-Based Security, VBS)을 이용하여 중요한 보안 정보를 일반 운영체제 환경과 분리한다.

```text
일반 Windows 환경
       │
       ├── lsass.exe
       │
       └── 사용자 및 시스템 프로세스

격리된 보안 환경
       │
       └── lsaiso.exe
             ↓
       중요한 자격 증명 보호
```

- 공격자가 일반적인 운영체제 환경에서 LSASS 프로세스의 자격 증명을 탈취하는 것을 어렵게 하는 것이 주요 목적이다.
- 특히 Credential Guard와 같은 보안 기능과 함께 사용된다.

> 참고: `lsaiso.exe`가 항상 실행되는 것은 아니며, Windows의 보안 구성 및 Credential Guard 등의 기능 활성화 여부에 따라 동작이 달라질 수 있다.
> 또한 "원격 인증을 요구할 경우 RPC 채널을 사용하여 요청을 프록시한다"는 내용을 lsaiso.exe의 일반적인 주요 기능으로 설명하는 것은 부정확하므로 제외하는 것이 좋다.

---

# Windows 주요 프로세스 관계

Windows의 주요 시스템 프로세스는 부모-자식 관계를 통해 어느 정도 실행 흐름을 파악할 수 있다.

```text
System
  │
  └── smss.exe
       │
       ├── Session 0
       │    └── wininit.exe
       │          ├── services.exe
       │          │     └── svchost.exe
       │          │
       │          └── lsass.exe
       │
       └── 사용자 세션
            ├── csrss.exe
            ├── winlogon.exe
            │
            └── explorer.exe
                  └── 사용자 응용 프로그램
```

> 실제 Windows 버전과 시스템 구성에 따라 프로세스의 부모-자식 관계 및 실행되는 프로세스는 달라질 수 있다.

---

# 프로세스 분석 시 확인해야 할 사항

의심스러운 프로세스를 발견했을 때는 프로세스 이름만 확인해서는 안 된다.

### 1) 프로세스 이름

- 정상적인 Windows 프로세스와 동일한 이름을 사용하는 악성 프로그램이 존재할 수 있다.

### 2) 실행 경로

- 정상적인 Windows 시스템 프로세스는 일반적으로 정해진 시스템 디렉터리에서 실행된다.
- 예:

```text
C:\Windows\System32\
```

- 동일한 이름이라도 사용자 폴더나 임시 폴더 등 비정상적인 위치에서 실행된다면 추가적인 확인이 필요하다.

### 3) 부모 프로세스

- 해당 프로세스를 어떤 프로세스가 실행했는지 확인한다.
- 정상적인 Windows 프로세스 트리와 비교하여 비정상적인 부모-자식 관계가 있는지 확인한다.

### 4) 디지털 서명

- Microsoft 등 신뢰할 수 있는 업체의 디지털 서명이 존재하는지 확인한다.

### 5) 네트워크 연결

- 외부 IP와 비정상적인 통신을 수행하는지 확인한다.

### 6) CPU 및 메모리 사용량

- 특정 프로세스가 CPU나 메모리를 과도하게 사용하는지 확인한다.
- 단순히 리소스 사용량이 높다는 이유만으로 악성 프로세스라고 판단해서는 안 된다.

---

# 정리

## Windows 주요 프로세스

| 프로세스 | 주요 역할 |
|---|---|
| System | 커널 모드 시스템 스레드 등 담당 |
| smss.exe | 세션 초기화 및 관리 |
| wininit.exe | Session 0 초기화 및 시스템 프로세스 실행 지원 |
| csrss.exe | Windows 핵심 사용자 모드 시스템 기능 담당 |
| services.exe | Windows 서비스 관리 |
| svchost.exe | DLL 기반 Windows 서비스 호스팅 |
| lsass.exe | 인증 및 보안 정책 관련 기능 |
| lsm.exe | Windows 7의 세션 관리 |
| winlogon.exe | 사용자 로그온/로그오프 및 보안 관련 기능 |
| explorer.exe | Windows 셸 및 파일 탐색기 |
| iexplore.exe | Internet Explorer |
| taskhost.exe | Windows 7의 DLL 기반 작업 호스팅 |
| taskhostw.exe | Windows 10의 DLL 기반 작업 호스팅 |
| RuntimeBroker.exe | Windows 앱의 권한 및 리소스 접근 중개 |
| lsaiso.exe | VBS/Credential Guard 환경에서 중요 자격 증명 보호 |

## 핵심

```text
프로그램
→ 저장 장치에 존재하는 실행 코드

프로세스
→ 실행 중인 프로그램의 인스턴스

스레드
→ 프로세스 내부에서 실제 작업을 수행하는 실행 단위

세션
→ 사용자 로그온 및 프로세스 실행 환경을 구분하는 논리적 공간

프로세스 분석
→ 이름 + 실행 경로 + 부모/자식 관계 + 서명 + 네트워크 + 리소스 사용량 등을 종합적으로 확인
```
