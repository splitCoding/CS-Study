# 5 CPU Scheduling

#### 5.1 Basic Concepts

다중 프로그램드된 OS의 필수요소

다중 프로그래밍 목적: CPU 이용률을 최대화 하기 위해 항상 실행중인 프로세스를 가지려고

다중 프로그래밍을 위해 스케줄링: 프로세스 대기 → OS는 CPU를 회수 → 다른 프로세스에 할당

# 5.1.1 CPU Scheduler

CPU 유휴 상태 → OS는 준비 큐에 있는 프로세스 중 하나 선택

이때 **CPU 스케줄러**에 의해 선택 절차 수행

> 이때 준비 큐는 반드시 FIFO 방식 큐가 아니어도 됨

# 5.1.2 CPU-I/O Bust Cycle

프로세스 실행은

- **CPU execution**과 **I/O wait** 사이클로 구성
- CPU Bust로 시작해서 CPU 버스트로 끝남: CPU Bust → I/O Bust → CPU Bust → ... → CPU Bust
- 이때 CPU Bust 지속 시간을 측정

![dbbe44d3-867a-461d-93f8-d29fa28aca71--Screenshot 2024-02-07 at 15.29.20.png](https://supernotes-resources.s3.amazonaws.com/image-uploads/dbbe44d3-867a-461d-93f8-d29fa28aca71--Screenshot%25202024-02-07%2520at%252015.29.20.png)

- 긴 버스트 시간 < 짧은 버스트 시간
- CPU 지향 프로그램인지, I/O 지향 프로그램인지에 따라 분포는 달라질 것

# 5.1.3 Preemptive and NonPreemptive Scheduling

스케줄링 결정은 4가지 상황에서 발생

1. 프로세스가 **실행** → **대기** 상태로 전환: I/O 요청 혹은 `wait()`호출
2. 프로세스가 **실행** → **준비 완료** 상태 전환: 인터럽트 발생
3. 프로세스가 **대기** → **준비 완료** 상태 전환: I/O 종료
4. 프로세스 종료

1,4의 경우, 새로운 프로세스가 반드시 선택. 2,4의 경우는 선택의 여지가 있음

- 1,4에서만 스케줄링이 발생할 경우(즉 프로세스가 직접 CPU를 반납할 경우): **비선점**, 혹은 **협조적**
- 나머지는 **선점**

선점 스케줄링에서는 **경쟁 조건 초래 가능**

OS는 거의 항상 인터럽트를 받아들일 필요가 있고, 인터럽트에 의한 영향을 받는 코드 부분은 **동시 사용**으로부터 보호 되어야함

# 5.1.4 Dispatcher

디스패처는 Core의 제어를 CPU 스케줄러가 선택한 프로세스에 주는 모듈

- 다른 프로세스로 문맥 교환
- 사용자 모드 전환
- 프로그램을 다시 시작하기 위해 사용자 프로그램 적절한 위치로 이동

문맥 교환시 호출되므로, 가능한 최고로 빨리 수행되어야함. 이때 소요되는 시간을 **디스패치 지연(dispatch latency)**

#### 5.2 스케줄링 기준

사용되는 기준:

- CPU 이용률(utilization)
- 처리량(throughput): 단위 시간당 완료된 프로세스의 개수
- 총 처리 시간(turnaround time): 프로세스 제출 시간과 완료 시간의 간격, 준비 큐 대기 시간
- 대기 시간(waiting time): 준비 큐에서 대기하면서 보낸 시간의 합
- 응답 시간(response time)

| 기준 이름                        | 기준값                                 |
| ---------------------------- | ----------------------------------- |
| CPU 이용률(utilization)         | 0 - 100 %                           |
| 처리량(throughput)              | 단위 시간당 완료된 프로세스의 개수                 |
| **총 처리 시간(turnaround time)** | **준비 큐 대기 시간 + CPU 실행 시간 + I/O 시간** |
| **대기 시간(waiting time)**      | **준비 큐 대기 시간의 합**                   |
| 응답 시간(response time)         | 응답이 시작되는데까지 걸린 시간                   |

#### 5.3 스케줄링 알고리즘

# CPU Scheduling Problem

- ready queue에 있는 어느 프로세스에게 CPU를 할당해 줄 것인가

## Solutions:

- FCFS: First-Come, First-Seved(문제가 많다)
- SJF: Shortest Job First(SRTF: Shortest Remaining Time First)
- RR: Round Robin - 시분할Time Sharing과 관련 있음
- Priority  based
- MLQ: Multi-Level Queue
- MLFQ: Multi-Level Feedback Queue

## FCFS Scheduling

- the simplest CPU-Scheduling Algorithm
- The process that requests the CPU first
- FIFO queue로 구현 가능
- **Average waiting time** highly depends on **CPU-burst times**
- FCFS = Non-preemptive
- Convoy Effect(호송차 효과)
   - 다른 프로세스들이 CPU 타임이 긴 프로세스로 인해 대기

## SJF Sheduling

- shortest-next-CPU-busrt-first scheduling
- 단순히 CPU Bust 타임이 작은 순서대로 CPU 할당
- 최소 average waiting time을 가짐을 입증 가능하다.
- 왜냐하면 짧은 프로세스를 긴 프로세스보다 먼저 처리할 경우, 항상 `줄어든 짧은 프로세스 대기 시간 > 늘어난 긴 프로세스 대기시간`이기 때문
- 구현에서는?
   - next CPU burst을 알 수 있는 방법이 없다.
- 따라서 대략적으로 구현해야한다.
   - 예측하기
- 예측은 어떻게 하나?
   - 통계적 방식: 과거 CPU Burst 측정 값을 통해 **지수적 통계** 내보기
   - $$\tau_{n+1}= \alpha \tau_{n} + (1-\alpha)\tau_n$$ where
      - $$\tau_n$$은 n번째 CPU burst 길이
      - $$\tau_{n+1}$$은 그 다음 예측한 CPU Burst 길이
      - $$\alpha$$ such that $$0<= \alpha <= 1$$
- 선점형과 비선점형 둘 중 하나 가능
   - 기존 프로세스 진행 중에 더 짧은 새로운 프로세스가 도착할 경우에 더 짧은 프로세스를 선점형으로 처리 가능

## SRTF Scheduling

- Shortest-Remaining-Time-First: Preemptive SJF Scheduling
- SRTF는 현재 실행중인 프로세스를 선점

## RR Scheduling

- Round  Robin: preemptive FCFS with time quantum
- A time quantum is a small unit of time (10ms to 100ms)
- Ready queue = circular queue
- 만약, one time quantum 보다 CPU burst가 작은 프로세스는?
   - 프로세스가 자발적으로 CPU 반납
   - 스케줄러가 다음 프로세스 실행
- 길다면?
   - 타이머가 종료되고 OS에 인터럽트
   - 컨텍스트 스위치
- 평균 대기 시간은 RR에서 좀 더 길어질 수 있다.
- RR은 **선점형**
- **Time Quantum**을 얼마로 주냐에 따라 성능이 달라진다.

## Priority-base Scheduling

- 각 프로세스에 우선순위 배정
- 가장 높은 우선순위 먼저 처리, 같으면 FCFS
- SFJ는 Priority-based Scheduling의 한 케이스
- **선점형** 혹은 **비선점형**이 될 수 있음
- **Starvation**(indefinite blocking) 가능
   - 프로세스가 처리되지 않고 대기만 하는 경우
   - 해결방안? **aging - 점차적으로 우선순위가 높아짐**
- 구현에 있어서는 RR과 Priority 섞어서 사용

## Multi-Level Queue Scheduling

- 각 우선순위 별로 큐를 따로 주는 방식
- 우선순위가 낮은 다른 큐의 프로세스가 실행되지 않을 수 있음

#### 5.4 스레드 스케줄링

대부분의 현대 OS는 실제적으로는 프로세스가 아닌 커널 스레드 스케줄링을 관리

- user thread는 thread library가 담당
- 커널 스레드는 유저 스레드를 맵핑만 해주면 됨

#### 5.5 실시간 CPU Scheduling

- Soft Realtime vs Hard Realtime
- Soft는 중요한 프로세스가 중요하지 않은 프로세스보다는 먼저 실행되도록 보장
- Hard는 일이 반드시 데드라인 전에 끝내도록 작업


