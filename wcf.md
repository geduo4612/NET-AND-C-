学习WCF的时候，往往被其配置文件搞得头晕脑涨，相关的API需要app.config或者web.confg(只能在这两个文件中)文件配合一起使用,初学者不明就里。<br>
可以在网上搜索到大量的初级入门教程。MSDN上面关于ServiceHost写的很简练，不同的API接口其对应的配置信息也不一样。比如：<br>
```C#
this.PS_SC = new ServiceHost(typeof(SCService)); this.PS_SC.Open();
```
那么需要的配置文件如下：<br>
>
      <service name="Hebcz.Framework.ServiceImplement.SCService" behaviorConfiguration="serviceBehaviorHebcz">       
        <host>
          <baseAddresses>
            <add baseAddress="net.tcp://localhost:6666/Service/SC"/>
          </baseAddresses>
        </host>        
        <endpoint contract="Hebcz.Framework.Service.ISCService" binding="netTcpBinding" bindingConfiguration="bindingHebcz"/>
      </service>
 
 标签`<host>`是不能省略的，那如果写成:<br>
 ```C#
  ServiceHost S_SH = new ServiceHost(type, new Uri(BaseAddresses));
 ```
 那么需要的配置文件如下：<br>
 >
      <service name="Hebcz.Framework.ServiceImplement.PlatformService.CAService" behaviorConfiguration="serviceBehaviorHebcz">      
        <endpoint contract="Hebcz.Framework.Service.PlatformService.ICAService" binding="netTcpBinding" bindingConfiguration="bindingHebcz"/>
      </service>
      
要记住配置文件和编码是等价的，需要做的是要了解ServiceHost内部实现机制。<br>
[自定义配置 扩展ServiceHost](https://blog.csdn.net/vivitue/article/details/9321943)。

 
  
