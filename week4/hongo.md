# 임계구역 문제 해결 방법
## 경쟁 상황(Race condition)과 임계 구역(Critical section)

커널 데이터, 공유메모리등 여러 프로세스가 공유 자원을 동시에 읽고 쓰는게 가능하다면, 예상치못한 결과를 도출할 수 있다. (ex. 공유 변수 count의 값이 5일 때, 프로세스A는 count--을 하고, 프로세스B는 count++을 할 경우, 프로세스A와 B가 어떤 시점에 변수를 읽고 쓰냐에 따라 count의 값이 달라진다. 4 or 5 or 6 )

이처럼 여러 개의 프로세스가 공유 자원에 접근하고, 코드의 실행 순서에 따라 결과가 달라지는 상황을 경쟁 상황(Race condition)이라고 한다. 그리고 경쟁 상황이 발생할 수 있는 코드의 묶음을 임계 구역(Critical section)이라고 한다.

각 프로세스가 임계 구역을 독립적으로 실행해야, 예상치못한 결과를 막을 수 있다! 때문에 프로세스를 어떤 순서로 임계 구역에 들여보내, 안전하게 동기화를 할지 여러 해결방안들이 있는데, 이것을 알아보자~


### 임계구역 문제 해결안의 세가지 요구 조건

* 상호 배제(Mutual exclusion) : 특정 프로세스가 임계 구역에서 실행되고 있다면, 다른 프로세스가 침범할 수 없다.
* 진행(Progress) : 임계 구역에서 실행중인 프로세스가 아무도 없을 때, 임계 구역에서 실행하고자 하는 프로세스가 있다면 그 중 하나는 임계 구역에 들어갈 수 있어야한다. 
* 한정된 대기(Bounded Waiting) : 프로세스가 임계 구역에 들어가기 위한 대기시간이 무한대일수는 없다.



### 피터슨 알고리즘 - 소프트웨어적인 방법!

피터슨 알고리즘은 두 개의 프로세스가 있을 때, 사용할 수 있는 임계구역 문제 해결방법이다. 

프로세스0과 프로세스1이 존재한다고 가정하겠다.

```cpp
// 프로세스i의 코드

me = i;
other = 1-i;

while(true) {
  flag[me] = true;
  turn = other;
  while(flag[other] && turn == other) {
    // 대기! 
  }
  크리티컬 섹션 실행
  flag[me] = false;
}

```

위 알고리즘을 사용하면 임계구역 문제를 해결할 수 있다!

물론 여전히 문제가 존재하긴 하는데, `busy waiting`과 `최신 컴퓨터 아키텍처에서는 동작을 보장할 수 없다` 는 문제가 있다.

`Busy waiting`은 프로세스가 할당받은 cpu자원을 대기하는 데만 사용하는 것을 의미한다. `while(flag[other] && turn == other)`조건이 충족되어있다면 해당 프로세스는 언젠가는 크리티컬 섹션을 실행할 수는 있겠지만, 한 클럭동안 계속 기다릴수도 있는 것이다.

> `스핀락` : 위 코드처럼 락을 사용할 수 있을 때까지 대기하는 방식을 의미한다.
>
> 오래 대기하면 `Busy waiting` 문제가 발생하기에 락이 짧은 시간동안 유지될 때 많이 사용되는 방식이다. (근데 짧을지 길지 어떻게 앎? 통계로 때려맞추는거임?)
>
> **"락이 짧은 시간동안 유지된다"**의 기준은 문맥 교환을 두 번하는 시간보다 짧은 경우를 의미한다고 한다. 
>
> * 왜 두 번의 문맥 교환인가? : 락을 기다려야할 경우 두 번의 문맥 교환이 필요하기 때문
>   * 1. 대기 상태로 변경되는 문맥
>     2. 락이 사용가능해지면 대기를 풀기위한 문맥

`최신 컴퓨터 아키텍처에서는 동작을 보장할 수 없다` 는 문제도 있는데, 특정 아키텍처에서는 성능을 향상하기 위해 종속성이 없는 코드의 순서를 재정렬할수도 있기 때문이다. 
<details>
<summary>예시</summary>
<div markdown="1">

