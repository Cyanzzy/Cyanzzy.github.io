---
title: Java IO 常见面试题总结
date: 2024-02-28 22:57:49
tags: 
  - Java
categories: 
  - Interview
password: zzy   
message: 仅管理员可见
---

# File 类

- File 类用于表示文件或目录的抽象路径名
- 它不直接处理文件的内容，而是提供一组方法，使得可以操作文件的属性、路径和目录结构
- File 类的实例可以表示磁盘上的文件或目录，也可以表示其他文件系统中的文件或目录

>  **创建 File 实例：** 

```java
File file = new File("example.txt");
```

>  **获取文件信息：** 

```java
String fileName = file.getName(); // 获取文件名
String filePath = file.getPath(); // 获取文件路径
long fileSize = file.length(); // 获取文件大小
```

>  **检查文件状态：** 

```java
boolean exists = file.exists(); // 判断文件是否存在
boolean isFile = file.isFile(); // 判断是否为文件
boolean isDirectory = file.isDirectory(); // 判断是否为目录
```

>  **操作文件：** 

````java
boolean created = file.createNewFile(); // 创建新文件
boolean deleted = file.delete(); // 删除文件
````

>  **遍历目录：** 

```java
File directory = new File("path/to/directory");
File[] files = directory.listFiles(); // 获取目录下的文件列表
```

