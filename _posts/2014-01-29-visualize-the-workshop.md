---
layout: post
title: "Visualize The Workshop"
date: 2014-01-29 02:32:01 +0800
comments: true
categories: visualize coaching
---

今天这篇文章是准备瞎扯的。平常工作的时候，我希望尽可能的将一切自动化，让自己尽可能的舒适与懒惰。两个输入设备（键盘+鼠标）太累，我只想用一个，例如我不希望翻箱倒柜的去翻找 GO-Agent 在哪里，我希望用一个命令就可以开启它。我希望用一个脚本就完成整个工程的依赖的下载，构建，打包，部署，然后让 Pipeline 去跑自动化的单元测试和集成测试。总之，好像有命令行就足够了，我不需要GUI。

但是，这是不可能的。即使在工作中 80% 的时间都在敲打键盘，但是其余的事件我都在浏览器上转悠，这需要鼠标（我知道 Chrome 上有类似 Vim 快捷键的东西，但是它是有副作用的，基于这种考虑，我很少在测试环境下使用它）。在其他的一些场景下，我也希望有接口有图形化的表示，例如，查看 git 的分支。还有一个领域，那就是 Coach。

<!--more-->

## 只有命令行的 Workshop 简直弱爆了

相比于 Session 来说，大家更喜欢动手。这可能是由于 Session 以 PPT 为主，例子较少。在不能够主动参与的情况下大家都昏昏欲睡。于是有了各种形式的 Workshop。但是我的确参加过类似于以下这种形式的 Workshop，完全没有可视化的内容，只有命令行的操作，我很不喜欢，我只能说，它弱爆了。

例如，我们现在有一个配置 kvm 的 Workshop。主持人开场之后告诉你，很多版本的 Linux 已经自带了 kvm。我给你的也是这个样子的，不但如此，我已经帮你进行了一些基础的配置。这样我们就可以略掉很多繁琐的操作。首先我们创建一个虚拟硬盘：

```bash
qemu-img create -f qcow2 winxp.img 10G
```

然后创建虚拟机：

```bash
sudo qemu-system-x86_64 -m 512 -drive file=/home/lm/kvm/winxp.img,cache=writeback -localtime -net nic,vlan=0,macaddr=52-54-00-12-34-01 -net tap,vlan=0,ifname=tap0,script=no -boot d -cdrom /home/lm/iso/winxp.iso -smp 2 -soundhw es1370
```

其中，`-m 512` 是分配 512MB 内存，`-drive file=/home/lm/kvm/winxp.img,cache=writeback` 是……

然后……好了我已经不想再说下去了。首先我很奇怪这种 Workshop 针对的对象是什么人呢？高手？那么他完全不需要来听这些东西。初学者？拜托，你希望他照猫画虎以达到培养量产型人才的目的吗？这种可怕的 Workshop 的结果可能是：你准备好了一切，然后大家觉得一共就几个命令，没有学到什么东西；或者在 Workshop 进行的过程中问题层出不穷，但是当出现问题的时候你的听众根本不知道如何解决问题，只能干等着你来救命，或者干脆放弃这次的 Workshop。

你说，想成为高手必须自己如何如何……老实说，我认为你在赤裸裸的逃避身为一个 Coach 的责任。为什么大家只要一步跟不上就步步跟不上，或者即使是跟不上下来之后只能自己去重新整理。因为你没有告诉他思路。你说不是啊，我告诉了啊，我每次都提前说，第一，……；第二，……；我只能说那是没有用的。你最好用一个图来可视化你的思路，或者此次 Workshop 的流程。例如：

```
┌────────────────┐   ┌────────────────┐   ┌────────────────┐ 
│  Hard disk     │ + │    Hardware    │ + │      Host      │ = Virtual Machine
└────────────────┘   └────────────────┘   └────────────────┘
        ↑ Install         ↑ Config              ↑ Download
        | system          | CPU, Mem            | and 
        | on              | Network..           | Install
┌────────────────┐   ┌────────────────┐   ┌────────────────┐ 
│ Install Media  │   │ configuration  │   │      kvm       │
└────────────────┘   └────────────────┘   └────────────────┘
```

