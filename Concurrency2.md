# Concurrency

## 二: 创建和启动线程

done like this:

```java
Thread thread = new Thread();
thread.start();
```

### 在线程中运行代码的两种方式

#### Thread子类

```java
public MyThread extends Thread{
        @Override
        public void run() {
            System.out.println("Hello, World!");
        }
}
```

#### Runable实现类

```java
public MyRunnable implements Runnable{
        public void run() {
            System.out.println("all so have to say hello, world!");
        }
}
```

##### 匿名内部类

```java
Runnable myRunnable = new Runnable() {
	public void run() {
    	System.out.println("Hello, World!");
	}
};
```

##### Lambda

```java
Runnable myRunable = {
	System.out.println("Hello, World!");   
};
```

### 两种方式对比

通过实现接口的方式，能够让代码书写更加精简。可以通过Lambda来简化代码。实现接口的方式能够有效分离业务逻辑与线程运行代码，能让线程池有效管理和调度任务，在线程池繁忙时进入队列等待线程池调度。这种方式能够迎合工作者模型。

有时候可以结合两种方式来使用，最典型的代表就是实现一个线程池。

### 常见错误：用run()方法来代替start()执行

```java
Thread newThread = new Thread(MyRunnable());
newThread.run();  //should be start();
```

仅执行run方法，并不会创建新的线程，而是在本线程来执行run方法里面的业务逻辑。切记创建一个新的线程，一定要执行start方法而非run方法。

### 线程名称

往往为了区分哪个线程在运行，会通过System.out的方式来打印线程名称。通过继承Thread的方式可以直接通过Thread的getName方法来打印名称。通过实现的方式并没有getName方法，此时可以通过Thread的静态方法currentThread来取得当前线程引用。

在创建线程时，可以通过Thread的构造方法传递线程名称;

```java
Thread myThread = new Thread("our-thread0") {
	@Override
    public void run() {
    	System.out.println("run by: " + getName());
    }
};

Runnable myRunnable = () -> {
	final Thread currentThread = Thread.currentThread();
	System.out.println("run by: " + currentThread.getName());
};

myThread.start();
new Thread(myRunnable, "our-thread1").start();
```

### Java线程实例

```java
public class ThreadExample {

    private static void startThread(int i) {
        new Thread("thread: " + i) {
            @Override
            public void run() {
                System.out.println(getName() + " running.");
            }
        }.start();
    }

    public static void main(String[] args) {
        System.out.println(Thread.currentThread().getName());
        IntStream.range(0, 10)
                .forEach(ThreadExample::startThread);
    }
    
}
```

如你所见，以上实例中，顺序创建并执行10个线程，但线程的运行并不一定是顺序，此时10个线程是并行执行的，线程的执行顺序是由jvm和操作系统共同决定的。

### 暂停线程

```java
try {
	// 当前线程睡眠 10 s
    Thread.sleep(10L * 1000L);
} catch (InterruptedException e) {
	e.printStackTrace();
}
```

可以通过Thread.sleep()方法，通过传递毫秒数来暂停线程。暂停时长以传递毫秒数为准。

### 停止线程

Thread对象中默认包含了stop()，pause()等方法。但已经被废弃了，默认的stop()方法并不能保证线程在何种状态下被停止。这意味着，所有被线程访问到的java对象都将在一个未知的状态下运行。如果其他线程需要访问相同的对象，那么你的应用将会出现不可预测的失败。

```java
public class ThreadStopExample implements Runnable {

    // 是否停止
    private boolean doStop = false;

    // 继续运行线程
    public synchronized boolean keepRunning() {
        return !doStop;
    }

    // 停止线程
    public synchronized void stop() {
        this.doStop = true;
    }

    @Override
    public void run() {
        while (keepRunning()) {
            System.out.println("Running");
            try {
                // 暂停 3 s
                Thread.sleep(3L * 1000L);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args) {
        final ThreadStopExample threadStopExample = new ThreadStopExample();
        new Thread(threadStopExample, "our thread").start();
        try {
            // 10 s 后停止线程
            Thread.sleep(10L * 1000L);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        threadStopExample.stop();
    }

}
```

