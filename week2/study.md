## 질문1

#### **스택 크기 고정안되어있어요?**

* 런타임에 할당
* RAM자체에서 할당할 수 있는 스택 메모리의 크기는 정해져 있긴 하다. ex) 3MB 
* 그 안에서 동적으로 늘어난다



## 질문2

#### cpu 바운드 프로세스는 대기큐에 안들어가나요?

**코어 할당을 기다릴 때는 준비큐. IO작업을 기다릴 때는 대기큐. IO작업이 없는 프로세스는 절대 대기큐에 안들어가나?**

* IO작업 말고도, 자식 프로세스의 종료를 기다릴 때도 대기큐에 들어가기도 한다. 즉, CPU 바운드 프로세스도 대기큐에 들어갈 수 있다.



## 질문3

#### 프로세스에서 포크 언제 쓸까? -> 왜 새 프로세스 생성함?

* 같은 코드, 데이터를 공유하고 싶을 때
* 동일한 내용의 프로그램을 동시에 실행하고 싶을 때 : 가상 메모리 & copy on write [참고](https://wslog.dev/fork-exec)
  * 자식은 부모의 주소 공간을 그대로 참조하다가, 자식이 write를 해야하는 시점에서만 복사



## 질문4

#### 프로세스를 모듈성 협력했을 때의 이득?

**IPC에서 정보를 공유하면 병렬로 실행할 수 있어서 좋다. 모듈성으로 해서 좋다. 모듈성이 무슨 의미?**

* 시스템 기능을 별도의 프로세스로 나눈 게 아니라, 프로세스 묶음 하나로 만드는 것. 
* 묶여진 프로세스들끼리는 데이터를 공유하는 것이 이득이다. 
* 저기서 의미하는 모듈은 프로세스의 묶음이고, 모듈안에서는 프로세스들끼리 협력할 수 있게하자



## 질문5

#### 소켓은 바이트 스트림만 통신하는 저수준 api인데, http 프로토콜을 어떻게 지원하는걸까요? http는 메시지 규격이 정해진 프로토콜 아닌가?

* http가 연결될 때는 소켓을 사용 (80포트 : wellknown 포트 중 하나. 보통은 1024이상의 포트를 사용한다.)
* 클라이언트가 80포트로 요청 보내면 연결
* http가 소켓 통신은 아니지만, 소켓을 사용하는 건 맞다
  * http는 인터넷 소켓을 사용하지만, 소켓이 http인 것은 아니다. http가 소켓 위에 있는 개념



## 질문6

#### RPC에서 나오는 스텁이 뭐지?

* 스텁은 클라이언트와 서버 간의 통신을 추상화한 것.

* 클라이언트 스텁 / RPC 시스템 demon(요청을 받는 스텁) / RPC 서버 스텁
  * RPC 시스템 demon(요청을 받는 스텁) : 적절한 RPC 스텁 호출, 매개변수 가공 작업



## 질문7

#### 그럼 forkjoinpool의 크기도 동적으로 결정되겠죠? 정적인건 아예 없겠지?

* java의 경우 최대 사이즈는 30000여개이고, 디폴트로 `logical cpu 크기 - 1개` 만큼 할당이 된다.

* [자바 문서](https://java-8-tips.readthedocs.io/en/stable/forkjoin.html#how-fork-join-works)를 보면, 필요한 만큼 스레드가 계속 늘어나는 것 같으나, 테스트 해봤을 때 `cpu크기 - 1개` 에서 더 늘어나진 않았다.

  

  ```java
  // 테스트 코드
  
      @Test
      void forkjoin() throws InterruptedException {
          final ForkJoinPool forkJoinPool = ForkJoinPool.commonPool();
          for (int i = 0; i < 50; i++) {
              final int number = i;
              forkJoinPool.execute(() -> {
                  System.out.println("size = " + forkJoinPool.getPoolSize());
              });
          }
          Thread.sleep(2000);
      }
  ```

  



## 질문8

#### 그럼 스레드 로컬은 스레드 메모리의 어느 영역에 할당이 될까요?

* `TLS`라는 메모리 영역에 저장
* https://learn.microsoft.com/ko-kr/windows/win32/procthread/thread-local-storage
