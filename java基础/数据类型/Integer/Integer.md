# Integer

> 继承Number抽象类  Number类实现了序列化接口 
>
> Integer实现了接口Comparable<Integer>   比较用

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

- DigitTens 100个数中十位的值

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

- DigitOnes 100个数中个位的值

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

- sizeTable 获取字符串长度打的表

jdk11中已经不再使用  由于兼容性继续保留


> // Left here for compatibility reasons, see JDK-8143900.

```java
final static int [] sizeTable = { 9, 99, 999, 9999, 99999, 999999, 9999999,
                                      99999999, 999999999, Integer.MAX_VALUE };
```

- value 存储对应的int的值

```java
 private final int value;
```

### Bit twiddling(比特玩弄)

- SIZE  int类型的位数

```java
@Native public static final int SIZE = 32;
```

- BYTES 占几比特空间

```java
 public static final int BYTES = SIZE / Byte.SIZE;
```



-  serialVersionUID  实现序列化产生的

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
### public static String toString(int i)

转换为字符串(10进制)

参数:

​	i   转换的数字  

源码:

jdk11/16:

```java
@HotSpotIntrinsicCandidate    //hotspo有基于cpu指令的高效率执行方法
    public static String toString(int i) {
        int size = stringSize(i);
        if (COMPACT_STRINGS) {
            byte[] buf = new byte[size];
            getChars(i, size, buf);
            return new String(buf, LATIN1);
        } else {
            byte[] buf = new byte[size * 2];
            StringUTF16.getChars(i, size, buf);
            return new String(buf, UTF16);
        }
    }
```

jdk8:

```java
public static String toString(int i) {
        if (i == Integer.MIN_VALUE)
            return "-2147483648";
        int size = (i < 0) ? stringSize(-i) + 1 : stringSize(i);
        char[] buf = new char[size];
        getChars(i, size, buf);
        return new String(buf, true);
    }
```

### public static String toUnsignedString(int i)

转换为无符号的字符串

参数:

​	i 转换的数字

源码:

```java
public static String toUnsignedString(int i) {
        return Long.toString(toUnsignedLong(i));
    }
```



### static int getChars(int i, int index, byte[] buf) {

将整形转换成字符串

> i 为Integer.MIN_VALUE时会溢出

参数:

源码:

jdk11/16

```java
 static int getChars(int i, int index, byte[] buf) {
        int q, r;
        int charPos = index;
	    //判断是否为负数
        boolean negative = i < 0;
        if (!negative) {
            i = -i;
        }
		//当数为三位以上的时候   一次取两位
        // Generate two digits per iteration
        while (i <= -100) {
            q = i / 100;
            r = (q * 100) - i;
            i = q;
            buf[--charPos] = DigitOnes[r];
            buf[--charPos] = DigitTens[r];
        }
		//最多两位数
        // We know there are at most two digits left at this point.
        q = i / 10;
        r = (q * 10) - i;
        buf[--charPos] = (byte)('0' + r);

        // Whatever left is the remaining digit.
        if (q < 0) {
            buf[--charPos] = (byte)('0' - q);
        }

        if (negative) {
            buf[--charPos] = (byte)'-';
        }
        return charPos;
    }
```

jdk8:

```java
static void getChars(int i, int index, char[] buf) {
        int q, r;
        int charPos = index;
        char sign = 0;

        if (i < 0) {  
            sign = '-';
            i = -i;   //会溢出的原因  int范围是-2147483648~2147483647
        }

        // Generate two digits per iteration
        while (i >= 65536) {
            q = i / 100;
        // really: r = i - (q * 100);
            r = i - ((q << 6) + (q << 5) + (q << 2));
            i = q;
            buf [--charPos] = DigitOnes[r];
            buf [--charPos] = DigitTens[r];
        }
		//快速模式
        // Fall thru to fast mode for smaller numbers
        // assert(i <= 65536, i);
        for (;;) {
            q = (i * 52429) >>> (16+3); //实际是除以10
            r = i - ((q << 3) + (q << 1));  // r = i-(q*10) ...
            buf [--charPos] = digits [r];
            i = q;
            if (i == 0) break;
        }
        if (sign != 0) {
            buf [--charPos] = sign;
        }
    }
```

### static int stringSize(int x)

获取数字转字符串的长度

参数:

​	 x 需要转换的数

源码:

jdk11/16

