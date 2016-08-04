---
layout: post
title: "ASP.NET MVC 从零开始 - 自动化部署（其一）"
date: 2014-01-12 01:38:51 +0800
comments: true
categories: asp.net mvc deploy powershell
---

这是这个系列的第四篇了，前三篇请参见：

* [ASP.NET MVC 从零开始 – Create and Run](http://lxconan.github.io/2013/12/27/aspnet-mvc-from-zero-create-and-run)
* [ASP.NET MVC 从零开始 – Web.config](http://lxconan.github.io/2013/12/29/aspnet-mvc-from-zero-webconfig/)
* [ASP.NET MVC 从零开始 - 请求处理](http://lxconan.github.io/2014/01/01/aspnet-mvc-from-zero-message-processing/)

这一篇中我们会写一些关于自动化部署的代码。我们会使用 *Powershell* 书写这类代码。我不会向你讲为什么要自动化部署，这种大道理讲的太多了。你将发现这篇文章中涉及的东西非常具体，有的要求甚至相当苛刻且可能不具有通用性。这是因为部署从来都是跟环境打交道，部署过程中协作的组建太多，相互之间的交集不可能太大。可能唯一能够通用的是自动化部署的基本原则（只是这篇文章的基本原则）：

* 每一次自动化部署结束之后，应用程序都会有相同的初始状态。
* 自动化部署的机器非常干净，只有相应的 *Windows Server* 系统和 *.NET Framework*。尤其是，不会有 Visual Studio。

<!--more-->

我们需要公开一些基本的环境信息：

* 64-bit Windows Server 2008/2012/2012 或者 64-bit Windows 7/8/8.1
* 我们的工程是使用 *Microsoft Visual Studio 2013* 开发的；
* *.NET Framework* 版本为 4.5
* *ASP.NET MVC* 的版本为 5.0.0
* *Microsoft Build Tool* 的版本为 12.0

> 关于 *MSBuild* 这里需要多说几句。在 *Visual Studio* 2013 发布之前，*MSBuild* 是随 *.NET Framework* 一起发布的。这意味着我们可以不用安装 *Visual Studio* 就可以构建 .NET 应用程序。

> 这是多么美好的世界啊！因此，上一句话是假的。尤其是当你构建 *ASP.NET* Web 应用程序的时候，你马上就会遇到 "Cannot found ... WebApplication.target" 的错误。如果你 StackOverflow 一下就可以知道，我们需要做的是安装 *Visual Studio* 或者从另外一台安装了 *Visual Studio* 的机器上将相关的文件拷贝出来。这真是——丢人的设计！Linux 的拥趸又有了发泄的空间。

> 于是微软决定正视这个问题。从 *Visual Studio* 2013 开始，*MSBuild* 将与 *Visual Studio* 而不是 *.NET Framework* 一起发布。这引来了一片[骂声](http://blogs.msdn.com/b/visualstudio/archive/2013/07/24/msbuild-is-now-part-of-visual-studio.aspx)。我不得不说，大家在生活中不论遇到什么事情都要保持足够的冷静，往往先发脾气的得不到任何的好处。事实是，由于 *.NET Framework* 发布的周期是相对较长的，因此微软目前越来越倾向于使用 NuGet 进行类库的发布，这样有助于削减 *.NET Framework* 基础类库的大小，并缩短基础类库的发布周期。相应的 *MSBuild* 和 *Visual Studio* 的发布周期也希望进行独立的变化。*MSBuild* 将和 *Visual Studio* 保持发布周期的一致性，并不意味这 *MSBuild* 依赖于 *Visual Studio*。因此我们当然可以下载 *MSBuild* 的独立安装包 [*Microsoft Build Tools*](http://www.microsoft.com/en-us/download/details.aspx?id=40760)，并在一台没有 *Visual Studio* 的服务器上进行自动化构建。

再次强调，如果幸运，你可以直接运行这篇文章中的例子。但是你不要指望将这篇文章的脚本拷贝到你的工程就可以正常工作。因为部署是一个因地制宜的任务，需要具体分析。你可能会遇到各种各样的环境问题，但是我认为最重要的还是思路。

## 部署是什么

简单来说，部署就是 “构建（Build）” -> “拷贝” -> “配置”。那么我们就开始一个一个解决它。为了避免这篇文章过长，我们将分及部分介绍部署的过程。这一篇将着眼于构建。

## 编译和构建应用程序 - 思路

如何构建应用程序呢？答案是先把需要的东西都拷贝过来，然后再用工具构建。好极了，我们需要拷贝什么东西呢？当然是先把源代码拷贝过来。

```
┌────────────────┐
│ Src of MyApp   │
└────────────────┘
```

光源代码还不行，因为我们的程序依赖于第三方库（箭头代表依赖关系）。

```
┌────────────────┐
│ Dependent libs │
└────────────────┘
        ↑
┌────────────────┐ 
│ Src of MyApp   │
└────────────────┘
```

例如，对于我们构建的那个简简单单的 ASP.NET MVC 5 的应用程序，我们就需要在构建之前下载它依赖的类库：

* Microsoft.AspNet.Mvc 5.0.0
* Microsoft.AspNet.Razor 3.0.0
* Microsoft.AspNet.WebPages 3.0.0
* Microsoft.Web.Infrastructure 1.0.0.0

源代码和依赖库都拷贝完了，那么接下来我们就需要将构建工具也拷贝过来：

```
┌────────────────┐
│ Dependent libs │
└────────────────┘
        ↑
┌────────────────┐     ┌───────────────────┐
│ Src of MyApp   │────→│Tools for compiling│
└────────────────┘     └───────────────────┘
```

对于我们来说，构建的工具只有一个：

* MSBuild

如今工具，源代码一应俱全，我们可以开始编译了。

## 安装 MSBuild

如果你做实验的机器上安装了 *Visual Studio* 2013——Express 版本的也是可以的——那么你可以跳过这一步，否则请下载 [*Microsoft Build Tools*](http://www.microsoft.com/en-us/download/details.aspx?id=40760) 并安装。

## 下载应用程序的代码

我们的代码是现成的，只需要从 git repository clone 下来就可以了。你也可以从这个地址 clone 到一个范例代码，并转换到相应的 commit。

```
git clone https://github.com/lxconan/MvcFromZero.git
git reset --hard 340abb32433c99975bd6485f79db6ca077119477
```

好了，完成了，到目前为止一切都是那么的轻松愉快。

## 下载应用的依赖库

我们将使用 nuget 管理包的依赖。因此我们首先要下载 *NuGet* 的命令行客户端，可以从[这里](http://nuget.org/nuget.exe)下载。

接下来的行为都需要明确的目录结构。为了便于说明，我们将建立如下的目录结构。

```
MvcFromZero
 |- src
 |   |- FromZero.App
 |   |- packages
 |
 |- build
     |- tools
```

其中，src 目录存放 Solution 的源代码，而 build 目录存放自动化构建所需的各种工具和脚本。其中自动化构建需要的工具将放在 build/tools 目录下。因此我们也会将 nuget.exe 放在这个目录下。

接下来我们在 build 目录下建立 deploy.ps1 脚本，我们将在这个脚本中完成接下来的任务。我们先来下载依赖的包。NuGet 的包管理是通过 *package.config* 文件进行的。需要指出的是 *package.config* 也具有一定的层次关系。首先 Solution 级别的包信息存储在两个地方：

* 第一个是 Solution 目录下的 *.nuget/package.config* 文件，这个文件中存储的是非特定项目使用的包（如果并没有这种类型的包，则该目录不存在，或者该目录下没有任何文件）；
* 第二个是 Solution 目录下的 *packages/repositories.config* 文件，这个文件存储了该解决方案下的每一个工程的 *packages.config* 文件的路径。

而 Solution 下的每一个 Project 的包定义存储在 Project 目录下的 *package.config* 中。

为了下载这些依赖的包是否需要人为遍历这些 *config* 文件呢？原来是，而从 nuget 2.7 开始，可以直接支持 Solution 范围内的包下载，于是额我们就可以使用如下的简单函数完成包的下载了。

```powershell
Function Install-SolutionPackages() {
    iex "$global_nugetPath restore $global_solutionFilePath"
}
```

其中，`$global_nugetPath` 是 nuget.exe 的路径，而 `$global_solutionFilePath` 是工程文件（在这里是 FromZero.App.csproj）的路径。

接下来需要确定的是构建工具的位置，我们可以使用注册表查找 *MSBuild* 的位置的，由于我们使用了 2013 版本的 *MSBuild*（Toolset 的版本号为 12.0），因此我们的查找脚本为：

```powershell
Function Get-MsBuildPath() {
    $msBuildRegPath = "HKLM:\SOFTWARE\Microsoft\MSBuild\ToolsVersions\12.0"
    $msBuildPathRegItem = Get-ItemProperty $msBuildRegPath -Name "MSBuildToolsPath"
    $msBuildPath = $msBuildPathRegItem.MsBuildToolsPath + "msbuild.exe"
    return $msBuildPath
}
```

上述两步完成之后，就可以编译我们的工程了：

```powershell
Function Compile-Project() {
    iex -Command "& '$global_msBuildPath' $project_path"
}
```

你迫不及待的试验了你的代码，但是却发现出了问题，*MSBuild* 报告无法找到 *Microsoft.WebApplication.targets* 文件。这是怎么回事，我们已经明明在 *MSBuild* 中安装了 *Microsoft.WebApplication.targets* 了啊？这是因为为了保持和 *Microsoft Visual Studio* 2010 SP1 的工程的兼容性，*Visual Studio* 2012/2013 的工程默认将 `$(VisualStudioVersion)` 环境变量设置为 `10.0`。这样，target 文件的查询路径就成了：

```
$(MSBuildExtensionsPath32)\Microsoft\VisualStudio\v10.0
```

而不是：

```
$(MSBuildExtensionsPath32)\Microsoft\VisualStudio\v12.0
```

因此我们需要修正这个环境变量。请使用文本编辑器打开 *FromZero.App.csproj* 文件，找到如下的定义：

```xml
<PropertyGroup>
  <VisualStudioVersion Condition="'$(VisualStudioVersion)' == ''">10.0</VisualStudioVersion>
  <VSToolsPath 
    Condition="'$(VSToolsPath)' == ''">
    $(MSBuildExtensionsPath32)\Microsoft\VisualStudio\v$(VisualStudioVersion)
  </VSToolsPath>
</PropertyGroup>
```

将其中的 `VisualStudioVersion` 节替换为：

```xml
<VisualStudioVersion>12.0</VisualStudioVersion>
```

再次运行：祝贺你，你已经能够成功的下载所有的依赖，并完成工程的编译了！在本文的结束，附上 deploy.ps1 到目前为止的所有代码：

```powershell
$ErrorActionPreference = 'Stop'

# Environment helpers ------------------------------------

Function Get-MsBuildPath() {
    $msBuildRegPath = "HKLM:\SOFTWARE\Microsoft\MSBuild\ToolsVersions\12.0"
    $msBuildPathRegItem = Get-ItemProperty $msBuildRegPath -Name "MSBuildToolsPath"
    $msBuildPath = $msBuildPathRegItem.MsBuildToolsPath + "msbuild.exe"
    return $msBuildPath
}

# Environment variables ----------------------------------

$global_buildDirPath = Get-Location
$global_msBuildPath = Get-MsBuildPath
$global_solutionPath = "$global_buildDirPath\..\src"
$global_solutionFilePath = "$global_solutionPath\src.sln"
$global_nugetPath = "$global_buildDirPath\tools\nuget.exe"

# Install nuget packages ---------------------------------

Function Install-SolutionPackages() {
    iex "$global_nugetPath restore $global_solutionFilePath"
}

$project_path = $global_solutionPath + '\FromZero.App\FromZero.App.csproj'

Function Compile-Project() {
    iex -Command "& '$global_msBuildPath' '$project_path'"
}

Install-SolutionPackages
Compile-Project
```