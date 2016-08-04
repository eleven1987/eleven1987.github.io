---
layout: post
title: "谁占了我的端口 for Windows"
date: 2016-01-07 02:01:01 +0800
comments: true
categories: tips powershell
---

今天在本地调试 Blog 的时候意外的出现了一些错误：127.0.0.1 4000 端口已经被其他的进程占用了。如何找到占用端口的进程呢？

<!--more-->

```shell
Configuration file: /_config.yml
            Source: .
       Destination: ./_site
      Generating...
                    done.
  Please add the following to your Gemfile to avoid polling for changes:
    gem 'wdm', '>= 0.1.0' if Gem.win_platform?
 Auto-regeneration: enabled for '.'
Configuration file: /_config.yml
jekyll 2.4.0 | Error:  Permission denied - bind(2) for 127.0.0.1:4000
```

## 先手动一步一步来

我们可以使用 `netstat` 命令查看各种端口的被进程的占用情况，例如，开一个命令行或者 powershell 窗口输入 `netstat -ano`。就得到了如下的报告。

```shell
TCP    0.0.0.0:135            0.0.0.0:0              LISTENING       888
TCP    0.0.0.0:445            0.0.0.0:0              LISTENING       4
TCP    0.0.0.0:1536           0.0.0.0:0              LISTENING       564
TCP    0.0.0.0:1537           0.0.0.0:0              LISTENING       848
TCP    0.0.0.0:1538           0.0.0.0:0              LISTENING       1416
TCP    0.0.0.0:1539           0.0.0.0:0              LISTENING       1892
TCP    0.0.0.0:1540           0.0.0.0:0              LISTENING       732
TCP    0.0.0.0:1569           0.0.0.0:0              LISTENING       724
TCP    0.0.0.0:5357           0.0.0.0:0              LISTENING       4
TCP    127.0.0.1:1575         127.0.0.1:48303        ESTABLISHED     2104
TCP    127.0.0.1:4000         0.0.0.0:0              LISTENING       11476   <= Oops
TCP    127.0.0.1:4012         0.0.0.0:0              LISTENING       10888
TCP    127.0.0.1:4013         0.0.0.0:0              LISTENING       10888
```

说明一下参数：

* -a：显示所有链接和侦听端口
* -n：以数字形式显示地址和端口号而不会去尝试进行外部地址解析，能够显著的提高执行速度
* -o：显示拥有与每一个链接关联的 PID。其实他还有一个 fancy 的参数 **-b** 可以直接拿到与指定链接关联的可执行程序的名称。但是这个参数需要管理员权限！

嫌疑人 Get，是一个 PID 为 <span style="color:#cf0000">11476</span> 的进程占用了 4000 端口，除掉他之前先看看到底是谁？这个可以通过 `tasklist /svc /FI "PID eq 11476"` 搞定。

再说明一下参数：

* /svc：如果这个进程是一个 Windows 服务的话同时显示这个服务的名称；
* /FI：使用筛选器对结果进行筛选。

以下是运行结果：

```shell
Image Name                PID      Service
========================= ======== ============================================
FoxitProtect.exe             11476 FxService
```

好的，我昨天刚刚安装了一个 FixitReader。停掉服务，一切搞定。

## 再来一个自动一点儿的

写个 Powershell 脚本吧：

```shell
 param([string] $EndPointPattern)

 $pids = netstat -aon | ?{ $_.contains("$EndPointPattern") } | %{ $_.split(' ', [system.stringsplitoptions]::RemoveEmptyEntries)[4] } | select -unique
 $pids | %{ ps -Id $_ }
```

这里面没有考虑各种乱七八糟的情况，如果要用，请自行增补细节。好了，运行一下吧：

```shell
$ .\Search-EndPointProcess.ps1 -EndPointPattern "127.0.0.1:48303"

Handles  NPM(K)    PM(K)      WS(K) VM(M)   CPU(s)     Id  SI ProcessName
-------  ------    -----      ----- -----   ------     --  -- -----------
   1047      30     8516       5044   178            2104   0 svchost
```

不错嘛！:-D