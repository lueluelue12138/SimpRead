> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/qq_44715943/article/details/116501936)

一、什么是 IO 流
----------

*   I : **Input**
*   O : **Output**

通过 IO 可以完成硬盘文件的**读和写**。

二、IO 流的分类
---------

1.  按照 **`流的方向`** 进行分类：

以**内存**作为参照物：

*   往内存中**去**：叫做**输入 (Input)**。或者叫做**读 (Read)**。
*   从内存中**出来**：叫做**输出 (Output)**。或者叫做**写 (Write)**。

2.  按照 **`读取数据方式`** 不同进行分类：

*   按照 **`字节`** 的方式读取数据，一次读取 1 个字节 byte，等同于一次读取 8 个二进制位。

这种流是**万能**的，什么类型的文件都可以读取。包括：**`文本文件，图片，声音文件，视频文件`** 等…

**eg.**  
假设文件 file1.txt，采用字节流的话是这样读的：  
a 中国 bc 张三 fe  
第一次读：一个字节，正好读到’a’  
第二次读：一个字节，正好读到’中’字符的一半。  
第三次读：一个字节，正好读到’中’字符的另外一半。

*   按照 **`字符`** 的方式读取数据的，一次读取一个字符.

这种流是为了方便读取 **`普通文本文件`** 而存在的，这种流不能读取：图片、声音、视频等文件。只能读取 **`纯文本文件`**，连 word 文件都无法读取。

**注意：**  
纯文本文件，不单单是. txt 文件，还包括 **.java、.ini、.py** 。总之只要 **`能用记事本打开`** 的文件都是普通文本文件。

**eg.**  
假设文件 file1.txt，采用字符流的话是这样读的：  
a 中国 bc 张三 fe  
第一次读：'a’字符（'a’字符在 windows 系统中占用 1 个字节。）  
第二次读：'中’字符（'中’字符在 windows 系统中占用 2 个字节。）

**综上所述：流的分类：**

*   **`输入流、输出流`**
*   **`字节流、字符流`**

三、IO 流四大家族首领
------------

*   字节流

1.  java.io.**InputStream** 字节输入流
2.  java.io.**OutputStream** 字节输出流

*   字符流

3.  java.io.**Reader** 字符输入流
4.  java.io.**Writer** 字符输出流

**注意：**

1.  四大家族的首领都是**抽象类**。(abstract class)
2.  所有的流都实现了：  
    **java.io.Closeable** 接口，都是**可关闭**的，都有 **`close()`** 方法。  
    流是一个管道，这个是内存和硬盘之间的通道，用完之后一定要关闭，不然会耗费 (占用) 很多资源。养成好习惯，用完流一定要关闭。
3.  所有的 **`输出流`** 都实现了：  
    **java.io.Flushable** 接口，都是**可刷新**的，都有 **`flush()`** 方法。  
    养成一个好习惯，输出流在最终输出之后，一定要记得 flush() 刷新一下。这个刷新表示将通道 / 管道当中剩余未输出的**数据强行输出完**（清空管道！）刷新的作用就是**清空管道**。

**ps**：**`如果没有flush()可能会导致丢失数据`**。

4.  在 java 中只要 “**类名**” 以 **`Stream`** 结尾的都是**字节流**。以 “ **`Reader/Writer`** ” 结尾的都是**字符流**。

四、Java 要掌握的流（16 个）
------------------

1.  **文件**专属：

*   java.io.FileInputStream（掌握）
*   java.io.FileOutputStream（掌握）
*   java.io.FileReader
*   java.io.FileWriter

2.  **转换流**：（将字节流转换成字符流）

*   java.io.InputStreamReader
*   java.io.OutputStreamWriter

3.  **缓冲流**专属：

*   java.io.BufferedReader
*   java.io.BufferedWriter
*   java.io.BufferedInputStream
*   java.io.BufferedOutputStream

4.  **数据流**专属：

*   java.io.DataInputStream
*   java.io.DataOutputStream

5.  **标准输出**流：

*   java.io.PrintWriter
*   java.io.PrintStream（掌握）

6.  **对象专属**流：

*   java.io.ObjectInputStream（掌握）
*   java.io.ObjectOutputStream（掌握）

7.  **File 文件类**

*   java.io.File

补充：Windows/Linux 小知识点
---------------------

Windows：`D:\Soft\QQ\Plugin`  
Linux：      `D:/Soft/QQ/Plugin`

