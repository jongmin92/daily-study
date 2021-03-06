# 컴퓨터 구조에 대한 두 번째 이야기
컴퓨터 구조에 대한 근본적인 이해는 프로세스와 스레드를 이해하는 데도 큰 도움이 된다.

## 컴퓨터를 디자인 하자
가상의 컴퓨터를 디자인하는 과정에서 CPU가 하는 일과 구성 요소에 대해서 직접적으로 느껴보자. CPU 종류가 상당하고 표준이 없기 때문에 직접 디자인 해보는 것이 가치가 있다.
- 프로그래머 관점
    - 컴퓨터 구조를 잘 아는 프로그래머도 컴퓨터 디자인에 참여
    - 컴퓨터 디자인은 레지스터와 명령어 디자인
- 레지스터 디자인의 핵심
    - 레지스터는 **몇 비트**로 구성할 것인가?
        - n비트 시스템에서는 명령어의 길이도 n비트, 레지스터의 길이도 n비트
    - **몇 개** 정도로 레지스터를 구성할 것인가?
    - 레지스터 각각을 무슨 **용도**로 사용할 것인가?
        - 레지스터는 특별한 목적(용도)을 갖고 있는 저장 장치이다.
        - 용도를 갖고 있으면 명령어도 단순해지고 속도도 빨라진다.
        - r0 ~ r7
            - r0~r3: 연산을 위한 범용 레지스터
            - r4(ir): instruction register
                - 실행하게 될 명령어를 미리 가져다 놓는 용도로 사용
            - r5(sp): stack pointer
                - 가장 최근 입력된 데이터를 가리킴
            - r6(lr): link register
                - 서브루틴 함수에서 되돌아갈 주소를 저장, 예외 처리후 돌아갈 주소를 저장
            - r7(pc): program counter
                - 다음에 수행될 명령어의 주소값 저장
            
> 위의 레지스터 구조는 ARM 코어를 참조한 것이다. ARM 어셈블리 언어 형식은 다른 어느 CPU 보다 하드웨어 구성과 명령어 구조가 간단하다. 때문에 전력 소비가 아주 적어 핸드폰에 매우 적합한 코어이다.

## 명령어 구조 및 명령어 디자인
**CPU 구성형태(레지스터 구성형태)에 따라서 명령어 구조가 달라진다.** 때문에 어셈블리 언어로 구현된 프로그램은 구조가 다른 CPU로 이식이 불가능하다.
> 어셈블리 언어는 결국 CPU에 종속적 이라는 이야기.

- 명령어의 기본 모델
    - 16비트 명령어: 16비트 시스템에서는 명령어의 길이가 16비트 여야지만 CPU로 fetch될 때, 하나의 명령어가 하나의 레지스터에 저장이 가능하다.
- 사칙연산 명령어 구성
    - ![instruction-design](/computer-architecture/image/4-computer-architecture/instruction-design.png)
    - 저장소에는 연산결과를 저장할 레지스터 정보만 올 수 있도록 제한한다.
        > [Q] 연산결과를 레지스터가 아닌 메모리에 직접 저장하면 안되는가?  
        [A] 명령어 구조가 복잡해지고, 이에 따라 하드웨어 구성도 복잡해진다. 명령어를 처리하는 데 걸리는 시간도 증가할 수 있다.
    - 피연산자에는 레지스터 정보 혹은 숫자가 온다. (어떤것인지 표현하기 위한 별도의 비트가 필요)
    - 피연산자가 표현할 수 있는 숫자가 8개 밖에 되지 않는다.
- Control Unit 구조
    - Control Unit은 명령어를 해석하는 일을 맡고 있기 때문에, 구성하는 명령어의 형태에 따라 Control Unit 구조가 결정된다. (명령어의 형태에 따라서 Control Unit의 논리회로가 디자인 된다.)
    
![calculate-flow](/computer-architecture/image/4-computer-architecture/calculate-flow.png)