```java
static int stringSize(int x) {   
        int d = 1;//符号位所需长度(-)
        if (x >= 0) {  //如果为正数反转为负数 方便计算
            d = 0;
            x = -x;
        }
        int p = -10;   //比较用 对应jdk8中的表
        for (int i = 1; i < 10; i++) {
            if (x > p)
                return i + d;
            p = 10 * p;
        }
        return 10 + d;
    }
```

jdk8

```java
    static int stringSize(int x) {   //和表做比较确定长度
        for (int i=0; ; i++)
            if (x <= sizeTable[i])
                return i+1;
    }
```

### public static int parseInt(String s, int radix)

字符串转int

参数:

​	s 输入字符

​	radix 字符串s为几进制参数

源码:

jdk11/16

```java
public static int parseInt(String s, int radix)
                throws NumberFormatException
    {
        /*
         * WARNING: This method may be invoked early during VM initialization
         * before IntegerCache is initialized. Care must be taken to not use
         * the valueOf method.
         */
		//参数合法性判断
        if (s == null) {  
            throw new NumberFormatException("null");
        }

        if (radix < Character.MIN_RADIX) { 
            throw new NumberFormatException("radix " + radix +
                                            " less than Character.MIN_RADIX");
        }

        if (radix > Character.MAX_RADIX) {
            throw new NumberFormatException("radix " + radix +
                                            " greater than Character.MAX_RADIX");
        }

        boolean negative = false;
        int i = 0, len = s.length();
        int limit = -Integer.MAX_VALUE;

        if (len > 0) {  
            char firstChar = s.charAt(0);
            if (firstChar < '0') { // Possible leading "+" or "-" //第一位是否为符号位判断
                if (firstChar == '-') {
                    negative = true;
                    limit = Integer.MIN_VALUE;
                } else if (firstChar != '+') {
                    throw NumberFormatException.forInputString(s);
                }

                if (len == 1) { // Cannot have lone "+" or "-"
                    throw NumberFormatException.forInputString(s);
                }
                i++;
            }
            int multmin = limit / radix;   //溢出判断用
            int result = 0;
            while (i < len) {
                // Accumulating negatively avoids surprises near MAX_VALUE
                int digit = Character.digit(s.charAt(i++), radix);   //字符转数字   不合法值返回-1
                if (digit < 0 || result < multmin) {   //溢出判断
                    throw NumberFormatException.forInputString(s);
                }
                result *= radix;
                if (result < limit + digit) {  //溢出判断
                    throw NumberFormatException.forInputString(s);
                }
                result -= digit;
            }
            return negative ? result : -result;
        } else {
            throw NumberFormatException.forInputString(s);
        }
    }
```

jdk8:

````java
public static int parseInt(String s, int radix)
                throws NumberFormatException
    {
        /*
         * WARNING: This method may be invoked early during VM initialization
         * before IntegerCache is initialized. Care must be taken to not use
         * the valueOf method.
         */

        if (s == null) {
            throw new NumberFormatException("null");
        }

        if (radix < Character.MIN_RADIX) {
            throw new NumberFormatException("radix " + radix +
                                            " less than Character.MIN_RADIX");
        }

        if (radix > Character.MAX_RADIX) {
            throw new NumberFormatException("radix " + radix +
                                            " greater than Character.MAX_RADIX");
        }

        int result = 0;
        boolean negative = false;
        int i = 0, len = s.length();
        int limit = -Integer.MAX_VALUE;
        int multmin;
        int digit;

        if (len > 0) {
            char firstChar = s.charAt(0);
            if (firstChar < '0') { // Possible leading "+" or "-"
                if (firstChar == '-') {
                    negative = true;
                    limit = Integer.MIN_VALUE;
                } else if (firstChar != '+')
                    throw NumberFormatException.forInputString(s);

                if (len == 1) // Cannot have lone "+" or "-"
                    throw NumberFormatException.forInputString(s);
                i++;
            }
            multmin = limit / radix;
            while (i < len) {
                // Accumulating negatively avoids surprises near MAX_VALUE
                digit = Character.digit(s.charAt(i++),radix);
                if (digit < 0) {
                    throw NumberFormatException.forInputString(s);
                }
                if (result < multmin) {
                    throw NumberFormatException.forInputString(s);
                }
                result *= radix;
                if (result < limit + digit) {
                    throw NumberFormatException.forInputString(s);
                }
                result -= digit;
            }
        } else {
            throw NumberFormatException.forInputString(s);
        }
        return negative ? result : -result;
    }
