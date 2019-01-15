# Unity - UIElements

[官方文档传送门](https://docs.unity3d.com/2019.1/Documentation/Manual/UIElements.html)

[官方演示案例传送门](https://github.com/Unity-Technologies/UIElementsExamples)

---
### UIElements开发者指南
Unity编辑器用户界面主要是围绕即时模式UI系统构建的。虽然IMGUI在某些上下文中工作得很好，但它也有一些严重的设计限制，这些限制会影响开发编辑器特性和扩展的每个人的工作效率。
这就是创建UIElement的动机。UIElement是一个retained-mode(保留模式)的UI系统，它打开了提高性能的大门，提供了样式表、动态/上下文事件处理、可访问性和数据持久性。
UIElements中的许多概念都基于公认的Web技术。如果您熟悉XML、CSS、JQuery、HTML DOM和DOM事件系统，那么您可能已经熟悉了许多UIElements概念。
本指南的目标是通过描述框架背后的概念以及向您提供有关如何使用UIElements构建交互用户界面（UI）的清晰解释，帮助您利用UIElements。

### 选择一个UI工具
Unity提供了三种用户接口(UI)工具。你应该基于你对以下问题的答案选择一个UI工具：
- 你是为游戏开发还是为编辑开发？
- 如果您正在开发一款游戏，用户界面是否会随游戏一起提供？

|            | Runtime dev UI | Runtime game UI  | Editor        |
| ---------- | -------------- | ---------------- | ------------- |
| IMGUI      | for debugging  | not recommmended | ✔             |
| UGUI       | ✔              | ✔                | not available |
| UIElements | 2019.x         | 2020.x           | 2019.1        |

UIElements已经准备好成为游戏和编辑器用户界面开发的首选工具包。
---

### 分析
##### 1. 简要说明
Unity的UIElements是Unity官方新开发的UI工具，旨在整合原先的IMGUI和UGUI，实现Unity引擎在UI内容开发的流程和方法上的统一。

大致可以分为3个部分：
- UXML：定义用户界面（UI）的结构。
- USS：定义UI布局，形成style。
- C#代码：负责主要逻辑，并且加载UXML和USS文件。
  
##### 2. 视觉树
首先，我们可以从编辑器界面任一个窗口的右上角的下拉选项中打开 **`UIElements Debugger`** 窗口，如图。

![UIElements Debugger窗口]()

其中的每一个节点都是一个继承`VisualElements`类的实例，这其中包括已经有的控件，例如Label、Button、Toggle等，[控件参考]()。

这也意味着，我们可以通过继承`VisualElements`来拓展出自己想要的控件。

`VisualElements`可以通过`VisualElements.Add(new VisualElements())`函数来形成树状的结构。

`VisualContainer`是`VisualElements`的容器，可以尝试用`VisualContainer`来替代`VisualElements`，这样子元素就不会显示在Debugger窗口中。

这里我修改官方案例 **Example_3** 的代码，用`VisualElements`替换`VisualContainer`，如下图32、33行，

![修改代码]()

这样我们就可以在Debugger窗口中看到下图中的树状结构。

![Example_3 Tree]()