예를 들어 프로세스 i의 코드만 봤을 때 flag와 turn은 서로에게 영향을 주지않는, 종속성 없는 변수이므로 (1)과 (2) 코드의 줄이 재정렬이 되어 (2)가 앞으로 갈 수도 있다! 뭐이런등신같은 

```cpp
// 프로세스i의 코드

me = i;
other = 1-i;

while(true) {
  flag[me] = true; // (1)
  turn = other; // (2)
  while(flag[other] && turn == other) {
    // 대기! 
  }
  크리티컬 섹션 실행
  flag[me] = false;
}

```



그렇다면 코드는 아래처럼 변경되는데, 이 경우 `상호 배제` 조건을 만족하지 못한다.

```cpp
me = i;
other = 1-i;

while(true) {
  turn = other; // (2) 
  flag[me] = true; // (1)
  while(flag[other] && turn == other) {
    // 대기! 
  }
  크리티컬 섹션 실행
  flag[me] = false;
}
```

프로세스0이 먼저 실행되었다고 해보자!

* [프로세스0 실행중] turn = 1 <- 여기서 할당된 cpu사이클이 끝나 프로세스 1에게 cpu가 넘어갔다고 해보자!
* [프로세스1 실행중] turn = 0
* [프로세스1 실행중] flag[1] = true
* [프로세스1 실행중] **프로세스1이 크리티컬 섹션에 진입** <- flag[0]이 false라서 while문 빠져나옴
* [프로세스0 실행중] flag[ 0 ] = true
* [프로세스0 실행중] **프로세스0이 크리티컬 섹션에 진입** <- turn이 0이라서 while문 빠져나옴

프로세스 0과 1이 동시에 크리티컬 섹션에 들어가는 상황이 생길수도 있다!



### 메모리 장벽 (Memory Barriers)

그러나 시스템에는 메모리 장벽(== 메모리 모델)이라는 것이 존재하는데...

메모리의 변경사항을 모든 프로세스에게 전파하는 명령어이다. 메모리 장벽 명령어가 실행될 때, 시스템은 이후 코드에서 저장연산이 수행되기 전에 현재 시점의 모든 저장연산을 완료한다고 한다. 그래서 코드가 재정렬되더라도... 뭐.. 알아서 순서대로 잘 저장하는듯...

다음과 같이 메모리 장벽 명령어를 사용해서  `최신 컴퓨터 아키텍처에서는 동작을 보장할 수 없다` 문제를 해결할 수 있다. 

```cpp
me = i;
other = 1-i;

while(true) {
  flag[me] = true; // (1)
  memory_barrier(); // 메모리 장벽
  turn = other; // (2)
  while(flag[other] && turn == other) {
    // 대기! 
  }
  크리티컬 섹션 실행
  flag[me] = false;
}

```


</div>
</details>


<details>
<summary>turn이나 flag를 둘 다 사용해야하나? 하나만 사용하면 안되나?</summary>
<div markdown="1">


```cpp
me = i;
other = 1-i;

while(true) {
  flag[me] = true; // P0이 여기까지 실행한 후에 P1로 컨텍스트 스위칭된다면,
  while(flag[other]) { // P0과 P1의 flag 둘 다 true이므로 P0과 P1은 계속 서로의 flag가 false가 될 때까지 대기해야 함 - 한정된 대기 조건이 충족되지 않는다
    // 대기! 
  }
  크리티컬 섹션 실행
  flag[me] = false;
}
```

```cpp
me = i;
other = 1-i;

while(true) {
  turn = other;
  while(turn == other) {
    // 대기! 상대 프로세스가 turn을 내 것으로 만들어주지않는 이상 평생 대기
  }
  크리티컬 섹션 실행
}
```

</div>
</details>





## test_and_set과 compare_and_swap - 하드웨어적인 방법!

`test_and_set`과 `compare_and_swap`은 하드웨어에서 지원하는 원자적인 연산이다. 둘 다 어떤 변수의 값을 테스트하고, 새로 설정하는 작업을 하나로 묶어 원자적으로 실행하는 함수이다.

###  test_and_set을 사용한 문제 해결 예시

```cpp
boolean test_and_set(boolean *target) {
  boolean rv = *target;
  *target = true; // 새로운 값으로 설정
  return rv; // 오리지널 값 반환
}
```

```cpp
while(true) {
  while(test_and_set(&lock)) {
  	// 대기
  }
  크리티컬 섹션 실행
 	lock = false;
}
```

