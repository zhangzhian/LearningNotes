#[学习笔记] Android群英传：Activity与Activity调用栈分析

主要内容

- Activity的生命周期与工作模式
- Activity调用栈管理


##一.Activity

###1.起源
 Activity是用户交互的第一接口，他提供了一个用户完成指令的窗口，当开发者创建Activity之后，通过调用setContentView来指定一个窗口界面，并以此为基础，提供给用户交互的接口，系统采用Activity栈的方式来管理Activity

###2. Activity形态
 Activity一个最大的特点就是拥有多种形态，他可以在多种形态中自由切换，以此来控制自己的生命周期

- Activity/Running

>这个时候，Activity处于Activity栈的最顶层，可见，并与用户进行交互

- Paused

> Activity失去焦点，被一个新的非全屏的Activity或者一个透明的Activity放置在栈顶时，Activity就转换成了qaused形态，他是去了与用户交互的能力，所有状态信息，成员变量都还保留着，只有在系统内存极地的情况下，才会被系统回收

- Stopped

>如果一个Activity被另一个Activity完全覆盖，那么Activity就会进入stop形态，此时他不在可见，但依然保留着所有的状态和成员变量

- Killed

>当Activity被系统回收或者Activity从来没有创建过，Activity就处于killed状态，

###3.生命周期


```
	@Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }

    @Override
    protected void onStart() {
        super.onStart();
    }

    @Override
    protected void onResume() {
        super.onResume();
    }

    @Override
    protected void onRestart() {
        super.onRestart();
    }

    @Override
    protected void onPause() {
        super.onPause();
    }

    @Override
    protected void onStop() {
        super.onStop();
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
    }
```


####1.Activity启动和销毁过程

在系统调用onCreate方法之后，就会马上调用onStart，然后继续调用onResume来进图运行状态，最后都会停在onResume状态，完成启动，系统会调用onDestroy来结束一个Activity的生命周期让他毁掉kill状态

以上就是一个Activity的启动和销毁的过程

- onCreate中创建基本的UI元素
- onPause和onStop：清除Acvtivity的资源，避免浪费
- onDestroy:因为引用会在Activity销毁的时候销毁，而线程不会，所以清除开启的线程

####2.Activity的暂停和恢复过程

当栈顶的Activity部分不可见的时候，Activity进入onPause

- onPause：释放系统资源，
- onResume：需要重新初始化onPause释放的资源

####3. Activity的停止过程

栈顶的Activity部分不可见的时候，实际上后续会有两种可能，从部分不可见到可见，也就是恢复过程，从部分不可见到完全不可见，也就是停止过程，系统在当前Activity不可见的时候调用onPause

####4.Activity的重新创建过程

系统长时间处于stop的状态，而此时系统需要更多的内存或者系统内存比较紧张的时候，系统就会回收你的Activity，而系统为了补偿你，会将你的Activity状态通过onRestoreInstanceState()方法保存到Bundle中去，当然你也可以额外增加键值对去保存这些状态，当你重新需要创建这个Activity的时候，保存的Bundle对象就会传递到Activity的onRestoreInstanceState（）方法中去与onCreate方法中去，这也是onCreate的重要参数——saveInstanceState的来源

savedInstanceState方法并不是每次当Activity离开前台就会调用，如果用户使用finish方法结束，则不会调用，而且Android系统已经默认实现了控件的缓存状态，一次来减少开发者需要实现的缓存逻辑

##二.Android任务栈简介

一个Android应用程序功能通常会被拆分为多个Activity,而各个Activity之间通过Intcnt进行连接,而Android系统，通过栈结构来保存整个App的Activity,栈底的元素是整个任务栈的发起者。一个合理的任务调度栈不仅是性能的保证, 更是提供性能的基础。

当一个App启动时,如果当前环境中不存在该App的任务栈，那么系统就会创建一个任务栈，这个app所启动的Activity都将在这个任务栈中被管理,这个栈也被称为Task,即表示若干个acnVity的集合，他们组合在一起形成一个Task。另外,需要特别注意的是,一个Task中的ActiVity可以来自不同的App,同一个App的Acnvity也可能不在一个Task中。

关于栈结构,后进先出(lastin First out)的线性表。根据Activity在当前栈结构中的位置,来决定该Acavity的状态。正常情况下的android任务栈，当一个Activity启动了另一个Activity的时候,新启动的Acnvity就会置于任务栈的顶端, 并处于活动状态,而启动它的Activity虽然功成身退,但依然保留在任务栈中, 处于停止状态,当用户按下返回键或者调用finish()方法时,系统会移除顶部Activity,让后面的Acnvity恢复活动状态。 

可以给Activity进行一些设置，就是通过在 AndroidMainifest文件中的属性android:1aunchMode来设置或者是通过intent的flag来设置的。

##三.AndroidManifest启动模式
在AndroidManifest文件中一共有四种启动模式

- standard
- singleTop
- singleTask
- singleInstance

###1. standard
默认的启动模式，如果不指定Activity的启动模式，则使用这种模式来启动Activity，每次点击standard模式创建Activity之后，都会创建新的MainActivity覆盖在原有的Activity上

###2.singleTop
singletop，那么在启动的时候，系统会判断当前栈顶Activity是不是要启动的那个，如果是则不创建新的Activity，如果不是则创建新的Activity

