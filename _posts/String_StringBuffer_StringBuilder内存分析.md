title: String，StringBuffer，StringBuilder内存分析
date: 2016-04-16 18:33:52
tags: Java
---


## 拼接字符串

```
public class StringText1 {
    final static int LOOP_TIMES = 200000;
    final static String CONSTANT_STRING = "坂田银时";

    public static void main(String[] args) {

        startup();

    }

    public static void testString() {
        String string = "";
        long beginTime = System.currentTimeMillis();
        for (int i = 0; i < LOOP_TIMES; i++) {
            string += CONSTANT_STRING;
        }
        long endTime = System.currentTimeMillis();
        System.out.print("String : " + (endTime - beginTime) + "\t");
    }

    public static void testStringBuffer() {
        StringBuffer buffer = new StringBuffer();
        long beginTime = System.currentTimeMillis();
        for (int i = 0; i < LOOP_TIMES; i++) {
            buffer.append(CONSTANT_STRING);
        }
        buffer.toString();
        long endTime = System.currentTimeMillis();
        System.out.print("StringBuffer : " + (endTime - beginTime) + "\t");
    }

    public static void testStringBuilder() {
        StringBuilder builder = new StringBuilder();
        long beginTime = System.currentTimeMillis();
        for (int i = 0; i < LOOP_TIMES; i++) {
            builder.append(CONSTANT_STRING);
        }
        builder.toString();
        long endTime = System.currentTimeMillis();
        System.out.print("StringBuilder : " + (endTime - beginTime) + "\t");
    }

    public static void startup() {
        for (int i = 0; i < 6; i++) {
            System.out.print("The " + i + " [\t    ");
            testString();
            testStringBuffer();
            testStringBuilder();
            System.out.println("]");
        }
    }
}

```

<!-- more -->


>测试结果：

```
The 0 [	    String : 154863	StringBuffer : 10	StringBuilder : 7	]
The 1 [	    String : 114700	StringBuffer : 3	StringBuilder : 3	]
The 2 [	    String : 102340	StringBuffer : 3	StringBuilder : 3	]
The 3 [	    String : 97736	StringBuffer : 3	StringBuilder : 3	]
The 4 [	    String : 109092	StringBuffer : 3	StringBuilder : 5	]
The 5 [	    String : 104015	StringBuffer : 3	StringBuilder : 4	]
```

### 结论

#### 1、
底层实际上是将循环体内的 string += CONSTANT_STRING; 语句转成了:
string = (new StringBuilder(String.valueOf(string))).append("min-snail").toString();
所以在二十万次的串联字符串中, 每一次都先去创建 StringBuilder 对象, 然后再调 append() 方法来完成 String 类的 "+" 操作。

这里的大部分时间都花在了对象的创建上, 而且每个创建出来的对象的生命都不能长久, 朝生夕灭, 因为这些对象创建出来之后没有引用变量来引用它们,

那么它们在使用完成时候就处于一种不可到达状态, java 虚拟机的垃圾回收器(GC)就会不定期的来回收这些垃圾对象。因此会看到上图堆内存中的曲线起伏变化很大。

>String

