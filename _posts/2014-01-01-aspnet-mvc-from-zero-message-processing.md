---
layout: post
title: "ASP.NET MVC 从零开始 - 请求处理"
date: 2014-01-01 02:24:52 +0800
comments: true
categories: aspnet httphandler httpmodule
---

这是这个系列的第三篇了。前两篇文章请参见：

* [ASP.NET MVC 从零开始 - Create and Run](http://lxconan.github.io/2013/12/27/aspnet-mvc-from-zero-create-and-run)
* [ASP.NET MVC 从零开始 - Web.config](http://lxconan.github.io/2013/12/29/aspnet-mvc-from-zero-webconfig/)

这一篇仍然是原理上的东西，不涉及代码。我保证从下一篇开始，我们就开始写代码了。所以还请忍耐。

## ASP.NET 的请求处理流程

在这篇文章中，我们讨论的问题是 ASP.NET 对 HTTP 请求的处理流程。但是这个流程并不止限于 ASP.NET。许许多多的 Web Server 都是采用了相似的处理流程。这个流程就像是张三在上课的时候给他的好朋友（或者死对头）（我们叫他：赵六）传纸条。首先我假定张三很尊重老师的，不想伤害他脆弱的心灵，于是他无法把纸条直接传到赵六的手中，他必须先把纸条交给离他最近的同学手中，然后依次传递。我们不妨假定这个过程必须经过这几个人：

<!--more-->

> 张三 --> 李四 --> 王五 --> 赵六

那么不难想到，纸条回来的路径也需要依次反向的经过相同的人。也就是

> 赵六 --> 王五 --> 李四 --> 张三

那么传纸条开始，首先张三写了一张纸条，其内容是：（少儿不宜，故隐去）。纸条送到了李四的手中，李四是一个有理想有道德的青年，于是拒绝继续传递下去。张三无奈，只得又写了一张新的。其内容是：“下课后是否去一起吃饭？”。纸条送到了李四的手中，李四看后把纸条递给了王五；王五看后将纸条递给了赵六。赵六看后，欣然写到：“好滴”，又把纸条交到了王五手中；王五接到纸条不觉有些心动，于是在纸条上也添了一笔：“我也去！”，就把纸条交给了李四；李四直接将纸条传回张三手中，于是张三看到了如下的内容：“好滴。我也去（王五）”。

我们不妨将张三的行为视为张三向赵六发出的请求。而在请求传递的过程中经过了李四和王五。整个过程一共涉及到了两种角色：第一种角色是赵六，他是请求到达的终结点，也是创建响应的人；而第二种角色就是李四和王五，他们在整个过程中都可以看到请求和响应的内容，并可以对请求或者响应进行过滤或者更改。

在 ASP.NET 中，这两种角色分别称之为：`Http Handler` 以及 `Http Module`。`Http Handler` 作为 Request 处理的终结点存在，并负责处理请求以及生成响应。几乎所有的高级 Web 特性，例如页面的生成、MVC 都是 `Http Handler`。而相应的 `Http Module` 作为请求传递的中间环节，主要对 Request/Response 进行过滤和修饰。常见的任务例如身份验证，Session 管理都是使用 `Http Module` 进行处理的。

不难理解，每一个 Request 最多只可能有一个 `Http Handler` 对其进行处理；而每一个 Request 可能会有多个 `Http Module` 对其进行过滤或者修饰。 

在 ASP.NET 中，`Http Handler` 实现了 `IHttpHandler` 接口而 `Http Module` 实现了 `IHttpModule` 接口。

## 观察 Http Handler 和 Http Module

如何观察我们的 Web App 搭建了怎样的处理流程呢？

### 在 IIS 中观察

第一种方式是在 IIS 中观察，这种方式非常简单实用。首先打开 IIS 管理器，选择需要观察的 Web App 或者某一个虚拟目录。之后，在右侧的“处理程序映射”中可以看到所有启用了的 `Http Handler`。例如，我的 Web App 启用了如下的 `Http Handler`：

* 对于 `\*` 类型的请求，使用 `System.Web.Handlers.TransferRequestHandler` 进行处理（这个 `Http handler` 也是 ASP.NET MVC 的基础）；
* 对于 `*.aspx` 类型的请求，使用 `System.Web.UI.PageHandlerFactory` 进行处理（这个 `Http handler` 是 WebForm 的基础）；
* 对于 `*.cshtml` 类型的请求，使用 `System.Web.HttpForbiddenHandler` 进行处理（因为客户端是不允许访问服务端代码的）；
* ……

至于 `Http Module`，可以在 “模块” 中进行观察。在“分组依据”中选择“托管模块”就可以观察到 ASP.NET 注册的 `Http Module` 了。

### 在 Powershell 中观察

第二种方式是在 Powershell 中使用 IIS Extension 进行观察。这个非常符合我们的胃口，也为 Automation 提供了可能。在观察之前，请确认你已经安装了 Powershell 的 WebAdministration 模块。然后以管理员身份启动 Powershell，在 Powershell 中进行如下输入：

```
PS C:\> IIS:
PS IIS:\> cd Sites
PS IIS:\Sites> cd Default # Suppose we have a website named 'Default'
PS IIS:\Sites\Default> 
```

此时我们就进入了 Default Website 的虚拟路径。我们可以使用 `Get-WebHandler` 查看当前虚拟路径下的 `Http handler` 设置。

```
PS IIS:\Sites\Default> Get-WebHandler
Name                          Path       Verb                          Modules
----                          ----       ----                          -------
WebDAV                        *          PROPFIND,PROPPATCH,MKCOL,P... WebDAVModule
ISAPI-dll                     *.dll      *                             IsapiModule
AXD-ISAPI-4.0_64bit           *.axd      GET,HEAD,POST,DEBUG           IsapiModule
PageHandlerFactory-ISAPI-4... *.aspx     GET,HEAD,POST,DEBUG           IsapiModule
SimpleHandlerFactory-ISAPI... *.ashx     GET,HEAD,POST,DEBUG           IsapiModule
WebServiceHandlerFactory-I... *.asmx     GET,HEAD,POST,DEBUG           IsapiModule
HttpRemotingHandlerFactory... *.rem      GET,HEAD,POST,DEBUG           IsapiModule
HttpRemotingHandlerFactory... *.soap     GET,HEAD,POST,DEBUG           IsapiModule
... ...
```

为了查看 ASP.NET 注册的 `Http Module` 我们可以使用如下的命令：`Get-WebManagedModule`

```
PS IIS:\Sites\Default> Get-WebManagedModule
Name                      Precondition                       Type
----                      ------------                       ----
UrlRoutingModule-4.0     managedHandler,runtimeVersionv4.0  System.Web.Routing.Url...
ScriptModule-4.0         managedHandler,runtimeVersionv4.0  System.Web.Handlers.Sc...
OutputCache              managedHandler                     System.Web.Caching.Out...
Session                  managedHandler                     System.Web.SessionStat...
WindowsAuthentication    managedHandler                     System.Web.Security.Wi...
FormsAuthentication      managedHandler                     System.Web.Security.Fo...
DefaultAuthentication    managedHandler                     System.Web.Security.De...
RoleManager              managedHandler                     System.Web.Security.Ro...
UrlAuthorization         managedHandler                     System.Web.Security.Ur...
FileAuthorization        managedHandler                     System.Web.Security.Fi...
AnonymousIdentification  managedHandler                     System.Web.Security.An...
Profile                  managedHandler                     System.Web.Profile.Pro...
UrlMappingsModule        managedHandler                     System.Web.UrlMappings...
```

### 在 web.config 文件中观察

在[上一篇](http://lxconan.github.io/2013/12/29/aspnet-mvc-from-zero-webconfig)中，我们介绍了 web.config 文件是有一种继承体系的。那么我们可以在这个体系涉及的各个文件中查找 `Http handler` 以及 `Http Module` 的配置。

`Http handler` 的配置在 `.config` 文件的 `<system.webServer>` 的 `<handlers>` 元素下。例如，我们可以在 IIS 的 applicationHost.config 文件中发现如下的 `HTTP handler` 的映射信息：

```xml
<system.webServer>
    <handlers accessPolicy="Read, Script">
        <add 
            name="ExtensionlessUrlHandler-Integrated-4.0" 
            path="*." 
            verb="GET,HEAD,POST,DEBUG" 
            type="System.Web.Handlers.TransferRequestHandler" 
            preCondition="integratedMode,runtimeVersionv4.0" responseBufferLimit="0" />
        ... ...
    </handlers>
</system.webServer>
```

而 `Http Module` 是配置在 `<system.webServer>` 的 `<modules>` 下。例如，我们可以在 IIS 的 applicationHost.config 文件中发现如下的 `Http Module` 信息：

```xml
<system.webServer>
    <modules>
        ... ...
        <add 
            name="OutputCache" 
            type="System.Web.Caching.OutputCacheModule" 
            preCondition="managedHandler" />
        ... ...
    </modules>
</system.webServer>
```

这一篇的介绍就到这里，从下一篇开始我们将开始尽可能的构造自动化部署脚本。