
title: String,StringBuffer和StringBuilder
date: 2016-03-08 18:33:52
tags: Java
---


# String,StringBuffer和StringBuilder

## String是constant的,其值在创建之后不能被改变，String buffer支持可变的String。String包括常见的对单独的characters的操作，包括

### 0、数据结构

- `implements Serializable,Comparable,CharSequence`

- char[] value
- int hash
- 一个序列化的UID（static）：不会被序列化
- ObjectStreamField[]的对象(不知道干啥的)

<!-- more -->

### 1、比较字符串(按照字典排序法进行两个字符串的大小比较)

```
compareTo和compareToIgnoreCase方法
```

实现细则(compareTo)：

```
 public int compareTo(String anotherString) {
        int len1 = value.length;
        int len2 = anotherString.value.length;
        int lim = Math.min(len1, len2);
        char v1[] = value;
        char v2[] = anotherString.value;

        int k = 0;
        while (k < lim) {
            char c1 = v1[k];
            char c2 = v2[k];
            if (c1 != c2) {
                return c1 - c2;
            }
            k++;
        }
        return len1 - len2;
    }
```

而`compareToIgnoreCase`方法则是在比较的时候在每个string的位置上（对不同的状况）进行大小的比较。以下：

```
char c1 = s1.charAt(i);
                char c2 = s2.charAt(i);
                if (c1 != c2) {
                    c1 = Character.toUpperCase(c1);
                    c2 = Character.toUpperCase(c2);
                    if (c1 != c2) {
                        c1 = Character.toLowerCase(c1);
                        c2 = Character.toLowerCase(c2);
                        if (c1 != c2) {
                            // No overflow because of numeric promotion
                            return c1 - c2;
                        }
                    }
                }
```

这里先比较两个地方原有的字符的大小，再统一`toUpperCase`，如果还是不符，就`toLowerCase
`。

#### 1.1、string的`compareToIgnoreCase`方法

```
public int compareToIgnoreCase(String str) {
        return CASE_INSENSITIVE_ORDER.compare(this, str);
    }
```

这样的实现一个比较的规则的实例（`CASE_INSENSITIVE_ORDER = new CaseInsensitiveComparator()` ），并直接传入参数进行比较的模式比较讨巧。


### 2、搜索字符串

### 3、拆分字符串

#### 3.1、 `subString`

```
new String(value, beginIndex, subLen)
```

然后

```
this.value = Arrays.copyOfRange(value, offset, offset+count);
```

其中`Arrays.copyOfRange`会调用

```
System.arraycopy(original, from, copy, 0,
                         Math.min(original.length - from, newLength));
```

生成一个char[]类型的数组，就是String中的成员属性`value`。

> `System.arraycopy是一个java的native方法`


### 4、拷贝一个字符串并将其全部字符转化成大写或者小写



### 5、关于String的一些拼接字符的操作都是通过StringBuilder或者StringBuffer继承而来。

#### 5.1、`concat`

`concat`的操作仍旧是通过`System.arraycopy`操作复制出一个char[],然后在new出一个String。


# StringBuffer

## thread-safe,mutable sequence characters，并且在任意的时间，都包括一个可变的sequences of characters。`sb.append(x)`和`sb.insert(sb.length(),x)`具备一样的功能。

## 数据结构

在很多方法上使用了`synchronized`关键做同步

## API
#### 1、`insert`

```
System.arraycopy(value, offset, value, offset + len, count - offset);
System.arraycopy(str, 0, value, offset, len);
```

在插入的时候回分批次的进行插入，第一次把成员变量`char[] value`扩展成一个`offset+len`长度的`char[]`，并把之前的value值复制进进去（基于自身的扩容），第二次就直接把需要扩充的`str`复制到value中，并返回`this`，这样，就实现了`insert`的操作。

# StringBuilder

默认构造器是16个character

其实和StringBuffer基本是一样的。



