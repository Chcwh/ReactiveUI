# ReactiveUI 路由和导航

许多应用，特比是那些屏幕较小的移动应用，采用基于页面的导航架构——用户可以沿着一个视图向前或向后水平导航。一些平台，比如 WP8，有一个硬件返回按钮，特别为方便导航设计的，而其他平台，比如 WinRT 在页面上有一个返回按钮。

但是，所有这些平台都是为视图优先模型设计的，视图既是框架自动创建的，也是导航的核心组件。尽管这种方法是最直接的，它也是不可测试的。 

一种可选的方法是创建导航 “栈” 作为视图模型的栈，并允许相关视图在运行时，基于视图模型中的数据，自动创建。这种方法也允许我们在被操作系统要求挂起时，更容易序列化应用的状态，这对于移动平台来说特别重要。

### 路由支持在哪里？

基于视图模型的路由支持 Xamarin.Forms，WinRT，WP8 以及 WPF 桌面应用。路由在 iOS 和 Android 也是可能的，无需 Xamarin.Forms，但是支持是初级且容易破碎的。

### 如何设置路由

1. 为你的主窗口设置一个视图模型，并让实现 `IScreen` （它应该是一个 ReactiveObject）。在设个类中，导航到你第一个视图模型。 

1. 让你应用中代表页的视图模型派生自 `IRoutableViewModel`
   
1. 向你的主页面或窗体添加一个 RoutedViewHost 实例，并将其绑定到 IScreen 实例上。

1. (在 WP8 / WinRT 上) - 设置 ReactiveUI.Mobile ，这样在 WP8 上面，就能够正确处理硬件返回按钮了。

### 关于路由是如何工作的基础部分

基于视图模型的导航由几部分构成：

##### 视图模型

* `IRoutableViewModel` - 一个能够指示指定视图模型是一个能够被导航到的视图模型（比如，它是一个能够导航回去的页）。使用该接口标记的视图模型能够被导航到，通过 `IScreen.Router.Navigate.Execute`。

  该类有一个 `UrlPathSegment` 属性，在有些平台上被用作该框架的标题（比如，在 iOS 中的一个 UINavigationViewController 中）。

* `IScreen` - 一个标记视图模型是导航栈返回的主页。在大多是应用中，可能只有一个这样视图模型，其在概念上表示**应用主窗体的视图模型**。

* `RoutingState` - 该类表示**返回栈** —— 当前可用的视图模型的栈。栈上最后一个视图模型是当前显示的那个。RoutingState 也提供了几个 ReactiveCommand 用于前后导航。

##### 视图

* `IViewFor<TViewModel>` - 由于每个 `IRoutableViewModel` 都会使用 ReactiveUI 的视图定位系统，视图需要预先注册。

* `RoutedViewHost` - RoutingState 的“视图”；其将会订阅到 RoutingState 并当新视图模型推送入导航栈时，该视图将会被显示。该类与 `ViewModelViewHost` 类似，但是跟踪的是路由状态。

### 测试导航

基于视图模型进行导航的一个重要好处是，在页面间导航可以在单元测试中验证。因为 RoutingState 相对来说是一个简单数据对象，对诸如“在登陆页的 OK 按钮被单击之后，确保进入欢迎页面”的验证就能相对简单的完成了。