**注意：** Windows 各个文件之间分隔符为：” **\** “；Linux 各个文件之间分割符为：” **/** “

补充：IDEA 默认的当前路径是？
-----------------

工程 Project 的**根**就是 IDEA 的默认当前路径

五、java.io.FileInputStream
-------------------------

文件字节输入流，万能的，任何类型的文件都可以采用这个流来读

### 构造方法

<table><thead><tr><th>构造方法名</th><th>备注</th></tr></thead><tbody><tr><td>FileInputStream(String name)</td><td>name 为文件路径</td></tr><tr><td>FileInputStream(File file)</td><td></td></tr></tbody></table>

### 方法

<table><thead><tr><th>方法名</th><th>作用</th></tr></thead><tbody><tr><td>int read()</td><td>读取一个字节，返回值为该字节 ASCII 码；读到文件末尾返回 - 1</td></tr><tr><td>int read(byte[] b)</td><td>读 b 数组长度的字节到 b 数组中，返回值为读到的字节个数；读到文件末尾返回 - 1</td></tr><tr><td>int read(byte[] b, int off, int len)</td><td>从 b 素组 off 位置读 len 长度的字节到 b 数组中，返回值为读到的字节个数；读到文件末尾返回 - 1</td></tr><tr><td>int available()</td><td>返回文件有效的字节数</td></tr><tr><td>long skip(long n)</td><td>跳过 n 个字节</td></tr><tr><td>void close()</td><td>关闭文件输入流</td></tr></tbody></table>

```
public class FileInputStreamTest04 {
    public static void main(String[] args) {
        FileInputStream fis = null;
        try {
            fis = new FileInputStream("chapter23/src/tempfile3");
            // 开始读，采用byte数组，一次读取多个字节。最多读取“数组.length”个字节。
            byte[] bytes = new byte[4];// 准备一个4个长度的byte数组，一次最多读取4个字节。
            int readCount = 0;
            // 这个方法的返回值是：读取到的字节数量。（不是字节本身）;1个字节都没有读取到返回-1(文件读到末尾)
            while((readCount = fis.read(bytes)) != -1) {
            	// 不应该全部都转换，应该是读取了多少个字节，转换多少个。
                System.out.print(new String(bytes, 0, readCount));
            }
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
        	// 在finally语句块当中确保流一定关闭。
            if (fis != null) {// 避免空指针异常！
            	// 关闭流的前提是：流不是空。流是null的时候没必要关闭。
                try {
                    fis.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

```
public class FileInputStreamTest05 {
    public static void main(String[] args) {
        FileInputStream fis = null;
        try {
            fis = new FileInputStream("tempfile");
            System.out.println("总字节数量：" + fis.available());
            // 读1个字节
            //int readByte = fis.read();
            // 还剩下可以读的字节数量是：5
            //System.out.println("剩下多少个字节没有读：" + fis.available());
            // 这个方法有什么用？
            byte[] bytes = new byte[fis.available()]; // 这种方式不太适合太大的文件，因为byte[]数组不能太大。
            // 不需要循环了。
            // 直接读一次就行了。
            int readCount = fis.read(bytes); // 6
            System.out.println(new String(bytes)); // abcdef

            // skip跳过几个字节不读取，这个方法也可能以后会用！
            fis.skip(3);
            System.out.println(fis.read()); //100

        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (fis != null) {
                try {
                    fis.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

六、java.io.FileOutputStream
--------------------------

### 构造方法

<table><thead><tr><th>构造方法名</th><th>备注</th></tr></thead><tbody><tr><td>FileOutputStream(String name)</td><td>name 为文件路径</td></tr><tr><td>FileOutputStream(String name, boolean append)</td><td>name 为文件路径，append 为 true 表示在文件末尾追加；为 false 表示清空文件内容，重新写入</td></tr><tr><td>FileOutputStream(File file)</td><td></td></tr><tr><td>FileOutputStream(File file, boolean append)</td><td>append 为 true 表示在文件末尾追加；为 false 表示清空文件内容，重新写入</td></tr></tbody></table>

### 方法

<table><thead><tr><th>方法名</th><th>作用</th></tr></thead><tbody><tr><td>void write(int b)</td><td>将指定字节写入文件中</td></tr><tr><td>void write(byte[] b)</td><td>将 b.length 个字节写入文件中</td></tr><tr><td>void write(byte[] b, int off, int len)</td><td>将 b 素组 off 位置开始，len 长度的字节写入文件中</td></tr><tr><td>void flush()</td><td>刷新此输出流并强制写出所有缓冲的输出字节</td></tr><tr><td>void close()</td><td>关闭文件输出流</td></tr></tbody></table>

```
public class FileOutputStreamTest01 {
    public static void main(String[] args) {
        FileOutputStream fos = null;
        try {
            // myfile文件不存在的时候会自动新建！
            // 这种方式谨慎使用，这种方式会先将原文件清空，然后重新写入。
            //fos = new FileOutputStream("myfile");

            // 以追加的方式在文件末尾写入。不会清空原文件内容。
            fos = new FileOutputStream("tempfile3", true);
            // 开始写。
            byte[] bytes = {97, 98, 99, 100};
            // 将byte数组全部写出！
            fos.write(bytes); // abcd
            // 将byte数组的一部分写出！
            fos.write(bytes, 0, 2); // 再写出ab

            // 字符串
            String s = "我是一个中国人，我骄傲！！！";
            // 将字符串转换成byte数组。
            byte[] bs = s.getBytes();
            // 写
            fos.write(bs);

            // 写完之后，最后一定要刷新
            fos.flush();
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (fos != null) {
                try {
                    fos.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

七、java.io.FileReader
--------------------

FileReader 文件字符输入流，只能读取普通文本。读取文本内容时，比较方便，快捷。

### 构造方法

<table><thead><tr><th>构造方法名</th><th>备注</th></tr></thead><tbody><tr><td>FileReader(String fileName)</td><td>name 为文件路径</td></tr><tr><td>FileReader(File file)</td><td></td></tr></tbody></table>

### 方法

<table><thead><tr><th>方法名</th><th>作用</th></tr></thead><tbody><tr><td>int read()</td><td>读取一个字符，返回值为该字符 ASCII 码；读到文件末尾返回 - 1</td></tr><tr><td>int read(char[] c)</td><td>读 c 数组长度的字节到 c 数组中，返回值为读到的字符个数；读到文件末尾返回 - 1</td></tr><tr><td>int read(char[] c, int off, int len)</td><td>从 c 素组 off 位置读 len 长度的字符到 c 数组中，返回值为读到的字符个数；读到文件末尾返回 - 1</td></tr><tr><td>long skip(long n)</td><td>跳过 n 个字符</td></tr><tr><td>void close()</td><td>关闭文件输入流</td></tr></tbody></table>

```
public class FileReaderTest {
    public static void main(String[] args) {
        FileReader reader = null;
        try {
            // 创建文件字符输入流
            reader = new FileReader("tempfile");
            
            // 开始读
            char[] chars = new char[4]; // 一次读取4个字符
            int readCount = 0;
            while((readCount = reader.read(chars)) != -1) {
                System.out.print(new String(chars,0,readCount));
            }
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (reader != null) {
                try {
                    reader.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

八、java.io.FileWriter
--------------------

FileWriter 文件字符输出流。写。只能输出普通文本。

### 构造方法

<table><thead><tr><th>构造方法名</th><th>备注</th></tr></thead><tbody><tr><td>FileWriter(String fileName)</td><td>name 为文件路径</td></tr><tr><td>FileWriter(String fileName, boolean append)</td><td>name 为文件路径，append 为 true 表示在文件末尾追加；为 false 表示清空文件内容，重新写入</td></tr><tr><td>FileWriter(File file)</td><td></td></tr><tr><td>FileWriter(File file, boolean append)</td><td>append 为 true 表示在文件末尾追加；为 false 表示清空文件内容，重新写入</td></tr></tbody></table>

### 方法

<table><thead><tr><th>方法名</th><th>作用</th></tr></thead><tbody><tr><td>void write(int c)</td><td>将指定字符写入文件中</td></tr><tr><td>void write(char[] c)</td><td>将 c.length 个字符写入文件中</td></tr><tr><td>void write(char[] c, int off, int len)</td><td>将 c 素组 off 位置开始，len 长度的字符写入文件中</td></tr><tr><td>void write(String str)</td><td>将字符串写入文件中</td></tr><tr><td>void write(String str, int off, int len)</td><td>从字符串 off 位置开始截取 len 长度的字符串写入文件</td></tr><tr><td>void flush()</td><td>刷新此输出流并强制写出所有缓冲的输出字符</td></tr><tr><td>void close()</td><td>关闭文件输出流</td></tr></tbody></table>

```
public class FileWriterTest {
    public static void main(String[] args) {
        FileWriter out = null;
        try {
            // 创建文件字符输出流对象
            //out = new FileWriter("file");
            out = new FileWriter("file", true);

            // 开始写。
            char[] chars = {'我','是','中','国','人'};
            out.write(chars);
            out.write(chars, 2, 3);

            out.write("我是一名java软件工程师！");
            // 写出一个换行符。
            out.write("\n");
            out.write("hello world!");

            // 刷新
            out.flush();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (out != null) {
                try {
                    out.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

九、 java.io.BufferedReader、java.io.InputStreamReader
---------------------------------------------------

BufferedReader 带有缓冲区的字符输入流。使用这个流的时候不需要自定义 char 数组，或者说不需要自定义 byte 数组。自带缓冲。

InputStreamReader 字节输入流转字符输入流

### 构造方法

<table><thead><tr><th>构造方法名</th><th>备注</th></tr></thead><tbody><tr><td>BufferedReader(Reader in)</td><td>in 为 reader 对象（可以是 reader 的实现类）</td></tr></tbody></table>

### 方法

<table><thead><tr><th>方法名</th><th>作用</th></tr></thead><tbody><tr><td>int read()</td><td>读取一个字符，返回值为该字符 ASCII 码；读到文件末尾返回 - 1</td></tr><tr><td>int read(char[] c)</td><td>读 c 数组长度的字节到 c 数组中，返回值为读到的字符个数；读到文件末尾返回 - 1</td></tr><tr><td>String readLine()</td><td>读取文件一行</td></tr><tr><td>long skip(long n)</td><td>跳过 n 个字符</td></tr><tr><td>void close()</td><td>关闭文件输入流</td></tr></tbody></table>

**包括包装流的方法**

```
public class BufferedReaderTest01 {
    public static void main(String[] args) throws Exception{

        FileReader reader = new FileReader("Copy02.java");
        // 当一个流的构造方法中需要一个流的时候，这个被传进来的流叫做：节点流。
        // 外部负责包装的这个流，叫做：包装流，还有一个名字叫做：处理流。
        // 像当前这个程序来说：FileReader就是一个节点流。BufferedReader就是包装流/处理流。
        BufferedReader br = new BufferedReader(reader);

        // br.readLine()方法读取一个文本行，但不带换行符。
        String s = null;
        while((s = br.readLine()) != null){
            System.out.print(s);
        }

        // 关闭流
        // 对于包装流来说，只需要关闭最外层流就行，里面的节点流会自动关闭。（可以看源代码。）
        br.close();
    }
}
```

**转换流：InputStreamReader**

```
public class BufferedReaderTest02 {
    public static void main(String[] args) throws Exception{

        /*// 字节流
        FileInputStream in = new FileInputStream("Copy02.java");

        // 通过转换流转换（InputStreamReader将字节流转换成字符流。）
        // in是节点流。reader是包装流。
        InputStreamReader reader = new InputStreamReader(in);

        // 这个构造方法只能传一个字符流。不能传字节流。
        // reader是节点流。br是包装流。
        BufferedReader br = new BufferedReader(reader);*/

        // 合并
        BufferedReader br = new BufferedReader(new InputStreamReader(new FileInputStream("Copy02.java")));

        String line = null;
        while((line = br.readLine()) != null){
            System.out.println(line);
        }

        // 关闭最外层
        br.close();
    }
}
```

十、java.io.BufferedWriter、 java.io.OutputStreamWriter
----------------------------------------------------

**BufferedWriter：带有缓冲的字符输出流。  
OutputStreamWriter：字节输出流转字符输出流**

### 构造方法

<table><thead><tr><th>构造方法名</th><th>备注</th></tr></thead><tbody><tr><td>BufferedWriter(Writer out)</td><td>out 为 Writer 对象（可以是 reader 的实现类）</td></tr></tbody></table>

### 方法

<table><thead><tr><th>方法名</th><th>作用</th></tr></thead><tbody><tr><td>void write(int c)</td><td>将指定字符写入文件中</td></tr><tr><td>void write(char[] c, int off, int len)</td><td>将 c 素组 off 位置开始，len 长度的字符写入文件中</td></tr><tr><td>void write(String str, int off, int len)</td><td>从字符串 off 位置开始截取 len 长度的字符串写入文件</td></tr><tr><td>void flush()</td><td>刷新此输出流并强制写出所有缓冲的输出字符</td></tr><tr><td>void close()</td><td>关闭文件输出流</td></tr></tbody></table>

```
public static void main(String[] args) throws Exception{
        // 带有缓冲区的字符输出流
        BufferedWriter out = new BufferedWriter(new OutputStreamWriter(new FileOutputStream("copy", true)));
        // 开始写。
        out.write("hello world!");
        out.write("\n");
        out.write("hello kitty!");
        // 刷新
        out.flush();
        // 关闭最外层
        out.close();
    }
}
```

**包括包装流的方法**

十一、java.io.DataInputStream
--------------------------

DataInputStream: 数据字节输入流。

DataOutputStream 写的文件，只能使用 DataInputStream 去读。并且读的时候你需要提前知道写入的顺序。

**读的顺序需要和写的顺序一致。才可以正常取出数据。**。

### 构造方法

<table><thead><tr><th>构造方法名</th><th>备注</th></tr></thead><tbody><tr><td>DataInputStream(InputStream in)</td><td>in 为 InputStream 对象</td></tr></tbody></table>

### 方法

<table><thead><tr><th>方法名</th><th>作用</th></tr></thead><tbody><tr><td>boolean readBoolean()</td><td>从文件中读取 boolean 字节数据</td></tr><tr><td>byte readByte()</td><td>从文件中读取 byte 字节数据</td></tr><tr><td>char readChar()</td><td>从文件中读取 char 字节数据</td></tr><tr><td>double readDouble()</td><td>从文件中读取 double 字节数据</td></tr><tr><td>float readFloat()</td><td>从文件中读取 float 字节数据</td></tr><tr><td>int readInt()</td><td>从文件中读取 int 字节数据</td></tr><tr><td>long readLong()</td><td>从文件中读取 long 字节数据</td></tr><tr><td>short readShort()</td><td>从文件中读取 short 字节数据</td></tr></tbody></table>

```
public class DataInputStreamTest01 {
    public static void main(String[] args) throws Exception{
        DataInputStream dis = new DataInputStream(new FileInputStream("data"));
        // 开始读
        byte b = dis.readByte();
        short s = dis.readShort();
        int i = dis.readInt();
        long l = dis.readLong();
        float f = dis.readFloat();
        double d = dis.readDouble();
        boolean sex = dis.readBoolean();
        char c = dis.readChar();

        System.out.println(b);
        System.out.println(s);
        System.out.println(i + 1000);
        System.out.println(l);
        System.out.println(f);
        System.out.println(d);
        System.out.println(sex);
        System.out.println(c);

        dis.close();
    }
}
```

十二、 java.io.DataOutputStream
----------------------------

java.io.DataOutputStream：数据字节输出流。

这个流可以将 **`数据连同数据的类型`** 一并写入文件。

注意：**这个文件不是普通文本文档**。（这个文件使用记事本打不开。）

### 构造方法

<table><thead><tr><th>构造方法名</th><th>备注</th></tr></thead><tbody><tr><td>DataOutputStream(OutputStream out)</td><td>out 为 OutputStream 对象</td></tr></tbody></table>

### 方法

<table><thead><tr><th>方法名</th><th>作用</th></tr></thead><tbody><tr><td>void writeBoolean(boolean v)</td><td>将 boolean 字节写入文件</td></tr><tr><td>void writeByte(int v)</td><td>将 byte 字节写入文件</td></tr><tr><td>void writeBytes(String s)</td><td>将 bytes 字节（字符串）写入文件</td></tr><tr><td>void writeChar(int v)</td><td>将 char 字节写入文件</td></tr><tr><td>void writeChars(String s)</td><td>将 chars 字节（字符串）写入文件</td></tr><tr><td>void writeDouble(double v)</td><td>将 double 字节写入文件</td></tr><tr><td>void writeFloat(float v)</td><td>将 float 字节写入文件</td></tr><tr><td>void writeInt(int v)</td><td>将 int 字节写入文件</td></tr><tr><td>void writeLong(long v)</td><td>将 long 字节写入文件</td></tr><tr><td>void writeShort(int v)</td><td>将 short 字节写入文件</td></tr><tr><td>void flush()</td><td>刷新此输出流并强制写出所有缓冲的输出字符</td></tr></tbody></table>

```
public class DataOutputStreamTest {
    public static void main(String[] args) throws Exception{
        // 创建数据专属的字节输出流
        DataOutputStream dos = new DataOutputStream(new FileOutputStream("data"));
        // 写数据
        byte b = 100;
        short s = 200;
        int i = 300;
        long l = 400L;
        float f = 3.0F;
        double d = 3.14;
        boolean sex = false;
        char c = 'a';
        // 写
        dos.writeByte(b); // 把数据以及数据的类型一并写入到文件当中。
        dos.writeShort(s);
        dos.writeInt(i);
        dos.writeLong(l);
        dos.writeFloat(f);
        dos.writeDouble(d);
        dos.writeBoolean(sex);
        dos.writeChar(c);

        // 刷新
        dos.flush();
        // 关闭最外层
        dos.close();
    }
}
```

十三、java.io.PrintStream
----------------------

java.io.PrintStream：标准的字节输出流。默认输出到控制台。

### 构造方法

<table><thead><tr><th>构造方法名</th><th>备注</th></tr></thead><tbody><tr><td>PrintStream(File file)</td><td></td></tr><tr><td>PrintStream(OutputStream out)</td><td></td></tr><tr><td>PrintStream(String fileName)</td><td>fileName 文件地址</td></tr></tbody></table>

### 方法

<table><thead><tr><th>方法</th><th>作用</th></tr></thead><tbody><tr><td>println(参数类型不定 x)</td><td>输出 x 带换行</td></tr><tr><td>print(参数类型不定 x)</td><td>输出 x 不带换行</td></tr><tr><td>void flush()</td><td>刷新此输出流并强制写出所有缓冲的输出字符</td></tr><tr><td>void close()</td><td>关闭流</td></tr></tbody></table>

### 改变流的输出方向

**`System.setOut(PrintStream对象)`**

**注意：**

1.  **标准输出流不需要手动 close() 关闭**。
2.  可以改变标准输出流的输出方向

```
public class PrintStreamTest {
    public static void main(String[] args) throws Exception{
        // 可以改变标准输出流的输出方向吗？ 可以// 标准输出流不再指向控制台，指向“log”文件。
        PrintStream printStream = new PrintStream(new FileOutputStream("log"));
        // 修改输出方向，将输出方向修改到"log"文件。
        System.setOut(printStream);// 修改输出方向
        // 再输出
        System.out.println("hello world");
        System.out.println("hello kitty");
        System.out.println("hello zhangsan");
    }
}
```

### 补充：学习对象流前言

1、java.io.NotSerializableException: Student 对象不支持序列化！！！！

2、参与**序列化**和**反序列化**的对象，必须实现 **`Serializable`** 接口。

3、注意：通过源代码发现，**Serializable** 接口只是一个 **`标志接口`**：

```
public interface Serializable {
}
```

这个接口当中什么代码都没有。

#### 3.1Serializable 接口起什么作用呢？

*   起到 **`标识`** 的作用，标志的作用，java 虚拟机看到这个类实现了这个接口，可能会对这个类进行特殊待遇。
*   Serializable 这个标志接口是给 java 虚拟机参考的，java 虚拟机看到这个接口之后，会为该类自动生成一个序列化版本号。

### 4、序列化版本号有什么用呢？

区分两个类是否相同。

### 5、java 语言中是采用什么机制来区分类的？

1.  第一：首先通过 **`类名`** 进行比对，如果类名不一样，肯定不是同一个类。
2.  第二：如果类名一样，再怎么进行类的区别？靠 **`序列化版本号`** 进行区分。

**eg.**  
小明编写了一个类：com.baidu.java.bean.Student implements Serializable  
小红编写了一个类：com.baidu.java.bean.Student implements Serializable  
不同的人编写了同一个类，但 “这两个类确实不是同一个类”。这个时候序列化版本就起上作用了。  
对于 java 虚拟机来说，java 虚拟机是可以区分开这两个类的，因为这两个类都实现了 Serializable 接口，都有默认的序列化版本号，他们的序列化版本号不一样。所以区分开了。（这是自动生成序列化版本号的好处）

### 6、这种自动生成序列化版本号有什么缺陷？

Java 虚拟机看到 Serializable 接口之后，会**自动生成**一个序列化版本号。

这种自动生成的序列化版本号缺点是：一旦代码确定之后，**不能进行后续的修改**，因为只要修改，必然会重新编译，此时会生成全新的序列化版本号，这个时候 java 虚拟机会认为这是一个全新的类。（这样就不好了！）

### 7、最终结论：

凡是一个类实现了 Serializable 接口，建议给该类**提供一个固定不变的序列化版本号**。  
这样，以后这个类即使代码修改了，但是版本号不变，java 虚拟机会认为是同一个类。

### 8、怎样使某个属性不序列化

使用 **`transient`** 关键字

transient 关键字表示游离的，**不参与序列化**。

```
public user implements Serializable{
    private int no;
    private transient String name; // name不参与序列化操作！
}
```

十四、java.io.ObjectOutputStream
-----------------------------

ObjectOutputStream：序列化对象

### 构造方法

<table><thead><tr><th>构造方法名</th><th>备注</th></tr></thead><tbody><tr><td>ObjectOutputStream(OutputStream out)</td><td>out 为 OutputStream 对象</td></tr></tbody></table>

### 方法

**参考 API**

```
public class ObjectOutputStreamTest01 {
    public static void main(String[] args) throws Exception{
        // 创建java对象
        Student s = new Student(1111, "zhangsan");
        // 序列化
        ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("students"));

