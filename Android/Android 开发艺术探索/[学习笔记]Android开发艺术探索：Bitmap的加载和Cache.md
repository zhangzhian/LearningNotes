# [学习笔记]Android开发艺术探索：Bitmap的加载和Cache

## Bitmap的高效加载

**BitmapFactory**类提供四种方法：decodeFile、decodeResource、decodeStream和decodeByteArray；其中decodeFile和decodeResource间接的调用了decodeStream方法；这四个方法最终在Android底层实现。

如何高效的加载Bitmap？核心思想：按需加载；很多时候ImageView并没有原始图片那么大，所以没必要加载原始大小的图片。采用**BitmapFactory.Options**来加载所需尺寸的图片。 通过BitmapFactory.Options来缩放图片，主要是用到了它的inSampleSize参数，即采样率。 inSampleSize应该为2的指数，如果不是系统会向下取整并选择一个最接近2的指数来代替；缩放比例为1/（inSampleSize的二次方）。

**Bitmap内存占用**：拿一张1024 * 1024像素的图片来说，假定采用ARGB8888格式存储，那么它占用的内存为1024 * 1024 * 4，即4MB。

通过采样率高效地加载图片，代码示例：

```java
 public static Bitmap decodeBitmapFromResource(Resources res, int resId, int reqWidth, int reqHeight) {
        BitmapFactory.Options options = new BitmapFactory.Options();
        //1. 将BitmapFactory.Options的inJustDecodeBounds参数设置为true并加载图片。
        options.inJustDecodeBounds = true;
        BitmapFactory.decodeResource(res, resId, options);

        //2. 根据采样率的规则并结合目标View的所需大小计算出采样率inSampleSize。
        options.inSampleSize = calcuateInSampleSize(options, reqWidth, reqHeight);

        //3. 将BitmapFactory.Options的inJustDecodeBounds参数设置为false，然后重新加载图片。
        options.inJustDecodeBounds = false;
        return BitmapFactory.decodeResource(res, resId, options);
    }
     //获取采样率
    private static int calcuateInSampleSize(BitmapFactory.Options options, int reqWidth, int reqHeight) {
        int width = options.outWidth;
        int height = options.outHeight;
        int inSampleSize = 1;

        if (height > reqHeight || width > reqWidth) {
            int halfHeight = height / 2;
            int halfWidth = width / 2;
            while ((halfHeight / inSampleSize) >= reqHeight && (halfWidth / inSampleSize) >= reqWidth) {
                inSampleSize *= 2;
            }
        }
        return inSampleSize;
    }
// 显示图片
Bitmap bitmap = DecodeBitmap.decodeBitmapFromResource(getResources(), R.mipmap.haimei2, 400, 400);
imageView.setImageBitmap(bitmap);
```

当inJustDecodeBounds参数为true时，BitmapFactory只会解析图片的原始宽/高信息，并不会真正的加载图片。需要注意这时候BitmapFactory获取的图片宽/高信息和图片的位置与程序运行的设备有关。



## Android的缓存策略

如何减少流量消耗？**缓存**。当程序第一次从网络上加载图片后，将其缓存在存储设备中，下次使用这张图片的时候就不用再从网络从获取了。一般情况会把图片存一份到内存中，一份到存储设备中，如果内存中没找到就去存储设备中找，还没有找到就从网络上下载。

目前常用的缓存算法是**LRU**，是近期最少使用算法，当缓存满时，优先淘汰那些近期最少使用的缓存对象。采用LRU算法的缓存有两种：LRUCache（内存缓存）和DiskLruCache（存储缓存）。

**LruCache**是Android3.1所提供的一个缓存类，通过support-v4兼容包可以兼容到早期的Android版本。LruCache是一个泛型类，是线程安全的，内部采用LinkedHashMap以强引用的方式存储外界缓存对象，并提供get和put方法来完成缓存的获取和添加操作，当缓存满时，LruCache会移除较早的使用的缓存对象。LruCache初始化时需重写**sizeOf**方法，用于计算缓存对象的大小。

- **强引用**：直接的对象引用
- **软引用**：当一个对象只有软引用存在的时候，系统内存不足的时此对象会被GC回收。
- **弱引用** : 当一个对象只有弱引用存在的时候，此对象随时会被GC回收。

**DiskLruCache**用于实现磁盘缓存，DiskLruCache得到了Android官方文档推荐，但它不属于Android SDK的一部分

1. DiskLruCache不能通过构造方法来创建，提供了 open() 方法来创建自身。
2. 通过 edit(String key) 方法来获取Editor对象
3. 通过Editor的 newOutputStream(int cacheIndex) 方法打开一个文件输出流，然后通过这个流将下载的图片写入到文件系统中。
4. 调用Editor的 commit() 来提交写入操作。如果图片下载过程中发生异常，通过Editor 的 abort() 来回退整个操作。

DiskLruCache的缓存查找：

1. 将图片url转换为key，通过 get(String key) 方法得到一个Snapshot对象。
2. 通过Snapshort对象即可得到缓存的文件输入流
3. 从输入流中得到Bitmap对象
4. 通过 BitmapFactory.Options 对象来加载一张缩放后的图片，对FileInputStream的缩放存在问题，因为FileInputStream是一种有序的文件流，而两次 decodeStream 调用影响了文 件流的位置属相，导致第二次 decodeStream 时得到的是null。所以一般通过文件流来得到 对应的文件描述符，通过 BitmapFactory.decodeFileDescriptor() 来加载一张缩放后的图片。

自己实现一个ImageLoader，包含

1. 图片压缩功能
2. 内存缓存和磁盘缓存
3. 同步加载和异步加载的接口设计

## ImageLoader的使用

优化列表卡顿现象

1. 不要在getView中执行耗时操作，不要在getView中直接加载图片。
2. 控制异步任务的执行频率：如果用户刻意频繁上下滑动，getView方法会不停调用，从而产生大量的异步任务。可以考虑在列表滑动停止加载图片；给ListView或者GridView设置setOnScrollListener并在OnScrollListener的onScrollStateChanged方法中判断列表是否处于滑动状态，如果是的话就停止加载图片。
3. 大部分情况下，可以使用硬件加速解决莫名卡顿问题，通过设置android:hardwareAccelerated="true"即可为Activity开启硬件加速。

