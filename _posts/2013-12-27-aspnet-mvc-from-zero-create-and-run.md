---
layout: post
title: "ASP.NET MVC 从零开始 - create and run"
date: 2013-12-27 01:38:51 +0800
comments: true
categories: asp.net mvc
---

如果你想用 ASP.NET MVC 创建一个网络应用，那么你可以搜到很多的文章。但是没有多少文章告诉你，如何从零开始创建一个 ASP.NET MVC 应用程序。这对我们是非常有害的，我们希望了解每一个细节，搞清 Components 之间的联系。然后，再放心的使用向导创建 ASP.NET MVC 应用程序。需要表明一点的是，我说的从零开始，是从零开始一步一步创建 App，我希望你应该熟悉 C#，并至少对 ASP.NET MVC 多多少少有些了解。

<!--more-->

我们将使用 ASP.NET MVC 5，因此请至少准备好一个支持 .NET 4.5 的 Visual Studio 版本（2012 或者 2013）。如果没有银子购买专业版或者以上版本，那么用免费的 Express 版本也是不错的。

首先我们要创建一个空白的 Web Application。打开 visual studio 并创建一个空白的 Web Application。为了照顾新同学我大发善心的告诉你，应该选择 `ASP.NET Empty Web Application`。微软会非常友好的添加一大堆你可能一辈子都用不着的程序集引用。因此我们第一件事儿就是删掉他们。删到如下程度为止：

```
Solution
|-Homework.App
    |-Properties
    |   |-AssemblyInfo.cs
    |-References
    |   |-System
    |   |-System.Core
    |-Web.config
```

## 安装 ASP.NET MVC 5 Packages

在前几个 ASP.NET MVC 版本的时候，你需要从官方网站上下个包，然后点 Setup 然后……总之一切都显得有点儿不专业，好在现在不用了。打开 nuget manager ，安装 `Microsoft ASP.NET MVC`，或者在 package management console 里面输入：

```
Install-Package Microsoft.AspNet.Mvc -Version 5.0.0
```

这个 package 会安装以下三个依赖包：`Microsoft ASP.NET Razor`、`Microsoft ASP.NET Web Pages` 以及 `Microsoft Web Infrastructure`。以下的东西不用记，如果有兴趣，你可以当看小说似的看一下：

* 其中第一个包是 ASP.NET MVC 的 Razor 的语言解析器，用于将 .cshtml 的服务端页面解析为抽象语法树，并提供编辑器支持。
* 第二个包，也就是 `ASP.NET Web Pages`，提供了 ASP.NET MVC 服务端页面编辑需要的各种辅助工具。
    * 例如，防止 XSS 和 CSRF 攻击的辅助类型（参见 `System.Web.Helpers`、`System.Web.Helpers.AntiXsrf` 名字空间）
    * Model 的客户端验证类型（就是类似“必须填写”，“必须在xxx范围之内”这种东西，当然，我对这些东西不抱有好感），以及最基本的 HTML 元素输出的支持（`TagBuilder`，这可是个好同志，如果你不用他书写自己的 `HtmlHelper` 扩展方法，而是还在用 `string.Format()`输出 HTML，那么，快去改掉那些该死的代码）。这些都在 `System.Web.Mvc` 名字空间里。
    * 服务端页面的基本类型：`WebPageBase`。你可能惊讶的发现了一个名为 `WebPage` 的类型，并且里面还有 `Model`，`Layout` 等你耳熟能详的属性，你兴奋的以为你找到了组织但是我只能告诉你你搞错了。这个 `WebPage` 真的是一坨 Rubbish。真正的 ASP.NET MVC WebPage 在 `System.Web.Mvc.WebViewPage` 里，这是一个为了向下兼容而遗留下来的历史问题。
    * Razor 服务端页面的编译逻辑（在 `System.Web.WebPages.Razor`里）。
    * 一些好用的和过时了的页面辅助类型（请看 `System.Web.Helpers`）。这里加入我的个人吐槽，如果你以后创建了一个 Framework，那么千万不要把辅助类加到你的 Framework 的包中，而是当作一个附加的包供大家自由下载使用。否则为了向后兼容性你不得不让这些丑陋的设计一直存在下去，并被持续吐槽。
* 第三个包，也就是 `Microsoft Web Infrastructure` 提供了 ASP.NET 的基础架构——才怪！里面充斥了对反射功能的封装（服务端页面需要这些特性）和系统环境的检查（例如注册表）。总之，加上它，ASP.NET MVC 需要它，但是我们不会使用它。

安装好以后我们的工程看起来貌似有些料了：

```
Solution
|-Homework.App
    |-Properties
    |   |-AssemblyInfo.cs
    |-References
    |   |-Microsoft.Web.Infrastructure
    |   |-System
    |   |-System.Core
    |   |-System.Web.Helpers
    |   |-System.Web.Mvc
    |   |-System.Web.Razor
    |   |-System.Web.WebPages
    |   |-System.Web.WebPages.Deployment
    |   |-System.Web.WebPages.Razor
    |-Web.config
```

## 创建基本工程结构