        // 序列化对象
        oos.writeObject(s);
        // 刷新
        oos.flush();
        // 关闭
        oos.close();
    }
}
```

一次序列化多个对象可以将对象放到集合当中，**序列化集合**。

**提示：**  
参与序列化的 ArrayList 集合以及集合中的元素 User 都需要实现 **`java.io.Serializable`** 接口。

```
public class ObjectOutputStreamTest02 {
    public static void main(String[] args) throws Exception{
        List<User> userList = new ArrayList<>();
        userList.add(new User(1,"zhangsan"));
        userList.add(new User(2, "lisi"));
        userList.add(new User(3, "wangwu"));
        ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("users"));

        // 序列化一个集合，这个集合对象中放了很多其他对象。
        oos.writeObject(userList);

        oos.flush();
        oos.close();
    }
}
```

十五、java.io.ObjectInputStream
----------------------------

ObjectInputStream：反序列化对象

### 构造方法

<table><thead><tr><th>构造方法名</th><th>备注</th></tr></thead><tbody><tr><td>ObjectInputStream(InputStream in)</td><td>in 为 InputStream 对象</td></tr></tbody></table>

### 方法

**参考 API**

```
public class ObjectInputStreamTest01 {
    public static void main(String[] args) throws Exception{
        ObjectInputStream ois = new ObjectInputStream(new FileInputStream("students"));
        // 开始反序列化，读
        Object obj = ois.readObject();
        // 反序列化回来是一个学生对象，所以会调用学生对象的toString方法。
        System.out.println(obj);
        ois.close();
    }
}
```

反序列化集合

```
public class ObjectInputStreamTest02 {
    public static void main(String[] args) throws Exception{
        ObjectInputStream ois = new ObjectInputStream(new FileInputStream("users"));
        //Object obj = ois.readObject();
        //System.out.println(obj instanceof List);//true
        List<User> userList = (List<User>)ois.readObject();
        for(User user : userList){
            System.out.println(user);
        }
        ois.close();
    }
}
```

十六、java.io.File
---------------

1.  File 类和四大家族没有关系，所以 File 类不能完成文件的读和写。
2.  File 对象代表什么？  
    **`文件`** 和 **`目录路径名`** 的**抽象表示**形式。

**eg.**

*   C:\Drivers 这是一个 File 对象
*   C:\Drivers\Lan\Realtek\Readme.txt 也是 File 对象。
*   一个 File 对象有可能对应的是**目录**，也可能是**文件**。
*   File 只是一个 **`路径名`** 的**抽象表示**形式。

### 构造方法

<table><thead><tr><th>构造方法名</th><th>备注</th></tr></thead><tbody><tr><td>File(String pathname)</td><td>pathname 文件 / 文件夹路径</td></tr></tbody></table>

### 方法

<table><thead><tr><th>方法名</th><th>作用</th></tr></thead><tbody><tr><td>boolean delete()</td><td>删除文件 / 文件夹</td></tr><tr><td>boolean exists()</td><td>判断文件 / 文件夹是否存在</td></tr><tr><td>--------</td><td>--------</td></tr><tr><td>File getAbsoluteFile()</td><td>获取文件 / 文件夹的绝对路径（返回值：File）</td></tr><tr><td>String getName()</td><td>获得文件 / 文件夹名字</td></tr><tr><td>String getParent()</td><td>获取文件 / 文件夹的父文件 / 文件夹</td></tr><tr><td>File getParentFile()</td><td>获取文件 / 文件夹的父文件 / 文件夹（返回值：File）</td></tr><tr><td>String getPath()</td><td>获取文件 / 文件夹的路径</td></tr><tr><td></td><td></td></tr><tr><td>boolean isDirectory()</td><td>判断该文件 / 文件夹是不是文件夹</td></tr><tr><td>isFile()</td><td>判断该文件 / 文件夹是不是文件</td></tr><tr><td>isHidden()</td><td>判断该文件 / 文件夹是否隐藏</td></tr><tr><td>--------</td><td>--------</td></tr><tr><td>long lastModified()</td><td>获取文件 / 文件夹最后一次修改时间</td></tr><tr><td>long length()</td><td>获取文件大小；获取文件夹里面的文件个数</td></tr><tr><td>String[] list()</td><td>获取文件夹的文件名字以 String[] 返回</td></tr><tr><td>File[] listFiles()</td><td>获取文件夹的文件名字以 File[] 返回</td></tr><tr><td>boolean mkdir()</td><td>创建文件 / 文件夹</td></tr><tr><td>boolean mkdirs()</td><td>创建多重文件夹</td></tr></tbody></table>

```
class FileTest01{
    public static void main(String[] args) {
        File f1 = new File("D:/IO/File1");
        if (!f1.exists()){
            try {
                f1.createNewFile();//创建文件
            } catch (IOException e) {
                e.printStackTrace();
            }
        }

        File f2 = new File("D:/IO/File2");
        if (!f2.exists()){
            f2.mkdir();//创建文件夹
        }

        File f3 = new File("D:/IO/File3/a/b/c/d/e/f/g/h/i");
        if (!f3.exists()){
            f3.mkdirs();//创建多重文件夹
        }

        File f5 = new File("D:\\IO\\FileDelete");
        f5.delete();

        File f4 = new File("D:\\Data\\新建文件夹");
        String s1 = f4.getName();//新建文件夹
        System.out.println(s1);

        String s2 = f4.getParent();//D:\Data
        System.out.println(s2);

        String s3 = f4.getPath();//D:\Data\新建文件夹
        System.out.println(s3);

        String s4 = f4.getAbsolutePath();//D:\Data\新建文件夹
        System.out.println(s4);

        File asf = f4.getAbsoluteFile();
        System.out.println(asf.getAbsolutePath());//D:\Data\新建文件夹

        File pf = f4.getParentFile();
        System.out.println(pf.getAbsolutePath());//D:\Data

        System.out.println(f4.isDirectory());//true

        System.out.println(f4.isFile());//false

        System.out.println(f4.isHidden());//false

        System.out.println(f4.isAbsolute());//true

        File f6 = new File("D:\\IO\\Day24.java");
        System.out.println(f6.length());//5743字节

        long lastModify = f6.lastModified();//最后修改时间
        Date d = new Date(lastModify);
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        String date = sdf.format(d);
        System.out.println(date);//2021-05-03 22:55:06

        File f7 = new File("D:\\Data\\新建文件夹\\6、2020年最新 Java零基础入门到精通【完整资料】\\00_课程引入【马士兵说】");
        String[] strList = f7.list();
        for (String s : strList){
            System.out.println(s);
        }

        System.out.println("-----------------------------------------");
        File[] fileList = f7.listFiles();
        for (File f : fileList){
            //System.out.println(f.getPath());
            System.out.println(f.getAbsolutePath());
        }
    }
}
```

附：测试代码
------

```
import java.io.*;
import java.text.SimpleDateFormat;
import java.util.ArrayList;
import java.util.Date;
import java.util.List;
import java.util.Objects;

