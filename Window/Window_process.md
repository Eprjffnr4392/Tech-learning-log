## 윈도우 주요 프로세스
* 프로그램: 하드 디스크 등에 저장되어 있는 실행코드
* 프로세스: 연속적으로 실행되고 있는 컴퓨터 프로그램의 작은 단위
* 스레드: 프로세스에서 작업의 최소 단위

## 세션
* 윈도우 내에 어플리케이션이 동작하기 위해서는 실행하고 있는 세션이 필요
* 윈도우의 세션은 일종의 사용자를 뜻하며 Session 0부터 시작
* 윈도우 Vista 이전 버전에서는 처음 로그인한 사용자를 Session 0으로 정의
* Vista 이후부터 Session 0은 단지 시스템 프로세스와 서비스 실행에서만 동작하는 공간으로 정의, 로그인 사용자에게 각 세션 번호 부여

- 프로세스 종류(Windows 7)
  1) System: 대부분의 커널 모드 스레드를 담당
  2) smss.exe
  Session Manager Process 의미
  새로 생성되는 세션을 담당

<img width="728" height="181" alt="Image" src="https://github.com/user-attachments/assets/fe3a6549-b132-477b-a85b-0590b78eed29" />

  csrss.exe 프로세스 시작
  wininit.exe에서 생성되는 Session 0을 초기화
  winlogon.exe에서 생성되는 Session 1 이상의 새 세션 초기화
  최소 2개 필요함 - 작업관리자에서도 두 개 관측 가능

<img width="753" height="185" alt="Image" src="https://github.com/user-attachments/assets/7d684fb0-73ff-400e-a619-6212a6a7b749" />
<img width="741" height="252" alt="Image" src="https://github.com/user-attachments/assets/826db41f-afd1-424a-92b3-e493f1992293" />

  System 프로세스, Wininit 프로세스와 같은 deth에서 실행되고 있는 것을 볼 수 있었다.


  * 의심스러운 동작 발견 시 ex) 프로세스 이름이 같은 악성 프로그램 의심 => 부모자식 관계를 본다.

  3) wininit.exe
     윈도우 초기화 시켜주는 역할
     Session 0에서 백그라운드로 실행
     하위에서 services.exe, lsass.exe, lsm.exe 동작
  4) taskhost.exe
     윈도우 작업을 위한 프로세스
     모든 DLL 기반 서비스나 그룹 서비스의 호스트를 제공
  5) lsass.exe
     로컬 보안 인증 서브시스템 서버 프로세스
     유저의 인증을 위한 프로세스
  6) csrss.exe
     Clinet/Server Run-Time Subsystem으로 윈도우 서브시스템을 위한 유저모드의 프로세스
     프로세스와 스레드 등을 관리하는 등의 역할
  7) services.exe
     서비스와 작업 스케줄을 관리하는 프로세스
  8) svchost.exe
      윈도우 서비스의 호스트 프로세스
      DLL을 이용한 서비스가 실행하도록 제공
      여러 개의 프로세스를 생성할 수 있음
