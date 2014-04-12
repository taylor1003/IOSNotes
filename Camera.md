使用AVCaptureSession自定义拍照界面
===============================

主要类及其初始化
-------------
	AVCaptureSession           session
	AVCaptureVideoPreviewLayer 预览layer
	AVCaptureDeviceInput       输入
	AVCaptureStillImageOutput  输出图片
	AVCaptureMovieFileOutput to output to a movie file (输出一个 视频文件) 
	AVCaptureVideoDataOutput if you want to process frames from the video being captured (可以采集数据从指定的视频中) 
	AVCaptureAudioDataOutput if you want to process the audio data being captured (采集音频) 
	AVCaptureStillImageOutput if you want to capture still images with accompanying metadata (采集静态图片) 
	
AVCaptureDevice.h中几个属性的宏定义

	// 前置和后置摄像头
	typedef NS_ENUM(NSInteger, AVCaptureDevicePosition) {
		AVCaptureDevicePositionUnspecified         = 0,
		AVCaptureDevicePositionBack                = 1,
		AVCaptureDevicePositionFront               = 2
	} NS_AVAILABLE(10_7, 4_0);
	
	// 闪光灯开关
	typedef NS_ENUM(NSInteger, AVCaptureFlashMode) {
		AVCaptureFlashModeOff  = 0,
		AVCaptureFlashModeOn   = 1,
		AVCaptureFlashModeAuto = 2
	} NS_AVAILABLE(10_7, 4_0);
	
	// 手电筒开关
	typedef NS_ENUM(NSInteger, AVCaptureTorchMode) {
		AVCaptureTorchModeOff  = 0,
		AVCaptureTorchModeOn   = 1,
		AVCaptureTorchModeAuto = 2,
	} NS_AVAILABLE(10_7, 4_0);
	
	// 焦距调整
	typedef NS_ENUM(NSInteger, AVCaptureFocusMode) {
		AVCaptureFocusModeLocked              = 0,
		AVCaptureFocusModeAutoFocus           = 1,
		AVCaptureFocusModeContinuousAutoFocus = 2,
	} NS_AVAILABLE(10_7, 4_0);
	
	// 自动对焦焦距限制条件
	typedef NS_ENUM(NSInteger, AVCaptureAutoFocusRangeRestriction) {
		AVCaptureAutoFocusRangeRestrictionNone = 0,
		AVCaptureAutoFocusRangeRestrictionNear = 1,
		AVCaptureAutoFocusRangeRestrictionFar  = 2,
	} NS_AVAILABLE_IOS(7_0);
	
	// 曝光量调节
	typedef NS_ENUM(NSInteger, AVCaptureExposureMode) {
		AVCaptureExposureModeLocked					= 0,
		AVCaptureExposureModeAutoExpose				= 1,
		AVCaptureExposureModeContinuousAutoExposure	= 2,
	} NS_AVAILABLE(10_7, 4_0);
	
	// 白平衡
	typedef NS_ENUM(NSInteger, AVCaptureWhiteBalanceMode) {
		AVCaptureWhiteBalanceModeLocked				        = 0,
		AVCaptureWhiteBalanceModeAutoWhiteBalance	        = 1,
		AVCaptureWhiteBalanceModeContinuousAutoWhiteBalance = 2,
	} NS_AVAILABLE(10_7, 4_0);
	
	// 应用程序对摄像头的访问权限，未选择、没有被授权、明确拒绝、授权
	typedef NS_ENUM(NSInteger, AVAuthorizationStatus) {
		AVAuthorizationStatusNotDetermined = 0,
		AVAuthorizationStatusRestricted,
		AVAuthorizationStatusDenied,
		AVAuthorizationStatusAuthorized
	} NS_AVAILABLE_IOS(7_0);

### AVcaptureSession初始化

session用于控制预览、输入、输出

session用于初始化实时预览页面的AVCapturePreviewLayer

session用于添加输入设备，即前、后摄像头，AVCaptureDeviceInput需要由AVCaptureDevice初始化

之后用session的canAddInput判断是否可以输入，如果可以，调用session的addInput函数添加输入设备

