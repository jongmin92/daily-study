# 전자 서명

## 목적
전자 서명의 목적은 "인증 기관"이라는 제 3자가가 "데이터"에 대해 문제가 없음을 "보증"하기 위한 것이다.
- 인증 기관 -> CA (Certificate Authority)
- 데이터 -> 인증서 (공개키를 포함하고 있다.)

## 정의
- 사전적 정의: 전자 문서의 위조나 변조를 방지하기 위하여 작성자를 확인할 수 있도록 해당 문서에 삽입하는 암호화된 정보 형태의 서명.
- 기술적 측면에서의 정의: 전자 문서의 해시(HASH)값을 서명자의 개인키(전자 서명 생성 정보)로 변환(암호화)한 것.

인증 기관이 제공하는 인증서를 인증 기관이 보증해주기 위해서 서명을 하게 되는데, 이때 서명이란 "공개키가 해시된 값"을 "인증기관의 비밀키"로 "암호화" 하는 것을 말한다. (= 디지털 서명)
> 공개키는 데이터 중 하나의 예시일 뿐이다.

## 과정
Remind --> 인증기관이 제공하는 인증서를 인증기관이 보증해주기 위해서 서명을 하게 되는데, 이때 **서명이란 "공개키가 해시된 값"을 "인증기관의 비밀키"로 "암호화"하는 것**을 말한다. (= 디지털 서명)

![digital-signature](/security/image/digital-signature/digital-signature.png)

- 서명 (Signing)
    1. 인증기관이 제공받은 Data를 해싱한 후 인증기관의 비밀키로 암호화 한다. → 서명(Signature)
    2. "Data" + "Certificate"(인증기관의 공개키 + Data의 발급자에 대한 정보 + 인증기관의 정보) + "Signature"(서명) 를 합쳐 "Digitally signed data"를 생성한다.
- 검증 (Verification)
    1. "Digitally signed data" 로부터 "Data", "Signature"를 얻는다.
    2. "Signature"를 인증기관의 공개키를 이용해 복호화한다.
    3. Data를 해싱한 후 결과가 2의 결과 값과 같은지 비교한다.

위의 내용을 바탕으로 SSL에 사용되는 인증서로 예를 들면 다음과 같은 과정을 거친다.

![ssl-flow](/security/image/digital-signature/ssl-flow.png)
