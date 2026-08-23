# Linux Command 정리

Linux 학습 과정에서 자주 사용하는 기본 명령어와 시스템 관리 명령어를 정리한 문서

---

## 1. Vim 편집기

Linux에서 파일을 생성하거나 수정할 때 사용하는 대표적인 편집기

### Vim 실행

```bash
vim 파일명
```
파일이 존재하지 않으면 새로운 파일을 생성

### Vim 주요 모드

- **Normal Mode** : 명령 실행
- **Insert Mode** : 내용 입력
- **Command Mode** : 저장, 종료 등의 명령

### 주요 명령어

```text
i       입력 모드
Esc     일반 모드
:w      저장
:q      종료
:wq     저장 후 종료
:q!     저장하지 않고 종료
dd      한 줄 삭제
yy      한 줄 복사
p       붙여넣기
u       실행 취소
```
## 2. Linux 구조와 Shell의 이해

Linux는 사용자, Shell, Kernel, Hardware의 구조로 이해할 수 있다.

- **사용자(User)** : 명령을 입력
- **Shell** : 사용자의 명령을 해석하여 Kernel에 전달
- **Kernel** : CPU, 메모리, 디스크 등의 하드웨어 자원을 관리
- **Hardware** : 실제 컴퓨터 장치

### Shell 확인

```bash
echo $SHELL
```
현재 사용 중인 Shell을 확인할 수 있다.

### 명령어 위치 확인

```bash
which 명령어
```
예:

```bash
which ls
```
## 3. Linux 기본 명령어

### 시스템 종료 및 재시작

#### 시스템 즉시 종료

```bash
shutdown -h now
halt
init 0
```
init 0은 Run Level 0으로 전환하여 시스템을 종료

### 시스템 즉시 재시작
```bash
shutdown -r now
reboot
init 6
```
init 6은 Run Level 6으로 전환하여 시스템을 재시작

### 현재 위치 확인
```bash
pwd
```
### 디렉터리 내용 확인
```bash
ls
ls -l
ls -a
ls -al
```
- **-l** : 상세 정보 출력
- **-a** : 숨김 파일 포함
  
### 디렉터리 이동
```bash
cd 디렉터리
cd ..
cd ~
cd /
```
- **..** : 상위 디렉터리
- **~** : 현재 사용자의 홈 디렉터리
- **/** : 최상위 디렉터리

### 화면 정리
```bash
clear
```

## 파일과 디렉터리의 이해
### 파일 생성
```bash
touch test.txt
```
빈 파일을 생성
```bash
vim test.txt
```
Vim을 이용하여 파일을 생성하고 내용을 작성할 수 있다.

### 숨김 파일
Linux에서는 파일 이름 앞에 .을 붙이면 숨김 파일로 취급
```bash
touch .test
```
숨김 파일 확인:
```bash
ls -a
```

### 디렉터리 생성
```bash
mkdir test
mkdir -p test/a/b
```
-p 옵션을 사용하면 필요한 상위 디렉터리도 함께 생성

### 파일 및 디렉터리 복사
```bash
cp test.txt backup.txt
cp -r test backup
```
- **cp** : 파일 복사
- **-r** : 디렉터리를 하위 내용까지 복사

### 파일 및 디렉터리 이동
```bash
mv test.txt /tmp/
```
파일 이동뿐만 아니라 파일 이름 변경에도 사용할 수 있다.
```bash
mv old.txt new.txt
```

### 파일 삭제
```bash
rm test.txt
rm -r test
rm -rf test
```
- **-r** : 디렉터리 삭제
- **-f** : 삭제 여부를 묻지 않고 강제 삭제

### 파일 내용 확인
```bash
cat test.txt
less test.txt
head test.txt
tail test.txt
```
### 로그 확인에 자주 사용하는 명령어
```bash
tail -f /var/log/messages
```

## 파일시스템 이해와 디스크 관리
### Linux 주요 디렉터리
| 디렉터리 | 설명               
| ------- | ---------------- 
| `/`     | 최상위 디렉터리         
| `/home` | 일반 사용자 홈 디렉터리    
| `/root` | root 사용자의 홈 디렉터리  
| `/etc`  | 시스템 설정 파일           
| `/var`  | 로그 및 가변 데이터      
| `/tmp`  | 임시 파일            
| `/usr`  | 프로그램 및 라이브러리     
| `/bin`  | 기본 명령어           
| `/sbin` | 시스템 관리 명령어       
| `/dev`  | 장치 파일            
| `/proc` | 프로세스 및 커널 정보     


### 디스크 사용량 확인
```bash
df -h
```
파일시스템의 전체 용량과 사용량을 확인
```bash
du -sh 디렉터리
```

### 디스크 및 파티션 확인
```bash
lsblk
fdisk -l
```
디스크와 파티션 정보를 확인

### 현재 디렉터리의 파일시스템 확인
```bash
df -h .
```

## 6. 로그 이해와 관리
Linux에서는 시스템 및 프로그램에서 발생하는 여러 이벤트를 로그로 기록
대표적인 로그 관리 데몬:
```bash
rsyslog
```
주요 로그 파일은 일반적으로 /var/log 아래에 존재
```bash
ls /var/log
```
예:
```bash
/var/log/messages
/var/log/secure
/var/log/cron
```

### rsyslog 로그 우선순위
로그의 심각도는 일반적으로 다음과 같이 이해할 수 있다.
```bash
emerg
alert
crit
err
warning
notice
info
debug
```

### 주요 Severity
| Level     | 설명                      
| --------- | ----------------------- 
| `emerg`   | 시스템을 사용할 수 없는 매우 심각한 상황 
| `alert`   | 즉각적인 조치가 필요한 상황         
| `crit`    | 심각한 문제                  
| `err`     | 오류 발생                   
| `warning` | 주의가 필요한 상황              
| `notice`  | 특별히 주의할 필요가 있는 일반적인 상황  
| `info`    | 일반적인 정보                 
| `debug`   | 디버깅을 위한 상세 정보           

### rsyslog 설정 예시
```bash
facility.priority    action
```
예:
```bash
*.info;mail.none;authpriv.none;cron.none    /var/log/messages
```
none은 특정 facility를 제외할 때 사용할 수 있다.
예를 들어:
```bash
*.info;mail.none
```

### 실무에서 서버 운영 중 보이는 메시지
#### kernel panic
Linux Kernel에서 복구하기 어려운 심각한 오류가 발생한 상태
```bash
Kernel panic
```
원인이 매우 다양하기 때문에 로그와 시스템 상태를 확인하여 원인을 파악해야 한다.
#### segmentation falut
프로그램이 허용되지 않은 메모리 영역에 접근할 때 발생할 수 있다.
```bash
Segmentation fault
```
주로 프로그램 자체의 버그나 잘못된 메모리 접근 등이 원인이 될 수 있다.
#### Hang
스템이나 프로그램이 응답하지 않고 멈춘 것처럼 보이는 상태를 의미한다.
```bash
서버가 hang 상태
서버가 뻗었다
```
=> emerg 메시지들, 원인이 너무 다양하기 때문에 재부팅으로 해결 (hang 상태: 서버가 멈춰있는 상태, 서버 뻗음)

## 7. 사용자 이해와 관리
Linux에서는 여러 사용자가 하나의 시스템을 사용할 수 있으며, 사용자와 그룹을 이용하여 권한을 관리






























