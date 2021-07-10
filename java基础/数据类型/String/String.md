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

# 方法