````

### public static int parseInt(CharSequence s, int beginIndex, int endIndex, int radix)

从jdk9加入,截取字符串转换进制

参数:

​	s 输入的字符参数

​	beginIndex 开始的位置

​	endIndex 截止的位置

​	radix 输入参数的进制

>CharSequence  字符串序列接口,  实现类有String,StringBuilder,StringBuffer

源码:

```java
  public static int parseInt(CharSequence s, int beginIndex, int endIndex, int radix)
                throws NumberFormatException {
        
      	//数据合法性判断
      	s = Objects.requireNonNull(s);

        if (beginIndex < 0 || beginIndex > endIndex || endIndex > s.length()) {
            throw new IndexOutOfBoundsException();
        }
        if (radix < Character.MIN_RADIX) {
            throw new NumberFormatException("radix " + radix +
                                            " less than Character.MIN_RADIX");
        }
        if (radix > Character.MAX_RADIX) {
            throw new NumberFormatException("radix " + radix +
                                            " greater than Character.MAX_RADIX");
        }

        boolean negative = false;
        int i = beginIndex;
        int limit = -Integer.MAX_VALUE;

        if (i < endIndex) {
            char firstChar = s.charAt(i);
            if (firstChar < '0') { // Possible leading "+" or "-"
                if (firstChar == '-') {
                    negative = true;
                    limit = Integer.MIN_VALUE;
                } else if (firstChar != '+') {
                    throw NumberFormatException.forCharSequence(s, beginIndex,
                            endIndex, i);
                }
                i++;
                if (i == endIndex) { // Cannot have lone "+" or "-"
                    throw NumberFormatException.forCharSequence(s, beginIndex,
                            endIndex, i);
                }
            }
            int multmin = limit / radix;
            int result = 0;
            while (i < endIndex) {
                // Accumulating negatively avoids surprises near MAX_VALUE
                int digit = Character.digit(s.charAt(i), radix);
                if (digit < 0 || result < multmin) {
                    throw NumberFormatException.forCharSequence(s, beginIndex,
                            endIndex, i);
                }
                result *= radix;
                if (result < limit + digit) {
                    throw NumberFormatException.forCharSequence(s, beginIndex,
                            endIndex, i);
                }
                i++;
                result -= digit;
            }
            return negative ? result : -result;
        } else {
            throw NumberFormatException.forInputString("");
        }
    }
```

### public static int parseInt(String s)

直接调用的 parseInt(String s ,int radix)

参数:

​	s 输入要转换的10进制

源码:

```java
    public static int parseInt(String s) throws NumberFormatException {
        return parseInt(s,10);
    }
```

### public static int parseUnsignedInt(String s, int radix)

无符号字符串转int

参数:

​	s 输入的字符串

​	radix 输入参数的进制数

源码:

```java
    public static int parseUnsignedInt(String s, int radix)
                throws NumberFormatException {
        //参数合法性判断
        if (s == null)  {
            throw new NumberFormatException("null");
        }

        int len = s.length();
        if (len > 0) {
            char firstChar = s.charAt(0);
            if (firstChar == '-') {   //因为是无符号  所以-不可能存在
                throw new
                    NumberFormatException(String.format("Illegal leading minus sign " +
                                                       "on unsigned string %s.", s));
            } else {
                if (len <= 5 || // Integer.MAX_VALUE in Character.MAX_RADIX is 6 digits
                    (radix == 10 && len <= 9) ) { // Integer.MAX_VALUE in base 10 is 10 digits
                    return parseInt(s, radix);   //可以直接做转换
                } else {
                    long ell = Long.parseLong(s, radix);   //通过long 的方法转换
                    if ((ell & 0xffff_ffff_0000_0000L) == 0) {  //判断是否超过int的最大值  判断是否越界
                        return (int) ell;
                    } else {
                        throw new
                            NumberFormatException(String.format("String value %s exceeds " +
                                                                "range of unsigned int.", s));
                    }
                }
            }
        } else {
            throw NumberFormatException.forInputString(s, radix);
        }
    }
```

### public static int parseUnsignedInt(CharSequence s, int beginIndex, int endIndex, int radix)

截取字符串的进制转换    从jdk9开始