-> 두 프로세스가 동시에 test_and_set(&lock)을 실행하면 하나는 true가 반환되고, 하나는 false가 반환되어서 한 프로세스만 크리티컬 섹션에 진입한다.

-> 크리티컬 섹션을 끝낸 프로세스가 lock을 false로 바꿔주기전까지 다른 프로세스들은 test_and_set에서 true가 반환되어 계속 대기한다.



### compare_and_swap을 사용한 문제 해결 예시

```cpp
int compare_and_swap(int *value, int expected, int new_value) {
  int temp = *value;
  if (*value == expected) {
    *value = new_value; // 새로운 값으로 설정
  }
  return temp; // 오리지널 값 반환
}
```

```cpp
while(true) {
  while(compare_and_swap(&lock, 0, 1)) {
  	// 대기
  }
  크리티컬 섹션 실행
 	lock = false;
}
```



### 한정된 대기 문제까지 해결한 방법

위 두가지 방식은 상호배제와 진행은 만족하지만, 한정된 대기 조건은 만족하지 않는다. 프로세스0이 대기중인데 운좋게 다른 프로세스들만 계속 락을 얻을수도 있기 때문.

한정된 대기 문제를 해결하려면 아래와 같이 변형해볼 수 있다. waiting이라는 배열을 추가하여 프로세스의 대기 상태를 저장하고, 각 프로세스가 크리티컬 섹션을 끝낼 때, waiting 배열을 차례대로 순회하며 대기중인 프로세스에게 임계구역에 들어갈 수 있는 권한을 넘긴다. 이렇게 하면 프로세스들은 최대 n-1개의 프로세스가 한 번씩 임계구역에 들어가는 것만 대기하면 자신의 차례가 오는 것을 보장할 수 있다.

```cpp
waiting[n]; // 전부 false로 초기화
lock = false;
```

```cpp
me = i;

while(true) {
  localLock = true;
  waiting[me]
    
  while(waiting[me] && localLock == true) {
  	localLock = compared_and_set(&lock, 0, 1)
  }
  
  waiting[me] = false;
  크리티컬 섹션 실행
    
  nextProcess = (me + 1) % n;
  while((nextProcess!=me) && !waiting[nextProcess]) {
    nextProcess = (nextProcess + 1) % n;
  }
  
 	if (nextProcess == me) { // 대기중인 프로세스가 없다면
    lock = false; // 락만 풀어두고 종료
  }
  else { // 대기중인 프로세스가 있다면
    waiting[nextProcess] = false; // 해당 프로세스의 대기를 풀어줌
  }
}
```


피터슨 알고리즘과 하드웨어 메서드를 통해 임계구역 문제를 해결하는 방법을 알아보았다. 그러나 개발자가 매번 해당 코드를 작성하는 것은 번거롭다. 이를 위해 위에서 작성한 코드와 같은 기능을 제공하는 추상적인 자료형이 존재한다.

# 뮤텍스, 세마포어, 모니터
## 뮤텍스

락을 획득하는 `acquire()` 과 락을 반환하는`release()`메서드를 가지고 있다. `available` 은 위에서 써왔던  lock 변수와 마찬가지로 프로세스가 임계구역에 들어갈 수 있는가 여부를 결정한다.

```c++
acquire() {
  while (!available) {
    // 대기
  }
  available = false
}
```

```c++
release() {
  available = true;
}
```

```c++
while(true) {

	acquire()
	크리티컬 섹션 실행
	release()
}
```

`acquire()` 과 `release()` 는 원자적으로 실행되어야하므로 내부적으로  CAS연산을 사용한다고 한다.



## 세마포어

* 변수  `S` - 정수, 사용할 수 있는 공유자원의 개수를 의미
  * 카운팅 세마포어 : S가 2이상 / 이진 세마포어 : S가 1

* `wait(S)`와 `signal(S)`메서드 제공 (`P(S)`와 `V(S)`로 부르기도) - 얘네도 당연히 원자적인 메서드들임

```c++
wait(S) {
  while(S <= 0) {
    // 대기
  }
  S--;
}
```

```c++
signal(S) {
  S++;
}
```



### Busy Waiting을 해결한 세마포어 - Block&Wakeup방식

세마포어를 스핀 락방식으로 구현할수도 있지만,  다른 방식으로도 구현할 수 있다.

