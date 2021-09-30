# looper和handler的同步问题

```java
import android.os.Handler;
import android.os.Looper;
import android.os.Message;
import android.util.Log;

/**
 * looper和handler的同步问题
 *
 * 测试代码：
 * 1、t_1线程中创建t_2线程
 * 2、t_2线程转换成looper线程，通过looper线程来处理消息
 * 3、在t_1线程中得到t_2线程的looper，并且根据这个looper来创建handler，向t_2线程发送消息
 *
 * 问题：
 * > 两个线程没有同步机制，looperThread.myLooper很大概率拿到的就是null
 *
 * 解决方案：
 * > 自己解决：wait/notify
 * > 系统方案：HandlerThread
 *
 * @author panda
 * created at 2021/9/30 4:16 下午
 */
public class Han {
	public static class LooperThread extends Thread {
		private Looper myLooper = null;

		public LooperThread() {
			super("t_2");
		}

		@Override
		public void run() {
			Looper.prepare();
			synchronized (this) {
				myLooper = Looper.myLooper();
				notifyAll();
			}
			Looper.loop();
		}

		public Looper getMyLooper() {
			if (!isAlive()) {
				return null;
			}

			synchronized (this) {
				while (isAlive() && myLooper == null) {
					try {
						wait();
					} catch (InterruptedException e) {}
				}
			}
			return myLooper;
		}
	}

	public void test() {
		new Thread(() -> {
			LooperThread looperThread = new LooperThread();
			looperThread.start();
			Looper looper = looperThread.getMyLooper();
			Handler handler = new Handler(looper, msg -> {
				Log.e("1111", "handleMessage....");
				return false;
			});
			handler.sendMessage(Message.obtain());
		}, "t_1").start();
	}
}

```

# ThreadLocal的使用场景

简单的使用：

```java
public class Han {
	ThreadLocal<String> tl;
	ThreadLocal<String> tl2;

	public void test() {
		tl2 = new ThreadLocal<>();
		tl2.set("tl...");
		Log.e("1111", "tl2: " + tl2.get());

		new Thread(() -> {
			tl = new ThreadLocal<>();
			tl.set("tl...");
			Log.e("1111", "tl: " + tl.get() + ", tl2: " + tl2.get());
		}).start();
	}
}

output：
2021-09-30 20:48:45.034 32640-32640/com.panda.myapplication E/1111: tl2: tl...
2021-09-30 20:48:45.037 32640-32670/com.panda.myapplication E/1111: tl: tl..., tl2: null
```

ThreadLocal hash冲突与内存泄漏问题：

[冲突与内存泄漏问题](https://blog.csdn.net/Summer_And_Opencv/article/details/104632272)

使用场景：[ThreadLocal三种使用场景](https://cloud.tencent.com/developer/article/1636025)

* 线程中处理一个非常复杂的业务，可能方法有很多，那么，使用 ThreadLocal 可以代替一些参数的显式传递；
* 线程内上线文管理器、数据库连接等可以用到 ThreadLocal;
* 在一些多线程的情况下，如果用线程同步的方式，当并发比较高的时候会影响性能，可以改为 ThreadLocal 的方式，例如高性能序列化框架 Kyro 就要用 ThreadLocal 来保证高性能和线程安全；

使用注意事项：

* 使用 ThreadLocal 的时候，最好要声明为静态的；
* 使用完 ThreadLocal ，最好手动调用 remove() 方法，例如上面说到的 Session 的例子，如果不在拦截器或过滤器中处理，不仅可能出现内存泄漏问题，而且会影响业务逻辑；

