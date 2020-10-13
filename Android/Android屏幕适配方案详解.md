# Android屏幕适配方案详解

先说结论，个人意见，大型、注重稳定性的项目，建议使用Smallest限定符方案，快速、敏捷开发的项目使用基于今日头条方案的AutoSize。选用适合自己项目的方案。

UI设计上明显更适合使用wrap_content,match_parent,layout_weight等，就要使用，**而且在高度维度，要依照情况设计为可滑动的方式，或者match_parent,尽量不要写死。**

所有的适配方案都不是用来取代match_parent,wrap_content的，而是用来完善他们的。

## 一、基础知识

**px**是真实像素单位，Pixel像素，不同手机的分辨率可能不同，比如一个100*100像素的控件在分辨率越来越高的手机上会在整体UI中看起来越来越小。

**dp **（dip）指的是设备独立像素，在不同分辨率和尺寸的手机上代表了不同的真实像素，比如在分辨率较低的手机中，可能1dp=1px,而在分辨率较高的手机中，可能1dp=2px，这样的话，一个100*100dp的控件，在不同的手机中就能表现出差不多的大小了。

**dpi**是像素密度，指的是在**系统软件上**指定的单位尺寸（英寸）的像素数量，它往往是写在系统出厂配置文件的一个固定值。

**density** 是密度，指屏幕上每平方英寸中含有的像素点数量。

android中的dp在渲染前会将dp转为px，计算公式：

px = density * dp;

density = dpi / 160;

**px = dp * (dpi / 160)**;

dpi是根据屏幕真实的分辨率和尺寸来计算的，每个设备都可能不一样的。

**屏幕尺寸、分辨率、像素密度三者关系**

通常情况下，一部手机的分辨率是宽x高，屏幕大小是以寸为单位，那么三者的关系是：

