# （七） FileChannel

Java NIO中的FileChannel是一个连接到文件的通道。可以通过文件通道读写文件。

FileChannel无法设置为非阻塞模式，它总是运行在阻塞模式下。

### 打开FileChannel

在使用FileChannel之前，必须先打开它。但是，我们无法直接打开一个FileChannel，需要通过使用一个InputStream、OutputStream或RandomAccessFile来获取一个FileChannel实例。下面是通过RandomAccessFile打开FileChannel的示例：		

```java
RandomAccessFile aFile = new RandomAccessFile("data/nio-data.txt", "rw");
FileChannel inChannel = aFile.getChannel();
```

### 从FileChannel读取数据

调用多个read()方法之一从FileChannel中读取数据。如：

```java
ByteBuffer buf = ByteBuffer.allocate(48);
int bytesRead = inChannel.read(buf);
```

首先，分配一个Buffer。从FileChannel中读取的数据将被读到Buffer中。

然后，调用FileChannel.read()方法。该方法将数据从FileChannel读取到Buffer中。read()方法返回的int值表示了有多少字节被读到了Buffer中。如果返回-1，表示到了文件末尾。	

### 向FileChannel写数据

使用FileChannel.write()方法向FileChannel写数据，该方法的参数是一个Buffer。如：	

```java
String newData = "New String to write to file..." + System.currentTimeMillis();

ByteBuffer buf = ByteBuffer.allocate(48);
buf.clear();
buf.put(newData.getBytes());

buf.flip();

while(buf.hasRemaining()) {
	channel.write(buf);
}
```

注意FileChannel.write()是在while循环中调用的。因为无法保证write()方法一次能向FileChannel写入多少字节，因此需要重复调用write()方法，直到Buffer中已经没有尚未写入通道的字节。	

### 关闭FileChannel

用完FileChannel后必须将其关闭。如：

```java
channel.close();
```

### FileChannel的position方法

有时可能需要在FileChannel的某个特定位置进行数据的读/写操作。可以通过调用position()方法获取FileChannel的当前位置。

也可以通过调用position(long pos)方法设置FileChannel的当前位置。

这里有两个例子:

```java
long pos = channel.position();
channel.position(pos +123);
```

如果将位置设置在文件结束符之后，然后试图从文件通道中读取数据，读方法将返回-1 —— 文件结束标志。

如果将位置设置在文件结束符之后，然后向通道中写数据，文件将撑大到当前位置并写入数据。这可能导致“文件空洞”，磁盘上物理文件中写入的数据间有空隙。

### FileChannel的size方法

FileChannel实例的size()方法将返回该实例所关联文件的大小。如:

```java
long fileSize = channel.size();
```

### FileChannel的truncate方法

可以使用FileChannel.truncate()方法截取一个文件。截取文件时，文件将中指定长度后面的部分将被删除。如：

```java
channel.truncate(1024);
```

这个例子截取文件的前1024个字节。

### FileChannel的force方法

FileChannel.force()方法将通道里尚未写入磁盘的数据强制写到磁盘上。出于性能方面的考虑，操作系统会将数据缓存在内存中，所以无法保证写入到FileChannel里的数据一定会即时写到磁盘上。要保证这一点，需要调用force()方法。

force()方法有一个boolean类型的参数，指明是否同时将文件元数据（权限信息等）写到磁盘上。

下面的例子同时将文件数据和元数据强制写到磁盘上：

```jvaa
channel.force(true);	
```

# （八） SocketChannel

Java NIO中的SocketChannel是一个连接到TCP网络套接字的通道。可以通过以下2种方式创建SocketChannel：

1. 打开一个SocketChannel并连接到互联网上的某台服务器。
2. 一个新连接到达ServerSocketChannel时，会创建一个SocketChannel。

### 打开 SocketChannel

下面是SocketChannel的打开方式：

```java
SocketChannel socketChannel = SocketChannel.open();
socketChannel.connect(new InetSocketAddress("http://jenkov.com", 80));
```

### 关闭 SocketChannel

当用完SocketChannel之后调用SocketChannel.close()关闭SocketChannel：

```java
socketChannel.close();
```

### 从 SocketChannel 读取数据

要从SocketChannel中读取数据，调用一个read()的方法之一。以下是例子：

```java
ByteBuffer buf = ByteBuffer.allocate(48);
int bytesRead = socketChannel.read(buf);
```