参数:

​	s 输入的字符参数

​	beginIndex 开始的位置

​	endIndex 截止的位置

​	radix 输入参数的进制

源码:

```java
 public static int parseUnsignedInt(CharSequence s, int beginIndex, int endIndex, int radix)
                throws NumberFormatException {
        s = Objects.requireNonNull(s);

        if (beginIndex < 0 || beginIndex > endIndex || endIndex > s.length()) {
            throw new IndexOutOfBoundsException();
        }
        int start = beginIndex, len = endIndex - beginIndex;

        if (len > 0) {
            char firstChar = s.charAt(start);
            if (firstChar == '-') {
                throw new
                    NumberFormatException(String.format("Illegal leading minus sign " +
                                                       "on unsigned string %s.", s));
            } else {
                if (len <= 5 || // Integer.MAX_VALUE in Character.MAX_RADIX is 6 digits
                        (radix == 10 && len <= 9)) { // Integer.MAX_VALUE in base 10 is 10 digits
                    return parseInt(s, start, start + len, radix);
                } else {
                    long ell = Long.parseLong(s, start, start + len, radix);
                    if ((ell & 0xffff_ffff_0000_0000L) == 0) {
                        return (int) ell;
                    } else {
                        throw new
                            NumberFormatException(String.format("String value %s exceeds " +
                                                                "range of unsigned int.", s));
                    }
                }
            }
        } else {
            throw new NumberFormatException("");
        }
    }
```

### public static int parseUnsignedInt(String s) throws NumberFormatException

源码:

```java
    public static int parseUnsignedInt(String s) throws NumberFormatException {
        return parseUnsignedInt(s, 10);
    }
```

### public static Integer valueOf(String s, int radix)

源码:

```java
    public static Integer valueOf(String s, int radix) throws NumberFormatException {
        return Integer.valueOf(parseInt(s,radix));
    }
```

### public static Integer valueOf(String s)

源码:

```java
    public static Integer valueOf(String s) throws NumberFormatException {
        return Integer.valueOf(parseInt(s, 10));
    }
```

### public static Integer valueOf(int i)

参数:

​	i 要转换为Integer类的int值

源码:

```java
    @HotSpotIntrinsicCandidate   //hotspot有更高效率基于cpu指令的运行方式
    public static Integer valueOf(int i) {
        if (i >= IntegerCache.low && i <= IntegerCache.high)  //判断缓存中是否有这个对象
            return IntegerCache.cache[i + (-IntegerCache.low)];
        return new Integer(i);  //否则new一个
    }
```

### public static int hashCode(int value)

int的hashcode就是自己

源码

```java
   public static int hashCode(int value) {
        return value;
    }
```

### public boolean equals(Object obj)

判断值是否相等

源码:

```java
    public boolean equals(Object obj) {
        if (obj instanceof Integer) {
            return value == ((Integer)obj).intValue();
        }
        return false;
    }
```

### public static Integer getInteger(String nm, int val)

尝试从系统配置中获取nm参数的变量的数值(字符串),然后进行进制解析转换为Integer包装类

如果nm不存在,则返回val的值

参数:

​	nm 参数值

​	val 如果不存在返回的数值

源码:

```java
 public static Integer getInteger(String nm, int val) {
        Integer result = getInteger(nm, null);
        return (result == null) ? Integer.valueOf(val) : result;
    }
```

### public static Integer getInteger(String nm, Integer val) 

尝试从系统配置中获取nm参数的变量的数值(字符串),然后进行进制解析转换为Integer包装类

如果nm不存在,则返回val的值

参数:

​	nm 参数值

​	val 如果不存在返回的数值

源码:

```java
 public static Integer getInteger(String nm, Integer val) {
        String v = null;
        try {
            v = System.getProperty(nm);   //从系统中获取参数
        } catch (IllegalArgumentException | NullPointerException e) {
        }
        if (v != null) {
            try {
                return Integer.decode(v);   //进制转化
            } catch (NumberFormatException e) {
            }
        }
        return val;   //nm参数不存在的时候直接返回val的值
    }
```

### public static Integer decode(String nm)

对参数做进制转化,默认为10进制,支持输入参数为16进制或者8进制

参数:

​	nm 16/10/8进制字符串

源码:

