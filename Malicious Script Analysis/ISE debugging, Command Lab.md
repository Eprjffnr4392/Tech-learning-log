## ISE 디버깅과 Base64/Encoded Command 실습

### PowerShell ISE 디버깅 기본

* PowerShell ISE는 Windows에 기본 탑재된 통합 스크립팅 환경으로, 스크립트 실행을 원하는 시점에 멈추고 변수의 현재 값을 확인할 수 있다.
* 중단점(Breakpoint)을 설정하면 특정 코드에서 실행을 일시 중지하여 변수와 실행 흐름을 확인할 수 있다.

| 단축키 | 동작 |
|---|---|
| F9 | 중단점(Breakpoint) 설정, 해제 |
| F5 | 스크립트 실행 시작 |
| F10 | 한 줄씩 실행(Step Over) |
| F11 | 함수 내부로 진입(Step Into) |
| Shift+F11 | 현재 함수를 빠져나옴(Step Out) |

* 중단점에 도달하면 하단 콘솔에서 변수명을 입력하여 해당 시점의 변수 값을 확인할 수 있다.

<img width="1117" height="563" alt="image" src="https://github.com/user-attachments/assets/55f20b2a-0b83-43b6-a977-9f32c98732dc" />

### EncodedCommand 개요

* `powershell.exe`에 `-EncodedCommand` 옵션과 Base64 문자열을 넘겨 명령을 실행하는 방식이다.
  - 인코딩 규격은 UTF-16LE(Unicode Little Endian)이므로 UTF-8로 인코딩한 값을 그대로 전달하면 정상적으로 해석되지 않는다.
  - PowerShell의 `[System.Text.Encoding]::Unicode` 클래스가 UTF-16LE에 해당한다.
  - 특수문자나 따옴표 등의 셸 해석 문제를 줄일 수 있어 관리 작업뿐만 아니라 공격에서도 관찰된다.
  - Base64는 암호화가 아니라 인코딩이므로 누구나 원래 문자열로 디코딩할 수 있다.

```bash
powershell.exe -EncodedCommand [Base64문자열]
```

### 디버깅 환경 검증

* `p1.ps1`을 이용해 ISE 디버깅이 정상적으로 동작하는지 확인하기 위해 스크립트의 2번째 줄에 커서를 둔 후 F9 키를 눌러 중단점을 설정한다.
* F5 키를 눌러 스크립트를 실행하면 설정한 중단점에서 실행이 정지하는 것을 확인한다.
* 콘솔에 `$b64`를 입력하여 변수값을 확인한다.

<img width="1117" height="563" alt="image" src="https://github.com/user-attachments/assets/55f20b2a-0b83-43b6-a977-9f32c98732dc" />

### 실습 1 - Base64 인코딩

* `calc.exe`를 실행하는 PowerShell 명령을 UTF-16LE Base64로 인코딩하는 실습을 진행한다.
* Windows PowerShell ISE에서 새 파일을 생성한 후 아래의 코드를 입력하여 `test.ps1`로 저장한다.

```bash
# 목표: 아래명령을UTF-16LE Base64로인코딩
Start-Process calc.exe
# 인코딩코드예시
$cmd = 'Start-Process calc.exe'
$bytes = [System.Text.Encoding]::Unicode.GetBytes($cmd)
$b64 = [Convert]::ToBase64String($bytes)
Write-Host $b64
```

* F5로 실행한다.

<img width="1102" height="690" alt="image" src="https://github.com/user-attachments/assets/987245ae-23be-493b-8539-1641db449c26" />

* cmd 창에서 Powershell.exe `-EncodeCommand` 뒤에 Base64 문자열을 붙여 실행하면 계산기 프로그램이 실행되는 것을 확인한다.

<img width="1261" height="767" alt="image" src="https://github.com/user-attachments/assets/7dc3ee5a-6aa4-41de-8452-cc9929364b96" />

### 실습 2 - 디코딩 후 IEX 실행

