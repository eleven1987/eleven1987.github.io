---
layout: post
title: "空难与 IT 系统的设计和维护 (上)"
date: 2014-09-28 12:35:28 +0800
comments: true
categories: design maintenance
---

最近观看了总共十三季的 __[Air crash investigation](http://en.wikipedia.org/wiki/Air_Crash_Investigation)__，感到 IT 研发和运维的就像是这些场景的重现。事实上，我们做的许多 IT 系统的复杂程度都接近甚至超过了飞机。因为我们要维护比飞机多得多的内部状态，只不过飞机对稳定性和可靠性的要求比一般 IT 系统要多得多。前人的经验非常值得我们学习。如果你没有那么多时间全部看完，那么请看以下的吐槽（有些案例，例如 __法国航空8969号，1994__ 并未过多涉及行业经验故而略过），挑选自己感兴趣的看一看吧。

<!--more-->

### 第一季

__美国航空1420号.1999__：

* 计划在执行时应该根据合理情况变化调整，尤其是在遇到不可抗力的时候。强硬的设定 deadline 通过提高强度（加班）或是强硬执行可能造成更加悲剧的结果。；
* 一个人同时执行的任务越多（不断增添新的任务，或者把一个人同时安排在两个团队里混合工作），犯错误的机会也越大。；
* 后端（在公司支持的团队）如果发现任何能够威胁 on-site team （在客户现场工作的团队）工作的情况，必须及时告知，这个时候 delay 越大，风险也就越大；
* 如果一个环节的执行对于整个任务是 critical 的，做 checklist，并严格按照执行，人脑在压力面前往往会忘掉一些事情；

__联合航空811号.1989__：

* 设计瑕疵，或者技术债。必须尽早修正。否则原本相对次要的问题可能在将来引发严重的后果；
* 作为技术人员，要保证的是产品的质量符合要求；作为公司，很多时候关心的是怎么赚钱。要认识到这两者的区别。有些问题可以通过政治方式得到一时的缓解但是并不能消除技术上缺陷导致的风险；

__越洋航空236号.2001__：

* 不接受正常的维护流程而走捷径（例如，直接替换某些组件而放弃部署流程）更容易由于依赖不匹配造成严重的问题。而在这种问题上，从经济角度出发的部门或人员，更倾向于建议你放弃正常维护流程；
* 不可能所有的情况有据（手册或者文档）可查，一些特殊的情况必须进行必要的推理做出正确判断。为了做出正确的判断，最好的方式就是理解系统的构造和不断的训练；
* 在发布 patch 的时候，确保该 patch 能够正确的获得所有依赖；

__瑞士航空111号.1998__：

* 锦上添花的功能也可以对系统带来毁灭性的打击，添加功能需谨慎；
* 设计一个系统时要注意具有相似控制行为的操作应具有一致性，否则非常容易造成操作者出现混淆而犯错；
* 对于一些危险的系统行为（例如，造成错误数据），应当立即进行必要的措施而非期望能够获得好运气而挺过去。心存侥幸的行为可能造成严重后果；
* 通过测试和真实的系统行为是两回事儿，必须在真实环境下对系统进行验证（只有单元测试是不够的）；
* 虽然我们能够通过日志记录的原始数据推断系统的行为，但是最好也能够直接询问客户的使用体验以相互印证；

__秘鲁航空603号.1996__：

* 维护过程必须有明确的 instruction 与监督措施；
* 惊天动地的系统问题往往是由一个显而易见的小问题造成的。这种问题往往不具备清晰的逻辑性，并且极有可能造成异常参数爆发性出现。查阅最近的维护记录往往能够发现解决问题的方案；
* 别以为系统维护人员和你（研发团队）一样了解这个系统。系统设计要从使用和维护的角度出发。尤其对于危险的操作，应该予以充分的警示和简单的操作（操作步骤越复杂，出错的可能性越高）以避免灾难发生；

__阿拉斯加航空261号.2000__：

* 系统的设计要考虑到 fail-safe 的问题，阻止问题继续蔓延避免使整个系统处于无法恢复的状态；
* 系统的维护是一个严肃的问题，侥幸心理（这一项不检查应该也没有问题）往往会造成严重的问题。维护费用和整个系统失效比起来显得微不足道；
* 系统的关键部分最好考虑冗余处理防止系统整体性失效；

### 第二季

__英国航空5390号.1990__：

* 现代 IT 系统的规模已经超出了一个人的理解能力。因此即使是技术专家也不可能了解系统的各个方面。应该积极的吸收在特定方面更有经验的人士的意见；
* 通过确切的数据而不是 __感觉__ 来确保组件的版本号或者编号和预期相匹配；
* 维护人员往往以不按照 checklist 照本宣科为荣，这是非常有害的。而更深层次的问题是，如果他们严格按照 checklist 进行工作，可能会严重降(延)低(长) _效(时)率(间)_，而影响其 KPI，而他们也会乐于将这种经验传递给新人；

__大西洋东南航空529号.1995__：

* 设计瑕疵，或者技术债。必须尽早修正。等到引起严重的系统问题时，往往为时已晚；
* 维护的目的是使系统能够正常的工作而并不是使系统 __看上去__ 是好的；

__乌伯林根空难.2002__：

* 不同系统间遵守一致的标准是非常重要的（典型的如时区，度量单位），否则将发生冲突，从而导致问题；
* 不要将多个任务同时加在一个人身上，这往往造成问题。当问题出现时，这个人将成为替罪羊，但是究其根源，则是管理的问题；
* 计划的制定者往往并不在一线，不了解一线的工作方式，在这个时候，技术人员应该对计划进行制约。否则不科学的计划可能会引发严重后果；

__美国航空965号.1995__：

* 在沟通的过程中，使用简洁而准确的话语避免双方理解不一致，最好能够复述并请对方确认；
* 不要想当然的认为系统应该是怎么工作的，在执行重要操作之前，检查所有选项是否和期望一致；
* 确保在任务之前做好准备，避免在短时间内处理大量事务而增加出错的几率；

__哥伦比亚航空052号.1990__：

* 简单粗暴的对执行层施加压力可能会导致不计后果的行为出现，对于任务的成功没有任何好处；
* 在进行任务交接或双方沟通的时候，要完整的交代上下文和目标，__尤其是__ 将目前存在的问题交代清楚；
* 永远准备一套备用方案，当预期方案难以实施时果断采用备用方案避免事态恶化进入进退两难的境地；

### 第三季

__阿罗哈航空243号.1988__：

* 系统维护是一个严肃的工作，但是为了经济上的利益，往往在这个上面打折扣；

__货机惊魂.2003__：

* 对于系统的关键部分进行冗余设计以防止系统全面失效；

__日本航空123号.1985__：

* 仍然是维护的问题，这次仍然是不按照手册进行维护造成了严重的后果；

__联邦快递705号.1994__：

* 不患寡而患不均，不患贫而患不安。盖均无贫，和无寡，安无倾。对于 IT 人员来说，公正的评判对其有非常重要的意义；
* 让心怀怨念的员工去 on-site 简直就是一场灾难；

__菲律宾航空434号.1994__：

* 安全系统的设计难处在于只要有一处漏洞整个防御体系将变得一文不值。而一种新的攻击形式在于该攻击是由多个步骤组成的，但是每一个步骤都是合法的操作；
* 无视上有公司发布的安全警报，不及时升级系统，安装相应的安全更新会使整个系统陷入安全风险中；

__布里斯托航空56C号1995__：

* 随着技术的进步，一些老的标准和测试方法已经变得不再适用，此时应该及时更新测试手段，使测试与系统相匹配；
* 技术的进步解决了一些问题，但是也带来了更多的问题，在使用新技术前对其 pros 和 cons 都应该进行调查；

__俄罗斯航空593号.1994__：

* 严禁非相关人士对产品系统进行操作；
* 当系统发生明显的状态改变，或者系统认为该状态改变可能引起重大问题时，应该以多种明确的方式通知操作人员，确保操作人员收到这些信息；

### 第四季

__法国航空358号，2005__：

* 在安全设计上，应该坚持设计标准，任何安全设计的裁剪都可能造成严重的问题；
* 系统日志的反馈对于维护人员来说非常重要，因此系统日志尽可能的包含多的上下文以便维护人员正确的判断系统的状态；
* 技术人员过度疲劳会使出错的几率大大增加；

__英国航空009号班机，1982__：

* 对于环境原因引发的系统问题，应该进行详细记录。因为这些问题非常难以重现，完备的记录将为以后的维护提供宝贵的经验；
* 沟通，尤其是对于 on-site team 来说，是非常重要的，当出现严重的系统问题的时候，队员之间的沟通能够综合更多的信息找到解决方案；
 
__加拿大航空797号，1983__：

* 对于严重的系统异常，如果能够及时进行报告则再好不过了。这种报告形式应该是主动式的，例如电话，短信或者邮件。
* 应当为维护人员提供可靠的工具以协助其工作。

__大韩航空801号，1997__：

* 当用户可用性和系统安全有冲突时，应该将系统的安全可靠放在第一位。
* 过期的文档非常有害。确保on site team的操作手册与系统匹配。
* 系统集成测试以及部署演练应当尽量模拟客户的环境。
* 不应该过度依赖自动化，自动化可能带来严重的知识传递丢失。从而在自动化手段失效或出现故障时束手无策。
* 在工具上的投资是非常值得的。
* 检查越完备，使用体验可能越差，导致人员故意忽略或跳过检查。对此应当坚决制止。另外，忽略流程而走捷径对系统的威胁是巨大的。

__波音737连环事故，1991-1994-1996__：

* 系统表现在极端情况下和低压力情况下可能截然不同。在不同的压力下对系统的可靠性进行测试是必要的。

__中华航空 006号，1985__

* 在相关人员进行一项工作的中间打断他去做另一项工作可以大大增加其犯错误的机会。例如，打断一个正在编码之中的程序员。
* 系统异常的情况下，以往所做的一切自动化方案都可能失效，因为那些方案是在假定系统行为正常的情况下制定的。另外，为了系统安全，自动化手段往往只能够控制系统的一部分功能，而这些有限的功能可能不足以让系统恢复正常。此时需要的是人为介入。
* 系统自动化程度越高，了解系统构造和行为的人就越少，出事故的时候就越不知道怎么处理。
* 一个团队内有各种不同的人员，每一个人都有其独特的能力，作为 Leader，专心做好自己该做好的事情而不是去插手不该去管的事情（除非明确提出请求）。
* 当系统发生问题的时候，第一个应该想到的应该是系统的安全，例如，保证系统的数据不被破坏，或者错误不再蔓延；忽略大前提而直接去解决这个错误可能造成严重的后果。
* 将整个系统的概要状态可视化非常必要，以便在解决问题的过程中掌握系统概况。防止只见树木不见森林的情况出现。
* 系统日志显示出很离谱的数据的时候，大部分不是因为记录错误，而是反映了系统的真实状态。
* 该睡觉的时间没有办法好好写代码。

__墨西哥航空498号，1986__：

* 确保每一个异常情况都有相应的日志记录。
* 自动化工具对于提升效率与避免错误有很大好处。使用并及时升级它们。

__美国空军 IFO 21号，1996__：

* 对于积累更新系统（例如 feed，或者 transactional replication），更新失误或不正确的数据可使系统逐渐偏离正确状态。应该考虑另外一套无状态系统进行状态核对。
* 在的部署之前，详细了解系统运行的环境配置以便选择正确的方案和工具。这些方案和工具应该尽可能稳妥，避免冒险的方案。
* 如果你所使用的服务提供商，例如云平台，他们的运营很吃紧，那么不要使用他们的服务。
* 没有问题往往指的是暂时没出问题，并不意味着从逻辑上它是没有问题的。我们的单元测试往往意味着前者。

__闪光航空604号，2004__：

* 即使是资浅人员，也有必要在认为情况不正确而相关资深人员并没有做出反应时提出质疑或接管操作。
* 在结对或多人合作时，开对方的玩笑可能对合作带来负面影响，对新人的影响尤为突出。有可能阻止对方发表自己的意见。
* 关键性的信息，例如日志，文档必须进入管理并留有备份。

__太阳神航空522号，2005__：

* 任何系统测试或试验性质的配置更改都必须保证在结束后恢复到原本的状态。当配置较多时，应使用自动化手段。
* 对于不同类型的错误必须明确区分。确实不能区分的，应该加以记载以便日后问题排查。
* 对于危险或高级别操作，必须在ui上进行明确区分。避免忽略重要信息。

### 第五季

__中西航空5481号，2003__：

* 过期的文档和经验有可能对系统造成重大影响。保持关键文档 up to date，并时时更新自己的相关知识。建立过期机制，在使用时间过长的结论之前确保其有效性。

__美国航空96号，1972；土耳其航空981号，1974__

* 无法忍受设计延迟可能的结果是更容易出现严重事故，但是忍受可能让自己一时处于商业竞争的下风。其实前者自然更重要，只要一发生，消费者无法再对你建立信心。但是现实是，催催催，催你妹啊催。
* 在已经知道系统有重大缺陷的情况下，无视安全强制系统上线也是饮鸩止渴的行为。但是，往往是，催催催，催你妹啊催。
* 不建卡就指望着有人把这个缺陷修了是不可能的，就算你指定了人员，他也会忘，你也会忘，于是这个问题就忘了。
* 如果系统的某个指示是间接的（根据其他数据推算出来的），但是我们并不能肯定这种推算是否合理，那么还不如将数据透明化的进行展示，让人去决断。
* 别信你的云运营商会给你的数据做冗余，会保证安全，会建立异地的数据中心，直到有一天出了严重的事故为止。

__加拿大航空 143 号，1983__

* 在系统集成时需要对数据的单位达成一致，例如时区，时间长度等等。在进行变量命名时明确单位，例如 seconds，minutes 等等。

__德尔塔航空 191 号，1985__：

* 听清全部的信息之后再做决定，而不是在说话中就打断别人。这是我等最容易犯的毛病。

__南方航空 242 号，1977__：

* 间接的数据有时候比透明的直接数据更具有直观性，但是间接数据在超出设计预期的情况下并不能保证其结论的正确性。
* 又是过期信息惹的祸。这些信息对 on site team 的影响是非常大的。

__南非航空 295 号，1987__：

* 系统的容错设计应当考虑到系统已经处于异常，否则按照正常运行的系统进行设计可能都会导致该设计失效；

__印度航空 182 号，1985__：

* 当客户朝你发脾气的时候，你该干啥就干啥。不要违背流程，不要被他的态度改变。

__伯根航空301号，1996__：

* 在系统异常的情况时，任何之前设定的自动任务都可能失效，因为他并不是为异常情况设计的。此时应该由运维人员介入，而不是让自动机制自动运行。在 IT 系统中，这些自动系统往往是诸如任务调度，邮件发送等附加系统。
* 资历和是解决方案是否正确没有必然的联系，即使是资历较浅的人员在发现 pair 不能够正确的听取和采取正确的措施时也应当采取措施甚至接管控制权，使 pair 的 driver 和 observer 角色变换。
* 当系统上线出现问题的时候，即使接近 dead line 也需要评估问题的影响。当足以影响系统安全（例如，数据正确性），必须进行推迟。
* 运维的关键是按流程操作，这很重要，虽然可能会很枯燥。

__东方航空 401 号，1972__：

* pair 中一个是 driver 另一个是 observer 是有原因的。如果两个人并不能分清角色而扎堆可能会忽略重要的信息。同时不让一个人大权独揽是降低人为失误的关键。
* 在系统有重要的状态切换的时候，应当进行明确的提示，这种提示应该分为几个部分第一是明确的界面提示，第二是邮件提示，而第三是系统日志；

__卓越航空 N600 号，戈尔航空 1907 号空中相撞，2006__：

* 当系统出现警告时或关键的状态改变时，确保在 UI （通过颜色和大小等多种维度，而非一种）或声音或以其他明显的方式明确进行指示，并通过邮件或日志记录；
* 指责他人会造成无人挺身而出坦承失误的文化，而一旦企图隐瞒事实一切都会被粉饰太平，不久还是会重蹈覆辙；

_To be continued..._