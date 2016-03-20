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

