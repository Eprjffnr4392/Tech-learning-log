## ISE 디버깅과 Base64/Encoded Command 실습

### PowerShell ISE 디버깅 기본
* PowerShell ISE는 Windows에 기본 탑재된 통합 스크립팅 환경으로, 스크립트 실행을 원하는 시점에 멈추고 변수를 확인할 수 있다.
단축기      동작
F9          중단점(Breakpoint) 설정, 해제
F5          스크립트 실행 시작
F10         한 줄씩 실행(Step Over)
F11         함수 내부로 진입(Step Into)
Shift+F11   현재 함수를 빠져나옴(Step Out)
중단점 도달 시 하단 콘솔에서 변수명 입력하면 실시간 값을 확인할 수 있다.

### EncodedCommand 개요
powershell.exe에-EncodedCommand옵션과Base64문자열을넘겨명령을실행하는방식입니다.
–인코딩규격은UTF-16LE(Unicode Little Endian)이므로UTF-8로인코딩하면실행되지않습니다.
–PowerShell의[System.Text.Encoding]::Unicode 클래스가UTF-16LE에해당합니다.
–특수문자나따옴표이스케이프문제를회피할수있어공격에서매우자주사용됩니다.
```bash
powershell.exe -EncodedCommand [Base64문자열]
```

