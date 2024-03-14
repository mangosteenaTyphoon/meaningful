> IO流是实现JAVA输入和输出的基础，IO流是一个重要的方面，在读和写方面，都可以做出很大的优化。这里是Java在优化上的一个大的痛点。

## 文件的操作

在学习IO流之前，我们先来看一下Java中对于创建文件的操作，只要有了文件，那么流使用的对象也就有了。文件是**保存数据的地方**，比如word文档，txt文件，excel文件，都是文件。它能保存图片、视频、声音等。**文件在程序中是以流的形式来操作的。**

### 常用的文件操作

#### 构造器

以pathname为路径创建File对象，可以是绝对路径或者相对路径，如果pathname是相对路径，则默认的当前路径在系统属性user.dir中存储。

```Java
public File(String pathname)
```

以parent为父路径，child为子路径创建File对象。

```Java
public File(String parent,String child)
```

根据一个父File对象和子文件路径创建File对象

```Java
public File(File parent,String child)
```

#### 常用方法

通常的获取操作

| 方法                            | 说明                                           |
| ------------------------------- | ---------------------------------------------- |
| public String getAbsolutePath() | 获取绝对路径                                   |
| public String getPath()         | 获取路径                                       |
| public String getName()         | 获取名称                                       |
| public String getParent()       | 获取上层文件目录路径。若无，返回null           |
| public long length()            | 获取文件长度（即：字节数）。不能获取目录的长度 |
| public long lastModified()      | 获取最后一次的修改时间，毫秒值                 |
| public String[] list()          | 获取指定目录下的所有文件或者文件目录的名称数组 |
| public File[] listFiles()       | 获取指定目录下的所有文件或者文件目录的File数组 |

重命名操作

| 方法                               | 说明                         |
| ---------------------------------- | ---------------------------- |
| public boolean renameTo(File dest) | 把文件重命名为指定的文件路径 |

常用的判断操作

| 方法                         | 说明               |
| ---------------------------- | ------------------ |
| public boolean isDirectory() | 判断是否是文件目录 |
| public boolean isFile()      | 判断是否是文件     |
| public boolean exists()      | 判断是否存在       |
| public boolean canRead()     | 判断是否可读       |
| public boolean canWrite()    | 判断是否可写       |
| public boolean isHidden()    | 判断是否隐藏       |

常用的创建文件的操作

| 方法                           | 说明                                                         |
| ------------------------------ | ------------------------------------------------------------ |
| public boolean createNewFile() | 创建文件。若文件存在，则不创建，返回false                    |
| public boolean mkdir()         | 创建文件目录。如果此文件目录存在，就不创建了；如果此文件目录的上层目录不存在，也不创建。 |
| public boolean mkdirs()        | 创建文件目录。如果上层文件目录不存在，一并创建               |

文件的删除功能

| 方法                    | 说明               |
| ----------------------- | ------------------ |
| public boolean delete() | 删除文件或者文件夹 |

删除注意事项：Java中的删除不走回收站。要删除一个文件目录，请注意该文件目录内不能包含文件或者文件目录。

### 实践操作

在电脑D盘上创建目录名为myBook的目录，在此目录下创建两个名为幻城和暮光之城两个目录。并且在目录中分别创建名为book.txt的文件

```Java
 public static void main(String[] args) throws IOException {
        //创建目录在d盘
        Scanner sc = new Scanner(System.in);
        String dir="D:\\myBook";

        File file = new File(dir);
        if(!file.exists()){
            file.mkdir();
            String s=null;
            do {
                System.out.println("请输入要创建父目录的名称：");
                s = sc.next();
                boolean flag = new File(dir + "\\" + s).mkdir();
                if(flag){
                    System.out.println("创建成功");
                    System.out.println("请输入你要创建文件的名称");
                    String pathFile = sc.next();
                    boolean flags = new File(dir + "\\" + s + "\\" + pathFile).createNewFile();
                    if(flags) System.out.println("文件创建成功");
                }
            }while (!s.equals("不创建了"));

        }

    }
```

## IO流的体系结构

在 IO 流中有很多不同的流，因此我们需要根据不同的状况去使用不同的流。

![img](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/image.png)

### 根据数据的流向分类

有输入流和输出流两种操作方式。

- 输入流

  输入：读取外部数据（磁盘、光盘等存储设备的数据）到程序（内存）中

- 输出流

  输出：把程序(内存)中的内容输出到磁盘、光盘等存储设备中

### 根据数据的类型分类

- 字节流 数据流中最小的数据单元是字节
  - 字节输入流 读取数据 InputStream
  - 字节输出流 写入数据 OutputStream
- 字符流 数据流中最小的数据单元是字符， Java中的字符是Unicode编码，一个字符占用两个字节
  - 字符输入流 读取数据 Reader
  - 字符输出流 写入数据 Writer

### 关于字符流和字节流的使用区别

**字符流通常用来操作文办文件：**

包括只能操作普通文本文件。最常见的文本文件：.txt，.java，.c，.cpp 等语言的源代码。尤其注意.doc,excel,ppt这些不是文本文件。

**字节流**可以操作二进制文件。

## InputStream类

inputStream是字节输入流的接口类，他下面有关于各种Java处理字节输入流的实现类。

![img](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/image.png)

### FileInputStream类

