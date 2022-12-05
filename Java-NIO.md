---
title: "Java NIO"
date: 2021-12-16T08:37:55Z
draft: false
tags: ["java","nio","netty"]
categories: ["note"]
---

Four main IO models

1. Blocking IO
2. Non-blocking IO (NIO)
3. IO Multiplexing -> JAVA NIO (New IO)
4. Asynchronous IO

Three main components of Java NIO

1. Channel
2. Buffer
3. Selector

## Buffer

Eight kinds of buffer,including ByteBuffer, CharBuffer, DoubleBuffer, FloatBuffer, IntBUffer, LongBuffer, ShortBuffer, and MappedByBuffer.

```java
    // Buffer.java
    // Invariants: mark <= position <= limit <= capacity
    private int mark = -1;
    private int position = 0;
    private int limit;
    private int capacity;
```

### Methods

1. allocate()
2. put()
3. flip()
4. get()
5. rewind()
6. mark()
7. reset()
8. clear()

## Channel

Four main channels, including FileChannel, SocketChannel, ServerSocketChannel, and DatagramChannel.

1. FileChannel: file channel
2. SocketChannel: TCP socket
3. ServerSocketChannel: Listen TCP socket
4. DatagramChannel: UDP

### FileChannel

get FileChannel from FileInputStream, FileOutputStream, and RandomAccessFile.

`int read(ByteBuffer buf);` return the count of bytes read.

`int write(ByteBuffer buf)` return the count of bytes written.

`void close()`

`void force(boolean b)` force to refresh to disk

### SocketChannel & ServerSocketChannel

`configureBlocking` false non-blocking, true blocking

#### get socket channel

```java
//client
SocketChannel socketChannel = SocketChannel.open();
socketChannel.configureBlocking(false);
socketChannel.connect(new InetSocketAddress("127.0.0.1",89));

while(!socketChannel.finishConnect()){
    //wait
}

//server
ServerSocketChannel server = (ServerSocketChannel) key.channel();
SocketChannel socketChannel = server.accept();
socketChannel.configureBlocking(false);
```

#### read SocketChannel data*

```java
ByteBuffer buffer = ByteBuffer.allocate(1024);

// >0 length of data read
// 0 non data
// -1 finish flag
int bytesRead = socketChannel.read(buf);
```

#### write SocketChannel data

```java
buffer.flip();
socketChannel.write(buffer);
```

#### close SocketChannel

```java
socketChannel.shutdownOutput();
socketChannel.close();
```

### DatagramChannel

#### get datagramChannel

```java
DatagramChannel channel = DatagramChannel.open();
datagramChannel.configureBlocking(false);
//accept data
channel.socket().bind(new InetSocketAddress(18080));
```

#### read datagramChannel data

```java
ByteBuffer buffer = ByteBuffer.allocate(1024);
SocketAddress clientAddress = datagramChannel.receive(buffer);
```

#### write datagramChannel data

```java
buffer.flip();
channel.send(buffer, new InetSocketAddress("127.0.0.1",90));
buffer.clear();
```

#### close datagramChannel

`channel.close();`

## NIO Selector

select model

IO event type

1. SelectionKey.OP_READ (1 << 0)
2. SelectionKey.OP_WRITE (1 << 1)
3. SelectionKey.OP_CONNECT (1 << 2)
4. SelectionKey.OP_ACCEPT (1 << 3)

monitor multiple events by using '\&'

only classes extends `SelectableChannel` can be selected (FileChannel cannot be selected)

### Selector usage process

```java
Selector selector = Selector.open();

serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);

While (selector.select() > 0){
    Set selectedKeys = selector.selectedKeys();
    Iterator keyIterator = selectedKeys.iterator();
    while(keyIterator.hasNext()){
        SelectionKey key = keyIterator.next();
        
        // do something

    }
}
```

select() had multiple overload

1. select() blocking
2. select(long timeout)
3. selectNow() non-blocking

## Reactor

`attach()` attach a handler
`attachment()` get the handler

### One thread reactor

when is OP_ACCEPT, register and attach the key.  
when is OP_READ, do something.

### Multiple threads reactor

use thread pool, separate IO event thread and process thread.

#### IO event thread

create the same counts of `Selector` and `Reactor Thread`.

every thread works for a Selector to search and select.

#### Tack thread

use thread pool to process tasks.

## Future

### Callable

Runnable with return value

```java
public  interface Callable<V>{
    V call() throw Exception;
}
```

### FutureTask

`FutureTask` implements `RunnableFuture` extends `Runnable` and `Future`.

```java
public FutureTask(Callable<V> callable){
    if(callable == null)
        throw new NullPointerException();
    this.callable = callable;
    this.state = new;
}
```

FutureTask class can be used as Thread class's target (Runnable), execute async.

FutureTask has an attribute `outcome` to save result. There is a method `get()` to get outcome.

### Future interface

```java
public interface Future<V>{
    boolean cancel(boolean mayInterruptZRunning);
    boolean isCancelled();
    boolean isDone();
    V get() throws InterruptedException, ExecutionException;
    V get(long timeout, TimeUnit unit) throws InterruptedException, ExecutionException, TimeoutException;
}
```
