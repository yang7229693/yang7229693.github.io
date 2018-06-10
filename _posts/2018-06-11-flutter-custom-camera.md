---
layout: post
title: Flutter 相机定制
categories: reading-notes
excerpt: 本文主要介绍Flutter中相机定制，使用第三方库中出现的问题。
tag: [Flutter, Layout, Widget, Camera]
comments: true
image:
  feature: pic-book-1.jpg
---

Flutter中与硬件相关的部分，一直都挺蛋疼的。方案基本上有两种，自己写，或者等出相关的库。

最近做的一个项目中，需要对相机做定制。有过相关模块开发经验的，就知道这种需求并不简单，况且是这种跨平台解决方案的初期。

需求来了，怎么办呢？那就只能硬着头皮上了。先去pub上找找，有没有可以使用的库。初步挑到两个库，一个camera，另一个是image_picker。

image_picker试了下，基本上就pass了，只能调用系统相机或者选择相册，相机相关部分，肯定是没法使用。相册部分倒是可以拿来使用。

camera试着运行了下demo，感觉这个库可以使用，直接将相机预览封装成一个flutter widget。我们可以很方便的在上面进行各种定制。

设计图上需要相机全屏显示，试着在demo上修改成全屏，悲剧出现了，风一般的拉伸效果。添加一个调取相机的页面，在退出相机页面后，demo置后台，切换到前台的时候，Android这边crash了，试了N多次，100%的crash，我勒个擦，谷歌官方的插件写的这么随意~

然后，重点来了，`本文主要是解决这两个问题，一个是全屏显示的问题，另一个则是crash问题`。

先上一张Android端的拍照效果图：

![Flutter 相机定制](http://whysodiao.com/images/flutter-camera-sample.jpg)

## Android端全屏拉伸问题

对于这种相机拉伸问题，做过相机定制相关的，都会知道是预览的分辨率选择错误导致的，知道这一点过后，修改起来就简单的多了。直接拉camer plugin的源码，Android的实现，相对还是比较简单，一个文件700来行代码。找到计算预览尺寸的方法。

```
    private void computeBestPreviewAndRecordingSize(
        StreamConfigurationMap streamConfigurationMap, Size minPreviewSize, Size captureSize) {
        ....
    }
```

考虑到以后camera插件升级的问题，直接单独新建一个文件进行最佳预览尺寸的计算，然后在调用处进行替换即可。

## Android端前后台切换必现crash问题

Android端前后台切换的问题，查看log发现是resume过后崩溃的，直接撸源代码，发现插件监听了Activity的生命周期，在resume的时候进行了open操作。

```
  @Override
  public void onActivityResumed(Activity activity) {
    if (requestingPermission) {
      requestingPermission = false;
      return;
    }
    if (activity == CameraPlugin.this.activity) {
      if (camera != null) {
        camera.open(null);
      }
    }
  }
```

问题来了，关键是，插件没有进行unregister操作，在退出相机页面过后，调用dispose方法，会将camera关闭，并且将cameraDevice置为null，其他生命周期回调中调用cameraDevice的都会crash。onActivityResumed调用camer.open也会crash。这些crash的根本原因是因为没有将回调unregister掉。了解这些过后，修改起来就简单了，在dispose的时候，将插件的生命周期回调给unregister掉。修改完成后，试下效果，果然都没有crash。

## 后话

相关代码段我就不贴出来了，关于全屏预览尺寸的计算，网上太多的资料了，将previewSize计算正确即可。第二个crash的问题，添加一个unregisterActivityLifecycleCallbacks即可。对于修改的地方，做好注释，后续升级插件合入的时候不错乱即可。

目前来看Flutter第三方真的很差，如果只是自己个人项目，或者一些偏向于数据展示的项目，可以试一下。但是对于商业项目，尤其是硬件依赖性比较强的，还是建议一段时间过后再看看。