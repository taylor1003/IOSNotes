1.Xcode不能organizer中真机显示为绿色，但不能调试，可能是甚至中目标系统版本号大于真机版本。解决方法，将Target->General->Deployment Target的版本号设置为小于等于真机版本。在相隔一段时间后再次遇到，记下来。

2.-[UIButton setDiyComObj:]: unrecognized selector sent to instance 0x100cc790

 *** Terminating app due to uncaught exception 'NSInvalidArgumentException', reason: '-[UIButton setDiyComObj:]: unrecognized selector sent to instance 0x100cc790'

 错误原因是我的倏忽，解决问题用了两个小时。原来使用UIButton，但因为前端的逻需求用后台的逻辑太勉强，我用后台的逻辑自动创建了一个ViewModal绑定到UIButton上，形成DIYButton。前面定义的名称是DIYButton，后面初始化的时候没注意更改，还是UIButton初始化，自然就没有diyComObj这个属性，导致赋值出错。

 从第一句-[UIButton setDiyComObj:]是UIButton而不是DIYButton就已经可以找到出错原因，但一直没注意这个问题，导致找出BUG的效率降低。
 
 NOTE：注意细节！