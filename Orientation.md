控制器控制屏幕旋转
=======================
view controller全部的旋转
-----------------------
### iOS屏幕旋转控制在View Controller里面，包含三种controller。
	其一：UIViewController及其子类。
	其二：UINavigationController及其子类。
	其三：UITabBarController及其子类。
	每一种controller及其子类都可以写屏幕旋转控制代码。但是记住一个原则，谁加载谁获得屏幕控制的权限，被加载的controller如果要添加自适应代码，可以在- (void)willRotateToInterfaceOrientation:(UIInterfaceOrientation)toInterfaceOrientation duration:(NSTimeInterval)duration函数中实现。
  
view controller中部分旋转  
-----------------------
	部分旋转同样需要实现controller中的旋转函数，只是返回值为NO。但在返回之前需要给消息中心发送一个消息，如：[[NSNotificationCenter defaultCenter] postNotificationName:kNotificationOrientationChange object:nil]
	当然，在这之前应该在viewDidLoad函数中加入通知消息中心监听kNotificationOrientationDidChange，[[NSNotificationCenter defaultCenter] addObserver:self selector:@Selector(orientationDidChange:) name:kNotificationOrientationChange object:nil]
	之后就需要- (void)orientationDidChange:(NSNotification *)notification函数中实现部分视图旋转的控制代码。
	不过，最后别忘了在dealloc函数中取消监听，否则会造成内存泄露的：[[NSNotificationCenter defaultCenter] removeObserver:self name:kNotificationOrientationChange object:nil]。