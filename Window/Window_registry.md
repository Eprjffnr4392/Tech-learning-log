## 윈도우 레지스트리의 이해

# 레지스트리란?

- **Windows Registry**는 Windows 운영체제와 응용 프로그램의 **구성 정보 및 설정 정보를 저장하는 계층적 데이터베이스**이다.
- 사용자 계정, 하드웨어, 설치된 프로그램, 운영체제 설정, 서비스, 파일 연결, 응용 프로그램 설정 등의 다양한 정보를 저장한다.
- Windows의 32비트 및 64비트 환경에서 사용된다.
- 레지스트리는 단순히 사용자 설정만 저장하는 것이 아니라 **운영체제의 동작에 필요한 다양한 시스템 구성 정보**를 저장한다.

### 과거의 설정 방식

- 초기 Windows 및 일부 프로그램에서는 `.ini` 파일과 같은 설정 파일을 사용하여 프로그램의 설정 정보를 저장했다.
- 현재 Windows에서도 일부 프로그램은 자체 설정 파일을 사용할 수 있지만, Windows 운영체제의 핵심적인 구성 정보는 레지스트리에 저장되는 경우가 많다.

> 참고: 프로그램이 레지스트리를 사용하지 않고 필요한 설정과 데이터를 자체 파일에 저장한다고 해서 해당 프로그램을 반드시 "포터블 프로그램"이라고 정의할 수 있는 것은 아니다.
> **포터블(Portable) 프로그램**은 일반적으로 설치 과정이나 시스템에 대한 의존성을 최소화하여 프로그램 폴더를 다른 컴퓨터로 옮겨서도 실행할 수 있도록 설계된 프로그램을 의미한다.

---

# 레지스트리의 기본 구조

Windows 레지스트리는 다음과 같은 계층 구조로 구성된다.

```text
레지스트리
   │
   ├── 루트 키(Hive Root)
   │      │
   │      └── 키(Key)
   │             │
   │             ├── 하위 키(Subkey)
   │             │
   │             └── 값(Value)
   │                    ├── 값 이름
   │                    ├── 데이터 형식
   │                    └── 데이터
```

## 키(Key)

- Windows 파일 시스템의 **폴더와 유사한 개념**
- 레지스트리에서 설정 정보를 계층적으로 구분하기 위한 단위
- 하나의 키 안에는 여러 개의 하위 키(Subkey)와 값(Value)이 존재할 수 있다.
- 레지스트리 경로를 이용하여 특정 키의 위치를 표현할 수 있다.

예:

```text
HKEY_LOCAL_MACHINE
└── SOFTWARE
    └── Microsoft
        └── Windows
            └── CurrentVersion
```

---

## 값(Value)

- 키에 저장되는 실제 설정 데이터
- 하나의 값은 일반적으로 다음과 같은 정보를 가진다.

```text
값 이름
값 형식
값 데이터
```

예:

```text
Name      : Test
Type      : REG_SZ
Data      : Hello
```

- 값 데이터는 여러 가지 데이터 형식으로 저장될 수 있다.

### 주요 레지스트리 값 형식

- `REG_SZ`
  - 문자열 데이터

- `REG_EXPAND_SZ`
  - 환경 변수를 포함할 수 있는 문자열 데이터

- `REG_MULTI_SZ`
  - 여러 개의 문자열을 저장

- `REG_DWORD`
  - 32비트 정수 값

- `REG_QWORD`
  - 64비트 정수 값

- `REG_BINARY`
  - 이진 데이터

---

# 레지스트리 편집기

Windows에서는 레지스트리 편집기(Registry Editor)를 이용하여 레지스트리를 확인하고 수정할 수 있다.

### 실행 방법

```text
Win + R
    ↓
regedit
    ↓
Enter
```

또는 CMD에서:

```cmd
regedit
```

### 레지스트리 편집기 구조

- **왼쪽 영역**
  - 레지스트리의 키와 하위 키를 트리 구조로 표시

- **오른쪽 영역**
  - 선택한 키에 포함된 값 이름, 데이터 형식, 데이터를 표시