![String](https://raw.githubusercontent.com/fantianwen/MarkdownView/master/String.png)

>StringBuffer

![String](https://raw.githubusercontent.com/fantianwen/MarkdownView/master/StringBuffer.png)


>StringBuilder

![String](https://raw.githubusercontent.com/fantianwen/MarkdownView/master/StringBuilder.png)


但如果是遇到如下情况:

```
String concat1 = "lian" + "jia" + "ok";
 
String concat2 = "lian";
concat2 += "jia";
concat2 += "ok";
```

java 对 concat1 的处理速度也是快的惊人。耗时基本上都是 0 毫秒。这是因为 concat1 在编译期就可以被确定是一个字符常量。

当编译完成之后 concat1 的值其实就是 "lianjiaok", 因此, 在运行期间自然就不需要花费太多的时间来处理 concat1 了。如果是站在这个角度来看, 使用StringBuilder 完全不占优势, 在这种情况下, 如果是使用 StringBuilder 反而会使得程序运行需要耗费更多的时间。

但是 concat2 不一样, 由于 concat2 在编译期间不能够被确定, 因此, 在运行期间 JVM 会按老一套的做法, 将其转换成使用 StringBuilder 来实现。


#### 2、
从表格数据可以看出, StringBuilder 与 StringBuffer 在耗时上并不相差多少, 只是 StringBuilder 稍微快一些, 但是 StringBuilder 是冒着多线程不安全的潜在风险。这也是 StringBuilder 为赚取表格数据中的 1.7 毫秒( 若按表格的数据来算, 性能已经提升 20% 多 )所需要付出的代价。

 

#### 3、
综合来说:
StringBuilder 是 java 为 StringBuffer 提供的一个等价类, 但不保证同步。在不涉及多线程的操作情况下可以简易的替换 StringBuffer 来提升系统性能; StringBuffer 在性能上稍略StringBuilder, 但可以不用考虑线程安全问题; String 的 "+" 符号操作起来简单方便,String 的使用也很简单便捷, java 底层会转换成 StringBuilder 来实现, 特别如果是要在循环体内使用, 建议选择其余两个。 


## 使用带有参数的StringBuilder进行构造字符串

```
void expandCapacity(int minimumCapacity) {
        int newCapacity = value.length * 2 + 2;
        if (newCapacity - minimumCapacity < 0)
            newCapacity = minimumCapacity;
        if (newCapacity < 0) {
            if (minimumCapacity < 0) // overflow
                throw new OutOfMemoryError();
            newCapacity = Integer.MAX_VALUE;
        }
        value = Arrays.copyOf(value, newCapacity);
    }
```

>测试用例

```
private static final int LOOP_TIMES = 1000000;
    private static final String CONSTANT_STRING = "lianjia";


    public static void main(String[] args) {

        startup();
    }

    public static void testStringBuilder() {
        StringBuilder builder = new StringBuilder();
        long beginTime = System.currentTimeMillis();
        for (int i = 0; i < LOOP_TIMES; i++) {
            builder.append(CONSTANT_STRING);
        }
        builder.toString();
        long endTime = System.currentTimeMillis();
        System.out.print("StringBuilder : " + (endTime - beginTime) + "\t");
    }

    public static void testCapacityStringBuilder() {
        StringBuilder builder = new StringBuilder(LOOP_TIMES * CONSTANT_STRING.length());
        long beginTime = System.currentTimeMillis();
        for (int i = 0; i < LOOP_TIMES; i++) {
            builder.append(CONSTANT_STRING);
        }
        builder.toString();
        long endTime = System.currentTimeMillis();
        System.out.print("StringBuilder : " + (endTime - beginTime) + "\t");
    }

    public static void startup() {
        for (int i = 0; i < 10; i++) {
            System.out.print("The " + i + " [\t    ");
            testStringBuilder();
            testCapacityStringBuilder();
            System.out.println("]");
        }
    }

```



>测试结果

```
The 0 [	    StringBuilder : 42	StringBuilder : 17	]
The 1 [	    StringBuilder : 35	StringBuilder : 17	]
The 2 [	    StringBuilder : 22	StringBuilder : 10	]
The 3 [	    StringBuilder : 19	StringBuilder : 11	]
The 4 [	    StringBuilder : 37	StringBuilder : 14	]
The 5 [	    StringBuilder : 16	StringBuilder : 11	]
The 6 [	    StringBuilder : 22	StringBuilder : 11	]
The 7 [	    StringBuilder : 18	StringBuilder : 18	]
The 8 [	    StringBuilder : 34	StringBuilder : 15	]
The 9 [	    StringBuilder : 48	StringBuilder : 10	]
```

明显可以看出使用了指定大小缓冲区的构造器更具优势




