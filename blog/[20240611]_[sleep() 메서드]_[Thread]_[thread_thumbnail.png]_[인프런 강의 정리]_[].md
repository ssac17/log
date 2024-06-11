# sleep

* sleep() 메서드는 지정된 시간동안 현재 스레드의 실행을 일시 정지하고 대기상태로 빠졌다가 시간이 지나면 실행대기 상태로 전환됩니다.
* 네이티브 메서드로 연결되며 시스템 콜을 통해 커널모드에서 수행 후 유저모드로 전환합니다.

## API 및 예외
* static sleep(long millis) throws InterruptedException
  * 지정한 밀리초 시간 동안 스레드를 수면 상태로 만듭니다.
  * 밀리초에 대한 인수 값은 음수가 될수 없으며 음수 일 경우 IllegalArgumentException이 발생합니다.
* static sleep(long millis, int nanos) throws InterruptedException
  * 지정한 밀리초에 나노초를 더한 시간 동안 스레드를 수면 상태로 만듭니다.
  * 나노초의 범위는 0에서 999999 입니다.
* InterruptedException
  * 스레드가 수면 중에 인터럽트 될 경우 InterruptedException 예외를 발생 합니다.
  * 다른 스레드는 잠자고 있는 스레드에게 인터럽트, 즉 중단(멈춤) 신호를 보낼수 있습니다.
  * InterruptedException 예외가 발생하면 스레드는 수면상태에서 깨어나고 실행 대기 상태로 전환되어 실행상태를 기다립니다.

## Sleep() 작동방식

#### 지정된 시간이 지남

![sleep](./img/thread/sleep.png)

interrupt() 발생시
