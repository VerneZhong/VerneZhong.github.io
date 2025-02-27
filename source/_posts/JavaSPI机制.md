---
title: Java SPI
date: 2020-09-11 23:12:08
categories: 
- Java技术
tags:
- Java SPI
---

![](https://github.com/user-attachments/assets/316d91ef-6b1a-415d-b712-2707a3ca7b9f)

<!-- more -->

## Java SPI 

### 什么是Java SPI

Java SPI（服务提供者接口）是动态加载服务的机制。通过遵循特定的规则集，我们可以在我们的应用程序中实现Java SPI，并使用ServiceLoader类加载服务。

### Java SPI 组件

SPI实现中包含四个组件：

1. 服务提供者接口：定义服务提供者实现类的协定的接口或抽象类
2. 服务提供者：实际提供者的实现类
3. SPI配置文件：一个特殊的文件，用于提供查找服务实现的逻辑。文件名必须存在于**META-INF/services**目录中。文件名应与服务提供商接口标准名称完全相同。文件中的每一行都有一个实现服务类详细信息，再次是服务提供者类的完全限定名称。
4. ServiceLoader：Java SPI主类，用于为服务提供者接口加载服务。ServiceLoader中有多种实用方法可用于获取特定的实现，对其进行迭代或重新加载服务。

### Java 服务提供者接口示例

`java.util.spi`软件包提供了许多服务提供者接口，可以将其实现以提供服务。

1. ResourceBundleControlProvider：提供`ResourceBundle.Control`实现的服务提供者的接口。
2. LocaleServiceProvider，CalendarDataProvider，CalendarNameProvider，CurrencyNameProvider，TimeZoneNameProvider和LocaleNameProvider：用于实现特定于区域设置的服务提供者。

![](https://github.com/user-attachments/assets/ac5d1f5e-2cef-409c-a3ed-912f089f7c12)

### Java SPI 示例

让我们创建SPI的实现，并使用ServiceLoader类加载一些服务。

#### 服务提供者接口

假设我们有一个`MessageServiceProvider`接口，用于定义服务提供者实现发送消息。

```java
/**
 * 消息服务提供者接口
 * @author Mr.zxb
 * @date 2020-09-11 23:46:59
 */
public interface MessageServiceProvider {
    /**
     * 发送消息
     * @param message
     */
    void sendMessage(String message);
}
```

#### 服务提供者实现类

我们希望支持电子邮件和推送通知消息。因此，我们将创建`MessageServiceProvider`接口的两个服务提供者实现`EmailServiceProvider`和`PushNotificationServiceProvider`。

```java
/**
 * 邮件服务提供者
 * @author Mr.zxb
 * @date 2020-09-11 23:47:38
 */
public class EmailServiceProvider implements MessageServiceProvider {
    @Override
    public void sendMessage(String message) {
        System.out.println("Sending Email with Message = " + message);
    }
}
```

```java
/**
 * 推送通知服务提供者实现
 * @author Mr.zxb
 * @date 2020-09-11 23:48:12
 */
public class PushNotificationServiceProvider implements MessageServiceProvider {
    @Override
    public void sendMessage(String message) {
        System.out.println("Sending Push Notification with Message = " + message);
    }
}
```

#### 服务提供者配置文件

必须在**META-INF/services**目录中创建配置文件。其名称应为“ **com.chivalry.spi.message.MessageServiceProvider** ”。我们将在此文件中指定两个实现类。

```properties
com.chivalry.spi.message.EmailServiceProvider
com.chivalry.spi.message.PushNotificationServiceProvider
```

#### 加载服务的ServiceLoader示例

最后，我们必须使用ServiceLoader类加载服务。这是一个显示其用法的简单测试程序。

```java
/**
 * {@link ServiceLoader} 加载服务示例
 * @author Mr.zxb
 * @date 2020-09-11 23:53:02
 */
public class ServiceLoaderTest {
    public static void main(String[] args) {
        ServiceLoader<MessageServiceProvider> serviceProviders = ServiceLoader.load(MessageServiceProvider.class);

        // 遍历调用所有的实现类
        for (MessageServiceProvider serviceProvider : serviceProviders) {
            serviceProvider.sendMessage("Hello");
        }

        // 使用 Java 8 Optional 获取第一个 service，注意：findFirst()方法是JDK9版本提供的方法
        Optional<MessageServiceProvider> firstService = serviceProviders.findFirst();
        firstService.ifPresent(messageServiceProvider -> messageServiceProvider.sendMessage("Hello Friend"));

        // 使用 Java 8 forEach() 方法
        serviceProviders.forEach((service) -> service.sendMessage("Have a Nice Day!"));

        // 已加载服务总数
        System.out.println(serviceProviders.stream().count());
    }
}
```

当我们运行上面的程序时，我们得到以下输出：

```powershell
Sending Email with Message = Hello
Sending Push Notification with Message = Hello
Sending Email with Message = Hello Friend
Sending Email with Message = Have a Nice Day!
Sending Push Notification with Message = Have a Nice Day!
2
```

下图显示了我们的最终项目结构和SPI组件：

![](https://github.com/user-attachments/assets/3ad86ad6-de26-4026-9deb-afc1fd04da9f)

### ServiceLoader  结构

```java
// JDK11 版本的 ServiceLoader，与JDK8的版本略有差异
public final class ServiceLoader<S> implements Iterable<S> {
    // The class or interface representing the service being loaded
    private final Class<S> service;

    // The class of the service type
    private final String serviceName;

    // The module layer used to locate providers; null when locating
    // providers using a class loader
    private final ModuleLayer layer;

    // The class loader used to locate, load, and instantiate providers;
    // null when locating provider using a module layer
    private final ClassLoader loader;

    // The access control context taken when the ServiceLoader is created
    private final AccessControlContext acc;

    // The lazy-lookup iterator for iterator operations
    private Iterator<Provider<S>> lookupIterator1;
    private final List<S> instantiatedProviders = new ArrayList<>();

    // The lazy-lookup iterator for stream operations
    private Iterator<Provider<S>> lookupIterator2;
    private final List<Provider<S>> loadedProviders = new ArrayList<>();
    private boolean loadedAllProviders; // true when all providers loaded

    // Incremented when reload is called
    private int reloadCount;

    private static JavaLangAccess LANG_ACCESS;
    static {
        LANG_ACCESS = SharedSecrets.getJavaLangAccess();
    }
    
    // ...
    
    // 可以看出 ServiceLoader 是延迟初始化服务接口实现类的
    private final class LazyClassPathLookupIterator<T> implements Iterator<Provider<T>> {
        static final String PREFIX = "META-INF/services/";

        Set<String> providerNames = new HashSet<>();  // to avoid duplicates
        Enumeration<URL> configs;
        Iterator<String> pending;

        Provider<T> nextProvider;
        ServiceConfigurationError nextError;

        LazyClassPathLookupIterator() { }
        
    }
}
```

让我们看一下ServiceLoader类的重要方法。

- load()：加载特定SPI服务的静态方法。
- findFirst()：返回可用于该服务提供者的第一个服务。
- forEach()：对于在此服务加载器实例中的每个服务提供者运行一些代码很有用。
- stream()：返回此服务加载器中服务提供者的流。
- iterator()：返回服务提供者的迭代器。
- reload()：重新加载服务提供者实现类。当我们即时更改服务提供者实现类的配置并希望重新加载服务列表时，这很有用。

### SPI 在开源框架中的应用

- 数据库驱动加载接口实现类的加载：JDBC加载不同类型数据库的驱动
- 日志门面接口实现类加载：slf4j加载不同提供商的日志实现类
- Spring中大量使用了SPI，比如：对servlet3.0规范对ServletContainerInitializer的实现、自动类型转换Type Conversion SPI(Converter SPI、Formatter SPI)等
- Dubbo中也大量使用SPI的方式实现框架的扩展, 不过它对Java提供的原生SPI做了封装，允许用户扩展实现Filter接口

### 总结

Java SPI提供了一种在我们的应用程序中动态配置和加载服务的简便方法。但是，这在很大程度上取决于服务配置文件，并且文件中的任何更改都可能破坏应用程序。

使用Java SPI机制的优势是实现解耦，使得第三方服务模块的装配控制的逻辑与调用者的业务代码分离，而不是耦合在一起。应用程序可以根据实际业务情况启用框架扩展或替换框架组件。

相比使用提供接口jar包，供第三方服务模块实现接口的方式，SPI的方式使得源框架，不必关心接口的实现类的路径，可以不用通过下面的方式获取接口实现类。

- 代码硬编码import 导入实现类
- 指定类全路径反射获取：例如在JDBC4.0之前，JDBC中获取数据库驱动类需要通过**Class.forName("com.mysql.jdbc.Driver")**，类似语句先动态加载数据库相关的驱动，然后再进行获取连接等的操作
- 第三方服务模块把接口实现类实例注册到指定地方，源框架从该处访问实例

通过SPI的方式，第三方服务模块实现接口后，在第三方的项目代码的`META-INF/services`目录下的配置文件指定实现类的全路径名，框架即可找到实现类。

### 参考文献

- [ServiceLoader API文档](https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/util/ServiceLoader.html)