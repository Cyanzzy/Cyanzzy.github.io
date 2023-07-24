---
title: JavaBasic-9-IO
date: 2023-03-16 19:14:30
tags: 
  - Java
categories: 
  - Language
swiper_index: 
---
# File类

* 文件和目录可以通过File封装成对象

* **Flie封装的仅仅是一个路径名**，可以是存在的也可以是不存在的

* 构造方法

  | 方法名                           | 说明                                           |
  | -------------------------------- | ---------------------------------------------- |
  | File(String pathname)            | 通过路径名字符串创建一个File对象               |
  | File(String parent,String child) | 通过父路径和子路径拼接创建一个File对象         |
  | File(File parent,String child)   | 通过父路径File对象和子路径拼接创建一个File对象 |

* 相对路径和绝对路径

  * 绝对路径是从**盘符**开始给出文件的路径
  * 相对路径是相对当前项目下的文件路径

* 常用方法

  | 方法名                         | 说明                                                         |
  | ------------------------------ | ------------------------------------------------------------ |
  | public boolean createNewFile() | 创建一个文件，如果该文件已存在返回false，如果不存在则创建该文件并且返回true，注意该文件所在文件夹必须存在 |
  | public boolean mkdir()         | 创建一个单级文件夹，如果文件夹已存在则返回false，如果不存在则创建文件夹并且返回true |
  | public boolean mkdirs()        | 创建一个单级或多级文件夹，如果文件夹已存在则返回false，如果不存在则创建文件夹并且返回true（常用，既能创建单级也能创建多级） |
  | public boolean delete()        | 删除一个文件或者一个空文件夹，不能删除含有文件的文件夹       |
  | public boolean isDirectory()   | 判断该路径表示的File对象是不是目录                           |
  | public boolean isFile()        | 判断该路径表示的File对象是不是文件                           |
  | public boolean exists()        | 判断该路径表示的File对象是否存在                             |
  | public String getName()        | 返回该路径表示的File对象的文件名（含后缀）或者文件夹名       |

* 特殊方法

  | 方法名                    | 说明                                           |
  | ------------------------- | ---------------------------------------------- |
  | public File[] listFiles() | 返回一个File数组，里面装有该路径下的文件和目录 |

  * 当调用者不存在时，返回null
  * 当调用者是一个文件时，返回null
  * 当调用者是一个空文件夹时，返回长度为0的数组
  * 当调用者是一个非空的文件夹时，返回一个装有该目录下所有的文件（含隐藏文件）和文件夹的File数组
  * 当调用者是一个需要权限的文件夹时，返回null

* 如何删除含有文件和文件夹的文件夹？

  ```java
  public static void deleteDir(File src){
      // 先删除文件夹里面的内容，再删除文件夹
      // 使用递归思路
      File[] files = src.listFiles();
      for(File file:files){
          if(file.isFile()){
              // 如果是文件则删除
              file.delete();
          }else{
              // 如果是文件夹则递归删除
              deleteDir(file);
          }
      }
      src.delete();// 最后删除文件夹
  }
  ```

# IO流的分类

* 按流向分

  *  输⼊流：数据从数据源(⽂件)到程序(内存)的路径 
  *  输出流：数据从程序(内存)到数据源(⽂件)的路径 

{% mermaid %}
  graph TB;
  IO流-->输入流
  IO流-->输出流
{% endmermaid %}


* 按数据类型分

{% mermaid %}
 graph TB;
  IO流-->字节流
  IO流-->字符流
  字节流-->操作所有类型的文件
  字符流-->只能操作纯文本文件
{% endmermaid %}


# 字节流

> 字节输出流**FileOutputStream**

* **构造方法**

| 方法名                                                  | 说明                             |
| ------------------------------------------------------- | -------------------------------- |
| public FileOutputStream(String finepath)                | 根据文件路径创建字节输出流对象   |
| public FileOutputStream(File file)                      | 根据文件对象创建字节输出流对象   |
| public FileOutputStream(String filepath,boolean append) | append指定是否允许在文件追加数据 |
| public FileOutputStream(File file,boolean append)       | append指定是否允许在文件追加数据 |

