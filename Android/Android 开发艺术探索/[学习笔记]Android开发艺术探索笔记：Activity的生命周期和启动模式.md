# [学习笔记]Android开发艺术探索笔记：Activity的生命周期和启动模式

## Activity 生命周期的全面分析
### 典型情况下的生命周期分析
1. onCreate : 生命周期第一个方法，可做一些初始化工作；
2. onRestart ： Activity 重新启动，由不可见变为可见；
3. onStart ： Activity 正在启动，此时 Activity 已经可见，但没有出现在前台，还无法和用户交互；
4. onResume ： Activity 可见，出现在前台，并开始活动；
5. onPause ： Activity 正在停止，正常情况下 onStop 会紧接着被调用，但是该方法中不宜做耗时操作，因为当该方法执行完毕后，新 Activity 的 onResume 方法才会执行；打开新的 Activity 时，当前 Activity 的 onPause 方法先执行完毕，新 Activity 的 onCreate 方法才会执行 ；
6. onStop ： Activity 即将停止，可以做一些耗时操作，但不能太耗时；
7. onDestroy ： Activity 即将被销毁，这是 Activity 生命周期中最后一个回调，在这里我们可以做一些回收工作和资源释放；

>**注意**当用户打开新的 Activity 或者切换到桌面的时候，回调如下： onPause -> onStop ，这里有一种特殊的情况，如果新的 Activity 采用了透明主题或 Dialog 主题，那么当前的 Activity 不会回调 onStop 。
onStart和onStop是从activity是否可见的角度来回调的。onResume和onPause是从activity是否定位于前台这个角度来回调的。

旧的Activity先onPause，新的Activity再启动。



### 异常情况下的生命周期分析

#### 资源相关的系统配置发生改变导致 Activity 被杀死并重新创建
当系统配置发生改变后，Activity 会被销毁，其 onPause 、 onStop 、 onDestroy 均会被调用，同时由于 Activity 是在异常情况下终止的，系统会调用 onSaveInstanceState 来保存当前 Activity 的状态。

这个方法的调用时机是在 onStop 之前，它和 onPause 没有既定的时序关系，可能在 onPause 之前，也可能在 onPause 之后。

该方法只会在 Activity 被异常销毁时才会被调用，正常情况下不会调用该方法。

当 Activity 被重新创建后，系统会调用 onRestoreInstanceState ，并且把 Activity 销毁时 onSaveInstanceState 方法所保存的 Bundle 对象作为参数同时传递给 onRestoreInstanceState 方法和 OnCreate 方法。

因此，我们可以通过 onRestoreInstanceState 方法和 OnCreate 方法判断 Activity 是否被重建了，进而恢复之前保存的数据，从时序上讲 onRestoreInstanceState 的调用时机在 onStart 之后。

同时，在 onSaveInstanceState 和 onRestoreInstanceState 方法中，系统已经自动为我们恢复了一些数据，如：用户输入的数据、 ListView 滚动的位置等，有关具体哪些信息被系统自动恢复，可以查看相应 View 的 onSaveInstanceState 和 onRestoreInstanceState 方法的具体实现。

#### 资源内存不足导致低优先级的 Activity 被杀死 
比较难模拟内存不足的情况，Activity 的优先级一般为下面三种：

1. 前台 Activity ，和用户交互的 Activity 优先级最高；
2. 可见非前台 Activity ，如弹出个 Dialog ，无法与用户交互；
3. 后台 Activity ，已经被暂停的 Activity ，如执行了 onStop ，优先级最低；

当系统内存不足时，系统会按照上述优先级去杀死目标 Activity 所在的进程，并在后续通过 onSaveInstanceState 和 onRestoreInstanceState 来保存和恢复数据；如果一个进程中没有四大组件在运行，那么这个进程很容易被杀死。

## Activity 的启动模式 
1. standard : 标准模式。即系统默认模式，每次启动 Activity 时都会重新创建一个该 Activity 的实例，无论之前是否创建过。 Activity 只能在任务栈中创建，由于 ApplicationContext 不在任务栈中，所以当我们直接用 ApplicationContext 启动一个 Activity 时，总是会报异常，此时需要为待启动的 Activity 指定 FLAG_ACTIVITY_NEW_TASK 标记位，这样启动的时候就会为它创建一个任务栈，当任务栈中没有 Activity 时，系统就会回收任务栈，通过这种方式启动的 Activity 其启动模式为 singleTask；
2. singleTop : 栈顶复用模式。如果新 Activity 已经位于任务栈的栈顶，那么此时 Activity 不会重新被创建，同时它的 onNewIntent 方法会被调用，通过此方法的参数我们可以取出当前的请求信息。
3. singleTask : 栈内复用模式。如果任一栈内已有该 Activity 的实例，那么系统会将该 Activity 放到栈顶，并调用它的 onNewIntent 方法，通过此方法的参数我们可以取出当前的请求信息。
4. singleInstance ： 单实例模式。这是一种加强的 singleTask 模式，具备 singleTask 的所有特性，另外此模式的 Activity 只能单独位于一个任务栈中。

