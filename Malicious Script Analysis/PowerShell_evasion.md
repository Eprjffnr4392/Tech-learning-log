## PowerShell 회피 기법 이론

### 왜 회피 기법이 필요한가

* 공격자가 PowerShell 스크립트를 그대로 실행하면 여러 방어선에 걸릴 수 있다.
  - 백신은 파일의 정적 시그니처와 악성코드 패턴 등을 검사해 알려진 악성 코드를 탐지한다.
  - EDR은 프로세스의 행위, 프로세스 관계, 명령줄, 파일 및 네트워크 활동 등을 실시간으로 감시한다.
  - AMSI는 PowerShell과 같은 애플리케이션이 처리하는 스크립트나 콘텐츠를 백신 엔진에 전달해 검사할 수 있도록 지원한다.
* 이러한 방어선을 피하기 위해 공격자는 난독화, AMSI 우회, Anti-Analysis 등의 기법을 조합할 수 있다.

### 난독화(Obfuscation) 개요

* 난독화: 코드를 의도적으로 알아보기 어렵게 변형하여 정적 탐지와 분석을 방해하는 기법
  - 원본 문자열을 다른 형태로 변환해 저장하고, 실행 시점에 원래 값으로 복원한다.
  - 단순 문자열 검색이나 시그니처 매칭만으로는 악성 행위를 판단하기 어렵게 만들 수 있다.
  - PowerShell 공격에서는 XOR, ASCII 배열, ROT 치환 등이 대표적인 난독화 방식으로 사용된다.

### XOR 난독화

* XOR(Exclusive OR)는 두 비트가 다를 때 `1`, 같을 때 `0`을 반환하는 논리 연산
  - 핵심 성질: `A ⊕ B ⊕ B = A`
  - 같은 값으로 두 번 XOR하면 원본을 복구할 수 있다.
  - 고정 키로 각 바이트를 XOR하여 변환하고, 실행 시 동일한 키로 다시 XOR하여 원래 값을 복구할 수 있다.
  - 단일 바이트 키(예: `0x41`)와 여러 바이트가 순환되는 멀티바이트 키가 있다.

```powershell
# 단일바이트키 XOR 예시
$key = 0x41
$enc = @(8,36,49,17,32,78,109,119,...)
$decoded = ($enc | ForEach-Object { [char]($_ -bxor $key) }) -join ''
IEX $decoded
```

### ASCII chr() 변환

* 문자열의 각 문자를 ASCII 코드 값(정수)으로 분해해 배열로 저장하는 방식이다.
  - 예: `'IEX'` → `[73, 69, 88]`로 저장
  - 실행 시 `[char]` 캐스팅으로 다시 문자로 변환한 뒤 `-join`으로 합쳐 원본을 복원한다.
  - 스크립트 안에서 특정 문자열을 직접 확인하기 어려워져 단순 문자열 기반 탐지를 방해할 수 있다.
  - 다만 숫자 배열 자체나 복원 과정, 실행 행위 등을 통해 탐지할 수 있으므로 난독화만으로 탐지를 완전히 회피할 수 있는 것은 아니다.

```powershell
$arr = @(73, 69, 88)
$str = ($arr | ForEach-Object { [char]$_ }) -join ''
# $str = 'IEX'
```

### ROT 치환 암호

* ROT(Rotation)는 알파벳을 일정 값(N)만큼 이동시키는 단순 치환 방식
  - 오래된 치환 방식 중 하나이며, 대표적으로 ROT13이 있다.
  - 예: ROT13에서 `A → N`, `B → O`와 같이 13칸 이동한다.
  - 공격자는 문자열마다 서로 다른 N값(2, 3, 4, 12, 15, 21 등)을 사용해 특정 ROT 값만을 대상으로 하는 탐지를 방해할 수 있다.
  - GitLab PAT 토큰과 같은 문자열 형태의 IoC를 숨기는 데에도 사용할 수 있지만, ROT 자체는 보안성이 없는 단순 치환 방식이므로 복원이나 탐지가 어렵지 않다.

