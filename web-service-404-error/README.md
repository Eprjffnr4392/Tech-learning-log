# Tomcat 404 및 DB 연결 장애 분석

> 업무 중 발생한 Tomcat 웹 서비스 장애를 분석하며 수행한 점검 과정과
> 원인 추적 방법을 정리한 기술 학습 기록입니다.

※ 보안 및 정보보호를 위해 실제 서버의 IP, 경로, 제품명, 프로세스명,
DB명, 포트번호 및 로그의 일부 내용을 비식별화했습니다.

---

## 1. 장애 현상

Tomcat 프로세스 자체는 정상적으로 기동된 것으로 확인되었으나,
웹 브라우저 및 `curl`을 이용하여 서비스에 접근했을 때 `HTTP 404`가 발생했습니다.

확인 명령:

```bash
curl -k -I https://<SERVER_IP>:<HTTPS_PORT>/CM001.do

# Tomcat 404 및 DB 연결 장애 분석

> 업무 중 발생한 Tomcat 웹 서비스 장애를 분석하며 수행한 점검 과정과  
> 원인 추적 방법을 정리한 기술 학습 기록입니다.

> [!NOTE]
> 보안 및 정보보호를 위해 실제 서버의 IP, 경로, 제품명, 프로세스명,  
> DB명, 포트번호 및 로그의 일부 내용을 비식별화했습니다.

---

## 1. 장애 현상

Tomcat 프로세스 자체는 정상적으로 기동된 것으로 확인되었으나,  
웹 브라우저 및 `curl`을 이용하여 서비스에 접근했을 때 `HTTP 404`가 발생했습니다.

```

### 확인 결과

```text
HTTP/1.1 404
Content-Type: text/html;charset=utf-8
```

단순히 HTTPS 포트가 열려 있지 않은 상황이라면 연결 자체가 실패해야 하지만,  
HTTP 응답 코드 `404`가 반환되었으므로 Tomcat의 HTTPS Connector까지는  
정상적으로 요청이 전달되고 있는 것으로 판단했습니다.

따라서 단순한 네트워크 연결 문제가 아니라 Tomcat 내부의  
Web Application 초기화 상태를 추가로 확인했습니다.

---

## 2. ROOT Web Application 확인

Tomcat의 ROOT 애플리케이션에 설정된 `index.jsp`를 확인했습니다.

### 확인 명령

```bash
cat /usr/local/tomcat/webapps/ROOT/index.jsp
```

### 확인 내용

```jsp
<%@ page language="java" contentType="text/html;charset=utf-8" %>
<jsp:forward page="CM001.do" />
```

ROOT 페이지에 접근하면 단순한 HTML 페이지를 출력하는 것이 아니라  
`CM001.do` 요청으로 Forward하도록 구성되어 있었습니다.

따라서 `CM001.do`를 처리하는 Web Application이 정상적으로  
초기화되었는지 추가적인 확인이 필요했습니다.

---

## 3. Tomcat 기동 상태 및 로그 확인

Tomcat의 기동 로그를 실시간으로 확인했습니다.

### 확인 명령

```bash
tail -f /usr/local/tomcat/logs/catalina.out
```

### 확인 결과

다음과 같은 메시지가 확인되었습니다.

```text
심각 [main] org.apache.catalina.core.StandardContext.startInternal
하나 이상의 리스너들이 시작하지 못했습니다.

심각 [main] org.apache.catalina.core.StandardContext.startInternal
이전 오류들로 인해 컨텍스트 []의 시작이 실패했습니다.
```

또한 HTTPS Connector 자체는 다음과 같이 시작된 것을 확인했습니다.

```text
프로토콜 핸들러 ["https-jsse-nio-<HTTPS_PORT>"]을(를) 시작합니다.
서버가 [약 4초] 내에 시작되었습니다.
```

이를 통해 다음과 같이 구분하여 판단했습니다.

```text
Tomcat 프로세스 기동
        ↓
HTTPS Connector 기동
        ↓
ROOT Web Application 배포
        ↓
Spring Context 초기화
        ↓
Listener 및 Bean 초기화
```

Tomcat 프로세스 및 HTTPS Connector는 기동되었지만  
Web Application의 초기화 과정에서 문제가 발생했을 가능성이 있다고 판단했습니다.

---

## 4. Tomcat 로그에서 Exception 확인

Tomcat 로그에서 주요 오류를 검색했습니다.

### 확인 명령

```bash
grep -RniE "SEVERE|심각|Exception|Caused by|ERROR|error|Failed|실패" \
/usr/local/tomcat/logs/ | tail -100
```

### 확인 결과

