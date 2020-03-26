# 메모리 계층 구조
- 레지스터 -> L1 캐쉬 -> L2 캐쉬 -> 메인 메모리 -> 하드 디스크
    - 각 계층을 비교할 때는 같은 속성을 기준으로 비교해야 한다.
    - 각 계층에서 볼 때는 상위에 있는 것이 캐쉬와 다름없다.

# 캐쉬와 캐쉬 알고리즘
- 지역성으로 인해 CPU가 사용하고자 하는 데이터가 캐쉬에 존재할 확률이 90%가 넘는다. (높다.)


## 프로그램의 일반적 특성
- 성능 향상과 캐쉬메모리
    - Locality
        - Temporal Locality: 반복접근 (시간 지역성)
        - Spatial Locality: 주변접근 (공간 지역성)
            - 90% 이상의 캐쉬 hit이 발생하는 이유