我们先来看看FileInputStream类，此类为Java字节流文件的常用处理类

#### 构造器

| 方法                                  | 说明                                                         |
| ------------------------------------- | ------------------------------------------------------------ |
| FileInputStream(File file)            | 通过打开一个到实际文件的连接来创建一个 FileInputStream，该文件通过文件系统中的 File 对象 file 指定。 |
| FileInputStream(FileDescriptor fdObj) | 通过使用文件描述符 fdObj 创建一个 FileInputStream，该文件描述符表示到文件系统中某个实际文件的现有连接。 |
| FileInputStream(String path)          | 通过打开一个到实际文件的连接来创建一个 FileInputStream，该文件通过文件系统中的路径名 path指定 |

#### 常用方法

![img](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/Image.png)

#### 实践操作

使用字节输入流来读取文件并且输出到控制台

单个字节读取：

```Java
 public static void main(String[] args)  {
        FileInputStream stream=null;
        try {
            stream = new FileInputStream("D:\\myBook\\幻城\\book.txt");
            int data ;
            while ((data=stream.read())!=-1){
                System.out.print((char)data);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }finally {
            if(stream!=null){
                try {
                    stream.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
```

注意其中代码细节，先创建一个空的字符输入流，用来关闭流做好准备。

字节数组读取

```Java
public static void inputStreamByArray(){
        FileInputStream stream=null;
        try {
            stream = new FileInputStream("D:\\myBook\\幻城\\book.txt");
            byte[] cbuf=new byte[5];
            int len;
            while ((len=stream.read(cbuf))!=-1){
                String str=new String(cbuf,0,len);
                System.out.print(str);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }finally {
            if(stream!=null){
                try {
                    stream.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
```

### BufferedInputStream类

BufferedInputStream:缓冲字节输出流是一个高级流(处理流)，与其他低级流配合使用。其实简单的说就是把其他流套里面就直接用了。

`BufferedInputStream`继承于`FilterInputStream`，提供缓冲输入流功能。缓冲输入流相对于普通输入流的优势是，它提供了一个缓冲数组，每次调用read方法的时候，它首先尝试从缓冲区里读取数据，若读取失败（缓冲区无可读数据），则选择从物理数据源（譬如文件）读取新数据（这里会尝试尽可能读取多的字节）放入到缓冲区中，最后再将缓冲区中的内容部分或全部返回给用户.由于从缓冲区里读取数据远比直接从物理数据源（譬如文件）读取速度快。

#### 构造器

| 方法                                                | 说明                                                         |
| --------------------------------------------------- | ------------------------------------------------------------ |
| BufferedInputStream(InputStream in)                 | 创建一个 BufferedInputStream 并保存其参数，即输入流 in，以便将来使用。创建一个内部缓冲区数组并将其存储在 buf 中,该buf的大小默认为8192 |
| public BufferedInputStream(InputStream in,int size) | 创建具有指定缓冲区大小的 BufferedInputStream 并保存其参数，即输入流 in，以便将来使用。创建一个长度为 size 的内部缓冲区数组并将其存储在 buf 中 |

#### 常用方法

| 方法                                       | 说明                                                         |
| ------------------------------------------ | ------------------------------------------------------------ |
| public int read()                          | 从该输入流中读取一个字节                                     |
| public int read(byte[] b,int off,int len); | 从此字节输入流中给定偏移量处开始将各字节读取到指定的 byte 数组中。 |

#### 实践举例

从文件中读取数据

```Java
    public static void bufferedInputStreamTest(){
        FileInputStream stream=null;
        BufferedInputStream bufferedInputStream=null;
        try {
            stream = new FileInputStream("D:\\myBook\\幻城\\book.txt");
            bufferedInputStream = new BufferedInputStream(stream);
            byte[] cbuf=new byte[1024];
            int len;
            while ((len=bufferedInputStream.read(cbuf))!=-1){
                System.out.println(len);
                String str=new String(cbuf,0,len);
                System.out.println(str);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }finally {
            if(stream!=null){
                try {
                    bufferedInputStream.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
```

## OutputStream类

此抽象类是表示输出字节流的所有类的超类。输出流接受输出字节并将这些字节发送到某个接收器。

![img](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/image.png)

### FileOutputStream类

我们先说FileOutputStream，他是处理文件字符输入流的实现类。也是低级处理流。

#### 构造方法

| 方法                                        | 说明                                                         |
| ------------------------------------------- | ------------------------------------------------------------ |
| FileOutputStream(String name)               | 创建一个向具有指定名称的文件中写入数据的输出文件流。         |
| FileOutputStream(File file)                 | 创建一个向指定 File 对象表示的文件中写入数据的文件输出流。   |
| FileOutputStream(File file, boolean append) | 追加创建一个向指定 File 对象表示的文件中写入数据的文件输出流。 |

#### 常用方法

| 方法                                               | 说明                     |
| -------------------------------------------------- | ------------------------ |
| void write(char[] cbuf)                            | 写入字符数组。           |
| abstract void write(char[] cbuf, int off, int len) | 写入字符数组的某一部分。 |
| void write(int c)                                  | 写入单个字符。           |
| void write(String str)                             | 写入字符串。             |
| void write(String str, int off, int len)           | 写入字符串的某一部分。   |
| abstract void close()                              | 关闭此流，但要先刷新它。 |