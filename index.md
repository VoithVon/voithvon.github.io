## iOS 底层摸鱼 - alloc流程分析
### 1. alloc 做了啥
在业务层开发时，很少会对Objc源码进行探究。所以对一些系统方法底层具体执行了哪个库的哪写方法，就不得而知，往往止步于此。
下面总结查找源码的3个方法：
1.符号断点一步步执行
2.ctrl + step into
3.运行时汇编代码，开启汇编调试。具体开启方法：
`（Xcode->Debug->Debug workflow->Always Show Disassembly）`

👇进入正题（撸代码）：
在测试项目新建一个继承`NSObject`的类，并`alloc`出一个实例对象：
`FLObject *object = [FLObject alloc];`
打好断点后运行，并开启汇编调试，查看运行时汇编代码
![symbol](symbol.png)
如图，我们可以看到下一条指令的汇编注释`symbol stub for: objc_alloc`得知立即调用存根符号为`objc_alloc`的函数，所以我们不妨增加一个`objc_alloc`的符号断点，接着运行可以看到
![libObjc](libObjc.png)

所以可以看到在调用`FLObject *object = [FLObject alloc];`    时，其实调用的是`objc`库中的`objc_alloc`方法。

*注：callq：调用函数的意思，即调用函数时的压栈出栈*

### 2.alloc&init 探索