>CISC <-> RISC
>- CISC(Complex Instruction Set Computer)
>   - 명령어 종류가 많아서 프로그램을 구현하는데 있어 편리함을 가져다 준다.
>   - 수십 줄에 걸쳐서 구현해야 하는 기능을 단 몇줄로 완성시킬 수 있다.
>   - 단, 명렁어 수가 많고, 그 크기가 일정하지 않기 떄문에 CPU는 복잡해진다.
>- RISC(Reduced Instruction Set Computer)  
>   - CISC 구조 CPU가 지니는 전체 명령어 중 주로 사용하는 명령어가 10% 정도밖에 되지 않는 데서 착안된 구조이다.
>   - CISC 보다 적은 클럭 수로 같은 일을 할 수 있다.
>   - 메인 메모리 주소 정보를 피연산자로 올 수 있도록 명령어 구조를 설계하지 않았다.
>   - 오늘날 대부분의 컴퓨터는 RISC 구조이다.
>   - 초당 클럭 수를 높이는 것도 성능을 향상시키는 일이지만, 이보다 더 중요한 것은 클럭당 처리할 수 있는 명령어의 개수이다. **RISC는 명령어 길이가 동일하고 명령어를 처리하는 과정이 일정하기 때문에 클럭당 둘 이상의 명령어 처리가 가능(Pipelining
> 기법)하다.** 바로 이점이 성능 향상에 있어서 RISC 구조가 지니는 가장 큰 장점이 된다.
>   > **Pipelining 기법**  
기존에 명령어 처리시 메모리에서 첫번째 명령어를 가지고 와서 명령어에 해당하는 연산을 수행하고, 다음 명령어를 또 메모리에 가져와 연산을 수행한다. 이때 다음 명령어가 도착할 때까지 쉬어야 하는데, Pipelining 기법은 명령어 연산을 수행하는 동안 다음 명령어를 가져다 놓는다. 즉, ALU가 명령어 A를 연산하고 있을 때, 다음 명령어를 Fetch 하는 것이다.


# LOAD & STORE 명령어 디자인
- 명령어 제약사항
    - 연산결과를 레지스터에만 저장
    - 모든 피연산자는 메인 메모리의 주소값이 올 수 없다.
    - 피연산자로 올 수 있는 것은 숫자와 레지스터로 제한

이전에 디자인한 사칙연산 명령어에 비해서 피연산자가 두 개면 된다. (메인 메모리 정보, 레지스터 정보)  
즉, 명령어의 종류에 따라서 정보를 담고 있는 방식도 다르다.

- LOAD: 메모리에 있는 데이터를 레지스터로 불러옴.
- STORE: 레지스터에 있는 값을 메모리에 저장.

![load-store](/computer-architecture/image/4-computer-architecture/load-store.png)

```
LOAD  r1,  0x10         // 0x10번지에 저장된 데이터를 r1로 이동
LOAD  r2,  0x20         // 0x20번지에 저장된 데이터를 r2로 이동
ADD   r3,  r1,  r2      // r1, r2에 저장된 값을 더해서 r3에 결과 저장
STORE  r3,  0x30        // r3에 저장된 값을 0x30번지에 저장
```

![process-memory](/computer-architecture/image/4-computer-architecture/process-memory.png)


# Direct 모드와 Indirect 모드
- 하나의 명령어에 여러 정보를 담다 보니 표현하는 데이터 크기에 제한이 생기는 문제가 발생함.
    - ex) LOAD 명령어의 메인 메모리의 주소 값을 나타내는 source가 0x0000~0x00ff 까지 밖에 표현하지 못함.
- 기존의 방식으로 메모리의 모든 영역에 대한 접근이 불가능하다.
- 이를 해결하기 위해 Indirect Addressing Mode가 등장함.
> 포인터가 필요한 이유와 관련있다고 생각한다. 포인터는 결국 "메모리의 주소"를 저장하기 위한 메모리 공간일 뿐이다.


- **Direct Addressing Mode**: 명령어에서 지정하는 위치의 메모리를 참조하는 방식.
- **Indirect Addressing Mode**: 명령어에서 지정하는 위치에 저장된 값을 주소값으로 하여 메모리를 참조하는 방식.

![indirect-mode](/computer-architecture/image/4-computer-architecture/indirect-mode.png)
![indirect-mode-bit](/computer-architecture/image/4-computer-architecture/indirect-mode-bit.png)

## Indirect 모드 활용 예제
![indirect-mode-ex](/computer-architecture/image/4-computer-architecture/indirect-mode-ex.png)
```
LOAD  r1,  0x0010         // r1=10 (a)

MUL  r0,  4,  4                 // r0=16
MUL  r2,  4,  4                 // r2=16
MUL  r3,  r0,  r2              // r3=256(=0x100)

STORE  r3,  0x0030
LOAD  r2,  [0x0030]      // r2=20 (b)

ADD  r3,  r1,  r2             // r3=30 (a+b)
STORE  r2,  0x0020
```

