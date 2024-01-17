# 컴퓨터 시스템 구조
- 각 IO 디바이스들에 Device Controller가 존재
	- 디바이스 전담 작은 CPU와 같음
	- 작업 공간 = local buffer
- CPU는 메모리에 올라간 명령 Instruction만 수행
	- Register: 메모리보다 빠른 정보 저장 공간
	- Mode Bit: CPU에서 실행되는 프로그램이 사용자인지, 시스템인지 알려줌
- IO에게 일 시키는 법
	- IO Controller에게 작업 시킴
	- Local Buffer에 데이터 저장
	- IO 작업이 오래 걸림 → 컨트롤러가 알아서 실행
	- CPU는 메모리 접근을 하면서 다른 명령 수행
- Interrupt Line
	- 메모리 인스트럭션을 수행하다가 중간중간 점검
	- IO, Timer 등 작업 결과나 예외 상황 전달
	- OS에게 제어권이 돌아감
- Timer
	- 특정 프로그램의 CPU 독점 방지

## Mode bit
- 0 모니터 모드: OS 코드 수행
- 1 사용자 모드: 사용자 코드 수행 (한정됨)

보안상의 이유로 보호 장치 필요
- Interrupt나 Exception 발생시 mode bit을 0으로 바꿈
- 사용자 프로그램에게 CPU를 넘기기 전에 mode bit을 1로 세팅

## Timer
- 정해진 시간이 흐른 뒤 OS에 제어권이 넘어가도록
- 타이머는 매 **클럭**마다 1씩 감소
- 타이머 값이 0이 되면 타이머 인터럽트 발생
- CPU를 특정 프로그램이 독점하는 것으로부터 보호
- 타이머는 time sharing을 구현하기 위해 이용됨
- 현재 시간 계산에도 사용

## Device Controller
- I/O 전담 작은 CPU
- control register, system register 등이 있음
- I/O는 실제 디바이스와 local buffer 사이에서 일어남
- interrupt를 통해 작업이 끝난 것을 알림
- 헤드 스핀 등 관리
- 각 장치를 통제하는 일종의 작은 CPU → HW

### Device Driver
- OS 코드 중 각 장치별 처리 루틴, CPU가 장치를 호출하기 위한 코드 → SW

다만 Interrupt가 너무 많이 걸림 → **DMA Controller**

## Direct Memory Access (DMA) Controller
CPU에 Interrupt의 빈도를 줄이기 위해, 디바이스들로부터 받은 정보를 복사해 인터럽트를 줄임

## I/O 수행
- 모든 입출력 명령은 모니터 모드에서만 가능
- 사용자 프로그램은 어떻게 I/O를 하는가?
	- System Call(시스템 콜)
	- 기존 사용자 코드에서는 메모리 주소 점프를 통해 이동
	- 하지만 시스템 콜의 경우는 메모리 주소 점프 불가능(모니터 모드가 아니라서)
	- 따라서 Interrupt를 통해 시스템 콜
- 올바른 요청인지 확인하는 작업

### 수행 과정
1. 소프트웨어 인터럽트를 통해 시스템 콜
2. 하드웨어 인터럽트를 통해 작업 완료 알림

## Interrupt
- 인터럽트 당한 시점의 레지스터와 Program Counter를 저장한 뒤 CPU의 제어를 인터럽트 처리 루틴(OS)에 넘긴다.
- 넓은 의미
	- 하드웨어 인터럽트: 하드웨어(디바이스, 타이머 등)가 발생
	- Trap(SW Interrupt)
		- Exception: 오류
		- System Call: 프로그램이 커널 함수를 호출
- 관련 용어
	- 벡터
		- 해당 이터럽트의 처리 루틴 주소를 가지고 있음
	- 처리 루틴(Interrupt Service Routine, Interrupt Handler)
		- 해당 인터럽트를 처리하는 커널 함수

# 동기식 입출력, 비동기식 입출력

## 동기식 입출력
- I/O 요청 후 작업이 완료된 후에야 **제어**가 **사용자 프로그램**에게 넘어감

### 구현 방법 1.
- I/O가 끝날 때까지 CPU를 낭비
- 매시점 하나의 I/O만 일어날 수 있음
### 구현 방법 2.
- I/O가 완료될 때가지 해당 프로그램에게서 CPU를 빼앗음
- I/O 처리를 기다리는 줄에 그 프로그램을 줄 세움
- 다른 프로그램에게 CPU를 줌
- CPU가 놀지 않고 일할 수 있음

## 비동기식 입출력
- I/O가 시작된 후 입출력이 **작업이 끝나기를 기다리지 않고** **제어가 사용자 프로그램에 즉시 넘어감**

두 경우 모두 I/O 완료 시에 인터럽트 발생시킴

# DMA
- CPU 외의 메모리 접근 장치
- 빠른 입출력 장치를 메모리에 가까운 속도로 처리하기 위해 사용
- CPU의 중재 없이 Device Controller가 Device의 Buffer Storage의 내용을 메모리에 block 단위로 직접 전송
- 바이트 단위가 아닌 block 단위로 인터럽트 발생

# 서로 다른 입출력 명령어
- I/O를 수행하는 특별 명령어에 의해
- Memory Mapped I/O에 의해
	- I/O 디바이스들에 메모리 연장 주소를 부여

# 저장 장치 계층 구조
- Speed: Primary > Secondary
- Cost: Primary > Secondary
- Volatility: Primary > Secondary
- Volume: Primary < Secondary

# 프로그램의 실행
- 메모리에 적재 방식
- 당장 필요한 메모리만 물리 메모리에 올림, 필요 없는 부분은 **Swap Area**라고 부르는 디스크에 적재
- Virtual Memory: 각 프로세스 별로 가지고 있는 독자적인 메모리 공간
	- 프로그램이 종료되면 없어짐

# 커널 주소 공간의 내용
## Code
- 커널 코드
- 시스템콜, 인터럽트 처리 코드
- 자원 관리를 위한 코드
- 편리한 서비스 제공을 위한 코드
## Data
- 운영체제가 사용하는 여러 자료구조
- PCB(Process Control Blcok): 프로세스를 관리하기 위한 자료구조

## Stack
- Process의 커널 스택
- 사용자 프로그램마다 커널 스택을 따로 사용

# 사용자 프로그램이 사용하는 함수
- 사용자 정의 함수
- 라이브러리 함수
- 커널 함수 → Kernel Address Space
	- 운영체제 프로그램의 함수
	- 커널 함수 호출 = 시스템 콜
