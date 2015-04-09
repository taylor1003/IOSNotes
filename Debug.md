1.Xcode不能organizer中真机显示为绿色，但不能调试，可能是甚至中目标系统版本号大于真机版本。解决方法，将Target->General->Deployment Target的版本号设置为小于等于真机版本。在相隔一段时间后再次遇到，记下来。

2.-[UIButton setDiyComObj:]: unrecognized selector sent to instance 0x100cc790

 *** Terminating app due to uncaught exception 'NSInvalidArgumentException', reason: '-[UIButton setDiyComObj:]: unrecognized selector sent to instance 0x100cc790'

 错误原因是我的倏忽，解决问题用了两个小时。原来使用UIButton，但因为前端的需求用原来后台的逻辑太勉强，我用后台的逻辑自动创建了一个ViewModal绑定到UIButton上，形成DIYButton。前面定义的名称是DIYButton，后面初始化的时候没注意更改，还是UIButton初始化，自然就没有diyComObj这个属性，导致赋值出错。

 其实，从第一句-[UIButton setDiyComObj:]是UIButton而不是DIYButton就已经可以找到出错原因，但一直没注意这个问题，导致找出BUG的效率降低。

 NOTE：注意细节！
 
 3.写文件缓存，遍历管理缓存的字典NSMutableDictionary，偶尔会出现一个奇怪的现象，在遍历过程中程序会crash，而且没有一点错误信息，百思不得其解。首先以为是key的问题，调试后发现问题不在这儿。纠结了很久才想到可能是我非常二的在遍历过程中移除了过期和不完整的缓存相应key的键值，导致管理遍历的内部函数出错。改为将key值记录下来，遍历之后再删除，问题得到解决。请原谅我，没遇到之前我真不知道在遍历过程中不能移除，以后再也不犯了。
