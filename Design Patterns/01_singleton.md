## 更新时间:2020/10/21
## 1 单例模式

单例模式：整个系统只需要拥有一个的全局对象，如服务器配置信息，数据库连接信息等

### 1.1 饿汉模式

```java
package singleton;


/**
 * 可用，推荐使用
 * 饿汉模式
 * 类加载到内存后，就实例化一个单例，JVM保证线程安全
 * 简单使用，推荐使用
 * 唯一缺点，不管用到与否，类装载时就完成实例化
 * 话说你不用，你装载它干嘛？
 * 
 * @author hahahaWhat
 *
 */

public class HungryMan01 {
	private static final HungryMan01 INSTANCE = new HungryMan01();

	private HungryMan01() {}

	public static HungryMan01 getInstance() {
		return INSTANCE;
	}

	public static void main(String[] args) {
		HungryMan01 lazyMan = HungryMan01.getInstance();
		HungryMan01 lazyMan2 = HungryMan01.getInstance();
		System.out.println(lazyMan == lazyMan2);
	}
}

```

### 1.2 懒汉模式1

```java
package singleton;

/**
 * 不可用
 * 懒汉模式
 * 也称为懒加载
 * 虽然达到了按需初始化的目的，但却带来线程不安全的问题，不可用
 * 
 * @author hahahaWhy
 *
 */

public class LazyMan02 {
	private static LazyMan02 INSTANCE;

	private LazyMan02() {}

	public static LazyMan02 getInstance() {
		if (INSTANCE == null) {
			// 让线程睡1ms，给其他线程打断它的机会，测试用
			try {
				Thread.sleep(1);
			} catch (InterruptedException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
			INSTANCE = new LazyMan02();
		}
		return INSTANCE;
	}

	public static void main(String[] args) {
		for (int i = 0; i < 100; i++) {
			new Thread(() -> {
				// 同一个类的不同对象哈希码不同
				System.out.println(LazyMan02.getInstance().hashCode());
			}).start();
		}
	}

}

```

### 1.3 懒汉模式2

```java
package singleton;

/**
 * 可用，不推荐使用
 * 懒加载
 * 可以通过加synchronized来解决，但也带来效率下降问题
 * @author hahahaWhy
 *
 */

public class LazyManSynchronized03 {
	private static LazyManSynchronized03 INSTANCE;

	private LazyManSynchronized03() {}

	// 线程每次运行到这里有没有锁，然后我有没有申请到这把锁，然后才能进行操作，效率降低
	public static synchronized LazyManSynchronized03 getInstance() {
		if (INSTANCE == null) {
			// 让线程睡1ms，给其他线程打断它的机会，测试用
			try {
				Thread.sleep(1);
			} catch (InterruptedException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
			INSTANCE = new LazyManSynchronized03();
		}
		return INSTANCE;
	}

	public static void main(String[] args) {
		for (int i = 0; i < 100; i++) {
			new Thread(() -> {
				// 同一个类的不同对象哈希码不同
				System.out.println(LazyManSynchronized03.getInstance().hashCode());
			}).start();
		}
	}
}

```

### 1.4 懒汉模式 3

```java
package singleton;

/**
 * 不可用 
 * 懒汉模式 
 * 也称为懒加载
 * 可以通过加synchronized来解决，但也带来效率下降问题
 * 将synchronized加到需要加锁的代码上，其他代码不加锁，带来效率的提高，但是带来线程安全问题
 * 
 * @author hahahaWhy
 *
 */

public class LazyManSynchronized04 {
	private static LazyManSynchronized04 INSTANCE;

	private LazyManSynchronized04() {
	};

	public static LazyManSynchronized04 getInstance() {
		if (INSTANCE == null) {
			//妄图通过减小代码同步代码块的方式提高效率，然而不可行
			synchronized (LazyManSynchronized04.class) {
				// 让线程睡1ms，给其他线程打断它的机会，测试用
				try {
					Thread.sleep(1);
				} catch (InterruptedException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}
				INSTANCE = new LazyManSynchronized04();
			}
		}

		return INSTANCE;
	}

	public static void main(String[] args) {
		for (int i = 0; i < 100; i++) {
			new Thread(() -> {
				// 同一个类的不同对象哈希码不同
				System.out.println(LazyManSynchronized04.getInstance().hashCode());
			}).start();
		}
	}
}

```

### 1.5 懒汉模式4，双重锁模式

```java
package singleton;

/**
 * 可用，完美写法，追求完美可用
 * 双重锁，double check lock
 * 懒汉模式 
 * 也称为懒加载
 * 可以通过加synchronized来解决，但也带来效率下降问题
 * 将synchronized加到需要加锁的代码上，其他代码不加锁，带来效率的提高，但是带来线程安全问题
 * 通过双重判断，将线程不安全问题解决
 * 
 * @author hahahaWhy
 *
 */

public class LazyManSynchronized05 {
//	volatile是避免cpu指令重排导致的错误
//  类的创建大概分三步，申请内存空间并初始化->成员变量赋值->建立关联，若发生指令重排，
//	比如后面两步颠倒，另一个线程刚好过来，则得到一个半初始化对象
	private static volatile LazyManSynchronized05 INSTANCE;

	private LazyManSynchronized05() {}

	public static LazyManSynchronized05 getInstance() {
		//这个判断是提高效率用的，避免所有线程去抢锁浪费时间
		if (INSTANCE == null) {
			//双重检测
			synchronized (LazyManSynchronized05.class) {
				if (INSTANCE == null) {
					// 让线程睡1ms，给其他线程打断它的机会，测试用
					try {
						Thread.sleep(1);
					} catch (InterruptedException e) {
						// TODO Auto-generated catch block
						e.printStackTrace();
					}
					INSTANCE = new LazyManSynchronized05();
				}
				
			}
		}

		return INSTANCE;
	}

	public static void main(String[] args) {
		for (int i = 0; i < 100; i++) {
			new Thread(() -> {
				// 同一个类的不同对象哈希码不同
				System.out.println(LazyManSynchronized05.getInstance().hashCode());
			}).start();
		}
	}
}

```

### 1.6 静态内部类模式

```java
package singleton;

/**
 * 可用，完美写法
 * 静态内部类
 * jvm保证单例，保证线程安全
 * 加载外部类时不会加载内部类，这样可以实现懒加载
 * @author hahahaWhy
 *
 */

public class StaticInner06 {
	private StaticInner06() {
	}
	//静态内部类保证了不用这个类的时候，不会初始化对象，用的时候才初始化，这就实现了懒加载
	private static class StaticInnerHolder{
		private final static StaticInner06 INSTANCE = new StaticInner06();
	}

	public static StaticInner06 getINSTANCE() {
		return StaticInnerHolder.INSTANCE;
	}

	public static void main(String[] args) {
		for (int i = 0; i < 100; i++) {
			new Thread(() -> {
				// 同一个类的不同对象哈希码不同
				System.out.println(LazyManSynchronized05.getInstance().hashCode());
			}).start();
		}
	}
	
}

```

### 1.7 枚举单例

```java
package singleton;

/**
 * 可用，完美写法
 * 枚举单例
 * 不金解决线程同步，还可以防止反序列化
 * 
 * @author hahahaWhy
 *
 */

public enum EnumSingleTon07 {
	INSTANCE;
	
	public static void main(String[] args) {
		for (int i = 0; i < 100; i++) {
			new Thread(()->{
				System.out.println(EnumSingleTon07.INSTANCE.hashCode());
			}).start();
		}
	}
	

}

```

