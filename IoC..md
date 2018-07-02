.NET框架下的IoC
===============
1.微软自带的[Unity](http://www.cnblogs.com/think8848/archive/2008/10/25/1319616.html)<br>
　理解Unity[配置文件](http://www.cnblogs.com/doriandeng/archive/2008/02/23/1078386.html)。<br>
　[简单的使用说明](https://jingyan.baidu.com/article/c74d6000840b260f6b595d78.html)，主要学习`Unity`的配置文件是怎么简化繁杂的编码的，使用单例模式来解决到处`new UnityContainer()`的烦恼。关于`Unity`的配置文件中需要理解的是`typeAlias`，为什么会有这个标签呢？原因在于IoC底层原理是依靠反射来完成对象创建的，反射的时候必须知道“具体的类名，命名空间，所在的程序集”等必要信息，这些信息的路径长度可能有时候比较长，所以直接配置起来不太方便，这就产生了`typeAlias`标签，想想C#可以使用`using`关键字起别名也就理解了。另外，在`<type>`标签中的`type`类型指明Interface，而`<mapto>`标签则指明具体实现Interface的实体类（**`反射生成的就是这个类`**）。<br>

2.Autofac
  [Autofac4.0中文说明](http://autofaccn.readthedocs.io/zh/latest/getting-started/index.html#id2)<br>

3.性能对比
  [各大主流.Net的IOC框架性能测试比较](http://www.cnblogs.com/liping13599168/archive/2011/07/17/2108734.html)<br>

HebCZ项目分析
=========================
  HebCZ项目用到的是Unity，并且利用Hessian协议。那么两者是如何结合在一起的呢？<br>
  从前面分析可知在配置文件中需要mapto到具体的实现类中，在平台的HessianServiceProvider类实现了IServiceProvider接口，构造器如下：<br>
  >
  ```C#
    PUBLIC HessianServiceProvider(string url)
    {
      _url = url;
      Service = (T)new CHessianProxyFactory().Create(typeof(T), _url);
    }
  ```
  
  配置文件中是这样配置的：<br>
  >
  ```XML
   <typeAliases>
     <typeAlias alias="IServiceProvider" type="Hebcz.ASP.Framework.Core.IServiceProvider`1,Hebcz.ASP.Framework.Core"/> 
     <typeAlias alias="ServiceProvider" type="Hebcz.ASP.Framework.Core.HessianServiceProvider`1,Hebcz.ASP.Framework.Core"/> 
   </typeAliases>   
    <Container>
      <!--注意 type、mapTo引用了别名 -->
      <register type = "IServiceProvider" mapTo="ServiceProvider" name="EPscService">
        <constructor>
          <param name="url" value="http://ifis.hecz.cn:7200/HebYSZXEP/retmoting/scService"/>
        </constructor>
    </Container>
  ```
  那么<br>
  >
 
  ```C#
    var serviceProvider = IoC.Resolve<IServiceProvider<IEPscService>>(Resources.EPscService);<br> ----1
    T Service = serviceProvider.Service;<br> ----2
 ```
  其中，1语句执行过程中实际上IoC组件会读取配置文件，找到HessianServiceProvider，看到是`Constructor`标签并且标签里面含有`param`属性，根据配置直接调用找到HessianServiceProvider类的构造函数，并把url传进去，如上述中构造函数`Service = (T)new CHessianProxyFactory().Create(typeof(T), _url);`直接生成对应的远程服务代理，2语句则是返回该代理。<br>
  就这样，Unity与Hessian协议结合起来了。
 
  
  
  
  

