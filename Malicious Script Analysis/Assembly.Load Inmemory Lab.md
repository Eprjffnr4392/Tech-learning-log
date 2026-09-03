## Assembly.Load 인메모리 실행 실습

### CLR과 IL

* CLR(Common Language Runtime)은 .NET 프로그램을 실제로 실행해주는 런타임 엔진이다.
  - C 언어: 소스 → 컴파일 → 네이티브 코드 → 바로 실행한다.
  - C# 언어: 소스 → 컴파일 → IL(중간 코드) → CLR이 실행 가능한 코드로 변환 → 실행한다.
  - 즉, C#으로 만든 EXE 파일에는 .NET 어셈블리와 IL 형태의 중간 코드가 포함된다.
  - 이 IL 바이트를 CLR에 메모리에서 로드하면 파일을 별도로 생성하지 않고 .NET 프로그램을 실행할 수 있다.

### Assembly.Load 문법

* `[System.Reflection.Assembly]::Load()`는 바이트 배열을 .NET 어셈블리로 메모리에 로드하는 함수이다.
  - 1) Base64 → 바이트 배열 변환
  - 2) 바이트 배열을 어셈블리로 메모리 로드
  - 3) `GetTypes`로 어셈블리 내부의 모든 타입 조회
  - 4) 원하는 클래스 타입 선택
  - 5) `Main` 메서드를 `Invoke`로 실행

```bash
$bytes = [Convert]::FromBase64String($b64)
$asm = [Reflection.Assembly]::Load($bytes)
$types = $asm.GetTypes()$type =
$types | Where-Object { $_.FullName -eq 'CalcLauncher.Launcher' }
$type.GetMethod('Main').Invoke($null, $null)
```

### 실습 환경 준비

* 실습에 사용할 파일과 도구를 준비하기 위해 ISE를 실행하고, 파일 열기로 `p1.ps1`을 로드한다.
  - 이 스크립트는 Base64로 인코딩된 `$b64` 변수와 `Assembly.Load`를 이용한 실행 코드를 포함한다.
  - 첫 줄에 F9로 중단점을 걸고 F5로 실행한 뒤, F11로 한 줄씩 진행하며 콘솔에 변수명을 입력해 값을 확인한다.

<img width="1062" height="578" alt="image" src="https://github.com/user-attachments/assets/ab48f95d-514e-4e3c-98e4-1af51b789bfd" />

### 실습 1 = $b64 확인

* `p1.ps1`의 `$b64` 변수에 담긴 Base64 문자열이 실제로 무엇인지 확인한다.
  - `p1.ps1`의 1번째 줄(`$bytes = ...`)에 F9로 중단점을 설정한다.
  - F5로 실행하면 첫 줄(`$b64 = ...`)이 처리된 뒤 중단점에서 정지한다.
  - 하단 콘솔에 `$b64`를 입력하면 매우 긴 Base64 문자열이 출력된다.
  - 문자열이 `'TVqQAAMAAAAEAAA...'`로 시작하는 것을 확인할 수 있으며, 이는 `MZ` 매직 넘버가 Base64로 인코딩된 결과이다.

```bash
# 콘솔에입력하여확인
$b64
```

* `$b64` 확인
<img width="1090" height="722" alt="스크린샷 2026-09-03 232927" src="https://github.com/user-attachments/assets/e92468f5-8c93-423e-952b-6bc3a5480e3d" />


## 실습 2 - 바이트 배열 확인 (MZ 매직)

* `$bytes` 변수가 채워진 뒤, 콘솔에서 실제 바이트 값을 확인하기 위해 F11 키를 눌러 `p1.ps1`의 2번째 줄(`$bytes = [Convert]::FromBase64String($b64)`)을 실행한다.
  - 콘솔에 `$bytes.Length`를 입력하면 바이트 배열의 크기가 출력된다.
  - 콘솔에 `$bytes[0..3]`을 입력하면 첫 4바이트가 출력된다.
  - 첫 두 바이트가 `77`과 `90`이며, ASCII로는 `M`과 `Z`로 PE 파일의 매직 넘버이다.
  - 즉, `$b64`에는 PE 형식의 EXE 또는 DLL 데이터가 Base64로 인코딩되어 있을 가능성이 있음을 확인할 수 있다.

```bash
# 콘솔에입력해확인
$bytes.Length       # 전체바이트크기
$bytes[0..3]        # 첫4바이트→ 77(M) 90(Z) 144 0 ...
```

* 확인

<img width="1085" height="789" alt="image" src="https://github.com/user-attachments/assets/7887e294-f53c-4b02-a553-5546c8a96afd" />

### 실습 3 - 어셈블리 로드 확인

