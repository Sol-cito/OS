## Note11. 쓰레드의 이해 Part 2

#### 쓰레드와 동기화(Synchronization) 이슈

쓰레드는 한 프로세스의 자원을 공유한다고 하였다.

그러다보니 여러 쓰레드가 공유하고있는 한 프로세스의 변수 등의 Data에 쓰레드가 동시에 접근하게 된다면, 

**의도치 않은 비정상적인 결과**가 도출될 가능성이 존재한다.

따라서 이러한 상황을 막기 위해 **동기화이슈를 관리**해야 하는 상황이 발생한다.

<br>

아래 python 예시 코드를 보자.

```python
import threading # 파이썬의 쓰레드 생성 외부 라이브러리

g_count = 0 # 글로벌 변수 선언

def thread_main(): # 글로벌 변수 g_count에 1을 십만 번씩 더하는 함수
    global g_count
    for i in range(100000):
        g_count +=1
        
threads = []

for i in range(50): # list에 50개의 쓰레드를 붙인다.
    th = threading.Thread(Target = thread_main)
    threads.append(th)

for th in threads: # list 내 쓰레드의 함수를 실행시킨다.
    th.start()

# join() 은 쓰레드 간 동기화 함수
# 모든 쓰레드의 실행이 종료될 때까지 대기하게 만듬.
for th in threads: 
    th.join()
    
print('g_count = ', g_count)
```

위 코드를 실행시켰을 때, **동기화 이슈가 발생할 수 있다.**

그 이유는 아래와 같다.

- g_count +=1 이라는 코드를 실행시켰을 때, 컴퓨터 내부 구조는 다음과 같이 동작한다(컴퓨터 구조에 대한 이해 필요).
  - (1) g_count의 현재값을 register에 저장한다.
  - (2) register 값을 1 증가시킨다.
  - (3) register에 기록된 값을 g_count 메모리 값에 덮어씌운다.

- 그런데 현재 위의 thread_main 함수에서 위 (1), (2), (3)을 실행하는 반복문이 100000번 이므로, **만약 연산 시간이 길다면 쓰레드 사이에서 반복문 중간에 Context Switching이 발생할 수 있다** (반복문이 100000번이 아니라 100번 정도라면, 연산 속도가 짧으므로 Context Switching이 발생하지 않음).

- Context Switching이 발생할 경우 다음 쓰레드가 thread_main 함수를 실행시키면서, **동일 register를 초기값 0으로 만들고 g_count 값을 덮어씌우고 연산을 시작한다.**
- 다시 Context Switching이 발생하면, **동일한 register에 값이 덧씌워지고 다시 끝난 지점부터 연산을 시작하게 된다.**

- 이처럼 쓰레드 사이의 Context Switching이 빈번하게 발생하다 보면 register에 저장된 값이(변수값) **계속 변화하므로 연산 결과값이 제대로 반영되지 못할 수 있다.**

이것이 바로 **쓰레드 간 변수를 공유함에 따라 발생하는 쓰레드 동기화 이슈**의 한 예이다.

<br>

그럼, 저기서 문제가 되는 thread_main() 함수의 연산 부분을 아래와 같이 바꾸면..

```python
def thread_main():
    global g_count
    lock.acquire() # 락을 걸어준다.
    for i in range(100000):
        g_count +=1
    lock.release() # 락이 풀린다.
    
lock = threading.Lock()
```

lock.acquire() 함수를 특정 쓰레드가 실행하게 되면, release될 때 까지는 다른 쓰레드가 acquire() 함수를 실행하지 못하고 기다리게 된다.

따라서 for문 연산은 **하나의 쓰레드만 접근할 수 있게 된다.**

위와 같은 기법을 **Mutual Exclusion**(상호 배제)라고 하며, 

쓰레드끼리 겹치는 for문 연산과 같은 영역을 **임계 영역**이라고 한다.

임계 영역에 여러 스레드가 들어갈 수 있게 만드는 기법을 **세마포어**라 한다.

<br>

