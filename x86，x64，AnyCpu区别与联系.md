### 出来混迟早要还的
                倪坤《无间道2》
                  
对于技术债也是如此。

这两天做项目，需要对数据进行加密，采用第三方加密机，C#封装了对方的C++接口，其版本分别有32位和64位的dll。启动Asp.net，程序一走到封装的地方就报**正在加载格式不正确的dll**,
让人十分郁闷。开始折腾工程的`/platform`，由于基础不牢，分不清楚x86，x64，AnyCpu与OS本身的关系，问题迟迟不能定位，一会儿怀疑第三方加密的dll有问题，一会儿怀疑是不是调用的方法
写的不对。浪费了不少时间，问题还是没有得以解决。于此，下定决心搞明白到底问题出在哪里，`/platform`选项之间的关系是怎么样的。

### 关于.NET编译的目标平台(AnyCPU,x86,x64)区别

[这篇文章](http://blog.sina.com.cn/s/blog_78b94aa301014i8r.html)，大体上能够解惑我的一部分问题。但是我觉得它对于解决方案得描述并不清晰。

首先我觉得分清楚以下几个要素：

- 1 参与方有：OS，exe，dll（为什么会区分exe，dll是因为AnyCPU模式下有区别）。
- 2 然后是32位或者64位。
- 3 32位的OS上面如论如何只能跑32位的exe,exe则调用32位的dll；而64位的OS则可以执行64位exe，调用64位的dll(不能调用32位的dll）或者执行32位exe,调用32位dll(不能调用64位的dll)。
为什么是这样，参考[什么是SysWow64](https://blogs.msdn.microsoft.com/tianlin/2011/10/26/syswow64/)。

有了这个基础之后，再来看/platform的[设定](https://msdn.microsoft.com/zh-cn/library/zekwfyz4(VS.80).aspx)。注意文中的下面这句话：

> 用`/platform:anycpu` 编译的 DLL 将在与加载它的进程相同的 CLR 上执行。

“加载它的进程”也就是exe的进程，dll执行环境是受到exe的控制。那么AnyCpu到底指的是什么呢？为什么C++工程没有这个编译选项，而Net工程却有呢？

想想Net的IL语言吧，C#语言会生成中间语言IL，这对所有的编译模式是一样的，但JIT如何把IL编译成机器语言却是受到编译模式的控制。“The compilation mode is just an instruction for the JIT compiler for how it is allowed to compile the IL code into machine code. The IL code itself is the same for all modes.”
根据以上知识点，我们可以得出一张表：

|OS	|EXE	 |  DLL	 |执行环境CLR |
|:---|------:|-------:|:-----------:|
|32	|x86	 | x86	 |   32      |
|		|      | AnyCpu|           |
|	  |AnyCpu| x86	 |           |
|		|      | AnyCpu|           |
|64	|x86	 | x86	 |   32      |
|		|      | AnyCpu|           |
|	  |x64	 | x64	 |   64      |
|		|      | AnyCpu|	         |
|	  |AnyCpu| x64	 |           |
|		|      | AnyCpu|	         |


对照这张表就可以快速的配置/platform了，重新启动程序，测试，结果还是报错！！！静下心来，重新思考问题，应该说这张表肯定是没有问题的，而且整个工程都是按照表进行配置的，问题到底出现哪里呢？仔细观察，，
asp.net调试时页面是host在IIS Express上面的，那么这里的exe就应该是IIS Express启动的进程。参考[C#调用C++代码遇到的问题总结](https://www.cnblogs.com/neverstop/p/5901652.html)，文章中说到

>默认asp.net项目在调试时会运行在32位下的iisexpress进程中，如果你的项目是64位的，那么需要在VS中将iisexpress配置为**64位模式**。

### 至此，真相浮出水面。

PS:这篇[文章](https://www.owcer.com/2009/08/anycpu-x86-x64-whats-the-difference/)也值得一看。