> **文件路径分隔符：** 

 在Windows系统中为`\`，在Unix/Linux系统中为`/`。 

```java
String separator = File.separator; // 获取文件路径分隔符
```



# IO 流

Java I/O（Input/Output）流是用于处理输入和输出数据的机制。它提供一组类和方法，使 Java 能够与文件、网络、内存等进行数据交互。

> I/O 流分为输入流和输出流

输入流：数据输入到内存的过程，用于读取数据

输出流：数据输出到外部存储的过程，用于写入数据

> 根据数据的处理方式又分为字节流和字符流

- **字节流：** 字节流以字节为单位进行操作，适用于处理二进制数据 

- **字符流：** 字符流以字符为单位进行操作，适用于处理文本数据 

> Java IO 流抽象类基类

- `InputStream`/`Reader`：所有的输入流的基类，前者是字节输入流，后者是字符输入流。
- `OutputStream`/`Writer`：所有输出流的基类，前者是字节输出流，后者是字符输出流。

# 字节流

## InputStream（字节输入流）

* InputStream 是用于从输入流中读取字节的抽象类
* 它是所有输入流类的父类，定义基本的方法，包括读取字节、跳过字节、关闭流等

>  InputStream 是抽象类，不能直接实例化，但可以通过其子类来实现具体的输入流操作 

| 类                   | 说明                           |
| -------------------- | ------------------------------ |
| FileInputStream      | 从文件中读取字节数据           |
| ByteArrayInputStream | 从字节数组中读取字节数据       |
| BufferedInputStream  | 提供缓冲功能，可以提高读取效率 |

> 常用方法

| 方法                             | 说明                                                         |
| -------------------------------- | ------------------------------------------------------------ |
| read()                           | 从输入流中读取一个字节的数据。返回值是读取的字节，如果到达文件末尾，返回-1 |
| read(byte b[ ])                  | 从输入流中读取一组字节的数据，并将其存储在字节数组 `b` 中。返回值是实际读取的字节数，如果到达文件末尾，返回-1 |
| read(byte b[], int off, int len) | 从输入流中读取最多 `len` 个字节的数据，并将其存储在字节数组 `b` 中，从偏移量 `off` 处开始。返回值是实际读取的字节数，如果到达文件末尾，返回-1 |
|skip(long n)                   | 跳过输入流中的 `n` 个字节。返回值是实际跳过的字节数          |
| available()                   | 返回可以从输入流中读取而不被阻塞的字节数                     |
| close()                        | 关闭输入流，释放与之关联的资源                               |

> Java 9 新增方法 

| 方法                                   | 说明                                   |
| -------------------------------------- | -------------------------------------- |
| readAllBytes()                         | 读取输入流中的所有字节，返回字节数组   |
| readNBytes(byte[] b, int off, int len) | 阻塞直到读取 `len` 个字节              |
| transferTo(OutputStream out)           | 将所有字节从一个输入流传递到一个输出流 |

>  通过`FileInputStream`打开文件 "example.txt" 并读取其中的字节数据
>
>  为确保资源被正确关闭，使用 try-with-resources 语句 

```java
try (FileInputStream inputStream = new FileInputStream("example.txt")) {
    int byteRead;
    while ((byteRead = inputStream.read()) != -1) {
        System.out.print((char) byteRead);
    }
} catch (IOException e) {
    e.printStackTrace();
}
```

## OutputStream（字节输出流）

- OutputStream 是用于向输出流中写入字节的抽象类
- 它是所有输出流类的父类，定义基本的方法，包括写入字节、刷新输出流、关闭流等

>OutputStream 是抽象类，不能直接实例化，但可以通过其子类来实现具体的输出流操作

| 类                    | 说明                           |
| --------------------- | ------------------------------ |
| FileOutputStream      | 向文件写入字节数据             |
| ByteArrayOutputStream | 向字节数组写入字节数据         |
| BufferedOutputStream  | 提供缓冲功能，可以提高写入效率 |

> 常见方法

| 方法                              | 说明                                                         |
| --------------------------------- | ------------------------------------------------------------ |
| write(int b)                      | 向输出流中写入一个字节的数据                                 |
| write(byte b[ ])                  | 向输出流中写入字节数组 `b` 的数据                            |
| write(byte[] b, int off, int len) | 向输出流中写入字节数组 `b` 的数据，从偏移量 `off` 处开始，写入长度为 `len` 的数据 |
| flush()                           | 刷新输出流，将缓冲区中的数据写入到目的地                     |
| close()                           | 关闭输出流，释放与之关联的资源                               |

>  通过 `FileOutputStream` 创建文件 "output.txt" 并向其中写入字节数据
>
>  为确保资源被正确关闭，使用 try-with-resources 语句

```java
try (FileOutputStream outputStream = new FileOutputStream("output.txt")) {
    String data = "Hello, Java I/O!";
    byte[] bytes = data.getBytes();
    outputStream.write(bytes);
} catch (IOException e) {
    e.printStackTrace();
}
```



# 字符流

>  不管是文件读写还是网络发送接收，信息的最小存储单元都是字节
>
>  那为什么 I/O 流操作要分为字节流操作和字符流操作？ 

字符流和字节流的区分主要是为方便处理文本数据和二进制数据

**字符编码：** 

- 文本数据在计算机内部以字节形式存储，而不同的字符集和编码方式通过不同的字节序列表示相同的字符。
- **字符流提供字符编码的支持**，使得在读写文本数据时可以方便地进行字符集的转换。

**缓冲处理：** 

- **字符流通常会提供缓冲功能**，允许程序以较大块的数据进行读写，以提高性能
- 而字节流通常没有这种缓冲处理，每次只能读写一个字节。

> 常见字符编码

字符流默认采用的是 `Unicode` 编码

| 字符编码 | 占用空间                     |
| -------- | ---------------------------- |
| utf8     | 英文占 1 字节，中文占 3 字节 |
| unicode  | 任何字符都占 2 个字节        |
| gbk      | 英文占 1 字节，中文占 2 字节 |



## Reader（字符输入流）

- Reader 是用于从字符输入流中读取字符的抽象类 

- 它是所有字符输入流类的父类，定义基本的方法，包括读取字符、跳过字符、关闭流等

  

> 常用方法

| 方法                                | 说明                                                         |
| ----------------------------------- | ------------------------------------------------------------ |
| read()                              | 从输入流中读取一个字符的数据。返回值是读取的字符，如果到达流的末尾，返回-1 |
| read(char[] cbuf)                   | 从输入流中读取一组字符的数据，并将其存储在字符数组 `cbuf` 中。返回值是实际读取的字符数，如果到达流的末尾，返回-1 |
| read(char[] cbuf, int off, int len) | 从输入流中读取最多 `len` 个字符的数据，并将其存储在字符数组 `cbuf` 中，从偏移量 `off` 处开始。返回值是实际读取的字符数，如果到达流的末尾，返回-1 |
| skip(long n)                        | 跳过输入流中的 `n` 个字符。返回值是实际跳过的字符数。        |
| close()                             | 关闭输入流，释放与之关联的资源。                             |

> Reader 是抽象类，不能直接实例化，但可以通过其子类来实现具体的字符输入流操作

| 类             | 说明                           |
| -------------- | ------------------------------ |
| FileReader     | 从文件中读取字符数据           |
| StringReader   | 从字符串中读取字符数据         |
| BufferedReader | 提供缓冲功能，可以提高读取效率 |

>  通过`FileReader`打开文件 "example.txt" 并读取其中的字符数据
>
> 为确保资源被正确关闭，使用 try-with-resources 语句 

````java
try (FileReader reader = new FileReader("example.txt")) {
    int charRead;
    while ((charRead = reader.read()) != -1) {
        System.out.print((char) charRead);
    }
} catch (IOException e) {
    e.printStackTrace();
}
````

>  `InputStreamReader` 是字节流转换为字符流的桥梁，其子类 `FileReader` 是基于该基础上的封装，可以直接操作字符文件。 

````java
// 字节流转换为字符流的桥梁
public class InputStreamReader extends Reader {
}
// 用于读取字符文件
public class FileReader extends InputStreamReader {
}
````



## Writer（字符输出流）

- 用于向字符输出流中写入字符的抽象类
- 它是所有字符输出流类的父类，定义基本的方法，包括写入字符、刷新输出流、关闭流等

> 常用方法

| 方法                                 | 说明                                                         |
| ------------------------------------ | ------------------------------------------------------------ |
| write(int c)                         | 向输出流中写入一个字符的数据                                 |
| write(char[] cbuf)                   | 向输出流中写入字符数组 `cbuf` 的数据                         |
| write(char[] cbuf, int off, int len) | 向输出流中写入字符数组 `cbuf` 的数据，从偏移量 `off` 处开始，写入长度为 `len` 的数据 |
| write(String str)                    | 向输出流中写入字符串 `str` 的数据                            |
| write(String str, int off, int len)  | 向输出流中写入字符串 `str` 的数据，从偏移量 `off` 处开始，写入长度为 `len` 的数据 |
| append(CharSequence cq)              | 将指定的字符序列附加到指定的 `Writer` 对象并返回该 `Writer` 对象 |
| append(char c)                       | 将指定的字符附加到指定的 `Writer` 对象并返回该 `Writer` 对象 |
| flush()                              | 刷新输出流，将缓冲区中的数据写入到目的地                     |
| close()                              | 关闭输出流，释放与之关联的资源。                             |

> Writer 是抽象类，不能直接实例化，但可以通过其子类来实现具体的字符输出流操作

| 类             | 说明                           |
| -------------- | ------------------------------ |
| FileWriter     | 向文件写入字符数据             |
| StringWriter   | 向字符串写入字符数据           |
| BufferedWriter | 提供缓冲功能，可以提高写入效率 |



>  `OutputStreamWriter` 是字符流转换为字节流的桥梁，其子类 `FileWriter` 是基于该基础上的封装，可以直接将字符写入到文件 

```java
// 字符流转换为字节流的桥梁
public class OutputStreamWriter extends Writer {
}
// 用于写入字符到文件
public class FileWriter extends OutputStreamWriter {
}
```

>  通过`FileWriter`创建文件 "output.txt" 并向其中写入字符数据
>
> 为确保资源被正确关闭，使用 try-with-resource 语句

```java
try (FileWriter writer = new FileWriter("output.txt")) {
    String data = "Hello, Java I/O!";
    writer.write(data);
} catch (IOException e) {
    e.printStackTrace();
}
```



# 字节缓冲流

字节缓冲流是将数据从 **硬盘读/写到缓冲区** 中，待缓冲区满了再进行读/写，不用每次都到硬盘中操作，提高了读写的效率。字节缓冲流 **仅仅提供缓冲区**，而真正读写数据还得依靠基本的字节流对象进行操作 

## BufferedInputStream（字节缓冲输入流）

- BufferedInputStream 提供缓冲功能，可以提高从输入流中读取数据的效率
- BufferedInputStream 继承自 FilterInputStream 类，它装饰了其他的输入流，向其添加了缓冲的功能
- BufferedInputStream 的主要特点是，它使用**内部缓冲区**来减少对底层输入流的直接访问次数，从而提高读取效率
- 通过读取一块块的数据到缓冲区，减少对底层资源（如文件或网络）的频繁访问

> BufferedInputStream 内部维护一个缓冲区（字节数组）

BufferedInputStream 不会一个字节一个字节的读取，而是会先将读取到的字节存放在缓存区，并从内部缓冲区中单独读取字节，大幅减少 IO 次数，提高读取效率

```java
public
class BufferedInputStream extends FilterInputStream {
    // 内部缓冲区数组
    protected volatile byte buf[];
    // 缓冲区的默认大小 8192B
    private static int DEFAULT_BUFFER_SIZE = 8192;
    // 使用默认的缓冲区大小
    public BufferedInputStream(InputStream in) {
        this(in, DEFAULT_BUFFER_SIZE);
    }
    // 自定义缓冲区大小
    public BufferedInputStream(InputStream in, int size) {
        super(in);
        if (size <= 0) {
            throw new IllegalArgumentException("Buffer size <= 0");
        }
        buf = new byte[size];
    }
}
```

> 构造函数

```java
BufferedInputStream(InputStream in)
BufferedInputStream(InputStream in, int size)
```

> 常用方法

| 方法                             | 说明                                                         |
| -------------------------------- | ------------------------------------------------------------ |
| read()                           | 从输入流中读取一个字节的数据。返回值是读取的字节，如果到达流的末尾，返回-1 |
| read(byte[] b, int off, int len) | 从输入流中读取最多 `len` 个字节的数据，并将其存储在字节数组 `b` 中，从偏移量 `off` 处开始。返回值是实际读取的字节数，如果到达流的末尾，返回-1 |
| skip(long n)                     | 跳过输入流中的 `n` 个字节。返回值是实际跳过的字节数          |
| mark(int readlimit)              | 在当前位置设置标记，最多可以跳过 `readlimit` 个字节          |
| reset()                          | 将输入流的位置重置到之前设置的标记位置                       |
| markSupported()                  | 判断输入流是否支持标记和重置                                 |
| close()                          | 关闭输入流，释放与之关联的资源                               |

>  通过`BufferedInputStream`包装了`FileInputStream`，实现对文件的缓冲读取
>
> 为确保资源被正确关闭，使用 try-with-resources 语句 

```java
try (BufferedInputStream bis = new BufferedInputStream(new FileInputStream("example.txt"))) {
    int byteRead;
    while ((byteRead = bis.read()) != -1) {
        System.out.print((char) byteRead);
    }
} catch (IOException e) {
    e.printStackTrace();
}
```



## BufferedOutputStream（字节缓冲输出流）

- BufferedOutputStream 提供缓冲功能，可以提高向输出流中写入数据的效率
- BufferedOutputStream 继承自 FilterOutputStream 类，它装饰了其他的输出流，向其添加了缓冲的功能
- BufferedOutputStream 的主要特点是，它使用内部缓冲区来减少对底层输出流的直接访问次数，从而提高写入效率
- 通过将数据写入一块块的缓冲区，减少对底层资源（如文件或网络）的频繁写入

> BufferedOutputStream 不会一个字节一个字节的写入，而是会先将要写入的字节存放在缓存区，并从内部缓冲区中单独写入字节，大幅减少 IO 次数，提高读取效率

```java
public
class BufferedOutputStream extends FilterOutputStream {
    