while문을 돌며 락을 얻을 때까지 계속 대기하는 것이 아니라, 대기해야 할 경우 대기큐에 해당 프로세스를 집어넣고, 바로 다른 프로세스에게 cpu를 할당하는 것이다. 각 세마포어마다 대기큐를 가지고 있다면 구현이 가능하다. (+ `block()`과 `wakeup(P)` 메서드 추가)

```cpp
typedef struct {
  int value;
  struct Process *list;
} Semaphore;
```

```cpp
// 변경된 wait(S)

wait(Semaphore *S) {
  S.value--;
  if (S.value < 0) { // 공유 자원들이 이미 전부 사용중이라면
    S.list.push(this);
    block()
  }
}
```

* 세마포어 변수가 음수가 될 수 있다. 음수일 경우 S를 얻기위해 대기중인 프로세스의 수가 된다.

```cpp
// 변경된 signal(S)

signal(Semphore *S) {
  S.value++;
  if (S.value <= 0) { // 누군가가 대기중이라면
    Process nextProcess = S.list.pop();
    wakeup(nextProcess);
  }
}
```

* 이전 방식의 `S`가 **사용할 수 있는 자원의 개수**를 의미한다면, 위 방식의 `S`는 **자원을 사용하기 위해 대기중인 프로세스의 개수**에 초점이 맞춰져있다.



#### `BusyWaiting` vs `Block&Wakeup`

`Block&Wakeup`또한 프로세스를 block하고 wakeup하는 오버헤드가 존재하나 일반적으로 `BusyWaiting`보다 효율적인듯. 다만 이전 포스팅에서 언급했듯이 대기 시간이 짧다면 `BusyWaiting`이 더 효율적일수도 있다...



### 뮤텍스 & 세마포어의 문제

#### 개발자의 능지 이슈

세마포어는 `wait(S)`이후에 `signal(S)`를 실행해야 올바르게 동작한다. 그러나 개발자의 실수로 두 연산의 순서를 바꾼다면 여러 개의 프로세스가 동시에 임계구역에 진입할 수도 있다.

```cpp
signal(mutex);
크리티컬 섹션 진행
wait(mutex);
```



즉, 개발자가 직접 `signal(S)`, `wait(S)`와 같은 메서드들을 호출하는 것이 문제이다!!!

때문에  `signal(S)`, `wait(S)`를 개발자가 직접 호출하지 않고, 더~~~ 추상화된 메서드를 제공하는 모니터가 존재한다.



## 모니터



모니터는 내부 메서드를 사용했을 때만 공유데이터에 접근할 수 있게 강제한다. 개발자가 직접 `wait(S)`, ` signal(S)`을 안해도 되는 추상자료형이 모니터다.

모니터는 `공유 데이터`, `공유 데이터에 접근하는 메서드`, `condition` 변수로 이루어져있다. 

* 모니터는 항상 하나의 프로세스만 내부 메서드에 접근할 수 있게 강제한다. 때문에 개발자가 락을 걸 필요가 없다. 혼자 알아서 Lock 개념을 사용한다. 세마포어와 가장 큰 차이~ (물론 모니터 내부적으로 세마포어 쓰고있긴 함~)

* 모니터는 하나 이상의  `condition` 변수를 가진다. `condition`은 큐를 가지고 있으며, PCB들을 저장해 특정 자원에 접근하기 위해 대기하는 프로세스들을 관리한다. `condition` 변수는 `wait()`와 `signal`연산을 통해서만 접근이 가능하다. 



예시를 통해 이해해보자. `Bounded-Buffer` 문제를 세마포어와 모니터를 사용해 해결해보자.



### 세마포어를 사용한 `Bounded-Buffer`문제 해결

(Bounded-Buffer문제 설명은 아래 챕터에~~)

* `empty`와 `full`은 대기중인 프로세스의 수를 나타내기 위한 세마포어 변수(카운팅 세마포어)
* `mutex` 임계 구역에 들어갈 수 있는지 여부를 나타내는 세마포어 변수(이진 세마포어) 

```cpp
empty = N; (0~N)
full = 0; (0~N)
mutex = true;
```

```cpp
// Producer

while(true) {
  wait(empty);
  wait(mutex);
  
	buffer.push(value);
  
  signal(mutex);
  signal(full);
}
```