* 실습 1에서 만든 Base64 문자열을 스크립트 내에서 디코딩한 뒤 `IEX`로 실행한다.
  - `FromBase64String`으로 Base64 → 바이트 배열로 변환한다.
  - `Encoding.Unicode.GetString`으로 바이트 배열 → 원본 문자열로 복원한다.
  - `IEX`로 문자열을 코드로 실행하여 계산기를 실행한다.

```bash
$b64 = 'UwB0AGEAcgB0AC0AUAByAG8AYwBlAHMAcwAgAGMAYQBsAGMALgBlAHgAZQA='
$decoded = [System.Text.Encoding]::Unicode.GetString(
[Convert]::FromBase64String($b64))
IEX $decoded
```

* 계산기 프로그램 실행을 확인한다.

<img width="1077" height="792" alt="image" src="https://github.com/user-attachments/assets/6cbf33c1-a974-4b9e-9fba-0bc1236ecc39" />

### 실습 3 - 변수 없이 한 줄로 작성

* 실습 2의 코드를 중간 변수 없이 한 줄로 압축해 작성한다.
  - 중간 변수(`$decoded`)를 제거하면 코드가 짧아지지만, 이것만으로 로그·모니터링 도구의 탐지를 피할 수 있는 것은 아니다.
  - 공격자가 짧은 한 줄 페이로드를 만드는 대표적인 방식 중 하나이다.

```bash
IEX ([System.Text.Encoding]::Unicode.GetString(  
[Convert]::FromBase64String(    
'UwB0AGEAcgB0AC0AUAByAG8AYwBlAHMAcwAgAGMAYQBsAGMALgBlAHgAZQA=')))
```

* 계산기 프로그램 실행을 확인한다.

<img width="1089" height="787" alt="image" src="https://github.com/user-attachments/assets/653ecad3-583d-48ad-856c-bbcb5266f494" />

### 실습 4 - 파이프 형태로 변경

* 실습 3에서 `IEX`를 파이프(`|`)를 통해 연결하여 동일한 동작을 구현한다.
  - 파이프 좌측의 문자열이 우측 `IEX`로 인자값으로 전달된다.
  - 이 파이프 방식은 코드가 짧아 보이고 자연스러운 형태로 작성할 수 있어 실제 공격에서도 자주 관찰된다.

```bash
[System.Text.Encoding]::Unicode.GetString(
[Convert]::FromBase64String(
'UwB0AGEAcgB0AC0AUAByAG8AYwBlAHMAcwAgAGMAYQBsAGMALgBlAHgAZQA=')) | IEX
```

* 계산기 프로그램 실행을 확인한다.

<img width="1044" height="760" alt="image" src="https://github.com/user-attachments/assets/26473697-1887-4f1b-b842-5cbfa8e3414c" />

### 실습 5 - IEX 문자열 은닉

* 실습 2와 동일한 동작을 하되, 코드 어디에도 `IEX` 문자열이 직접 보이지 않게 작성한다.
  - `IE`와 `X`를 분리해 변수에 담고 `&` (Call) 연산자로 합쳐 호출한다.
  - 스크립트 내에 `'IEX'` 문자열이 직접 존재하지 않아 단순한 IEX 키워드 기반 탐지를 방해할 수 있다.
  - 다만 실제 보안 제품은 문자열 하나만을 기준으로 판단하지 않고, 명령 실행 흐름과 프로세스 행위, AMSI, EDR 등의 다양한 정보를 함께 활용할 수 있다.

```bash
$a = 'IE'; $b = 'X'
$decoded = [System.Text.Encoding]::Unicode.GetString(
  [Convert]::FromBase64String(
    'UwB0AGEAcgB0AC0AUAByAG8AYwBlAHMAcwAgAGMAYQBsAGMALgBlAHgAZQA='))
& ($a + $b) $decoded
```

* 계산기 프로그램 실행을 확인한다.

<img width="1041" height="796" alt="image" src="https://github.com/user-attachments/assets/aeec7e89-8a44-4715-9b51-dc52aaff63f8" />