    protected byte buf[];
    protected int count;

    public BufferedOutputStream(OutputStream out) {
        this(out, 8192);
    }

    public BufferedOutputStream(OutputStream out, int size) {
        super(out);
        if (size <= 0) {
            throw new IllegalArgumentException("Buffer size <= 0");
        }
        buf = new byte[size];
    }
}
```

>  **构造函数：** 

```java
BufferedOutputStream(OutputStream out)
BufferedOutputStream(OutputStream out, int size)
```

> 常用方法

| 方法                              | 说明                                                         |
| --------------------------------- | ------------------------------------------------------------ |
| write(int b)                      | 向输出流中写入一个字节的数据                                 |
| write(byte[] b, int off, int len) | 向输出流中写入字节数组 `b` 的数据，从偏移量 `off` 处开始，写入长度为 `len` 的数据 |
| flush()                           | 刷新输出流，将缓冲区中的数据写入到目的地                     |
| close()                           | 关闭输出流，释放与之关联的资源                               |

>  通过 BufferedOutputStream 包装了`FileOutputStream`，实现了对文件的缓冲写入
>
> 为确保资源被正确关闭，使用try-with-resources 语句

```java
try (BufferedOutputStream bos = new BufferedOutputStream(new FileOutputStream("output.txt"))) {
    String data = "Hello, Java I/O!";
    byte[] bytes = data.getBytes();
    bos.write(bytes);
} catch (IOException e) {
    e.printStackTrace();
}
```



# 字符缓冲流

## BufferedReader（字符缓冲输入流）

- BufferedReader 它提供缓冲功能，可以提高从字符输入流中读取数据的效率
- BufferedReader 继承自`Reader`类，它装饰了其他的字符输入流，向其添加了缓冲的功能
- BufferedReader 的主要特点是，它使用内部缓冲区来减少对底层输入流的直接访问次数，从而提高读取效率
- 通过读取一块块的数据到缓冲区，减少对底层资源（如文件或网络）的频繁访问。

> BufferedReader 内部都维护一个字节数组作为缓冲区

```java
public class BufferedReader extends Reader {