```cpp
// Consumer

while(true) {
  wait(full);
  wait(mutex);
  
  int value = buffer.pop();
  
  signal(mutex);
  signal(empty);

}
```



### 모니터를 사용한 `Bounded-Buffer`문제 해결

```cpp
Monitor BoundedBuffer {
  int buffer[N];
  condition full, empty;
  
  void produce(int value) {
    if (empty.isFull()) {
      empty.wait();
    }
    empty.push(value);
    full.signal();
  }
  
  int consume() {
    if (full.isFull()) {
      full.wait();
    }
    int value = full.pop();
    empty.signal();
    return value;
  }
}
```



### Signal and wait & Signal and continue

프로세스 A와 B가 있다고 하자. 어떤 모니터의 condition에서 프로세스B가 대기중이다.

프로세스 A가 모니터의 `condition.signal()` 을 실행하면, B는 바로 실행될까? A의 코드가 전부 끝난 이후에 실행되어야 할까?

* `Signal and wait` 은 A가 signal을 호출한 후에 wait상태가 되는 것을 의미하고,
* `Signal and continue`는 A가 signal을 호출해도 계속 A의 코드를 이어서 실행하는 것을 의미한다.

어떤 게 더 좋을까?!  잘 모르겠다~ `Signal and wait` 은 `signal()`을 호출할 때마다 대기해야하니 컨텍스트 스위칭 비용이 더 높을 수도 있고,  `Signal and continue`는 continue하는 동안 B가 다시 대기 상태에 빠져버리는 경우가 생길수도 있다.

Java는 기본적으로 `Signal and continue` 방식을 사용한다는듯



### Deadlock & Starvation

위 상황의 경우, 상대 프로세스가 먼저 자원을 획득하고 있어 데드락이 발생할 수 있다.

두 프로세스가 S -> Q와 같이 wait하는 순서를 동일하게 맞춘다면 데드락이 발생하지 않으나, 개발자가 항상 자원의 획득 순서를 염두하고 있어야한다는 불편함이 있다. (이외 해결방안은 나중에 ~...)

또한, 여전히 Starvation문제도 존재한다. 프로세스가 세 개 이상일 때, 각 프로세스를 실행하는 데 우선순위가 있다면, 우선순위가 늦은 프로세스는 오래 대기할 수도 있다. <- 에이징으로 해결 가능


# 고전적인 동기화 문제들

## 유한 버퍼 문제(Bounded-Buffer Problem)

여러 개의 생산자와 소비자가 존재하는 버퍼가 있을 때 생산자들간의 경쟁, 소비자들간의 경쟁을 해결해야한다.

* 하나의 생산자만 버퍼에 데이터를 쓸 수 있다, 버퍼가 가득차면 생산자는 대기해야한다. -> 쓰기 작업 대기 관리 필요
* 하나의 소비자만 버퍼에서 데이터를 읽을 수 있다, 버퍼가 비어있으면 소비자는 대기해야한다. -> 읽기 작업 대기 관리 필요
* 생산자가 버퍼에 데이터를 쓸 때는 소비자가 데이터를 읽어올 수 없다. (반대의 상황도 마찬가지) -> 버퍼 접근에 대한 락 필요



이전 포스팅에서 나왔었지만, 세마포어로 구현하면 다음과 같다.

쓰기와 읽기 작업에 대한 대기를 관리하기 위해 `full`과 `empty` 세마포어를 사용했다. ( 0 <= `full`, `empty`<= N)

버퍼 접근을 관리하기 위해 `mutex` 세마포어를 사용했다. (  `mutex` = 0 or 1)

```cpp
// Producer

while(true) {
  wait(empty);
  wait(mutex);
  
	buffer.push(value);
  
  signal(mutex);
  signal(full);
}
```

```cpp
// Consumer

while(true) {
  wait(full);
  wait(mutex);
  
  int value = buffer.pop();
  
  signal(mutex);
  signal(empty);

}
```



## Readers-Writers 문제

* Reader들끼리는 DB에 동시에 접근할 수 있다.
* Writer가 DB에 접근할 때는 다른 프로세스들이 접근할 수 없다.



#### 세마포어 코드 예시

* `rw_mutex` : Writer를 독립적으로 실행하기 위한 이진 세마포어

