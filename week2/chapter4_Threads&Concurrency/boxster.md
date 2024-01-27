
# Chapter 4 스레드와 병행성

## 개요
스레드는 cpu 이용의 기본 단위

스레드 Id, program counter(PC), 레지스터 집합, 스택으로 구성됨
스레드는 같은 프로세스에 있는 다른 스레드와 코드, 데이터 섹션, 파일, 신호와 같은 운영체제 자원을 공유한다


## 다중 스레드 장점

1. 응답성 : 응용 프로그램이 긴 작업을 수행하더라도, 프로그램 수행이 계속되는 것을 허용하여 사용자의 응답성을 증가시킨다,.
2. 자원 공유 : 프로세스는 공유 메모리, 메시지 전달로 자원을 공유해야하지만 스레드는 자동으로 공유할 수 있다.
3. 경제성 : 프로세스를 생성하는 것은 비용이 많이 든다. 하지만 스레드는 경제적이다.
4. 규모 적응성 : 다중 처리기 구조에서 각각의 스레드가 다른 처리기에서 병렬로 수행될 수 있어서 이득이다.


## 다중 코어 프로그래밍

여러 개의 CPU를 사용

병행성은 모든 작업이 진행되게 하여 둘 이상의 작업을 지원하는 것
병렬성은 둘 이상의 작업을 동시에 수행할 수 있는 것

하지만 프로세스는 병행하게 실행되고 빠른 전환으로 병렬성으로 보이게끔 한다.

### 프로그래밍 도전 과제

병렬처럼 보이게끔 스케줄링 알고리즘을 개발할 필요가 있다.

그럴 때 필요한 5가지의 필요조건
1. 테스크 인식 : 병행 가능한 태스크를 나눌 수 있도록 영역을 찾는 작업이 필요
2. 균형 : 작업에 균등한 기여도를 가지도록 태스크를 나누는 것이 중요
3. 데이터 분리 : 독립된 태스크로 나누는만큼 태스크에 대한 데이터도 독립적으로 관리하는 것이 좋다
4. 데이터 종속성 : 태스크가 접근하는 데이터를 둘 이상 태스크 사이에 종속성이 있는지 확인하여함
5. 시험 및 디버깅 : 여러 태스크가 병렬로 실행될 때 다양한 실행 경로가 존재하여야 함

### 병렬 실행의 유형

1. 데이터 병렬 실행 : 동일한 데이터의 부분집합을 다수의 계산 코어에 분배 후 코어에서 동일한 연산을 실행.
2. 태스크 병렬 실행 : 태스크를 다수의 코어에 분배하고 각 스레드는 고유 연산을 실행한다.

두 가지 전략을 혼합해서 사용할 수 있다.

## 다중 스레드 모델

스레드는 사용자 스레드와 커널 스레드가 있다.

### 다대일 모델 (사용자가 many)

그린 스레드 (solaris 시스템을 위한 스레드 라이브러리?) 가 다대일 모델을 사용했다, java의 초기 버전에도 사용됨
하지만 다중 처리 코어의 이점을 살릴 수 없어서 이제 사용하지 않음

### 일대일 모델

하나의 스레드가 봉쇄적 시스템 콜을 호출하더라도 다른 스레드가 실행될 수 있어서 다대일보다 더 많은 병렬성을 제공
하지만 사용자 스레드를 만드려면 매번 커널 스레드를 만들어야하기 때문에 시스템에 많은 부하를 줄 수 있다. 
그래서 Linux는 windows 운영체제 제품과 함께 일대일 모델을 구현

### 다대다 모델

다대일 모델과 일대일 모델의 단점을 어느 정도 해결
많은 사용자 스레드를 적거나 같은 수의 커널 스레드로 멀티플렉스 시킨다. 하지만 구현이 어렵다.
(살짝 이해안됨 설명 바람)

## 스레드 라이브러리

프로그래머에게 스레드를 생성하고 관리하기 위한 API를 제공. 스레드 라이브러리는 사용자 공간에서만 작동되게 하거나, 커널 영역까지 존재하는 라이브러리를 제공할 수 있다

스레드 생성법
1. 비동기 스레딩 : 부모가 자식 스레드를 생성한 후 부모의 실행을 재개하여 부모, 자식 스레드가 서로 독립적으로 실행 그래서 서로 데이터 공유 거의 없음
2. 동기 스레딩 : 부모 스레드가 하나 이상의 자식 스레드를 생성하고 자식 스레드가 모두 종료되면 부모를 실행. 그래서 데이터 공유를 많이할 수 있음. (ex, 부모 스레드는 자식 스레드의 계산 결과를 통합할 수 있다.)

### Pthreads

POSIX가 스레드 생성과 동기화를 위해 제정한 표준 API (그냥 명세)

## 암묵적 스레딩

스레딩에 생성과 관리 책임을 컴파일러와 런타임 라이브러리에게 넘겨주는 것

### 스레드 풀

스레드를 계속 만들어 제공한다면 언젠가 cpu 시간, 메모리 공간 같은 자원이 고갈됨. 그래서 스레드 풀을 사용

스레드 풀은 프로세스를 시작할 때 일정한 수의 스레드들을 미리 풀로 만들어두는 것이다.

스레드 풀의 장점
1. 새 스레드를 만들어주는 것보다 속도가 종종 빠르다.
2. 스레드의 개수에 제한을 두어 스레드를 병렬 처리할 수 없는 시스템에 도움이 된다.
3. 태스크를 생성하는 방법을 태스크에서 분리하면 태스크 실행을 다르게 할 수 있다.

### Fork Join

부모 스레드가 하나 이상의 자식 스레드를 생성(fork)한 다음 자식의 종료를 기다린 후 join하고 자식의 결과를 확인하고 결합할 수 있다.
fork join 모델은 라이브러리가 생성할 실제 스레드 수를 결정하는 동기 버전의 스레드 풀이다.

## 스레드와 관련된 문제

### Fork(), Exec() 시스템 콜
보통 fork()를 실행하면 모든 스레드를 복사하거나, 호출한 스레드만 복사하는 경우가 있다.
exec()를 호출하면 모든 스레드를 포함한 전체 프로세스를 대체한다.

그래서 fork()의 두가지 버전 중 어느 것을 선택할 지는 응용 프로그램에 달려있다. fork 를 하자마자 다시 exec를 한다면 불필요하게 스레드를 복제하기 때문이다.

### 신호 처리
신호는 프로세스에 어떤 이벤트가 일어났다고 알려주기 위해 사용된다. 신호는 이벤트의 이유에 따라 동기 혹은 비동기로 전달된다.

하지만 신호를 보내는 시나리오는 아래와 같다
1. 신호는 특정 이벤트가 일어나야 생성된다.
2. 생성된 신호가 프로세스에 전달된다.
3. 신호가 전달되면 반드시 처리되어야 한다.

이하 중요하지 않다 판단하여 생략
...