* `Assembly.Load`를 실행한 뒤, 콘솔에서 어셈블리 정보를 확인하기 위해 F11 키를 눌러 3번째 줄(`$asm= [Reflection.Assembly]::Load($bytes)`)을 실행한다.
  - 이 순간 디스크에 별도의 어셈블리 파일을 생성하지 않고 메모리에 .NET 어셈블리를 로드할 수 있다.
  - 콘솔에 `$asm.FullName`을 입력하면 `'CalcLauncher, Version=0.0.0.0, Culture=neutral, PublicKeyToken=null'`이 출력된다.
  - 콘솔에 `$asm.Location`을 입력하면 아무 값도 출력되지 않거나 빈 문자열이 반환될 수 있다.
  - 메모리에서 바이트 배열로 로드된 어셈블리에서는 `Location`이 비어 있는 것이 일반적이다.
  - 다만 `Location` 하나만으로 어셈블리의 로드 방식을 절대적으로 판단하기보다는 로드 방식과 실행 프로세스의 다른 정보도 함께 확인해야 한다.

```bash
# 콘솔에입력해확인
$asm.FullName    # → CalcLauncher, Version=0.0.0.0, Culture=neutral, ...
$asm.Location    # → (빈값) 디스크경로가존재하지않음
```

* `$asm.FullName`과 `$asm.Location`을 콘솔에 입력한다.
* `$asm.Location`이 빈 값으로 출력되는 것을 통해 파일 경로가 연결된 일반적인 어셈블리 로드와 다른 형태로 로드되었음을 확인할 수 있다.

<img width="1087" height="725" alt="image" src="https://github.com/user-attachments/assets/ee507516-b123-4f5c-942e-f77a0efd9815" />

### 실습 4 - 진입점 클래스 찾기

* 어셈블리 안에는 여러 클래스가 있을 수 있고, 그중 `Main` 메서드를 가진 클래스가 프로그램의 진입점 역할을 할 수 있다.
* EXE를 직접 실행하면 운영체제와 .NET 실행 환경이 진입점을 찾아 실행하지만, `Assembly.Load`로 어셈블리를 메모리에만 올린 경우에는 코드에서 원하는 타입과 메서드를 직접 찾아 호출할 수 있다.
* F11로 `GetTypes`와 `Where-Object`까지 실행한 뒤, 콘솔에서 어떤 타입이 담겼는지 확인한다.
  - F11 키를 눌러 `try` 블록의 `$types = $asm.GetTypes()`를 실행한다.
  - 콘솔에 `$types`를 입력하면 어셈블리 내부 클래스 목록이 출력된다. 이 예제는 학습용이라 `Launcher` 클래스 하나만 표시된다.
  - F11로 계속 진행해 `Where-Object`로 `CalcLauncher.Launcher`를 선택하는 라인까지 실행한다.
  - 콘솔에 `$type`을 입력하면 `CalcLauncher.Launcher` 타입 정보가 출력된다.
  - 실행하려는 클래스 이름은 어셈블리 제작 시 정해지므로, 코드에서는 해당 이름을 미리 알고 지정할 수 있다.

```bash
# 콘솔에입력해확인
$types           # 어셈블리내모든타입목록
$type            # 선택된CalcLauncher.Launcher 타입

```

* `$type`을 콘솔에 입력하여 `CalcLauncher.Launcher` 타입 정보를 확인한다.

<img width="1105" height="796" alt="image" src="https://github.com/user-attachments/assets/d33f7caa-3ae9-4552-90f6-21d1320c31b5" />

### 실습 5 - Main 메서드 실행

* F11로 `GetMethod`와 `Invoke`까지 진행해 실제로 프로그램이 실행되는 것을 확인한다.
  - F11 키를 눌러 `$method = $type.GetMethod('Main')` 라인을 실행한다.
  - 콘솔에 `$method`를 입력하면 `Main` 메서드의 정보가 출력된다.
  - F11로 다음 줄인 `$method.Invoke($null, $null)`을 실행하면 계산기가 화면에 뜬다.
* 이 실습에서는 `Assembly.Load`를 통해 어셈블리를 메모리에 로드한 후 메서드를 직접 호출하므로, 실행 과정에서 새로운 EXE 파일이 디스크에 생성되지 않는 것을 확인할 수 있다.

```bash
# 콘솔에입력해확인
$method    # GetMethod('Main') 결과— Main 메서드정보
```

* F11 키를 눌러 `$method = $type.GetMethod('Main')` 라인을 실행한 후 콘솔에 `$method`를 입력하면 `Main` 메서드의 정보가 출력된다.

<img width="1093" height="793" alt="image" src="https://github.com/user-attachments/assets/f1f665c1-c7ae-4808-b143-c73164354419" />

* F11로 다음 줄인 `$method.Invoke($null, $null)`을 실행하면 계산기가 화면에 뜨는 것을 확인한다.

<img width="1114" height="812" alt="image" src="https://github.com/user-attachments/assets/230a95aa-ea59-4ee5-bb8e-4f7fb1a824c0" />
