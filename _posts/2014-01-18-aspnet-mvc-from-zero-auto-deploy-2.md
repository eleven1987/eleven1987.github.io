---
layout: post
title: "ASP.NET MVC 从零开始 - 自动化部署（其二）"
date: 2014-01-18 20:41:19 +0800
comments: true
categories: aspnet mvc deploy msbuild
---

这是这个系列的第五篇了，前四篇请参见：

* [ASP.NET MVC 从零开始 – Create and Run](http://lxconan.github.io/2013/12/27/aspnet-mvc-from-zero-create-and-run)
* [ASP.NET MVC 从零开始 – Web.config](http://lxconan.github.io/2013/12/29/aspnet-mvc-from-zero-webconfig/)
* [ASP.NET MVC 从零开始 - 请求处理](http://lxconan.github.io/2014/01/01/aspnet-mvc-from-zero-message-processing/)
* [ASP.NET MVC 从零开始 - 自动化部署（其一）](http://lxconan.github.io/2014/01/12/aspnet-mvc-from-zero-auto-deploy-1/)

简单来说，部署就是 “构建（Build）” -> “拷贝（打包）” -> “配置”。在前一篇中，我们介绍了“构建”，那么这一篇就说说拷贝（好像我们更习惯于说打包，那么以后我们就叫它打包吧）的事情。为什么要打包呢？在应用程序发布的时候我们当然只希望发布运行时需要的文件，而其他的文件，例如：工程文件，源代码等等是不需要进行发布的。因此我们需要将运行时所需的文件分离出来，做成一个干净的 Package。

<!--more-->

## 打包 - 思路

只需要解决楚两个问题，打包就完成了：第一个问题是，我们打的包应该有怎样的目录结构；第二个问题是，应该拷贝哪些文件夹到包的哪些目录里去。

## 应该拷贝哪些文件

在回答第一个问题之前，我们先来看看有哪些文件需要进行拷贝。构建好的程序集（.dll 和 .exe）需要拷贝，没错，但是除了它们以外还有其他文件需要进行拷贝。如果在 *Visual Studio* 中打开 Web Project，并观察每一个文件的 *Build Action* 属性，你会发现几乎所有的文件都属于以下四种 *Build Action*：

* None：这意味着这个文件在构建过程中将不做任何处理。典型的例子是 Readme 或者 EULA（End User License Agreement） 文件。这种文件**不会**在打包中进行拷贝；
* Compile：这类文件会在构建过程中进行编译，编译结果会嵌入到生成的程序集（dll 或者 exe）中。这类文件在打包的时候是**不会**进行拷贝的；
* Content：这个文件不会在构建过程中进行编译。但是这个文件属于整个工程发布的一个部分。因此这类文件在打包的时候**会**进行拷贝；
* Embedded Resources：这个文件的内容将作为一种嵌入式资源在构建过程中嵌入到程序集中。这个文件在打包的过程中**不会**被拷贝；

因此，除了构建好的程序集之外，所有 *Build Action* 为 *Content* 的文件类型也会在打包的时候被拷贝。

以我们的工程为例：

```
FromZero.App
│  Global.asax [Content]
│  Global.asax.cs [Compile]
│  packages.config [Content]
│  Web.config [Content]
│  Web.Debug.config [None]
│  Web.Release.config [None]
│
├─bin
│  /* All build results are stored in this directory. */
│
├─Controllers
│   HomeController.cs [Compile]
│
├─Properties
│   AssemblyInfo.cs [Compile]
│
└─Views
    └─Home
          Index.cshtml [Content]
```

那么需要拷贝的文件为：

* .\bin 文件夹下的所有文件；
* 所有 *Build Action* 为 *Content* 属性的文件：Global.asax、packages.config、Web.config、Index.cshtml。

## 包的目录结构

在上一节我们介绍了，所有构建生成的程序集和 *Build Action* 为 *Content* 的文件都会在打包过程中进行拷贝。那么它们会拷贝到什么地方去呢？答案是拷贝到相应的目录下面去。以我们的工程为例，假设我们希望将构建好的工程拷贝到一个名为 Package 的目录下去，那么这个 Package 目录在打包完毕之后应该是这个样子的：

```
Package
│  Global.asax
│  packages.config
│  Web.config
│
├─bin
│  /* All build results. */
│
└─Views
    └─Home
          Index.cshtml
```

等一下，Controller 和 Properties 目录到哪里去了？由于这两个目录下面没有一个文件需要进行发布，因此这个目录也就不会创建。

> 假设你的确需要一个 Controller 目录进行发布，该怎么办呢？那么我们可以利用规则创建一个 0KB 的 placeholder 文件。并且将这个文件的 *Build Action* 属性设置为 *Content*。

至此我们已经可以总结出打包的规则了：

* 拷贝所有构建过程中生成的程序集文件，以及 *Build Action* 为 *Content* 的文件；
* 将所有需要拷贝的文件拷贝到一个和其所在的工程目录对应的目录下面，如果某一个目录下没有一个文件需要在打包中进行拷贝，则不生成这个目录。

## 打包-代码

我们是否需要自己解析工程的 XML 结构然后按照上述规则进行打包呢？幸运的是，完全不用：这是因为在 ASP.NET Web 工程中会引用 *$(VSToolsPath)\Web\Microsoft.Web.Publishing.targets*，其中定义的 *_WPPCopyWebApplication* 过程正是我们以上描述的过程。我们只需要在上一个例子的基础上修改 `Compile-Project` 函数：

```powershell
Function Compile-Project() {
    iex -Command "& '$global_msBuildPath' /t:Rebuild /t:_WPPCopyWebApplication /p:WebProjectOutputDir='$global_buildDirPath\Package\' /p:UseWPP_CopyWebApplication=True /p:PipelineDependsOnBuild=False '$project_path'"
}
```

其中：

* `$global_msBuildPath` 是 msbuild.exe 的所在位置；
* `/t:Rebuild`：首先执行 Rebuild 过程，这将删除上一次的构建结果，然后重新构建整个项目；
* `/t:_WPPCopyWebApplication`：将该项目进行打包；
* `/p:WebProjectOutputDir='$global_buildDirPath\Package\'`：将整个打包结果存放在 buildDir 下的 Package 目录下。如果这个目录不存在则创建这个目录；
* `/p:UseWPP_CopyWebApplication=True`：从 *Visual Studio* 2010 开始，我们可以使用 Web.config.\$(Configuration).config 文件对 Web.config 在不同的编译选项下进行修正。为了使用能够这个功能，需要设定此变量值为 `True`；
* `/p:PipelineDependsOnBuild=False`：如果将 `UseWPP_CopyWebApplication` 设置为 `True`，则必须将 `PipelineDependsOnBuild` 变量设置为 `False` 否则将导致 *MSBuild* 的 Targets 的循环引用。具体的技术细节请参见[这里](http://blogs.planetsoftware.com.au/andy/archive/2011/06/17/web.config-transforms-xdt-in-team-build-2010.aspx)。

这么长的一坨命令非常不容易维护，因此我们可以将这些命令放在一个 *MSBuild* 工程中。首先，我们建立一个 XML 文件，不妨命名为 Deploy.xml：

```xml
<?xml version="1.0" encoding="utf-8"?>
<Project 
  xmlns="http://schemas.microsoft.com/developer/msbuild/2003" 
  ToolsVersion="12.0">
  <Target Name="Build">
    <MSBuild
      Projects="..\src\FromZero.App\FromZero.App.csproj" 
      Targets="Rebuild;_WPPCopyWebApplication"
      Properties="WebProjectOutputDir=$(WebAppPublishDir);UseWPP_CopyWebApplication=True;PipelineDependsOnBuild=False;"/>
  </Target>
</Project>
```

这样，我们只需要在 `Compile-Project` 函数中用 *MSBuild* 调用这个 Deploy.xml 文件，并将希望的包的输出目录赋值给 `$(WebAppPublishDir)` 变量即可：

```powershell
$global_deployProject = "$global_buildDirPath\deploy.xml"

Function Compile-Project() {
    iex -Command "& '$global_msBuildPath' /p:WebAppPublishDir='$global_buildDirPath\Package\' '$global_deployProject'"
}
```

到现在，`Compile-Project` 函数已经不止是在编译工程了，它还具备了打包的能力，因此我们将其重命名为 `Deploy-Project`。

## 附：deploy.ps1 到目前为止的代码

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
$global_deployProject = "$global_buildDirPath\deploy.xml"

# Install nuget packages ---------------------------------

Function Install-SolutionPackages() {
    iex "$global_nugetPath restore $global_solutionFilePath"
}

$project_path = $global_solutionPath + '\FromZero.App\FromZero.App.csproj'

Function Deploy-Project() {
    iex -Command "& '$global_msBuildPath' /p:WebAppPublishDir='$global_buildDirPath\Package\' '$global_deployProject'"
}

Install-SolutionPackages
Deploy-Project
```