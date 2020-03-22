# 윈도우에서의 유니코드
- 영상
    - https://www.youtube.com/watch?v=nmfNd8mIRDs&list=PLVsNizTWUw7E2KrfnsyEjTqo-6uKiQoxc&index=3
    - https://www.youtube.com/watch?v=wfcpJl3miOs&list=PLVsNizTWUw7E2KrfnsyEjTqo-6uKiQoxc&index=4
- 참고
    - https://cjwoov.tistory.com/20?category=800243
    - 

## 문자셋의 종류와 특성
- **SBCS** (Single Byte Character Set)
    - 문자를 표현하는데 1바이트 사용
    - ex) 아스키코드
- **MBCS** (Multi Byte Character Set)
    - 한글은 2바이트, 영문은 1바이트 사용
- **WBCS** (Wide Byte Character Set)
    - 문자를 표현하는데 2바이트 사용
    - ex) 유니코드

## C언어에서의 WBCS 기반의 프로그래밍
- WBCS를 위한 두가지
    - char를 대신해서 `wchar_t` 사용
    - "ABC" (아스키코드 기반)를 대신해서 L"ABC" (유니코드 기반) 사용
- WBCS 기반 문자열 선언 예
    - wchar_t str[] = L"ABC";

## MBCS와 WBCS 동시 지원 매크로
유니코드의 장점, 용이성들이 있음에도 불구하고 왜 MBCS를 모든 경우 유니코드를 사용할 수 있는 것은 아니다.  
기존에 MBCS 기준으로 개발된 프로그램과의 호환성, 아직 유니코드를 지원하지 않는 사용자의 시스템과 같이 여러 이유가 있다.

`windows.h` 안에는 MBCS와 WBCS를 동시에 지원하는 매크로가 있다.
> 이것도 어찌보면 문자셋에 대한 추상화의 한 부분이 아닐까...

```c
#ifdef UNICODE
    typedef    WCHAR     TCHAR;
    typedef    LPWSTR    LPTSTR;
    typedef    LPCWSTR   LPCTSTR;
#else
    typedef    CHAR      TCHAR;
    typedef    LPSTR     LPTSTR;
    typedef    LPCSTR    LPCTSTR;
#endif

#ifdef _UNICODE
    #define __T(X)  L ## x
#else
    #define __T(X)  x
#endif

#define _T(X)       __T(X)
#define _TEXT(X)    __T(X)
```

- 매크로 UNICODE가 정의 O
    - TCHAR arr[10]; -> WCHAR arr[10] -> wchar_t arr[10];
- 매크로 UNICODE가 정의 X
    - TCHAR arr[10]; -> CHAR arr[10] -> char arr[10];

```c
#define UNICODE
#define _UNICODE

int wmain(void) {
    TCHAR str[] = _T("1234567");
    pinrtf("sizeof: %d\n", sizeof(str));
    return 0;
}

// 실행결과
sizeof: 16
```
