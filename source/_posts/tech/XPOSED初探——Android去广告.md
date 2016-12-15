---
title: XPOSED初探——Android去广告
date: 2016-11-12 16:02:16
tags:
- XPOSED
- Android
---

最近学习了一下XPOSED框架，入门参照了这系列博文 [[Android 开发] Xposed 插件开发](http://blog.csdn.net/niubitianping/article/details/52571438)。

总结起来，XPOSED框架首先搭建好环境，包括引用`xposed`jar包，创立入口点`xposed_init`文件。

我所要去除的广告是应用的首屏广告。很多应用不会直接进入主界面，而是先显示一个过渡界面，这个过渡界面一般被称为`SplashActivity`。这是因为有的应用在第一次启动时，需要加载的资源比较多，如果不使用过渡界面，在加载资源的这段时间手机会黑屏，用户体验较差，使用过渡界面就可以解决这个问题。过渡界面一般用来显示应用Logo和提示加载资源的进度，但是很多应用机wu智chi的滥用这个技术，把过渡界面用来展示广告，而且还要显示好几秒，远远大于实际加载资源所需要的时间。

使用`XPOSED`这个神器可以去除这种首屏广告。`XPOSED`不同于普通的Android应用，它只是一个框架，不提供具体的应用，而开发者则可以使用它提供的API编写模块实现具体的功能。`XPOSED`的功能十分强大，它可以Hook任意的Java函数，提供类似AOP的功能。

具体的编写方法这里就略过，只讲思路。要跳过广告，就是跳过广告的`Activity`直接进入真正内容的`Activity`，使用XPOSED Hook住广告`Activity`的`onStart`方法，在`onStart`之前进行切入，启动主`Activity`并`finish()`广告`Activity`即可。

代码如下，
```Java
public class Main implements IXposedHookLoadPackage {
    protected void replaceLauncherActivity(final String activityName, ClassLoader classLoader) {
        XposedHelpers.findAndHookMethod("android.app.Activity",
                classLoader,
                "onStart",
                new XC_MethodHook() {
                    @Override
                    protected void beforeHookedMethod(MethodHookParam methodHookParam) throws Throwable {
                        // 获取当前activity
                        Activity activity = (Activity) methodHookParam.thisObject;

                        // 获取launcher activity，即广告activity
                        PackageManager pm = activity.getPackageManager();
                        Intent launcherIntent = pm.getLaunchIntentForPackage(activity.getPackageName());

                        // 如果当前activity为广告activity，则启动主activity并结束广告activity
                        Intent intent = activity.getIntent();
                        if (intent.getComponent().flattenToString().equals(
                                launcherIntent.getComponent().flattenToString())) {
                            Intent intent2 = new Intent();
                            intent2.setClassName(activity, activityName);
                            activity.finish();
                            activity.startActivity(intent2);
                        }
                    }
                }
        );
    }

    @Override
    public void handleLoadPackage(XC_LoadPackage.LoadPackageParam loadPackageParam) throws Throwable {
        // 一系列的广告去除定义
        if (loadPackageParam.packageName.equals("com.snda.wifilocating")) {
            replaceLauncherActivity("com.lantern.launcher.ui.MainActivityICS", loadPackageParam.classLoader);
        } else if (loadPackageParam.packageName.equals("com.sunrise.scmbhc")) {
            replaceLauncherActivity("com.sunrise.scmbhc.ui.activity.home.HomeActivity", loadPackageParam.classLoader);
        } else if (loadPackageParam.packageName.equals("ctrip.android.view")) {
            replaceLauncherActivity("ctrip.android.publicproduct.home.view.CtripHomeActivity", loadPackageParam.classLoader);
        } else if (loadPackageParam.packageName.equals("com.feheadline.news")) {
            replaceLauncherActivity("con.feheadline.news.ui.activity.MainActivity", loadPackageParam.classLoader);
        }
    }
} 
```

再记录一下遇到的几个坑，
1. XposedBridge.log，日志打不出来，`四川掌上营业厅`的日志打不出来，其它应用都可以，而且是在`handleLoadPackage`可以，到了`XC_MethodHook`中就不行了，怀疑是App把日志给禁用了。
2. 程序死活无效，后来看Xposed日志发现，在`Android Studio`里面，必须要把`Instant Run`关了才行。
3. Xposed程序编写，最大的难度在于它靠Hook函数实现功能。也就是说，你得知道实现某个功能需要Hook哪个函数，而被Hook的函数一般是第三方的App中的，你拿不到源码，只能通过反编译来寻找，因此写Xposed模块，对反编译Android程序的要求挺高的。