# Java bigdecimal class的四种陷阱

[原文](https://blogs.oracle.com/javamagazine/four-common-pitfalls-of-the-bigdecimal-class-and-how-to-avoid-them)

##  Pitfall #1: double类型构造方法

考虑下面的代码，

```java
BigDecimal x = new BigDecimal(0.1);
System.out.println("x=" + x);
//=>x=0.1000000000000000055511151231257827021181583404541015625
```

传入一个double类型值去创建一个BigDecimal x,然后打印该值会发现并不是所期待的那样，这是因为在计算机硬件中，浮点数是以2(二进制)为基数的分数表示的。然而，大多数十进制分数不能精确地表示为二进制分数。因此，实际存储在机器中的二进制浮点数仅接近您输入的十进制浮点数。因此，传递给双精度构造函数的值并不完全等于0.1。

而使用字符串构造函数则会符合你的预期：

```java
BigDecimal y = new BigDecimal("0.1");
System.out.println("y=" + y);
//=>y=0.1
```

如果不得不使用Double类型的值来构造BigDecimal怎么办？BigDecimal提供了valueof方法来转换，效果与`BigDecimal(String)`一致，其实valueof(Double)源码里面是调用`BigDecimal(Double.toString(val))`来完成的。

```java
BigDecimal y = BigDecimal.valueOf(0.1);
System.out.println("y=" + y);
//=>y=0.1
```



## Pitfall #2: 静态valueOf(double)方法

如果你是使用BigDecimal.valueOf(double)方法来创建BigDecimal，**注意在double精度上有限制** 考虑下面的代码片段，

```java
BigDecimal x = BigDecimal.valueOf(1.01234567890123456789);
BigDecimal y = new BigDecimal("1.01234567890123456789");
System.out.println("x=" + x);
System.out.println("y=" + y);
//=> x=1.0123456789012346
//=> y=1.01234567890123456789
```

这里，x值丢失了四个十进制数字，因为double的精度是15-17位（float的精度只有6-9位），而BigDecimal的是任意精度（仅受内存的限制）

因此，使用字符串构造函数实际上是个好主意，因为有效地避免了两个double类型引起问题。



## Pitfall #3: equals(bigDecimal) 方法

考虑下面代码片段，

```java
BigDecimal x = new BigDecimal("1");
BigDecimal y = new BigDecimal("1.0");
System.out.println(x.equals(y));
//=>false

BigDecimal x = new BigDecimal("1.00");
BigDecimal y = new BigDecimal("1.0");
System.out.println(x.equals(y));
//=>false
```

通过肉眼看起来应该是true，



```java
BigDecimal x = new BigDecimal("1");
BigDecimal y = new BigDecimal("1.0");
System.out.println(x.compareTo(y) == 0);
//=>true

BigDecimal x = new BigDecimal("1.00");
BigDecimal y = new BigDecimal("1.0");
System.out.println(x.compareTo(y) == 0);
//=>true
```



##  Pitfall #4: round(mathContext)方法

一些开发人员会尝试使用`round(new MathContext(precision, roundingMode))`方法来对BigDecimal进行四舍五入到两位有效数字，但这不是一个好的方法，考虑下面代码：

```java
BigDecimal x = new BigDecimal("12345.6789");
x = x.round(new MathContext(2, RoundingMode.HALF_UP));
System.out.println("x=" + x.toPlainString());
System.out.println("scale=" + x.scale());
//=>x=12000
//=>scale=-3
```

通过打印发现，x不是所期待的12345.68，scale也不是2.

该方法并没有将小数部分四舍五入，而是将值（ unscaled value ）四舍五入到给定的有效数字数(从左到右计算)，和小数点保持不变， 而且scale 的值是-3。

所以发生了什么？

请看下面的输出，

```java
BigDecimal x = new BigDecimal("12345.6789");
System.out.println("x.unscaledValue="+x.unscaledValue());
//=> x.unscaledValue=123456789
x = x.round(new MathContext(2, RoundingMode.HALF_UP));
System.out.println("x ="+x);
//=> x =1.2E+4
System.out.println("x.toPlainString=" + x.toPlainString());
//=> x.toPlainString=12000
System.out.println("x.scale=" + x.scale());
//=> x.scale=-3
System.out.println("x.unscaledValue="+x.unscaledValue());
//=> x.unscaledValue=12
```

其`unscaledValue`转换成为有效位数为2（123456789 =》 12），由于小数点是没有左移， BigDecimal真正的值应该是 12000.0000 。之所以变成12000是因为小数点后面四个0是没有意义的。

为什么12000的scale是-3而不是0呢？

BigDecimal的值等于 unscaledValue ^ 10-scale。在上面的代码中可以发现，在执行round之后，`unscaledValue`的值变成12，而12*10^3  == 12000。

此外，在上面代码中直接打印x是科学计数法。

 要获得预期的12345.68结果， 使用`setScale(scale, roundingMode)` `setScale(scale, roundingMode)` method, for example: 

```java
BigDecimal x = new BigDecimal("12345.6789");
x = x.setScale(2, RoundingMode.HALF_UP);
System.out.println("x=" + x));
//=> x=12345.68
```

 setScale(scale, roundingMode)方法根据指定的舍入模式将分数部分舍入小数点后两位。 