首先，分配一个Buffer。从SocketChannel读取到的数据将会放到这个Buffer中。

然后，调用SocketChannel.read()。该方法将数据从SocketChannel 读到Buffer中。read()方法返回的int值表示读了多少字节进Buffer里。如果返回的是-1，表示已经读到了流的末尾（连接关闭了）。

### 写入 SocketChannel

写数据到SocketChannel用的是SocketChannel.write()方法，该方法以一个Buffer作为参数。示例如下：

```java
String newData = "New String to write to file..." + System.currentTimeMillis();

ByteBuffer buf = ByteBuffer.allocate(48);
buf.clear();
buf.put(newData.getBytes());

buf.flip();

while(buf.hasRemaining()) {
    channel.write(buf);
}
```

注意SocketChannel.write()方法的调用是在一个while循环中的。Write()方法无法保证能写多少字节到SocketChannel。所以，我们重复调用write()直到Buffer没有要写的字节为止。

### 非阻塞模式

可以设置 SocketChannel 为非阻塞模式（non-blocking mode）.设置之后，就可以在异步模式下调用connect(), read() 和write()了。

#### connect()

如果SocketChannel在非阻塞模式下，此时调用connect()，该方法可能在连接建立之前就返回了。为了确定连接是否建立，可以调用finishConnect()的方法。像这样：

```java
socketChannel.configureBlocking(false);
socketChannel.connect(new InetSocketAddress("http://jenkov.com", 80));

while(! socketChannel.finishConnect() ){
    //wait, or do something else...
}
```

#### write()

非阻塞模式下，write()方法在尚未写出任何内容时可能就返回了。所以需要在循环中调用write()。前面已经有例子了，这里就不赘述了。

#### read()

非阻塞模式下,read()方法在尚未读取到任何数据时可能就返回了。所以需要关注它的int返回值，它会告诉你读取了多少字节。

### 非阻塞模式与选择器

非阻塞模式与选择器搭配会工作的更好，通过将一或多个SocketChannel注册到Selector，可以询问选择器哪个通道已经准备好了读取，写入等。Selector与SocketChannel的搭配使用会在后面详讲。

# （九） ServerSocketChannel

Java NIO中的 ServerSocketChannel 是一个可以监听新进来的TCP连接的通道, 就像标准IO中的ServerSocket一样。ServerSocketChannel类在 java.nio.channels包中。

这里有个例子：

```java
ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();

serverSocketChannel.socket().bind(new InetSocketAddress(9999));

while(true){
    SocketChannel socketChannel =
            serverSocketChannel.accept();

    //do something with socketChannel...
}
```

### 打开 ServerSocketChannel

通过调用 ServerSocketChannel.open() 方法来打开ServerSocketChannel.如：

```java
ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
```

### 关闭 ServerSocketChannel

通过调用ServerSocketChannel.close() 方法来关闭ServerSocketChannel. 如：

```java
serverSocketChannel.close();
```

### 监听新进来的连接

通过 ServerSocketChannel.accept() 方法监听新进来的连接。当 accept()方法返回的时候,它返回一个包含新进来的连接的 SocketChannel。因此, accept()方法会一直阻塞到有新连接到达。

通常不会仅仅只监听一个连接,在while循环中调用 accept()方法. 如下面的例子：

```java
while(true){
    SocketChannel socketChannel =
            serverSocketChannel.accept();

    //do something with socketChannel...
}
```

当然,也可以在while循环中使用除了true以外的其它退出准则。

### 非阻塞模式

ServerSocketChannel可以设置成非阻塞模式。在非阻塞模式下，accept() 方法会立刻返回，如果还没有新进来的连接,返回的将是null。 因此，需要检查返回的SocketChannel是否是null.如：

```java
ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();

serverSocketChannel.socket().bind(new InetSocketAddress(9999));
serverSocketChannel.configureBlocking(false);

while(true){
    SocketChannel socketChannel =
            serverSocketChannel.accept();

    if(socketChannel != null){
        //do something with socketChannel...
    }
}
```

# （十） Java NIO DatagramChannel

Java NIO中的DatagramChannel是一个能收发UDP包的通道。因为UDP是无连接的网络协议，所以不能像其它通道那样读取和写入。它发送和接收的是数据包。

### 打开 DatagramChannel

下面是 DatagramChannel 的打开方式：

```java
DatagramChannel channel = DatagramChannel.open();
channel.socket().bind(new InetSocketAddress(9999));
```

