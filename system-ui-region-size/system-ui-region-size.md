# 安卓界面装饰物的大小计算

## 结构图

![system-ui](image/system-ui.png)

### 部件说明

① System StatusBar 系统默认的状态栏

② System Toolbar 主题携带的标题栏

③ Custom StatusBar 使用 CoordinatorLayout 携带的状态栏，一般和 ④ 一块使用。

④ Custom Toolbar 当我们不想使用系统的 toolbar 或者需要实现某些效果时，会使用 `Theme.AppCompat.Light.NoActionBar` 之类的主题，并通过 `setSupportActionBar(toolbar)` 添加自定义的 toolbar，为了描述方便，暂且称之为 Custom Toolbar（如果使用系统自动生成的版本，记得将 Toolbar 的 `android:background="?attr/colorPrimary"` 去掉，否则会加剧重绘现象）。

③、④ 和 ⑤ 三部分合起来对应 id 为 `Window.ID_ANDROID_CONTENT` 的 `FrameLayout`。

![system-toolbar-contentview](image/system-toolbar-contentview.png)

图 1 使用系统的 ToolBar



![system-toolbar-contentview](image/custom-toolbar-contentview.png)

图 2 使用自定义的 Toolbar



⑥ NavigationBar 导航栏

指的是虚拟按键，不是实体按键。



## 为什么要计算这些部件的尺寸呢

理由1：onTouch 中我们拿到的坐标都是从屏幕左上角开始的。有时，在自定义 view 时需要绘制和触摸有关的东西时，我们就需要手动控制绘制的坐标，而绘制的左上角却从 ③ 的左上角开始，坐标为 (0, 0) ，所以需要坐标转换。

其他理由我暂时还不知道，欢迎补充。



## 背景知识补充

1. `DecorView` ：整个界面，在 onCreate() 执行后，可以通过 `getWindow().getDecorView()` 获得，进而获得 Width、Height 大小。在 view 中也可以通过 `getRootView()` 获得。

2. `WindowVisibleDisplayFrame` ：包括 ②、③、④、⑤ 四个部分，即除了系统的状态栏和虚拟按键外的全部区域。可以通过 `getWindow().getDecorView().getWindowVisibleDisplayFrame(rect)` 获得该区域的位置矩形，格式如下：`Rect(0, 50 - 720, 1184)` 。

3. `Window.ID_ANDROID_CONTENT` ：上文提到的，对应的 Framelayout 由 ③、④、⑤ 三部分组成。

   当添加一个 View，包括其父 View 在内，都使用 `match_parent` 标志宽高时，onMeasure() 执行后，我们也可以通过 `getWidth()` 和 `getHeight()` 获得 ③、④、⑤ 三个部分的宽高，不过 `getTop()` 获取到的值为 0 ，这就没什么用。

4. `mWindowManager.getDefaultDisplay().getXX()` ：除了 ⑥ 导航栏以外的全部。提供了 `getWidth()`、`getHeight()` 两个方法来获取大小，从 API 13 起被 deprecated，官方推荐使用 `getSize(out)` 来获取宽高。也可以通过 `getMetrics` 方法间接得到。三种写法如下：

   ```java
   WindowManager wm = (WindowManager) getSystemService(WINDOW_SERVICE);
   Log.i(TAG, "width: " + wm.getDefaultDisplay().getWidth()); // width: 720
   Log.i(TAG, "height: " + wm.getDefaultDisplay().getHeight()); // height: 1184
   ```

   ```java
   Point out = new Point();
   wm.getDefaultDisplay().getSize(out);
   Log.i(TAG, "getSize: " + out.toString()); // getSize: Point(720, 1184)
   ```

   ```java
   DisplayMetrics dm = new DisplayMetrics();
   wm.getDefaultDisplay().getMetrics(dm);
   Log.i("take", "dm.widthPixels: " + dm.widthPixels); // dm.widthPixels: 720
   Log.i("take", "dm.heightPixels: " + dm.heightPixels); // dm.heightPixels: 1184
   ```

5. `wm.getDefaultDisplay().getRealXX()` ：也是整个界面。有两种写法：

   ```java
   Point real = new Point();
   if (Build.VERSION.SDK_INT >= 17) {
     wm.getDefaultDisplay().getRealSize(real);
     Log.i("take", "getRealSize: " + real.toString()); // getRealSize: Point(720, 1280)
   }
   ```

   ```java
   DisplayMetrics dmReal = new DisplayMetrics();
   if (Build.VERSION.SDK_INT >= 17) {
     wm.getDefaultDisplay().getRealMetrics(dmReal);
     Log.i("take", "dmReal.widthPixels: " + dmReal.widthPixels); // dmReal.widthPixels: 720
     Log.i("take", "dmReal.heightPixels: " + dmReal.heightPixels); // dmReal.heightPixels: 1280
   }
   ```

   由于在 API 17 及以上才能使用，还不如直接使用 DecorView 来获取。



## 部件尺寸的计算

### System StatusBar