配置 Activity 的启动模式一般有一下两个方法：

1.通过 AndroidManifest.xml 为 Activity 指定启动模式；

```xml
<activity
	android:launchMode="singleTask"
/>
```
2.通过在 Intent 中设置标志位来为 Activity 指定启动模式；
```java
	Intent intent = new Intent() ;
	intent.setClass(xx , xxx.class) ;
	intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK) ;
	startActivity(intent) ;
```
上面两种方式都可以为 Activity 指定启动方式，但是二者之间还是有区别的。
首先，优先级上，第二种启动方式的优先级要高于第一种，即两种指定都存在时，以第二种方式为准；其次，在限定范围上不同，例如，第一种无法为 Activity 设定 FLAG_ACTIVITY_CLEAR_TOP 标识，而第二种无法为 Activity 指定 singleInstance 模式。

>**注意** adb shell dumpsys activity 命令可以查看 Activity 启动的相关信息。

### 任务栈相关参数 TaskAffinity 
TaskAffinity 可以翻译为“任务相关性”，这个参数标示了一个 Activity 所需的任务栈名称，默认情况下，所有 Activity 的任务栈名都为应用包名；当然，我们也可以为每个 Activity 单独指定 TaskAffinity 属性，如果该属性值与应用包名一样，就相当于没有指定，TaskAffinity 主要和 singleTask 启动模式或者 allowTaskReparenting 属性配对使用，在其他情况下没意义。另外，任务栈为前台任务栈和后台任务栈，后台任务栈中的 Activity 处于暂停状态，用户可以通过切换将后台任务栈再次调到前台。

当 TaskAffinity 和 singleTask启动模式配合使用时，该 Activity 会运行在名字和 TaskAffinity 相同的任务栈中。

当 TaskAffinity 和 allowTaskReparenting 结合的时候，若应用 A 启动了一个应用 B 中的某个 Activity C 后，如果 Activity C的 allowTaskReparenting 属性为 true ， 那么当应用 B 被启动后，此 Activity 会直接从应用 A 的任务栈转移到应用 B 的任务栈中；此时点击 Home 键回到桌面，点击应用 B 的桌面图标，这是不是启动应用 B 的主 Activity ，而是从新显示了被应用 A 启动的 Activity C ，或者说 C 中 A 的任务栈转移到了 B 的任务栈中。

```xml
<activity
	android:taskAffinity="com.xxx.task"
/>
```