这个例子打开的 DatagramChannel可以在UDP端口9999上接收数据包。

### 接收数据

通过receive()方法从DatagramChannel接收数据，如：

```java
ByteBuffer buf = ByteBuffer.allocate(48);
buf.clear();
channel.receive(buf);
```

receive()方法会将接收到的数据包内容复制到指定的Buffer. 如果Buffer容不下收到的数据，多出的数据将被丢弃。

### 发送数据

通过send()方法从DatagramChannel发送数据，如:

```java
String newData = "New String to write to file..." + System.currentTimeMillis();

ByteBuffer buf = ByteBuffer.allocate(48);
buf.clear();
buf.put(newData.getBytes());
buf.flip();

int bytesSent = channel.send(buf, new InetSocketAddress("jenkov.com", 80));
```

这个例子发送一串字符到”jenkov.com”服务器的UDP端口80。 因为服务端并没有监控这个端口，所以什么也不会发生。也不会通知你发出的数据包是否已收到，因为UDP在数据传送方面没有任何保证。

### 连接到特定的地址

可以将DatagramChannel“连接”到网络中的特定地址的。由于UDP是无连接的，连接到特定地址并不会像TCP通道那样创建一个真正的连接。而是锁住DatagramChannel ，让其只能从特定地址收发数据。

这里有个例子:

```java
channel.connect(new InetSocketAddress("jenkov.com", 80));
```

当连接后，也可以使用read()和write()方法，就像在用传统的通道一样。只是在数据传送方面没有任何保证。这里有几个例子：

```java
int bytesRead = channel.read(buf);
int bytesWritten = channel.write(but);
```

# （十一） Pipe

Java NIO 管道是2个线程之间的单向数据连接。`Pipe`有一个source通道和一个sink通道。数据会被写到sink通道，从source通道读取。

这里是Pipe原理的图示：

