---
layout: article
title: android7.0/7.1新特性小结
tags: android
---



### 多窗口

在android7.0中原生提供了多窗口模式和画中画模式，多窗口模式将屏幕分为上下或左右两块区域分别显示两个应用，画中画模式主要应用在android TV中，类似于windows中的多窗口。

#### 分屏

实现分屏功能只需要在AndroidManifest.xml中为application或特定的activity添加以下属性即可

```
android:resizeableActivity="true"
```
这样就可以在横屏或竖屏的情况下让应用分屏显示，效果图如下

+ 竖屏显示效果

![](http://upload-images.jianshu.io/upload_images/150061-0ca68bb937c85c5e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

+ 横屏显示效果

![](http://upload-images.jianshu.io/upload_images/150061-d2aed35acb79f67c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


#### 画中画

和分屏实现方式相同，只是需要设备支持，目前只是在android TV中有此功能，手机并没有开启该功能，需要进行特殊设置。实现方法也是在AndroidManifest.xml中为application或特定的activity添加以下属性

```
android:supportsPictureInPicture="true"
```

效果图如下

![](http://upload-images.jianshu.io/upload_images/150061-62524f214cc419f9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### Shortcut

android7.1中加入的shortcut类似于iOS的3D touch，只是将iPhone上的重按替换成了长按，长按后会弹出菜单，提供快捷操作，shortcut分为静态配置和动态配置。

![](http://upload-images.jianshu.io/upload_images/150061-08d1d9459d27cc4a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 静态配置
静态配置一般用于固定的选项，需要在资源文件重定义。在res下建立xml目录并新建shortcuts.xml文件，这里面定义shortcut的显示内容和跳转intent，代码如下

```
<shortcuts xmlns:android="http://schemas.android.com/apk/res/android">
    <shortcut
        android:shortcutId="static_1"
        android:enabled="true"
        android:icon="@mipmap/ic_launcher"
        android:shortcutShortLabel="@string/static_shortcut_short_name_1"
        android:shortcutLongLabel="@string/static_shortcut_long_name_1" >
        <intent
            android:action="android.intent.action.VIEW"
            android:targetPackage="com.test.newfeaturedemo"
            android:targetClass="MainActivity" />
        <categories android:name="android.shortcut.conversation" />
    </shortcut>
</shortcuts>
```

这里有几个主要属性用来设置菜单内容

+ id: shortcut的id，必须惟一，否则后面的会覆盖前面的
+ longLabel: 显示在弹出菜单中的label
+ shortLabel: 通过拖拽显示在桌面上的label
+ icon: 一个选项对应的icon
+ intent: 跳转intent，必须设置，否则会抛出```java.lang.NullPointerException: Shortcut Intent must be provided```的异常

之后还要在AndroidManifest中的activity中添加meta-data标签，代码如下

```
<activity android:name=".MainActivity">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>

    <meta-data
        android:name="android.app.shortcuts"
        android:resource="@xml/shortcuts"/>
</activity>
```
这样就可以通过资源文件的配置生成shortcut。需要注意的是，shortcut的数量并不能无限增加，当数量超过4时就会抛出如下异常
```
java.lang.IllegalArgumentException: Max number of dynamic shortcuts exceeded
```

#### 动态生成

动态生成shortcut由ShortcutInfo构成，ShortcutInfo有几个主要属性与静态配置相同，下面就通过ShortcutManager动态的生成两个shortcut

```
private void initShortcut() {
        ShortcutManager shortcutManager = getSystemService(ShortcutManager.class);

        ShortcutInfo shortcutInfo = new ShortcutInfo.Builder(this, "1")
                .setShortLabel("dynamic short 1")
                .setLongLabel("dynamic long 1")
                .setDisabledMessage("disable")
                .setIcon(Icon.createWithResource(this, R.mipmap.ic_launcher))
                .setIntent(new Intent(Intent.ACTION_VIEW, Uri.EMPTY, this, MainActivity.class))
                .build();

        ShortcutInfo shortcutInfo1 = new ShortcutInfo.Builder(this, "2")
                .setShortLabel("dynamic short 1")
                .setLongLabel("dynamic long 2")
                .setIcon(Icon.createWithResource(this, R.mipmap.ic_launcher))
                .setIntent(new Intent(Intent.ACTION_VIEW, Uri.EMPTY, this, SecondActivity.class))
                .build();

        List<ShortcutInfo> shortcutInfoList = new ArrayList<>();
        shortcutInfoList.add(shortcutInfo);
        shortcutInfoList.add(shortcutInfo1);

        shortcutManager.addDynamicShortcuts(shortcutInfoList);
    }
```

实现的效果图如下

![2016-12-20 19_55_31.gif](http://upload-images.jianshu.io/upload_images/150061-ef9755adf86fa7aa.gif?imageMogr2/auto-orient/strip)

此外还可以通过ShortcutManager中的updateShortcuts()和removeDynamicShortcuts()方法动态的更新和删除shortcut。

通过以上两种方法就可以在android7.1中加入快捷菜单，可以让用户在不进入应用时直接进入某个页面，实现类似iPhone中3D touch的效果。使用时也需要注意，添加数量的上限，将重要的快捷入口添加进来。还要注意体验问题，在Google的启动器中并不能标识出哪些应用有shortcut，尤其在二级菜单中如果没有就会变成长按创建快捷方式，体验很不好，在没有解决手势冲突之前，还是一把双刃剑。

### Quick Settings Tile

在android7.0中Google在下拉菜单中加入了自定义快捷菜单的功能，用户可以在快捷菜单中添加快捷键。

QuickSettingTile是由服务实现的，需要实现一个集成自TileService的服务，这里提供了几个主要的生命周期
+ onTileAdded: 用户将tile添加到快速设置面板上
+ onTileRemoved: 从快速设置面板上移除tile
+ onStartListening: 快速设置菜单展开，只有添加到菜单里才会回调
+ onStopListening: 快速设置菜单收起，只有添加到菜单里才会回调
+ onClick: 点击tile的事件

实现一个QuickSettingTile的代码如下

```
public class QuickSettingsTileService extends TileService {

    @Override
    public void onTileAdded() {
        super.onTileAdded();
        Toast.makeText(this, "添加到快速设置菜单中", Toast.LENGTH_SHORT).show();
    }

    @Override
    public void onTileRemoved() {
        super.onTileRemoved();
        Toast.makeText(this, "从快速设置菜单中移除", Toast.LENGTH_SHORT).show();
    }

    @Override
    public void onStartListening() {
        super.onStartListening();
        Toast.makeText(this, "下拉快速设置菜单", Toast.LENGTH_SHORT).show();
    }

    @Override
    public void onStopListening() {
        super.onStopListening();
        Toast.makeText(this, "收起快速设置菜单", Toast.LENGTH_SHORT).show();
    }

    @Override
    public void onClick() {
        super.onClick();
        Intent intent = new Intent(this, MainActivity.class);
        startActivity(intent);
    }
}
```

既然是一个服务，就需要在AndroidManifest中注册这个服务，代码如下

```
<service android:name=".QuickSettingsTileService"
    android:label="QuickSettingTile"
    android:icon="@mipmap/ic_launcher"
    android:permission="android.permission.BIND_QUICK_SETTINGS_TILE">
    <intent-filter>
        <action android:name="android.service.quicksettings.action.QS_TILE" />
    </intent-filter>
</service>
```

之后就可以将自定义的tile添加到下拉设置菜单中了，效果如下

![2016-12-20 19_56_20.gif](http://upload-images.jianshu.io/upload_images/150061-2aa1fc7294d5e2b7.gif?imageMogr2/auto-orient/strip)

### 小结
以上就是android7.x中新加的UI上的功能，分屏可以让用户在同一个屏幕上开启两个应用，在大屏手机上有明显优势。shortcut快捷入口可以让用户在不启动应用的时候直接进入某个页面，提升使用效率。QuickSettingTile可以让用户在快速设置使用应用入口，应用可以根据需求添加不同入口，提升用户体验。
