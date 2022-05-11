# 我整理的JAVA
## 基本类型和包装类型的区别？
+ 成员变量包装类型不赋值就是 null ，而基本类型有默认值且不是 null。
+ 包装类型可用于泛型，而基本类型不可以。
+ 基本数据类型的局部变量存放在 Java 虚拟机栈中的局部变量表中，基本数据类型的成员变量（未被 static 修饰 ）存放在 Java 虚拟机的堆中。包装类型属于对象类型，我们知道几乎所有对象实例都存在于堆中。
+ 相比于对象类型， 基本数据类型占用的空间非常小。

## 包装类型的缓存机制了解吗？
java基本数据类型的包装类型的大部分都用到了缓存机制来提升性能。但是不是所有。  

Byte,Short,Integer,Long 这 4 种包装类默认创建了数值 [-128，127] 的相应类型的缓存数据，Character 创建了数值在 [0,127] 范围的缓存数据，Boolean 直接返回 True or False。  
如果超出对应范围仍然会去创建新的对象，缓存的范围区间的大小只是在性能和资源之间权衡。    
两种浮点数类型的包装类 Float，Double 并没有实现缓存机制。
```java
Integer i1 = 33;
Integer i2 = 33;
System.out.println(i1 == i2);// 输出 true

Float i11 = 333f;
Float i22 = 333f;
System.out.println(i11 == i22);// 输出 false

Double i3 = 1.2;
Double i4 = 1.2;
System.out.println(i3 == i4);// 输出 false
```
再看一下下面的结果： 
```java
Integer i1 = 40;
Integer i2 = new Integer(40);
System.out.println(i1==i2);
```
