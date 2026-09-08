# 1. U-02 비밀번호 관리 정책 설정

## 1.1 점검 개요

| 구분 | 내용 |
|---|---|
| 점검 내용 | 비밀번호 관리 정책 설정 여부를 점검한다. |
| 점검 목적 | 사용자의 비밀번호 복잡성과 주기적인 변경을 통해 시스템 보안을 강화한다. |
| 보안 위협 | 비밀번호 관련 정책이 설정되지 않을 경우 무차별 대입 공격, 사전 대입 공격 등에 의해 비밀번호가 노출될 위험이 존재한다. |
| 참고 | 비밀번호 관리 정책에는 비밀번호 복잡성, 길이, 변경 주기 등의 정책이 포함된다. |
| 비밀번호 복잡성 | 비밀번호 설정 시 영문, 숫자, 특수문자 등을 포함하여 일정 수준 이상의 복잡성을 요구한다. |
| 양호 | 비밀번호 관리 정책이 설정된 경우 |
| 취약 | 비밀번호 관리 정책이 설정되지 않은 경우 |

---

# 2. U-03 계정 잠금 임계값 설정

## 2.1 점검 개요

| 구분 | 내용 |
|---|---|
| 점검 내용 | 사용자 계정의 로그인 실패 시 계정 잠금 임계값이 설정되어 있는지 점검한다. |
| 점검 목적 | 계정 탈취를 목적으로 하는 무차별 대입 공격 등이 발생할 때 해당 계정을 잠금으로써 반복적인 인증 시도를 제한하고 비밀번호 추측 공격을 어렵게 한다. |
| 보안 위협 | 계정 잠금 임계값이 설정되어 있지 않으면 공격자가 비밀번호가 일치할 때까지 지속적으로 인증을 시도할 수 있어 비밀번호가 노출될 위험이 존재한다. |
| 참고 | 사용자 로그인 실패 임계값은 시스템 로그인 과정에서 몇 번의 인증 실패가 발생했을 때 로그인을 차단할 것인지 결정하는 값이다. |
| 양호 | 계정 잠금 임계값이 10회 이하로 설정된 경우 |
| 취약 | 계정 잠금 임계값이 설정되어 있지 않거나 10회 이하로 설정되지 않은 경우 |

---

# 3. U-02 비밀번호 관리 정책 점검 실습

복잡도 및 길이 정책은 `/etc/security/pwquality.conf` 파일에서, 비밀번호 사용 기간 정책은 `/etc/login.defs` 파일에서 확인한다.

```bash
$ grep -vE "^\s*#|^\s*$" /etc/security/pwquality.conf
(출력 없음 — 활성화된 정책 없음)

$ grep -E "^PASS_(MAX|MIN|WARN)_DAYS" /etc/login.defs
PASS_MAX_DAYS 99999
PASS_MIN_DAYS 0
PASS_WARN_AGE 7
```

복잡도 정책이 활성화되어 있지 않고 `PASS_MAX_DAYS`가 `99999`로 설정되어 있어 비밀번호 사용 기간 제한이 사실상 매우 길게 설정된 상태이다.

따라서 실습 환경에서는 비밀번호 관리 정책이 충분하게 적용되지 않은 상태로 판단하여 **취약한 상태**로 판정한다.

<img width="889" height="206" alt="image" src="https://github.com/user-attachments/assets/91e92a94-5893-46e8-b353-abb2ac01b492" />

---

# 4. U-02 비밀번호 관리 정책 조치 실습 — 정책 적용

`pwquality.conf`에 비밀번호 복잡도 및 길이 정책을 설정하고, `login.defs`에 비밀번호 사용 기간 정책을 설정한다.

```bash
# 1단계 — pwquality.conf 편집
$ sudo vim /etc/security/pwquality.conf
# (파일 끝에 다음 5줄 추가 후 저장)
minlen = 8
dcredit = -1
ucredit = -1
lcredit = -1
ocredit = -1

# 2단계 — login.defs 편집
$ sudo vim /etc/login.defs
# (다음 두 항목의 값을 수정 후 저장)
PASS_MAX_DAYS   90     (기존 99999 → 90)
PASS_MIN_DAYS   1      (기존 0 → 1)
```

설정한 비밀번호 복잡도 정책은 다음과 같다.