这个图并不一定绝对严谨，但是他有三个好处：第一，极大的减轻听众的心理压力，因为他们看到了这张图会感觉到这个事情可能并不是那么难做；第二，防止听众 Lost，即便是他没有跟上如何配置 Hardware 这一节，但是看了图就知道这并不影响学习其余两个部分；因此他就不会在下载并安装 kvm 这一节还在不断的琢磨，我刚才怎么就配不成 Hardware 呢？继而加剧 Lost。第三，在某些部分 Lost 的情况下有据可查，方便线下自行练习。

另外，使用这种流程图形还有一个好处，那就是听众会记住这些流程，进而会把自己的脚本按流程进行隔离，这总比把一坨东西都揉在一起好多了。

## 图形是理解抽象的必杀技

有些时候我们不得不面对一些不容易描述的技术且难以调试的技术，例如 C# 的 Task Parallel System 。当你面对着如下的代码的时候：

```csharp
[Fact]
public async Task complex_await()
{
    var numbers = new int[2];
    string result = await Task.Factory.StartNew(
        () => V.Start("task 1").Sleep(1).Finish("The numbers are: "));
    for (int i = 0; i < 2; ++i)
    {
        int number = await Task.Factory.StartNew(
            state =>
            {
                int stateIndex = (int) state;
                return V.Start("task " + (stateIndex + 2)).Sleep(2).Finish(stateIndex*4);
            },
            i);
        numbers[i] = number;
    }

    // ...
}
```

有两个急切需要解决的问题：

1. 如何描述，大家才能够了解这段代码到底是如何执行的；
2. 如果听众想自己做实验，写一段类似的代码，那么他有办法知道这段代码是如何执行的吗；

这两个问题解决好了，那么你的 Workshop 不仅仅有**鱼**，而且有**渔**。所有的参与者会非常好奇，他们不会满足于只看懂你提供的例子，他们希望自己写一些代码，然后解释他们的代码是怎么执行的。为了达到这个目的，我写了一个小小的工具，使得当你执行上述单元测试的时候，提供如下的输出：

<img src="{{root_url}}/images/blog/visualize_workshop_1.png" alt="task visualization 1"/>

一切都是一目了然的，循环外的 `Task` —— *Task 1* 首先在 Id 为 7 的线程上执行，在执行完毕之后，循环中的两个 `Task` 先后在 Id 为 8 和 Id 为 7 的线程上执行。三个 `Task` 有明确的执行先后关系，估计不用我说你也大概明白 `await` 的作用是什么了。如果你好奇心足够强，就会自己写一些更酷的代码做测试，例如：

```csharp
[Fact]
public async Task complex_await()
{
    var ids = new[] {1, 2, 3, 4};
    await Task.Factory.StartNew(() => V.Start("task 0").Sleep(1).Finish());
    await Task.WhenAll(ids.Select(
        id => Task.Factory.StartNew(() => V.Start("task " + id).Sleep(2).Finish())));
    await Task.Factory.StartNew(() => V.Start("task 5").Sleep(1).Finish());
}
```

那么你将得到如下的可视化执行效果：

<img src="{{root_url}}/images/blog/visualize_workshop_2.png" alt="task visualization 2"/>

你甚至可以用它探索 Task Parallel Library 中封装的算法：

```csharp
[Fact]
public void dig_into_the_algorithm()
{
    Parallel.For(0, 10, i => V.Start("task " + i).Sleep(i % 3 + 1).Finish());
}
```

你将得到类似下面的执行过程：

<img src="{{root_url}}/images/blog/visualize_workshop_3.png" alt="task visualization 3"/>

## 用神奇的图形抓住听众

说实话，上面的图表的截图无法表现出他的全部。这些图表不但是有一定交互性的，例如，将鼠标移动到某一个任务上，将显示这个任务的开始，结束，执行长度等信息；而且是可以标记的。怎么样，有趣吗？如果你也想试一试，请参考我的 [Workshop](https://github.com/lxconan/TplWorkshop)。如果你想看答案，请参考 lx_dev 这个分支。

我相信，只要我的自动化图表足够炫，你会有这种去下载代码的冲动。这是听众的主动行为，代表了他们的兴趣，有兴趣还有什么搞不定的呢。

现实中复杂的东西实在太多，所以能够理解这些问题的，我们会认为他们很厉害。但是这种厉害分为两种，第一种是他讲的我都听不懂，有种不明觉厉的感觉；第二种是，这么复杂的东西能够用简单的例子描述出来，真厉害。我喜欢第二种。那么在制作 Workshop 之前，我们是不是应当好好考虑一下如何让听众觉得即形象，又具备足够的吸引力呢？