session用AVCaptureStillImageOutput的实例添加输入设备，输入设备需要用NSDictionary的实例配置输出类型

接着用session的canAddOutput函数判断是否可以添加输出，如果可以，调用session的addOutput添加输出设备

所有配置完成之后记得用[session startRunning]启动session。

	- (void)addSession
	{
		AVCaptureSession *session = [[[AVCaptureSession alloc] init] autorelease];
		self.session = session;
		/*
		 * 设置质量
		 * AVCaptureSessionPresetHigh High  Highest recording quality. This varies per device.
		 * AVCaptureSessionPresetMedium Medium  Suitable for WiFi sharing. The actual values may change.
		 * AVCaptureSessionPresetLow Low  Suitable for 3G sharing. The actual values may change.
		 * AVCaptureSessionPreset640x480 640x480  VGA.
		 * AVCaptureSessionPreset1280x720 1280x720  720p HD.
		 * AVCaptureSessionPresetPhoto Photo  Full photo resolution. This is not supported for video output.
		 */
		_session.sessionPreset = AVCaptureSessionPresetPhoto;
	}

### AVCaptureVideoPreviewLayer 用session初始化

	- (void)addVideoPreviewLayerWithRect:(CGRect)previewRect
	{
		AVCaptureVideoPreviewLayer *preview = [[AVCaptureVideoPreviewLayer alloc] initWithSession:_session];
		preview.videoGravity = AVLayerVideoGravityResizeAspectFill;
		preview.frame = previewRect;
		self.previewLayer = preview;
	}
	
### AVCaptureDeviceInput 添加输入设备

	- (void)addVideoInputFrontCamera:(BOOL)front
	{
		NSArray *devices = [AVCaptureDevice devices];
		AVCaptureDevice *frontCamera;
		AVCaptureDevice *backCamera;
	
		for (AVCaptureDevice *device in devices) {
			// 判断是否有摄像头
			if ([device hasMediaType:AVMediaTypeVideo]) {
			
				if ([device position] == AVCaptureDevicePositionBack) {
					DLog(@"Device position: back");
					backCamera = device;
				} else {
					DLog(@"Device position: front");
					frontCamera = device;
				}
			}
		}
	
		NSError *error = nil;
	
		if (front) {
			// AVCaptureDeviceInput继承AVCaptureInput
			AVCaptureDeviceInput *frontFacingCameraDeviceInput = [AVCaptureDeviceInput deviceInputWithDevice:frontCamera error:&error];
			if (!error) {
				if ([_session canAddInput:frontFacingCameraDeviceInput]) {
					[_session addInput:frontFacingCameraDeviceInput];
					self.inputDevice = frontFacingCameraDeviceInput;
				} else {
					DLog(@"Couldn't add front facing video input");
				}
			}
		} else {
			AVCaptureDeviceInput *backFacingCameraDeviceInput = [AVCaptureDeviceInput deviceInputWithDevice:backCamera error:&error];
			if (!error) {
				if ([_session canAddInput:backFacingCameraDeviceInput]) {
					[_session addInput:backFacingCameraDeviceInput];
					self.inputDevice = backFacingCameraDeviceInput;
				} else {
					DLog(@"Couldn't add back facing video input");
				}
			}
		}
	}
	
###	AVCaptureStillImageOutput 添加输出设备

	- (void)addStillImageOutput
	{
	
		AVCaptureStillImageOutput *tmpOutput = [[AVCaptureStillImageOutput alloc] init];
		NSDictionary *outputSettings = [[NSDictionary alloc] initWithObjectsAndKeys:AVVideoCodecJPEG, AVVideoCodecKey, nil]; // 输出jpeg
		tmpOutput.outputSettings = outputSettings;
	
		if ([_session canAddOutput:tmpOutput]) {
			[_session addOutput:tmpOutput];
			self.stillImageOutput = tmpOutput;
		}
	}
	
摄像头设置相关
------------
### 切换前后摄像头

	- (void)switchCamera:(BOOL)isFrontCamera
	{
		if (!_inputDevice) {
			return;
		}
		[_session beginConfiguration];
	
		[_session removeInput:_inputDevice];
		[self addVideoInputFrontCamera:isFrontCamera];
	
		[_session commitConfiguration];
	}
	