* **写数据**

  | 方法名                               | 说明                                                   |
  | ------------------------------------ | ------------------------------------------------------ |
  | void write(int b)                    | 一次写一个字节数据，**这里的int值对应成ASCII中的字符** |
  | void write(byte[] b)                 | 一次写一个字节数组数据                                 |
  | void write(byte[] b,int off,int len) | 一次写一个字节数组某长度的数据                         |

  * 如果在写数据时需要换行，可以使用`write("\r\n".getBytes())`在windows系统中写入一个换行符，linux使用**\n**，mac使用**\r**
  * **默认情况下每次打开文件写数据时会将文件内容清空，如果需要实现追加的功能**，可以使用带有**append**参数的构造方法来创建FileOutputStream对象

* **字节输出流写文件的步骤**

  * 创建字节输出流对象

    文件不存在会自动创建，文件存在如果不使用含有append参数的构造方法则会清空文件

  * 写数据

  * 释放资源

* 异常处理，使用finally块

  ```java
  // try..catch..finally块处理异常的模板
  FileOutputStream fos = null;
  try {
      fos = new FileOutputStream("a.txt");
      fos.write(98);
  } catch (IOException e) {
      e.printStackTrace();
  } finally {
      // finally块的代码无论是否产生异常，都会执行
      if(fos != null){
          try {
              fos.close();
          } catch (IOException e) {
              e.printStackTrace();
          }
      }
  }
  ```

> 字节输入流**FileInputStream**

* **常用构造方法**

  | 方法名                                  | 说明                           |
  | --------------------------------------- | ------------------------------ |
  | public FileInputStream(String filepath) | 根据文件路径创建字节输入流对象 |
  | public FileInputStream(File file)       | 根据文件对象创建字节输入流对象 |

* **读数据**

  | 方法名                     | 说明                                                         |
  | -------------------------- | ------------------------------------------------------------ |
  | int read()                 | 从输入流中读一个字节，返回的是该字符的ASCII值                |
  | int read(byte[] b)         | 从输入流中最多将b.length个字节读取到数组中，返回的是**实际读到的字节数** |
  | int read(byte[] b,off,len) | 从输入流中最多将len个字节读取到数组中，返回的是**实际读到的字节数** |

  * 当使用读出来的数据是 **-1** 时，表明文件已经读到底部

  * 连续读取多个数据范例

    ```java
    FileInputStream fis = null;
    int b;
    try {
        fis = new FileInputStream("a.txt");
        while ((b = fis.read()) != -1) {
            System.out.print((char)b);//由于读出来的数据是字节，需要强转
        }
    } catch (IOException e) {
        e.printStackTrace();
    } finally {
        try {
            fis.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    ```

  * 文件拷贝范例1（单个字节读写）

    ```java
    FileInputStream fis = null;
    FileOutputStream fos = null;
    int b;
    try {
        fis = new FileInputStream("D:\\Code\\HTML\\project1.html");
        fos = new FileOutputStream("a.txt");
        while((b = fis.read()) != -1){
            fos.write(b);
        }
    } catch (IOException e) {
        e.printStackTrace();
    } finally {
        try {
            fis.close();
            fos.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    ```

  * 文件拷贝范例2（多个字节读写）

    ```java
    FileInputStream fis = null;
    FileOutputStream fos = null;
    int len;
    byte[] temp = new byte[1024];
    try {
        fis = new FileInputStream("D:\\Code\\HTML\\project1.html");
        fos = new FileOutputStream("a.txt");
        while((len = fis.read(temp)) != -1){
            fos.write(temp,0,len);
        }
    } catch (IOException e) {
        e.printStackTrace();
    } finally {
        try {
            fis.close();
            fos.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    ```

# 字节缓冲流

字节缓冲流是将数据从**硬盘读/写到缓冲区**中，待缓冲区满了再进行读/写，不用每次都到硬盘中操作，提高了读写的效率

字节缓冲流**仅仅提供缓冲区**，而真正读写数据还得依靠基本的字节流对象进行操作

> 字节缓冲输出流**BufferedOutputStream**

