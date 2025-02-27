---
title: Java RMI
date: 2020-09-13 11:58:42
categories: 
- Java技术
tags:
- Java RMI 
---

![](https://github.com/user-attachments/assets/b970dc4d-cdfd-4f18-b8f0-fd4a6f18e115)

<!-- more -->

## Java RMI

### 什么是Java RMI

**Java远程方法调用**，即**Java RMI**（Java Remote Method Invocation）是[Java](https://zh.wikipedia.org/wiki/Java)编程语言里，一种用于实现[远程过程调用](https://zh.wikipedia.org/wiki/远程过程调用)的[应用程序编程接口](https://zh.wikipedia.org/wiki/应用程序编程接口)。它使客户机上运行的程序可以调用远程服务器上的对象。远程方法调用特性使Java编程人员能够在网络环境中分布操作。RMI全部的宗旨就是尽可能简化远程接口对象的使用。

Java RMI极大地依赖于接口。在需要创建一个远程对象的时候，程序员通过传递一个接口来隐藏底层的实现细节。客户端得到的远程对象句柄正好与本地的根代码连接，由后者负责透过网络通信。这样一来，开发者只需关心如何通过自己的接口句柄发送消息。

接口的两种常见实现方式是：最初使用[JRMP](https://zh.wikipedia.org/wiki/JRMP)（Java Remote Message Protocol，Java远程消息交换协议）实现；此外还可以用与[CORBA](https://zh.wikipedia.org/wiki/CORBA)兼容的方法实现。**RMI**一般指的是编程接口，也有时候同时包括JRMP和API（[应用程序编程接口](https://zh.wikipedia.org/wiki/应用程序编程接口)），而[RMI-IIOP](https://zh.wikipedia.org/w/index.php?title=RMI-IIOP&action=edit&redlink=1)则一般指RMI接口接管绝大部分的功能，以支持[CORBA](https://zh.wikipedia.org/wiki/CORBA)的实现。

最初的RMI API设计为通用地支持不同形式的接口实现。后来，CORBA增加了传值（pass by value）功能，以实现RMI接口。然而[RMI-IIOP](https://zh.wikipedia.org/w/index.php?title=RMI-IIOP&action=edit&redlink=1)和[JRMP](https://zh.wikipedia.org/wiki/JRMP)实现的接口并不完全一致。

所使用Java包的名字是`java.rmi`。

### RMI应用程序的概念

RMI应用程序可以分为两部分，**客户端**程序和**服务器**程序。一个**服务器**程序创建一些远程对象，使其可用于客户端的引用调用方法就可以了。一个**客户端**的远程程序申请数据服务器和调用它们的方法的对象。**存根**和**骨架**是用于与远程对象通信的两个重要对象。

### Stub（存根）

在RMI中，`Stub`是一个对象，用作客户端的网关。 所有传出的请求都通过它发送。 当客户端在存根对象上调用该方法时，将在内部执行以下操作：

1. 使用远程虚拟机建立连接。
2. 然后，它将参数传输到远程虚拟机。 这也称为法警
3. 在第二步之后，它将等待输出。
4. 现在，它读取作为输出来的值或异常。
5. 最后，它将值返回给客户端。

### Skeleton（骨架）

在RMI中，`Skeleton`是一个对象，用作服务器端的网关，所有传入的请求都通过它发送。 当服务器在`Skeleton`对象上调用该方法时，将在内部执行以下操作：

1. 读取所有参数以获取远程方法。
2. 该方法在远程对象上调用。
3. 然后，它写入并传输结果的参数。

![](https://github.com/user-attachments/assets/1fd59185-125d-4cdb-b942-1779a77b59b6)

### Stub 和 Skeleton

`Stub`充当客户端程序的网关。 它位于客户端，并与Skeleton对象通信。 它建立远程对象之间的连接并向其发送请求。

![](https://github.com/user-attachments/assets/39cce3ec-7e83-4fec-9af4-780b79eab1ca)

`Skeleton`对象驻留在服务器程序上。 它负责将请求从存根传递到远程对象。

### 创建一个简单RMI应用示例步骤

1. 定义一个远程接口。
2. 实现远程接口。
3. 创建并启动远程应用程序
4. 创建并启动客户端应用程序

### 定义一个远程接口

远程接口指定客户端可以远程调用的方法。 客户端程序与远程接口通信，而不是与实现它的类通信。 要成为远程接口，接口必须扩展`java.rmi`包的`Remote`接口。

***Remote 接口***

```java
package java.rmi;

/**
 * The <code>Remote</code> interface serves to identify interfaces whose
 * methods may be invoked from a non-local virtual machine.  Any object that
 * is a remote object must directly or indirectly implement this interface.
 * Only those methods specified in a "remote interface", an interface that
 * extends <code>java.rmi.Remote</code> are available remotely.
 *
 * <p>Implementation classes can implement any number of remote interfaces and
 * can extend other remote implementation classes.  RMI provides some
 * convenience classes that remote object implementations can extend which
 * facilitate remote object creation.  These classes are
 * <code>java.rmi.server.UnicastRemoteObject</code> and
 * <code>java.rmi.activation.Activatable</code>.
 *
 * <p>For complete details on RMI, see the <a
 * href="{@docRoot}/../specs/rmi/index.html">RMI Specification</a> which
 * describes the RMI API and system.
 *
 * @since   1.1
 * @author  Ann Wollrath
 * @see     java.rmi.server.UnicastRemoteObject
 * @see     java.rmi.activation.Activatable
 */
public interface Remote {}

```

### 创建远程接口 

***AddServiceInterface.java***

```java
import java.rmi.Remote;

/**
 * 远程接口
 * @author Mr.zxb
 * @date 2020-09-13 16:41:00
 */
public interface AddServiceInterface extends Remote {
    void sum(int a, int b);
}
```

### 实现远程接口 

为了实现远程接口，类必须扩展`UnicastRemoteObject`或使用`UnicastRemoteObject`类的`exportObject()`方法。

***Adder.java***

```java
import java.rmi.RemoteException;
import java.rmi.server.UnicastRemoteObject;

/**
 * 远程接口实现
 * @author Mr.zxb
 * @date 2020-09-13 16:43:01
 */
public class Adder implements AddServiceInterface {

    @Override
    public int sum(int a, int b) {
        return a + b;
    }
}
```

### 创建 RegistryServer服务

RMI注册表是一个放置所有服务器对象的名称空间。 每次服务器创建一个对象时，它将向RMIregistry注册该对象（使用bind() 或reBind() 方法）。 这些使用唯一的名称（称为绑定名称）注册。

要调用远程对象，客户端需要该对象的引用。 那时，客户端使用其绑定名称（使用lookup() 方法）从注册表中获取对象。

下图说明了整个过程：

![](https://github.com/user-attachments/assets/85000957-685e-45b3-968b-5e602a0b2a57)

***RegistryServer.java***

```java
import java.io.IOException;
import java.rmi.registry.LocateRegistry;

/**
 * 注册中心的实现
 *
 * @author Mr.zxb
 * @date 2020-08-16 15:17
 */
public class RegistryServer {

    public static void main(String[] args) {
        try {
            LocateRegistry.createRegistry(8000);
            System.out.println("registry start...");
            // 阻塞 RegistryServer
            System.in.read();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

### 创建AddServer服务

需要创建一个服务器应用程序并在其中托管rmi服务`Adder`。 这是使用`java.rmi.Naming`类的`rebind()`方法完成的。 `rebind`方法带有两个参数，第一个表示对象引用的名称，第二个参数是对`Adder`实例的引用。

```java
import java.rmi.AlreadyBoundException;
import java.rmi.RemoteException;
import java.rmi.registry.LocateRegistry;
import java.rmi.registry.Registry;
import java.rmi.server.UnicastRemoteObject;

/**
 * Server
 * @author Mr.zxb
 * @date 2020-09-13 16:49:15
 */
public class AddServer {
    public static void main(String[] args) {
        try {
            AddServiceInterface addService = new Adder();

            // 导出服务，使用5000端口
            AddServiceInterface skeleton = (AddServiceInterface) UnicastRemoteObject.exportObject(addService, 5000);

            // 获取 Registry
            Registry registry = LocateRegistry.getRegistry("127.0.0.1", 8000);

            // 注册 Skeleton 服务
            registry.bind("AddService", skeleton);
        } catch (RemoteException | AlreadyBoundException e) {
            e.printStackTrace();
        }
    }
}
```

### 创建客户端程序

客户端应用程序包含一个Java程序，该程序调用Naming类的`lookup()`方法。 此方法接受一个参数rmi URL，并返回对`AddServerInterface`类型的对象的引用。 所有远程方法调用都在此对象上完成。

```java
import java.rmi.NotBoundException;
import java.rmi.RemoteException;
import java.rmi.registry.LocateRegistry;
import java.rmi.registry.Registry;

/**
 * Client
 * @author Mr.zxb
 * @date 2020-09-13 16:52:32
 */
public class AddClient {
    public static void main(String[] args) {
        try {
            // 获取注册中心
            Registry registry = LocateRegistry.getRegistry("127.0.0.1", 8000);

            //  查找远程服务
            AddServiceInterface stub = (AddServiceInterface) registry.lookup("AddService");

            // 访问远程服务
            System.out.println("result = " + stub.sum(5, 5));
        } catch (NotBoundException | RemoteException e) {
            e.printStackTrace();
        }
    }
}
```

### RMI 的目标

- 为了最小化应用程序的复杂性。
- 为了保持类型安全。
- 分布式垃圾回收。
- 最小化使用本地对象和远程对象之间的差异。

### 总结

Java RMI是远程过程调用（RPC）的纯Java解决方案，用于在Java中创建分布式应用程序。Stub和Skeleton对象用于客户端和服务器端之间的通信。

