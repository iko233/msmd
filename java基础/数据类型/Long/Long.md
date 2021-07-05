继承了Number抽象类,时间了Comparable<Long>比较接口

![image-20210706051227178](https://raw.githubusercontent.com/bitchigo/msImg/main/20210706051227.png)

# 常量

## MIN_VALUE

最小值

```java
@Native public static final long MIN_VALUE = 0x8000000000000000L;
```

## MAX_VALUE

最大值

```java
 @Native public static final long MAX_VALUE = 0x7fffffffffffffffL;
```

## TYPE

对象类

```java
 @SuppressWarnings("unchecked")
    public static final Class<Long>     TYPE = (Class<Long>) Class.getPrimitiveClass("long");
```

#  digits

进制转化用到的表

```java
    final static char[] digits = {
        '0' , '1' , '2' , '3' , '4' , '5' ,
        '6' , '7' , '8' , '9' , 'a' , 'b' ,
        'c' , 'd' , 'e' , 'f' , 'g' , 'h' ,
        'i' , 'j' , 'k' , 'l' , 'm' , 'n' ,
        'o' , 'p' , 'q' , 'r' , 's' , 't' ,
        'u' , 'v' , 'w' , 'x' , 'y' , 'z'
    };
```

# 方法

## public static String toString(long i, int radix) 

转换成字符串输出

参数:

​	i 输入的数值

​	radix 输出为几进制

源码:

jdk11

```java
    public static String toString(long i, int radix) {
        //判断参数是否合法
        if (radix < Character.MIN_RADIX || radix > Character.MAX_RADIX)
            radix = 10;
        //不需要进制转化直接10进制输出
        if (radix == 10)
            return toString(i);
		//判断压缩数组
        if (COMPACT_STRINGS) {
            byte[] buf = new byte[65];
            int charPos = 64;
            boolean negative = (i < 0);
			//是负数
            if (!negative) {
                i = -i;
            }
			//取余除法转换
            while (i <= -radix) {
                buf[charPos--] = (byte)Integer.digits[(int)(-(i % radix))];
                i = i / radix;
            }
            buf[charPos] = (byte)Integer.digits[(int)(-i)];

            if (negative) {
                buf[--charPos] = '-';
            }
            return StringLatin1.newString(buf, charPos, (65 - charPos));
        }
        return toStringUTF16(i, radix);
    }
```

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

## private static String toStringUTF16(int i, int radix)

参数:

​	i 输入数值

​	radix 转换的进制数

源码:

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

## public static String toUnsignedString(int i, int radix)

参数:

​	i 输入数值

​	radix 转换的进制数

源码:

```java
 public static String toUnsignedString(long i, int radix) {
        if (i >= 0)
            return toString(i, radix);
        else {
            switch (radix) {
            case 2:
                return toBinaryString(i);

            case 4:
                return toUnsignedString0(i, 2);

            case 8:
                return toOctalString(i);

            case 10:
                /*
                 * We can get the effect of an unsigned division by 10
                 * on a long value by first shifting right, yielding a
                 * positive value, and then dividing by 5.  This
                 * allows the last digit and preceding digits to be
                 * isolated more quickly than by an initial conversion
                 * to BigInteger.
                 */
                //先除10
                //把数字转换成一个无视符号的值
                long quot = (i >>> 1) / 5;
                //取出个位的值
                long rem = i - quot * 10;
                return toString(quot) + rem;

            case 16:
                return toHexString(i);

            case 32:
                return toUnsignedString0(i, 5);

            default:
                return toUnsignedBigInteger(i).toString(radix);
            }
        }
    }
```

## private static BigInteger toUnsignedBigInteger(long i)

源码:

```java
    private static BigInteger toUnsignedBigInteger(long i) {
        if (i >= 0L)
            return BigInteger.valueOf(i);
        else {
            int upper = (int) (i >>> 32);
            int lower = (int) i;

            // return (upper << 32) + lower
            return (BigInteger.valueOf(Integer.toUnsignedLong(upper))).shiftLeft(32).
                add(BigInteger.valueOf(Integer.toUnsignedLong(lower)));
        }
    }
```

## public static String toHexString(int i)

转换为16进制

源码:

```java
public static String toHexString(int i) {
        return toUnsignedString0(i, 4);
    }
```

## public static String toOctalString(int i)

转换为8进制

源码:

```java
    public static String toOctalString(int i) {
        return toUnsignedString0(i, 3);
    }
```

## public static String toBinaryString(int i)

转换为2进制

源码:

```java
public static String toBinaryString(int i) {
        return toUnsignedString0(i, 1);
    }
```

## static String toUnsignedString0(long val, int shift)

源码:

```java
static String toUnsignedString0(long val, int shift) {
        // assert shift > 0 && shift <=5 : "Illegal shift value";
   		//求出使用了几位
        int mag = Long.SIZE - Long.numberOfLeadingZeros(val);
    	//计算生成字符数量    
    	int chars = Math.max(((mag + (shift - 1)) / shift), 1);
        //压缩字符串
    	if (COMPACT_STRINGS) {
            byte[] buf = new byte[chars];
            formatUnsignedLong0(val, shift, buf, 0, chars);
            return new String(buf, LATIN1);
        } else {
            byte[] buf = new byte[chars * 2];
            formatUnsignedLong0UTF16(val, shift, buf, 0, chars);
            return new String(buf, UTF16);
        }
    }
```

## static void formatUnsignedLong0(long val, int shift, byte[] buf, int offset, int len) 

源码:

```java
    static void formatUnsignedLong0(long val, int shift, byte[] buf, int offset, int len) {
        int charPos = offset + len;
        int radix = 1 << shift;
        int mask = radix - 1;
        do {
            buf[--charPos] = (byte)Integer.digits[((int) val) & mask];
            val >>>= shift;
        } while (charPos > offset);
    }
```

## private static void formatUnsignedLong0UTF16(long val, int shift, byte[] buf, int offset, int len)

源码:

```java
    private static void formatUnsignedLong0UTF16(long val, int shift, byte[] buf, int offset, int len) {
        int charPos = offset + len;
        int radix = 1 << shift;
        int mask = radix - 1;
        do {
            StringUTF16.putChar(buf, --charPos, Integer.digits[((int) val) & mask]);
            val >>>= shift;
        } while (charPos > offset);
    }
```

## static String fastUUID(long lsb, long msb)

参数:

​	lsb

​	msb

源码:

```java
static String fastUUID(long lsb, long msb) {
        if (COMPACT_STRINGS) {
            byte[] buf = new byte[36];
            formatUnsignedLong0(lsb,        4, buf, 24, 12);
            formatUnsignedLong0(lsb >>> 48, 4, buf, 19, 4);
            formatUnsignedLong0(msb,        4, buf, 14, 4);
            formatUnsignedLong0(msb >>> 16, 4, buf, 9,  4);
            formatUnsignedLong0(msb >>> 32, 4, buf, 0,  8);

            buf[23] = '-';
            buf[18] = '-';
            buf[13] = '-';
            buf[8]  = '-';

            return new String(buf, LATIN1);
        } else {
            byte[] buf = new byte[72];

            formatUnsignedLong0UTF16(lsb,        4, buf, 24, 12);
            formatUnsignedLong0UTF16(lsb >>> 48, 4, buf, 19, 4);
            formatUnsignedLong0UTF16(msb,        4, buf, 14, 4);
            formatUnsignedLong0UTF16(msb >>> 16, 4, buf, 9,  4);
            formatUnsignedLong0UTF16(msb >>> 32, 4, buf, 0,  8);

            StringUTF16.putChar(buf, 23, '-');
            StringUTF16.putChar(buf, 18, '-');
            StringUTF16.putChar(buf, 13, '-');
            StringUTF16.putChar(buf,  8, '-');

            return new String(buf, UTF16);
        }
    }
```

