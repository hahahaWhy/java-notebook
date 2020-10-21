## 更新时间：2020/10/21
## 2 策略模式

 * 设计理念：完成一项任务，往往可以有多种不同的方式，每一种方式称为一个策略，我们可以根据环境或者条件的不同选择不同的策略来完成该项任务。
 * 实现方式：实现同一个接口，使用这个策略的函数只需调用实现这个接口的对象，就可以调用相应的策略
 * 也可以用继承去实现
 * 和多态的概念很像

### 2.1 例子

- 比较对象属性值的大小，比如猫的身高，猫的体重

 * 由于我不知道以后我还要比较什么其他对象的属性值，比如我以后可能还会比较人的身高，体重，胡须长度等
 * 如果开始就写死了只能比较猫的身高，以后要比较猫的体重，只能去源码中修改
 * 如果将其他类型对象、其他属性的比较写成比较策略，就不需修改类本身的源码，添加比较策略即可
 * 例子中StrategyPattern通过调用不同的策略，实现了不懂对象不同属性的比较
 * 其中CatHeightComparator，CatWeightComparator，PersonHeightComparator这些对象，实现的是Comparator这个接口

![策略模式](https://github.com/hahahaWhy/java-notebook/blob/main/Design%20Patterns/img/%E7%AD%96%E7%95%A5%E6%A8%A1%E5%BC%8F.png)

#### Cat

```java
package strategy;

public class Cat{
	private float weight;
	private float height;
	
	public float getWeight() {
		return weight;
	}
	public void setWeight(float weight) {
		this.weight = weight;
	}
	public float getHeight() {
		return height;
	}
	public void setHeight(float height) {
		this.height = height;
	}
	public Cat(float w,float h) {
		this.weight=w;
		this.height=h;
	}
	@Override
	public String toString() {
		return "Cat [weight=" + weight + ", height=" + height + "]";
	}
}

```

#### CatHeightComparator

```java
package strategy;

import java.util.Comparator;

public class CatHeightComparator implements Comparator<Cat>{

	@Override
	public int compare(Cat o1, Cat o2) {
		if(o1.getHeight()>o2.getHeight())
			return 1;
		else if (o1.getHeight()<o2.getHeight()) {
			return -1;
		}else return 0;
	}

}

```

#### StrategyPattern

```java
package strategy;

import java.util.Comparator;

/**
 * main
 * hahahaWhy	
 */

public class StrategyPattern {

	public static void main(String[] args) {
		//比较猫的weight
		CatWeightComparator catWeightComparator = new CatWeightComparator();
		int result=catWeightComparator.compare(new Cat(1, 2), new Cat(3, 4));
		System.out.println(result);
		
		//比较人的height
		PersonHeightComparator personHeightComparator = new PersonHeightComparator();
		int result1=personHeightComparator.compare(new Person(2.0f), new Person(1.9f));
		System.out.println(result1);
		
	}
}

```

