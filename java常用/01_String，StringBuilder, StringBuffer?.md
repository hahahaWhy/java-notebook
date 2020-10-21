## 更新时间：2020/10/2
## java字符串操作选择String，StringBuilder, StringBuffer?

### 看如何操作：

常量的声明，少量的变量运算，选String

频繁的字符串运算，如拼接，修改，删除等，选StringBuilder或StringBuffer

### StringBuilder和StringBuffer又如何选？

涉及线程安全，选StringBuffer

不涉及线程安全，选StringBuilder

(StringBuilder运行效率高，一般很少遇到多线程去操作字符串，所以一般都是用StringBuilder)

## 为什么这样选?

### String

String 类是不可改变的，所以你一旦创建了 String 对象，那它的值就无法改变了

当两个字符串拼接时，会产生一个新的字符串去接收拼接的结果，但前两个字符串并没有销毁，当拼接操作变多以后，产生大量的String 临时类，消耗内存，增加对垃圾回收占用CPU的时间

### StingBuilder和StringBuffer

StringBuffer 和 StringBuilder 类的对象能够被多次的修改，并且不产生新的未使用对象。