    private Reader in;

    private char cb[];
    private int nChars, nextChar;

    private static final int INVALIDATED = -2;
    private static final int UNMARKED = -1;
    private int markedChar = UNMARKED;
    private int readAheadLimit = 0; 

    private boolean skipLF = false;

    private boolean markedSkipLF = false;

    private static int defaultCharBufferSize = 8192;
    private static int defaultExpectedLineLength = 80;

    public BufferedReader(Reader in, int sz) {
        super(in);
        if (sz <= 0)
            throw new IllegalArgumentException("Buffer size <= 0");
        this.in = in;
        cb = new char[sz];
        nextChar = nChars = 0;
    }

    public BufferedReader(Reader in) {
        this(in, defaultCharBufferSize);
    }

}
```

>  **构造函数：** 

```java
BufferedReader(Reader in)
BufferedReader(Reader in, int size)
```

> 常用方法

| 方法                                | 说明                                                         |
| ----------------------------------- | ------------------------------------------------------------ |
| readLine()                          | 从输入流中读取一行数据，并返回字符串。如果到达流的末尾，返回null |
| read()                              | 从输入流中读取一个字符的数据。返回值是读取的字符，如果到达流的末尾，返回-1 |
| read(char[] cbuf, int off, int len) | 从输入流中读取最多 `len` 个字符的数据，并将其存储在字符数组 `cbuf` 中，从偏移量 `off` 处开始。返回值是实际读取的字符数，如果到达流的末尾，返回-1 |
| skip(long n)                        | 跳过输入流中的 `n` 个字符。返回值是实际跳过的字符数          |
| mark(int readAheadLimit)            | 在当前位置设置标记，最多可以跳过 `readAheadLimit` 个字符     |
| reset()                             | 将输入流的位置重置到之前设置的标记位置                       |
| markSupported()                     | 判断输入流是否支持标记和重置                                 |
| close()                             | 关闭输入流，释放与之关联的资源                               |

>  通过`BufferedReader`包装了`FileReader`，实现对文件的缓冲读取
>
> 为确保资源被正确关闭，使用 try-with-resources 语句

```java
try (BufferedReader reader = new BufferedReader(new FileReader("example.txt"))) {
    String line;
    while ((line = reader.readLine()) != null) {
        System.out.println(line);
    }
} catch (IOException e) {
    e.printStackTrace();
}
```



## BufferedWriter（字符缓冲输出流）

- BufferedWriter 继承自 Writer，它装饰了其他的字符输出流，向其添加了缓冲的功能
- BufferedWriter 主要作用是提供一个缓冲区，从而减少对底层字符输出流的直接访问次数，提高写入效率

> BufferedWriter 内部都维护了一个字节数组作为缓冲区

```java
public class BufferedWriter extends Writer {

