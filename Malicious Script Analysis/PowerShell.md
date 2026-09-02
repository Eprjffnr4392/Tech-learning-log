## PowerShell

- **PowerShell**은 Microsoft에서 개발한 명령줄 셸(Shell)이자 스크립팅 언어
- Windows뿐만 아니라 Linux, macOS에서도 사용할 수 있는 크로스 플랫폼 자동화 도구
- Windows PowerShell은 .NET Framework를 기반으로 동작하며, PowerShell 6 이후의 PowerShell(Core)은 최신 .NET을 기반으로 동작
- 일반적인 명령 실행뿐만 아니라 파일, 프로세스, 서비스, 레지스트리, 네트워크 등 Windows 시스템의 다양한 요소를 자동화하고 관리할 수 있다.
- CMD와 달리 명령의 결과를 단순한 텍스트뿐만 아니라 **객체(Object)** 형태로 파이프라인을 통해 전달할 수 있다.

### CMD와 PowerShell의 차이

```text
CMD
→ 주로 문자열 기반으로 명령 결과를 처리
→ 단순한 명령 실행 및 배치 작업에 적합

PowerShell
→ 객체(Object) 기반의 파이프라인
→ .NET API 및 Windows 관리 기능과 연동
→ 복잡한 시스템 관리 및 자동화에 적합
→ 스크립팅 언어 기능 제공
```

- Windows PowerShell은 Windows 7부터 기본적으로 포함되었다.
- 이후 Windows 10, Windows 11에서도 Windows PowerShell 5.1이 기본 제공된다.
- 최신 PowerShell은 **PowerShell 7.x** 계열로 별도로 설치할 수 있다.

> `powershell.exe`는 Windows PowerShell 5.1 계열의 실행 파일이며, PowerShell 7의 일반적인 실행 파일은 `pwsh.exe`이다.

---

# PowerShell 스크립트 파일

- PowerShell 스크립트의 확장자는 `.ps1`을 사용한다.

```text
example.ps1
```

> `.ps1`의 `1`이 PowerShell 엔진 버전 1을 의미하는 것은 아니다.
> PowerShell 스크립트 파일의 확장자를 나타내는 이름일 뿐이며, PowerShell 버전이 변경되어도 `.ps1` 확장자는 그대로 사용된다.

---

# 왜 공격자는 PowerShell을 사용하는가?

PowerShell은 Windows 관리 및 자동화에 널리 사용되는 정상적인 도구이기 때문에 공격에도 악용될 수 있다.

## 1) 운영체제에 기본적으로 존재

- Windows PowerShell 5.1은 현대적인 Windows 환경에 기본적으로 포함되어 있다.
- 별도의 악성 프로그램을 설치하지 않고도 다양한 시스템 기능을 사용할 수 있다.

## 2) 정상적인 관리 도구

- PowerShell은 시스템 관리자와 IT 담당자가 정상적인 업무에서도 사용하는 도구이다.
- 따라서 단순히 `powershell.exe`가 실행되었다는 사실만으로 악성 행위라고 판단할 수 없다.
- 보안 제품에서는 **실행된 명령, 부모 프로세스, 사용자, 네트워크 통신, 파일 접근 등의 행위**를 함께 분석해야 한다.

## 3) 강력한 시스템 접근 기능

PowerShell을 이용하면 다양한 Windows 기능을 스크립트로 제어할 수 있다.

```text
파일 시스템
프로세스
서비스
레지스트리
네트워크
Windows API
.NET 클래스
WMI
Active Directory
```

## 4) 메모리 내 실행 및 원격 콘텐츠 처리에 악용 가능

- PowerShell은 네트워크에서 데이터를 가져오거나 문자열을 코드로 해석하는 기능 등을 제공한다.
- 이러한 기능이 악성 코드 실행과 결합될 경우 디스크에 파일을 직접 생성하지 않는 형태의 공격에 악용될 수 있다.

> 단, PowerShell을 사용한다고 해서 자동으로 "파일리스 공격"이 되는 것은 아니다.

## 5) 다양한 난독화 기법에 악용 가능

공격자는 탐지를 어렵게 만들기 위해 다음과 같은 방법을 사용할 수 있다.

- 문자열 인코딩
- 문자열 분할
- 변수 및 함수 이름 난독화
- Base64 인코딩
- 명령어 구조 변경
- 여러 PowerShell 기능의 조합

