# sleep

* sleep() 메서드는 지정된 시간동안 현재 스레드의 실행을 일시 정지하고 대기상태로 빠졌다가 시간이 지나면 실행대기 상태로 전환됩니다.
* 네이티브 메서드로 연결되며 시스템 콜을 통해 커널모드에서 수행 후 유저모드로 전환합니다.

## API 및 예외
- static sleep(long millis) throws InterruptedException
  - 지정한 밀리초 시간 동안 스레드를 수면 상태로 만듭니다.
  - 밀리초에 대한 인수 값은 음수가 될수 없으며 음수 일 경우 IllegalArgumentException이 발생합니다.
- static sleep(long millis, int nanos) throws InterruptedException
  - 지정한 밀리초에 나노초를 더한 시간 동안 스레드를 수면 상태로 만듭니다.
  - 나노초의 범위는 0에서 999999 입니다.
- InterruptedException
  - 스레드가 수면 중에 인터럽트 될 경우 InterruptedException 예외를 발생 합니다.
  - 다른 스레드는 잠자고 있는 스레드에게 인터럽트, 즉 중단(멈춤) 신호를 보낼수 있습니다.
  - InterruptedException 예외가 발생하면 스레드는 수면상태에서 깨어나고 실행 대기 상태로 전환되어 실행상태를 기다립니다.

  interrupt의 뜻 : 방해하다, 중단하다


## Sleep() 작동방식

#### 지정된 시간이 지남
![sleep](./img/thread/sleep.png)
#### interrupt() 발생시
![interrupt](./img/thread/interrupt.png)


### sleep(0)과 sleep(n)의 의미
#### sleep(0)
- 스레드가 커널모드로 전환 후 스케줄러는 현재 스레드와 동일한 우선순위(Priority)의 스레드가 있을 경우 실행대기상태 스레드에게 CPU를 할당함으로 컨텍스트 스위칭이 발생합니다.
- 만약 우선순위가 동일한 실행대기 상태의 다른 스레드가 없으면 스케줄러는 현재 스레드에게 계속 CPU를 할당해서 컨텍스트 스위칭이 없고 모드 전환만 일어납니다.

#### sleep(n)
- 스레드가 커널모드로 전환 후 스케줄러는 조건에 상관없이 현재 스레드를 대기상태에 두고 다른 스레드에게 CPU를 할당함으로 모든 전환과 함께 컨텍스트 스위칭이 발생합니다.
#### 정리
- sleep(millis) 메서드는 네이티브 메서드이기 때문에 sleep(millis)을 실행하게 되면 시스템 콜을 호출하게 되어 유저모드에서 커널모드로 전환합니다.
- 다른 스레드에게 명확하게 양보하기 위함이라면 sleep(0) 보다는 sleep(1)을 사용하기를 권장합니다.

- - -

### code
기본 코드
```java
try {
    System.out.println("2초 후에 메시지가 출력");
    Thread.sleep(2000);
    System.out.println("출력");
} catch (InterruptedException e) {
    throw new RuntimeException(e);
}

//2초 후에 메시지가 출력
//출력
```

실행 중 interrupt 호출시
```java
Thread sleepingThread = new Thread(() -> {
    try {
        System.out.println("10초 동안 sleep");
        Thread.sleep(10000);
        System.out.println("스레드가 깨어 났습니다.");
    } catch (InterruptedException e) {
        System.out.println("인터럽트 발생!");
        e.printStackTrace(System.out);
    }
});
sleepingThread.start();
Thread.sleep(1000);
sleepingThread.interrupt();

//10초 동안 sleep
//인터럽트 발생!
//java.lang.InterruptedException: sleep interrupted ...
```

### sleep() 작동 방식 정리
> - sleep()이 되면 OS 스케줄러는 현재 스레드를 지정된 시간동안 대기 상태로 전환하고 다른 스레드 혹은 프로세스에게 CPU를 사용하도록 합니다.
> - 대기 시간이 끝나면 스레드 상태는 바로 실행상태가 아닌 실행 대기 상태로 전환하고 CPU가 실행을 재개할 때까지 기다립니다.
> - 실행상태가 되면 스레드는 남은 지점부터 실행을 다시 시작합니다.
> - 동기화 메서드 영역에서 수면 중인 스레드는 획득한 모니터나 락을 잃지 않고 계속 유지합니다.
> - sleep() 중인 스레드에게 인터럽트가 발생할 경우 현재 스레드는 대기에서 해제되고 실행상태로 전환되어 예외를 처리하게 됩니다.
> - 스레드의 수면 시간은 OS 스케줄러 및 시스템 기능에 따라 제한되기 때문에 정확성이 보장되지 않으며 시스템의 부하가 많고 적음에 따라 지정한 수면 시간과 차이가 날수 있습니다. (예 : sleep(3000) -> 3001밀리초, 2999밀리초)