    private Writer out;

    private char cb[];
    private int nChars, nextChar;

    private static int defaultCharBufferSize = 8192;

    private String lineSeparator;

    public BufferedWriter(Writer out) {
        this(out, defaultCharBufferSize);
    }

    public BufferedWriter(Writer out, int sz) {
        super(out);
        if (sz <= 0)
            throw new IllegalArgumentException("Buffer size <= 0");
        this.out = out;
        cb = new char[sz];
        nChars = sz;
        nextChar = 0;

        lineSeparator = java.security.AccessController.doPrivileged(
            new sun.security.action.GetPropertyAction("line.separator"));
    }
}
```

>  **构造函数：** 

```java
BufferedWriter(Writer out)
BufferedWriter(Writer out, int size)
```

| 方法                                 | 说明                                                         |
| ------------------------------------ | ------------------------------------------------------------ |
| write(int c)                         | 向输出流中写入一个字符的数据                                 |
| write(String str)                    | 向输出流中写入字符串 `str` 的数据                            |
| write(char[] cbuf, int off, int len) | 向输出流中写入字符数组 `cbuf` 的数据，从偏移量 `off` 处开始，写入长度为 `len` 的数据 |
| newLine()                            | 写入一个行分隔符。行分隔符的具体表示方式取决于平台           |
| flush()                              | 刷新输出流，将缓冲区中的数据写入到目的地                     |
| close()                              | 关闭输出流，释放与之关联的资源                               |

> 通过`BufferedReader`和`BufferedWriter`实现了对文件的缓冲读写
>
> 为确保资源被正确关闭，使用 try-with-resources 语句

```java
// 读取文件并写入新文件
try (BufferedReader reader = new BufferedReader(new FileReader("input.txt"));
     BufferedWriter writer = new BufferedWriter(new FileWriter("output.txt"))) {

    String line;
    while ((line = reader.readLine()) != null) {
        writer.write(line);
        writer.newLine(); // 写入行分隔符
    }
} catch (IOException e) {
    e.printStackTrace();
}
```

# 对象流

## ObjectInputStream（对象输入流）

- ObjectInputStream 用于从输入流中读取字节流并反序列化成对象
- 它继承自 InputStream 类，并提供方法用于读取对象

>  **构造函数：** 

```java
ObjectInputStream(InputStream in)
```

> 常用方法

| 方法                             | 说明                                                         |
| -------------------------------- | ------------------------------------------------------------ |
| readObject()                     | 从输入流中读取一个对象，并返回该对象。需要注意的是，读取的对象类型必须与写入时的类型一致，否则可能会抛出`ClassNotFoundException` |
| read()                           | 从输入流中读取一个字节的数据。返回值是读取的字节，如果到达流的末尾，返回-1 |
| read(byte[] b, int off, int len) | 从输入流中读取最多 `len` 个字节的数据，并将其存储在字节数组 `b` 中，从偏移量 `off` 处开始。返回值是实际读取的字节数，如果到达流的末尾，返回-1 |
| close()                          | 关闭输入流，释放与之关联的资源                               |

>  通过`ObjectInputStream`从文件 "person.ser" 中读取序列化的`Person`对象，并在控制台上打印出反序列化后的对象信息
>
> 读取的对象类型必须与写入时的类型一致

```java
try (ObjectInputStream ois = new ObjectInputStream(new FileInputStream("person.ser"))) {
    // 从文件中读取对象
    Person person = (Person) ois.readObject();
    System.out.println("Object has been deserialized");
    System.out.println("Deserialized Person: " + person);
} catch (IOException | ClassNotFoundException e) {
    e.printStackTrace();
}
```

## ObjectOutputStream（对象输出流）

- ObjectOutputStream 用于将对象序列化成字节流并写入输出流
- 它继承自 OutputStream 类，并提供方法用于写入对象。

>  **构造函数：** 

```java
ObjectOutputStream(OutputStream out)
```

> 常用方法

| 方法                              | 说明                                                         |
| --------------------------------- | ------------------------------------------------------------ |
| writeObject(Object obj)           | 将对象 `obj` 序列化并写入输出流。被序列化的对象必须实现`Serializable`接口 |
| write(int b)                      | 向输出流中写入一个字节的数据。                               |
| write(byte[] b, int off, int len) | 向输出流中写入字节数组 `b` 的数据，从偏移量 `off` 处开始，写入长度为 `len` 的数据 |
| flush()                           | 刷新输出流，将缓冲区中的数据写入到目的地                     |
| close()                           | 关闭输出流，释放与之关联的资源                               |

> 通过`ObjectOutputStream`将`Person`对象序列化并写入文件 "person.ser"
>
>被序列化的对象必须实现`Serializable`接口

```java
class Person implements Serializable {
    private String name;
    private int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public String toString() {
        return "Person{name='" + name + "', age=" + age + '}';
    }
}