### Dynamic Execution 개요

* Dynamic Execution은 스크립트 실행 도중에 새로운 코드를 만들거나 로드하여 실행하는 기법이다.
  - `IEX`와 유사하게 실행 시점에 코드가 결정될 수 있지만, PowerShell 코드뿐만 아니라 C# 코드 등을 런타임에 컴파일하거나 로드하여 실행하는 방식도 포함할 수 있다.
  - PowerShell의 `Add-Type` 명령은 C# 코드를 런타임에 컴파일하여 .NET 타입으로 사용할 수 있도록 한다.
  - 이 방식을 이용하면 Win32 API를 직접 호출하는 코드를 구성할 수 있다.

### Add-Type과 P/Invoke

* `Add-Type`과 P/Invoke
  - `Add-Type`은 PowerShell에서 C# 코드를 런타임에 컴파일하여 .NET 타입으로 사용할 수 있게 한다.
  - `DllImport` 속성을 사용하면 Windows의 DLL에 포함된 Win32 API를 PowerShell에서 직접 호출할 수 있다.
  - 이러한 방식으로 운영체제의 네이티브 API를 호출하는 것을 P/Invoke(Platform Invoke)라고 부른다.

```powershell
$code = @'
using System;
using System.Runtime.InteropServices;
public class Win32 {  
[DllImport("kernel32")]  
public static extern IntPtr VirtualAlloc(    
IntPtr addr, uint size, uint type, uint protect);
}
'@
Add-Type -TypeDefinition $code
[Win32]::VirtualAlloc(0, 4096, 0x3000, 0x40)
```

### VirtualAlloc과 CreateThread

* 셸코드를 메모리에서 실행하는 공격 기법에서는 메모리 할당과 실행 흐름을 연결하기 위해 여러 Win32 API가 사용될 수 있다.

| API | 역할 |
|---|---|
| `VirtualAlloc` | 프로세스의 가상 메모리 영역을 할당 |
| `CreateThread` | 지정된 시작 주소를 기준으로 새 스레드를 생성 |
| 대안 | 함수 포인터로 직접 호출(콜백 캐스팅)해 `CreateThread` 없이 실행하기도 함 |

* 메모리의 실행 권한이 중요하다.
  - 일반적인 데이터 메모리는 실행 권한이 없을 수 있기 때문에, 실행 가능한 메모리 영역이 필요하다.
  - 공격 기법에서는 `RWX`(읽기, 쓰기, 실행) 권한을 가진 메모리를 사용하는 경우가 있지만, 이러한 메모리 권한 조합 자체가 보안 제품에서 의심스러운 행위로 탐지될 수 있다.
  - 따라서 실제 환경에서는 메모리 보호 속성 변경, 실행 권한이 있는 메모리 할당 등의 행위 자체가 중요한 탐지 지표가 될 수 있다.

### AMSI

* AMSI(Antimalware Scan Interface)는 Windows가 제공하는 악성코드 탐지 연계 인터페이스이다.
  - 애플리케이션과 백신 엔진(Defender·V3 등) 사이에서 콘텐츠를 전달하여 검사를 수행할 수 있도록 한다.
  - PowerShell·WScript·Office VBA 등의 애플리케이션이 처리하는 스크립트나 콘텐츠를 백신에 검사 요청할 수 있다.
  - 단순한 파일 스캔뿐만 아니라 메모리에서 처리되는 스크립트 콘텐츠도 검사 대상이 될 수 있다.
  - 파일리스 공격이나 스크립트 기반 공격에 대응하기 위한 Windows의 보안 기능 중 하나이다.

### AmsiScanBuffer 함수

