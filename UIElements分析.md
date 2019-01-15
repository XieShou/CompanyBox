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

#### 1. 简要说明
Unity的UIElements是Unity官方新开发的UI工具，旨在整合原先的IMGUI和UGUI，实现Unity引擎在UI内容开发的流程和方法上的统一。

大致可以分为3个部分：
- UXML：定义用户界面（UI）的结构。
- USS：定义UI布局，形成style。
- C#代码：负责主要逻辑，并且加载UXML和USS文件。

> PS：个人猜测官网希望的正常开发流程：
> 1. 通过加载UXML文件创建出视觉树中的各元素。
> 2. 将USS文件中的style应用到视觉树中的最上层root节点，style会自动向下同步。
> 3. 而C#更多的负责UXML、USS配置文件的加载和逻辑的处理，而不是创建节点和节点的布局。

#### 2. 视觉树
首先，从编辑器界面任一个窗口的右上角的下拉选项中打开 **`UIElements Debugger`** 窗口，如图。

![UIElements Debugger窗口](https://raw.githubusercontent.com/XieShou/CompanyBox/master/Textures/UIElements%20Debugger.png)

其中的每一个节点都是一个继承`VisualElements`类的实例，这其中包括已经有的控件，例如Label、Button、Toggle等，[控件参考]()。

这也意味着，可以通过继承`VisualElements`类的方式来拓展出自己想要的控件。

`VisualElements`可以通过`VisualElements.Add(new VisualElements())`函数来形成树状的结构。

`VisualContainer`是`VisualElements`的容器，可以尝试用`VisualContainer`来替代`VisualElements`，这样子元素就不会显示在Debugger窗口中。

尝试修改官方案例 **Example_3** 的代码，用`VisualElements`替换`VisualContainer`，如下图32、33行，

![修改代码](https://raw.githubusercontent.com/XieShou/CompanyBox/master/Textures/Example_3.png)

可以在Debugger窗口中看到下图中的树状结构。

![Example_3 Tree](https://raw.githubusercontent.com/XieShou/CompanyBox/master/Textures/Example_3_tree.png)

#### 3. 快速开始：UIElement Editor Window ~~~
在`Assets`文件夹的目录下创建一个Editor文件夹，Unity将自动检测到这个文件夹。

可以在 **Project** 窗口的 **Create** 中找到 **UIElement Editor Window**，在文本框输入文件名后，就会在Editor文件夹目录下创建 `.cs`、`.uxml`、`.uss`三个文件。

![UIElement Editor Window](https://raw.githubusercontent.com/XieShou/CompanyBox/master/Textures/UIElement%20Editor%20Window.png)

---
### C#
##### 开始
根据上面的步骤，创建了一个文件名为Demo2的示例，自动弹出窗口如下图：

![Hello World](https://raw.githubusercontent.com/XieShou/CompanyBox/master/Textures/HelloWorld.png)

自动生成的代码如下：
```c#
using UnityEditor;
using UnityEngine;
using UnityEngine.UIElements;
using UnityEditor.UIElements;

public class Demo2 : EditorWindow
{   // ShowExample()函数格式固定，在编辑器的Window -> UIElements 中生成 Demo2 选项，点击就可以打开界面
    [MenuItem("Window/UIElements/Demo2")]
    public static void ShowExample()
    {
        Demo2 wnd = GetWindow<Demo2>();
        wnd.titleContent = new GUIContent("Demo2");
    }
    // 以前是在ONGUI()函数中写，现在在OnEnable()中
    public void OnEnable()
    {
        // 获取根节点
        VisualElement root = rootVisualElement;
        // 使用代码的方式创建一个Label，并添加到root节点下
        VisualElement label = new Label("Hello World! From C#");
        root.Add(label);
        // 可以试试:
        // VisualElement button = new Button() { style = { color = Color.red ,backgroundColor = Color.blue},
        // name = "Add Button" ,text = "Click Here"};
        // root.Add(button);

        // 读取一个UXML文件(刚一起生成的)，按照文件结构直接Clone整棵树
        var visualTree = AssetDatabase.LoadAssetAtPath<VisualTreeAsset>("Assets/UIElements_Editor/Editor/Resources/Demo2.uxml");
        VisualElement labelFromUXML = visualTree.CloneTree();
        root.Add(labelFromUXML);

        // 读取一个USS文件(刚一起生成的)，同样是代码创建，不过对创建的该VisualElement.Add添加了style
        var styleSheet = AssetDatabase.LoadAssetAtPath<StyleSheet>("Assets/UIElements_Editor/Editor/Resources/Demo2.uss");
        VisualElement labelWithStyle = new Label("Hello World! With Style");
        labelWithStyle.styleSheets.Add(styleSheet);//可以试试把这行的 labelWithStyle 改为 root
        root.Add(labelWithStyle);
    }
}
```


### UXML

### USS