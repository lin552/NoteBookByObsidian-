---
创建时间: "2026-01-06 14:10:24"
作者: wangxiaoming
tags:
---
#### 一、核心组件关系图
```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   Activity/     │────│  NavHostFragment │────│   NavController │
│   Fragment      │    │                  │    │                 │
└─────────────────┘    └──────────────────┘    └─────────────────┘
                                                        │
                              ┌─────────────────────────┼─────────────────────────┐
                              │                         │                         │
                    ┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
                    │  NavigatorProvider│────│   NavGraph       │────│  Destinations   │
                    └─────────────────┘    └──────────────────┘    └─────────────────┘
                              │
                    ┌─────────────────┐
                    │  Navigators    │
                    │ (各种Navigator) │
                    └─────────────────┘
```
#### 二、组件详解
##### 1)`NavHostFragment` -- 导航宿主
```xml
<!-- 在布局文件中定义 -->
<androidx.fragment.app.FragmentContainerView
    android:id="@+id/nav_host_fragment"
    android:name="androidx.navigation.fragment.NavHostFragment"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    app:defaultNavHost="true"
    app:navGraph="@navigation/nav_graph" />
```
**作用**：
- **容器作用**：作为 Fragment 的容器，承载导航过程中的各个 Fragment
- **生命周期管理**：管理 `NavController` 的生命周期
- **默认 `NavHost`**：`app:defaultNavHost="true"`表示处理系统返回键
**关键方法**：
```java
// 获取 NavController
NavHostFragment navHostFragment = (NavHostFragment) getSupportFragmentManager()
    .findFragmentById(R.id.nav_host_fragment);
NavController navController = navHostFragment.getNavController();
```

##### 2) `NavController` -- 导航控制器（核心）
**作用**：
- **导航执行**：处理所有的导航操作（`navigate()`, `popBackStack()`）
- **状态管理**：跟踪当前的导航状态（当前目的地、返回栈等）
- **生命周期感知**：与 `NavHostFragment` 的生命周期同步
**核心方法**：
```java
public class NavController {
    // 导航到指定目的地
    public void navigate(int resId);
    public void navigate(int resId, Bundle args);
    public void navigate(NavDirections directions);
    
    // 返回栈管理
    public boolean popBackStack();
    public boolean popBackStack(int destinationId, boolean inclusive);
    
    // 获取当前状态
    public NavDestination getCurrentDestination();
    public NavBackStackEntry getPreviousBackStackEntry();
    
    // 获取关联的组件
    public NavigatorProvider getNavigatorProvider();
    public NavGraph getGraph();
}
```

##### 3）`NavigatorProvider` -- 导航器提供者
**作用**
- **Navigator 注册表**：管理所有可用的 Navigator
- **Navigator 查找**：根据类型查找对应的 Navigator
- **扩展点**：允许注册自定义的 Navigator
**工作原理**：

```java
public class NavigatorProvider {
    private final HashMap<String, Navigator<?>> mNavigators = new HashMap<>();
    
    // 注册 Navigator
    public void addNavigator(Navigator<? extends NavDestination> navigator);
    public void addNavigator(String name, Navigator<? extends NavDestination> navigator);
    
    // 获取 Navigator
    @SuppressWarnings("unchecked")
    public <T extends Navigator<?>> T getNavigator(Class<T> navigatorClass);
    public Navigator<? extends NavDestination> getNavigator(String name);
}
```

##### 4) Navigator -- 导航执行器（抽象基类）
**作用**：
- **具体导航实现**：每个 Navigator 负责特定类型目的地的导航逻辑
- **事务管理**：执行 Fragment/Activity 的事务操作
- **状态跟踪**：跟踪自己管理的目的地状态
**主要实现类**：

```java
// Fragment 导航器
public class FragmentNavigator extends Navigator<FragmentNavigator.Destination> {
    // 负责 Fragment 的添加、替换、移除等操作
}

// Activity 导航器  
public class ActivityNavigator extends Navigator<ActivityNavigator.Destination> {
    // 负责 Activity 的启动
}

// DialogFragment 导航器
public class DialogFragmentNavigator extends Navigator<DialogFragmentNavigator.Destination> {
    // 负责 DialogFragment 的显示
}
```

