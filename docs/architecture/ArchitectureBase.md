# 方法论

## 架构

不同架构的合理性和美感是不一样的

> 组成派：
>
> - 软件系统的架构将系统描述为计算机组件以及组件之间的交互
> - 关注架构设计活动中的课题--软件

> 决策派：
>
> - 软件架构是在一些重要方面所作出的决策的集合。
> - 软件架构包含：
>   - 软件系统的组织  --  康威定律
>   - 选择组成系统的结构元素和它们之间的接口，以及这些元素相互协作时所体现的行为
>   - 如何组合这些元素，使它们组件合称为更大的子系统
>   - 用于指导这个系统的架构风格：元素、接口、协作和组合
>   - 软件架构并不仅仅注重软件本身的结构和行为，还注重其他特性，例如：易用性，可靠性
> - 关注架构设计活动中的主体 -- 人

架构定义的主流观点：

- IEEE/ANSI：体系架构是以组件、组件之间的关系、组件和环境之间的关系为内容的某一系统的基本组织结构，以及指导上述内容设计与演化的原则。 不仅强调系统的基本组成，还强调了体系结构的环境既与外界的交互。
- 软件设计过程中，超越计算中的算法设计和数据结构设计的一个层次。体系结构问题包括各个方面的组织和全局控制结构、通信协议、同步、数据存储，给设计元素分配特定功能，设计元素的组织，贵么和性能，在各个设计方案之间进行选择。
- 软件体系结构 = 组件（Component） +  连接件（connector）+ 约束（constraint），其中组件可以是一组代码，一个模块，或是一个完整的程序。

------

### 什么是软件架构

- 所在领域为软件
- 每个软件系统都有架构
- **定义了组件、组件间的交互、约束这三者间关系，以及设计原则和目标等**
- 忽略了组件中与组件之间与交互无关的内容
- 描述了软件的组件和结构背后的基本原则和主要流程
- **不存在普适的最后架构，只有适合某些场景的最优架构**
- 是可以逐步优化和演进的，不是一成不变的

### 软件架构的作用

- 软件架构是干系人相互交流的手段
  - 是一个系统的工程蓝图，指导软件系统的设计、开发、测试和维护等。
  - 架构作为一种常见的系统抽象，提供了一种公共语言，是各方理解系统、协商、达成共识的基础。
  - 每个干系人对架构的关心侧重点不同
  - 因为与很多人相关，架构不可能仅用一种视图表达清楚。
  - 架构内容不完整会妨碍干系人对系统的了解和沟通，从而增加项目失败的概率。
- 软件架构是最初设计决策的体现
  - 是功能需求，质量属性之间的权衡和折衷。
  - 架构设计体现了面向业务到面向技术的转换，同时微系统的实现设定了诸多约束，支撑或者阻碍质量属性的实现。
  - 架构是进行迭代开发和增量交付的基础。
  - 架构能提前预测系统功能、质量属性和竞争力。
  - 架构可以用于制导系统维护和功能增强。

- 软件架构是会影响项目团队的组织架构
  - 架构是基于组件开发的基础。
  - 架构层次会影响项目团队的组织结构。
  - 软件架构是可重用、可传递、可进化的，能够由此衍生出整个产品系列。

### 软件架构的要素

- 组件

  是系统架构的核心组成，一般是指一个功能相对完备且独立子系统或者模块。

- 连接器

  描述这些组件之间通讯的路径、通讯的机制、通讯的预期结果。

- 约束

  对象连接时的规则，或者组件连接的形式和条件。

### 软件架构的驱动因素

- 功能

  - 功能的实现需要各模块相互配合，形成模块协作链。
  - 高质量的架构需要支持完备的系统功能。
  - **识别和实现关键功能需求是架构师经验和能力的体现。**
  - 关键功能是对用户作用最大的需求，而不是设计模块最多的需求。

- 质量属性

  - 功能性，可靠性，易用性，效率，维护性，可移植性，可扩展性。
  - 功能是影响质量属性的核心/决定系统成败的是质量属性，满足质量属性往往比满足功能需求更为重要。
  - 不同利益相关者对质量属性的要求不相同。（ADD）