![image-20211026153513737](https://i.loli.net/2021/10/26/ht5lT1I9RCA8e2M.png)

### 创建管道

通过`Pipe.open()`方法打开管道。例如：

```java
Pipe pipe = Pipe.open();
```

### 向管道写数据

要向管道写数据，需要访问sink通道。像这样：

```java
Pipe.SinkChannel sinkChannel = pipe.sink();
```

通过调用SinkChannel的`write()`方法，将数据写入`SinkChannel`,像这样：

```java
String newData = "New String to write to file..." + System.currentTimeMillis();
ByteBuffer buf = ByteBuffer.allocate(48);
buf.clear();
buf.put(newData.getBytes());

buf.flip();

while(buf.hasRemaining()) {
    sinkChannel.write(buf);
}
```

### 从管道读取数据

从读取管道的数据，需要访问source通道，像这样：

```java
Pipe.SourceChannel sourceChannel = pipe.source();
```

调用source通道的`read()`方法来读取数据，像这样：

```java
ByteBuffer buf = ByteBuffer.allocate(48);
int bytesRead = sourceChannel.read(buf);
```

`read()`方法返回的int值会告诉我们多少字节被读进了缓冲区。

# （十二） Java NIO与IO

当学习了Java NIO和IO的API后，一个问题马上涌入脑海：

我应该何时使用IO，何时使用NIO呢？在本文中，我会尽量清晰地解析Java NIO和IO的差异、它们的使用场景，以及它们如何影响您的代码设计。

## Java NIO和IO的主要区别

下表总结了Java NIO和IO之间的主要差别，我会更详细地描述表中每部分的差异。

```
IO                NIO
面向流            面向缓冲
阻塞IO            非阻塞IO
无                选择器
```

### **面向流与面向缓冲**

Java NIO和IO之间第一个最大的区别是，IO是面向流的，NIO是面向缓冲区的。 Java IO面向流意味着每次从流中读一个或多个字节，直至读取所有字节，它们没有被缓存在任何地方。此外，它不能前后移动流中的数据。如果需要前后移动从流中读取的数据，需要先将它缓存到一个缓冲区。 Java NIO的缓冲导向方法略有不同。数据读取到一个它稍后处理的缓冲区，需要时可在缓冲区中前后移动。这就增加了处理过程中的灵活性。但是，还需要检查是否该缓冲区中包含所有您需要处理的数据。而且，需确保当更多的数据读入缓冲区时，不要覆盖缓冲区里尚未处理的数据。

### **阻塞与非阻塞IO**

Java IO的各种流是阻塞的。这意味着，当一个线程调用read() 或 write()时，该线程被阻塞，直到有一些数据被读取，或数据完全写入。该线程在此期间不能再干任何事情了。 Java NIO的非阻塞模式，使一个线程从某通道发送请求读取数据，但是它仅能得到目前可用的数据，如果目前没有数据可用时，就什么都不会获取。而不是保持线程阻塞，所以直至数据变的可以读取之前，该线程可以继续做其他的事情。 非阻塞写也是如此。一个线程请求写入一些数据到某通道，但不需要等待它完全写入，这个线程同时可以去做别的事情。 线程通常将非阻塞IO的空闲时间用于在其它通道上执行IO操作，所以一个单独的线程现在可以管理多个输入和输出通道（channel）。

### **选择器（**Selectors**）**

Java NIO的选择器允许一个单独的线程来监视多个输入通道，你可以注册多个通道使用一个选择器，然后使用一个单独的线程来“选择”通道：这些通道里已经有可以处理的输入，或者选择已准备写入的通道。这种选择机制，使得一个单独的线程很容易来管理多个通道。

## NIO和IO如何影响应用程序的设计

无论您选择IO或NIO工具箱，可能会影响您应用程序设计的以下几个方面：

1.  对NIO或IO类的API调用。
2. 数据处理。
3. 用来处理数据的线程数。

### **API调用**

当然，使用NIO的API调用时看起来与使用IO时有所不同，但这并不意外，因为并不是仅从一个InputStream逐字节读取，而是数据必须先读入缓冲区再处理。

### 数据处理

使用纯粹的NIO设计相较IO设计，数据处理也受到影响。

在IO设计中，我们从InputStream或 Reader逐字节读取数据。假设你正在处理一基于行的文本数据流，例如：

```
Name: Anna
Age: 25
Email: anna@mailserver.com
Phone: 1234567890
```

该文本行的流可以这样处理：
InputStream input = … ; // get the InputStream from the client socket

```java
BufferedReader reader = new BufferedReader(new InputStreamReader(input));

String nameLine   = reader.readLine();
String ageLine    = reader.readLine();
String emailLine  = reader.readLine();
String phoneLine  = reader.readLine();
```

请注意处理状态由程序执行多久决定。换句话说，一旦reader.readLine()方法返回，你就知道肯定文本行就已读完， readline()阻塞直到整行读完，这就是原因。你也知道此行包含名称；同样，第二个readline()调用返回的时候，你知道这行包含年龄等。 正如你可以看到，该处理程序仅在有新数据读入时运行，并知道每步的数据是什么。一旦正在运行的线程已处理过读入的某些数据，该线程不会再回退数据（大多如此）。下图也说明了这条原则：



![image-20211026153531182](https://i.loli.net/2021/10/26/UZNro1A7YKLHE3w.png)

​                                                                                                           （**Java IO: 从一个阻塞的流中读数据**）

​                                                                                  

 而一个NIO的实现会有所不同，下面是一个简单的例子：

```java
ByteBuffer buffer = ByteBuffer.allocate(48);

int bytesRead = inChannel.read(buffer);
```

注意第二行，从通道读取字节到ByteBuffer。当这个方法调用返回时，你不知道你所需的所有数据是否在缓冲区内。你所知道的是，该缓冲区包含一些字节，这使得处理有点困难。
假设第一次 read(buffer)调用后，读入缓冲区的数据只有半行，例如，“Name:An”，你能处理数据吗？显然不能，需要等待，直到整行数据读入缓存，在此之前，对数据的任何处理毫无意义。

所以，你怎么知道是否该缓冲区包含足够的数据可以处理呢？好了，你不知道。发现的方法只能查看缓冲区中的数据。其结果是，在你知道所有数据都在缓冲区里之前，你必须检查几次缓冲区的数据。这不仅效率低下，而且可以使程序设计方案杂乱不堪。例如：

```java
ByteBuffer buffer = ByteBuffer.allocate(48);

int bytesRead = inChannel.read(buffer);

while(! bufferFull(bytesRead) ) {

bytesRead = inChannel.read(buffer);

}
```

bufferFull()方法必须跟踪有多少数据读入缓冲区，并返回真或假，这取决于缓冲区是否已满。换句话说，如果缓冲区准备好被处理，那么表示缓冲区满了。

bufferFull()方法扫描缓冲区，但必须保持在bufferFull（）方法被调用之前状态相同。如果没有，下一个读入缓冲区的数据可能无法读到正确的位置。这是不可能的，但却是需要注意的又一问题。

如果缓冲区已满，它可以被处理。如果它不满，并且在你的实际案例中有意义，你或许能处理其中的部分数据。但是许多情况下并非如此。下图展示了“缓冲区数据循环就绪”：

![image-20211026153545581](https://i.loli.net/2021/10/26/MNe5bIg6a3BJdVU.png)

​                                                                                          **Java NIO:从一个通道里读数据，直到所有的数据都读到缓冲区里.**

## Summary

NIO可让您只使用一个（或几个）单线程管理多个通道（网络连接或文件），但付出的代价是解析数据可能会比从一个阻塞流中读取数据更复杂。

如果需要管理同时打开的成千上万个连接，这些连接每次只是发送少量的数据，例如聊天服务器，实现NIO的服务器可能是一个优势。同样，如果你需要维持许多打开的连接到其他计算机上，如P2P网络中，使用一个单独的线程来管理你所有出站连接，可能是一个优势。一个线程多个连接的设计方案如下图所示：

![image-20211026153604077](https://i.loli.net/2021/10/26/eJGCI8DvOuAFES1.png)

​                                                                                    **Java NIO: 单线程管理多个连接**

如果你有少量的连接使用非常高的带宽，一次发送大量的数据，也许典型的IO服务器实现可能非常契合。下图说明了一个典型的IO服务器设计：

![image-20211026153614869](https://i.loli.net/2021/10/26/iEvzCDejXqwo2SR.png)

​                                                                                 **Java IO: 一个典型的IO服务器设计- 一个连接通过一个线程处理.**

# Java NIO Path

Path接口是java NIO2的一部分。首次在java 7中引入。Path接口在java.nio.file包下，所以全称是java.nio.file.Path。 java中的Path表示文件系统的路径。可以指向文件或文件夹。也有相对路径和绝对路径之分。绝对路径表示从文件系统的根路径到文件或是文件夹的路径。相对路径表示从特定路径下访问指定文件或文件夹的路径。相对路径的概念可能有点迷糊。不用担心，我将在本文的后面详细介绍相关细节。

不要将文件系统的path和操作系统的环境变量path搞混淆。java.nio.file.Path接口和操作系统的path环境变量没有任何关系。

在很多方面，java.nio.file.Path接口和[java.io.File](http://tutorials.jenkov.com/java-io/file.html)有相似性，但也有一些细微的差别。在很多情况下，可以用Path来代替File类。

### 创建Path实例

为了使用java.nio.file.Path实例，必须首先创建它。可以使用Paths 类的静态方法Paths.get()来产生一个实例。以下是示例：

```java
import java.nio.file.Path;
import java.nio.file.Paths;

public class PathExample {

public static void main(String[] args) {

Path path = Paths.get("c:\\data\\myfile.txt");

}
}
```

请注意例子开头的两个import语句。想要使用Paths类和Path接口，必须首先引入相应包。其次，注意Paths.get(“c:\\data\\myfile.txt”)的用法。其使用了Paths.get方法创建了Path的实例。它是一个工厂方法。

### 创建绝对路径Path

调用传入绝对路径当做参数的Paths.get()工厂方法，就可以生成绝对路径Path。示例如下：

```java
Path path = Paths.get("c:\\data\\myfile.txt");
```

示例中的绝对路径是c:\data\myfile.txt。有两个\字符的原因是第一个\是转义字符，表示紧跟着它的字符需要被转义。\\表示需要向字符串中写入一个\字符。

上文示例的path是windows下的路径。在Unix系统(Linux,MacOS,FreeBSD等)中，上文中的path是这样的：

```java
Path path = Paths.get("/home/jakobjenkov/myfile.txt");
```

/home/jakobjenkov/myfile.txt就称作绝对路径。

如果把以/开头path的格式运行在windows系统中，系统会将其解析为相对路径。例如：

```java
/home/jakobjenkov/myfile.txt
```

将会被解析为路径是在C盘。对应的绝对路径是：

C:/home/jakobjenkov/myfile.txt

### 创建相对路径Path

相对路径指从一个已确定的路径开始到某一文件或文件夹的路径。将确定路径和相对路径拼接起来就是相对路径对应的绝对路径地址。

java NIO Path类也能使用相对路径。可以通过Paths.get(basePath, relativePath)创建一个相对路径Path。示例如下：

```
Path projects = Paths.get("d:\\data", "projects");

Path file = Paths.get("d:\\data", "projects\\a-project\\myfile.txt");
```

第一个例子创建了一个指向d:\data\projects文件夹的实例。第二个例子创建了一个指向 d:\data\projects\a-project\myfile.txt 文件的实例。

当使用相对路径的时候，可以使用如下两种特别的符号。它们是：

- .
- ..

.表示当前路径。例如，如果以如下方式创建一个相对路径：

```
Path currentDir = Paths.get(".");
System.out.println(currentDir.toAbsolutePath());
```

创建的Path实例对应的路径就是运行这段代码的项目工程目录。

如果.用在路径中，则其表示的就是当前路径下。示例：

Path currentDir = Paths.get(“d:\\data\\projects\.\a-project”);
对应的就是如下路径
d:\data\projects\a-project

..表示父类目录。示例：

Path parentDir = Paths.get(“..”);

Path对应的路径是当前运行程序目录的上级目录。

如果在path中使用..，表示上级目录的含义。例如：

```java
String path = "d:\\data\\projects\\a-project\\..\\another-project";
Path parentDir2 = Paths.get(path);
```

对应的绝对路径地址为：

d:\data\projects\another-project

在a-project目录后面的..符号，将指向的目录修改为projects目录，因此，最终path指向another-project目录。

.和..都可以在Paths.get()的双形参方法中使用。示例：

```
Path path1 = Paths.get("d:\\data\\projects", ".\\a-project");

Path path2 = Paths.get("d:\\data\\projects\\a-project",
 "..\\another-project");
```

下面介绍NIO 的Path类有关相对路径的其他使用方法。

### Path.normalize()

Path 的normalize()方法可以标准化路径。标准化的含义是路径中的.和..都被去掉，指向真正的路径目录地址。下面是Path.normalize()示例：

```
String originalPath =
 "d:\\data\\projects\\a-project\\..\\another-project";

Path path1 = Paths.get(originalPath);
System.out.println("path1 = " + path1);

Path path2 = path1.normalize();
System.out.println("path2 = " + path2);
```

上文示例，首先创建了一个包含..字符的路径地址。之后输出此路径。

之后，调用normalize方法，返回一个新的path对象。输出新对象的路径。

输出结果如下：

```
path1 = d:\data\projects\a-project\..\another-project
path2 = d:\data\projects\another-project
```

如你所见，标准化后的路径不再包含 a-project\..部分，因为它是多余的。

# Java NIO AsynchronousFileChannel

我们可以使用*AsynchronousFileChannel*提供的静态方法 *open()* 创建它。示例代码如下：

```java
Path path = Paths.get("data/test.xml");
AsynchronousFileChannel fileChannel = AsynchronousFileChannel.open(path, StandardOpenOption.READ);
```

第一个参数是一个 *[PATH](http://ifeve.com/java-nio-path-2/)* 的对像实例，它指向了那个与 *AsynchronousFileChannel* 相关联的文件。

第二个参数是一个或多个操作选项，它决定了 *AsynchronousFileChannel* 将对目标文件做何种操作。示例代码中我们使用了 *StandardOpenOption.READ* ，它表明我们将要对目标文件进行读操作。

#### **读取数据**

*AsynchronousFileChannel* 提供了两种读取数据的方式，都是调用它本身的 *read()* 方法。下面将对两种方式进行介绍。

#### **使用Futrue读取数据**

第一种反式是调用 *AsynchronousFileChannel* 的 *read()* 方法，该方法反回一个 *Future* 类型的对象。

```java
Future operation = fileChannelread(buffer, 0);
```



第一个参数是*ByteBuffer*，从 *AsynchronousFileChannel* 中读取的数据先写入这个 *ByteBuffer* 。

第二个参数表示从文件读取数据的开始位置。

此 *read()* 方法会立即返回，即使整个读的过程还没有完全结束。我们可以通过`operation.isDone()`来检查读取是否完成。这里的 *operation* 是上面调用 *read()* 方法返回的 *Future* 类型的实例。下面是一段详细的代码示例：

```java
AsynchronousFileChannel fileChannel = 
    AsynchronousFileChannel.open(path, StandardOpenOption.READ);

ByteBuffer buffer = ByteBuffer.allocate(1024);
long position = 0;

Future<Integer> operation = fileChannel.read(buffer, position);

while(!operation.isDone());

buffer.flip();
byte[] data = new byte[buffer.limit()];
buffer.get(data);
System.out.println(new String(data));
buffer.clear();
```

上面的程序首先创建了一个 AsynchronousFileChannel 对象，然后调用它的read()方法返回一个Future。其中read()方法需要两个参数，一个是ByteBuffer，另一个是读取文件的开始位置。然后通过循环调用isDone() 方法检测读取过程是否完成，完成后 isDone()方法将返回true。尽管这样让cpu空转了一会，但是我们还是应该等读取操作完成后再进行后续的步骤。

一旦读取完成，数据被存储到ByteBuffer，然后将数据转化为字符串既而输出。

#### **使用CompletionHandler读取数据**

第二种读取数据的方式是调用AsynchronousFileChannel 的另一个重载 read() 方法，改方法需要一个CompletionHandler 作为参数。下面是代码示例：

```java
fileChannel.read(buffer, position, buffer, new CompletionHandler<Integer, ByteBuffer>() {
    @Override
    public void completed(Integer result, ByteBuffer attachment) {
        System.out.println("result = " + result);

        attachment.flip();
        byte[] data = new byte[attachment.limit()];
        attachment.get(data);
        System.out.println(new String(data));
        attachment.clear();
    }

    @Override
    public void failed(Throwable exc, ByteBuffer attachment) {

    }
});
```

一旦读取操作完成，*CompletionHandler*的 *complete()* 方法将会被调用。它的第一个参数是个 *Integer*类型，表示读取的字节数。第二个参数 *attachment* 是 *ByteBuffer* 类型的，用来存储读取的数据。它其实就是由 *read()* 方法的第三个参数。当前示例中，我们选用 *ByteBuffer* 来存储数据，其实我们也可以选用其他的类型。

读取失败的时候，*CompletionHandler*的 *failed()*方法会被调用。

#### **写入数据**

就像读取一样，我们同样有两种方式向 *AsynchronousFileChannel* 写入数据。我们可以调用它的2个重载的 *write()* 方法。下面我们将分别加以介绍。

#### **使用Future读取数据**

AsynchronousFileChannel也可以异步写入数据。下面是一个完整的写入示例：

```java
Path path = Paths.get("data/test-write.txt");
AsynchronousFileChannel fileChannel = 
    AsynchronousFileChannel.open(path, StandardOpenOption.WRITE);

ByteBuffer buffer = ByteBuffer.allocate(1024);
long position = 0;

buffer.put("test data".getBytes());
buffer.flip();

Future<Integer> operation = fileChannel.write(buffer, position);
buffer.clear();

while(!operation.isDone());

System.out.println("Write done");
```

首先实例化一个写入模式的 *AsynchronousFileChannel*, 然后创建一个 *ByteBuffer* 并写入一些数据。再然后将数据写入文件。最后，检查返回的 *Future*，看是否写入完成。

注意，写入目标文件要提前创建好，如果它不存在的话，*writh()* 方法会抛出一个 *java.nio.file.NoSuchFileException*。

我们可以用以下方式来解决这一问题：

```java
if(!Files.exists(path)){
    Files.createFile(path);
}
```

**使用CompletionHandler写入数据**
我们也可以使用 *CompletionHandler*代替*Future*向*AsynchronousFileChannel*写入数据，这种方式可以更加直接的知道写入过程是否完成。下面是示例程序

```java
Path path = Paths.get("data/test-write.txt");
if(!Files.exists(path)){
    Files.createFile(path);
}
AsynchronousFileChannel fileChannel = 
    AsynchronousFileChannel.open(path, StandardOpenOption.WRITE);

ByteBuffer buffer = ByteBuffer.allocate(1024);
long position = 0;

buffer.put("test data".getBytes());
buffer.flip();

fileChannel.write(buffer, position, buffer, new CompletionHandler<Integer, ByteBuffer>() {

    @Override
    public void completed(Integer result, ByteBuffer attachment) {
        System.out.println("bytes written: " + result);
    }

    @Override
    public void failed(Throwable exc, ByteBuffer attachment) {
        System.out.println("Write failed");
        exc.printStackTrace();
    }
});
```

当写入程序完成时，*CompletionHandler*的*completed()*方法将会被调用，相反的如果写入失败则会调用*failed()*方法。

要留意*CompletionHandler*的方法的参数 *attachemnt*是怎么使用的。
