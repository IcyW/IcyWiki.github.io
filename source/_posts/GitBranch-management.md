---
title: 关于持续交付中Git分支管理的思考
toc: true
date: 2020-09-23 07:36:58
tags:  DevOps
categories: DevOps
---



> Aim at always writing production-ready code.




# 一、背景
&nbsp; &nbsp; &nbsp; &nbsp; 提升研发效率（EP， Engineering Productivity），建立CI/CD体系，让持续自动化和持续监控贯穿于应用的整个生命周期已经成为有技术追求的攻城狮们的共识。
![CI/CD](http://myimg.icyhh.xyz/branch_management/1-0.png)
&nbsp; &nbsp; &nbsp; &nbsp; 持续交付是对整个软件交付模式的变革，涉及到的内容非常多、非常广，在这个模型中大概有二十多个关键点。根据[EPC（Engineering Productivity Certification） 1.0]的要求，需要覆盖到的有以下12个维度：

    0、需求协作管理	
    1、配置管理	
    2、制品库管理	
    3、分支管理	
    4、代码质量管理	
    5、测试管理	
    6、持续集成管理	
    7、自动化测试	
    8、开发测试与测试环境管理	
    9、发布	
    10、架构	
    11、数据觉察
&nbsp; &nbsp; &nbsp; &nbsp; 虽然距离这些概念的提出已经有段时间了，对相关实践如何落地，大家大多处于探索、转变的阶段。经过这段时间的宅家分析与痛点沟通之后，我整理了一些想法，针对模型中的「分支管理」这一维度聚焦来谈谈。



# 二、现状分析
&nbsp; &nbsp; &nbsp; &nbsp; 我们所对接的业务方产品形态十分多样，从移动端iOS、Android，到微信小程序、H5页面，再到PC web等等，每个开发团队都在历史的长河中形成了各自不同的开发习惯和分支管理策略。

&nbsp; &nbsp; &nbsp; &nbsp; 以其中一个典型的项目为例，我调用了开放API简单分析了一下项目（已脱敏）当前的分支状况。

## 分支生存周期统计
### 壹
&nbsp; &nbsp; &nbsp; &nbsp; 首先我拉取了项目中所有分支的信息，简单画出它们从被创建（begin_time）到销毁（delete_time）总共存在了多长时间；如果还未销毁，就计算到今天为止。
![begin2delete](http://myimg.icyhh.xyz/branch_management/2-0.png)

- 虽然是新建不久的项目，但是分支已经有182个，其中有五个存在超过了100天。还有31%的分支超过了一个月。

![begin2delete_cal](http://myimg.icyhh.xyz/branch_management/2-1.png)

- 说不定只是因为**没有约定好删除分支的规范**，而非真的有3成的需求开发时间超过一个月呢？

### 贰
&nbsp; &nbsp; &nbsp; &nbsp; 所以假设**不再有提交的分支其实是可以被废弃的**。我又计算了一下分支从被创建（begin_time）到销毁（last_push_time）总共存在了多长时间。

![begin2last_push_cal](http://myimg.icyhh.xyz/branch_management/2-3.png)

- 虽然分支的使用时间的确是缩短了一些，可是只有40%的分支存在天数小于一周，剩下的长周期分支中依然有2成**存在时间大于一个月**。

### 叁
&nbsp; &nbsp; &nbsp; &nbsp; 接下来再仔细看看这些「超长周期」的分支是什么情况。

![long_time](http://myimg.icyhh.xyz/branch_management/2-4-good.png)

- 在这个项目中，tiyan分支是作为类似发布分支的存在，master分支退居二线做开发使用，而test分支存放的是**隔离开的单元测试和接口测试**等等代码；后续存在了超过五十天的大部分是**个人使用**的bugfix分支。
- 那么问题来了。诸位看官可以思考一下 ① 测试代码不是同源同管理会有什么弊端 ② 长期存在多个分支、没有限制更新与合入时间会有什么弊端。

### 肆
&nbsp; &nbsp; &nbsp; &nbsp; 大舒一口气，从分支角度看，至少我们的需求大部分并不是真的要开发数月才能上线，而是有机会做到小步快跑的？那么「存在周期中等长度」的分支又是什么情况呢。
![middle_time](http://myimg.icyhh.xyz/branch_management/2-5.png)

- 开发周期大于10天的需求依然不少，是否因为**需求拆解**不够小？或是**开发框架组件化**解耦得不够细？还是**自动化测试**的基建不够好呢？
- 除此之外还暴露出了另一个问题，分支的**命名格式**也太多样了吧。特别是经历过**项目交接**之后，不同团队中的不同个人都以各自的习惯提交？更不用说**git commit的规范**了，不方便回溯。

### 伍
&nbsp; &nbsp; &nbsp; &nbsp; 存在周期长的分支问题暴露了这么多，最后剩下的「较短周期」的分支应该总没问题了吧？

![short_time](http://myimg.icyhh.xyz/branch_management/2-6.png)

- 通过分支名统计了一下存在天数在十天以下的分支的用处，除了**命名不规范**没有被我统计到的以外，用于fix作用的分支与feature分支各占一半，比较符合预期。

# 三、分支管理策略
&nbsp; &nbsp; &nbsp; &nbsp; 经过上述的简单统计后，大家可能只是对该案例项目的分支之多、生存周期之长有深刻印象。那么在**EPC标准**中，对于分支管理道理有什么要求，在《持续交付2.0》中对于分支管理又有什么建议呢。


## 3.1分支管理 - EPC等级
![EPC](http://myimg.icyhh.xyz/branch_management/3-0-cut.png)
&nbsp; &nbsp; &nbsp; &nbsp; 将[EPC 1.0]要求列出来后可以发现，在分支管理这个维度上，最终目标（Level 4）倒也算清晰简洁；除了要求分支要统一的命名规范外，量化指标就俩： 

**1. 分支存在的周期要短，80%的分支<=2天， **

**2. 每次提交的内容要少，80%的提交 < 200行代码**（除客户端UI的布局文件和相关配置文件外）。

&nbsp; &nbsp; &nbsp; &nbsp; 当然，要在业务需求的开发过程中实现这两点目标，却不是一件容易的事情。

## 3.2持续交付 - 分支管理模式
&nbsp; &nbsp; &nbsp; &nbsp; 除了上述提及的两个分数要求外，大家应该也注意到了「分支开发，主干集成」的模式，悄悄地转变为了「主干开发，分支集成」的模式。
&nbsp; &nbsp; &nbsp; &nbsp; 而最理想化的状态竟是「主干开发，主干集成」。一开始看到这个理念我也很不解，一旦抛弃了分支的存在，强大如[Git所拥有的必杀技](https://www.runoob.com/git/git-branch.html)不就被禁足，一日被打回原型，与其他版本控制系统无异了吗？

![git branch](http://myimg.icyhh.xyz/branch_management/3-1.png)

&nbsp; &nbsp; &nbsp; &nbsp; 且慢，**这个「主干开发，主干集成」的理想国故事，还要从「主干开发，主干集成」的伊甸园开始说起。**


#### 3.2.1 初级「主干开发，主干集成」
![321-a](http://myimg.icyhh.xyz/branch_management/3-21a.png)

&nbsp; &nbsp; &nbsp; &nbsp; 摘录自乔老师在《持续集成2.0》中的描述。很好理解，其实个人开发或是2、3个人的小团队，需求不多时间不紧时，往往大家就是这样直接在master上修改，git提供的就是纯粹的代码备份服务。

![321-b](http://myimg.icyhh.xyz/branch_management/3-21b.png)

&nbsp; &nbsp; &nbsp; &nbsp; 如果master代码处于长期不可用状态，只有等到所有功能开发完后才进行联调和集成测试，这就是「低频交付」的模式。

#### 3.2.2 「分支开发，主干集成」
![322-a特性分支模式](http://myimg.icyhh.xyz/branch_management/3-22a.png)

&nbsp; &nbsp; &nbsp; &nbsp; 当一个发布周期中的需求逐渐多了起来，需要合作的开发同学越来越多，新老同学的技术水平参差不齐的时候，就会发现Git的分支模型非常稳妥地提供了一种并行开发的解决方案，安全有保障、协作无干扰，这也是「特性分支模式」为什么备受青睐，广为业务团队所接受的原因（之一）。
![322-b](http://myimg.icyhh.xyz/branch_management/3-22b.png)

&nbsp; &nbsp; &nbsp; &nbsp; 若是更大规模的团队（40人以上）共同开发一款产品，就更倾向于运用如上的分支开发模式。

![322-c](http://myimg.icyhh.xyz/branch_management/3-22c.png)

&nbsp; &nbsp; &nbsp; &nbsp; 总的来说这个模式挺好的，唯一的弊端可能出现在合并的时候，一旦多个需求的修改有冲突，就会比较费神。
&nbsp; &nbsp; &nbsp; &nbsp; 若要成功使用这种模式，其关键点在于：

**1. 让主干尽可能一直保持着在可发布状态； **

**2. 每个分支的生命周期尽可能短； **

**3. 主干代码尽早与分支同步； **

**4. 一切以主干代码为准，尽可能不要在各特性分支之间合并代码。**

&nbsp; &nbsp; &nbsp; &nbsp; 但是分支开发模式，其实从本质上就是与持续集成的理念相互冲突的。持续集成是希望每次修改都尽早的提交到主干，主干总是处于最完整和最新的可用状态，充分验证后就可以用它来进行生产部署。而使用分支开发模式时，由于无法及时合并到主干，那么**时间越长与主干差别越大，风险就越高，最终合并的时候就越痛苦**。所以持续交付不推荐使用分支开发的模式[Ref. 3]。

#### 3.2.3 「主干开发，分支集成」
![323-a](http://myimg.icyhh.xyz/branch_management/3-23a.png)
&nbsp; &nbsp; &nbsp; &nbsp; 来到发布前的集成测试节点了，功能已经全部开发完毕，通常这时候客户端团队就会从代码中拉出「发布」分支。

![323-b](http://myimg.icyhh.xyz/branch_management/3-23b.png)

&nbsp; &nbsp; &nbsp; &nbsp; 模型中所示的“质量打磨周期”（Branch Stabilization Time）越短，说明主干质量越好，当打磨周期极短时，就可以转换到高频的「主干开发，主干集成」模式。

![323-c](http://myimg.icyhh.xyz/branch_management/3-23c.png)

#### 3.2.4 高级「主干开发，主干集成」
![321-a](http://myimg.icyhh.xyz/branch_management/3-21a.png)

&nbsp; &nbsp; &nbsp; &nbsp; 是的，与3.2.1是同一张图，但是看到这里，相信大家应当对为何「主干开发，主干集成」是最低级的模式同时又是最高级的模式有所理解了。相比于「低频交付」的状态，「高频交付」状态下对代码质量的要求相当高，几乎是落子无bug的境界（笑）。
&nbsp; &nbsp; &nbsp; &nbsp; 需要进行版本控制的**不仅是源代码**，还有测试代码、数据库脚本、构建和部署脚本、依赖的库文件等，并且对构建产物的版本控制也同样重要。只有这些内容都纳入版本控制了，才能够确保所有的开发、测试、运维活动能够正常开展，系统能够被完整的搭建。持续交付建议的方式是频繁的提交代码，并且最好工作在主干上，这样一来**修改对所有项目成员都快速可见**，然后通过持续集成的机制，对修改触发快速的自动化验证和反馈，再往后如果能通过各种维度的验证测试，最终将成为潜在可发布和部署到生产环境的中版本[Ref. 3]。
&nbsp; &nbsp; &nbsp; &nbsp; 深入了解持续交付中对于分支管理的要求或者说期许之后，希望没有打击到大家的信心 / 希望反而能**激起大家的技术追求**。那么下一篇章就来谈谈一些近期搜刮到的/实用的/接地气的辅助方案了。


# 四、细节建议方案
&nbsp; &nbsp; &nbsp; &nbsp;个人认为， 上述提到的向「主干开发，主干集成」靠拢的模式思路更适用于**发布频率固定且快、质量要求高**的处于快速生长期的客户端（e.g. iOS、Android）/类客户端（e.g. 小程序）产品。不同的产品形态、不同的产品周期还是要因地制宜地选择适合当前发展状态的分支管理模式。比如对于嵌入在APP不同位置的H5页面，因为相互独立故而可以选择建立分别的仓库直接采用主干开发的方式。
&nbsp; &nbsp; &nbsp; &nbsp;有些场景下，迫不得已要采用分支开发的模式，比如并行需求太多且相互干扰，或者在需求开发的同时有大块的重构工作要做，或者针对特定的用户开发特殊的功能，以及需要进行与主线无关的试验等等。这时拉出分支其实意味着已经在持续集成/持续交付上做出了妥协，那么我们建议至少要使用一些折中的方案[Ref. 3]：

1. **尽量缩短分支的周期，最长也不要超过迭代周期；**
2. **每个分支上运行单独的测试流水线，保证质量。虽然这种方式浪费资源，而且其实也没进行”真正的“集成；**
3. **分支只与主干合并代码，分支彼此之间尽量不做合并；**
4. **分支定期合并主干上的变更；**

## 【重点】针对问题项目的分支管理改进

&nbsp; &nbsp; &nbsp; &nbsp;针对第二节中分析的典型项目，我想可以有如下的改进方式：

![4-0](http://myimg.icyhh.xyz/branch_management/4-0.png)

1. 按照上图所示的分支模式进行管理，除了主干与发布分支以外，其他不必要的分支均删去，减少与主干产生大差异的机会；
2. **测试代码**、数据库脚本、构建和部署脚本、依赖的库文件等等合入主干与源代码同源管理；
3. 分支与需求绑定起来，使得每一次的修改有据可循（可参考4.1）；
4. 约定特性分支的命名规范（比如feat_20200229_market，表示该特性分支最多存在到二月29日就要删去，功能是market），可以通过插件约束不规范分支的提交（可参考4.3）；
5. 遵循git commit的提交信息规范，限制不合规范的messages的提交（可参考4.2）；
6. 除非特殊需要，所有特性分支的存在周期都尽量压缩到五天以内；持续暗示自己分支不过周末。

## 4.1 分支与需求单
&nbsp; &nbsp; &nbsp; &nbsp; 在3.2.2的模式中，每一个特性分支的创建都是为需求服务的。要想达到每个分支都在很短时间内消失的目标，不可否认前提条件是**产品对需求的拆解**和**开发对代码的解耦**都具备很高的能力，这是值得另开篇章阐述的话题，此处先留个#href的坑。
&nbsp; &nbsp; &nbsp; &nbsp; 为了解决当前分支凌乱的问题，有一种办法是在需求单**转入开发中时自动创建分支**，git commit时提交关键字**与需求ID绑定**起来，不仅可以追溯每一次代码的变更都为了达成什么目的，划分模块责任人，更可以在git push的同时**一键转单评论**，繁琐的流程和鼠标点点点操作通通不存在。
&nbsp; &nbsp; &nbsp; &nbsp;  参考《TAPD（腾讯敏捷产品研发平台）-工蜂Git关联新特性》，只需三步，轻松上手。

 1. TAPD项目下启用「源码」应用，应用设置中关联相应的GIT仓库。
 2. 需求单与GIT分支关联。
 3. Smart Commit关键字提交时触发状态流转。

效果示例：
```bash
     $ git commit -m "fix: test --story=857210425 TAPD vs GIT #finish"
     $ git push
```

![4-1](http://myimg.icyhh.xyz/branch_management/4-1-mas.png)   

&nbsp; &nbsp; &nbsp; &nbsp; 分支关联需求单，commit关联需求提测 / bug解决，状态流转一步到位。

## 4.2. 代码提交规范  
&nbsp; &nbsp; &nbsp; &nbsp;编写规范良好的commit messages的优点无需赘言。不仅提高后续code review的效率，帮助团队成员清晰地了解到每一次提交修改了什么特性，也方便了版本日志的生成。
&nbsp; &nbsp; &nbsp; &nbsp;业界最常用的[Angular规范](https://www.ruanyifeng.com/blog/2016/01/commit_message_change_log.html)如下（只展示Header要求）：

    <type>[optional scope]: <description>


&nbsp; &nbsp; &nbsp; &nbsp; Header部分只有一行，其中`<type>`用于说明 commit 的类别，只允许使用下面7个标识：

    feat：新功能（feature）
    fix：修补bug
    docs：文档（documentation）
    style： 格式（不影响代码运行的变动）
    refactor：重构（即不是新增功能，也不是修改bug的代码变动）
    test：增加测试
    chore：构建过程或辅助工具的变动

&nbsp; &nbsp; &nbsp; &nbsp; `<scope>`用于说明 commit 影响的范围，比如数据层、控制层、视图层等等，视项目不同而不同。是可选项。
&nbsp; &nbsp; &nbsp; &nbsp; `<description>`是 commit 目的的简短描述，不超过50个字符。

## 4.3. 辅助工具

&nbsp; &nbsp; &nbsp; &nbsp; 无论是何种规范的制定，最终目的都是服务于研发流程的提效，如果它的存在阻碍了开发的丝滑进行，那它就不是一个好的规范。
&nbsp; &nbsp; &nbsp; &nbsp; 所以我们需要一些自动化的提示和约束，来方便标准化规范的落实。

### 4.3.1 分支命名规范
&nbsp; &nbsp; &nbsp; &nbsp; 参考《Feflow在CI中检查项目Git规范》提供的前端方案，feflow-plugin-check插件（后续可能会对外开源：https://github.com/iv-web/feflow）:



    * 分支版本命名规则：分支类型_分支发布时间_分支功能。比如：feature_20170401_fairy_flower
    * 分支类型包括：feature、 bugfix、refactor三种类型，即新功能开发、bug修复和代码重构
    * 时间使用年月日进行命名，不足2位补0
    * 分支功能命名使用snake case命名法，即下划线命名。

**效果示例：**
![4-3-1](http://myimg.icyhh.xyz/branch_management/4-2-mas.png)

### 4.3.2 关联tapd并规范commit
参考《优雅提交git commit并强制关联tapd需求单》提供的方案（后续可能会对外开源），只需两步：
（1）添加package.json 和 commitlint.config.js 文件。
（2）执行tnpm install

**效果示例：**
a) 没有输入正确的tapd关联
![没有输入正确的tapd关联](http://myimg.icyhh.xyz/branch_management/4-4a.png)
b) 没有输入正确的Angular规范
![没有输入正确的Angular规范](http://myimg.icyhh.xyz/branch_management/4-4b.png)
c) 符合要求的commit message
![符合要求的commit message](http://myimg.icyhh.xyz/branch_management/4-4c.png)



# 结语
&nbsp; &nbsp; &nbsp; &nbsp; 良好的分支管理只是实现持续交付持续部署的其中一个必不可少的环节，要让DevOps更丝滑，研发流程效率更高，还需要产品-开发-测试同学更加专业化、更加规范化地一同努力。

# 附录

### 持续交付中的分支管理
Ref. 1 [乔梁-《持续交付2.0业务引领的DevOps精要》](http://www.read678.com/JdBook/index/6589)

Ref. 2 [{内部}持续交付体系设计与实践 - 工程效率提升之路](http://km.oa.com/group/11831/articles/show/299416)

Ref. 3 [百度资深敏捷教练：深度解析持续交付之全面配置管理](https://mp.weixin.qq.com/s?__biz=MzI4NTA1MDEwNg==&mid=2650757465&idx=1&sn=ec6c9c4aee13deaecf48619ea0c5cd95&chksm=f3f9ecccc48e65dac50638240f9ee6d6af1648b00d8e6baa84f446075809defed0f74e8ce0da&mpshare=1&scene=23&srcid=0211hJxP8eYjCTcPmENhn0Uy%23rd)

Ref. 4 [化繁为简的企业级 Git 管理实战（三）：分支管理策略](https://www.hahack.com/work/enterprise-class-git-version-control-3/)

### Git分支规范好工具

Ref. 5 [{内部}Git commit message和工作流规范](http://km.oa.com/group/29185/articles/show/295984)

Ref. 6 [{内部}优雅提交git commit并强制关联tapd需求单](http://km.oa.com/group/502/articles/show/398772)

Ref. 7 [Git Commit 规范 | Feflow](https://feflowjs.com/zh/guide/rule-git-commit.html)

Ref. 8  [{内部}使用Feflow在CI中检查项目Git规范](http://km.oa.com/group/33258/articles/show/401115)