다음과 같은 DB 연결 오류가 반복적으로 확인되었습니다.

```text
org.springframework.jdbc.CannotGetJdbcConnectionException
```

```text
org.postgresql.util.PSQLException:
Connection to 127.0.0.1:<DB_PORT> refused.
```

또한 Spring Context 초기화 과정에서도 다음과 같은 오류가 확인되었습니다.

```text
Context initialized 이벤트를
org.springframework.web.context.ContextLoaderListener
클래스의 인스턴스에 전송하는 동안 예외 발생
```

그리고 Bean 초기화 실패도 확인되었습니다.

```text
BeanCreationException:
Error creating bean with name 'monMgr'
Invocation of init method failed
```

따라서 단순히 특정 URL의 Controller가 존재하지 않는 문제가 아니라  
Spring Application Context 자체가 정상적으로 초기화되지 못한 정황을 확인했습니다.

---

## 5. DB 연결 설정 확인

Tomcat Web Application에서 사용하는 DB 설정을 확인했습니다.

### 확인 명령

```bash
vim /usr/local/tomcat/webapps/ROOT/WEB-INF/classes/conf/jdbc.properties
```

### 확인 내용

```properties
db.Url = jdbc:postgresql://<DB_SERVER>:<DB_PORT>/<DB_NAME>
```

설정을 통해 해당 Web Application이 PostgreSQL DB를 사용하고 있으며,  
애플리케이션의 DB 접속 대상 정보를 확인할 수 있었습니다.

---

## 6. DB 설정이 실제 애플리케이션에 적용되는 과정 확인

Spring 설정에서 `jdbc.properties`가 실제로 참조되는지 확인했습니다.

### 확인 명령

```bash
grep -RniE "dataSource|DataSource|jdbc.properties|PropertyPlaceholder|context:property" \
/usr/local/tomcat/webapps/ROOT/WEB-INF/classes/conf/ 2>/dev/null
```

### 확인 결과

Spring 설정에서 다음과 같이 `jdbc.properties`를 참조하고 있었습니다.

```text
classpath:/conf/jdbc.properties
```

또한 Spring Application Context에서 `dataSource` Bean을 생성하고,  
MyBatis 역시 해당 DataSource를 사용하도록 구성되어 있었습니다.

전체적인 구조는 다음과 같이 정리할 수 있습니다.

```text
jdbc.properties
      ↓
Spring PropertyPlaceholder
      ↓
DataSource
      ↓
MyBatis
      ↓
DAO
      ↓
Service
      ↓
Spring Application Context
```

따라서 DB 연결 실패가 Spring Application Context 초기화 실패로  
이어질 가능성을 확인했습니다.

---

## 7. DB 연결 오류 재확인

Tomcat 디렉터리 전체에서 문제가 발생한 DB 포트를 검색했습니다.

### 확인 명령

```bash
grep -Rni "15432" /usr/local/tomcat/
```

### 확인 결과

Tomcat 로그에서 동일한 DB 연결 오류가 반복적으로 발생하고 있었습니다.

```text
Connection to 127.0.0.1:<DB_PORT> refused.
```

`Connection refused`는 해당 주소와 포트로 연결을 시도했으나  
연결을 받아줄 서비스가 실행 중이지 않거나 해당 포트에서  
연결을 받을 수 없는 상태에서 발생할 수 있습니다.

따라서 실제 DB 서비스 상태를 추가로 확인할 필요가 있었습니다.

---

## 8. 솔루션 로그에서 DB 연결 상태 확인

Tomcat뿐만 아니라 솔루션 프로세스의 로그에서도  
동일한 DB 연결 실패가 확인되었습니다.

### 확인 로그

```text
DataBase 연결실패
connection to server at "127.0.0.1",
port <DB_PORT> failed: Connection refused
```

이후 다음과 같은 초기화 실패 로그가 이어졌습니다.

```text
InitEncDB, hs_open_db error
InitDB fail
```

또한 다른 프로세스에서도 동일한 DB 연결 실패가 확인되었습니다.

```text
DataBase 연결실패
Connection refused
```

이를 통해 문제가 Tomcat에만 국한된 것이 아니라  
애플리케이션 및 솔루션 프로세스가 사용하는 DB 연결 계층에서도  
발생하고 있음을 확인했습니다.

---

## 9. 솔루션 기동 과정에서 확인된 DB 오류

솔루션 기동 로그에서도 PostgreSQL 연결 실패가 직접 확인되었습니다.

```text
sysbackup
SetDatabaseType, Databse Type = Master and Slave Mode

DataBase 연결실패(0).
msg=[connection to server at "127.0.0.1",
port <DB_PORT> failed: Connection refused
```

