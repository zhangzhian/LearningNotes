# [学习笔记]Android开发艺术探索：理解Window和WindowManager

Window是一个抽象类，具体实现是 PhoneWindow 。不管是 Activity 、 Dialog 、 Toast 它们的视图都是附加在Window上的，因此Window实际上是View的直接管理者。 WindowManager 是外界访问Window的入口，通过WindowManager可以创建Window，而 Window的具体实现位于 WindowManagerService 中，WindowManager和 WindowManagerService的交互是一个IPC过程。

##  Window和WindowManager

通过WindowManager添加Window的过程：

 ```java
mWindowManager = (WindowManager) getSystemService(Context.WINDOW_SERVICE);
mFloatingButton = new Button(this);
mFloatingButton.setText("click me");
mLayoutParams = new WindowManager.LayoutParams(
        LayoutParams.WRAP_CONTENT, LayoutParams.WRAP_CONTENT, 0, 0,
        PixelFormat.TRANSPARENT);
mLayoutParams.flags = LayoutParams.FLAG_NOT_TOUCH_MODAL
        | LayoutParams.FLAG_NOT_FOCUSABLE
        | LayoutParams.FLAG_SHOW_WHEN_LOCKED;
mLayoutParams.type = LayoutParams.TYPE_SYSTEM_ERROR;
mLayoutParams.gravity = Gravity.LEFT | Gravity.TOP;
mLayoutParams.x = 100;
mLayoutParams.y = 300;
mFloatingButton.setOnTouchListener(this);
mWindowManager.addView(mFloatingButton, mLayoutParams);
 ```

WindowManager的flags和type这两个属性比较重要： Flags代表Window的属性，控制Window的显示特性

1. FLAG_NOT_FOCUSABLE 在此模式下，Window不需要获取焦点，也不需要接收各种输 入事件，这个标记同时会启用FLAG_NOT_TOUCH_MODAL，最终事件会直接传递给下层具有焦点的Window。
2. FLAG_NOT_TOUCH_MODAL 在此模式下，系统将当前Window区域以外的点击事件传 递给底层的Window，当前Window区域内的单击事件则自己处理。一般需要开启此标 记。 
3. FLAG_SHOW_WHEN_LOCKED 开启此模式Window将显示在锁屏界面上。

type参数表示Window的类型。

1. 应用Window

2. 子Window 如Dialog 

3. 系统Window 如Toast和系统状态栏

Window是分层的，每个Window对应一个z-ordered，层级大的会覆盖在层级小的上面，和 HTM的z-index概念一样。在三类Window中，应用Window的层级范围是1~99，子Window的 层级范围是1000~1999，系统Window的层级范围是2000~2999，这些值对应 WindowManager.LayoutParams的type参数。一般系统Window选用 TYPE_SYSTEM_OVERLAY 或者 TYPE_SYSTEM_ERROR （同时需要权限  ）。

```xml
<uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW" />
```

WindowManager提供的功能很简单，常用的只有三个方法： 

1. 添加View 
2.  更新View 
3.  删除View

这个三个方法定义在 ViewManager 中，而WindowManager继承了ViewManager。

```java
public interface ViewManager
{
      /**
      * Assign the passed LayoutParams to the passed View and add the view to the windo
      w.
      * <p>Throws {@link android.view.WindowManager.BadTokenException} for certain prog
      ramming
      * errors, such as adding a second view to a window without removing the first vie
      w.
      * <p>Throws {@link android.view.WindowManager.InvalidDisplayException} if the win
      dow is on a
      * secondary {@link Display} and the specified display can't be found
      * (see {@link android.app.Presentation}).
      * @param view The view to be added to this window.
      * @param params The LayoutParams to assign to view.
      */
      public void addView(View view, ViewGroup.LayoutParams params);
      public void updateViewLayout(View view, ViewGroup.LayoutParams params);
      public void removeView(View view);
}
```

## Window的内部机制

Window是一个抽象的概念，每一个Window都对应着一个View和一个ViewRootImpl， Window和View通过ViewRootImpl来建立联系。因此Window并不是实际存在的，它是以 View的形式存在的。所以WindowManager的三个方法都是针对View的，说明View才是 Window存在的实体。在实际使用中无法直接访问Window，必须通过WindowManager来访问 Window。

Window的添加过程，删除过程 ，更新过程都是通过WindowManager的真正实现类—— WindowManagerImpl 类的三大方法的源码来对Window的内部机制进行分析。 具体点说就是，WindowManagerImpl并没有直接实现Window的三大操作，而是全部交 给 WindowManagerGlobal 处理。然后在 WindowManagerGlobal 内部都是通过 ViewRootImpl 里的 一个Binder对象 mWindowSession （ IWindowSession 类型）进行IPC调用 WindowManagerService 进行Window的三大操作。

##  Window的创建过程

View是Android中视图的呈现方式，但是View不能单独存在，必须附着在Window这个抽象的概念上面，因此有视图的地方就有Window。这些视图包括： Activity、Dialog、Toast、PopUpWindow等等。

1. 在创建视图并显示出来时，首先是通过创建一个Window对象，然后通过 WindowManager对象的 addView(View view, ViewGroup.LayoutParams params); 方法将 contentView 添加到Window中，完成添加和显示视图这两个过程。

2. 在关闭视图时，通过WindowManager来移除 DecorView， mWindowManager.removeViewImmediate( view); 。

3. Toast比较特殊，具有定时取消功能，所以系统采用了Handler，内部有两类IPC过程：

    i. Toast访问 NotificationManagerService

    ii. NotificationManagerService 回调Toast里的 TN 接口

显示和隐藏Toast都通过NotificationManagerService（NMS）来实现，而NMS运行在系统进程中，所以只能通过IPC来进行显示/隐藏Toast。而TN是一个Binder类，在Toast和NMS进行 IPC的过程中，当NMS处理Toast的显示/隐藏请求时会跨进程回调TN中的方法，这时由于TN 理解Window和Window Manager 运行再Binder线程池中，所以需要通过Handler将其切换到当前线程（即发起Toast请求所在的线程），然后通过WindowManager的 addView/removewView 方法真正完成显示和隐藏Toast。 本章节对源码的分析侧重的是整体流程，避免深入代码逻辑无法自拔的情形。