# Unity - UIElements
[官方文档传送门](https://docs.unity3d.com/2019.1/Documentation/Manual/UIElements.html)
##### UIElements开发者指南
Unity编辑器用户界面主要是围绕即时模式UI系统构建的。虽然IMGUI在某些上下文中工作得很好，但它也有一些严重的设计限制，这些限制会影响开发编辑器特性和扩展的每个人的工作效率。
这就是创建UIElement的动机。UIElement是一个retained-mode(保留模式)的UI系统，它打开了提高性能的大门，提供了样式表、动态/上下文事件处理、可访问性和数据持久性。
UIElements中的许多概念都基于公认的Web技术。如果您熟悉XML、CSS、JQuery、HTML DOM和DOM事件系统，那么您可能已经熟悉了许多UIElements概念。
本指南的目标是通过描述框架背后的概念以及向您提供有关如何使用UIElements构建交互用户界面（UI）的清晰解释，帮助您利用UIElements。

##### 选择一个UI工具
Unity提供了三种用户接口(UI)工具。你应该基于你对以下问题的答案选择一个UI工具：
- 你是为游戏开发还是为编辑开发？
- 如果您正在开发一款游戏，用户界面是否会随游戏一起提供？

               | Runtime dev UI  | Runtime game UI  | Editor
IMGUI          | for debugging   | not recommmended | ✔
UGUI           | ✔               | ✔               | not available
UIElements     | 2019.x          | 2020.x           | 2019.1

UIElements已经准备好成为游戏和编辑器用户界面开发的首选工具包。

##### 免责声明

UIElements是一个实验特性：它是不完整的，并且会受到api中断更改的影响。UIElements仍在积极开发中。

此外，在2018.3中对UIElements的更改将不会返回到旧版本。如果升级，还必须从以前的Unity版本升级一些元素。