```text
┌───────────────────────┬────────────────────────────┐
│ 레지스트리 키          │ 값 이름 / 형식 / 데이터     │
│                       │                            │
│ HKEY_LOCAL_MACHINE    │ Name    REG_SZ    Test    │
│ └─ SOFTWARE           │ Value   REG_DWORD  1       │
│    └─ Microsoft       │                            │
└───────────────────────┴────────────────────────────┘
```

> 레지스트리는 Windows의 핵심 설정 정보를 포함하고 있으므로 잘못된 값을 수정하거나 삭제하면 프로그램이나 운영체제가 정상적으로 동작하지 않을 수 있다.

---

# 주요 레지스트리 구조

Windows 레지스트리의 주요 루트 키는 다음과 같다.

## 1) HKEY_CLASSES_ROOT(HKCR)

- 파일 확장자와 파일 형식의 연결 정보
- COM 클래스 등록 정보
- Windows Shell에서 사용하는 파일 연결 및 클래스 정보를 제공한다.
- `HKEY_LOCAL_MACHINE\Software\Classes`와 `HKEY_CURRENT_USER\Software\Classes`의 정보를 통합하여 보여주는 뷰이다.

예:

```text
HKEY_CLASSES_ROOT
└── .txt
    └── txtfile
```

- `.txt` 파일을 어떤 파일 형식으로 처리할지 등의 정보가 연결될 수 있다.

---

## 2) HKEY_CURRENT_USER(HKCU)

- **현재 로그인한 사용자의 설정 정보를 저장**
- 현재 사용자의 바탕 화면, 환경 설정, 응용 프로그램 설정, 사용자별 Windows 설정 등이 포함될 수 있다.

예:

```text
HKEY_CURRENT_USER
└── Software
    └── Microsoft
        └── Windows
```

- HKCU는 현재 로그인한 사용자에게만 적용되는 설정을 저장하는 데 사용된다.

---

## 3) HKEY_LOCAL_MACHINE(HKLM)

- **현재 컴퓨터 전체에 적용되는 시스템 설정 정보를 저장**
- 모든 사용자에게 공통적으로 적용되는 운영체제 및 하드웨어 관련 정보가 포함된다.
- 설치된 소프트웨어, 서비스, 드라이버, 시스템 구성 정보 등이 저장될 수 있다.

예:

```text
HKEY_LOCAL_MACHINE
├── SOFTWARE
├── SYSTEM
└── HARDWARE
```

---

## 4) HKEY_USERS(HKU)

- Windows 시스템에 존재하는 **각 사용자 프로필의 레지스트리 설정을 저장**
- 각 사용자 계정은 SID(Security Identifier)를 기준으로 구분된다.

예:

```text
HKEY_USERS
├── S-1-5-18
├── S-1-5-19
├── S-1-5-20
└── S-1-5-21-...-1001
```

- `S-1-5-21-...-1001`과 같은 SID는 특정 사용자 계정을 나타낸다.
- 현재 로그인한 사용자의 HKCU는 내부적으로 해당 사용자의 HKEY_USERS 아래 사용자 프로필과 연결된다.

---

## 5) HKEY_CURRENT_CONFIG(HKCC)

- 현재 사용 중인 **하드웨어 프로필(Hardware Profile)**에 대한 정보를 제공하는 루트 키
- 실제로 독립적인 레지스트리 데이터베이스가 존재하는 것이 아니라 다른 레지스트리 위치에 대한 **논리적인 뷰 또는 별칭(alias)**로 볼 수 있다.
- 현재 하드웨어 구성과 관련된 정보를 확인할 때 사용된다.

---

# HKEY_CURRENT_USER와 HKEY_USERS의 관계

- `HKEY_CURRENT_USER`는 현재 로그인한 사용자의 레지스트리 설정을 편리하게 접근하기 위한 논리적인 루트이다.
- 실제 사용자별 설정은 `HKEY_USERS` 아래 해당 사용자의 SID에 해당하는 키에 저장된다.

```text
HKEY_USERS
│
├── S-1-5-18
├── S-1-5-19
├── S-1-5-20
└── S-1-5-21-...-1001
              ↑
              │
       현재 로그인한 사용자
              │
              ↓
     HKEY_CURRENT_USER
```

