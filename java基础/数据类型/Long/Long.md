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

## digits

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

## value

存储Long对应的long的数值

```java
private final long value;
```

# 构造方法

## public Long(long value)

```java
  @Deprecated(since="9")
    public Long(long value) {
        this.value = value;
    }
```

## public Long(String s)

```java
 @Deprecated(since="9")
    public Long(String s) throws NumberFormatException {
        this.value = parseLong(s, 10);
    }
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

​	msb 最高有效位

​	lsb 最低有效位

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

 ## public static String toString(long i)

源码:

jdk11:

```java
  public static String toString(long i) {
        int size = stringSize(i);  //获取字符串长度
        if (COMPACT_STRINGS) {
            byte[] buf = new byte[size];  //生成数组长度
            getChars(i, size, buf);  //获取每一位字符
            return new String(buf, LATIN1);  //生成字符串
        } else {
            byte[] buf = new byte[size * 2];  
            StringUTF16.getChars(i, size, buf);
            return new String(buf, UTF16);
        }
    }
```

jdk8:

```java
    public static String toString(long i) {
        if (i == Long.MIN_VALUE)
            return "-9223372036854775808";
        int size = (i < 0) ? stringSize(-i) + 1 : stringSize(i);  //jdk8的stringSize不能够判断负数
        char[] buf = new char[size];
        getChars(i, size, buf);
        return new String(buf, true);
    }
```

## public static String toUnsignedString(long i)

源码:

```java
 public static String toUnsignedString(long i) {
        return toUnsignedString(i, 10);
    }
```

## static int getChars(long i, int index, byte[] buf)

参数:

​	i	转换的参数

​	index 最低字符位置  为了从数组的右到左填充

​	buf 存储用数组

源码:

jdk11:

```java
static int getChars(long i, int index, byte[] buf) {
        long q;
        int r;
        int charPos = index;

        boolean negative = (i < 0);
        if (!negative) {  //是否为负数
            i = -i;
        }

        // Get 2 digits/iteration using longs until quotient fits into an int
        while (i <= Integer.MIN_VALUE) {	//除100快速得每一位数值
            q = i / 100;
            r = (int)((q * 100) - i);
            i = q;
            buf[--charPos] = Integer.DigitOnes[r];
            buf[--charPos] = Integer.DigitTens[r];
        }

        // Get 2 digits/iteration using ints
        int q2;
        int i2 = (int)i;
        while (i2 <= -100) {	//范围为int了	用int参数计算  加快速度
            q2 = i2 / 100;
            r  = (q2 * 100) - i2;
            i2 = q2;
            buf[--charPos] = Integer.DigitOnes[r];
            buf[--charPos] = Integer.DigitTens[r];
        }

        // We know there are at most two digits left at this point.
        q2 = i2 / 10;	//处理小于100情况
        r  = (q2 * 10) - i2;
        buf[--charPos] = (byte)('0' + r);

        // Whatever left is the remaining digit.
        if (q2 < 0) {
            buf[--charPos] = (byte)('0' - q2);
        }

        if (negative) {  //添加符号
            buf[--charPos] = (byte)'-';
        }
        return charPos;
    }
```

## static int stringSize(long x)

jdk8只能处理正数

源码:

jdk11:

```java
static int stringSize(long x) {
        int d = 1;
        if (x >= 0) {
            d = 0;
            x = -x;
        }
        long p = -10;
        for (int i = 1; i < 19; i++) {
            if (x > p)
                return i + d;
            p = 10 * p;
        }
        return 19 + d;
    }
```

jdk8:

```java
   static int stringSize(long x) {
        long p = 10;
        for (int i=1; i<19; i++) {
            if (x < p)
                return i;
            p = 10*p;
        }
        return 19;
    }
