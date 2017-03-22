# Synchronized

-  可修饰一个对象 的方法或代码块 多线程访问一个对象的方法进行同步 锁住的是对象 多线程谁先拿到该对象的锁 谁先执行


```

class Sync {

	public synchronized void test() {
		System.out.println("test开始..");
		try {
			Thread.sleep(1000);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		System.out.println("test结束.."); 
	}
}

/*or
class Sync {

	public  void test() {
	
	synchronized(this){
		System.out.println("test开始..");
		try {
			Thread.sleep(1000);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		System.out.println("test结束.."); 
		}
	}
}

*/

class MyThread extends Thread {

	public void run() {
		Sync sync = new Sync();
		sync.test();
	}
}

public class Main {

	public static void main(String[] args) {
		for (int i = 0; i < 3; i++) {
			Thread thread = new MyThread();
			thread.start();
		}
	}
}
	//运行结果： test开始.. test开始.. test开始.. test结束.. test结束.. test结束..
	
```


```
class MyThread extends Thread {

	private Sync sync;

	public MyThread(Sync sync) {
		this.sync = sync;
	}

	public void run() {
		sync.test();
	}
}

public class Main {

	public static void main(String[] args) {
		Sync sync = new Sync();
		for (int i = 0; i < 3; i++) {
			Thread thread = new MyThread(sync);
			thread.start();
		}
	}
	//运行结果： test开始.. test结束.. test开始.. test结束.. test开始.. test结束..	


}

```



- 可修饰静态方法类或者所有对象的方法或代码块同步 多线程访问类的所有对象的方法或代码块进行同步



```
class Sync {

	public  synchronized static void test() {
		System.out.println("test开始..");
		try {
			Thread.sleep(1000);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		System.out.println("test结束..");
	}
}
/*or
class Sync {

	public void test() {
	
	synchronized(Sync.class){
		System.out.println("test开始..");
		try {
			Thread.sleep(1000);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		System.out.println("test结束.."); 
		}
	}
}

*/


```


- static synchronized方法也相当于全局锁，相当于锁住了代码段。

- 我们在用synchronized关键字的时候，能缩小代码段的范围就尽量缩小，能在代码段上加同步就不要再整个方法上加同步。这叫减小锁的粒度，使代码更大程度的并发。原因是基于以上的思想，锁的代码段太长了，别的线程是不是要等很久，等的花儿都谢了。实现同步是要很大的系统开销作为代价的，甚至可能造成死锁，所以尽量避免无谓的同步控制。

  ​



