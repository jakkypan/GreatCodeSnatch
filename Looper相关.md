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
