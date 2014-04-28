UIView旋转
=========

### 一、获取旋转角度

	最近做一个视图的旋转、放大、拖动、拉伸，其他的都慢慢解决了，就是旋转之后各种问题不好处理。
	最终归结到旋转角度的不能获取，纠结了好几天，终于找到了获取旋转角度的方法。
	CGFloat radius = atan2f(view.transform.b, view.transform.a);
	CGFloat degree = radius * (180 / M_PI);

### 二、旋转方式

	1、旋转到x度
	view.transform = CGAffineTransformMakeRotation(x);
	2、在现在旋转角度的基础上再旋转x度
	CGAffineTransform currentTransform = view.transform;
	CGAffineTransform newTransform = CGAffineTransformRotate(currentTransform, x); // 在现在的基础上旋转指定角度
	view.transform = newTransform;
	
### 三、恢复到0度
	CGAffineTransform currentTransform = view.transform;
	CGFloat rotation = (0.0 - recordDegree) * M_PI / 180.0f; // recordDegree记录现在的旋转角度
	CGAffineTransform newTransform = CGAffineTransformRotate(currentTransform, rotation);
	view.transform = newTransform;
	当然，根据上面旋转方式1的方法，直接将x设置为0更简单，这里只是提供一种思路。