class FileInputStreamTest01{
    public static void main(String[] args) {
        FileInputStream fis = null;
        try {
            fis = new FileInputStream("D:\\IO\\read.txt");

            int res = fis.read();//读到返回该字符ASCII码，没读到返回-1
            System.out.println(res);//97
            res = fis.read();
            System.out.println(res);//98
            res = fis.read();
            System.out.println(res);//99
            res = fis.read();
            System.out.println(res);//100
            res = fis.read();
            System.out.println(res);//-1
            res = fis.read();
            System.out.println(res);//-1
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (fis != null) {
                try {
                    fis.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}

class FileInputStreamTest02{
    public static void main(String[] args) {
        FileInputStream fis = null;
        try {
            fis = new FileInputStream("D:/IO/read.txt");

            int res = 0;
            while((res = fis.read()) != -1){
                System.out.println(res);
            }

        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (fis != null) {
                try {
                    fis.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}

class FileInputStreamTest03{
    public static void main(String[] args) {
        FileInputStream fis = null;
        try {
            fis = new FileInputStream("D:/IO/read.txt");
            byte[] b = new byte[4];

            int readCount = fis.read(b);
            System.out.println(readCount);//4
            System.out.println(new String(b));//abcd
            readCount = fis.read(b);//2
            System.out.println(readCount);
            System.out.println(new String(b));//efcd  //数组不会清空，每一轮从0开始，读取存入
            readCount = fis.read(b);
            System.out.println(readCount);//-1

        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (fis != null) {
                try {
                    fis.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}

class FileInputStreamTest04{
    public static void main(String[] args) {
        FileInputStream fis = null;

        try {
            fis = new FileInputStream("D:/IO/read.txt");
            byte[] b = new byte[4];

            int readCount = fis.read(b);
            System.out.println(new String(b, 0, readCount));//abcd
            readCount = fis.read(b);
            System.out.println(new String(b, 0, readCount));//ef

        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (fis != null) {
                try {
                    fis.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}

class FileInputStreamTest05{
    public static void main(String[] args) {
        FileInputStream fis = null;

        try {
            fis = new FileInputStream("D:/IO/read.txt");
            byte[] b = new byte[30];//读中文时，数据需开大一点，否则会乱码（一个汉字等于两字节）
            int readCount = 0;

            while((readCount = fis.read(b)) != -1){
                System.out.println(new String(b, 0, readCount));
            }

        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (fis != null) {
                try {
                    fis.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}

class FileInputStreamTest06{
    public static void main(String[] args) {
        FileInputStream fis = null;
        try {
            fis = new FileInputStream("D:/IO/read.txt");

            /*int read = fis.read();
            System.out.println(fis.available());//5*/

            byte[] b = new byte[fis.available()];//不适合大数据量，因为内存中很难找到一块连续的空间
            fis.read(b);//一次读完
            System.out.println(new String(b));//abcdef

        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (fis != null) {
                try {
                    fis.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}

class FileInputStreamTest06_1{
    public static void main(String[] args) {
        FileInputStream fis = null;

        try {
            fis = new FileInputStream("D:/IO/read.txt");

            int read = fis.read();
            System.out.println((char)read);//a
            fis.skip(2);//跳过两个字节
            read = fis.read();
            System.out.println((char)read);//d

        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (fis != null) {
                try {
                    fis.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}

class FileOutputStreamTest01{
    public static void main(String[] args) {
        FileOutputStream fos = null;

        try {
            fos = new FileOutputStream("D:\\IO\\write1.txt");//没有文件会自动创建，每次自动清空文件内容，慎用！！！

            fos.write(65);
            fos.write(66);
            fos.write(67);
            fos.write(68);

            byte[] b = {97, 98, 99 , 100};
            fos.write(b);

            fos.write(b, 1, 2);

            fos.flush();
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (fos != null) {
                try {
                    fos.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}

class FileOutputStreamTest02{
    public static void main(String[] args) {
        FileOutputStream fos = null;

        try {
            fos = new FileOutputStream("D:/IO/write2.txt", true);

            byte[] b = {97, 98, 99 , 100};
            fos.write(b, 2, 1);
            String s = "我是中国人";
            byte[] bytes = s.getBytes();
            fos.write(bytes);

        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (fos != null) {
                try {
                    fos.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}

class Copy01{
    public static void main(String[] args) {
        FileInputStream fis = null;
        FileOutputStream fos = null;

        try {
            fis = new FileInputStream("D:\\Data\\新建文件夹\\6、2020年最新 Java零基础入门到精通【完整资料】\\00_课程引入【马士兵说】\\视频\\1.引入_授课说明【   www.52downloadcn】.mp4");
            fos = new FileOutputStream("D:/IO/授课说明.mp4");
            byte[] b = new byte[1024 * 1024];//1MB
            int readCount = 0;

            //一边读一边写
            while ((readCount = fis.read(b)) != -1){
                fos.write(b, 0 , readCount);
            }
            fos.flush();

        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (fos != null) {
                try {
                    fos.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if (fis != null) {
                try {
                    fis.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}

class FileReaderTest01{
    public static void main(String[] args) {
        FileReader in = null;

        try {
            in = new FileReader("D:\\IO\\read.txt");
            int readCount = 0;
            while ((readCount = in.read()) != -1){
                System.out.print((char)readCount);
            }
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (in != null) {
                try {
                    in.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}

class FileReaderTest02{
    public static void main(String[] args) {
        FileReader reader = null;

        try {
            reader = new FileReader("D:\\IO\\read.txt");
            char[] c = new char[4];
            int readCount = 0;
            while ((readCount = reader.read(c)) != -1){
                System.out.println(new String(c, 0, readCount));
            }
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (reader != null) {
                try {
                    reader.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}

class FileWriterTest{
    public static void main(String[] args) {
        FileWriter writer = null;

        try {
            writer = new FileWriter("D:/IO/writer3.txt", true);
            writer.write(87);
            writer.write("我是中国人");
            char[] c = {'\n', '你', '好', '中', '国'};
            writer.write(c);
            writer.write(c, 1, 2);
            writer.flush();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (writer != null) {
                try {
                    writer.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}

class Copy02{
    public static void main(String[] args) {
        FileReader reader = null;
        FileWriter writer = null;

        try {
            reader = new FileReader("D:\\IDEA_WorkPlace\\java_WorkPlace\\TestProject\\Practice\\src\\practice\\Day24.java");
            writer = new FileWriter("D:/IO/Day24.java");
            char[] c = new char[1024 * 512];//1MB
            int readCount = 0;
            //边读边写
            while((readCount = reader.read(c)) != -1){
                writer.write(c, 0, readCount);
            }
            writer.flush();
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally{
            if (writer != null) {
                try {
                    writer.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if (reader != null) {
                try {
                    reader.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}

class BufferedReaderTest01{
    public static void main(String[] args) {
        BufferedReader reader = null;
        try {
            FileReader fr = new FileReader("D:\\IO\\Day24.java");//节点流
            reader = new BufferedReader(fr);//包装流
            int readCount = 0;
            while ((readCount = reader.read()) != -1){//单个取
                System.out.print((char)readCount);//加ln排版有问题
            }
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally{
            if (reader != null) {
                try {
                    reader.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }

    }
}

class BufferedReaderTest02{
    public static void main(String[] args) {
        BufferedReader reader = null;
        try {
            reader = new BufferedReader(new FileReader("D:\\IO\\Day24.java"));
            int readCount = 0;
            char[] c = new char[10];//字节数组
            while ((readCount = reader.read(c)) != -1){
                System.out.print(new String(c, 0, readCount));//加ln排版有问题
            }
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally{
            if (reader != null) {
                try {
                    reader.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }

    }
}

class BufferedReaderTest03{
    public static void main(String[] args) {
        BufferedReader reader = null;
        try {
            reader = new BufferedReader(new FileReader("D:/IO/Day24.java"));
            String res =  "";
            while((res = reader.readLine()) != null){
                System.out.println(res);//readLine()读不到换行符，需要手动换行
            }
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally{
            if (reader != null) {
                try {
                    reader.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }

    }
}

class BufferWriterTest01{
    public static void main(String[] args) {
        BufferedWriter writer = null;
        try {
            FileWriter fw = new FileWriter("D:/IO/writer4.txt", true);
            writer = new BufferedWriter(fw);
            writer.write(97);
            writer.write("我是中国人");
            writer.write(new char[]{'福', '建', '人'});
            writer.flush();
        } catch (IOException e) {
            e.printStackTrace();
        }finally{
            if (writer != null) {
                try {
                    writer.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}

class BufferedReaderTest04{
    public static void main(String[] args) {
        BufferedReader reader = null;
        try {
            FileInputStream fis = new FileInputStream("D:/IO/Day24.java");
            InputStreamReader isr = new InputStreamReader(fis);//字节流转字符流
            reader = new BufferedReader(isr);

            String res =  "";
            while((res = reader.readLine()) != null){
                System.out.println(res);//readLine()读不到换行符，需要手动换行
            }
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally{
            if (reader != null) {
                try {
                    reader.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}

class BufferWriterTest02{
    public static void main(String[] args) {
        BufferedWriter writer = null;
        try {
            writer = new BufferedWriter(new OutputStreamWriter(new FileOutputStream("D:/IO/writer5.txt", true)));//三步合一
            writer.write(97);
            writer.write("我是中国人");
            writer.write(new char[]{'福', '建', '人'});
            writer.flush();
        } catch (IOException e) {
            e.printStackTrace();
        }finally{
            if (writer != null) {
                try {
                    writer.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}

class DataOutputStreamTest01{
    public static void main(String[] args) {
        DataOutputStream dos = null;

        try {
            dos = new DataOutputStream(new FileOutputStream("D:/IO/writer6.txt", true));
            byte b = 1;
            short s = 2;
            int i = 3;
            long l = 4L;
            float f = 3.99F;
            double d = 3.14;
            boolean flag = true;
            char sex = '男';
            dos.writeByte(b);
            dos.writeShort(s);
            dos.writeInt(i);
            dos.writeLong(l);
            dos.writeFloat(f);
            dos.writeDouble(d);
            dos.writeBoolean(flag);
            dos.writeChar(sex);
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally{
            if (dos != null) {
                try {
                    dos.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}

class DataInputStreamTest01{
    public static void main(String[] args) {
        DataInputStream dis = null;

        try {
            dis = new DataInputStream(new FileInputStream("D:/IO/writer6.txt"));
            System.out.println(dis.readByte());
            System.out.println(dis.readShort());
            System.out.println(dis.readInt());
            System.out.println(dis.readLong());
            System.out.println(dis.readFloat());
            System.out.println(dis.readDouble());
            System.out.println(dis.readBoolean());
            System.out.println(dis.readChar());
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (dis != null) {
                try {
                    dis.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}

class PrintStreamTest01{
    public static void main(String[] args) {

        try {
            //1.改变流的输出方向
            PrintStream ps = new PrintStream(new FileOutputStream("D:/IO/writer7.txt", true));
            //PrintStream ps = new PrintStream("D:/IO/writer7.txt");//会清空内容
            System.setOut(ps);

            System.out.println("hello world");
            System.out.println("你好世界");
            System.out.println("hi world");

            //标准输出流不需要关闭
            //ps.flush();
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        }

    }
}

class Logger{
    public static void log(String msg){
        try {
            System.setOut(new PrintStream(new FileOutputStream("D:/IO/Log.txt", true)));
            Date d = new Date();
            SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
            String date = sdf.format(d);
            System.out.println(date + ":" + msg);
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        }
    }
}

class LoggerTest{
    public static void main(String[] args) {
        Logger.log("用户登入");
        Logger.log("用户备份数据库记录");
        Logger.log("用户调用GC垃圾回收器");
        Logger.log("用户删除数据库信息");
        Logger.log("用户退出");
    }
}

class Logger02{
    public static void log(String msg){
        BufferedWriter writer = null;

        try {
            writer = new BufferedWriter(new OutputStreamWriter(new FileOutputStream("D:/IO/Log2.txt", true)));
            Date d = new Date();
            SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
            String date = sdf.format(d);
            writer.write(date + ":" + msg + '\n');
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally{
            if (writer != null) {
                try {
                    writer.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}

class LoggerTest02{
    public static void main(String[] args) {
        Logger02.log("用户登入");
        Logger02.log("用户备份数据库记录");
        Logger02.log("用户调用GC垃圾回收器");
        Logger02.log("用户删除数据库信息");
        Logger02.log("用户退出");
    }
}

class FileTest01{
    public static void main(String[] args) {
        File f1 = new File("D:/IO/File1");
        if (!f1.exists()){
            try {
                f1.createNewFile();//创建文件
            } catch (IOException e) {
                e.printStackTrace();
            }
        }

        File f2 = new File("D:/IO/File2");
        if (!f2.exists()){
            f2.mkdir();//创建文件夹
        }

        File f3 = new File("D:/IO/File3/a/b/c/d/e/f/g/h/i");
        if (!f3.exists()){
            f3.mkdirs();//创建多重文件夹
        }

        File f5 = new File("D:\\IO\\FileDelete");
        f5.delete();

        File f4 = new File("D:\\Data\\新建文件夹");
        String s1 = f4.getName();//新建文件夹
        System.out.println(s1);

        String s2 = f4.getParent();//D:\Data
        System.out.println(s2);

        String s3 = f4.getPath();//D:\Data\新建文件夹
        System.out.println(s3);

        String s4 = f4.getAbsolutePath();//D:\Data\新建文件夹
        System.out.println(s4);

        File asf = f4.getAbsoluteFile();
        System.out.println(asf.getAbsolutePath());//D:\Data\新建文件夹

        File pf = f4.getParentFile();
        System.out.println(pf.getAbsolutePath());//D:\Data

        System.out.println(f4.isDirectory());//true

        System.out.println(f4.isFile());//false

        System.out.println(f4.isHidden());//false

        System.out.println(f4.isAbsolute());//true

        File f6 = new File("D:\\IO\\Day24.java");
        System.out.println(f6.length());//5743字节

        long lastModify = f6.lastModified();//最后修改时间
        Date d = new Date(lastModify);
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        String date = sdf.format(d);
        System.out.println(date);//2021-05-03 22:55:06

        File f7 = new File("D:\\Data\\新建文件夹\\6、2020年最新 Java零基础入门到精通【完整资料】\\00_课程引入【马士兵说】");
        String[] strList = f7.list();
        for (String s : strList){
            System.out.println(s);
        }

        System.out.println("-----------------------------------------");
        File[] fileList = f7.listFiles();
        for (File f : fileList){
            //System.out.println(f.getPath());
            System.out.println(f.getAbsolutePath());
        }
    }
}

class student implements  Serializable{
    //鼠标放student上 alt+回车 快速生成序列化版本号
    private static final long serialVersionUID = -2060760799511982385L;
}

class ObjectOutputStreamTest01{
    public static void main(String[] args) {
        ObjectOutputStream oos = null;
        try {
            oos = new ObjectOutputStream(new FileOutputStream("D:/IO/writer8.txt"));
            oos.writeObject(new String("hello world"));
            oos.flush();
        } catch (IOException e) {
            e.printStackTrace();
        }finally {
            if (oos != null) {
                try {
                    oos.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}


class ObjectInputStreamTest01{
    public static void main(String[] args) {
        ObjectInputStream ois = null;
        try {
            ois = new ObjectInputStream(new FileInputStream("D:/IO/writer8.txt"));
            Object o = ois.readObject();
            if (o instanceof String){
                String s = (String)o;
                System.out.println(s);
            }
        } catch (IOException e) {
            e.printStackTrace();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } finally {
            if (ois != null) {
                try {
                    ois.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}

class HumanBeing implements Serializable {
    private static final long serialVersionUID = 7685244183746572805L;

    private int age;
    private String name;
    private double height;
    private transient float weight;//不参与序列化，反序列化出来为默认值

    public HumanBeing() {
    }

    public HumanBeing(int age, String name, double height, float weight) {
        this.age = age;
        this.name = name;
        this.height = height;
        this.weight = weight;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public double getHeight() {
        return height;
    }

    public void setHeight(double height) {
        this.height = height;
    }

    @Override
    public String toString() {
        return "HumanBeing{" +
                "age=" + age +
                ",  + name + '\'' +
                ", height=" + height +
                ", weight=" + weight +
                '}';
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        HumanBeing that = (HumanBeing) o;
        return age == that.age &&
                Double.compare(that.height, height) == 0 &&
                Objects.equals(name, that.name);
    }
}

class ObjectOutputStreamTest02{
    public static void main(String[] args) {
        ObjectOutputStream oos = null;

        try {
            oos = new ObjectOutputStream(new FileOutputStream("D:/IO/writer9.txt"));
            HumanBeing zhangsan = new HumanBeing(18, "zhangsan", 1.78, 150.0F);
            HumanBeing lisi = new HumanBeing(18, "lisi", 1.78, 123F);
            oos.writeObject(zhangsan);
            oos.writeObject(lisi);
            oos.flush();
        } catch (IOException e) {
            e.printStackTrace();
        } finally{
            if (oos != null) {
                try {
                    oos.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}

class ObjectInputStreamTest02{
    public static void main(String[] args) {
        ObjectInputStream ois = null;

        try {
            ois = new ObjectInputStream(new FileInputStream("D:/IO/writer9.txt"));
            Object o = ois.readObject();
            if (o instanceof HumanBeing){
                HumanBeing humanbeing = (HumanBeing) o;
                System.out.println(humanbeing);
            }
            o = ois.readObject();
            if (o instanceof HumanBeing){
                HumanBeing humanbeing = (HumanBeing) o;
                System.out.println(humanbeing);
            }
        } catch (IOException e) {
            e.printStackTrace();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } finally{
            if (ois != null) {
                try {
                    ois.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}

class ObjectOutputStreamTest03{
    public static void main(String[] args) {
        ObjectOutputStream oos = null;

        try {
            oos = new ObjectOutputStream(new FileOutputStream("D:/IO/writer10.txt"));
            List<HumanBeing> list = new ArrayList<HumanBeing>();
            list.add(new HumanBeing(18, "zhangsan", 178, 190));
            list.add(new HumanBeing(18, "lisi", 128, 155));
            list.add(new HumanBeing(18, "wangwu", 118, 132));
            list.add(new HumanBeing(18, "zhaoliu", 158, 112));

            oos.writeObject(list);
            oos.flush();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (oos != null) {
                try {
                    oos.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}

class ObjectInputStreamTest03{
    public static void main(String[] args) {
        ObjectInputStream ois = null;

        try {
            ois = new ObjectInputStream(new FileInputStream("D:/IO/writer10.txt"));
            Object o = ois.readObject();
            if (o instanceof List){
                ArrayList list = (ArrayList) o;
                for(int i = 0; i < list.size(); i++){
                    System.out.println(list.get(i));
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } finally {
            if (ois != null) {
                try {
                    ois.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```