##### 5) `NavGraph` -- 导航图
**作用**
- **导航蓝图**：定义所有可能的导航路径和目的地
- **目的地集合**：包含所有 Destination 节点
- **起始点定义**：指定应用的入口页面
XML 定义示例
```xml
<navigation xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/nav_graph"
    app:startDestination="@id/homeFragment">  <!-- 起始目的地 -->
    
    <!-- 目的地节点 -->
    <fragment
        android:id="@+id/homeFragment"
        android:name="com.example.HomeFragment"
        android:label="Home"
        tools:layout="@layout/fragment_home">
        
        <!-- 动作边 -->
        <action
            android:id="@+id/action_home_to_detail"
            app:destination="@id/detailFragment"
            app:enterAnim="@anim/slide_in_right" />
    </fragment>
    
    <fragment
        android:id="@+id/detailFragment"
        android:name="com.example.DetailFragment"
        android:label="Detail" />
</navigation>
```

##### 6) `NavDesitination` -- 导航目的地
**作用**：
- **目标定义**：表示导航的一个具体目标（Fragment/Activity）
- **参数配置**：定义目的地需要的参数
- **动画配置**：配置进入/退出动画
**类型体系**：

```java
// 所有目的地的基类
public abstract class NavDestination {
    protected String mClassName;  // 对应的类全名
    protected int mId;            // 目的地 ID
    protected Bundle mArgs;       // 默认参数
}

// Fragment 专用目的地
public class FragmentNavigator.Destination extends NavDestination {
    public void setClassName(String className);
    public String getClassName();
}

// Activity 专用目的地  
public class ActivityNavigator.Destination extends NavDestination {
    public void setComponentName(ComponentName componentName);
}
```

#### 三、导航执行流程
```java
// 1. 用户点击按钮触发导航
button.setOnClickListener(v -> {
    // 2. 获取 NavController
    NavController navController = Navigation.findNavController(view);
    
    // 3. 执行导航
    navController.navigate(R.id.action_home_to_detail, args);
});

// 内部执行流程：
// NavController.navigate()
//   → NavigatorProvider.getNavigator(FragmentNavigator.class)
//   → FragmentNavigator.navigate()
//     → FragmentTransaction.replace()
//     → 更新 NavGraph 中的当前目的地状态
//     → 更新返回栈
```
初始化流程
```java
// 1. Activity 创建 NavHostFragment
NavHostFragment navHost = (NavHostFragment) getSupportFragmentManager()
    .findFragmentById(R.id.nav_host_fragment);

// 2. NavHostFragment 创建 NavController
NavController navController = navHost.getNavController();

// 3. NavController 加载 NavGraph
navController.setGraph(R.navigation.nav_graph);

// 4. NavController 通过 NavigatorProvider 获取各种 Navigator
NavigatorProvider provider = navController.getNavigatorProvider();
FragmentNavigator fragmentNavigator = provider.getNavigator(FragmentNavigator.class);

// 5. 导航时，NavController 委托给对应的 Navigator 执行具体操作
```
#### 四、各组件的职责边界

| 组件                | 主要职责            | 关键能力                              |
| ----------------- | --------------- | --------------------------------- |
| NavHostFragment   | 容器和生命周期管理       | 承载 Fragment、管理 NavController 生命周期 |
| NavController     | 导航控制和状态管理       | 执行导航、管理返回栈、状态跟踪                   |
| NavigatorProvider | Navigator 注册和查找 | 管理所有 Navigator 实例                 |
| Navigator         | 具体导航实现          | 执行 Fragment/Activity 事务           |
| NavGraph          | 导航结构定义          | 定义目的地和导航路径                        |
| NavDestination    | 单个目标定义          | 描述具体的导航目标                         |

##### 层级关系
- `NavHostFragment` 在最外层，作为容器
- `NavController` 是大脑，负责决策和控制
- `NavigatorProvider`* 是工具箱，管理各种导航工具
- `Navigator` 是具体的工具，执行实际操作
- `NavGraph`​ 是地图，告诉系统有哪些路可以走
- `NavDestination`​ 是地图上的地点

##### 数据流
```
用户操作 → NavController → NavigatorProvider → Navigator → FragmentTransaction → UI更新
```