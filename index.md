## iOS 对象摸鱼 - alloc流程分析
### 1. alloc 做了啥
在业务层开发时，很少会对Objc源码进行探究。所以对系统方法的底层实现，往往不得而知，望而却步。

下面罗列查找源码的3个方法：

    1.  通过符号断点一步步执行
    2.  ctrl + step into
    3.  运行时汇编代码，开启汇编调试。具体开启方法：`（Xcode->Debug->Debug workflow->Always Show Disassembly）`

###### 举🌰（撸代码）：

在测试项目新建一个继承`NSObject`的类，并`alloc`出一个实例对象：

`FLObject *object = [FLObject alloc];`

打好断点后运行，并开启汇编调试，查看运行时汇编代码

![symbol](symbol.png)

如图，我们可以看到下一条指令的汇编注释`symbol stub for: objc_alloc`得知立即调用存根符号为`objc_alloc`的函数，所以我们不妨增加一个`objc_alloc`的符号断点，接着运行可以看到

![libObjc](libObjc.png)

所以可以看到在调用 `FLObject *object = [FLObject alloc];`   时，底层调用为`libobjc.A.dylib`动态链接库中的`objc_alloc`方法。

通过[苹果开源库](超链接地址 "https://opensource.apple.com/")可以下载相关开源库。

*注：callq：属于x86的指令，即调用函数时的压栈出栈。图一中`callq`指令会使程序跳到`0x10934b30`的地址中执行。除此之外，该指令还会将当前函数的下一条指令入栈*

### 2.alloc&init 探索

总所周知：`alloc`的调用是为对象开辟内存，具体如何实现 在拿到的`Objc`源码中一探究竟。

###### 怎么玩？

查看`alloc` 实现: 内部调用 `_rootAlloc` , ->

再调用`callAlloc`，->

而`callAlloc`内部经过编辑器优化判断，再调用`_objc_rootAllocWithZone`。

```
NEVER_INLINE
id
_objc_rootAllocWithZone(Class cls, malloc_zone_t *zone __unused)
{
    // allocWithZone under __OBJC2__ ignores the zone parameter
    return _class_createInstanceFromZone(cls, 0, nil,
                                         OBJECT_CONSTRUCT_CALL_BADALLOC);
}

```
`_objc_rootAllocWithZone` 两个参数：类名及zone；->

此时会调用`_class_createInstanceFromZone`。 这里着重看下它的内部实现：

1. size 即需要开辟的内存空间
```
size = cls->instanceSize(extraBytes);
```


2. calloc 开辟内存的地方，调用后返回内存地址
```
obj = (id)calloc(1, size);
```

3. 将开辟的内存与当前类进行绑定
```
obj->initInstanceIsa(cls, hasCxxDtor);
```
4. 最后返回该`obj`