```cpp
// Writer

while(true) {
  wait(rw_mutex);
  쓰기 작업
  signal(rw_mutex);
}
```



* `mutex` : read_count를 조회, 갱신하는 것을 원자적으로 수행하기 위한 이진 세마포어

```cpp
// Reader

while(true) {
  wait(mutex);
  read_count++;
  if (read_count == 1) { // 최초로 진입하는 reader
    wait(rw_mutex);
  }
  signal(mutex);
  
  읽기 작업
    
  wait(mutex);
  read_count--;
  if (read_count == 0) { // 마지막으로 빠져나가는 reader
    signal(rw_mutex);
  }
  signal(mutex);
}
```



#### 위 코드의 문제 - Starvation

위 코드는 임계 구역에 들어가 있는 Reader가 있을 경우, 다른 Reader들이 락을 얻지 않고도 바로 조회할 수 있는 구조이다. 때문에 어떤 Reader가 임계구역을 빠져나가기 전에, 다른 Reader들이 계속 들어온다면, Writer가 실행되지 못할 수도 있다. 

#### Starvation 해결 방안

* Writer가 임계 구역 진입 요청을 하면 새로운 Reader는 임계 구역에 들어가지 못하게 한다. -> 반대로 Reader Starvation 일어남
* 동시에 임계 구역에 접근할 수 있는 Reader의 수를 제한
* Reader들이 임계 구역에 들어갈 수 있는 시간을 제한
* 또 다시 등장하는 우선순위와 에이징



> Reader-Writer 락은 Writer보다 Reader의 개수가 많을 때 유용하다. 일반적인 세마포어보다 Reader-Writer의 오버헤드가 크기 때문에 Reader의 개수가 적으면 비효율적일 수 있으나, Reader의 개수가 많다면 여러 개의 Reader가 락을 얻지 않고도 실행 가능하기 때문.



## 식사하는 철학자들 문제(The Dining-Philosophers Problem)

N명의 철학자와 N개의 젓가락이 있을 때, 각 철학자들이 양 옆의 젓가락을 사용해서 식사하는 경우 발생하는 동기화 문제이다.

* 철학자는 한 번에 한 개의 젓가락만 집는 것도 가능하다. 꼭 두 젓가락을 동시에 집을 필요는 없다. (물론 식사는 두 젓가락이 있어야만 가능)



#### 세마포어 코드 예시 - 데드락 위험

* `chopsticks[i]` : 각 젓가락에 대한 접근 권한을 나타내는 이진 세마포어

```cpp
while (true) {
	wait(chopstick[i]);
  wait(chopstick[(i+1) % 5]);
  
  식사
    
  signal(chopstick[i]);
  signal(chopstick[(i+1) % 5]);
}
```

이 경우 모든 철학자들이 자신의 왼쪽에 있는 젓가락을 잡으면 데드락 발생.

데드락 해결 방안이 있긴 함

* 한 번에 같은 방향의 젓가락을 못 집게 하면 됨. (ex. 짝수번째 철학자는 오른쪽 젓가락을 먼저 집고, 홀수번째 철학자는 왼쪽 젓가락을 먼저 집는다.)
* 철학자 수를 한 명 줄임.
* 젓가락을 집을 때 양 쪽의 젓가락을 모두 집을 수 있을 때만 집게 함.

근데 위 처럼 해도 starvation까지 해결되는 건 아님.



#### 모니터 코드 예시

* `enum {THINKING, HUNGRY, EATING} state [N];`
  * i번째 철학자가 HUNGRY상태이고, 양쪽 철학자들이 식사하지 않을 때만 `state[i] = EATING`으로 설정할 수 있다.



* `condition self[N];`
  * 식사하고 싶은 철학자들이 대기하기 위한 변수

```cpp
Monitor DiningPhilosophers {
  enum {THINKING, HUNGRY, EATING} state [N];
  condition self[N];
  
  void pickup(int i) {
    state[i] = HUNGRY;
    test(i);
    if (state[i] != EATING) {
      self[i].wait(); // 인접한 철학자가 젓가락을 다 사용한 후에 깨워줘야함
    }
  }
  
  void putdown(int i) {
    state[i] = THINKING;
    test((i+1) % N); // 식사후에는 대기중인 인접한 철학자들을 깨워줌
    test((i+N-1) % N);
  }
  
  void test(int i) {
    if(state[(i+1) % N] != EATING && state[(i+N-1) % N] != EATING && state[i] == HUNGRY) {
      state[i] = EATING;
      self[i].signal(); // signal()은 대기중인 애들이 없다면 아무 작업도 하지 않는다
    }
  }
}
```