이후 다음과 같은 초기화 실패 로그가 확인되었습니다.

```text
InitEncDB, hs_open_db error
InitDB fail
```

다른 프로세스에서도 동일한 오류가 확인되었습니다.

```text
hrxicmon
DataBase 연결실패
connection to server at "127.0.0.1",
port <DB_PORT> failed: Connection refused
```

이를 통해 솔루션 프로세스가 기동되는 과정에서도  
DB 연결을 시도하고 있으며, DB가 연결되지 않는 경우  
관련 초기화 과정에 영향을 받을 수 있음을 확인했습니다.

---

## 10. Tomcat Listener 초기화 오류 확인

Tomcat 로그에서 Web Application 초기화 과정에서  
Listener 관련 오류도 확인했습니다.

### 확인 명령

```bash
grep -RniE "listenerStart|listenerStop|Context initialized|NoClassDefFoundError|ClassNotFoundException" \
/usr/local/tomcat/logs/
```

### 확인 결과

```text
Context initialized 이벤트를
org.springframework.web.context.ContextLoaderListener
클래스의 인스턴스에 전송하는 동안 예외 발생
```

또한 다음과 같은 오류가 확인되었습니다.

```text
java.lang.NoClassDefFoundError:
org/apache/log4j/LogManager
```

```text
java.lang.ClassNotFoundException:
org.apache.log4j.LogManager
```

해당 오류는 Web Application 초기화 또는 종료 과정에서  
필요한 클래스가 정상적으로 로딩되지 못했음을 나타냅니다.

다만 전체 장애 흐름에서는 PostgreSQL `Connection refused`가  
확인되었으므로 DB 연결 실패와 Web Application 초기화 실패의  
시간적 관계를 함께 확인했습니다.

---

## 11. 장애 원인 추적

지금까지 확인한 로그와 설정을 시간 순서대로 연결하면  
다음과 같은 흐름으로 장애가 발생한 것으로 판단했습니다.

```text
DB 서비스 중지 또는 DB 접속 불가 상태
                ↓
PostgreSQL Connection Refused
                ↓
Web Application의 DataSource DB 연결 실패
                ↓
Spring Bean 초기화 실패
                ↓
Spring Context 초기화 실패
                ↓
ContextLoaderListener 초기화 실패
                ↓
Web Application 정상 초기화 실패
                ↓
Tomcat HTTPS Connector 자체는 기동
                ↓
웹 요청 수신
                ↓
HTTP 404 응답
```

특히 Tomcat 자체에서는 HTTPS Connector가 정상적으로  
시작되었다는 로그가 확인되었습니다.

```text
프로토콜 핸들러 ["https-jsse-nio-<HTTPS_PORT>"]을(를) 시작합니다.
서버가 시작되었습니다.
```

따라서 `404`를 단순한 네트워크 또는 HTTPS 포트 문제로 판단하기보다  
Web Application 초기화 실패와 연관하여 분석했습니다.

---

## 12. 조치

DB 서비스 상태를 정상화한 후 Tomcat을 재기동하여  
Spring Application Context가 다시 초기화되도록 조치했습니다.

### Tomcat 상태 확인

```bash
systemctl status tomcat
```

### Tomcat 재기동

```bash
systemctl restart tomcat
```

재기동 후 Tomcat 로그를 다시 확인하고  
Web Application 초기화 여부를 확인했습니다.

```bash
tail -f /usr/local/tomcat/logs/catalina.out
```

이후 웹 서비스 접근 여부를 다시 확인했습니다.

```bash
curl -k -I https://<SERVER_IP>:<HTTPS_PORT>/CM001.do
```

Tomcat 재기동 이후 Web Application이 다시 초기화되면서  
웹 서비스가 정상화되는 것을 확인했습니다.

---

## 13. 최종 결론

이번 장애는 Tomcat의 HTTPS Connector 자체가 동작하지 않은  
문제라기보다 **DB 서비스가 중지된 상태에서 Web Application이  
PostgreSQL DB에 연결하지 못하면서 Spring Context 초기화에  
실패한 것이 주요 원인으로 판단됩니다.**

장애 분석 과정에서 확인된 핵심 로그는 다음과 같습니다.

```text
org.postgresql.util.PSQLException:
Connection to 127.0.0.1:<DB_PORT> refused
```

그리고 이후:

```text
BeanCreationException
```

```text
Context initialized 이벤트 예외
```

```text
StandardContext.startInternal
하나 이상의 리스너들이 시작하지 못했습니다.
```

위 로그를 종합하면 다음과 같은 연관성을 확인할 수 있습니다.

