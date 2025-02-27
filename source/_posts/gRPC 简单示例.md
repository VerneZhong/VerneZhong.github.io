---
title: gRPC 简单示例
date: 2020-08-12 20:27:17
categories: 
- Java技术
tags:
- gRPC
- RPC框架
---

![](https://github.com/user-attachments/assets/20fd0982-4922-4657-850b-9efd7cf6c37e)

<!-- more -->

## gRPC  高性能的RPC框架简单示例

### 什么是 gRPC ？

gRPC由 Google 开发，是一款跨语言、跨平台、开源的远程过程调用的(RPC)高性能框架。

在gRPC中，客户端应用程序可以直接在其他计算机上的服务器应用程序上调用方法，就好像它是本地对象一样，从而使您更轻松地创建分布式应用程序和服务。 与许多RPC系统一样，gRPC围绕定义服务的思想，指定可通过其参数和返回类型远程调用的方法。 在服务器端，服务器实现此接口并运行gRPC服务器以处理客户端调用。 在客户端，客户端具有一个存根（在某些语言中仅称为客户端），提供与服务器相同的方法。

### gRPC  示意图

![image-20200812203844651](https://github.com/user-attachments/assets/8267be3c-eb0e-46e5-9db4-85630e66b733)

gRPC 客户端和服务端可以在多种环境中运行和交互 - 从 Google 内部的服务器到你自己的笔记本，并且可以用任何 gRPC [支持的语言](http://doc.oschina.net/grpc?t=58008#quickstart)来编写。所以，你可以很容易地用 Java 创建一个 gRPC 服务端，用 Go、Java、Python、Ruby等语言来创建客户端。此外，Google 最新 API 将有 gRPC 版本的接口，使你很容易地将 Google 的功能集成到你的应用里。

### 使用  Protocol buffers

gRPC 默认使用 *protocol buffers*，这是 Google 开源的一套成熟的结构数据序列化机制（当然也可以使用其他数据格式如 JSON）。正如你将在下方例子里所看到的，使用 *proto files* 创建 gRPC 服务，用 protocol buffers 消息类型来定义方法参数和返回类型。

### 开始创建第一个 HelloRpc.proto  文件吧

```protobuf
syntax = "proto3";

option java_multiple_files = true;
option java_package = "com.chivalry.grpc.examples";
option java_outer_classname = "HelloRpc";

package examples;

// hello service
service HelloRpcService {
    rpc sayHello(HelloRpcRequest) returns (HelloRpcReply) {}
}

// gRPC request
message HelloRpcRequest {
    string name = 1;
}

// gRPC response
message HelloRpcReply {
   string name = 1;
}
```

### Maven  依赖

```xml
<properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <grpc.version>1.31.0</grpc.version><!-- CURRENT_GRPC_VERSION -->
        <protobuf.version>3.12.0</protobuf.version>
        <protoc.version>3.12.0</protoc.version>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
    </properties>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>io.grpc</groupId>
                <artifactId>grpc-bom</artifactId>
                <version>${grpc.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <dependencies>
        <dependency>
            <groupId>com.google.protobuf</groupId>
            <artifactId>protobuf-java-util</artifactId>
            <version>${protobuf.version}</version>
        </dependency>
        <dependency>
            <groupId>io.grpc</groupId>
            <artifactId>grpc-netty-shaded</artifactId>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>io.grpc</groupId>
            <artifactId>grpc-protobuf</artifactId>
        </dependency>
        <dependency>
            <groupId>io.grpc</groupId>
            <artifactId>grpc-stub</artifactId>
        </dependency>
        <dependency>
            <groupId>org.apache.tomcat</groupId>
            <artifactId>annotations-api</artifactId>
            <version>6.0.53</version>
            <scope>provided</scope> <!-- not needed at runtime -->
        </dependency>
        <dependency>
            <groupId>io.grpc</groupId>
            <artifactId>grpc-testing</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>com.google.errorprone</groupId>
            <artifactId>error_prone_annotations</artifactId>
            <version>2.3.4</version> <!-- prefer to use 2.3.3 or later -->
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.mockito</groupId>
            <artifactId>mockito-core</artifactId>
            <version>2.28.2</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.0</version>
            <scope>provided</scope>
        </dependency>
    </dependencies>

    <build>
        <extensions>
            <extension>
                <groupId>kr.motd.maven</groupId>
                <artifactId>os-maven-plugin</artifactId>
                <version>1.6.2</version>
            </extension>
        </extensions>
        <plugins>
            <!-- gRPC 提供的生成Java代码的插件 -->
            <plugin>
                <groupId>org.xolstice.maven.plugins</groupId>
                <artifactId>protobuf-maven-plugin</artifactId>
                <version>0.6.1</version>
                <configuration>
                    <protocArtifact>com.google.protobuf:protoc:${protoc.version}:exe:${os.detected.classifier}</protocArtifact>
                    <pluginId>grpc-java</pluginId>
                    <pluginArtifact>io.grpc:protoc-gen-grpc-java:${grpc.version}:exe:${os.detected.classifier}</pluginArtifact>
                </configuration>
                <executions>
                    <execution>
                        <goals>
                            <goal>compile</goal>
                            <goal>compile-custom</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-enforcer-plugin</artifactId>
                <version>1.4.1</version>
                <executions>
                    <execution>
                        <id>enforce</id>
                        <goals>
                            <goal>enforce</goal>
                        </goals>
                        <configuration>
                            <rules>
                                <requireUpperBoundDeps/>
                            </rules>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
```

### 生成  RPC  Java 代码

```shell
mvn clean package
```

![WX20200814-151102@2x](https://github.com/user-attachments/assets/dc430f7d-b2d0-46cc-97c8-f9818429fa6f)

### 创建  RpcServer  端

```java
/**
 * Hello Rpc Service examples
 * @author Mr.zxb
 * @date 2020-08-12 19:54:37
 */
public class HelloRpcServer {

    private static final Logger logger = Logger.getLogger(HelloRpcServer.class.getName());

    // 服务端监听端口
    private final int port = 8000;

    private final Server server;

    public HelloRpcServer() throws IOException {
        this.server = ServerBuilder.forPort(port)
                .addService(new HelloRpcServerImpl())
                .build();
    }

    private void start() throws IOException {
        this.server.start();
        logger.info("gRPC Server started, listening on " + port);

        // Jvm shutdown sync off gRPC server
        Runtime.getRuntime().addShutdownHook(new Thread(() -> {
            System.err.println("*** shutting down gRPC server since JVM is shutting down");
            try {
                HelloRpcServer.this.stop();
            } catch (InterruptedException e) {
                e.printStackTrace(System.err);
            }
            System.err.println("*** server shut down");
        }));
    }

    private void stop() throws InterruptedException {
        if (server != null) {
            server.awaitTermination(30, TimeUnit.SECONDS);
        }
    }

    /**
     * Await termination on the main thread since the grpc library uses daemon threads.
     */
    private void blockUntilShutdown() throws InterruptedException {
        if (server != null) {
            server.awaitTermination();
        }
    }

     /**
     * HelloRpcService implement
     */
    static class HelloRpcServerImpl extends HelloRpcServiceGrpc.HelloRpcServiceImplBase {
        @Override
        public void sayHello(HelloRpcRequest request, StreamObserver<HelloRpcReply> responseObserver) {
            HelloRpcReply reply = HelloRpcReply.newBuilder().setName("Hello " + request.getName()).build();
            // StreamObserver 应答观察者，一个特殊的接口，服务器用应答来调用它
            // HelloReply 返回给客户端，然后表明我们已经完成了对 RPC 的处理。
            responseObserver.onNext(reply);
            responseObserver.onCompleted();
        }
    }

    public static void main(String[] args) throws IOException, InterruptedException {
        //  Main launches the server from the command line.
        HelloRpcServer rpcService = new HelloRpcServer();
        rpcService.start();
        rpcService.blockUntilShutdown();
    }
}
```

### 创建 RpcClient  端

```java
/**
 * Hello Rpc Client examples.
 * @author Mr.zxb
 * @date 2020-08-12 20:16:42
 */
public class HelloRpcClient {
    private static final Logger logger = Logger.getLogger(HelloRpcClient.class.getName());

    private final HelloRpcServiceGrpc.HelloRpcServiceBlockingStub blockingStub;

    public HelloRpcClient(Channel channel) {
        // 将通道传递给代码可以使代码更易于测试，并且更易于重用通道。
        this.blockingStub = HelloRpcServiceGrpc.newBlockingStub(channel);
    }

    /**
     * 调用 RPC server
     * @param name
     */
    private void send(String name) {
        logger.info("Will try to greet " + name + " ...");
        // 创建并填充一个 HelloRequest 发送给服务。
        HelloRpcRequest request = HelloRpcRequest.newBuilder().setName(name).build();
        HelloRpcReply rpcReply = blockingStub.sayHello(request);
        logger.info("result: " + rpcReply.getName());
    }

    public static void main(String[] args) throws InterruptedException {
        // rpc服务地址
        String target = "localhost:8000";

        // 创建到服务器的通信通道，称为通道。 通道是线程安全的并且可重用。
        // 通常在应用程序的开头创建通道并重复使用它们，直到应用程序关闭.
        ManagedChannel managedChannel = ManagedChannelBuilder
                .forTarget(target)
                // 默认情况下，通道是安全的（通过SSL / TLS）。 对于该示例，我们禁用TLS以避免需要证书。
                .usePlaintext()
                .build();
        try {
            HelloRpcClient rpcClient = new HelloRpcClient(managedChannel);
            rpcClient.send("world");
        } finally {
            // ManagedChannels使用诸如线程和TCP连接之类的资源。 
            // 为防止泄漏这些资源，当不再使用该通道时，应将其关闭。 
            // 如果可以再次使用，请使其运行。
            managedChannel.awaitTermination(5, TimeUnit.SECONDS);
        }
    }
}
```

### 跨平台扩展

你可以尝试用不同语言在客户端和服务端构建并运行例子。或者你可以尝试 gRPC 最有用的一个功能，不同的语言间的互操作性，即在不同的语言运行客户端和服务端。每个服务端和客户端使用从同一过 proto 文件生成的接口代码，则意味着任何 `HelloRpc` 客户端可以与任何 `HelloRpc` 服务端对话。

### 总结

gRPC 基于 HTTP/2 标准设计，带来诸如双向流、流控、头部压缩、单 TCP 连接上的多复用请求等特。这些特性使得其在移动设备上表现更好，更省电和节省空间占用。