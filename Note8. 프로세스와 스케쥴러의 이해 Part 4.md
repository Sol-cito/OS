## Note8. 프로세스와 스케쥴러의 이해 Part 4

#### 프로세스와 컨텍스트 스위칭

컨텍스트 스위칭이란, 스케쥴러가 **프로세스 A에서 프로세스 B로 running되는 프로세스를 교체해주는 메커니즘이다.**

<br>

프로세스가 메모리에 올라가면, 프로세스의 메모리 영역은 크게 4개로 나뉜다.

![image](https://www.includehelp.com/operating-systems/images/memory-layout-of-a-process-1.jpg)

(출처 : https://www.includehelp.com/operating-systems/)

- **Stack** : return address / 지역변수 / 매개변수 / 객체 참조변수 등이 담김. -- Subroutine

- **Heap** : 객체 메모리가 담김(참조변수) -- 사용자의 동적 할당. ex) C언어의 malloc

- **Data** : 전역변수 / static 변수가 담김. **BSS / DATA**  두 가지 영역으로 분리됨.
  - BSS : 초기화되지 않은 전역변수
  - DATA : 초기화된 전역변수

```c
int global_data1; // 초기화되지 않음 - BSS
int global_data2 = 1; // 초기화됨 - DATA
```

- **Text** : 컴파일된 2진 코드가 담김.

>  Tip : Stack영역은 Thread마다 하나씩 할당되며, 다른 쓰레드의 stack영역에 접근할 수 없다.
>
> 반면 heap 영역은 모든 Thread에서 접근 가능.
>
> heap 영역에 동적 메모리가 늘어날 수록 stack영역이 차지할 수 있는 공간은 줄어듬. Vice versa.

<br>

왜 갑자기 뜬금없이 프로세스의 메모리구조를 살펴보았는가?

**컨텍스트 스위칭(프로세스 교체)은 Program Counter와 Stack Pointer를 변경해주는 작업이기 때문이다.**

Program Counter(PC)는 CPU가 읽어들일 '**다음에 실행될 명령의 주소'**를 가리키는 역할을 하며,

Stack Pointer(SP)는 **'stack 메모리의 다음번 빈 공간의 주소'**를 가리키는 역할을 한다.

PC, SP 는 둘 다 CPU 레지스터의 일종이다.

자, 그럼 프로세스의 Running 과정을 보자.

<br>

프로세스가 Running할 때 CPU는

- 메모리에 올라간 프로세스의 명령어(Text)를 PC가 가리키는 주소에 따라 접근하여
- Stack영역에서 SP를 따라 명령을 수행한다.
- 이 때, **컨텍스트 스위칭**이 발생하면 (프로세스를 ready상태로 바꾸고 다른 프로세스를 running상태로 교체하려면)
- **PCB**에 기존 프로세스의 상태정보 (PC의 주소값, SP의 주소값 등)를 저장하고 메모리 공간에 저장한 후,
- 교체하려는 프로세스의 새로운 정보(New Context)를 **해당 프로세스의 PCB에서 가져와** 그 PCB에 담긴 내용(새로운 프로세스의 PC주소값, SP의 주소값 등)으로 프로세스를 Running한다.

> PCB란 Process Control Block으로, 운영체제가 프로세스에 대한 정보를 저장하기 위해 구축한 자료구조이다.
>
> 저장하는 프로세스의 정보 : Process ID, Register 값, Scheduling information (Process State), Memory info...

핵심은, **프로세스의 상태정보를 PCB 자료구조에 저장하고, 새 프로세스의 PCB를 참조하여 프로세스의 상태를 변경하는 것이다.**

이것이 바로 운영체제의 Context Switching이다.

<br>

> Context Switching은 프로세스를 1초에 몇백번씩 바꾸어야 하는 CPU에게는 굉장한 Overhead cost를 유발한다.
>
> 따라서 최대한 그 속도를 높여야 하기에, 이 부분은 C언어가 아닌 Assembly로 작성되어있다.
>
> 어셈블리어로 작성되면 컴파일할 필요가 없어서 성능이 높아지기 때문이다.