```text
DB 서비스 중지
      ↓
PostgreSQL Connection Refused
      ↓
Spring DataSource DB 연결 실패
      ↓
Bean 초기화 실패
      ↓
Spring Context 초기화 실패
      ↓
Web Application 정상 초기화 실패
      ↓
HTTP 404
```

DB 서비스를 정상화한 이후 Tomcat을 재기동하여  
Web Application을 다시 초기화함으로써 웹 서비스가 정상적으로  
복구되는 것을 확인했습니다.

> 단, 로그 분석만으로 `DB 서비스 중지`가 404의 유일한 원인이라고  
> 단정하기보다는 `Connection refused → Spring 초기화 실패 →  
> Web Application 정상 기동 실패 → 404`라는 장애 흐름이  
> 가장 유력한 원인으로 판단했습니다.

---

## 14. 이번 분석에서 배운 점

### ① HTTP 404만 보고 URL 문제라고 판단하지 않는다.

HTTP `404`가 발생하더라도 Tomcat Connector가 정상적으로 동작하고 있다면  
Web Application의 배포 및 초기화 상태를 함께 확인해야 합니다.

### ② Tomcat 기동 성공과 Web Application 기동 성공은 다르다.

```text
Tomcat Process
      ≠
Web Application
```

Tomcat 프로세스가 정상적으로 실행 중이라고 해서  
내부의 Spring Application이 정상적으로 초기화되었다는 의미는 아닙니다.

### ③ `Connection refused`는 중요한 원인 추적 단서다.

DB 연결 오류가 반복된다면 애플리케이션 설정만 확인하는 것이 아니라  
실제 DB 서비스가 실행 중인지와 해당 포트에서 연결을 받고 있는지를  
함께 확인해야 합니다.

### ④ 로그는 시간 순서대로 연결해서 분석한다.

단일 오류만 확인하는 것이 아니라 여러 로그를 시간순으로 연결해야  
장애 발생 과정을 파악할 수 있습니다.

```text
DB Connection Refused
        ↓
BeanCreationException
        ↓
Context 초기화 실패
        ↓
Listener 초기화 실패
        ↓
Web Application 초기화 실패
        ↓
HTTP 404
```

---

## 15. 사용 명령어 정리

| 명령어 | 사용 목적 |
|---|---|
| `curl -k -I` | HTTPS 서비스의 HTTP 응답 및 상태 코드 확인 |
| `cat` | JSP 및 설정 파일 내용 확인 |
| `tail -f` | Tomcat 실시간 로그 확인 |
| `grep -RniE` | 로그에서 Exception 및 오류 메시지 검색 |
| `grep -Rni "15432"` | 특정 DB 포트가 사용되는 위치 검색 |
| `grep -RniE "dataSource..."` | Spring DataSource 및 DB 설정 참조 관계 확인 |
| `vim jdbc.properties` | DB 접속 설정 확인 |
| `systemctl status tomcat` | Tomcat 서비스 상태 확인 |
| `systemctl restart tomcat` | Tomcat 재기동 |
| `curl -k -I` | 조치 이후 웹 서비스 정상 응답 여부 확인 |

---

## 16. Troubleshooting Flow

```text
[웹 서비스 404 발생]
          ↓
curl로 HTTP 응답 확인
          ↓
HTTPS Connector 정상 여부 확인
          ↓
Tomcat 로그 확인
          ↓
Spring Context / Listener 오류 확인
          ↓
PostgreSQL Connection Refused 확인
          ↓
jdbc.properties 및 Spring DataSource 설정 확인
          ↓
솔루션 로그에서도 DB 연결 실패 확인
          ↓
DB 서비스 상태 정상화
          ↓
Tomcat 재기동
          ↓
Web Application 재초기화
          ↓
curl로 서비스 재확인
          ↓
[웹 서비스 정상화]
```

---

## 17. 핵심 요약

**문제 상황**

> Tomcat은 기동되어 있었으나 Web Application 접근 시 HTTP 404 발생

**주요 로그**

> PostgreSQL `Connection refused`

**원인 추적**

> DB 연결 실패 → Spring Context 초기화 실패 → Web Application 정상 초기화 실패

**조치**

> DB 서비스 정상화 → Tomcat 재기동 → Web Application 재초기화

**결과**

> 웹 서비스 정상화 확인

---

### Security Note

본 문서는 실제 업무 환경에서 수행한 장애 분석 과정을 학습 목적으로  
재구성한 기록입니다.

실제 환경의 IP 주소, DB 정보, 서버 경로, 포트 번호, 제품명,  
프로세스명 및 기타 식별 가능한 정보는 비식별화하여 작성했습니다.