> Base64 인코딩 자체가 악성 행위를 의미하는 것은 아니다.
> 정상적인 PowerShell에서도 인코딩된 문자열을 사용할 수 있으므로 실행 과정과 행위를 함께 분석해야 한다.

---

# 관리자 도구와 악성 행위의 경계

- PowerShell은 기업 환경에서 시스템 관리 및 자동화를 위해 널리 사용된다.
- 특히 **Active Directory(AD)** 환경에서 사용자, 컴퓨터, 그룹 및 권한 등을 관리하기 위한 자동화에 활용된다.
- 따라서 공격자가 PowerShell을 사용하는 경우 정상적인 관리자 활동과 외형상 유사하게 보일 수 있다.

```text
정상적인 관리자
    ↓
PowerShell 실행
    ↓
시스템 관리
    ↓
파일 / 서비스 / 사용자 / 네트워크 관리

공격자
    ↓
PowerShell 실행
    ↓
시스템 명령 실행
    ↓
정보 수집 / 악성 코드 실행 / 지속성 확보 등
```

- 따라서 보안 관제에서는 `powershell.exe` 실행 자체보다 **누가, 어떤 부모 프로세스에서, 어떤 명령을 실행했는지**를 분석하는 것이 중요하다.

---

# PowerShell ISE

- PowerShell ISE(Integrated Scripting Environment)는 PowerShell 스크립트를 작성하고 실행하며 디버깅할 수 있는 GUI 기반 개발 환경
- Windows PowerShell 스크립트 작성 및 디버깅에 사용할 수 있다.
- 실행 파일:

```text
powershell_ise.exe
```

### 주요 기능

- PowerShell 스크립트 편집
- 구문 강조
- 스크립트 실행
- 중단점 설정
- 단계별 실행
- 변수 및 디버깅 정보 확인

> PowerShell ISE는 Windows PowerShell 5.1 계열의 도구이며, **PowerShell 7에서는 ISE가 지원되지 않는다.**
> PowerShell 7 환경에서는 일반적으로 Visual Studio Code와 PowerShell 확장 등을 사용한다.

---

# ISE 디버깅 기본

PowerShell ISE는 스크립트 실행을 특정 지점에서 중단하고 변수의 상태를 확인할 수 있는 디버깅 기능을 제공한다.

- `F9`
  - 현재 코드에 중단점(Breakpoint) 설정/해제

- `F5`
  - 스크립트 실행 또는 디버깅 시작

- `F10`
  - 한 줄씩 실행
  - **Step Over**
  - 함수 호출이 있어도 함수 내부로 들어가지 않고 다음 코드로 진행

- `F11`
  - 함수 내부로 진입
  - **Step Into**

- `Shift + F11`
  - 현재 함수에서 빠져나옴
  - **Step Out**

### 디버깅 과정

```text
중단점 설정
    ↓
F5로 실행
    ↓
중단점에서 실행 중지
    ↓
변수 값 확인
    ↓
F10 / F11로 코드 진행
    ↓
실행 흐름 분석
```

---

# 공격에 악용되는 주요 PowerShell 실행 방식

PowerShell은 다양한 방식으로 명령과 데이터를 처리할 수 있으며, 이러한 기능이 악성 행위에 악용될 수 있다.

## 1) EncodedCommand

- `-EncodedCommand` 옵션을 사용하여 명령을 Base64 형태로 전달하는 방식
- PowerShell에서 사용하는 명령은 **UTF-16LE 인코딩 후 Base64로 인코딩**하는 방식이 일반적이다.
- 특수문자나 따옴표 등으로 인한 명령줄 파싱 문제를 줄이는 용도로도 사용할 수 있다.
- 보안 관점에서는 인코딩된 명령을 복호화하여 실제 실행 내용을 확인하는 것이 중요하다.

```powershell
powershell.exe -EncodedCommand [Base64문자열]
```

> `-EncodedCommand`를 사용했다는 사실만으로 악성 행위라고 판단할 수는 없다.

---

## 2) 원격 콘텐츠 다운로드 및 실행

PowerShell은 네트워크를 통해 데이터를 가져오는 기능을 제공한다.

예:

```text
원격 서버
    ↓
PowerShell
    ↓
콘텐츠 다운로드
    ↓
메모리 또는 파일로 처리
```

