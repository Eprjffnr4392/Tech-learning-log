## PowerShell
* 마이크로소프트가 개발한 명령줄 셸(Shell)이자 스크립팅 언어
  - .NET 프레임워크 위에서 동작해 Windows 시스템 전체를 코드로 제어할 수 있다
  - cmd는 단순 명령 실행기이지만, PowerShell은 .NET 객체를 직접 다루는 완전한 프로그래밍 언어
  - Windows 7 이후 모든 Windows에 기본 탑재되어 별도의 설치 필요 x
* 스크립트 파일의 확장자는 .ps1 사용
  - 숫자 1은 PowerShell 엔진 버전 1을 의미하며, 현재 버전 7까지 나왔지만 호환성 유지를 위해 그대로 사용

### 왜 공격자는 PowerShell을 쓰는가
1) 모든 Windows에 존재
   Windows 7 이후 기본 탑재로 어느 시스템에서든 실행 가능
2) 신뢰된 도구
   정상 관리 도구이므로 백신, EDR이 프로세스 자체를 차단하기 어려움
3) 파일리스 실행 가능
   DownloadString-Assembly.Load로 디스크에 흔적 없이 메모리에서 실행
4) 실행 정책 우회 용이
   기본 정책은 외부 스크립트를 막지만, 옵션 하나로 우회
5) 탐지 우회 기법 풍부
   Base64 인코딩, 문자열 분할, AMSI 우회 기법이 공개적으로 알려짐

### 관리자 도구와의 경계 모호성
* PowerShell은 기업 IT 관리자가 가장 많이 사용하는 자동화 도구이기도 함
  - AD(Active Directory)를 통해 사용자, 컴퓨터, 권한을 중앙 관리하는 데 PowerShell이 표준
  - 관리자도 편의를 위해 파일리스 형태로 관리 스크립트를 실행하는 경우가 다수
  - 결과적으로 정상 관리 활동과 악성 활동이 겉모습에서 거의 구분되지 않음
    => 이 경계 모호성이 PowerShell을 공격자에게 가장 최적화된 도구로 만듦

### PowerShell ISE
PowerShell ISE(Intergrated Scripting Environment)는 PowerShell 스크립트를 작성, 실행, 디버깅할 수 있는 통합 개발 환경
- 단순 콘솔이 아닌 스크립트 편집, 실행, 디버깅까지 가능한 GUI 도구
- Windows에 기본 탑재, powershell_ise.exe로 실행
- 악성 스크립트 분석 실습에서 가장 많이 사용하는 도구

### ISE 디버깅 기본
* ISE는 스크립트 실행을 원하는 시점에 멈추고 변수 상태를 확인할 수 있는 디버깅 기능 제공
  - F9: 코드 라인에 중단점(Breakpoint) 설정, 해제
  - F5: 스크립트 실행 시작
  - F10: 한 줄씩 실행 (Step Over) - 함수 호출을 하나의 단위로 넘김
  - F11: 함수 내부로 진입 (Step Into)
  - Shift + F11: 현재 함수를 빠져나옴 (Step Out)
중단점 도달 시 자동으로 정지, 콘솔에서 변수명을 입력하면 실시간 값을 확인할 수 있다.

### 공격자의 3가지 주요 실행 방식
* 공격자는 탐지를 피하기 위해 다음 세 가지 실행 방식을 조합하거나 변형
  - EncodedCommand: 명령 전체를 Base64로 인코딩하여 전달, 특수문자, 따옴표 문제 회피
  - DownloadString + IEX: 원격 URL에서 코드 문자열을 받아 메모리에서 즉시 실행 (파일리스)
  - Assembly.Load: 바이트 배열을 .NET 어셈블리로 메모리에 로드해 실행 (DLL,EXE 없음)
 