```java
    public static Integer decode(String nm) throws NumberFormatException {
        int radix = 10;   //进制数
        int index = 0;   //读取字符串位置
        boolean negative = false;  //是否为负数
        Integer result; //返回的结果
		//判断参数是否合法
        if (nm.isEmpty())
            throw new NumberFormatException("Zero length string");
        char firstChar = nm.charAt(0);
        // Handle sign, if present
        if (firstChar == '-') {
            negative = true;
            index++;
        } else if (firstChar == '+')
            index++;

        // Handle radix specifier, if present
        if (nm.startsWith("0x", index) || nm.startsWith("0X", index)) {
            index += 2;
            radix = 16;
        }
        else if (nm.startsWith("#", index)) {
            index ++;
            radix = 16;
        }
        else if (nm.startsWith("0", index) && nm.length() > 1 + index) {
            index ++;
            radix = 8;
        }

        if (nm.startsWith("-", index) || nm.startsWith("+", index))
            throw new NumberFormatException("Sign character in wrong position");

        try {
            result = Integer.valueOf(nm.substring(index), radix);
            result = negative ? Integer.valueOf(-result.intValue()) : result;
        } catch (NumberFormatException e) {
            // If number is Integer.MIN_VALUE, we'll end up here. The next line
            // handles this case, and causes any genuine format error to be
            // rethrown.
            //Integer.MIN_VALUE会异常
            String constant = negative ? ("-" + nm.substring(index))
                                       : nm.substring(index);
            result = Integer.valueOf(constant, radix);
        }
        return result;
    }
```

### public int compareTo(Integer anotherInteger)

继承比较接口而来的

源码:

```java
    public int compareTo(Integer anotherInteger) {
        return compare(this.value, anotherInteger.value);
    }

```

### public static int compare(int x, int y)

源码:

```java
    public static int compare(int x, int y) {
        return (x < y) ? -1 : ((x == y) ? 0 : 1);
    }

```

### public static int compareUnsigned(int x, int y)

通过==与Integer.MIN_VALUE相加==再比较实现的  防止溢出

源码:

```java
public static int compareUnsigned(int x, int y) {
        return compare(x + MIN_VALUE, y + MIN_VALUE);
   }
```

### public static long toUnsignedLong(int x)

一个数值跟根据二进制转换为long类型

源码:

```java
public static long toUnsignedLong(int x) {
        return ((long) x) & 0xffffffffL;  //0xffffffffL是为了让符号位当正常数值来处理
    }
```

### public static int divideUnsigned(int dividend, int divisor)

无符号除法

源码:

```java
 public static int divideUnsigned(int dividend, int divisor) {
        // In lieu of tricky code, for now just use long arithmetic.
        return (int)(toUnsignedLong(dividend) / toUnsignedLong(divisor));
    }
```

### public static int remainderUnsigned(int dividend, int divisor)

无符号取余

源码:

 ```java
 public static int remainderUnsigned(int dividend, int divisor) {
         // In lieu of tricky code, for now just use long arithmetic.
         return (int)(toUnsignedLong(dividend) % toUnsignedLong(divisor));
     }
 ```

### public static int highestOneBit(int i)



## 内部类

### IntegerCache

原理:

预先把==-128到127(可通过vm调整)的数预先生成提高new对象的效率

源码:

```java
 private static class IntegerCache {
        static final int low = -128;//最小值
        static final int high;//最大值
        static final Integer cache[];//存储预先new的对象的数组

        static {
            // high value may be configured by property
            int h = 127;//默认最大值
            //从配置中读取最大值
            String integerCacheHighPropValue =
                VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
            if (integerCacheHighPropValue != null) {
                try {
                    //最大值最小为127
                    int i = parseInt(integerCacheHighPropValue);
                    i = Math.max(i, 127);
                    // Maximum array size is Integer.MAX_VALUE
                    h = Math.min(i, Integer.MAX_VALUE - (-low) -1);//不超过最大长度Integer.MAX_VALUE
                } catch( NumberFormatException nfe) {
                    // If the property cannot be parsed into an int, ignore it.
                }
            }
            high = h;
			//创建存储数组  并生成对应对象
            cache = new Integer[(high - low) + 1];
            int j = low;
            for(int k = 0; k < cache.length; k++)
                cache[k] = new Integer(j++);

            // range [-128, 127] must be interned (JLS7 5.1.7)
            assert IntegerCache.high >= 127;
        }

        private IntegerCache() {}
    }
```