- 공격자는 이러한 기능을 악성 코드와 결합하여 원격에서 코드를 가져와 실행하는 방식으로 악용할 수 있다.
- 보안 분석에서는 PowerShell 프로세스의 **네트워크 연결, 다운로드 대상, 이후 실행된 명령** 등을 함께 확인해야 한다.

---

## 3) .NET Assembly를 메모리에 로드

- PowerShell은 .NET과 연동할 수 있기 때문에 .NET의 `System.Reflection` 기능 등을 사용할 수 있다.
- 바이트 배열 형태의 .NET 어셈블리를 메모리에 로드하는 방식이 존재한다.
- 이러한 기능은 정상적인 소프트웨어에서도 사용될 수 있지만, 악성 코드가 디스크에 실행 파일을 남기지 않고 코드를 실행하는 데 악용될 수 있다.

---

# EncodedCommand

### 기본 문법

```powershell
powershell.exe -EncodedCommand [Base64문자열]
```

### 특징

- 명령 자체를 Base64 형태로 전달
- PowerShell 명령줄에서 특수문자 및 따옴표 처리 문제를 줄일 수 있음
- 원본 명령이 명령줄에 평문으로 나타나지 않을 수 있음
- 보안 분석 시 Base64를 디코딩하여 실제 명령을 확인할 필요가 있음

```text
Base64 문자열
      ↓
디코딩
      ↓
PowerShell 명령
      ↓
실행
```

> Base64는 암호화가 아니라 인코딩(Encoding)이다.
> 따라서 누구나 디코딩하여 원본 데이터를 확인할 수 있다.

---

# .NET 호출 구조

PowerShell은 .NET 클래스와 직접 연동할 수 있다.

```powershell
[System.Net.WebClient]
[System.Convert]
[System.Text.Encoding]
[System.Reflection.Assembly]
[Microsoft.Win32.Registry]
```

### 주요 .NET 클래스

| .NET 클래스 | 주요 용도 |
|---|---|
| `[System.Net.WebClient]` | 네트워크 리소스 접근 및 데이터 다운로드 |
| `[System.Convert]` | Base64 및 바이트 배열 등의 변환 |
| `[System.Text.Encoding]` | UTF-8, UTF-16LE 등의 문자 인코딩 처리 |
| `[System.Reflection.Assembly]` | .NET 어셈블리 로드 및 Reflection |
| `[Microsoft.Win32.Registry]` | Windows 레지스트리 접근 |

> 이러한 클래스는 모두 정상적인 시스템 관리 및 개발에도 사용될 수 있다.
> 따라서 특정 .NET 클래스의 사용만으로 악성 여부를 판단해서는 안 된다.

---

# IEX(Invoke-Expression)

- `IEX`는 `Invoke-Expression`의 별칭(Alias)
- 문자열을 PowerShell 표현식으로 해석하여 실행한다.

```powershell
IEX "Write-Host 'hello'"
```

위 코드는 문자열로 전달된 PowerShell 코드를 해석하여 실행한다.

### 파이프라인을 이용한 형태

```powershell
"Write-Host 'hello'" | IEX
```

### 특징

- 문자열 형태의 명령을 동적으로 실행할 수 있다.
- 정상적인 자동화에서도 사용할 수 있지만, 외부에서 가져온 문자열을 실행하는 코드와 결합될 경우 보안상 위험할 수 있다.
- 공격자는 명령을 동적으로 생성하거나 난독화하는 과정에서 사용할 수 있다.

> `IEX`를 사용하면 정적 분석에서 무조건 탐지되지 않는다는 의미는 아니다.
> 현대적인 보안 제품은 명령줄, 스크립트 내용, 프로세스 행위, AMSI, PowerShell 로그 등을 종합하여 탐지할 수 있다.

---

# 문자열을 이용한 동적 실행

```powershell
$a = 'IE'
$b = 'X'
& ($a + $b) $decoded
```

- 문자열을 조합하여 명령 이름을 동적으로 생성하는 방식
- 이러한 방식은 정상적인 스크립트에서도 사용할 수 있지만, 악성 스크립트에서는 명령어를 숨기거나 난독화하는 목적으로 사용될 수 있다.

---

# 원격 콘텐츠 처리 방식

| 방식 | 주요 동작 | 디스크 사용 여부 |
|---|---|---|
| `DownloadString()` | 원격 콘텐츠를 문자열로 가져옴 | 문자열 처리 자체는 디스크 저장이 필요하지 않음 |
| `DownloadFile()` | 원격 파일을 지정된 경로에 저장 | 디스크 저장 |
| `Invoke-WebRequest` | HTTP/HTTPS 요청을 통해 웹 리소스에 접근 | 사용 방식에 따라 달라짐 |

