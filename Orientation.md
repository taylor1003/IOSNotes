控制器控制屏幕旋转
=======================
iOS屏幕旋转控制在View Controller里面，包含三种controller。
  其一：UIViewController及其子类。
  其二：UINavigationController及其子类。
  其三：UITabBarController及其子类。
  每一种controller及其子类都可以写屏幕旋转控制代码。但是记住一个原则，谁加载谁获得屏幕控制的权限，被加载的controller如果要添加自适应代码，可以在- (void)willRotateToInterfaceOrientation:(UIInterfaceOrientation)toInterfaceOrientation duration:(NSTimeInterval)duration函数中实现。