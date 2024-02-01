```java
Thread thread = new Thread(new Runnable() {

			@Override
			public void run() {
				// TODO Auto-generated method stub
				System.out.println("We are now in thread " + Thread.currentThread().getName()); //eclipse랑 다르게 thread에만 breakpoint 잡은 거 아니면 thread 안의 breakpoint가 잡히지 않는다.
				System.out.println("We are now in thread " + Thread.currentThread().getPriority());

			}
		});
		
		thread.setName("New Worker Thread"); //debugging할 때 나중에 의미 있는 이름이 필요함.
		thread.setPriority(Thread.MAX_PRIORITY); //OS에 thread의 동적 우선순위를 알려줌. 10이고. 제일 우선순위 떨어지는 것.
		System.out.println("we ard in thread: " + Thread.currentThread().getName() + " before starting a new thread");
		thread.start();
		System.out.println("we ard in thread: " + Thread.currentThread().getName() + " after starting a new thread");

		//Thread.sleep(10_000);// OS에 지시하는 것. 이 시간 동안 CPU르 쓰지 않음.
		/*
		 * we ard in thread: main before starting a new thread 
		 * we ard in thread: main after starting a new thread 
		 * We are now in thread Thread-0
		 */
		//start하는 데 시간이 걸려서 main after가 먼저 나옴.
		
		
		Thread thread1 = new Thread(new Runnable() {

			@Override
			public void run() {
				throw new RuntimeException("Internal Exception");
			}
		});
		
		thread1.setName("misbehaving thread");
		thread1.setUncaughtExceptionHandler(new UncaughtExceptionHandler() {
			@Override
			public void uncaughtException(Thread t, Throwable e) {
				System.out.println("A critical Error happened in thread" + t.getName() + " the error is " + e.getMessage());				
			}
		});
		thread1.start();
```