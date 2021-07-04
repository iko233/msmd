# Integer

> 继承Number抽象类  实现了序列化接口

## 构造方法

> 从jdk9已经不建议使用  建议使用静态工厂方法valueOf ( int ) 
>
> It is rarely appropriate to use this constructor . The static factory valueOf ( int ) is generally a better choice, as it is likely to yield significantly better space and time

- Integer(int value)   

```java
@Deprecated(since="9")
    public Integer(int value) {
        this.value = value;
    }
```

- Integer(String s)

```java
    @Deprecated(since="9")
    public Integer(String s) throws NumberFormatException {
        this.value = parseInt(s, 10);
    }
```

## 常量
- MIN_VALUE  能够取得的最小值

``` java
@Native public static final int   MIN_VALUE = 0x80000000;
```

- MAX_VALUE  能够取得的最大值

```java
@Native public static final int   MAX_VALUE = 0x7fffffff;
```

- TYPE 获取当前类型的class对象

源码:

```java
public static final Class<Integer>  TYPE = (Class<Integer>) Class.getPrimitiveClass("int");
```

等价于

```java
Class<Integer> integerClass = int.class;
```

- digits 将数字转换为char的所有可能性
```java
static final char[] digits = {
        '0' , '1' , '2' , '3' , '4' , '5' ,
        '6' , '7' , '8' , '9' , 'a' , 'b' ,
        'c' , 'd' , 'e' , 'f' , 'g' , 'h' ,
        'i' , 'j' , 'k' , 'l' , 'm' , 'n' ,
        'o' , 'p' , 'q' , 'r' , 's' , 't' ,
        'u' , 'v' , 'w' , 'x' , 'y' , 'z'
};
```

- DigitTens

```java
static final byte[] DigitTens = {
        '0', '0', '0', '0', '0', '0', '0', '0', '0', '0',
        '1', '1', '1', '1', '1', '1', '1', '1', '1', '1',
        '2', '2', '2', '2', '2', '2', '2', '2', '2', '2',
        '3', '3', '3', '3', '3', '3', '3', '3', '3', '3',
        '4', '4', '4', '4', '4', '4', '4', '4', '4', '4',
        '5', '5', '5', '5', '5', '5', '5', '5', '5', '5',
        '6', '6', '6', '6', '6', '6', '6', '6', '6', '6',
        '7', '7', '7', '7', '7', '7', '7', '7', '7', '7',
        '8', '8', '8', '8', '8', '8', '8', '8', '8', '8',
        '9', '9', '9', '9', '9', '9', '9', '9', '9', '9',
        } ;
```

- DigitOnes

```java
 static final byte[] DigitOnes = {
        '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
        '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
        '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
        '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
        '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
        '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
        '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
        '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
        '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
        '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
        } ;
```

## 方法

### public static String toString(int i, int raddix)  
参数

​	i          要转换的数字

​	raddix 要转化为几进制

源码

jdk11/16

```java
 public static String toString(int i, int radix) {
     	//  判断进制数(radix)是否合理,不合理按照10进制计算
        if (radix < Character.MIN_RADIX || radix > Character.MAX_RADIX)
            radix = 10;
		
     	//如果是十进制就没比较转换了
        /* Use the faster version */
        if (radix == 10) {
            return toString(i);
        }
		
        if (COMPACT_STRINGS) { 
            byte[] buf = new byte[33]; //存储转换后字符
            boolean negative = (i < 0); //判断是否为负数
            int charPos = 32;
		   
            // 取正数方便进制转换计算
            if (!negative) {
                i = -i;
            }
		   //常规取余除法进行进制转换
            while (i <= -radix) {
                buf[charPos--] = (byte)digits[-(i % radix)];
                i = i / radix;
            }
            buf[charPos] = (byte)digits[-i];
		   //如果是负数补充负号
            if (negative) {
                buf[--charPos] = '-';
            }

            return StringLatin1.newString(buf, charPos, (33 - charPos));
        }
        return toStringUTF16(i, radix);
    }
```