```cpp
DiningPhilosophers.pickup(i);
식사
DigingPhilosophers.putdown(i);
```

위 코드의 경우 양 옆의 젓가락을 동시에 잡을 수 있는 상황에서만 젓가락을 집으므로 데드락이 발생하지는 않는다. 물론 starvation 문제가 생길 수는 있음. (starvation 해결하려면 또선순위 설정하던가, 대기리스트에 배고픈 철학자들 다 집어넣고, 누가 젓가락 내려놓을 때 대기리스트 순회하면서 식사 가능한 애들한테 권한 주던가~~)



#  Java에서의 동기화

## Java 모니터

Java에서는 `synchronized` 메서드와 블럭을 사용해 하나의 스레드만 임계 구역에 진입하게 할 수 있다. `synchronized`는 Java 모니터를 사용한다. 인스턴스와 연결된 락을 획득해야만 임계 구역에 진입할 수 있고, 진입 집합(entry set)과 대기 집합이라는 것을 제공한다.

* `synchronized` 블록 또는 메서드를 벗어나면 자동으로 락이 해제된다. 예외가 발생하더라도 자동으로 락이 해제된다. (finally문 쓰는듯)

* `진입 집합` : 스레드A가 락을 획득하려는데, 다른 스레드가 이미 락을 사용중일 경우, 스레드A는 진입 집합으로 들어간다.
* `대기 집합` : 락을 얻는 조건은 만족할 수 있지만, 특정 조건이 충족되지않아 코드를 더 이상 실행할 수 없는 스레드는 대기 집합으로 들어간다. ("특정 조건이 충족되지 않았다"의 예시: 버퍼의 Reader인데 버퍼가 비어있을 경우)



**`synchronized`내부에서는 `wait()`, `notify()`, `notifyAll()`메서드를 사용할 수 있다.**

* `wait()` : 스레드를 대기 집합으로 넣는다.
  * 스레드가 획득했던 락을 반환한다.
  * 스레드의 상태를 "봉쇄됨"으로 변경한다.
  * 스레드를 대기 집합에 넣는다.

* `notify()` : 대기 집합의 스레드를 진입 집합으로 옮긴다.

  * 대기 집합에서 임의의 스레드A를 선택한다.

  * A를 대기 집합에서 진입 집합으로 옮긴다.

  * A의 상태를 "봉쇄됨"에서 "실행 가능"으로 변경한다.



## 재진입 락(Reentrant Locks)

공유 자원에 대한 상호 배타적 접근을 제공한다. 명시적으로 락을 획득하고 해제하는 메서드(`lock()`과 `unlock()`)를 제공한다. 즉, 개발자가 직접 락의 획득, 해제를 관리하는데 장점일수도 단점일수도~~~ 예외 발생 시 자동으로 락을 해제하지 않으므로, `finally` 블록 내에서 `unlock()`을 명시적으로 호출해야 한다.

Java모니터와 달리 락의 상태를 확인하거나, 락 획득 시 시간 제한을 설정하는 등 더 세밀한 제어가 가능하다.

- 공정성 설정 가능 : 오래 기다리는 스레드에게 우선적으로 락을 줄 수 있다.
  - `new ReentrantLock(true); // 공정한 락 설정` 생성할 때, true 넣으면 오래 기다리는 애 우선적으로 락 주는듯
- **조건 변수 지원** : `Condition`객체를 사용해 다양한 조건에 따른 스레드 대기 및 깨우기가 가능하다.

```java
Lock lock = new ReentrantLock();
Condition condition = lock.newCondition(); 

// condition은 await()와 signal()을 가진다.
condition.await();
condition.signal();
```



## 세마포어

`Semaphore`객체 생성 가능.

```java
Semaphore sem = new Semaphore(1); // 세마포어 초기값 할당. 음수도 가능.

try {
  sem.acquire();
  크리티컬 섹션
} catch (InterruptedException e) {
  ...
} finally {
  sem.release();
}
```