![img](https://mmbiz.qpic.cn/mmbiz_png/5EcwYhllQOgM19n6iawpWQRCfcibxicoBYG51prmqwNCLAVALyK5Rhv4uSbrU5FQKQL6bZI3iaibTJaz3NMpEQ8zWAA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

举个例子：屏幕分辨率为：1920*1080，屏幕尺寸为5吋的话，那么dpi为440。

![img](https://mmbiz.qpic.cn/mmbiz_png/5EcwYhllQOgM19n6iawpWQRCfcibxicoBYG0aE9VoUJylxjwZHClXzKeeiadnQyvpLwsyZfES4axmPkmrwZ1jtyibKA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**存在问题**

假设我们UI设计图是按屏幕宽度为360dp来设计的，那么在上述设备上，屏幕宽度其实为1080/(440/160)=392.7dp，也就是屏幕是比设计图要宽的。这种情况下， 使用dp也是无法在不同设备上显示为同样效果的。 同时还存在部分设备屏幕宽度不足360dp，这时就会导致按360dp宽度来开发实际显示不全的情况。

而且上述屏幕尺寸、分辨率和像素密度的关系，很多设备并没有按此规则来实现， 因此dpi的值非常乱，没有规律可循，从而导致使用dp适配效果差强人意。

## 二、今日头条方案

### 1. 适配方案

从dp和px的转换公式 ：**px = dp * density** 

可以看出，如果设计图宽为360dp，想要保证在所有设备计算得出的px值都正好是屏幕宽度的话，我们只能修改 density 的值。

通过阅读源码，我们可以得知，density 是 DisplayMetrics 中的成员变量，而 DisplayMetrics 实例通过 **Resources#getDisplayMetrics** 可以获得，而Resouces通过Activity或者Application的Context获得。

DisplayMetrics 中和适配相关的几个变量：

- **DisplayMetrics#density** 就是上述的density
- **DisplayMetrics#densityDpi** 就是上述的dpi
- **DisplayMetrics#scaledDensity** 字体的缩放因子，正常情况下和density相等，但是调节系统字体大小后会改变这个值

dp和px的转换基本都是通过 DisplayMetrics 来计算的

**方案**

核心：**density = 当前设备屏幕总宽度（单位为像素）/ 设计图总宽度（单位为 dp)**

下面假设设计图宽度是360dp，以宽维度来适配。

那么适配后的 density = 设备真实宽(单位px) / 360，接下来只需要把我们计算好的 density 在系统中修改下即可，代码实现如下：

![img](https://mmbiz.qpic.cn/mmbiz_jpg/5EcwYhllQOgM19n6iawpWQRCfcibxicoBYGOqZUB55MX5uoJ57bRICLjBV3GmlJocWGpQFzEiaAYfANvVNbxO4B1gQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

同时在 Activity#onCreate 方法中调用下。代码比较简单，也没有涉及到系统非公开api的调用，因此理论上不会影响app稳定性。

忽略了DisplayMetrics#scaledDensity的特殊性，将DisplayMetrics#scaledDensity和DisplayMetrics#density设置为同样的值，从而某些用户在系统中修改了字体大小失效了，但是我们还不能直接用原始的scaledDensity，直接用的话可能导致某些文字超过显示区域，因此我们可以通过计算之前scaledDensity和density的比获得现在的scaledDensity，方式如下：

![img](https://mmbiz.qpic.cn/mmbiz_jpg/5EcwYhllQOgM19n6iawpWQRCfcibxicoBYGUle8F5PrpBI2LZ3icLOq2hQHl0ujhsMyjta7mcZ8Iqbs0BECMN64kVw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

系统设置中切换字体，再返回应用，字体并没有变化。于是还得监听下字体切换，调用 Application#registerComponentCallbacks 注册下 onConfigurationChanged 监听即可。

因此最终方案如下：

![img](https://mmbiz.qpic.cn/mmbiz_jpg/5EcwYhllQOgM19n6iawpWQRCfcibxicoBYGhxiapFRVjOtiaWzcERXwjaRDJgyyoIibSq2AJrby8q2aExttHeZfk0VZQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



当然以上代码只是以设计图宽360dp去适配的，如果要以高维度适配，可以再扩展下代码即可。

### 2. 优点

使用成本非常低，操作非常简单，使用该方案后在页面布局时不需要额外的代码和操作，这点可以说完虐其他屏幕适配方案

1. 侵入性非常低，该方案和项目完全解耦，在项目布局时不会依赖哪怕一行该方案的代码，而且使用的还是 **Android** 官方的 **API**，意味着当你遇到什么问题无法解决，想切换为其他屏幕适配方案时，基本不需要更改之前的代码，整个切换过程几乎在瞬间完成，会少很多麻烦，节约很多时间，试错成本接近于 0
2. 可适配三方库的控件和系统的控件(不止是是 **Activity** 和 **Fragment**，**Dialog**、**Toast** 等所有系统控件都可以适配)，由于修改的 **density** 在整个项目中是全局的，所以只要一次修改，项目中的所有地方都会受益
3. 不会有任何性能的损耗

### 3. 缺点

第三个优点既是这个方案的优点也同样是缺点，也是非常致命的。

将整个项目进行适配，适配范围是不可控的，项目中的系统控件、三方库控件、等非项目自身设计的控件，设计图尺寸并不会和项目自身的设计图尺寸一样。

另外，项目中的系统控件、三方库控件、等非项目自身设计的控件，设计图尺寸并不会和项目自身的设计图尺寸一样。

当这个适配方案不分类型，将所有控件都强行使用我们项目自身的设计图尺寸进行适配时，这时就会出现问题，**当某个系统控件或三方库控件的设计图尺寸和和我们项目自身的设计图尺寸差距非常大时，这个问题就越严重**

**解决方案1**：

调整设计图尺寸，因为三方库可能是远程依赖的，无法修改源码，也就无法让三方库来适应我们项目的设计图尺寸，所以只有我们自身作出修改，去适应三方库的设计图尺寸，我们将项目自身的设计图尺寸修改为这个三方库的设计图尺寸，就能完成项目自身和三方库的适配

**这时项目的设计图尺寸修改了，所以项目布局文件中的 dp 值，也应该按照修改的设计图尺寸，按比例增减，保持与之前设计图中的比例不变**

但是如果为了适配一个三方库修改整个项目的设计图尺寸，是非常不值得的，所以这个方案支持以 Activity 为单位修改设计图尺寸，相当于每个 Activity 都可以自定义设计图尺寸，因为有些 Activity 不会使用三方库 View，也就不需要自定义尺寸，所以每个 Activity都有控制权的话，这也是最灵活的

但这也有个问题，当一个 Activity使用了多个设计图尺寸不一样的三方库 View，就会同样出现上面的问题，这也就只有把设计图改为与几个三方库比较折中的尺寸，才能勉强缓解这个问题

**解决方案2**：

第二个方案是最简单的，也是按Activity为单位，取消当前Activity的适配效果，改用其他的适配方案

### 4. 注意的问题

可以将设计图尺寸填写成以 **px** 为单位的宽度和高度，这样我们在布局文件中，也就能直接填写设计图上标注的 **px** 值，省掉了将 **px** 换算为 **dp** 的时间 (大部分公司的设计图都只标注 **px** 值)，而且照样能完美适配，

但是建议大家千万不要这样做。

**第一个问题：**强耦合于这个方案

**第二个问题：**当某个系统控件或三方库控件的设计图尺寸和和我们项目自身的设计图尺寸差距非常大时，问题就很严重

## 三、AutoSize方案

屏幕适配框架 [**AndroidAutoSize**](https://github.com/JessYanCoding/AndroidAutoSize)是根据 **今日头条屏幕适配方案** 优化的.

### 1. 结构

```
├── external
│   ├── ExternalAdaptInfo.java
│   ├── ExternalAdaptManager.java
│── internal
│   ├── CancelAdapt.java
│   ├── CustomAdapt.java
│── unit
│   ├── Subunits.java
│   ├── UnitsManager.java
│── utils
│   ├── AutoSizeUtils.java
│   ├── LogUtils.java
│   ├── Preconditions.java
│   ├── ScreenUtils.java
├── ActivityLifecycleCallbacksImpl.java
├── AutoAdaptStrategy.java
├── AutoSize.java
├── AutoSizeConfig.java
├── DefaultAutoAdaptStrategy.java
├── DisplayMetricsInfo.java
├── FragmentLifecycleCallbacksImpl.java
├── InitProvider.java
```

### 2. 功能介绍

**AndroidAutoSize** 在使用上只需要填写设计图尺寸这一步即可接入项目。

**AndroidAutoSize** 有两种类型的布局单位可以选择，一个是 **主单位 (dp、sp)**，一个是 **副单位 (pt、in、mm)**，两种单位面向的应用场景都有不同，也都有各自的优缺点

- **主单位**: 使用 **dp、sp** 为单位进行布局，侵入性最低，会影响其他三方库页面、三方库控件以及系统控件的布局效果，但 **AndroidAutoSize** 也通过这个特性，使用 **ExternalAdaptManager** 实现了在不修改三方库源码的情况下适配三方库的功能
- **副单位**: 使用 **pt、in、mm** 为单位进行布局，侵入性高，对老项目的支持比较好，不会影响其他三方库页面、三方库控件以及系统控件的布局效果，可以彻底的屏蔽修改 **density** 所造成的所有未知和已知问题，但这样 **AndroidAutoSize** 也就无法对三方库进行适配

大家可以根据自己的应用场景在 **主单位** 和 **副单位** 中选择一个作为布局单位，建议想引入老项目并且注重稳定性的人群使用 **副单位**，只是想试试本框架，随时可能切换为其他屏幕适配方案的人群使用 **主单位**

其实 **AndroidAutoSize** 可以同时支持 **主单位** 和 **副单位**，但 **AndroidAutoSize** 可以同时支持 **主单位** 和 **副单位** 的目的，只是为了让使用者可以在 **主单位** 和 **副单位** 之间灵活切换，因为切换单位的工作量可能非常巨大，不能立即完成，但领导又要求马上打包上线，这时就可以起到一个很好的过渡作用

### 3. 主单位

#### 基本使用

```xml
<manifest>
    <application>            
        <meta-data
            android:name="design_width_in_dp"
            android:value="360"/>
        <meta-data
            android:name="design_height_in_dp"
            android:value="640"/>           
     </application>           
</manifest>
```

在使用主单位时，`design_width_in_dp` 和 `design_height_in_dp` 的单位必须是 dp

#### 注意事项

- 注意在 AndroidManifest.xml 中填写的 dp 尺寸和布局中填写的尺寸的一致性
- `design_width_in_dp `和 `design_height_in_dp` 都需要填写，但是只会将高度和宽度其中的一个作为基准进行适配，一方作为基准，另一方就会变为备用，默认以宽度为基准进行适配。可以通过 **AutoSizeConfig#setBaseOnWidth(Boolean)** 切换。

#### 自动运行是如何做到的

声明一个 **ContentProvider**，在它的 **onCreate** 方法中启动框架即可，在 **App** 启动时，系统会在 **App** 的主进程中自动实例化声明的这个 **ContentProvider**，并调用它的 **onCreate** 方法，执行时机比 **Application#onCreate** 还靠前，可以做一些初始化的工作。

需要注意的是，如果项目拥有多进程，系统只会在主进程中实例化一个声明的 **ContentProvider**，这时就需要在 **Application#onCreate** 中调用下 **ContentProvider#query** 执行查询操作， **ContentProvider** 就会在当前进程中实例化 (每个进程中只会保证有一个实例)。

所以应用到框架中就是，如果需要在多个进程中都进行屏幕适配，那就需要在 **Application#onCreate** 中调用 **AutoSize#initCompatMultiProcess** 方法

#### 进阶使用

**自定义 Activity**

在 **AndroidManifest.xml** 中填写的设计图尺寸，是整个项目的全局设计图尺寸，但是如果某些 **Activity** 页面由于某些原因，设计图尺寸不一样，可以让这个页面的 **Activity** 实现 **CustomAdapt** 接口即可实现你的需求，**CustomAdapt** 接口的第一个方法可以修改当前页面的设计图尺寸，第二个方法可以切换当前页面的适配基准

```java
public class CustomAdaptActivity extends AppCompatActivity implements CustomAdapt {

	 /**
     * 是否按照宽度进行等比例适配 (为了保证在高宽比不同的屏幕上也能正常适配, 所以只能在宽度和高度之中选择一个作为基准进行适配)
     *
     * @return {@code true} 为按照宽度进行适配, {@code false} 为按照高度进行适配
     */
    @Override
    public boolean isBaseOnWidth() {
        return false;
    }

	 /**
     * 这里使用 iPhone 的设计图, iPhone 的设计图尺寸为 750px * 1334px, 高换算成 dp 为 667 (1334px / 2 = 667dp)
     * <p>
     * 返回设计图上的设计尺寸, 单位 dp
     * {@link #getSizeInDp} 须配合 {@link #isBaseOnWidth()} 使用, 规则如下:
     * 如果 {@link #isBaseOnWidth()} 返回 {@code true}, {@link #getSizeInDp} 则应该返回设计图的总宽度
     * 如果 {@link #isBaseOnWidth()} 返回 {@code false}, {@link #getSizeInDp} 则应该返回设计图的总高度
     * 如果您不需要自定义设计图上的设计尺寸, 想继续使用在 AndroidManifest 中填写的设计图尺寸, {@link #getSizeInDp} 则返回 {@code 0}
     *
     * @return 设计图上的设计尺寸, 单位 dp
     */
    @Override
    public float getSizeInDp() {
        return 667;
    }
}
```

如果某个 **Activity** 想放弃适配，让这个 **Activity** 实现 **CancelAdapt** 接口即可，比如修改 **density** 影响到了老项目中的某些 **Activity** 页面的布局效果，这时就可以让这个 **Activity** 实现 **CancelAdapt** 接口

```java
public class CancelAdaptActivity extends AppCompatActivity implements CancelAdapt {

}
```

**自定义 Fragment**

**Fragment** 的自定义方式和 **Activity** 是一样的，只不过在使用前需要先在 **App** 初始化时开启支持

```java
AutoSizeConfig.getInstance().setCustomFragment(true);
```

实现 **CustomAdapt**

```java
public class CustomAdaptFragment extends Fragment implements CustomAdapt {

    @Override
    public boolean isBaseOnWidth() {
        return false;
    }

    @Override
    public float getSizeInDp() {
        return 667;
    }
}
```

实现 **CancelAdapt**

```java
public class CancelAdaptFragment extends Fragment implements CancelAdapt {

}
```

**适配三方库页面**

在使用主单位时可以使用 **ExternalAdaptManager** 来实现在不修改三方库源码的情况下，适配三方库的所有页面 (**Activity、Fragment**)

- 通过 **ExternalAdaptManager#addExternalAdaptInfoOfActivity(Class, ExternalAdaptInfo)** 将需要自定义的类和自定义适配参数添加进方法即可替代实现 **CustomAdapt** 的方式
- 通过 **ExternalAdaptManager#addCancelAdaptOfActivity(Class)** 将需要取消适配的类添加进方法即可替代实现 **CancelAdapt** 的方式

需要注意的是 **ExternalAdaptManager** 的方法虽然可以添加任何类，但是只能支持 **Activity、Fragment**，并且 **ExternalAdaptManager** 是支持链式调用的，以便于持续添加多个页面

当然 **ExternalAdaptManager** 不仅可以对三方库的页面使用，也可以让自己项目中的 **Activity、Fragment** 不用实现 **CustomAdapt**、**CancelAdapt** 即可达到自定义适配参数和取消适配的功能

### 4. 副单位

#### 基本使用

首先和 **主单位** 一样也需要先在 **app** 的 **AndroidManifest.xml** 中填写上设计图尺寸，但和 **主单位** 不一样的是，当在使用 **副单位** 时 `design_width_in_dp` 和 `design_height_in_dp` 的单位不需要一定是 **dp**，可以直接填写设计图的 **px** 尺寸，在布局文件中每个控件的大小也可以直接填写设计图上标注的 **px** 尺寸，无需再将 **px** 转换为 **dp**，这是 **副单位的** 特性之一

```xml
<manifest>
    <application>            
        <meta-data
            android:name="design_width_in_dp"
            android:value="1080"/>
        <meta-data
            android:name="design_height_in_dp"
            android:value="1920"/>           
     </application>           
</manifest>
```

由于 **AndroidAutoSize** 提供了 **pt、in、mm** 三种类型的 **副单位** 供使用者选择，所以在使用 **副单位** 时，还需要在 **APP** 初始化时，通过 **UnitsManager#setSupportSubunits(Subunits)** 方法选择一个你喜欢的副单位，然后在布局文件中使用这个副单位进行布局，三种类型的副单位，其实效果都是一样，大家按喜欢的名字选择即可

副单位是为了彻底屏蔽修改 **density** 所造成的对三方库页面、三方库控件以及系统控件的布局效果的影响，所以在使用副单位时建议调用 **UnitsManager#setSupportDP(false)** 和 **UnitsManager#setSupportSP(false)**，关闭 **AndroidAutoSize** 对 **dp** 和 **sp** 的支持。

#### 自定义 **Activity** 和 **Fragment**

在使用 **副单位** 时自定义 **Activity** 和 **Fragment** 的方式是和 **主单位** 是一样

#### 适配三方库页面

如果你的项目在使用 **副单位** 并且关闭了对 **主单位 (dp、sp)** 的支持， **ExternalAdaptManager** 对三方库的页面是不起作用的，只对自己项目中的页面起作用，除非三方库的页面也使用了副单位 **(pt、in、mm)** 进行布局

其实 **副单位** 之所以能彻底屏蔽修改 **density** 所造成的对三方库页面、三方库控件以及系统控件的布局效果的影响，就是因为三方库页面、三方库控件以及系统控件基本上使用的都是 **dp、sp** 进行布局，所以只要 **AndroidAutoSize** 关闭了对 **dp、sp** 的支持，转而使用 **副单位** 进行布局，就能彻底屏蔽修改 **density** 所造成的对三方库页面、三方库控件以及系统控件的布局效果的影响

但这也同样意味着使用 **副单位** 就不能适配三方库的页面了，**ExternalAdaptManager** 也就对三方库的页面不起作用了

## 四、SmallestWidth限定符方案

### 1. 宽高限定符适配

```xml
├── src/main
│   ├── res
│   ├── ├──values
│   ├── ├──values-800x480
│   ├── ├──values-860x540
│   ├── ├──values-1024x600
│   ├── ├──values-1024x768
│   ├── ├──...
│   ├── ├──values-2560x1440
```

设定一个基准的分辨率，其他分辨率都根据这个基准分辨率来计算，在不同的尺寸文件夹内部，根据该尺寸编写对应的dimens文件。

比如以480x320为基准分辨率

- 宽度为320，将任何分辨率的宽度整分为320份，取值为x1-x320
- 高度为480，将任何分辨率的高度整分为480份，取值为y1-y480

那么对于800*480的分辨率的dimens文件来说，

x1=(480/320)*1=1.5px

x2=(480/320)*2=3px

...

**但是这个方案有一个致命的缺陷，那就是需要精准命中才能适配**，否则就只能用统一的默认的dimens文件了。而使用默认的尺寸的话，UI就很可能变形，简单说，就是容错机制很差。

### 2. SmallestWidth适配

SmallestWidth适配，或者叫sw限定符适配。指的是Android会识别屏幕可用高度和宽度的最小尺寸的dp值（其实就是手机的宽度值），然后根据识别到的结果去资源文件中寻找对应限定符的文件夹下的资源文件。

**机制和上文提到的宽高限定符适配原理上是一样的，都是系统通过特定的规则来选择对应的文件。**

举个例子，小米5的dpi是480,横向像素是1080px，根据px=dp(dpi/160)，横向的dp值是1080/(480/160),也就是360dp,系统就会去寻找是否存在value-sw360dp的文件夹以及对应的资源文件。

```
├── src/main
│   ├── res
│   ├── ├──values
│   ├── ├──values-sw320dp
│   ├── ├──values-sw360dp
│   ├── ├──values-sw400dp
│   ├── ├──values-sw411dp
│   ├── ├──values-sw480dp
│   ├── ├──...
│   ├── ├──values-sw600dp
│   ├── ├──values-sw640dp
```

SmallestWidth限定符适配和宽高限定符适配**最大的区别在于，前者有很好的容错机制，如果没有value-sw360dp文件夹，系统会向下寻找，比如离360dp最近的只有value-sw350dp，那么Android就会选择value-sw350dp文件夹下面的资源文件。**这个特性就完美的解决了上文提到的宽高限定符的容错问题。

这套方案是上述几种方案中最接近完美的方案。

首先，从开发效率上，它不逊色于上述任意一种方案。根据固定的放缩比例，我们基本可以按照UI设计的尺寸不假思索的填写对应的dimens引用。

以360个像素宽度的设计稿为例，在values-sw360dp文件夹下的dimens文件应该怎么编写呢？

这个文件夹下，意味着手机的最小宽度的dp值是360，我们把360dp等分成360等份，每一个设计稿中的像素，大概代表smallestWidth值为360dp的手机中的1dp，那么接下来的事情就很简单了，假如设计稿上出现了一个10px*10px的ImageView,那么，我们就可以不假思索的在layout文件中写下对应的尺寸。

```xml
<ImageView
	android:layout_width="@dimem/dp_10"
	android:layout_height="@dimem/dp_10" />
```

而这种diemns引用，在不同的values-sw<N>dp文件夹下的数值是不同的，

比如values-sw360dp：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<resources>
	<dimen name="dp_1">1dp</dimen>
	<dimen name="dp_2">2dp</dimen>
	<dimen name="dp_3">3dp</dimen>
	<dimen name="dp_4">4dp</dimen>
	<dimen name="dp_5">5dp</dimen>
	<dimen name="dp_6">6dp</dimen>
	<dimen name="dp_7">7dp</dimen>
	<dimen name="dp_8">8dp</dimen>
	<dimen name="dp_9">9dp</dimen>
	<dimen name="dp_10">10dp</dimen>
	...
	<dimen name="dp_356">356dp</dimen>
	<dimen name="dp_357">357dp</dimen>
	<dimen name="dp_358">358dp</dimen>
	<dimen name="dp_359">359dp</dimen>
	<dimen name="dp_360">360dp</dimen>
</resources>
```

values-sw400dp：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<resources>
	<dimen name="dp_1">1.1111dp</dimen>
	<dimen name="dp_2">2.2222dp</dimen>
	<dimen name="dp_3">3.3333dp</dimen>
	<dimen name="dp_4">4.4444dp</dimen>
	<dimen name="dp_5">5.5556dp</dimen>
	<dimen name="dp_6">6.6667dp</dimen>
	<dimen name="dp_7">7.7778dp</dimen>
	<dimen name="dp_8">8.8889dp</dimen>
	<dimen name="dp_9">10.0000dp</dimen>
	<dimen name="dp_10">11.1111dp</dimen>
	...
	<dimen name="dp_355">394.4444dp</dimen>
	<dimen name="dp_356">395.5556dp</dimen>
	<dimen name="dp_357">396.6667dp</dimen>
	<dimen name="dp_358">397.7778dp</dimen>
	<dimen name="dp_359">398.8889dp</dimen>
	<dimen name="dp_360">400.0000dp</dimen>
</resources>
```

当系统识别到手机的SmallestWidth值时，就会自动去寻找和目标数据最近的资源文件的尺寸。

### 3. 优点

1. 非常稳定，极低概率出现意外
2. 不会有任何性能的损耗
3. 适配范围可自由控制，不会影响其他三方库
4. 在插件的配合下，学习成本低

### 4. 缺点

- 那就是它是在Android 3.2 以后引入的，Google的本意是用它来适配平板的布局文件（但是实际上显然用于diemns适配的效果更好），不过目前所有的项目应该最低支持版本应该都是4.0了，所以，这问题其实也不重要了

- 无法覆盖全部机型，想覆盖更多机型的做法就是生成更多的资源文件，但这样会增加 App 体积，在没有覆盖的机型上还会出现一定的误差；如果想使用 sp ，也需要生成一系列的 dimens ，导致再次增加 App 体积；不能自动支持横竖屏切换时的适配，想自动支持横竖屏切换时的适配，需要使用 `values-w<N>dp` 或 屏幕方向限定符 再生成一套资源文件，再次增加体积。
- 在布局中引用 dimens 的方式，虽然学习成本低，但是在日常维护修改时较麻烦
- 侵入性高，如果项目想切换为其他屏幕适配方案，因为每个 Layout 文件中都存在有大量 dimens 的引用，修工作量巨大，切换成本非常高
- 不能以高度为基准进行适配，方案的名字本身就叫 **最小宽度限定符适配方案**，所以在使用这个方案之前就应该要知道这个方案只能以宽度为基准进行适配

### 5. 注意的问题

不需要先将 **px** 换算成 **dp**，把设计图的 **px** 总宽度设置成 **最小宽度基准值** 就可以了。

直接将 **最小宽度基准值** 和布局中的引用都以 **px** 作为单位就可以直接填写设计图上标注的 **px**。

### 6. 今日头条方案和SmallestWidth限定符适配方案对比

|      对比项目      |                 对比对象 A                 | 对比结果 |                     对比对象 B                      |
| :----------------: | :----------------------------------------: | :------: | :-------------------------------------------------: |
| 适配效果(越高越好) |              今日头条适配方案              |    ≈     | SW 限定符适配方案(在未覆盖的机型上会存在一定的误差) |
|  稳定性(越高越好)  |              今日头条适配方案              |    <     |                  SW 限定符适配方案                  |
|  灵活性(越高越好)  |              今日头条适配方案              |    >     |                  SW 限定符适配方案                  |
|  扩展性(越高越好)  |              今日头条适配方案              |    >     |                  SW 限定符适配方案                  |
|  侵入性(越低越好)  |              今日头条适配方案              |    <     |                  SW 限定符适配方案                  |
| 使用成本(越低越好) |              今日头条适配方案              |    <     |                  SW 限定符适配方案                  |
| 维护成本(越低越好) |              今日头条适配方案              |    <     |                  SW 限定符适配方案                  |
|      性能损耗      |        今日头条适配方案没有性能损耗        |    =     |            SW 限定符适配方案没有性能损耗            |
|       副作用       | 今日头条适配方案会影响一些三方库和系统控件 |    ≈     |         SW 限定符适配方案会影响 App 的体积          |

**SmallestWidth 限定符适配方案** 主打的是稳定性，在运行过程中极少会出现安全隐患，适配范围也可控，不会产生其他未知的影响，而 **今日头条适配方案** 主打的是降低开发成本、提高开发效率，使用上更灵活，也能满足更多的扩展需求。

## 参考

[一种极低成本的Android屏幕适配方式](https://mp.weixin.qq.com/s/d9QCoBP6kV9VSWvVldVVwA)

[Android 目前稳定高效的UI适配方案](https://mp.weixin.qq.com/s/X-aL2vb4uEhqnLzU5wjc4Q)

[今日头条屏幕适配方案终极版正式发布!](http://jessyan.me/autosize-publish/)

[Github：AndroidAutoSize](https://github.com/JessYanCoding/AndroidAutoSize)

[Github：生成values-sw](https://github.com/ladingwu/dimens_sw)

**欢迎关注我的公众号，持续分析优质技术文章**
![欢迎关注我的公众号](https://img-blog.csdnimg.cn/20190906092641631.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzMyMjM3NzE5,size_16,color_FFFFFF,t_70)

