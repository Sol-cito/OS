## Note6. 프로세스와 스케쥴러의 이해 Part 2

> Tip : 프로그램과 프로세스의 차이
>
> 프로그램은 실행 파일 자체를 의미하고, 프로세스는 프로그램 실행 시 메모리에 만들어지는 '인스턴스'라고 생각하면 된다.
>
> 하나의 프로그램이 여러가지 프로세스를 생성할 수도 있다.



#### 프로세스 상태와 멀티 프로그래밍

만약 Program A, B, C, D가 있을 때, 어떻게 CPU자원을 극대화할 수 있을까? (멀티프로그래밍의 관점에서)



프로세스는 크게 생성 - Running - 소멸 의 생명주기를 가지는데, 

**Running이 될 때 CPU가 해당 프로세스를 수행한다.**

그리고 **프로세스 상태**에 따라 CPU가 running할 프로세스를 고르게 된다.



자세한 프로세스 상태정보는 다음과 같다.

![image_1](http://itwiki.kr/images/d/da/%ED%94%84%EB%A1%9C%EC%84%B8%EC%8A%A4_%EC%83%81%ED%83%9C%EC%A0%84%EC%9D%B4%EB%8F%84.png)

(출처 : http://itwiki.kr/w/%ED%94%84%EB%A1%9C%EC%84%B8%EC%8A%A4_%EC%83%81%ED%83%9C )



**'디스패치'**란, 프로세스를 CPU에 올리는 작업을 뜻한다.

프로세스 상태 중 중요하게 살펴봐야 할 것은 **Running, Wait, Ready**의 세 가지 상태정보이다.

**Running**은 CPU에서 실행중인 상태,

**Waiting(혹은 blocked)**은 외부자원을 활용할 때의 대기상태 (Ex. 시스템콜로 프린팅을 한 후 결과 리턴을 받을 때 까지 대기) 

-> Wating이 끝나면 **인터럽트**를 통하여 다시 프로세스를 Ready queue에 넣는다.

**Ready**는 외부자원 활용이 끝난 후 다시 디스패치를 기다리는 상태를 의미한다.



가령, 내가 특정 프로그램을 실행시켜서 비밀번호를 입력한다고 하자.

비밀번호 입력 input이 들어오기 전까지 CPU는 그냥 대기하고있어야 되므로, **CPU자원낭비가 발생한다.**

따라서 이 때 CPU는 키보드 입력이 들어올 때까지 해당 프로세스를 **Waiting**상태로 바꾸어 놓고, 

**Ready Queue**에서 다른 프로세스를 꺼내서 실행하기 시작한다.

키보드 입력이 다 끝나고 엔터를 누르면 해당 프로세스는 **Ready**상태가 되고, **다시 Ready Queue에 insert된다.**



**종료(Exit)상태**에서는, 프로세스가 붙잡고 있는 **시스템 리소스**를 다시 풀어준다 

(Ex).자바에서 FileReader 사용 시 close() 함수를 반드시 호출해야 함)



그런데, **Ready Queue**에 있는 프로세스가 여러개가 있으면, 

CPU는 **'어떤 기준으로'** running할 프로세스를 고르게 될까?



다음과 같이 동작하는 Process 3개가 있다고 하자. CPU실행시간 1초당 / 기호로 나누어졌다고 가정하자.

- Process A : Running / Running
- Process B : Running / Wait / Wait / Wait / Running
- Process C : Running / Running / Wait / Wait / Wait / Running

우선 **프로세스 상태별로 Queue를 3개 만든다 : ** **Ready Queue, Running Queue, Wait Queue**

프로세스를 실행시켰을 때, 우선 Running상태인 프로세스들을 Running queue에 넣는다.

(처음엔 순서대로 A, B, C, 순으로 넣는다고 가정)

1초 후, Running Queue에서 pop한 프로세스의 다음 상태가 Running이면 다시 Running Queue에 넣고, 

Wait면 Wait Queue에 넣는다.

Wait Queue에 있는 프로세스가 Wait 시간을 만족하면, 다시 꺼내서 Ready Queue에 넣는다.

위 순서를 반복하면, 최종적으로 아래와 같은 스케쥴링이 완성된다 : 



**Process A / Process B / Process C / Process A / Process C / Process B / idle / idle / Process C**



> CPU가 아무 작업도 하지 않는 것을 'Idle 상태(not being used)'라 한다.

위 결과에서 Idle 상태가 만들어진 이유는, 

Process C가 wait 상태일 때 Process A 와 B는 이미 Running이 끝나고 Exit 되었으므로 

**CPU가 돌릴 Process가 더 이상 존재하지 않기 때문이다.**



<br/>

#### 선점형과 비선점형 스케쥴러 차이

- 선점형 : 하나의 프로세스가 다른 프로세스 대신에 CPU를 차지할 수 있음.
  - 시분할 시스템은 선점형이 지원되어야만 한다.
  - 대부분의 운영체제는 선점형 스케쥴러를 갖추고 있다.
- 비선점형 : 하나의 프로세스가 끝나지 않으면 다른 프로세스는 CPU를 사용할 수 없음.
  - 배치처리 프로그램

<br/>

비선점형은, 프로세스가 **자발적으로 wait 상태에 들어가거나, 실행이 끝났을 때만** CPU에 다른 프로세스를 올리는 것이 가능하다.

<br/>

선점형의 경우, **특정 우선순위를 기준으로 하여** CPU가 프로세스를 대체한다(동적/정적 우선순위).



> 마우스, 키보드 입력 렉이 걸리는 이유 ? --> 프로세스 스케쥴러가 Queue에 있는 프로세스를 제대로 처리하지 못하여 발생하는 현상!