### 拍照

控制摄像头拍到的图片首先需要获取输出设备对应的连接

	- (AVCaptureConnection *)findVideoConnection
	{
		AVCaptureConnection *videoConnection = nil;
		for (AVCaptureConnection *connection in _stillImageOutput.connections) {
			for (AVCaptureInputPort *port in connection.inputPorts) {
				if ([[port mediaType] isEqual:AVMediaTypeVideo]) {
					videoConnection = connection;
					break;
				}
			}
			if (videoConnection) { break; }
		}
		return videoConnection;
	}

拍照功能实现

	- (void)takePicture:(DidCapturePhotoBlock)block
	{
		// 从输出AVCaptureStillImageOutput中获取AVMediaTypeVideo的连接
		AVCaptureConnection *videoConnection = [self findVideoConnection];
	
		[videoConnection setVideoScaleAndCropFactor:_scaleNum];
	
		DLog(@"about to request a capture from: %@", _stillImageOutput);
	
		[_stillImageOutput captureStillImageAsynchronouslyFromConnection:videoConnection completionHandler:^(CMSampleBufferRef imageDataSampleBuffer, NSError *error) {
		
			CFDictionaryRef exifAttachments = CMGetAttachment(imageDataSampleBuffer, kCGImagePropertyExifDictionary, NULL);
			if (exifAttachments) {
				DLog(@"attachments: %@", exifAttachments);
			} else {
				DLog(@"no attachment");
			}
			NSData *imageData = [AVCaptureStillImageOutput jpegStillImageNSDataRepresentation:imageDataSampleBuffer];
			UIImage *image = [[UIImage alloc] initWithData:imageData];
			DLog(@"originImage: %@", [NSValue valueWithCGSize:image.size]);
		
			CGFloat squareLength = TP_APP_SIZE.width;
			CGFloat headHeight = _previewLayer.bounds.size.height - squareLength;  // _previewLayer的frame是(0, 44, 320, 320 + 44)
			CGSize size = CGSizeMake(squareLength * 2, squareLength * 2); // double of screen
		
			UIImage *scaledImage = [image resizedImageWithContentMode:UIViewContentModeScaleAspectFill bounds:size interpolationQuality:kCGInterpolationHigh];
			DLog(@"scaledImage:%@", [NSValue valueWithCGSize:scaledImage.size]);
			// 重置image，将隐藏部分显示出来
			CGRect cropFrame = CGRectMake((scaledImage.size.width - size.width) / 2, (scaledImage.size.height - size.height) / 2 + headHeight, size.width, size.height);
			DLog(@"cropFrame: %@", [NSValue valueWithCGRect:cropFrame]);
			UIImage *croppedImage = [scaledImage croppedImage:cropFrame];
			DLog(@"coroppedImage: %@", [NSValue valueWithCGSize:croppedImage.size]);
		
			UIDeviceOrientation orientation = [UIDevice currentDevice].orientation;
			if (orientation != UIDeviceOrientationPortrait) {
			
				CGFloat degree = 0;
				if (orientation == UIDeviceOrientationPortraitUpsideDown) {
					degree = 180; // M_PI
				} else if (UIDeviceOrientationLandscapeLeft) {
					degree = -90; // -M_PI_2
				} else if (orientation == UIDeviceOrientationLandscapeRight) {
					degree = 90; // M_PI_2
				}
				croppedImage = [croppedImage rotatedByDegrees:degree];
			}
		
			// block、delegate、notification 3选1、传值
			if (block) {
				block(croppedImage);
			} else if ([_delegate respondsToSelector:@selector(didCapturePhoto:)]) {
				[_delegate didCapturePhoto:croppedImage];
			} else {
				[[NSNotificationCenter defaultCenter] postNotificationName:kCapturedPhotoSuccessfully object:croppedImage];
			}
		}];
	}
	