public class ObjectOutputStreamExample {
    public static void main(String[] args) {
        try (ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("person.ser"))) {
            // 创建一个序列化的对象
            Person person = new Person("John", 30);
            // 将对象序列化并写入文件
            oos.writeObject(person);
            System.out.println("Object has been serialized");
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

## 注意事项

> Serializable 接口

对象需要序列化必须实现 **Serializable** 接口，该接口是标记接口，没有任何抽象方法，只要是实现该接口的类，它的对象就能序列化

> 序列化后修改对象怎么办

用对象序列化流序列化一个对象后，如果修改对象所属的类，再次读取该对象时会抛出**InvalidClassException** 异常

* 因为 **serialVersionUID** 是由 **虚拟机自动生成**，写入文件时与读取文件时的serialVersionUID不一致造成的

* 解决办法是在序列化的类中手动给出serialVersionUID，并且这个值固定不变，格式是：

  `private static final long serialVersionUID = long值;`

> 禁止序列化

如果类中某个成员变量的值不想要被序列化，可以给该成员变量加上 **transient** 关键字修饰，表示该成员变量不参与序列化过程

读取多个对象时难以分辨循环中是否读取到文件末尾，所以在存对象时可以将对象先存放在 **集合** 中再写入文件，这样读取对象时可以只使用一次读操作将该集合读取出来

# 转化流

Java 转换流是一种特殊的字符流，用于在字节流和字符流之间进行转换

##  InputStreamReader

- InputStreamReader 用于将字节输入流转换为字符输入流（硬盘-->内存）
- 它继承自`Reader`类，可以指定字符集编码，将字节流按照指定的编码转换为字符流 

> 构造方法

```java
// 使用系统默认字符集将字节输入流转换为字符输入流
InputStreamReader(InputStream in);
// 使用指定的字符集将字节输入流转换为字符输入流。
// 其中，charsetName参数是字符集的名称，例如："UTF-8"、"GBK"等。
InputStreamReader(InputStream in, String charsetName);
```

> 通过`FileInputStream`读取字节输入流，然后通过`InputStreamReader`将其转换为字符输入流
>
>`UTF-8`是指定的字符集编码，确保正确地将字节转换为字符
>
>为确保资源被正确关闭，使用 try-with-resources 语句 

 ```java
try (FileInputStream fis = new FileInputStream("example.txt");
     InputStreamReader isr = new InputStreamReader(fis, "UTF-8")) {
    int charRead;
    while ((charRead = isr.read()) != -1) {
        System.out.print((char) charRead);
    }
} catch (IOException e) {
    e.printStackTrace();
}
 ```

## OutputStreamWriter

- OutputStreamWriter 用于将字符输出流转换为字节输出流（内存-->硬盘）
- 它继承自`Writer`类，可以指定字符集编码，将字符流按照指定的编码转换为字节流

> 构造方法

```java
// 使用系统默认字符集将字符输出流转换为字节输出流
OutputStreamWriter(OutputStream out);
// 使用指定的字符集将字符输出流转换为字节输出流
// 其中，charsetName参数是字符集的名称，例如："UTF-8"、"GBK"等。
OutputStreamWriter(OutputStream out, String charsetName);	
```

> 通过`FileOutputStream`创建字节输出流，然后通过`OutputStreamWriter`将其转换为字符输出流
>
>`UTF-8`是指定的字符集编码，确保正确地将字符转换为字节
>
>为确保资源被正确关闭，使用 try-with-resources 语句

```java
try (FileOutputStream fos = new FileOutputStream("output.txt");
     Writer writer = new OutputStreamWriter(fos, "UTF-8")) {
    String data = "Hello, Java I/O!";
    writer.write(data);
} catch (IOException e) {
    e.printStackTrace();
}
```



# DataInputStream

- DataInputStream 用于以机器无关的方式从输入流中读取基本Java数据类型的值
- 它继承自`FilterInputStream`类，提供方法用于读取各种基本数据类型的值

> 构造方法

```java
DataInputStream(InputStream in)
```

> 常用方法

| 方法                             | 说明                                                         |
| -------------------------------- | ------------------------------------------------------------ |
| read()                           | 读取输入流中的下一个字节                                     |
| read(byte[] b)                   | 从输入流中读取字节并将其存储在字节数组 `b` 中                |
| read(byte[] b, int off, int len) | 从输入流中读取最多 `len` 个字节的数据，并将其存储在字节数组 `b` 中，从偏移量 `off` 处开始。返回值是实际读取的字节数，如果到达流的末尾，返回-1 |
| readBoolean()                    | 从输入流中读取一个布尔值                                     |
| readByte()                       | 从输入流中读取一个字节                                       |
| readShort()                      | 从输入流中读取一个短整数                                     |
| readChar()                       | 从输入流中读取一个字符                                       |
| readInt()                        | 从输入流中读取一个整数                                       |
| readLong()                       | 从输入流中读取一个长整数                                     |
| readFloat()                      | 从输入流中读取一个浮点数                                     |
| readDouble()                     | 从输入流中读取一个双精度浮点数                               |
| readUTF()                        | 从输入流中读取一个UTF-8编码的字符串                          |
| close()                          | 关闭输入流，释放与之关联的资源                               |

>  通过`DataInputStream`从文件 "data.dat" 中读取整数、双精度浮点数和字符串
>
> 为确保资源被正确关闭，使用 try-with-resources 语句

```java
try (DataInputStream dis = new DataInputStream(new FileInputStream("data.dat"))) {
    int intValue = dis.readInt();
    double doubleValue = dis.readDouble();
    String stringValue = dis.readUTF();

    System.out.println("Read values: " + intValue + ", " + doubleValue + ", " + stringValue);
} catch (IOException e) {
    e.printStackTrace();
}
```



# DataOutputStream

- DataOutputStream 它允许应用程序以机器无关的方式将基本Java数据类型写入输出流
- 它继承自`FilterOutputStream`类，用于写入各种基本数据类型的值 

> 构造方法

```java
DataOutputStream(OutputStream out)
```

> 常用方法

| 方法                              | 说明                                                     |
| --------------------------------- | -------------------------------------------------------- |
| write(int b)                      | 将指定的字节写入输出流                                   |
| write(byte[] b)                   | 将字节数组中的数据写入输出流                             |
| write(byte[] b, int off, int len) | 将字节数组中从偏移量 `off` 开始的 `len` 个字节写入输出流 |
| writeBoolean(boolean v)           | 将一个布尔值写入输出流                                   |
| writeByte(int v)                  | 将一个字节值写入输出流                                   |
| writeShort(int v)                 | 将一个短整数值写入输出流                                 |
| writeChar(int v)                  | 将一个字符值写入输出流                                   |
| writeInt(int v)                   | 将一个整数值写入输出流                                   |
| writeLong(long v)                 | 将一个长整数值写入输出流                                 |
| writeFloat(float v)               | 将一个浮点数值写入输出流                                 |
| writeDouble(double v)             | 将一个双精度浮点数值写入输出流                           |
| writeUTF(String str)              | 将一个UTF-8编码的字符串写入输出流                        |
| close()                           | 关闭输出流，释放与之关联的资源                           |

> 通过`DataOutputStream`将整数、双精度浮点数和字符串写入文件 "data.dat"
>
> 为确保资源被正确关闭，使用 try-with-resources 语句

```java
try (DataOutputStream dos = new DataOutputStream(new FileOutputStream("data.dat"))) {
    dos.writeInt(42);
    dos.writeDouble(3.14);
    dos.writeUTF("Hello, DataOutputStream!");
} catch (IOException e) {
    e.printStackTrace();
}
```

# 打印流

## PrintStream

- PrintStream 是 Java I/O 中的一个字节打印流，用于将数据打印到输出流
- 它是 `OutputStream` 的子类，提供了一系列的 `print` 和 `println` 方法，用于打印各种数据类型的值

```java
public class PrintStream extends FilterOutputStream
    implements Appendable, Closeable {
}
```

> 构造函数

```java
PrintStream(OutputStream out)
PrintStream(OutputStream out, boolean autoFlush)
PrintStream(OutputStream out, boolean autoFlush, String encoding)
```

>  通过 `PrintStream` 向文件 "output.txt" 中输出整数、双精度浮点数和字符串
>
> 为确保资源被正确关闭，使用 try-with-resources 语句

```java
try (PrintStream ps = new PrintStream(new FileOutputStream("output.txt"))) {
    int intValue = 42;
    double doubleValue = 3.14;
    String stringValue = "Hello, PrintStream!";

    // 使用 print 方法
    ps.print("Value: ");
    ps.print(intValue);
    ps.print(", ");
    ps.print(doubleValue);
    ps.print(", ");
    ps.println(stringValue); // 使用 println 方法自动换行

} catch (Exception e) {
    e.printStackTrace();
}
```

```java
// System.out.println();
public final class System {
    ...
	public final static PrintStream out = null;
```



## PrintWriter

- PrintWriter  是 Java I/O 中的一个字符打印流，用于方便地将各种数据类型打印到输出流中
- 它是 `Writer` 类的子类，通常用于处理字符数据

```java
public class PrintWriter extends Writer {
}
```

> 构造方法

```java
PrintWriter(Writer out)
```

>  过 `PrintWriter` 将整数、双精度浮点数和字符串打印到文件 "output.txt" 中
>
> 为确保资源被正确关闭，使用 try-with-resources 语句。 

```java
try (PrintWriter pw = new PrintWriter(new FileWriter("output.txt"))) {
    int intValue = 42;
    double doubleValue = 3.14;
    String stringValue = "Hello, PrintWriter!";

    // 使用 print 方法
    pw.print("Value: ");
    pw.print(intValue);
    pw.print(", ");
    pw.print(doubleValue);
    pw.print(", ");
    pw.println(stringValue); // 使用 println 方法自动换行

} catch (Exception e) {
    e.printStackTrace();
}
```



#  随机访问流

- 随机访问流（RandomAccessFile）是 Java I/O 中的一种特殊的文件操作流，它可以直接跳到文件的任意位置进行读写操作，而不需要按顺序读写
- RandomAccessFile 既可以读取文件的内容，也可以向文件中写入数据，提供了对文件的随机访问能力 

> 构造方法

```java
// openAndDelete 参数默认为 false 表示打开文件并且这个文件不会被删除
public RandomAccessFile(File file, String mode) // 可以指定 `mode`（读写模式）。 
    throws FileNotFoundException {
    this(file, mode, false);
}
// 私有方法
private RandomAccessFile(File file, String mode, boolean openAndDelete)  throws FileNotFoundException{
  // 省略大部分代码
}
```

| 模式  | 说明                       |
| ----- | -------------------------- |
| `r`   | 只读模式                   |
| `rw`  | 读写模式                   |
| `rws` | 相读写并同步文件内容       |
| `rwd` | 读写并同步文件内容和元数据 |

- 文件内容指的是文件中实际保存的数据
- 元数据则是用来描述文件属性比如文件的大小信息、创建和修改时间

> 常用方法

```java
long length() // 返回文件的长度，即文件中的字节数。

void seek(long pos) // 将文件指针移动到指定位置。

int read() // 从文件中读取一个字节，返回读取的字节数据。

int read(byte[] b) // 从文件中读取最多 b.length 个字节的数据，并将其存储在字节数组 b 中。返回实际读取的字节数，如果到达文件末尾，返回 -1。

void write(int b) // 将指定的字节写入文件。

void write(byte[] b) // 将字节数组中的数据写入文件。

void write(byte[] b, int off, int len) // 将字节数组中从偏移量 off 开始的 len 个字节写入文件。

void close() // 关闭随机访问文件，释放资源。
```

- RandomAccessFile 中有一个文件指针用来表示下一个将要被写入或者读取的字节所处的位置
- 可以通过 RandomAccessFile 的 seek(long pos) 方法来设置文件指针的偏移量（距文件开头 pos 个字节处）
- 如果想要获取文件指针当前的位置的话，可以使用 getFilePointer() 方法

> RandomAccessFile  的 `write` 方法在写入对象的时候如果对应的位置已经有数据时会将其覆盖掉：

```java
RandomAccessFile randomAccessFile = new RandomAccessFile(new File("input.txt"), "rw");
randomAccessFile.write(new byte[]{'H', 'I', 'J', 'K'});
```

  

