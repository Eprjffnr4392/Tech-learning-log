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

확인 결과
HTTP/1.1 404
Content-Type: text/html;charset=utf-8

단순히 HTTPS 포트가 열려 있지 않은 상황이라면 연결 자체가 실패해야 하지만, HTTP 응답 코드 404가 반환되었으므로 Tomcat의 HTTPS Connector까지는 정상적으로 요청이 전달되고 있는 것으로 판단했습니다.

따라서 네트워크 또는 HTTPS 포트 자체의 문제보다는 Tomcat Web Application 초기화 상태를 확인할 필요가 있다고 판단했습니다.
