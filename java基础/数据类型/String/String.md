实现了序列化 ,比较,字符序列接口

![image-20210710100830395](https://raw.githubusercontent.com/bitchigo/msImg/main/20210710100830.png)

# 常量

![image-20210710102057019](https://raw.githubusercontent.com/bitchigo/msImg/main/20210710102057.png)

## value[]

jdk11:

```java
@Stable
    private final byte[] value;
```

jdk8:

```java
 private final char value[];
```

> 压缩数组 改byte来表示char了

## coder

确定哪种编码模式

```java
private final byte coder;
```



## hash

> 默认为0

```java
 private int hash; // Default to 0
```

## serialVersionUID

序列化

```java
private static final long serialVersionUID = -6849794470754667710L;
```

##  COMPACT_STRINGS

压缩数组   

```java
static final boolean COMPACT_STRINGS;

    static {
        COMPACT_STRINGS = true;
    }
```

## serialPersistentFields

```java
   private static final ObjectStreamField[] serialPersistentFields =
        new ObjectStreamField[0];
```

# 构造方法

## public String() 

源码:

jdk11:

```java
  public String() {
        this.value = "".value;
        this.coder = "".coder;
    }
```

jdk8:

```java
 public String() {
        this.value = "".value;
    }
```

## public String(String original) 

源码:

jdk11:

````java
  @HotSpotIntrinsicCandidate
    public String(String original) {
        this.value = original.value;
        this.coder = original.coder;
        this.hash = original.hash;
    }
````

jdk8:

```java
    public String(String original) {
        this.value = original.value;
        this.hash = original.hash;
    }
```

## public String(char value[])

> 11是通过compress  8是通过copyOf

源码:

jdk11:

```java
 public String(char value[]) {
        this(value, 0, value.length, null);
    }
```

jdk8:

```java
    public String(char value[]) {
        this.value = Arrays.copyOf(value, value.length);
    }
```

## public String(char value[], int offset, int count)

参数:

​	value 存放的数组

​	offset 偏移量

​	count 字符数量

源码:

jdk11:

```java
public String(char value[], int offset, int count) {
        this(value, offset, count, rangeCheck(value, offset, count));
    }
```

jdk8:

```java
    public String(char value[], int offset, int count) {
        if (offset < 0) {   //偏移量不合法
            throw new StringIndexOutOfBoundsException(offset);
        }
        if (count <= 0) {  //字符数量不合法
            if (count < 0) {
                throw new StringIndexOutOfBoundsException(count);
            }
            if (offset <= value.length) {
                this.value = "".value;  //偏移量不合法
                return;
            }
        }
        // Note: offset or count might be near -1>>>1.
        if (offset > value.length - count) {  //参数不合法
            throw new StringIndexOutOfBoundsException(offset + count);
        }
        this.value = Arrays.copyOfRange(value, offset, offset+count);  //截取复制
    }
```

## public String(int[] codePoints, int offset, int count)

源码:

jdk11:

```java
    public String(int[] codePoints, int offset, int count) {
        checkBoundsOffCount(offset, count, codePoints.length);   //检查参数合法性
        if (count == 0) {  //空字符串情况
            this.value = "".value;
            this.coder = "".coder;
            return;
        }
        if (COMPACT_STRINGS) {  //是否为压缩数组
            byte[] val = StringLatin1.toBytes(codePoints, offset, count);	//生成字符数组
            if (val != null) {// 配置相关信息
                this.coder = LATIN1;
                this.value = val;
                return;
            }
        }
        this.coder = UTF16;
        this.value = StringUTF16.toBytes(codePoints, offset, count);
    }
```

jdk8:

```java
 public String(int[] codePoints, int offset, int count) {
        //判断参数合法性
     	if (offset < 0) {
            throw new StringIndexOutOfBoundsException(offset);
        }
        if (count <= 0) {
            if (count < 0) {
                throw new StringIndexOutOfBoundsException(count);
            }
            if (offset <= codePoints.length) {
                this.value = "".value;
                return;
            }
        }
        // Note: offset or count might be near -1>>>1.
        if (offset > codePoints.length - count) {
            throw new StringIndexOutOfBoundsException(offset + count);
        }

        final int end = offset + count;	//字符截止位置

        // Pass 1: Compute precise size of char[]
        int n = count;
        for (int i = offset; i < end; i++) {  //确定需要多大的char数组
            int c = codePoints[i];
            if (Character.isBmpCodePoint(c))	//用单char能够表示
                continue;
            else if (Character.isValidCodePoint(c))  //双char表示
                n++;
            else throw new IllegalArgumentException(Integer.toString(c));	//特殊情况
        }

        // Pass 2: Allocate and fill in char[]
        final char[] v = new char[n];

        for (int i = offset, j = 0; i < end; i++, j++) {  //将数据填充
            int c = codePoints[i];
            if (Character.isBmpCodePoint(c))
                v[j] = (char)c;
            else
                Character.toSurrogates(c, v, j++);
        }

        this.value = v;
    }

```

## public String(byte ascii[], int hibyte, int offset, int count)

源码:

jdk11:

```java
 @Deprecated(since="1.1")
    public String(byte ascii[], int hibyte, int offset, int count) {
        //判断参数合法性
        checkBoundsOffCount(offset, count, ascii.length);
        if (count == 0) {	//配置信息
            this.value = "".value;
            this.coder = "".coder;
            return;
        }
        if (COMPACT_STRINGS && (byte)hibyte == 0) {
            this.value = Arrays.copyOfRange(ascii, offset, offset + count);
            this.coder = LATIN1;
        } else {
            hibyte <<= 8;
            byte[] val = StringUTF16.newBytesFor(count);
            for (int i = 0; i < count; i++) {
                StringUTF16.putChar(val, i, hibyte | (ascii[offset++] & 0xff));
            }
            this.value = val;
            this.coder = UTF16;
        }
    }
```

jdk8:

```java
    @Deprecated
    public String(byte ascii[], int hibyte, int offset, int count) {
        checkBounds(ascii, offset, count);
        char value[] = new char[count];

        if (hibyte == 0) {
            for (int i = count; i-- > 0;) {
                value[i] = (char)(ascii[i + offset] & 0xff);
            }
        } else {
            hibyte <<= 8;
            for (int i = count; i-- > 0;) {
                value[i] = (char)(hibyte | (ascii[i + offset] & 0xff));
            }
        }
        this.value = value;
    }
```

## public String(byte ascii[], int hibyte)

源码:

```java
    @Deprecated(since="1.1")
    public String(byte ascii[], int hibyte) {
        this(ascii, hibyte, 0, ascii.length);
    }
```

## public String(byte bytes[], int offset, int length, String charsetName)

参数:

​	bytes	存储用数组

​	offset	偏移量

​	length	字符长度

​	charsettName 字符集

源码:

jdk11:

```java
    public String(byte bytes[], int offset, int length, String charsetName)
            throws UnsupportedEncodingException {
         //数据合法性
        if (charsetName == null)
            throw new NullPointerException("charsetName");
        checkBoundsOffCount(offset, length, bytes.length); 
        //获取信息
        StringCoding.Result ret =
            StringCoding.decode(charsetName, bytes, offset, length);  //以置顶编码生成字符串 为空则为"ISO-8859-1"
        this.value = ret.value;
        this.coder = ret.coder;
    }
```

jdk8:

```java
    public String(byte bytes[], int offset, int length, String charsetName)
            throws UnsupportedEncodingException {
        if (charsetName == null)
            throw new NullPointerException("charsetName");
        checkBounds(bytes, offset, length);
        this.value = StringCoding.decode(charsetName, bytes, offset, length);
    }
```

## public String(byte bytes[], String charsetName)

源码:

```java
public String(byte bytes[], String charsetName)
            throws UnsupportedEncodingException {
        this(bytes, 0, bytes.length, charsetName);
    }
```

## public String(byte bytes[], int offset, int length) 

源码:

```java
 public String(byte bytes[], int offset, int length) {
     	//检查参数合法性
        checkBoundsOffCount(offset, length, bytes.length);
     	//生成数据
        StringCoding.Result ret = StringCoding.decode(bytes, offset, length);
     	//存储属性
        this.value = ret.value;
        this.coder = ret.coder;
    }
```

## public String(byte[] bytes)

源码:

```java
    public String(byte[] bytes) {
        this(bytes, 0, bytes.length);
    }
```

## public String(StringBuffer buffer)

源码:

```java
    public String(StringBuffer buffer) {
        this(buffer.toString());
    }
```

## public String(StringBuilder builder)

源码:

```java
   public String(StringBuilder builder) {
        this(builder, null);
    }
```

# 方法

## private static Void rangeCheck(char[] value, int offset, int count) 

jdk11独有

参数:

​	value 存储用数组

​	offset 偏移量

​	count 字符数量

源码:

```java
 private static Void rangeCheck(char[] value, int offset, int count) {
        checkBoundsOffCount(offset, count, value.length);  //判断数据合法性  是否越界
        return null;
    }
```

## public int length()

获取字符串的长度  jdk11因为添加压缩数组的原因 需要位移(也就是除以2)因为占两个字符

源码:

jdk11:

```java
  public int length() {
        return value.length >> coder();   //coder获取是字符编码需要挪动几位  目前只有0和1
    }
```

jdk8:

```java
    public int length() {
        return value.length;
    }
```

##  public boolean isEmpty()

判断是不是空字符串

源码:

```java
  public boolean isEmpty() {
        return value.length == 0;
    }
```

## public char charAt(int index)

源码:

jdk11:

```java
public char charAt(int index) {
        if (isLatin1()) {
            return StringLatin1.charAt(value, index);
        } else {
            return StringUTF16.charAt(value, index);
        }
    }
```

jdk8:

```java
```