```

## public static long parseLong(String s, int radix)

参数:

​	s 要转换的参数化

​	radix	s为几进制数

源码:

jdk11:

```java
  public static long parseLong(String s, int radix)
              throws NumberFormatException
    {
      	//判断参数合法性
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
        long limit = -Long.MAX_VALUE;

        if (len > 0) {	
            char firstChar = s.charAt(0);
            if (firstChar < '0') { // Possible leading "+" or "-"
                if (firstChar == '-') {
                    negative = true;
                    limit = Long.MIN_VALUE;
                } else if (firstChar != '+') {
                    throw NumberFormatException.forInputString(s);
                }
				//不能只有正负号
                if (len == 1) { // Cannot have lone "+" or "-"
                    throw NumberFormatException.forInputString(s);
                }
                i++;
            }
            //开始做转换
            long multmin = limit / radix;
            long result = 0;
            while (i < len) {
                // Accumulating negatively avoids surprises near MAX_VALUE
                int digit = Character.digit(s.charAt(i++),radix); //字符转数字
                if (digit < 0 || result < multmin) {
                    throw NumberFormatException.forInputString(s);
                }
                result *= radix;
                if (result < limit + digit) {
                    throw NumberFormatException.forInputString(s);
                }
                result -= digit;
            }
            return negative ? result : -result;
        } else {//空字符串情况
            throw NumberFormatException.forInputString(s);
        }
    }