### DownloadString()

- 원격 데이터를 문자열 형태로 가져온다.
- 이후 다른 PowerShell 기능과 결합하여 해당 데이터를 처리할 수 있다.

### DownloadFile()

- 원격 파일을 로컬 디스크에 저장한다.
- 이후 별도의 실행 과정이 필요할 수 있다.

### Invoke-WebRequest

- HTTP/HTTPS를 통해 웹 리소스에 접근하는 PowerShell 명령
- 별칭으로 `iwr`을 사용할 수 있다.

```powershell
Invoke-WebRequest
iwr
```

> `DownloadString() + IEX`가 모든 공격에서 사용되는 것은 아니며, 특정 비율의 공격에서 사용된다는 식의 수치는 출처와 조사 범위를 확인하지 않으면 단정하기 어렵다.

---

# CLR과 IL 개념

## CLR(Common Language Runtime)

- **CLR(Common Language Runtime)**은 .NET 프로그램이 실행될 수 있도록 관리하는 실행 환경
- 메모리 관리, 예외 처리, 보안 관련 기능, 형 변환, JIT 컴파일 등의 기능을 제공한다.

## IL(Intermediate Language)

- C# 등의 .NET 언어로 작성된 소스 코드는 일반적으로 컴파일되어 **IL(Intermediate Language)** 형태의 코드와 메타데이터를 포함하는 어셈블리가 생성된다.
- 실행 시 CLR이 IL을 필요한 시점에 **JIT(Just-In-Time) 컴파일**하여 네이티브 코드로 변환하고 CPU에서 실행할 수 있도록 한다.

```text
C# 소스 코드
      ↓
   컴파일
      ↓
IL + 메타데이터
      ↓
.NET Assembly
      ↓
CLR
      ↓
JIT 컴파일
      ↓
네이티브 코드
      ↓
CPU 실행
```

### PowerShell과 .NET

- Windows PowerShell 5.1은 .NET Framework를 기반으로 동작한다.
- PowerShell은 .NET 클래스와 객체를 직접 사용할 수 있기 때문에 다양한 시스템 기능에 접근할 수 있다.

---

# Assembly.Load - 메모리 로드

- `[System.Reflection.Assembly]::Load()`는 바이트 배열 등의 데이터를 .NET 어셈블리로 메모리에 로드하는 기능을 제공한다.
- 정상적인 프로그램에서도 동적 어셈블리 로딩을 위해 사용할 수 있다.
- 보안 공격에서는 악성 .NET 어셈블리를 디스크에 별도의 DLL 파일로 저장하지 않고 메모리에 로드하는 방식에 악용될 수 있다.

```powershell
$bytes = [Convert]::FromBase64String($b64)
$asm = [Reflection.Assembly]::Load($bytes)
$type = $asm.GetType('CalcLauncher.Launcher')
$type.GetMethod('Main').Invoke($null, $null)
```

### 동작 구조

```text
Base64 데이터
      ↓
Base64 디코딩
      ↓
바이트 배열
      ↓
Assembly.Load()
      ↓
메모리에 .NET Assembly 로드
      ↓
Reflection을 이용한 타입/메서드 접근
```

> `Assembly.Load()`를 사용한다고 해서 DLL이나 EXE가 시스템에 "존재하지 않는 것"은 아니다.
> 해당 어셈블리가 디스크 파일이 아닌 다른 형태의 데이터로 제공되어 메모리에 로드될 수 있다는 의미이다.

---

# 파일리스(Fileless) 공격

- **파일리스 공격(Fileless Attack)**은 전통적인 파일 기반 악성코드와 달리 악성 행위의 일부 또는 주요 부분을 메모리, 운영체제의 정상 기능 또는 스크립팅 환경 등을 이용하여 수행하는 공격 방식
- 디스크에 악성 실행 파일을 직접 생성하지 않는 형태가 있을 수 있다.

### 대표적인 특징

```text
악성 데이터
    ↓
메모리 / 정상 시스템 도구
    ↓
코드 실행
    ↓
디스크에 전통적인 악성 실행 파일이 남지 않을 수 있음
```

### PowerShell과의 관계

PowerShell은 다음과 같은 이유로 파일리스 또는 파일 경량화 공격에 악용될 수 있다.

