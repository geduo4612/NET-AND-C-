  在项目中看到一个服务入口类大量使用static 方法，[微软说static](https://docs.microsoft.com/zh-cn/dotnet/csharp/language-reference/keywords/static)，故此特意搜索下static到底有哪些优缺点。<br>
  
[Static vs. Non-Static method in C#](https://theburningmonk.com/2010/07/static-vs-non-static-method-in-csharp/)，讨论一些static特性。
主要是static的性能问题，文章中给出两种观点：<br>
>
  认为性能提升不大<br>
```
Please note that changing your methods to static methods is unlikely to help much on ambitious performance goals, 
but it can help a tiny bit and possibly lead to further reductions.
```
>
  认为性能提升可观<br>
```
After you mark the methods as static, the compiler will emit nonvirtual call sites to these members. 
Emitting nonvirtual call sites will prevent a check at runtime for each call that makes sure that the current object pointer is non-null. 
This can achieve a measurable performance gain for performance-sensitive code.
```
  我更倾向于前者。
  更多的见讨论参看这篇文章末尾的reference。