- 约束

  - 客户需求以及业务相关约束（行业标砖，政策法规，遗留系统）
  - 用户及使用环境约束 （用户特点，用户水平，早起版本的使用习惯）
  - 开发组织及开发环境的约束 （技术特点，组织分布，软件资产）

  <img src="C:\Users\lwx639077\AppData\Roaming\Typora\typora-user-images\image-20210714150654779.png" alt="image-20210714150654779" style="zoom:80%;" />

三大因素驱动的架构设计

影响架构的因素多而杂   =》  全面有序的理解需求  =》 确定关键功能，确定系统质量属性商业属性，确定约束条件

![image-20210714150903794](C:\Users\lwx639077\AppData\Roaming\Typora\typora-user-images\image-20210714150903794.png)



### 架构设计的本质

就是在功能需求、质量属性和约束之间进行权衡和折衷

- 对质量属性的高要求是有成本的
- 不可能以孤立的方式实现质量属性
  - 如何一个质量属性的实现都会对其他质量属性带来影响，在质量属性间系统的做好设计权衡至关重要
  - 架构师应对系统设计的质量属性进行权衡取舍，决定重点支持哪些质量属性需求、支撑的程度

------



## 软件架构的描述方法

### 软件架构需要描述到什么程度

- 不同背景，设计程度会有所不同。
- 应该为开发人员提供足够的指导和限制。
- 具体详细程度应该以：
  - 是否将架构的各个重要方面描述清楚。
  - 设计人员是否可依据此架构进行详细设计和子系统设计、模块儿设计。
  - 描述出来的架构能否确保不会出现歧义，不会导致不同的人有不同的见解。

### 软件架构的通病

- 高来高去。精细化设计比高层设计难度小得多，对经验和能力要求非常高。
- 缺乏重要的架构视图，遗漏某些重要视图，遗漏了了对团队某些角色的指导。
- 浅尝辄止，不能深入。将重大技术风险遗留到后续开发中。
- 名不副实的分层架构。对各层之间的交互接口、交互机制的设计严重不足。

### 软件架构的描述方法

- Architecture Cube

  Ties(物理层), Layers(逻辑层), Capabilities(能力层)

- SEI 3视图

  模块视图，组件-连接器视图，分配视图

- RUP 4+1 视图

  用例视图，逻辑视图，运行视图，开发视图，物理视图

  <img src="C:\Users\lwx639077\AppData\Roaming\Typora\typora-user-images\image-20210715142156632.png" alt="image-20210715142156632" style="zoom:67%;" />

- VXD 9视图

  一种质量属性方法，使用9个方面

  （生态视图，部署视图，运维视图）；（技术视图，开发视图）；（上下文视图）；（功能视图，数据视图，运行视图）

------

## 架构模式及模式应用

### 什么是架构模式

- 模式
  - 模式是为一个在特定设计环境中出现的设计问题提供的一个通用的解决方案。
  - 模式使用文档记录下现存的经过充分考研的设计经验。
  - 模式为设计原则提供一种公共的词汇和理解。
  - 模式是为软件体系结构建立文档的一种手段。
  - 模式支持用已定义的属性来构造软件。
- 模式的图式
  - 语境：引发一个设计问题的设计场景
  - 问题：语境中重复出现的强制条件集
  - 解决方案：用来平衡强制条件的配置
- 架构模式
  - 架构模式描述了一个出现在特定设计语境中的，通用的，再现的设计问题，并未它的解决方案提供了一个经过充分验证的通用图式。
  - 架构模式包括：(1)一组预先确定的子系统和它们的职责；(2)子系统间的相互关联规则

### 常用的架构模式

pattern-oriented software architecture

- 层
  - 分子任务组，放置于不同的层
  - 下层为上层提供服务
- 管道和过滤器
- 黑板
- 代理者
  - 服务都发布给代理，由代理转发
- 模型-视图-控制
- 标识-抽象-控制
- 微核
- 映像

另外说法：

![image-20210715144147656](C:\Users\lwx639077\AppData\Roaming\Typora\typora-user-images\image-20210715144147656.png)

### 架构模式 vs 设计模式

设计模式：偏重于编程和小模块实现

创建型模式，结构型模式，行为型模式



架构模式：偏重于架构方面设计

## 软件架构设计方法

区分识别主要矛盾，次要矛盾

方法包含：

- CDD
- FDD
- ADD
- MDD/MDA
- TDD
- POD
- OOD

## 架构师