| 설정 | 의미 |
|---|---|
| `minlen = 8` | 비밀번호 최소 길이를 8자로 설정한다. |
| `dcredit = -1` | 숫자를 최소 1개 포함하도록 요구한다. |
| `ucredit = -1` | 영문 대문자를 최소 1개 포함하도록 요구한다. |
| `lcredit = -1` | 영문 소문자를 최소 1개 포함하도록 요구한다. |
| `ocredit = -1` | 기타 문자, 일반적으로 특수문자를 최소 1개 포함하도록 요구한다. |

비밀번호 사용 기간 정책은 다음과 같이 설정한다.

- `PASS_MAX_DAYS 90` : 최대 비밀번호 사용 기간을 90일로 설정한다.
- `PASS_MIN_DAYS 1` : 비밀번호 변경 후 최소 1일이 지나야 다시 변경할 수 있도록 설정한다.

`sudo vim /etc/security/pwquality.conf`

<img width="851" height="505" alt="image" src="https://github.com/user-attachments/assets/3f63e634-06e2-454d-8e8a-92e241dad1dc" />

`sudo vim /etc/login.defs`

<img width="323" height="176" alt="image" src="https://github.com/user-attachments/assets/146e9093-15e0-4f1b-917a-7ed3b3a0f95c" />

---

# 5. U-02 비밀번호 관리 정책 조치 실습 — 정책 적용 검증

`user` 계정에서 `sudo` 없이 `passwd` 명령을 실행하여 약한 비밀번호가 거부되는지 확인한다.

```bash
$ passwd
Changing password for user.
Current password: lab
New password: weak
BAD PASSWORD: The password is shorter than 8 characters
New password: ^C
(Ctrl+C로 취소 — 실제 비밀번호 변경하지 않음)
```

`weak`와 같이 정책에서 요구하는 최소 길이를 만족하지 못하는 비밀번호가 거부되는 것을 확인할 수 있다.

따라서 설정한 비밀번호 복잡도 정책이 정상적으로 적용되고 있음을 확인할 수 있으며, 실습 환경에서는 **U-02 항목이 양호한 상태로 전환된 것**으로 판단한다.

<img width="523" height="136" alt="image" src="https://github.com/user-attachments/assets/3d63e634-06e2-454d-8e8a-92e241dad1e1" />

비밀번호 정책 검증은 실제 사용자 인증 흐름에서 수행해야 하며, root 권한을 이용한 비밀번호 변경은 일반 사용자 인증 과정과 동작이 다를 수 있으므로 주의해야 한다.

---

# 6. U-03 계정 잠금 임계값 점검 실습

PAM의 인증 흐름이 정의된 `/etc/pam.d/common-auth` 파일과 잠금 정책 값이 설정되는 `/etc/security/faillock.conf` 파일을 확인한다.

```bash
$ grep -E "pam_faillock|pam_tally" /etc/pam.d/common-auth
(출력 없음 — 잠금 모듈 미적용)

$ grep -vE "^\s*#|^\s*$" /etc/security/faillock.conf
(출력 없음 — 활성화된 정책 없음)
```

`pam_faillock` 모듈이 PAM 인증 스택에 적용되어 있지 않고 `faillock.conf`에도 활성화된 정책이 설정되어 있지 않으므로, 실습 환경에서는 **계정 잠금 정책이 적용되지 않은 취약한 상태**로 판정한다.

<img width="885" height="132" alt="image" src="https://github.com/user-attachments/assets/31787833-b731-4ba6-bb17-be2de35abee3" />

---

# 7. U-03 계정 잠금 임계값 조치 실습 — 정책 파일 수정

`faillock.conf`에 계정 잠금 정책을 설정하고 `common-auth`에 `pam_faillock` 모듈을 추가한다.

```bash
# 1단계 — faillock.conf 편집
$ sudo vim /etc/security/faillock.conf
# (다음 두 줄의 주석(#)을 제거하고 값을 설정 후 저장)
deny = 5
unlock_time = 600

# 2단계 — common-auth 편집
$ sudo vim /etc/pam.d/common-auth
# (다음 세 줄을 적절한 위치에 추가 후 저장)
auth required pam_faillock.so preauth
auth [success=3 default=ignore] pam_unix.so nullok
auth [success=2 default=ignore] pam_sss.so use_first_pass

auth [default=die] pam_faillock.so authfail
auth requisite pam_deny.so
auth sufficient pam_faillock.so authsucc

auth required pam_permit.so
auth optional pam_cap.so
```

설정한 정책은 다음과 같다.