> 따라서 `HKEY_CURRENT_USER > HKEY_USERS`처럼 단순한 우선순위 관계로 이해하는 것은 정확하지 않다.
> **HKCU는 현재 사용자에 대한 편리한 접근 경로이며, HKU에는 사용자별 프로필이 저장된다.**

---

# 레지스트리 하이브(Hive)

- Windows 레지스트리에서 특정 데이터를 파일 형태로 저장하는 논리적인 단위
- 주요 하이브는 다음과 같다.

```text
HKEY_LOCAL_MACHINE\SAM
HKEY_LOCAL_MACHINE\SYSTEM
HKEY_LOCAL_MACHINE\SOFTWARE
HKEY_USERS
```

- Windows에서는 이러한 레지스트리 데이터가 여러 하이브 파일로 저장된다.
- 예:

```text
C:\Windows\System32\config\SAM
C:\Windows\System32\config\SYSTEM
C:\Windows\System32\config\SOFTWARE
```

- 사용자 프로필과 관련된 레지스트리 하이브는 일반적으로 사용자 프로필 디렉터리의 `NTUSER.DAT` 파일과 관련된다.

```text
C:\Users\<사용자>\NTUSER.DAT
```

> 하이브 파일은 Windows가 사용하는 중요한 시스템 파일이므로 직접 수정하거나 삭제해서는 안 된다.

---

# 레지스트리와 자동 실행

레지스트리는 Windows 시작 시 자동으로 실행되는 프로그램과 관련된 정보를 저장하는 데에도 사용된다.

대표적인 위치:

```text
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
```

- 컴퓨터 전체 사용자에게 적용되는 자동 실행 설정에 사용될 수 있다.

현재 사용자에게만 적용되는 자동 실행 위치:

```text
HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run
```

### 예

```text
HKEY_LOCAL_MACHINE
└── SOFTWARE
    └── Microsoft
        └── Windows
            └── CurrentVersion
                └── Run
```

- 해당 키에 등록된 프로그램은 Windows 로그인 과정에서 자동으로 실행될 수 있다.

<img width="780" height="218" alt="image" src="https://github.com/user-attachments/assets/0e40bee2-4262-4797-aedd-0dcfec682183" />

> 참고: `Run` 키는 Windows 자동 실행 메커니즘 중 하나일 뿐이며, Windows의 모든 자동 실행 프로그램이 이 위치에 등록되는 것은 아니다.
> 작업 스케줄러, 서비스, 시작 폴더, Winlogon 관련 항목 등 다양한 자동 실행 지점이 존재한다.

---

# 사용자 SID와 레지스트리

`HKEY_USERS` 아래에서는 각 사용자의 SID를 이용하여 사용자별 설정을 구분한다.

예:

```text
HKEY_USERS
└── S-1-5-21-466302358-3968130502-2079693883-1001
```

- `S-1-5-21-...`은 Windows 사용자 또는 컴퓨터 계정을 식별하는 SID의 형태이다.
- 마지막 `1001`과 같은 값은 특정 사용자 계정을 식별하는 RID(Relative Identifier)의 일부이다.

<img width="675" height="351" alt="image" src="https://github.com/user-attachments/assets/5be79be9-edf4-4a16-935e-03d4d97b79aa" />

---

# REG 명령어

Windows에서는 CMD의 `reg` 명령어를 이용하여 레지스트리를 명령줄에서 조회하고 관리할 수 있다.

### 도움말 확인

```cmd
reg /?
```

또는 특정 명령어의 도움말:

```cmd
reg query /?
reg add /?
reg delete /?
```

---

## REG QUERY

- 지정한 레지스트리 키의 값과 하위 키를 조회한다.

```cmd
reg query "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Run"
```

- 레지스트리의 자동 실행 항목 등을 확인할 때 사용할 수 있다.

---

## REG ADD

- 레지스트리 키 또는 값을 추가한다.
- 기존 값이 있는 경우 값을 수정할 수도 있다.

```cmd
reg add "HKCU\Software\Test" /v TestValue /t REG_SZ /d "Hello" /f
```

주요 옵션:

- `/v`
  - 값 이름 지정

- `/t`
  - 값 형식 지정

- `/d`
  - 데이터 지정

- `/f`
  - 확인 과정 없이 강제로 실행

---

## REG DELETE

- 지정한 레지스트리 키 또는 값을 삭제한다.

```cmd
reg delete "HKCU\Software\Test" /v TestValue
```

> 레지스트리 값을 삭제하면 프로그램이나 Windows 기능이 정상적으로 동작하지 않을 수 있으므로 주의해야 한다.

---

## REG COPY

- 레지스트리 키와 하위 트리를 다른 위치로 복사한다.

```cmd
reg copy 원본키 대상키
```

---

## REG SAVE

- 지정한 레지스트리 키와 하위 트리를 파일로 저장한다.
- 레지스트리 백업 등에 사용할 수 있다.

```cmd
reg save HKLM\SOFTWARE C:\backup\software.hiv
```

---

## REG RESTORE

- `REG SAVE`로 저장한 레지스트리 하이브 파일을 지정한 레지스트리 키로 복원한다.

```cmd
reg restore HKLM\SOFTWARE C:\backup\software.hiv
```

> 시스템에서 사용 중인 하이브를 복원하는 경우 권한이나 사용 상태 등에 따라 제한이 발생할 수 있다.

---

## REG LOAD

- 저장된 레지스트리 하이브 파일을 현재 레지스트리에 임시로 로드한다.
- 원래 위치와 다른 키 이름으로 하이브를 연결하여 내용을 확인하거나 수정할 때 사용할 수 있다.

```cmd
reg load HKLM\TempHive C:\backup\software.hiv
```

---

## REG UNLOAD

- `REG LOAD`로 로드한 레지스트리 하이브를 언로드한다.

```cmd
reg unload HKLM\TempHive
```

---

## REG COMPARE

- 두 레지스트리 키를 비교하여 서로 다른 부분 또는 동일한 부분을 확인한다.

```cmd
reg compare HKCU\Software\Test HKCU\Software\Test2
```

- 레지스트리 변경 사항을 비교하거나 문제를 분석할 때 사용할 수 있다.

---

## REG EXPORT

- 지정한 레지스트리 키와 값을 `.reg` 파일로 내보낸다.
- 사람이 읽거나 다른 Windows 시스템에서 가져오기 쉬운 형태로 백업할 때 사용할 수 있다.

```cmd
reg export HKCU\Software\Test C:\backup\test.reg
```

---

## REG IMPORT

- `REG EXPORT`로 생성한 `.reg` 파일을 레지스트리에 가져온다.

```cmd
reg import C:\backup\test.reg
```

> `.reg` 파일에는 레지스트리 변경 내용이 포함되어 있으므로 출처가 불분명한 파일을 실행하지 않는 것이 좋다.

---

## REG FLAGS

- 특정 레지스트리 키의 **레지스트리 가상화 플래그 등의 플래그를 조회하거나 설정**할 때 사용한다.
- Windows Vista 이후의 UAC(User Account Control) 및 레거시 응용 프로그램 호환성과 관련된 레지스트리 가상화 기능과 연관된다.

```cmd
reg flags /?
```

> `REG FLAGS`를 단순히 "Windows Vista 이후부터 적용된 레지스트리 가상화 기능"이라고 정의하기보다는, **레지스트리 키에 대한 플래그를 관리하는 명령어**라고 이해하는 것이 정확하다.

---

# 레지스트리 보안

레지스트리는 Windows 운영체제의 중요한 설정 정보를 포함하므로 접근 권한이 설정되어 있다.

- 일반 사용자에게 제한된 키가 존재한다.
- 관리자 권한이 필요한 키가 존재한다.
- 특정 레지스트리 키에는 ACL(Access Control List)을 통해 접근 권한이 설정되어 있다.
- 잘못된 레지스트리 수정은 프로그램 오류나 Windows 부팅 문제를 발생시킬 수 있다.

### 레지스트리 수정 시 주의사항

```text
레지스트리 수정 전
       ↓
변경할 키 확인
       ↓
기존 값 백업
       ↓
값 수정
       ↓
Windows 또는 프로그램 동작 확인
```