```

jdk8:

```java
    public static long parseLong(String s, int radix)
              throws NumberFormatException
    {
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

        long result = 0;
        boolean negative = false;
        int i = 0, len = s.length();
        long limit = -Long.MAX_VALUE;
        long multmin;
        int digit;

        if (len > 0) {
            char firstChar = s.charAt(0);
            if (firstChar < '0') { // Possible leading "+" or "-"
                if (firstChar == '-') {
                    negative = true;
                    limit = Long.MIN_VALUE;
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
```

## public static long parseLong(String s)

源码:

```java
    public static long parseLong(String s) throws NumberFormatException {
        return parseLong(s, 10);
    }
```

## public static long parseUnsignedLong(String s, int radix)

源码:

```java
 public static long parseUnsignedLong(String s, int radix)
                throws NumberFormatException {
        //参数合法性判断
    	if (s == null)  {
            throw new NumberFormatException("null");
        }

        int len = s.length();
        if (len > 0) {
            //因为是无符号转换  所以不可能存在负数
            char firstChar = s.charAt(0);
            if (firstChar == '-') {
                throw new
                    NumberFormatException(String.format("Illegal leading minus sign " +
                                                       "on unsigned string %s.", s));
            } else {//不能存在符号问题 直接转换
                if (len <= 12 || // Long.MAX_VALUE in Character.MAX_RADIX is 13 digits
                    (radix == 10 && len <= 18) ) { // Long.MAX_VALUE in base 10 is 19 digits
                    return parseLong(s, radix);
                }

                // No need for range checks on len due to testing above.
                long first = parseLong(s, 0, len - 1, radix);
                int second = Character.digit(s.charAt(len - 1), radix);
                if (second < 0) {
                    throw new NumberFormatException("Bad digit at end of " + s);
                }
                long result = first * radix + second;

                /*
                 * Test leftmost bits of multiprecision extension of first*radix
                 * for overflow. The number of bits needed is defined by
                 * GUARD_BIT = ceil(log2(Character.MAX_RADIX)) + 1 = 7. Then
                 * int guard = radix*(int)(first >>> (64 - GUARD_BIT)) and
                 * overflow is tested by splitting guard in the ranges
                 * guard < 92, 92 <= guard < 128, and 128 <= guard, where
                 * 92 = 128 - Character.MAX_RADIX. Note that guard cannot take
                 * on a value which does not include a prime factor in the legal
                 * radix range.
                 */
                int guard = radix * (int) (first >>> 57);
                if (guard >= 128 ||
                    (result >= 0 && guard >= 128 - Character.MAX_RADIX)) {
                    /*
                     * For purposes of exposition, the programmatic statements
                     * below should be taken to be multi-precision, i.e., not
                     * subject to overflow.
                     *
                     * A) Condition guard >= 128:
                     * If guard >= 128 then first*radix >= 2^7 * 2^57 = 2^64
                     * hence always overflow.
                     *
                     * B) Condition guard < 92:
                     * Define left7 = first >>> 57.
                     * Given first = (left7 * 2^57) + (first & (2^57 - 1)) then
                     * result <= (radix*left7)*2^57 + radix*(2^57 - 1) + second.
                     * Thus if radix*left7 < 92, radix <= 36, and second < 36,
                     * then result < 92*2^57 + 36*(2^57 - 1) + 36 = 2^64 hence
                     * never overflow.
                     *
                     * C) Condition 92 <= guard < 128:
                     * first*radix + second >= radix*left7*2^57 + second
                     * so that first*radix + second >= 92*2^57 + 0 > 2^63
                     *
                     * D) Condition guard < 128:
                     * radix*first <= (radix*left7) * 2^57 + radix*(2^57 - 1)
                     * so
                     * radix*first + second <= (radix*left7) * 2^57 + radix*(2^57 - 1) + 36
                     * thus
                     * radix*first + second < 128 * 2^57 + 36*2^57 - radix + 36
                     * whence
                     * radix*first + second < 2^64 + 2^6*2^57 = 2^64 + 2^63
                     *
                     * E) Conditions C, D, and result >= 0:
                     * C and D combined imply the mathematical result
                     * 2^63 < first*radix + second < 2^64 + 2^63. The lower
                     * bound is therefore negative as a signed long, but the
                     * upper bound is too small to overflow again after the
                     * signed long overflows to positive above 2^64 - 1. Hence
                     * result >= 0 implies overflow given C and D.
                     */
                    throw new NumberFormatException(String.format("String value %s exceeds " +
                                                                  "range of unsigned long.", s));
                }
                return result;
            }
        } else {
            throw NumberFormatException.forInputString(s);
        }
    }
```

## public static long parseUnsignedLong(String s)

源码:

```java
    public static long parseUnsignedLong(String s) throws NumberFormatException {
        return parseUnsignedLong(s, 10);
    }

```

## public static Long valueOf(String s, int radix)

源码:

```java
    public static Long valueOf(String s, int radix) throws NumberFormatException {
        return Long.valueOf(parseLong(s, radix));
    }
```

## public static Long valueOf(long l)

源码:

```java
    @HotSpotIntrinsicCandidate
    public static Long valueOf(long l) {
        final int offset = 128;  //数组是从0开始的   偏移量
        if (l >= -128 && l <= 127) { // will cache   //命中缓存
            return LongCache.cache[(int)l + offset];
        }
        return new Long(l);
    }
```

## public static Long decode(String nm)

源码:

```java
 public static Long decode(String nm) throws NumberFormatException {
        int radix = 10;
        int index = 0;
        boolean negative = false;
        Long result;
		//判断数据合法性
        if (nm.isEmpty())
            throw new NumberFormatException("Zero length string");
        char firstChar = nm.charAt(0);
        // Handle sign, if present
     	//判断正负号
        if (firstChar == '-') {
            negative = true;
            index++;
        } else if (firstChar == '+')
            index++;
		//判断进制头
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
		//判断合法性
        if (nm.startsWith("-", index) || nm.startsWith("+", index))
            throw new NumberFormatException("Sign character in wrong position");
		//转换
        try {
            result = Long.valueOf(nm.substring(index), radix);
            result = negative ? Long.valueOf(-result.longValue()) : result;
        } catch (NumberFormatException e) {
            // If number is Long.MIN_VALUE, we'll end up here. The next line
            // handles this case, and causes any genuine format error to be
            // rethrown.
            String constant = negative ? ("-" + nm.substring(index))
                                       : nm.substring(index);
            result = Long.valueOf(constant, radix);
        }
        return result;
    }
```

## public static int hashCode(long value)

源码:

```java
    public static int hashCode(long value) {
        return (int)(value ^ (value >>> 32));
    }
```



# 内部类

## private static class LongCache

源码:

```java
    private static class LongCache {
        private LongCache(){}

        static final Long cache[] = new Long[-(-128) + 127 + 1];

        static {
            for(int i = 0; i < cache.length; i++)
                cache[i] = new Long(i - 128);
        }
    }
```

生成-128到127之间的Long类型参数

