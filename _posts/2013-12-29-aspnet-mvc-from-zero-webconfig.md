---
layout: post
title: "ASP.NET MVC 从零开始 - Web.config"
date: 2013-12-29 23:24:20 +0800
comments: true
categories: aspnet mvc webconfig
---

在上一篇中，我们从零开始创建了一个非常简单的 ASP.NET MVC 应用程序。接下来，你是不是期望我们能够给这个新生的应用程序添加各种各样的功能呢？可惜，不是这样的。我们下面的工作是创造一个自动部署这个应用程序的脚本。这在任何时候都是非常重要的。

这个重要的任务很难在一篇文章中完成，因此我们先看一看自动部署中一个非常重要的部分：web.config 文件。在这篇文章中，我们将解决如下的问题：

* web.config 文件是个什么东西
* web.config 对部署的影响
<!--more-->

## web.config 文件是什么

一般来说，`web.config` 文件包含了所有运行 ASP.NET 应用程序所需的配置信息。你也许马上发现了破绽：真的是“所有”吗？在上一个应用程序中它一共只有不到10行代码：

```xml
<?xml version="1.0"?>

<configuration>
  <system.web>
    <compilation debug="true" targetFramework="4.5" />
    <httpRuntime targetFramework="4.5" />
  </system.web>
</configuration>
```

但是这个应用却可以依靠着这个配置文件运行在 IIS Express 中，当然，也会运行在 IIS 中。

如果只算这个 `web.config` 文件，当然没有包含所有的配置。为了运行我们的 Web 应用，至少要包含三个方面的信息：

* 我们的 Web 应用也是一个 .NET 应用，因此会包含一些 .NET 应用通用配置信息。例如 `<connectionString>`、`<system.data>` 配置节的内容就是此类配置的典型；
* 其次我们的 Web 应用是一个 ASP.NET Web 应用程序，因此会包含 ASP.NET 相关的配置信息。例如 `<system.Web>` 配置节的内容就是此类配置的典型；
* 到目前为止，我们的 Web 应用是 Host 在 IIS 上的，因此需要包含 IIS 相关的配置信息。例如 `<system.webServer>`、`<system.applicationHost>`、`<system.ftpServer>` 配置节就是此类配置的典型；

那么这么多的信息是如何组织起来的呢？实际上这些信息是通过一种继承的方式进行组合的。以我们所做的 Web 应用为例。当应用分别部署在 IIS 和 IIS Express 上的时候，会有如下的配置继承结构：

```
IIS 7:
{.NET Framework Dir}\Config\machine.config
{.NET Framework Dir}\Config\web.config
{IIS Dir}\Config\applicationHost.config
    |-{Our Web Application Dir}\web.config

IIS Express
{.NET Framework Dir}\Config\machine.config
{.NET Framework Dir}\Config\web.config
{IIS Express Dir}\config\AppServer\applicationHost.config
    |-{Our Web Application Dir}\web.config
```

继承结构的第一级是服务器范围内的根配置文件，这种配置文件有三个：

* 首先出现的是 `machine.config` 和根 `web.config`。其中前者的配置将应用于本机的所有 .NET 应用，而后者的配置将应用于所有的 ASP.NET 应用。这两个文件的路径都位于 .NET Framework 的特定版本所在的目录。例如，例如 .NET 2.0 的安装路径是 `%windir%\Microsoft.NET\Framework\v2.0.50727`, .NET 4.0 的安装路径是 `%windir%\Microsoft.NET\Framework\v4.0.30319`（64-bit 的 .NET Framework 的 Framework 文件夹的名字是 `Framework64`）。
* 服务器范围内的配置文件还有 `applicationHost.config`。这个配置文件是用于存储 IIS 的运行配置的，它的值将作用于所有部署于本机 IIS / IIS Express 的应用。

继承结构的第二级就是我们的 Web 应用，这种应用可以存在于 Web 应用的根路径和各级虚拟目录。他们的继承层次和虚拟目录的层次是一致的。

## web.config 和部署

继承机制是非常不错的，但是这引入了另外一个问题，如果目标服务器并没有这些默认配置，或者这些配置在不同环境不尽相同，那么我们的应用程序不就不能正常工作了吗？很不幸，被你言中了。这就是为什么当你先安装了 .NET Framework 而后安装了 IIS 服务的话需要执行 `aspnet_regiis` 的原因了。不过我们还是有几种方法来应对这个问题：

* 把所有的配置覆写一遍（找抽），这里所谓的覆写仅限于 .NET 应用配置和 ASP.NET 应用配置，IIS 的 `applicationHost.config` 不要手动更改可以通过 .NET IIS Interop 或者 powershell 的 IIS Extension 进行更改。
* 重写当前应用程序使用到的的配置节（即使它已经存在在开发环境下的 `machine.config` 和根 `web.config` 中）；
* 重置当前服务器的默认配置文件(`machine.config` 以及 `web.config`)，然后以此为基础，仅仅书写和默认配置不同的配置。

上述三种方法中，后两种方法是比较可行的。但是在阅读了下一篇，也就是关于 ASP.NET 处理机制的介绍之后，你会发现第二种方法仍然存在相当的难度（因为需要覆写的东西实在太多）。因此目前广泛采用的是第三种方法。

以上就是这一篇的内容，现在还不是我们写部署脚本的时候 :-)。下一篇我们将介绍 ASP.NET 的处理机制。

## 附1：machine.config 的默认设定

* 默认使用 RSA 对受保护的 config 文件进行加密；
* 默认连接本地 aspnetdb.mdf 数据库；
* 默认激活如下的 WCF Service Behavior：
    * Windows Workflow Foundation 相关 Service Behavior；
    * WS*-HTTP，以及 RESTful WCF Service Behavior（和 .NET 3.5SP1 兼容，但是之后应该使用 WebApi）；
    * WCF Service Discovery Behavior；
    * WCF 服务 Buffer 传输支持（相对的还有 Stream 传输支持）；
* WCF 的 WS*-HTTP，TCP，web-HTTP，basic-HTTP 绑定支持；
* WCF 的各种 endpoint 支持；
* 默认使用 SQL Server 进行 Membership 和 Role 的管理（Authentication 相关）；
* 默认使用 SQL Server 存储 Profile 的结果；

## 附2：根 web.config 的默认设定

* 指定了不同的 trust level 所对应的根 web.config 文件；
* 默认的 trust level 是 `Full`；
* 将 C# 和 VB 的编译器添加到服务端页面编译器中；
* 添加服务端页面编译时默认引用的程序集，添加需要编译的文件类型及其相应的 Build Provider；
* 使用 Internet Settings 中的代理服务器策略；
* 允许所有用户访问 Web 应用程序；
* 添加 Log 相关的默认缓存设置（包括大小，Flush 间隔等），将系统日志 Logger 设置为默认的 Log Provider，并将不同类型的日志映射到系统日志的类型；
* 添加默认的 HTTP Handlers（这可能使咱们最常接触的内容了）；
* 添加默认的 HTTP Modules；
* 添加 Web UI 控件支持和站点地图支持（ASP.NET Webform）；
* 启用 Url 映射支持（就是把 Request 的 URI 映射到另外一个 URI）
* 默认 WCF 的 tcp，MS消息队列，命名管道并不和 ASP.NET 处理管道相互兼容（他们是分开运行的）；