### Activity 的 Flags 
Activity 的 Flags 比较多，这里不一一介绍，要注意的是：有些 Flag 是系统内部使用的，应用程序不需要手动去设置这些 Flag ，以防出现问题。
### IntentFilter 的匹配规则 
启动 Activity 分为两种：显示调用和隐式调用。显示调用需要明确地指定被启动对象的信息，包括包名和类名；隐式调用需要明确指定组件信息。原则上一个 Intent 不应该既是显示调用又是隐式调用，如果二者共存，以**显示调用**为准。
隐式调用需要 Intent 能够匹配目标组件的 IntentFilter 中所设置的过滤信息，若不匹配则无法启动目标 Activity 。 IntentFilter 中的过滤信息有 action 、 category 、 data ， 只有一个 Activity 同时匹配 action 、 category 、 data 才算完全匹配，只有完全匹配才能成功启动目标 Activity ；另外，一个 Activity 中可以有多组 IntentFilter ，只要 Intent 可以匹配任何一组 IntentFilter ，都可以成功启动该 Activity 。 
#### action 的匹配规则 
action 是一个字符串，当 Intent 中的 action 和 IntentFilter 中的 action 完全一样（包括大小写）时，才算匹配成功。存在多个action只需匹配一个即可完全匹配
#### category 的匹配规则 
category 也是一个字符串，Intent 可以没有 category ，此时系统会默认给 Intent 添加 “android.intent.category.DEFAULT”  这个 category ，如果为 Intent 指定了 category ，那么不管为 Intent 设置了多少个 category 都要能够和 IntentFilter 中的任何一个 category 相同。有多少个category就要匹配多少个。
#### data 的匹配规则 
data 的语法格式如下：
```xml
	<data
		android:scheme="string"
		android:host="string"
		android:port="string"
		android:path="string"
		android:pathPattern="string"
		android:pathPrefix="string"
		android:mimeType="string"

        // api 19 and higher 
		android:ssp="string"
		android:sspPattern="string"
		android:sspPrefix="string"
        >
	</data>
```
data 由两部分组成 mimeType 和 URI ，mimeType 是指媒体类型，如 image/jpeg ， video/mp4 等； URI 的结构如下：
```xml
	<scheme>://<host>:<port>[<path>|<pathPattern>|<pathPrefix>]
```
如果 URI 中没有指定 scheme 和 host ，那么这个 URI 是无效的。path 、 pathPattern 和 pathPrefix ： path 表示完整的路径信息；pathPattern 也表示完整的路径信息，但是他里面可以包含通配符 “\*” ， “\*” 表示 0 个或多个任意字符，需要注意的是，由于正则表达式的规范，如果想表示真实的字符串，那么 “*” 要写成 “\\\\\*”  ， “\” 要写成 “\\\\\\\”；data 匹配和 action 类似，也需要完全匹配，下面分两种情况介绍下 data 的匹配规则：

1.只有 mimeType
```xml
	<data android:mimeType="image/*" />
```
这种情况下虽然没有指定 URI ，但却有默认值，URI 的默认值为 content 和 file 。也就是说 data 使用默认值的时候，Intent 中的 URI 部分必须为 content 或 file 才能匹配成功。另外，如果要为 Intent 指定完整的 data ，必须要调用 setDataAndType 方法，不能先调用 setData 和 setType 中的任何一个，再调用另一个，因为 setData 和 setType 会彼此情况对方的值。

2.完整的 data 格式
这种和上面的 “data 语法格式” 内容一样，信息比较全，需要完全匹配。


>**注意：**关于 AndroidManifest.xml 中 data 的书写格式有种特殊情况，下面的两种写法是等效的：
```xml
	<intent-filter ...>
		<data android:scheme="file" android:host="www.baidu.com" />
		...
	</intent-filter>
```
```xml
	<intent-filter ...>
		<data android:scheme="file" />
		<data android:host="www.baidu.com" />
		...
	</intent-filter>
```
**最后**，当我们隐式启动一个 Activity 的时候，需要先判断一下是否能够找到能够匹配我们 Intent 的 Activity ，否则会报 ActivityNotFoundException 异常。判断方法可以使用 PackageManager 的 resolveActivity 或 Intent 的 resolveActivity ，如果它们找不到匹配的 Activity 则返回 null ，这样我们可以规避上述异常。另外，PackageManager 还提供了 queryIntentActivities ，这个方法不是返回最佳匹配的 Activity 信息，而是返回所有成功匹配的 Activity 信息。 下面让我们看一下这两个方法原型：
```java
	public abstract List<ResolveInfo> queryIntentActivities(Intent intent , int flag) ;
	public abstract ResolveInfo resolveActivity(Intent intent , int flag) ;
```
上面两个方法的第一个参数好理解，第二个参数需要注意，我们要使用 MATCH_DEFAULT_ONLY 这个标记位，这个标记位的含义是仅仅匹配那些在 intent-filter 中声明了 <category android:name="android.intent.category.DEFAULT" /> 这个 category 的 Activity 。使用这个标记位的意义在于，只有上述两个方法不返回 null ，那么 startActivity 一定可以成功。如果不使用这个标记位，就可以把 intent-filter 中 category 不含 DEFAULT 的那些 Activity 给匹配出来，从而导致 startActivity 可能失败。因为不含 DEFAULT 这个 category 的 Activity 是无法接收隐式 Intent 的。在 action 和 category 中，有一类 action 和 category 比较重要，它们是：
```	xml
	<action android:name="android.intent.action.MAIN" />
	<category android:name="android.intent.category.LAUNCHER" />
```
这二者共同作用是用来标明这是一个入口 Activity 并且会出现在系统的应用列表中，少了任何一个都没意义，也无法出现在系统应用列表中。另外，针对 Service 和 BroadcastReceive，PackageManager 同样提供了类似的方法去获取成功匹配的组件信息。
