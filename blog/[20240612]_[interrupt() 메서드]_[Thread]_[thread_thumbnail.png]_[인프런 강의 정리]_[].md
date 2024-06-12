# interrupt

* Interrupt의 사전적 의미는 '방해하다'라는 뜻으로 어떤 주체의 행동이나 실행흐름을 방해한다는 의미로 해석할수 있습니다.
* 자바 스레드에서 interrupt()는 특정한 스레드에게 인터럽트 신호를 알려 줌으로써 스레드의 실행을 중단하거나, 작업취소, 강제 종료 등으로 사용할수 있습니다.

</br>

## interrupt()
* interrupt()는 스레드에게 인터럽트가 발생했다는 신호를 보내는 메커니즘입니다.
* interrupt()는 스레드가 현재 실행 흐름을 멈추고 인터럽트 이벤트를 먼저 처리하도록 시그널을 보내는 장치라 할수 있습니다.

**interrupted 속성**

* 스레드는 인터럽트 상태(Interrupt State)로 알려진 interrupted를 가지고 있으며 인터럽트 발생 여부를 확인 할 수 있는 상태값입니다. 기본값은 false.
* 인터럽트된 스레드가 처리해야 하는 특별한 규칙이나 정해진 기준은 없으나 일반저으로 인터럽트 상태를 사용해서 스레드를 중지하거나, 작업을 취소하거나, 스레드를 종료하는 등의 기능을 구현할수 있습니다.
* 한 스레드가 다른 스레드를 인터럽트 할수 있고 자기 자신을 인터럽트 할수 있습니다.
* interrupt()는 횟수 제한이 없으며, 인터럽트 할때마다 스레드의 인터럽트 상태를 true로 변경합니다.



</br>

## 인터럽트 상태 확인
**static boolean interrupted()**

* 스레드의 인터럽트 상태를 반환하는 정적 메소드입니다.
* 만약 현재 인터럽트 상태가 true인 경우, true를 반환하고 인터럽트 상태를 다시 false로 초기화하여 인터럽트를 해제하는 역활을 합니다.
* 인터럽트를 해제하는 경우 다른곳에서 스레드에 대한 인터럽트 상태를 체크하는곳이 있다면 별도의 처리가 필요할수 있습니다.
* 인터럽트를 강제로 해제했기 때문에 다시 인터럽트를 걸어서 인터럽트의 상태를 유지할수 있습니다.

**boolean isInterrupted()**

* 스레드의 인터럽트 상태를 반환하는 인스턴스 메서드입니다.
* 이 메서드는 스레드의 인터럽트 상태를 변경하지 않고 상태값만 반환합니다.
* 인터럽트 상태를 확인하는 용도로만 사용할 경우 interrupt()보다 이 메소드를 사용하는 것이 더 좋습니다.


#### Code
스레드2에서 스레드1에 인터럽트를 거는 경우
```java
Thread thread1 = new Thread(() -> {
            System.out.println("스레드1 작업 시작");
            System.out.println("스레드1 인터럽트 상태: " + Thread.currentThread().isInterrupted());
        });

        Thread thread2 = new Thread(() -> {
            System.out.println("스레드2 작업 시작");
            thread1.interrupt(); //스레드2가 스레드1을 인터럽트
            System.out.println("스레드2 인터럽트 상태: " + Thread.currentThread().isInterrupted());
        });

        thread2.start();
        Thread.sleep(1000);
        thread1.start();

        thread1.join();
        thread2.join();

        System.out.println("모든 스레드 작업 종료");

        //스레드2 작업 시작
        //스레드2 인터럽트 상태: false
        //스레드1 작업 시작
        //스레드1 인터럽트 상태: true
        //모든 스레드 작업 종료
```

</br>

인터럽트 걸고 초기화된 인터럽트를 원복하는 경우
```java
hread thread = new Thread(() -> {
            while (true) {
                System.out.println("스레드 작동 중..");
                if(Thread.interrupted()) {
                    //interrupt() 호출시 인터럽트는 true를 반환하고 다시 false로 초기화
                    System.out.println("인터럽트 상태가 초기화");
                    break;
                }
            }
            System.out.println("인터럽트 상태: " + Thread.currentThread().isInterrupted());
            Thread.currentThread().interrupt(); // 다시 true로 변경
            System.out.println("인터럽트 상태: " + Thread.currentThread().isInterrupted());
        });

        thread.start();

        try {
            Thread.sleep(500);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }

        thread.interrupt();

        //스레드 작동 중..
        //스레드 작동 중..
        //스레드 작동 중..
        //인터럽트 상태가 초기화
        //인터럽트 상태: false
        //인터럽트 상태: true

```


</br>

## InterruptedException

* InterruptedException은 interrput() 메커니즘의 일부이며 대기나 차단 등 블록킹 상태에 있거나 블록킹 상태를 만나는 시점의 스레드에 인터럽트 할때 발생하는 예외입니다.
* InterruptedException 이 발생 하면 인터럽트 상태는 자동으로 초기화됩니다. 즉 Thread.interrupted() 한것과 같은 상태로 됩니다. (interrupted == false)
* 다른 곳에서 인터럽트 상태를 참조하고 있다면 예외 구문에서 대상 스레드에 다시 interrupt() 해야 할수 있습니다.

 **InterruptedException 발생할 경우**

 * Thread.sleep(), Thread.join(), Object.wait()
 * Future.get(), BlockingQueue.take()

</br>

#### Code





InterruptedException 발생 시
```java
Thread thread = new Thread(() -> {
            boolean currentThread = Thread.currentThread().isInterrupted();
            System.out.println("처음 인터럽트 상태: " + currentThread);
            
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                //예외 발생하면 인터럽트가 초기화, false
                System.out.println("예외 발생 후, 인터럽트 상태: " + currentThread);
                Thread.currentThread().interrupt();
                throw new RuntimeException(e);
            }
        });

        thread.start();

        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace(System.out);
        }

        thread.interrupt();
        thread.join();

        System.out.println("원복, 인터럽트 상태: " + thread.isInterrupted());

        //처음 인터럽트 상태: false
        //예외 발생 후, 인터럽트 상태: false  
        //원복, 인터럽트 상태: true
        //Exception in thread "Thread-0" java.lang.RuntimeException: java.lang.InterruptedException: sleep interrupted ..
```