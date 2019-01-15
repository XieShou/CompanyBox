# Unity - UIElements
[官方文档传送门](https://docs.unity3d.com/2019.1/Documentation/Manual/UIElements.html)
[官方演示案例传送门](https://github.com/Unity-Technologies/UIElementsExamples)
---
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
---

### 简要说明
Unity的UIElements是Unity官方新开发的UI工具，旨在整合原先的IMGUI和UGUI，实现Unity引擎在UI内容开发的流程和方法上的统一。

大致可以分为3个部分：
- C#：
- UXML：
- USS：