---

# 보안 및 악성코드 분석에서의 레지스트리

레지스트리는 악성코드 분석에서도 중요한 자료이다.

악성 프로그램이 시스템이 시작될 때 자동으로 실행되도록 레지스트리의 자동 실행 위치를 변경할 수 있기 때문이다.

대표적인 자동 실행 위치:

```text
HKCU\Software\Microsoft\Windows\CurrentVersion\Run

HKLM\Software\Microsoft\Windows\CurrentVersion\Run
```

따라서 의심스러운 프로그램이 발견되었을 때 다음과 같은 항목을 함께 확인할 수 있다.

- 자동 실행 레지스트리
- 서비스 등록 여부
- 작업 스케줄러 등록 여부
- 실행 파일 경로
- 부모-자식 프로세스 관계
- 디지털 서명
- 네트워크 연결
- 이벤트 로그

```text
의심스러운 프로세스 발견
        ↓
실행 파일 경로 확인
        ↓
부모 프로세스 확인
        ↓
자동 실행 레지스트리 확인
        ↓
서비스 / 작업 스케줄러 확인
        ↓
디지털 서명 및 해시 확인
        ↓
네트워크 통신 확인
        ↓
악성 여부 종합 분석
```

---

# 주요 레지스트리 구조 정리

| 루트 키 | 주요 역할 |
|---|---|
| HKEY_CLASSES_ROOT(HKCR) | 파일 연결 및 COM 클래스 등의 정보 |
| HKEY_CURRENT_USER(HKCU) | 현재 로그인한 사용자의 설정 |
| HKEY_LOCAL_MACHINE(HKLM) | 컴퓨터 전체에 적용되는 시스템 설정 |
| HKEY_USERS(HKU) | 사용자별 프로필 설정 |
| HKEY_CURRENT_CONFIG(HKCC) | 현재 사용 중인 하드웨어 프로필 관련 정보 |

---

# REG 명령어 정리

| 명령어 | 주요 기능 |
|---|---|
| `REG QUERY` | 레지스트리 키 및 값 조회 |
| `REG ADD` | 키 또는 값 추가/수정 |
| `REG DELETE` | 키 또는 값 삭제 |
| `REG COPY` | 레지스트리 트리 복사 |
| `REG SAVE` | 레지스트리 하이브 저장 |
| `REG RESTORE` | 저장된 하이브 복원 |
| `REG LOAD` | 하이브 로드 |
| `REG UNLOAD` | 로드한 하이브 언로드 |
| `REG COMPARE` | 두 레지스트리 키 비교 |
| `REG EXPORT` | 레지스트리를 `.reg` 파일로 내보내기 |
| `REG IMPORT` | `.reg` 파일을 레지스트리에 가져오기 |
| `REG FLAGS` | 레지스트리 키의 플래그 조회/설정 |

---

# 핵심 정리

```text
Windows Registry
→ Windows 운영체제와 프로그램의 구성 및 설정 정보를 저장하는 계층적 데이터베이스

Key
→ 폴더와 유사한 계층 구조의 단위

Value
→ 키에 저장되는 실제 설정 데이터

Hive
→ 레지스트리 데이터를 저장하는 논리적인 단위

주요 Root Key
→ HKCR
→ HKCU
→ HKLM
→ HKU
→ HKCC

레지스트리 편집기
→ Win + R
→ regedit

명령줄 관리
→ reg query
→ reg add
→ reg delete
→ reg export
→ reg import
→ reg save
→ reg restore
→ reg load
→ reg unload
→ reg compare
→ reg copy
→ reg flags
```

### 보안 관점에서 중요한 부분

```text
레지스트리
    ↓
시스템 설정
사용자 설정
서비스 설정
파일 연결
자동 실행
프로그램 설정
    ↓
악성코드가 지속성(Persistence)을 확보하는 데 악용할 수 있음
    ↓
레지스트리 분석
→ 자동 실행 항목 확인
→ 의심스러운 경로 확인
→ 서비스/작업 스케줄러와 연계 분석
→ 프로세스 및 네트워크 정보와 함께 종합 분석
```