### 拉近拉远摄像头

	- (void)pinchCameraViewWithScaleNum:(CGFloat)scale
	{
		_scaleNum = scale;
		if (_scaleNum < MIN_PINCH_SCALE_NUM) {
			_scaleNum = MIN_PINCH_SCALE_NUM;
		} else if (_scaleNum > MAX_PINCH_SCALE_NUM) {
			_scaleNum = MAX_PINCH_SCALE_NUM;
		}
		[self doPinch];
	}

	- (void)pinchCameraView:(UIPinchGestureRecognizer *)gesture
	{
		BOOL allTouchesAreOnThePreviewLayer = YES;
		NSUInteger numTouches = [gesture numberOfTouches];
		for (NSInteger i = 0; i < numTouches; ++i) {
			CGPoint location = [gesture locationOfTouch:i inView:_preview];
			CGPoint convertedLocation = [_previewLayer convertPoint:location fromLayer:_previewLayer.superlayer];
			if (![_previewLayer containsPoint:convertedLocation]) {
				allTouchesAreOnThePreviewLayer = NO;
				break;
			}
		}
	
		if (allTouchesAreOnThePreviewLayer) {
			_scaleNum = _preScaleNum * gesture.scale;
		
			if (_scaleNum < MIN_PINCH_SCALE_NUM) {
				_scaleNum = MIN_PINCH_SCALE_NUM;
			} else if (_scaleNum > MAX_PINCH_SCALE_NUM) {
				_scaleNum = MAX_PINCH_SCALE_NUM;
			}
		
			[self doPinch];
		}
	
		if ([gesture state] == UIGestureRecognizerStateEnded || [gesture state] == UIGestureRecognizerStateCancelled || [gesture state] == UIGestureRecognizerStateFailed) {
			_preScaleNum = _scaleNum;
			DLog(@"final scale: %f", _scaleNum);
		}
	}

	- (void)doPinch
	{
		AVCaptureConnection *videoConnection = [self findVideoConnection];
	
		CGFloat maxScale = videoConnection.videoMaxScaleAndCropFactor; // videoScaleAndCropFactor这个属性取值范围是1.0-videoMaxScaleAndCropFactor. iOS5+才可以用
		if (_scaleNum > maxScale) {
			_scaleNum = maxScale;
		}
	
		[CATransaction begin];
		[CATransaction setAnimationDuration:.025];
		[_previewLayer setAffineTransform:CGAffineTransformMakeScale(_scaleNum, _scaleNum)];
		[CATransaction commit];
	}

### 闪光灯切换

	- (void)switchFlashMode:(UIButton *)sender
	{
		Class captureDeviceClass = NSClassFromString(@"AVCaptureDevice");
		if (!captureDeviceClass) {
			UIAlertView *alert = [[[UIAlertView alloc] initWithTitle:@"提示信息" message:@"您的设备没有拍照功能" delegate:nil cancelButtonTitle:NSLocalizedString(@"Sure", nil) otherButtonTitles:nil] autorelease];
			[alert show];
			return;
		}
	
		NSString *imgStr = @"";
		AVCaptureDevice *device = [AVCaptureDevice defaultDeviceWithMediaType:AVMediaTypeVideo];
		[device lockForConfiguration:nil];
		if ([device hasFlash]) {
			if (device.flashMode == AVCaptureFlashModeOff) {
				device.flashMode = AVCaptureFlashModeOn;
				imgStr = @"flashing_on.png";
			} else if (device.flashMode == AVCaptureFlashModeOn) {
				device.flashMode = AVCaptureFlashModeAuto;
				imgStr = @"flashing_auto.png";
			} else if (device.flashMode == AVCaptureFlashModeAuto) {
				device.flashMode = AVCaptureFlashModeOff;
			}
		
			if (sender) {
				[sender setImage:[UIImage imageNamed:imgStr] forState:UIControlStateNormal];
			}
		} else {
			UIAlertView *alert = [[UIAlertView alloc] initWithTitle:@"提示信息" message:@"您的设备没有闪光灯功能" delegate:nil cancelButtonTitle:@"OK" otherButtonTitles:nil];
			[alert show];
		}
		[device unlockForConfiguration];
	}
	
	