# 공인인증서

## 정의
공인인증서는 "전자 서명의 검증에 필요한 공개 키", "전자 서명을 한 주체", "소유자 정보"를 추가하여 만든 일종의 전자 신분증 및 전자 인감증명이다.

## 구조
인증서에는 다양한 정보들을 포함하여 만들어진다.

![certificate-structure-1](/encryption/image/certificate/certificate-structure-1.png)

- 발급자
    - 공인 인증 기관이 인증서를 발급하기 때문에 인증 기관을 다른 말로 발급자라고 표현한다.
    - 발급자는 인증서를 요청하는 기업이나 개인에 대해서 철저하게 신원을 확인하기 때문에 제삼자가 네이버라는 이름을 사칭하여 인증서를 발급받을 수는 없다.
- 발급자의 서명
    - 인증서에 기록될 여러 정보를 하나로 모은 후 해시 함수에 입력하여 해시 값을 얻어내고, 발급자의 암호화키(개인키)로 해당 해시값을 암호환 결과를 말한다.

![certificate-structure-2](/encryption/image/certificate/certificate-structure-2.png)

# PKI

## 정의
공개 키 기반구조(Public Key Infrastructure): 공개 키를 효과적으로 운용하기 위해 정해진 많은 규격이나 선택 사양의 총칭.
-> 비대칭 암호시스템에 기반해서 디지털 인증서의 생성, 관리, 저장, 배분, 삭제에 필요한 하드웨어, 소프트웨어, 사람, 정책 및 절차를 정의하고 있다.

## 목적
안전하고, 편리하고, 효율적으로 공개키를 획득 및 관리

## 구성 요소

![pki-structure](/encryption/image/certificate/pki-structure.png)

## 인증 기관의 역할

### 키 쌍의 작성
- 키 쌍을 작성하는 두 가지 방법
    - PKI의 이용자 스스로가 수행하는 경우
    - 인증기관이 수행하는 경우
        - 인증기관은 한 쌍의 공개키와 개인키를 생성
        - 안전한 방법으로 "개인키를 이용자에게 보내는" 일을 할 필요가 있다.
        - 그 방법은 PKCS #12(Personal Information Exchange Syntax Standard) 규격을 따른다.
        
### 인증서 등록
- 이용자 스스로가 키 쌍을 만들었을 경우
    - 이용자는 인증기관에 공개키를 등록하기 위해서 해당 공개키를 인증기관에 보내고 인증서 작성을 의뢰한다.
    - 인증서를 신청할 때의 규격인 PKCS #10을 따른다.
    
### 인증서 폐지와 CRL
- 이용자가 개인키를 분실 혹은 도난 당하는 경우, 인증 기관은 해당 인증서를 폐지(revoke)해서 무효화한다.
- CRL(Certificate Revocation List)
    - 인증 기관이 폐지한 인증서 목록
    - 폐지된 인증서의 일련 번호의 목록에 대해 인증 기관이 디지털 서명을 붙인 것
        - 인증서의 일련 번호: 인증기관이 발행한 인증서에 붙여진 순서 번호

# X.509 인증서

## 정의
인증서는 인증 기관에서 발행하고, 이용자가 그것을 검증하는 것이 되기 때문에 인증서의 형식이 서로 다르면 매우 불편하다.

이를 해결하기 위해서 인증서의 표준 규격을 정하고 사용한다.

X.509는 인증서의 표준 규격이다.

## 구성 요소
- Version : 인증서의 버전
- Serial Number : CA가 할당한 정수로 된 고유 번호
- **Signature** : 서명 알고리즘 식별자
- **Issuer** : 발행자 (CA의 이름)
- Validity : 유효기간
- **Subject** : 소유자 (주로 사이트 소유자의 도메인 혹은 하위 CA 이름)
- Subject Public Key Info : 소유자 공개키 정보
    - Public Key Algorithm : 공개키 알고리즘의 종류
    - Subject Public Key : 공개키
- **Extensions** : 확장 필드
- Certificate Signature Algorithm : 디지털 서명의 알고리즘 종류
- Certificate Signature : 디지털 서명 값

![x-509](/encryption/image/certificate/x-509.png)

## 인증서 파일 확장자
X.509 표준의 인증서 형식은 ASN.1 (Abstract Syntax Notation One) 이라는 명명 규칙을 따른다. X.509 인증서의 각 구성 요소는 ASN.1 형식에 맞게 저장된다.

ASN.1 형식으로 되어 있는 X.509 인증서의 내용을 파일로 저장할 때 주로 두 가지 종류의 형식으로 저장한다.

- .cer / .der : X.509 인증서 내용을 "바이너리 형태"로 저장한다.
![x-509.cer](/encryption/image/certificate/x-509-cer.png)
- .pem : X.509 인증서 내용을 "Base64로 인코딩"하여 저장한다.
![x-509.pem](/encryption/image/certificate/x-509-pem.png)