> COMPACT_STRINGS 一个标志 是否可用一种内存改良的方式Latin1编码来生产字符串
>
> 参考:https://openjdk.java.net/jeps/254

> StringUTF16  大概意思是用两个byte来表示字符串原来的char(2字节)

> **在Java 9之前，Java**的标准内存表示`String`是在a中保存的UTF-16代码单元`char[]`。修改后的UTF-8用于其他环境; 例如，在“.class”文件中，以及对象序列化格式。
>
> 您可以通过查看`java.lang.String`类的源代码来确认这一点。
>
> 在Java 6更新21及更高版本中，有一个非标准选项（`-XX:UseCompressedStrings`）来启用压缩字符串。Java 7中删除了此功能。
>
> **对于Java 9及更高版本**，*默认情况下*，行为if `String`已更改为使用Strings的紧凑表示形式。

jdk8:

```java
 public static String toString(int i, int radix) {
        if (radix < Character.MIN_RADIX || radix > Character.MAX_RADIX)
            radix = 10;

        /* Use the faster version */
        if (radix == 10) {
            return toString(i);
        }

        char buf[] = new char[33];
        boolean negative = (i < 0);
        int charPos = 32;

        if (!negative) {
            i = -i;
        }

        while (i <= -radix) {
            buf[charPos--] = digits[-(i % radix)];
            i = i / radix;
        }
        buf[charPos] = digits[-i];

        if (negative) {
            buf[--charPos] = '-';
        }

        return new String(buf, charPos, (33 - charPos));
    }
```

### private static String toStringUTF16(int i, int radix)

实现压缩的字符串，jdk8中不存在

参数

​	i          要转换的数字

​	raddix 要转化为几进制

源码

```java
private static String toStringUTF16(int i, int radix) {
        byte[] buf = new byte[33 * 2];
        boolean negative = (i < 0);
        int charPos = 32;
        if (!negative) {
            i = -i;
        }
        while (i <= -radix) {
            StringUTF16.putChar(buf, charPos--, digits[-(i % radix)]);
            i = i / radix;
        }
        StringUTF16.putChar(buf, charPos, digits[-i]);

        if (negative) {
            StringUTF16.putChar(buf, --charPos, '-');
        }
        return StringUTF16.newString(buf, charPos, (33 - charPos));
    }
```

### public static String toUnsignedString(int i, int radix)

转换为无负号字符串

参数

​	i          要转换的数字

​	raddix 要转化为几进制

源码

````java
 public static String toUnsignedString(int i, int radix) {
        return Long.toUnsignedString(toUnsignedLong(i), radix);
    }
````

### public static String toHexString(int i)

转换为16进制字符串

参数: 

​	i          要转换的数字

源码:

```java
public static String toHexString(int i) {
        return toUnsignedString0(i, 4);
    }
```

### public static String toOctalString(int i)

转换为8进制字符串

参数:

​	i          要转换的数字

源码:

```java
public static String toOctalString(int i) {
        return toUnsignedString0(i, 3);
    }
```

### public static String toBinaryString(int i)

转换为二进制字符串

参数:

​	i          要转换的数字

源码:

```java
public static String toBinaryString(int i) {
        return toUnsignedString0(i, 1);
    }
```

### private static String toUnsignedString0(int val, int shift) 

进制转换最核心的函数

参数:

​	val 转换的数值

​	shift 转换为二的几次方的数值

源码:

jdk11/16

