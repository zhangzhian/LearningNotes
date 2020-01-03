## 简介

EventBus项目：https://github.com/greenrobot/EventBus 

EventBus 3.0.0 API：http://greenrobot.org/files/eventbus/javadoc/3.0/

EventBus是一种用于Android/Java的事件发布-订阅总线框架。

**特点：**

- 简化组件之间的通信

  - 分离事件发送者和接收者

  - 很好地处理Activities、Fragments和后台线程

  - 避免复杂且易出错的依赖关系和生命周期问题

- 使代码更简单

- 很快

- 很小

- 通过安装超过100000000次的应用程序在实践中得到验证

- 具有高级功能，如传递线程、订阅服务器优先级等。



##  导入库

**Gradle:**

```
implementation 'org.greenrobot:eventbus:3.1.1'
```

**Maven:**

```
<dependency>
    <groupId>org.greenrobot</groupId>
    <artifactId>eventbus</artifactId>
    <version>3.1.1</version>
</dependency>
```

**通过Maven Central下载 [最新 JAR](https://search.maven.org/remote_content?g=org.greenrobot&a=eventbus&v=LATEST)**

## 使用步骤

### 1. 定义事件

```java
public static class MessageEvent { /* Additional fields if needed */ }
```

### 2. 准备订阅服务

声明并标注订阅方法，还可以指定线程模式：

```java
@Subscribe(threadMode = ThreadMode.MAIN)  
public void onMessageEvent(MessageEvent event) {/* Do something */};
```

注册和注销订阅服务。例如在Android上，Activities、Fragments通常应该根据其生命周期进行注册：

```java
 @Override
 public void onStart() {
     super.onStart();
     EventBus.getDefault().register(this);
 }

 @Override
 public void onStop() {
     super.onStop();
     EventBus.getDefault().unregister(this);
 }
```

### 3. 发布事件

```java
 EventBus.getDefault().post(new MessageEvent());
```

## 使用详解

### 传递线程（ThreadMode）

EventBus可以处理线程：事件可以在不同于发布线程的线程中发布。

一个常见的例子是处理UI改变。在Android中，必须在主线程中进行UI更改，联网或任何耗时的任务都不得在主线程上运行。EventBus可以处理这些任务并与UI线程同步（而不必深入研究线程转换，使用AsyncTask等）。

在EventBus中，可以使用下列ThreadModes之一来定义将调用事件处理方法的线程。

#### ThreadMode：POSTING

订阅者将在发布事件的同一线程中被调用。这是默认值。事件传递是同步完成的，发布完成后，所有订阅者都将被呼叫。

此ThreadMode意味着开销最少，因为它可以完全避免线程切换。因此，这是简单任务的推荐模式，该任务在很短的时间内即可完成，而无需主线程。

使用此模式的事件处理程序应快速返回，以避免阻塞可能是主线程的发布线程。

例：

```java
// Called in the same thread (default)
// ThreadMode is optional here
@Subscribe(threadMode = ThreadMode.POSTING)
public void onMessage(MessageEvent event) {
    log(event.message);
}
```

#### ThreadMode：MAIN

订阅者将在Android的主线程中调用。如果发布线程是主线程，则将直接调用事件处理程序方法。

使用此模式的事件处理程序必须快速返回以避免阻塞主线程。

例：

```java
// Called in Android UI's main thread
@Subscribe(threadMode = ThreadMode.MAIN)
public void onMessage(MessageEvent event) {
    textField.setText(event.message);
}
```

#### ThreadMode：MAIN_ORDERED

订阅者将在Android的主线程中调用。该事件始终排入队列，通过队列传递给订阅者。事件处理具有更严格和更一致的顺序（因此名称为MAIN_ORDERED）。

例如，如果在具有MAIN线程模式的事件处理程序中发布另一个事件，则第二个事件处理程序将在第一个事件处理程序之前完成（因为它被同步调用：将它与方法调用进行比较）。使用MAIN_ORDERED，第一个事件处理程序将完成，然后第二个事件处理程序将在稍后的时间点（主线程具有容量后）被调用。

使用此模式的事件处理程序必须快速返回以避免阻塞主线程。

例：

```java
// Called in Android UI's main thread
@Subscribe(threadMode = ThreadMode.MAIN_ORDERED)
public void onMessage(MessageEvent event) {
    textField.setText(event.message);
}
```

#### ThreadMode：BACKGROUND

订阅者将在后台线程中被调用。

如果发布线程不是主线程，则将在发布线程中直接调用事件处理程序方法。如果发布线程是主线程，则EventBus使用单个后台线程，该线程将顺序传递其所有事件。

使用此模式的事件处理程序应尝试快速返回以避免阻塞后台线程。

例：

```java
// Called in the background thread
@Subscribe(threadMode = ThreadMode.BACKGROUND)
public void onMessage(MessageEvent event){
    saveToDisk(event.message);
}
```

#### ThreadMode：ASYNC

事件处理程序方法在单独的线程中调用。这始终独立于发布线程和主线程。

发布事件永远不会等待使用此模式的事件处理程序方法。

如果事件处理程序方法的执行可能需要一些时间（例如，用于网络访问），则应使用此模式。避免同时触发大量长时间运行的异步处理程序方法，以限制并发线程数。

EventBus使用**线程池**来有效地重用已完成的异步事件处理程序通知中的线程。

例：

```java
// Called in a separate thread
@Subscribe(threadMode = ThreadMode.ASYNC)
public void onMessage(MessageEvent event){
    backend.send(event.message);
}
```

### 配置

EventBusBuilder类配置EventBus的各个方面。

例如，以下是构建EventBus的方法，该事件总线在发布的事件没有订阅者的情况下保持安静：

```java
EventBus eventBus = EventBus.builder()
    .logNoSubscriberMessages(false)
    .sendNoSubscriberEvent(false)
    .build();
```

示例，当订阅者引发异常时失败：

```java
EventBus eventBus = EventBus.builder().throwSubscriberException(true).build();
```

*注意：默认情况下，EventBus捕获从订阅者方法引发的异常，并发送一个 SubscriberExceptionEvent，该事件可能要处理。*

检查[EventBusBuilder类及其JavaDoc](http://greenrobot.org/files/eventbus/javadoc/3.0/org/greenrobot/eventbus/EventBusBuilder.html)，以获取所有可能的配置可能性。

#### 配置默认的EventBus实例

使用EventBus.getDefault()是从应用程序中任何位置获取共享EventBus实例的简单方法。EventBusBuilder还允许使用**installDefaultEventBus ()**方法配置此默认实例 。

例如，可以将默认的EventBus实例配置为重新抛出在订阅者方法中发生的异常。但是，仅在DEBUG构建中使用此方法，因为这可能会在出现异常时使应用程序崩溃：

```java
EventBus.builder().throwSubscriberException(BuildConfig.DEBUG).installDefaultEventBus();
```

*注意：只能在第一次使用默认EventBus实例之前完成一次。随后对installDefaultEventBus()的调用将引发异常。这样可以确保您的应用程序中行为一致。Application类是在使用之前配置默认EventBus实例的好地方。*

### 粘性事件

在发布事件后，某些事件会携带需要的信息。例如，一个事件表示某些初始化已完成。或者，如果有一些传感器或位置数据，并且想要保留最新值。除了使用自己的缓存之外，还可以使用粘性事件。

因此，EventBus将某种类型的最后一个粘性事件保留在内存中。然后，粘性事件可以传递给订户或显式查询。因此，不需要任何特殊的逻辑即可考虑已经可用的数据。

#### 粘性示例
假设某个粘性事件是在一段时间前发布的：

```java
EventBus.getDefault().postSticky(new MessageEvent("Hello everyone!"));
```

现在开始一个新的Activity。**在注册期间，所有粘性订阅者方法将立即获得先前发布的粘性事件**：

```java
@Override
public void onStart() {
    super.onStart();
    EventBus.getDefault().register(this);
}
 
// UI updates must run on MainThread
@Subscribe(sticky = true, threadMode = ThreadMode.MAIN)
public void onEvent(MessageEvent event) {   
    textField.setText(event.message);
}
 
@Override
public void onStop() {
    EventBus.getDefault().unregister(this);    
    super.onStop();
}
```

#### 手动获取和删除粘性事件

最后一个粘性事件在注册时自动传递给匹配的订阅者。但是有时手动检查粘性事件可能更方便。另外，可能有必要删除（使用）粘性事件，以便不再发送它们。例：

```java
MessageEvent stickyEvent = EventBus.getDefault().getStickyEvent(MessageEvent.class);
// Better check that an event was actually posted before
if(stickyEvent != null) {
    // "Consume" the sticky event
    EventBus.getDefault().removeStickyEvent(stickyEvent);
    // Now do something with it
}
```

方法 removeStickyEvent被重载：当传入类时，它将返回之前保留的粘滞事件。使用此变体，可以改进前面的示例：

```java
MessageEvent stickyEvent = EventBus.getDefault().removeStickyEvent(MessageEvent.class);
// Better check that an event was actually posted before
if(stickyEvent != null) {
    // Now do something with it
}
```

### 优先事项和活动取消

大多数EventBus用例不需要优先级或事件取消，但在某些特殊情况下它们可能会派上用场。

#### 订阅者优先级

可以通过在注册过程中为订阅者提供优先级来更改事件传递的顺序。

```java
@Subscribe(priority = 1);
public void onEvent(MessageEvent event) {
    ...
}
```

在同一传递线程（ThreadMode）中，优先级较高的订户将在其他优先级较低的订户之前接收事件。默认优先级为0。

*注意：优先级不影响具有不同ThreadModes的订户之间的传递顺序！* 

#### 取消事件传送

可以通过从订阅者的事件处理方法中调用cancelEventDelivery （**对象**事件） 来取消事件传递过程 。任何进一步的事件传递将被取消，后续的订阅者将不会接收该事件。

```java
// Called in the same thread (default)
@Subscribe
public void onEvent(MessageEvent event){
    // Process the event
    ...
    // Prevent delivery to other subscribers
    EventBus.getDefault().cancelEventDelivery(event) ;
}
```

事件通常由优先级较高的订阅者取消。取消仅限于在发布线程中运行的事件处理方法（ThreadMode.PostThread）。

### 订阅者索引

订阅者索引是EventBus3的新功能。它是一项**可选的优化，可加快初始订阅者注册的速度**。

订阅者索引可以在构建期间使用EventBus注释处理器创建。虽然不需要使用索引，但**建议在Android上**使用它**以获得最佳性能**。

#### 索引前提

请注意，只有@Subscriber方法可以被索引，并且**订阅者和事件类是公共方法**。同样，由于Java注释处理本身的技术限制，@ Subscribe注释**无法在匿名类内部识别**。

当EventBus无法使用索引时，它将在运行时自动回退到反射。因此它仍然可以工作，只是速度稍慢一点。

#### 生成索引

**使用注解处理器**

*如果未使用2.2.0或更高版本的Android Gradle插件，请将该配置与android-apt配合使用。*

要启用索引生成，需要使用annotationProcessor属性将EventBus注解处理器添加到build中。还设置一个参数 eventBusIndex以指定要生成的索引的完全限定的类。因此，例如，将以下部分添加到您的Gradle构建脚本中：

```shell
android {
    defaultConfig {
        javaCompileOptions {
            annotationProcessorOptions {
                arguments = [ eventBusIndex : 'com.example.myapp.MyEventBusIndex' ]
            }
        }
    }
}
 
dependencies {
    implementation 'org.greenrobot:eventbus:3.1.1'
    annotationProcessor 'org.greenrobot:eventbus-annotation-processor:3.1.1'
}
```

**使用kapt**

在Kotlin代码中使用EventBus，需要使用 kapt 代替 annotationProcessor ：

```shell
apply plugin: 'kotlin-kapt' // ensure kapt plugin is applied
 
dependencies {
    implementation 'org.greenrobot:eventbus:3.1.1'
    kapt 'org.greenrobot:eventbus-annotation-processor:3.1.1'
}
 
kapt {
    arguments {
        arg('eventBusIndex', 'com.example.myapp.MyEventBusIndex')
    }
}
```

**使用android-apt**

如果以上方法都不适合，则可以使用[android-apt](https://bitbucket.org/hvisser/android-apt) Gradle插件将EventBus注释处理器添加到构建中。

将以下部分添加到您Gradle构建脚本中：

```shell
buildscript {
    dependencies {
        classpath 'com.neenbedankt.gradle.plugins:android-apt:1.8'
    }
}
 
apply plugin: 'com.neenbedankt.android-apt'
 
dependencies {
    compile 'org.greenrobot:eventbus:3.1.1'
    apt 'org.greenrobot:eventbus-annotation-processor:3.1.1'
}
 
apt {
    arguments {
        eventBusIndex "com.example.myapp.MyEventBusIndex"
    }
}
```

#### 使用索引

成功构建项目后，将为你生成用eventBusIndex指定的类 。然后在设置EventBus时将其传递为：

```java
EventBus eventBus = EventBus.builder().addIndex(new MyEventBusIndex()).build();
```

或者，如果想在整个应用中使用默认实例：

```java
EventBus.builder().addIndex(new MyEventBusIndex()).installDefaultEventBus();
// Now the default instance uses the given index. Use it like this:
EventBus eventBus = EventBus.getDefault();
```

#### 索引你的库

可以将相同的原理应用于库（而不是最终应用程序）的一部分代码。可能具有多个索引类，可以在EventBus设置过程中全部添加它们，例如：

```java
EventBus eventBus = EventBus.builder()
    .addIndex(new MyEventBusAppIndex())
    .addIndex(new MyEventBusLibIndex()).build();
```

### 代码混淆

在开发中使用代码混淆的时候需要加上:

```shell
-keepattributes *Annotation*
-keepclassmembers class * {
    @org.greenrobot.eventbus.Subscribe <methods>;
}
-keep enum org.greenrobot.eventbus.ThreadMode { *; }
 
# Only required if you use AsyncExecutor
-keepclassmembers class * extends org.greenrobot.eventbus.util.ThrowableFailureEvent {
    <init>(java.lang.Throwable);
}
```

*注意：无论是否使用订阅者索引都将需要此配置。*

### AsyncExecutor

AsyncExecutor就像一个线程池，但是具有异常处理。故障会引发异常，AsyncExecutor会将这些异常包装在事件中，该事件会自动发布。

*声明：AsyncExecutor是一个非核心实用程序类。它可以为您节省一些在后台线程中进行错误处理的代码，但它不是EventBus的核心类。*

通常，调用 AsyncExecutor.create ()创建实例并将其保留在Application范围内。然后执行一些操作，实现 RunnableEx接口并将其传递给AsyncExecutor的execute方法。与Runnable不同 ， RunnableEx可能会引发Exception。

如果 RunnableEx实现引发异常，则将捕获该异常并将其包装到[ThrowableFailureEvent中](http://greenrobot.org/files/eventbus/javadoc/3.0/org/greenrobot/eventbus/util/ThrowableFailureEvent.html)，并将其发布。

执行示例：

```java
AsyncExecutor.create().execute(
    new AsyncExecutor.RunnableEx() {
        @Override
        public void run() throws LoginException {
            // No need to catch any Exception (here: LoginException)
            remote.login();
            EventBus.getDefault().postSticky(new LoggedInEvent());
        }
    }
);
```

接收部分的示例：

```java
@Subscribe(threadMode = ThreadMode.MAIN)
public void handleLoginEvent(LoggedInEvent event) {
    // do something
}
 
@Subscribe(threadMode = ThreadMode.MAIN)
public void handleFailureEvent(ThrowableFailureEvent event) {
    // do something
}
```

**AsyncExecutor生成器**

如果要自定义AsyncExecutor实例，请调用静态方法 AsyncExecutor.builder()。它将返回一个生成器，可以**自定义EventBus实例，线程池和failure事件的类**。

另一个定制选项是**执行范围，它提供故障事件上下文信息**。例如，失败事件可能仅与特定的Activity实例或类有关。

如果您的自定义失败事件类实现了[HasExecutionScope](http://greenrobot.org/files/eventbus/javadoc/3.0/org/greenrobot/eventbus/util/HasExecutionScope.html)接口，则AsyncExecutor将自动设置执行范围。这样，您的订户可以查询失败事件的执行范围，并据此作出反应。



后续会输出EventBus使用Demo相关的文档，请持续关注。



**欢迎关注我的公众号，持续分析优质技术文章**
![欢迎关注我的公众号](https://img-blog.csdnimg.cn/20190906092641631.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzMyMjM3NzE5,size_16,color_FFFFFF,t_70)