- 메모리에서 데이터를 처리할 수 있음
- .NET 기능을 직접 사용할 수 있음
- 네트워크 기능을 제공
- 스크립트를 통해 운영체제 기능을 자동화할 수 있음
- Windows 환경에 기본적으로 존재하는 경우가 많음

### 하지만 "파일리스 = 흔적 없음"은 아니다.

파일리스 공격이라고 해서 시스템에 흔적이 전혀 남지 않는 것은 아니다.

다음과 같은 흔적이 남을 수 있다.

```text
PowerShell 프로세스
      ↓
명령줄 인자
      ↓
PowerShell Script Block Logging
      ↓
AMSI 이벤트
      ↓
네트워크 연결
      ↓
Windows Event Log
      ↓
프로세스 생성 관계
```

따라서 파일 기반 탐지만으로 판단하기보다 **프로세스, 메모리, 이벤트 로그, 네트워크 및 PowerShell 관련 로그를 함께 분석하는 것**이 중요하다.

---

# PowerShell 보안 및 탐지

PowerShell은 정상적인 관리 도구이기 때문에 단순히 PowerShell이 실행되었다는 이유만으로 악성 행위라고 판단하기 어렵다.

## 주요 분석 요소

### 1) 프로세스 생성 관계

```text
Office 프로그램
      ↓
powershell.exe
```

처럼 비정상적인 부모-자식 관계가 발생하는지 확인한다.

### 2) 명령줄(Command Line)

- PowerShell 실행 시 전달된 인자 확인
- `-EncodedCommand` 등의 옵션 사용 여부 확인
- 비정상적으로 난독화된 명령 확인

### 3) PowerShell 로그

- Script Block Logging
- Module Logging
- Transcription

등의 로깅 기능을 활용할 수 있다.

### 4) AMSI

- **AMSI(Antimalware Scan Interface)**
- Windows에서 응용 프로그램과 스크립트가 안티멀웨어 제품과 연동할 수 있도록 제공하는 인터페이스
- PowerShell 등의 스크립트 기반 콘텐츠를 보안 제품이 검사하는 데 활용될 수 있다.

### 5) 네트워크 통신

```text
PowerShell
    ↓
외부 IP / URL 연결
    ↓
데이터 다운로드
```

- PowerShell이 외부 서버와 통신하는 경우 목적지, 프로토콜, 요청 내용 등을 함께 분석할 수 있다.

---

# PowerShell 악성 행위 분석 흐름

```text
PowerShell 실행 확인
        ↓
부모 프로세스 확인
        ↓
실행 사용자 확인
        ↓
명령줄 인자 확인
        ↓
EncodedCommand 여부 확인
        ↓
스크립트 내용 확인
        ↓
네트워크 통신 확인
        ↓
파일 및 레지스트리 변경 확인
        ↓
PowerShell / Windows 이벤트 로그 확인
        ↓
정상적인 관리자 작업인지 악성 행위인지 판단
```

---

# PowerShell 핵심 정리

```text
PowerShell
→ Microsoft가 개발한 명령줄 셸 + 스크립팅 언어
→ 객체 기반 파이프라인
→ .NET과 강력하게 연동
→ Windows 시스템 관리 및 자동화에 사용
→ Windows PowerShell 5.1은 Windows에 기본 제공
→ 최신 PowerShell 7은 별도 설치 가능
→ 스크립트 확장자는 .ps1

PowerShell ISE
→ Windows PowerShell용 GUI 스크립트 작성/디버깅 도구
→ powershell_ise.exe
→ PowerShell 7에서는 지원되지 않음

공격에 악용될 수 있는 기능
→ EncodedCommand
→ 원격 콘텐츠 다운로드
→ 동적 코드 실행
→ .NET Assembly 로드
→ 레지스트리 / WMI / 서비스 접근

주요 분석 요소
→ 프로세스 관계
→ 명령줄
→ PowerShell 로그
→ AMSI
→ 네트워크 통신
→ 파일 및 레지스트리 변경
```

> **핵심:** PowerShell 자체는 악성 도구가 아니라 정상적인 Windows 관리 및 자동화 도구이다. 보안 관점에서는 PowerShell의 존재 자체보다 **어떤 프로세스가 어떤 명령을 어떤 계정으로 실행했으며, 그 결과 어떤 시스템·파일·레지스트리·네트워크 활동이 발생했는지**를 종합적으로 분석하는 것이 중요하다.
