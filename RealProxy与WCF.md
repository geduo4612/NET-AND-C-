#RealProxy
============
[使用RealProxy来实现AOP](http://www.cnblogs.com/hb_cattle/archive/2011/10/17/2215970.html)，这个例子比较直观易懂，而且IFIS项目中也是采用类似的手法，利用泛型监听所有通过WCF的调用方法。<br>
IFIS项目中实现方式：<br>
 ```C#
  public class WCF_ServiceProxy<T> : RealProxy
 ```
注意其中的泛型T，其Inovke方法实现如下：<br>
 >
   ```C#
        public override IMessage Invoke(IMessage msg)
        {
            string errMsg = this.ClassName + ".Invoke()在 调用服务方法 时出现错误！";
            try  {
                T channel = WCF_ChannelFactory.Create<T>(this.EndPointName, this.AS_Protocol, this.UrlFix, this.UserName, this.PWD, this.AS_IP, this.AS_Port, this.AS_PK).CreateChannel();
                IMethodCallMessage methodCall = (IMethodCallMessage)msg;
                object[] copiedArgs = Array.CreateInstance(typeof(object), methodCall.Args.Length) as object[];
                methodCall.Args.CopyTo(copiedArgs, 0);
                IMethodReturnMessage methodReturn = null;
                
                try  {
                    using (channel as IDisposable) {
                        if (string.IsNullOrEmpty(this.UserName) || string.IsNullOrEmpty(this.PWD)) {
                            object returnValue = methodCall.MethodBase.Invoke(channel, copiedArgs);
                            methodReturn = new ReturnMessage(returnValue, copiedArgs, copiedArgs.Length, methodCall.LogicalCallContext, methodCall);
                        }
                        else {//这里主要是在消息头中加入用户名和密码，每次调用都需要进行校验。
                            using (OperationContextScope contextScope = new OperationContextScope(channel as IContextChannel)) {
                                MessageHeader<string> userNameHeader = new MessageHeader<string>(this.UserName);
                                OperationContext.Current.OutgoingMessageHeaders.Add(userNameHeader.GetUntypedHeader("ClientUserCode", "http://www.hebcz.gov.cn"));

                                MessageHeader<string> userPwdHeader = new MessageHeader<string>(this.PWD);
                                OperationContext.Current.OutgoingMessageHeaders.Add(userPwdHeader.GetUntypedHeader("ClientUserPwd", "http://www.hebcz.gov.cn"));

                                MessageHeader<string> userXzqhHeader = new MessageHeader<string>(this.XzqhBm);
                                OperationContext.Current.OutgoingMessageHeaders.Add(userXzqhHeader.GetUntypedHeader("ClientXzqhBm", "http://www.hebcz.gov.cn"));

                                MessageHeader<string> userDbyearHeader = new MessageHeader<string>(this.DbYear);
                                OperationContext.Current.OutgoingMessageHeaders.Add(userDbyearHeader.GetUntypedHeader("ClientDbYear", "http://www.hebcz.gov.cn"));

                                object returnValue = methodCall.MethodBase.Invoke(channel, copiedArgs);
                                methodReturn = new ReturnMessage(returnValue, copiedArgs, copiedArgs.Length, methodCall.LogicalCallContext, methodCall);
                            }
                        }
                   }
                }
                catch (Exception ex)  {
                    if (ex.InnerException is CommunicationException || ex.InnerException is TimeoutException) {
                        (channel as ICommunicationObject).Abort();
                    }

                    if (ex.InnerException != null) {
                        methodReturn = new ReturnMessage(ex.InnerException, methodCall);
                    }
                    else {
                        methodReturn = new ReturnMessage(ex, methodCall);
                    }
                }

                return methodReturn;
            }
            catch (Exception ex) {
                ExceptionHelper.AddExceptionData(this, ex, "错误信息", errMsg);
                throw new ExceptionBase(errMsg, ex);
            }
        }
 ```       
        
那么Invoke方法是在什么时候被调用的呢，MSDN上说：<br>

>
When the transparent proxy that is backed by the RealProxy is called, it delegates the calls to the Invoke method. <br>
The Invoke method transforms the message in the msg parameter into a IMethodCallMessage, and sends it to the remote object that is represented by the current instance of RealProxy.

也就是说调用业务服务（透明代理的某个方法）的时候，CLR会把该请求转发给Invoke方法，该方法会把整个调用的Context截获，你可以在该方法中进行自己想要的处理：比如说加入日志或者像如上所示的那样加入用户名和密码。<br>
那么透明代理是怎么产生的呢？<br>
>
 ```C#
  WCF_ServiceProxy<T> serviceRealProxy = new WCF_ServiceProxy<T>(endPointName, this.AppCfg.AS_Protocol, urlFix, this.UserCode, this.UserPWD,this.XzqhBm,this.DbYear, this.AppCfg.AS_IP, this.AppCfg.AS_Port, this.AppCfg.AS_PK);
  return (T)(serviceRealProxy.GetTransparentProxy());
 ```
 可以看到通过GetTransparentProxy返回会透明代理对象，在通过类型转换成服务接口。那么透明代理的生成是否意味着服务端对应的提供服务的对象也生成了呢？从Invoke方法的实现来看：只有创立通道之后服务端真正提供服务的对象才会创立起来，通过`object returnValue = methodCall.MethodBase.Invoke(channel, copiedArgs)`进行调用，并把结果返回给透明代理，通过这种方式是的远程调用就像调用本地方法一样自然。<br>
 
 IFIS中涉及到的WCF是如何的呢？<br>
 ================================
 
 
