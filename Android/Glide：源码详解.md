# Glide: 源码详解 | 通过源码深入理解Glide

> 本文聚焦于Glide的源码

## 一、简介

[Glide的GitHub](https://muyangmin.github.io/glide-docs-cn/)

Glide是一个快速高效的Android图片加载库，注重于平滑的滚动。Glide提供了易用的API，高性能、可扩展的图片解码管道（decode pipeline），以及自动的资源池技术。

### 1. 简单使用

1、添加依赖：

```groovy
repositories {
  google()
  jcenter()
}

dependencies {
  implementation 'com.github.bumptech.glide:glide:4.11.0'
  annotationProcessor 'com.github.bumptech.glide:compiler:4.11.0'
}
```

2、添加网络权限

```xml
<uses-permission android:name="android.permission.INTERNET" />
```

3、一句代码加载图片到ImageView

```java
Glide.with(fragment).load(url).into(imageView);
```

进阶一点的用法，参数设置

```java
RequestOptions options = new RequestOptions()
            .placeholder(R.drawable.ic_launcher_background)
            .error(R.mipmap.ic_launcher)
            .diskCacheStrategy(DiskCacheStrategy.NONE)
    		.override(200, 100);
    
Glide.with(this).load(url).apply(options).into(imageView);
```

### 2. 对比

**Glide：**

- 多种图片格式的缓存，适用于更多的内容表现形式（如Gif、WebP、缩略图、Video）
- 生命周期集成（根据Activity或者Fragment的生命周期管理图片加载请求）
- 高效处理Bitmap（bitmap的复用和主动回收，减少系统回收压力）
- 高效的缓存策略，灵活（Picasso只会缓存原始尺寸的图片，Glide缓存的是多种规格），加载速度快且内存开销小（默认Bitmap格式的不同，使得内存开销是Picasso的一半）

**Fresco：**

- 最大的优势在于5.0以下(最低2.3)的bitmap加载。在5.0以下系统，Fresco将图片放到一个特别的内存区域(Ashmem区)
- 大大减少OOM（在更底层的Native层对OOM进行处理，图片将不再占用App的内存）
- 适用于需要高性能加载大量图片的场景

### 3. 进阶使用

关于Glide的详细使用请查看这个官方 [中文文档](https://muyangmin.github.io/glide-docs-cn/)，这里就不赘述了。

## 二、核心流程



### 1. with()



### 2. laod()



### 3. into()



## 三、缓存机制





## 四、回调与监听





## 五、图片变换





## 六、自定义模块











