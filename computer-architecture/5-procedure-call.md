# 절차적 함수 호출(Procedure Call) 지원 CPU 모델
함수가 호출되는 원리와 호출이 될 때마다 할당되는 메모리 관리 방식에 대해서 알아본다.
> 함수가 호출되는 방식은 CPU에 따라서 차이가 있다. ARM 코어에서는 함수의 전달인자와 리턴 어드레스(함수 호출 완료 후 되돌아 갈 주소)를 레지스터에 저장하고, 그 저장 방식에 대한 표준을 정의하고 있다.

## 스택 프레임(Stack Frame)
- 함수 호출 과정에서 할당되는 메모리 블록 (= 지역변수의 선언으로 인해 할당되는 메모리 블록)
- 함수 호출이 return 하고 나면 할당된 메모리가 반환되어 기존에 선언된 지역변수에 접근이 불가능하다.
- 함수가 호출되면서 함수에 선언된 변수가 스택에 할당된다. 이 메모리 블록을 가리켜 스택 프레임이라고 한다. 함수가 반환되면 이 스택 프레임은 모두 반환된다.

![stack-frame](/computer-architecture/image/5-procedure-call/stack-frame.png)

### SP(Stack Pointer) 레지스터
- 어디까지 메모리를 사용(저장)했는지 저장하는 레지스터
- 변수가 할당될 때마다 변수의 크기만큼 증가하고, 다음 변수가 할당될 메모리 위치를 가리킨다.
- 호출된 함수가 종료될 경우, 스택 프레임 단위로 SP 레지스터 값을 이동한다.
> 지역변수를 위한 메모리 공간을 스택이라 이름 붙인 이유는 메모리의 구조적 특성(First in, Last Out) 때문이다  스택 프레임은 가장 먼저 할당되면, 가장 나중에 반환된다.

![sp-register](/computer-architecture/image/5-procedure-call/sp-register.png)

- 햠수 호출시
    1. FP -> Memory(Stack)
    2. SP -> FP

### FP(Frame Pointer) 레지스터
- 함수 호출 이전의 위치를 저장하는 레지스터 (FP 레지스터는 SP의 백업이다 라고 생각하자.)
- 함수의 호출이 계속되면 FP의 값이 덮어씌워질 것이다. 그래서 함수 호출이 일어날 때마다 FP 레지스터에 저장된 값을 스택에 저장한다. 그리고 새로운 값으로 FP 레지스터를 채운다.

![fp-register](/computer-architecture/image/5-procedure-call/fp-register.png)

- 햠수 리턴시
    1. FP -> SP
    2. Memory -> FP

## PUSH & POP
- 함수 호출 시 실행 위치의 이동은 어떻게 이뤄지는가?
- 함수 호출 시 전달되는 인자들은 어떻게 함수 내부로 전달되는가?
- 함수 호출이 끝나고 나면 어떻게 이전 실행 위치로 복귀하는가?  


### 함수 호출 인자의 전달 방식
- 전달되는 인자(매개변수)도 지역변수와 같이 stack에 쌓인다.
- 지역변수와 같이 stack에 쌓이기 때문에 life cycle도 같다.
- 성능 향상을 위해서 일부 전달인자들은 레지스터를 할당해서 저장하기도 한다.
- 실제 ARM 코어와 64비트 인텔 코어는 레지스터를 적극 활용한다.

![function-parameter](/computer-architecture/image/5-procedure-call/function-parameter.png)

![after-call-function](/computer-architecture/image/5-procedure-call/after-call-function.png)

### 호출규약과 실행의 이동
함수 호출은 PC 값을 변경하는 것과 같다. 순차적으로 실행하는 것 뿐만 아니라 실행해야하는 위치를 변경하는 것도 가능하다.

- 함수호출 규약
    - 어떻게 실행할 것인가?
    - 매개변수는 오른쪽부터? 왼쪽부터?
    - Stack cleanup은 함수 호출자가 할 것인가? 호출된 함수가 할 것인가?
    - 전달되는 매개변수가 32bit 환경에서는 stack에 쌓였었다. 그러나 __fastcall의 경우는 매개변수 2개까지는 레지스터에 저장한다. (레지스터는 저장과 삭제가 엄청나게 빠르다.)
    - 64bit 환경에서도 매개변수의 특정 개수까지 레지스터에 저장한다. -> 함수 호출이 빨라진다는 이야기이다. (레지스터를 사용하는가 사용하지 않는가가 속도에 엄청나게 큰 영향을 미친다.)