* AMSI의 핵심 함수 중 하나는 `amsi.dll`의 `AmsiScanBuffer` 함수이다.
  - PowerShell과 같은 AMSI 연계 애플리케이션이 콘텐츠를 검사하도록 요청할 때 사용될 수 있다.
  - 검사 결과는 `AMSI_RESULT`를 통해 반환된다.
  - 검사 결과가 악성으로 판단되면 호출한 애플리케이션이 해당 콘텐츠의 실행을 차단하거나 추가적인 처리를 수행할 수 있다.

```text
HRESULT AmsiScanBuffer(
  HAMSICONTEXT amsiContext,
  PVOID buffer,            // 검사 대상 데이터
  ULONG length,            // 데이터 길이
  LPCWSTR contextName,
  AMSISESSION amsiSession,
  AMSI_RESULT *result      // 검사 결과 반환
);
```

### AMSI 우회 3가지 기법

| 기법 | 원리 | 실전 활용도 |
|---|---|---|
| Consumer Unhooking | AMSI를 사용하는 소비자(PowerShell 등)의 함수 자체를 변경해 AMSI 호출을 막음 | 낮음 |
| Memory Patching | `amsi.dll`의 `AmsiScanBuffer` 함수를 메모리에서 직접 고쳐 AMSI 검사 흐름을 방해 | 가장 자주 사용됨 |
| Provider Pathcing | AMSI 검사를 실제 수행하는 백신 제공자 측을 무력화 | 거의 사용 x |

* 위와 같은 분류는 AMSI 우회 기법을 설명하기 위한 개념적인 분류이다.
* 실제 공격에서는 Windows 버전, PowerShell 버전, 보안 제품의 탐지 방식 등에 따라 사용되는 기법과 성공 여부가 달라질 수 있다.

### Memory Patching 원리

* Memory Patching은 `AmsiScanBuffer`와 같은 메모리에 로드된 함수의 실행 코드를 변경하여 원래의 검사 흐름을 방해하는 방식이다.
  - `VirtualProtext`로 `amsi.dll`의 함수 메모리에 쓰기 권한을 부여
  - 함수 시작 지점에 특정 바이트를 덮어써 AMSI 함수의 정상적인 검사 흐름을 변경
  - 예시로 다음 바이트를 사용하는 방식이 알려져 있다.
    - `0xB8, 0x57, 0x00, 0x07, 0x80, 0xC3`
    - `mov eax, 0x80070057; ret`
  - 이 경우 `AmsiScanBuffer`가 정상적인 검사 과정을 수행하지 않고 특정 오류 코드와 함께 반환되도록 동작을 변경한다.
  - 다만 이러한 방식은 메모리 코드 변조 자체가 EDR이나 보안 제품에서 강하게 탐지될 수 있으며, Windows 및 보안 제품의 버전에 따라 동작 여부가 달라질 수 있다.

### Anti-Analysis 4가지

* Anti-Analysis는 자동화된 분석(샌드박스)과 수동 분석(리버스 엔지니어링)을 모두 방해하기 위한 기법이다.

| 회피 | 구체 기법 |
|---|---|
| 환경 탐지 | CPU 코어 수, 메모리 용량, 도메인 가입 여부 등의 환경 정보를 확인하여 VM이나 샌드박스 환경을 추정 |
| 시간 기반 회피 | 긴 `Sleep`으로 샌드박스의 분석 시간을 초과하도록 유도하거나 시간 관련 동작을 이용해 자동화 분석 환경을 추정 |
| 데드코드 삽입 | 실제로 호출되지 않는 함수나 코드를 대량으로 추가하여 정적·수동 분석을 어렵게 만듦 |
| 표적 조건 검사 | 특정 사용자명, IP, MAC 등의 조건에서만 동작하도록 하여 분석 환경에서는 악성 행위를 나타내지 않도록 구성 |

* Anti-Analysis 기법은 악성코드가 분석 환경에서 자신의 행위를 숨기기 위해 사용할 수 있지만, 환경 정보 확인이나 지연 실행과 같은 행위 자체가 EDR의 탐지 지표가 될 수도 있다.