```java
private static String toUnsignedString0(int val, int shift) {
        // assert shift > 0 && shift <=5 : "Illegal shift value";
    	//Integer的二进制长度,减去二进制左侧未使用(0)的长度,求出当前数使用了几位
        int mag = Integer.SIZE - Integer.numberOfLeadingZeros(val);
    	//计算会生成几个字符
        int chars = Math.max(((mag + (shift - 1)) / shift), 1);
   		//判断是否压缩数组
        if (COMPACT_STRINGS) {
            byte[] buf = new byte[chars];
            formatUnsignedInt(val, shift, buf, 0, chars);
            return new String(buf, LATIN1);
        } else {
            byte[] buf = new byte[chars * 2];
            formatUnsignedIntUTF16(val, shift, buf, 0, chars);
            return new String(buf, UTF16);
        }
    }
```

jdk8

```java
 private static String toUnsignedString0(int val, int shift) {
        // assert shift > 0 && shift <=5 : "Illegal shift value";
        int mag = Integer.SIZE - Integer.numberOfLeadingZeros(val);
        int chars = Math.max(((mag + (shift - 1)) / shift), 1);
        char[] buf = new char[chars];

        formatUnsignedInt(val, shift, buf, 0, chars);

        // Use special constructor which takes over "buf".
        return new String(buf, true);
    }
```

### static void formatUnsignedInt(int val, int shift, char[] buf, int offset, int len)

转换为无负号二进制

参数:

​	val 要转换的数组

​	shift 转换为二的几次方进制

​	buf 存储结果的数组

​	offset 数组偏移量 (从数组哪个位置开始存储)

​	len 转换完的结果的长度

源码:

jdk11/16

```java
static void formatUnsignedInt(int val, int shift, char[] buf, int offset, int len) {
        // assert shift > 0 && shift <=5 : "Illegal shift value";
        // assert offset >= 0 && offset < buf.length : "illegal offset";
        // assert len > 0 && (offset + len) <= buf.length : "illegal length";
        int charPos = offset + len;
        int radix = 1 << shift;
        int mask = radix - 1;
        do {
            buf[--charPos] = Integer.digits[val & mask];
            val >>>= shift;
        } while (charPos > offset);
    }
```

jdk8

```java
 static int formatUnsignedInt(int val, int shift, char[] buf, int offset, int len) {
        int charPos = len;
        int radix = 1 << shift;
        int mask = radix - 1;
        do {
            buf[offset + --charPos] = Integer.digits[val & mask];
            val >>>= shift;
        } while (val != 0 && charPos > 0);

        return charPos;
    }
```

原理:

​	charPos 计算从数组开始插入字符的位置(从右往左)

​	radix为了计算mask(掩码)

> 比如8进制radix为(二进制)100
>
> mask等于(二进制)11
>
> 一个数按位与mask  就等于结果最低位的值

### static void formatUnsignedInt(int val, int shift, byte[] buf, int offset, int len)

转换为无负号二进制  压缩版     jdk8不存在

参数:

​	val 要转换的数组

​	shift 转换为二的几次方进制

​	buf 存储结果的数组

​	offset 数组偏移量 (从数组哪个位置开始存储)

​	len 转换完的结果的长度

源码:

```java
    static void formatUnsignedInt(int val, int shift, byte[] buf, int offset, int len) {
        int charPos = offset + len;
        int radix = 1 << shift;
        int mask = radix - 1;
        do {
            buf[--charPos] = (byte)Integer.digits[val & mask];
            val >>>= shift;
        } while (charPos > offset);
    }
```

### private static void formatUnsignedIntUTF16(int val, int shift, byte[] buf, int offset, int len)

转换为无负号二进制  压缩版     jdk8不存在

参数:

​	val 要转换的数组

​	shift 转换为二的几次方进制

​	buf 存储结果的数组

​	offset 数组偏移量 (从数组哪个位置开始存储)

​	len 转换完的结果的长度

源码:

```java
private static void formatUnsignedIntUTF16(int val, int shift, byte[] buf, int offset, int len) {
        int charPos = offset + len;
        int radix = 1 << shift;
        int mask = radix - 1;
        do {
            StringUTF16.putChar(buf, --charPos, Integer.digits[val & mask]);
            val >>>= shift;
        } while (charPos > offset);
    }
```

