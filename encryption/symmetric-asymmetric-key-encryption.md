# 대칭키 암호화

![symmetric-key-encryption](/encryption/image/symmetric-asymmetric-key-encryption/symmetric-key-encryption.png)

- 송신자와 수산지가 동일한 비밀키 소유
- 일반적으로 송신자는 암호용으로, 수신자는 복호용으로 사용
    - 암호화와 복호화에 사용되는 키가 동일하다.
- 암호화를 사용해 통신하기 위해서는, 사전에 송수신자가 서로 비밀키를 공유해야 한다.
- 비대칭키 방식에 비해 속도가 매우 빠르다.
- 암복호화에 같은 키가 사용되기 때문에 서로 키를 서로 공유하는 과정에서 외부에 노출될 가능성이 있다.
- ex. DES(Data Encryption Standard), AES(Advanced Encryption Standard)

cc. https://www.devglan.com/corejava/java-aes-encypt-decrypt

# 비대칭키 암호화

![asymmetric-key-encryption](/encryption/image/symmetric-asymmetric-key-encryption/asymmetric-key-encryption.png)

- 사용자는 한 쌍의 키 (공개키/개인키) 생성
    - 공개키: 누구나 수신자의 공개키를 이용하여 암호문을 생성할 수 있다. → 공개키 암호화 방식이라고 부르는 이유
    - 개인키: 사용자의 비밀키로서 소유자만 암호문의 복호화를 할 수 있다.
    - 암호화와 복호화에 사용되는 키가 서로 다르다.
        - 암호화와 복호화에 사용되는 키가 다름에도 수학적 원리에 의해 해독이 가능하다.
        - 공개키로 암호화된 것을 비밀키로 복호화할 수 있을 뿐만 아니라, 비밀키로 암호화된 것을 공개키를 이용해서도 복호화할 수 있다. (하나의 키로만 암호화와 복호화가 불가능 한 것)
- 대칭키 방식에 비해 속도가 느리다.
    - 비대칭키 암호화 방식은 복잡한 수학적 원리로 이루어져 있어 많은 CPU 리소스를 소모한다.
- 복호화에 사용하는 개인키는 오직 자기 자신만 소유하며, 암호화를 위한 공개키를 외부(사용자)에 공개한다.
- 외부에 어떻게 안전(나의 공개키 임을 보증)하게 공개할 수 있을까?
    - 이때 공개키와 자신의 정보를 인증서로 만들어 "공인 인증 기관"을 통해서 제공할 수 있다.
        - 인감도장을 만들어서 동사무소에 인감을 등록하는 것과 비슷하다. 인감도장의 내용을 동사무소에 등록하면 동사무소는 이것이 내가 발행한 인감이라는 것을 보증해준다.
        - 마찬가지로 내가 발행한 공개키(인감도장, 인증서)을 인증 기관(동사무소, CA)이 "전자 서명"을 통해 보증해주는 것이다.
- "인증서" = "공개키" + "사이트의 정보" (+ 추가로 "공인 인증 기관의 전자서명"도 포함한다.)
- ex. RSA (Rivest–Shamir–Adleman)

cc. https://www.devglan.com/java8/rsa-encryption-decryption-java