| 설정 | 의미 |
|---|---|
| `deny = 5` | 인증 실패가 5회 발생하면 계정을 잠금 대상으로 처리한다. |
| `unlock_time = 600` | 잠금 후 600초, 즉 10분 동안 계정을 잠금 상태로 유지한다. |

`pam_faillock`은 PAM 인증 과정에서 로그인 실패 횟수를 관리하고 일정 횟수 이상 실패한 계정의 인증을 제한하는 역할을 한다.

`sudo vim /etc/security/faillock.conf`

<img width="857" height="501" alt="image" src="https://github.com/user-attachments/assets/3e7334ac-6985-4c64-860f-d40c62715253" />

`sudo vim /etc/pam.d/common-auth`

<img width="907" height="507" alt="image" src="https://github.com/user-attachments/assets/4be836eb-a57b-43ed-af20-1d2029ba2498" />

> **주의:** PAM 설정은 인증 전체에 영향을 미치므로 설정을 잘못 수정하면 정상적인 로그인이나 `sudo` 사용까지 차단될 수 있다. 실제 운영 환경에서는 설정 변경 전에 기존 설정을 백업하고 테스트 계정 등을 이용하여 검증하는 것이 안전하다.

---

# 8. U-03 계정 잠금 임계값 조치 실습 — 잠금 동작 검증

`user` 계정에 잘못된 비밀번호를 5회 입력하여 계정 잠금이 발생하는지 확인한다.

```bash
# user로 5회 잘못된 비밀번호 시도
$ su userPassword: (틀린 비밀번호)
su: Authentication failure.
.. (5회 반복)

# 잠금 상태 조회
$ sudo faillock --user user
user:
When                Type  Source        Valid
2026-06-08 10:30:00 RHOST                 V

# 잠금 해제
$ sudo faillock --user user --reset
```

인증 실패 기록이 생성된 것을 확인한 후 `faillock` 명령어로 해당 계정의 잠금 상태를 조회한다.

잠금 테스트가 끝난 후에는 다음 명령어를 이용하여 실패 기록을 초기화한다.

```bash
$ sudo faillock --user user --reset
```

설정한 `deny = 5` 정책에 따라 반복적인 인증 실패가 정상적으로 기록되고 계정 잠금 정책이 동작하는 것을 확인했으므로, 실습 환경에서는 **U-03 항목의 판단 기준에 따라 양호한 상태로 전환된 것**으로 판단한다.

`user`로 5회 잘못된 비밀번호 시도

<img width="746" height="500" alt="image" src="https://github.com/user-attachments/assets/4c0941c0-20cc-4d4c-86fe-75f07e0042bf" />

잠금 상태 조회 및 잠금 해제

<img width="885" height="369" alt="image" src="https://github.com/user-attachments/assets/d76e3f85-a9de-415e-b979-35f07e0042bf" />

---

# 9. U-02와 U-03 핵심내용

| 점검 항목 | 점검 내용 | 주요 설정 | 실습 결과 |
|---|---|---|---|
| U-02 | 비밀번호 관리 정책 설정 여부 | `pwquality.conf`, `login.defs` | 복잡도 및 사용 기간 정책 적용 후 양호 |
| U-03 | 계정 잠금 임계값 설정 여부 | `faillock.conf`, PAM | 5회 실패 후 잠금 정책 적용하여 양호 |

### 핵심 내용

- **U-02**는 비밀번호의 길이, 복잡도, 사용 기간 등의 관리 정책이 적절하게 설정되어 있는지 점검한다.
- `/etc/security/pwquality.conf`에서 비밀번호 품질 및 복잡도 관련 정책을 설정할 수 있다.
- `/etc/login.defs`에서는 비밀번호의 기본적인 사용 기간 관련 정책을 설정할 수 있다.
- **U-03**은 로그인 실패가 일정 횟수 이상 발생했을 때 계정을 잠그는 정책이 설정되어 있는지 점검한다.
- `pam_faillock`은 PAM 인증 과정에서 로그인 실패 횟수를 관리하고 계정 잠금 정책을 적용하는 데 사용할 수 있다.
- `faillock.conf`에서 `deny`, `unlock_time` 등의 정책을 설정할 수 있다.
- PAM 설정은 시스템 인증에 직접적인 영향을 주므로 변경 전에 설정을 백업하고 충분히 검증해야 한다.
- 비밀번호 정책과 계정 잠금 정책을 함께 적용하면 무차별 대입 공격과 비밀번호 추측 공격에 대한 방어 수준을 높일 수 있다.