* 构造方法 

  | 方法                                                   | 说明           |
  | ------------------------------------------------------ | -------------- |
  | public BufferedOutputStream(OutputStream out)          | 默认缓冲区大小 |
  | public BufferedOutputStream(OutputStream out,int size) | 指定缓冲区大小 |

* 构造的参数是OutputStream类型的对象

> 字节缓冲输入流**BufferedInputStream**

* 构造方法

  | 方法                                                | 说明           |
  | --------------------------------------------------- | -------------- |
  | public BufferedInputStream(InputStream in)          | 默认缓冲区大小 |
  | public BufferedInputStream(InputStream in,int size) | 指定缓冲区大小 |

* 构造的参数是InputStream类型的对象

> 使用缓冲流拷贝文件（单个字节读写）

```java
BufferedInputStream bis = null;
BufferedOutputStream bos = null;
int b;
try {
    bis = new BufferedInputStream(new FileInputStream("D:\\Code\\HTML\\project1.html"));
    bos = new BufferedOutputStream(new FileOutputStream("a.txt"));
    while((b = bis.read()) != -1){
        bos.write(b);
    }
} catch (IOException e) {
    e.printStackTrace();
} finally {
    try {
        bis.close();
        bos.close();
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

> 使用缓冲流拷贝文件（多个字节读写）

```java
BufferedInputStream bis = null;
BufferedOutputStream bos = null;
int len;
byte[] b = new byte[1024];
try {
    bis = new BufferedInputStream(new FileInputStream("D:\\Code\\HTML\\project1.html"));
    bos = new BufferedOutputStream(new FileOutputStream("a.txt"));
    while((len = bis.read(b)) != -1){
        bos.write(b,0,len);
    }
} catch (IOException e) {
    e.printStackTrace();
} finally {
    try {
        bis.close();
        bos.close();
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

# 字符流

由于编码和解码的方式不同，计算机中可能会出现乱码的问题

Windows默认使用的码表是GBK，一个汉字占**两个字节**，IDEA和以后工作默认使用Unicode的UTF-8编解码格式，一个汉字占**三个字节**

> 字符串中的编码和解码

| 方法                                           | 说明                                                         |
| ---------------------------------------------- | ------------------------------------------------------------ |
| byte[] getBytes()                              | 使用平台默认字符集将该String编码为一系列字节，将结果存储到字节数组中返回 |
| byte[] getBytes(String charsetName)            | 使用指定的字符集将该String编码为一系列字节，将结果存储到字节数组中返回 |
| public String(byte[] bytes)                    | String类的一个构造方法，使用平台默认的字符集对给定的字节数组进行解码（创建字符串对象） |
| public String(byte[] bytes,String charsetName) | String类的一个构造方法，使用指定的字符集对给定的字节数组进行解码（创建字符串对象） |

> 提示

如果需要将文本文件数据**读取**到内存或者将内存数据**写入**文本文件时，建议使用字符流，而文件拷贝使用字节流

> 字符输出流**FileWriter**

* **构造方法**

  | 方法名                                            | 说明                             |
  | ------------------------------------------------- | -------------------------------- |
  | public FileWriter(String filepath)                | 通过文件路径创建字符输出流对象   |
  | public FileWriter(File file)                      | 通过文件对象创建字符输出流对象   |
  | public FileWriter(String filepath,boolean append) | append指定是否允许在文件追加数据 |
  | public FileWriter(File file,boolean append)       | append指定是否允许在文件追加数据 |

  * 字符流底层使用到了字节流

* **写数据**

  | 方法名                                  | 说明                                 |
  | --------------------------------------- | ------------------------------------ |
  | void write(int c)                       | 写一个字符，**int值是字符的ASCII值** |
  | void write(char[] cbuf)                 | 写一个字符数组                       |
  | void write(char[] cbuf,int off,int len) | 写一个字符数组的指定长度             |
  | void write(String str)                  | 写一个字符串                         |
  | void write(String str,int off,int len)  | 写一个字符串的指定长度               |

* **flush()** 方法和 **close()** 方法

  `void flush()`：刷新流，后续仍然能写数据

  `void close()`：关闭流，关闭前会刷新一次，后续不能再写数据

* 写文件

  ```java
  FileWriter fw = null;
  char[] c = new char[]{'奏','直','漾'};
  try {
      fw = new FileWriter("a.txt");
      fw.write(c);
  } catch (IOException e) {
      e.printStackTrace();
  }finally {
      try {
          fw.close();
      } catch (IOException e) {
          e.printStackTrace();
      }
  }
  ```

> 字符输入流**FileReader**

* 构造方法

  | 方法名                             | 说明                           |
  | ---------------------------------- | ------------------------------ |
  | public FileReader(String filepath) | 通过文件路径创建字符输入流对象 |
  | public FileReader(File file)       | 通过文件对象创建字符输入流对象 |

  * 底层用到字节流

* **读数据**

  | 方法名                                | 说明                                                    |
  | ------------------------------------- | ------------------------------------------------------- |
  | int read()                            | 读一个字符，返回的**int**是字符的整数表示               |
  | int read(char[] cbuf)                 | 读最多cbuf.leng个字符到字符数组中，返回实际读到的字符数 |
  | int read(char[] cbuf,int off,int len) | 读最多len个字符到字符数组中，返回实际读到的字符数       |

* 读文件

  ```java
  FileReader fr = null;
  char[] chars = new char[1024];
  int len;
  try {
      fr = new FileReader("a.txt");
      while((len = fr.read(chars)) != -1){
          System.out.println(new String(chars,0,len));
      }
  } catch (IOException e) {
      e.printStackTrace();
  }finally {
      try {
          fr.close();
      } catch (IOException e) {
          e.printStackTrace();
      }
  }
  ```

# 字符缓冲流

字符缓冲流也是用于提高效率的

> 字符缓冲输出流**BufferedWriter**

* 构造方法

  | 方法                                       | 说明           |
  | ------------------------------------------ | -------------- |
  | public BufferedWriter(Writer out)          | 默认缓冲区大小 |
  | public BufferedWriter(Writer out,int size) | 指定缓冲区大小 |

  * 构造的参数是Writer

  * 特殊方法

    `public void newLine();`写入一个换行符

> 字符缓冲输入流**BufferedReader**

* 构造方法

  | 方法                                      | 说明           |
  | ----------------------------------------- | -------------- |
  | public BufferedReader(Reader in)          | 默认缓冲区大小 |
  | public BufferedReader(Reader in,int size) | 指定缓冲区大小 |

* 构造的参数是Reader

* 特殊方法

  `public String readLine();`读一行数据并返回，如果是文件末尾返回null

> 使用字符缓冲输出流范例

```java
BufferedWriter bw = null;
try {
    bw = new BufferedWriter(new FileWriter("a.txt"));
    bw.write("我是大帅逼");
} catch (IOException e) {
    e.printStackTrace();
}finally {
    try {
        bw.close();
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

> 使用字符缓冲输入流范例

```java
BufferedReader br = null;
char[] chars = new char[1024];
int len;
try {
    br = new BufferedReader(new FileReader("a.txt"));
    while((len = br.read(chars)) != -1){
        System.out.println(new String(chars,0,len));
    }
} catch (IOException e) {
    e.printStackTrace();
}finally {
    try {
        br.close();
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

# 转换流

> **InputStreamReader**：字节流转为字符流，硬盘-->内存

* InputStreamReader是FileReader的父类，构造FileReader对象底层是用到了InputStreamReader的构造方法
* 作用是将硬盘中的字节码读取出来，经过解码从而转换为字符，所以使用字符流使用到了转换流

> **OutputStreamWriter**：字符流转为字节流，内存-->硬盘

* OutputStreamWriter是FileWriter的父类，构造FileWriter对象底层是用到了OutputStreamWriter的构造方法
* 作用是将内存中的字符经过编码转换为字节码，然后写入硬盘，所以使用字符流使用到了转换流

转换流的构造方法可以**指定字符集**，在JDK11以前**字符流**不能指定字符集，而JDK11后字符流新增了可以指定字符集的**构造方法**，从而可以直接使用字符流来完成读写而无需使用转换流

# 对象流

将对象以流的形式传输，这种流称为对象流

> 分类

{% mermaid %}
graph TB;
对象流-->对象输入流
对象流-->对象输出流
对象输入流-->ObjectInputStream
对象输出流-->ObjectOutputStream
{% endmermaid %}

> 对象输出流

对象输出流又称为**对象序列化流**，用于将对象写入文件或在网络中传输，使用 **writeObject()** 方法写对象

| 方法                                        | 说明     |
| ------------------------------------------- | -------- |
| public ObjectOutputStream(OutputStream out) | 构造方法 |
| void writeObject(Object o)                  | 常用方法 |

> 对象输入流

对象输入流又称为**对象反序列化流**，用于将对象从文件中读到内存或接收网络中的对象，使用 **readObject()** 方法读对象，注意类型转换

| 方法                                     | 说明     |
| ---------------------------------------- | -------- |
| public ObjectInputStream(InputStream in) | 构造方法 |
| Object readObject()                      | 常用方法 |

> 注意事项

* 对象需要序列化必须实现**Serializable**接口，该接口是标记接口，没有任何抽象方法，只要是实现该接口的类，它的对象就能序列化

* 用对象序列化流序列化一个对象后，如果修改了对象所属的类，再次读取该对象时会抛出**InvalidClassException**异常

  * 因为**serialVersionUID**是由**虚拟机自动生成**，写入文件时与读取文件时的serialVersionUID不一致造成的

  * 解决办法是在序列化的类中手动给出serialVersionUID，并且这个值固定不变，格式是：

    `private static final long serialVersionUID = long值;`

* 如果类中某个成员变量的值不想要被序列化，可以给该成员变量加上**transient**关键字修饰，表示该成员变量不参与序列化过程

* 读取多个对象时难以分辨循环中是否读取到文件末尾，所以在存对象时可以将对象先存放在**集合**中再写入文件，这样读取对象时可以只使用一次读操作将该集合读取出来

# Properties类

> 基本介绍

* 是Map集合中的一个类，存放的是双列数据

* 含有跟IO相关的方法
* 键值对的数据类型一般使用String
* 由于是Map集合体系中的类，所以一般的增删改查方法与Map集合中的相同

> 相关方法

| 方法名                                      | 说明                                                         |
| ------------------------------------------- | ------------------------------------------------------------ |
| Object setProperty(String key,String value) | 设置集合的键和值，都是String类型，底层调用HashTable的put()方法 |
| String getProperty(String key)              | 根据键获取值                                                 |
| Set<String> stringPropertyNames()           | 返回一个装有键的Set集合，其中键值都是String类型              |

> load()和store()方法

| 方法名                                       | 说明                                                   |
| -------------------------------------------- | ------------------------------------------------------ |
| void load(InputStream in)                    | 从字节输入流中读取键值对到Peoperties集合               |
| void load(Reader reader)                     | 从字符输入流中读取键值对到Peoperties集合               |
| void store(OutputStream out,String comments) | 将Properties集合中的键值对写入字节输出流，保存到文件中 |
| void store(Writer writer,String comments)    | 将Properties集合中的键值对写入字符输出流，保存到文件中 |

* comments参数作用是在配置文件中写注释

* 一般存放Properties集合的文件后缀是 **.properties** ，一般用于配置文件
* 由于Properties集合没有关闭流的操作，所以不建议使用匿名内部类的形式来使用store()和load()方法

> store使用案例

```java
Properties properties = new Properties();
properties.setProperty("user","root");
properties.setProperty("password","root");

FileWriter fw = null;

try {
    fw = new FileWriter("prop.properties");
    properties.store(fw,null);
} catch (IOException e) {
    e.printStackTrace();
} finally {
    try {
        fw.close();
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

> load使用案例

```	java
Properties newProperties = new Properties();

FileReader fr = null;

try {
    fr = new FileReader("prop.properties");
    newProperties.load(fr);
    System.out.println(newProperties);
} catch (IOException e) {
    e.printStackTrace();
} finally {
    try {
        fr.close();
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```