Convension over configuration 是目前大家偏好的设计形式，Rails 在这方面非常典型，其他的 MVC Framework 也在这样做。那么我们就按照这种约定创建工程结构吧。

* 首先我们需要一个 Controllers 名字空间来存放（默认 Area，我们以后会涉及 Area 这个概念） 下面的 Controllers 类型；
* 另外我们需要一个 Views 文件夹来存放服务端页面。

像这样：

```
Solution
|-Homework.App
    |-Properties
    |   |-AssemblyInfo.cs
    |-References
    |   |-Microsoft.Web.Infrastructure
    |   |-System
    |   |-System.Core
    |   |-System.Web.Helpers
    |   |-System.Web.Mvc
    |   |-System.Web.Razor
    |   |-System.Web.WebPages
    |   |-System.Web.WebPages.Deployment
    |   |-System.Web.WebPages.Razor
    |-Controllers
    |-Views
    |-Web.config
```

## Hello World

看标题你就知道激动人心的时刻即将来临了。让我们开始最后的冲刺吧。我们要添加一个 HomeController，然后添加一个 View，View 上书写着不变的咒语：Hello World。

这个过程网络上的文章多如牛毛，我就简述了，总之你应该得到如下的工程结构：

```
Solution
|-Homework.App
    |-Properties
    |   |-AssemblyInfo.cs
    |-References
    |   |-Microsoft.Web.Infrastructure
    |   |-System
    |   |-System.Core
    |   |-System.Web.Helpers
    |   |-System.Web.Mvc
    |   |-System.Web.Razor
    |   |-System.Web.WebPages
    |   |-System.Web.WebPages.Deployment
    |   |-System.Web.WebPages.Razor
    |-Controllers
    |   |-HomeController.cs
    |-Views
    |   |-Home
    |       |-Index.cshtml
    |-Global.asax
    |   |-Global.asax.cs
    |-Web.config
```

以上的内容我认为唯一可能出问题的就是如何创建 `Global.asax` 文件，单击右键，选择创建 `Glboal Application Class` 就可以了。

下面我们简单过一下新添加的文件的内容。其中，Index.cshtml 中的内容应该是类似这样的：

```html
@inherits System.Web.Mvc.WebViewPage

@{
    Layout = null;
}

<!DOCTYPE html>

<html>
    <head>
        <title>Index</title>
    </head>
    <body>
        <h1>Hello World!</h1>
    </body>
</html>
```

这里我们有必要解释几句。第一行，也就是 `@inherits System.Web.Mvc.WebViewPage` 这句话描述了当前页面的类型是 `System.Web.Mvc.WebViewPage`。我们之前说到了，所有的 ASP.NET MVC 页面的基础类型是 `WebPage`，而 `WebViewPage` 是 `WebPage` 的一个派生类，他提供了 MVC 的页面需要的所有方法和属性，例如 `Model` 属性，`Html` 属性，`Layout` 属性。

第二行，也就是 `@{ Layout = null; }` 这一行，说明了我们这个服务端页面的所有布局信息都在这里，没有母板页的影响。关于 `Layout`我们后续文章会有涉及。

接下来我们看看 `HomeController`，它里面的内容是这样的。

```csharp
using System.Web.Mvc;

namespace Homework.App.Controllers
{
    [RoutePrefix("home")]
    public class HomeController : Controller
    {
        [Route("")]
        public ActionResult Index()
        {
            return View();
        }
    }
}
```

`HomeController` 类中仅仅包含一个 `GET` 方法 `Index()` 他直接返回了 `View()` 这样，ASP.NET MVC 将依据 `Home/Index.cshtml` 的内容生成页面并返回给客户端。

哦，我敢肯定你发现了什么不同：`[RoutePrefix]` 和 `[Route]` 是个什么东西，原来的 `Route` 不是需要用 `RouteCollection.MapRoute()` 方法注册吗？漂亮！这是 ASP.NET MVC 5 提供的新特性（从 sinatra 抄过来的，不过我一点没有鄙夷的意思，因为 ASP.NET MVC 在不断的改进。大家一直都在互相抄，当一个新的语言或者 Framework 形成的时候，大家做的第一件事情就是把我们在其他语言和 Framework 上做的工作完完整整的重新抄一遍）。

问题是，原来我们需要在 Web 应用程序启动的时候使用 `RouteTable.Routes.MapRoute()` 指定路由的模版，那么现在要怎么做呢？请打开 `Global.asax.cs` 文件，删掉那些不用的方法，只留下如下的内容。

```csharp
using System;
using System.Web;
using System.Web.Mvc;
using System.Web.Routing;

namespace Homework.App
{
    public class Global : HttpApplication
    {
        protected void Application_Start(object sender, EventArgs e)
        {
            RouteTable.Routes.MapMvcAttributeRoutes();
        }
    }
}
```

`Application_Start` 方法会在应用程序启动的时候调用，而其中的 `RouteTable.Routes.MapMvcAttributeRoutes();` 就是在声明我们将使用 `Attribute` 进行路由的解析。

你是否迫不及待的按下了 Ctrl + F5 并且看到了期待已久的 Hello World？恭喜你！在下一节中我们将讨论一些 web.config 的配置的问题。