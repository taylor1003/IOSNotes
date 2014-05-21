iOS中atomic的实现
================

原子性与非原子性
--------------

iOS中有两个属性non-atomic和atomic，前者是非原子性的（线程不安全），后者是原子性的（线程安全），一般情况下不会去重写它们，但某些时候确实有重写的需求。

那些int、float之类的类型，你重写想出错都很难。但是强引用类型（retain）就需要注意了。

简单的说一下非原子性的nonatomic实现，方式如下：

	- (void)setCurrentImage:(UIImage *)currentImage
	{
        if (_currentImage != currentImage) {
            [_currentImage release];
            _currentImage = [currentImage retain];
            
            // do something
        }
	}

	- (UIImage *)currentImage
	{
	        return _currentImage;
	}

atomic实现：
---------

关于atomic的实现最开始的方式如下：

	- (void)setCurrentImage:(UIImage *)currentImage
	{
	    @synchronized(self) {
	        if (_currentImage != currentImage) {
	            [_currentImage release];
	            _currentImage = [currentImage retain];
	            
	            // do something
	        }
	    }
	}

	- (UIImage *)currentImage
	{
	    @synchronized(self) {
	        return _currentImage;
	    }
	}

具体讲就是retain的同步版本，本来以为没问题，但在用GCD重绘currentImage的过程中，有时候currentImage切换太频繁。在完成之前就把之前的currentImage释放了，程序仍然会崩溃。还需要在resize过程中增加retain和release操作，代码如下：

	- (UIImage *)resizedImage:(CGSize)newSize interpolationQuality:(CGInterpolationQuality)quality {
	    // For multithreading
	    [self retain];
	    
	    BOOL drawTransposed;
	    CGAffineTransform transform = CGAffineTransformIdentity;
	    
	    // In iOS 5 the image is already correctly rotated. See Eran Sandler's
	    // addition here: http://eran.sandler.co.il/2011/11/07/uiimage-in-ios-5-orientation-and-resize/
	    
	    if([[[UIDevice currentDevice]systemVersion]floatValue] >= 5.0) {
	        drawTransposed = NO;
	    }
	    else {
	        switch(self.imageOrientation) {
	            case UIImageOrientationLeft:
	            case UIImageOrientationLeftMirrored:
	            case UIImageOrientationRight:
	            case UIImageOrientationRightMirrored:
	                drawTransposed = YES;
	                break;
	            default:
	                drawTransposed = NO;
	        }
	        
	        transform = [self transformForOrientation:newSize];
	    }
	    transform = [self transformForOrientation:newSize];
	    
	    UIImage *image = [self resizedImage:newSize transform:transform drawTransposed:drawTransposed interpolationQuality:quality];
	    [self release];
	    return image;
	}

原始版本的resize函数如下：

	- (UIImage *)resizedImage:(CGSize)newSize interpolationQuality:(CGInterpolationQuality)quality {	    
	    BOOL drawTransposed;
	    CGAffineTransform transform = CGAffineTransformIdentity;
	    
	    // In iOS 5 the image is already correctly rotated. See Eran Sandler's
	    // addition here: http://eran.sandler.co.il/2011/11/07/uiimage-in-ios-5-orientation-and-resize/
	    
	    if([[[UIDevice currentDevice]systemVersion]floatValue] >= 5.0) {
	        drawTransposed = NO;
	    }
	    else {
	        switch(self.imageOrientation) {
	            case UIImageOrientationLeft:
	            case UIImageOrientationLeftMirrored:
	            case UIImageOrientationRight:
	            case UIImageOrientationRightMirrored:
	                drawTransposed = YES;
	                break;
	            default:
	                drawTransposed = NO;
	        }
	        
	        transform = [self transformForOrientation:newSize];
	    }
	    transform = [self transformForOrientation:newSize];
	    
	    return [self resizedImage:newSize transform:transform drawTransposed:drawTransposed interpolationQuality:quality];
	}

但是之前在没有重写getter之前，用atomic的getter程序不会崩溃。于是我就想现在的getter和atomic自己实现的getter肯定有区别。

最后，答案出现：在getter的return之前retain，再autorelease一次就可以了。getter函数就变成了这样：

	- (UIImage *)currentImage
	{
	    @synchronized(self) {
	        UIImage *image = [_currentImage retain];
	        return [image autorelease];
	    }
	}

这样可以确保currentImage在调用过程中不会因为currentImage被释放或者改变，使它的retainCount次数变为0，再在调用时让程序直接崩溃。

Runtime方法
----------

在[Memory and thread-safe custom property methods](http://www.cocoawithlove.com/2009/10/memory-and-thread-safe-custom-property.html)这篇文章中还提到了一种Objective-C的runtime解决方案。

Objective-C的runtime中实现了以下函数：

	id <strong>objc_getProperty</strong>(id self, SEL _cmd, ptrdiff_t offset, BOOL atomic);
	void <strong>objc_setProperty</strong>(id self, SEL _cmd, ptrdiff_t offset, id newValue, BOOL atomic,
	    BOOL shouldCopy);
	void <strong>objc_copyStruct</strong>(void *dest, const void *src, ptrdiff_t size, BOOL atomic,
	    BOOL hasStrong);

这几个函数被实现了，但没有被声名。如果要使用他们，必须自己声名。它们比用@synchronized实现的要快。因为它的实现方式与一般情况不同，静态变量只在接收并发时才会锁住。

声名方式：

	#define <strong>AtomicRetainedSetToFrom</strong>(dest, source) \
	    objc_setProperty(self, _cmd, (ptrdiff_t)(&dest) - (ptrdiff_t)(self), source, YES, NO)
	#define <strong>AtomicCopiedSetToFrom</strong>(dest, source) \
	    objc_setProperty(self, _cmd, (ptrdiff_t)(&dest) - (ptrdiff_t)(self), source, YES, YES)
	#define <strong>AtomicAutoreleasedGet</strong>(source) \
	    objc_getProperty(self, _cmd, (ptrdiff_t)(&source) - (ptrdiff_t)(self), YES)
	#define <strong>AtomicStructToFrom</strong>(dest, source) \
	    objc_copyStruct(&dest, &source, sizeof(__typeof__(source)), YES, NO)

用这些宏定义，上面something的copy getter和setter方法将变成这样：

	- (NSString *)someString
	{
	    return AtomicAutoreleasedGet(someString);
	}
	- (void)setSomeString:(NSString *)aString
	{
	    AtomicCopiedSetToFrom(someString, aString);
	}

someRect存取方法将变成这样：

	- (NSRect)someRect
	{
		NSRect result;
		AtomicStructToFrom(result, someRect);
		return result;
	}
	- (void)setSomeRect:(NSRect)aRect
	{
		AtomicStructToFrom(someRect, aRect);
	}