这种启动模式虽然不能创建新的实例，但是系统任然会在Activity启动的时候调用onNewIntent()方法，举例子，如果当前任务栈中有ABC三个Activity，而C的启动模式是singleTop，那么这个时候再启动C，那么系统就不会去创建C的实例了，而是会调用C的onNewIntent方法，当前任务栈依然是ABC三个Activity

###3.singleTask
singleTask模式和singleTop模式有点类似，只不过singleTop是检测栈顶元素是否需要启动的Activity，而singleTask是检测整个Activity栈中是否存在当前启动的Activity，如果存在，就将他置于栈顶，并且将以上的activity全部销毁，不过这里也是指在同一个APP中启动整个singleTask的Activity，如果是其他的程序以singleTask模式来启动整个Activity，那么他将创建一个新的任务栈，不过这里有一点需要注意的是，如果启动的模式为singleTask的activity已经在后台的一个栈中，那么启动后，后台的一个任务栈将一起被切换到前台

当Activity2启动ActivityY的时候（启动模式为singleTask），所在的task被切换到前台，且按返回键返回的时候，也会先返回ActivityY所在Task的Activity

可以发现，使用这个模式创建的AcnVity不是在新的任务栈中被打开,就是将已打开的Activity换到前台, 所以这种启动模式通常可以用来退出整个应用，将主Acnvity设为singlelask模式,然后在要退出的AcnVity中转到主AcnVity,从而将主AcnVity之上的AcnVity全部销毁,然后重写主ActlVity的onNewIntent方法 在方法中加上一句finish，将最后一个Activity结束掉。

###4.singleInstance
singieInstance这种启动模式和使用的浏览器工作原理类似。在多个程序中访问浏览器时,如果当前浏览器没有打开则打开浏览器, 否则会在当前打开的浏览器中访问。申明为singleInstance的Activity会出现在一个新的任务栈中而且该任务栈中只存在这一个Activity，举个例子来说,如果应用A的任务栈中创建了MainActivity实例,且启动模式为singleInstance，如果应用B也要激活MainActivity 则不需要创建，两个应用共享该Activity实例，这种启动模式常用于需要与程序分离的界面:如在setupWizard中调用紧急呼叫,就是使用这种启动模式

关于singletop邵p和singleInstance这两种启动模式还有一点需要特殊说明: 如果在一个singleTop或者singleInstance的Activity中通过startActivityForResultO方法来启动另一个ActivityB, 那么系统将直接返回Activity_RESULT_CANCELED而不会再去等待返回。 这是由于系统在framework层做了对这两种启动模式的限制, 因为Android开发者认为，不同的Task中，默认是不能传递数据的。如果一定要传递数据的话，那么只能通过Intent去绑定数据


##四.Intent Flag启动模式
通过intent设置flag来设置一个Activity的启动模式

下面来介绍一些常用的Flag

- Intent.FLAG-ACTIVITY-NEW-TASK
>使用一个新的Task来启动一个Activity,但启动的每个Aetivity都在新的Task栈中，该flag通常使用在从service中启动的actiity场景，由于service中并不存在activity栈，所以用该flag来创建一个新的activity栈，并创建新的activity实例

- FLAG-ACTIVITY-SINGLE-TOP 
>使用singletop模式来启动一个Activity,与指定android:launchMode="singleTop"效果相同

- FLAG-ACTIVITY-CLEAR-Top
>使用SingleTask模式来启动一个Activity,与指定android:launchMode="singleTask"效果相同
- FLAG-ACTIVITY-NO-HISTORY
>使用这种模式启动Acuvity,当该Activity启动其他AcuVity后,该Activity就消失了,不会保留在activity栈中，例如A-B，B中以这种模式启动C，C再启动D，则当前activity栈为ABD

##五.清空任务栈
通常情况下，我们可以在activity的标签上使用以下几种属性来清空任务栈

- clearTaskOnLaunch
> clearTaskOnLaunch属性顾名思义，就是每次返回activity的时候，都将该activity上的所有activity清除，通过这个属性，可以让这个task每次初始化的时候，都只有一个activity

- finishTaskOnLaunch
> finishTaskOnLaunch这个属性和clearTaskOnLaunch有点类似，只不过clearTaskOnLaunch作用在别人身上，而finishTaskOnLaunch作用在自己身上，通过这个属性，当离开这个activity所处的task，那么用户再返回的时候，该activity会被finish掉

- alwaysRetainTaskState
> alwaysRetainTaskState属性给了task一道免死金牌，如果将activity这个属性设置为true，那么该activity所在的task将不接受任何清除命令，一直保持当前task的状态

##六.Activity任务栈使用
使用ActiVity任务栈的各种启动模式和清理方法,是为了更好地使用App中actvity,合理地设置Acuvity的启动模式会让程序运行更有效率,用户体验更好。但任务栈虽好, 却也不能滥用 

如果过度地使用Activity任务栈,则会导致整个App的栈管理混乱，不利于以后程序的拓展，而且在容易出现由于任务栈导致的显示异常，这样的bug是很难调的.所以,在App中使用acuvity任务栈一定要根据实际项目的需要,而不是为了使用任务栈而使用任务栈。



