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

> This leaves developers to do technical tasks, such as importing assets
, defining logic, and processing data. 
这使得开发人员需要完成技术任务，例如导入资源、定义逻辑和处理数据。

> PS：个人猜测官网希望的正常开发流程：
> 1. 通过加载UXML文件创建出视觉树中的各元素。
> 2. 将USS文件中的style应用到视觉树中的最上层root元素，style会自动向下同步。
> 3. 通过 `Query` 可以指定 `name` 和指定组件类型，从而获取到一棵视觉树中特定的组件。
> 4. 而C#更多的负责UXML、USS配置文件的加载和逻辑的处理，而不是创建元素和元素的布局。
> 通过上面的流程，可以使UI开发更加清晰明了，主要还是把UI的排版和逻辑处理进行了一定的分割，可能对于编辑器的插件的UI的美观性不是那么重要，但是结合Unity官方的发展介绍，这种工作的流程在游戏中的UI作用更加大，比如在游戏UI进行更新时，程序根本不需要关心这个UI的层级结构怎么变化了，因为获取的方式是通过 `Query`，始终是能拿到组件的，具体的UI布局和风格什么的可以直接交给相关人员去调整，更多的让我们拭目以待。

#### 2. 视觉树
首先，从编辑器界面任一个窗口的右上角的下拉选项中打开 **`UIElements Debugger`** 窗口，如图。

![UIElements Debugger窗口](https://raw.githubusercontent.com/XieShou/CompanyBox/master/Textures/UIElements%20Debugger.png)

用户界面的定义在根目录中。UI定义是一系列嵌套的XML元素，每个元素表示一个VisualElement。

每一个元素都是一个继承`VisualElements`类的实例，这其中包括已经有的控件，例如Label、Button、Toggle等，[控件参考]()。

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
# C#
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
        // 获取根元素
        VisualElement root = rootVisualElement;
        // 使用代码的方式创建一个Label，并添加到root元素下
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

---
---

# UXML
##### 1. 自定义VisualElement

[官方传送门](https://docs.unity3d.com/2019.1/Documentation/Manual/UIE-UXML.html)

可以实现自定义的组件，到UXML中进行智能提示
下面的代码演示了如何定义UXML traits类来初始化status属性作为StatusBar类的属性。status属性是从XML数据初始化的。
##### 2. 从C#加载UXML

```c#
public class MyWindow : EditorWindow  {
    [MenuItem ("Window/My Window")]
    public static void  ShowWindow () {
        EditorWindow w = EditorWindow.GetWindow(typeof(MyWindow));

        VisualTreeAsset uiAsset = AssetDatabase.LoadAssetAtPath<VisualTreeAsset>("Assets/MyWindow.uxml");
        VisualElement ui = uiAsset.CloneTree(null);

        w.GetRootVisualContainer().Add(ui);
    }
}
```

##### 3. UXML模板示例
UXML模板是使用定义用户界面逻辑结构的XML标记编写的文本文件。下面的代码示例展示了如何定义一个提示用户做出选择的简单面板
```xml
<?xml version="1.0" encoding="utf-8"?>
<engine:UXML
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:engine="UnityEngine.Experimental.UIElements"
    xsi:noNamespaceSchemaLocation="../UIElementsSchema/UIElements.xsd"
    xsi:schemaLocation="UnityEngine.Experimental.UIElements ../UIElementsSchema/UnityEngine.Experimental.UIElements.xsd">

    <engine:Label text="Select something to remove from your suitcase:"/>
    <engine:Box>
        <engine:Toggle name="boots" label="Boots" value="false" />
        <engine:Toggle name="helmet" label="Helmet" value="false" />
        <engine:Toggle name="cloak" label="Cloak of invisibility" value="false"/>
    </engine:Box>
    <engine:Box>
        <engine:Button name="cancel" text="Cancel" />
        <engine:Button name="ok" text="OK" />
    </engine:Box>
</engine:UXML>
```

- 文件的第一行是XML声明。声明是可选的，但是如果包含它，那么它必须在第一行，并且在它前面不能出现其他内容或空白。`version` 属性是必需的，但 `encoding` 编码是可选的。如果包含版本，则必须表示文件的字符编码。
- 下一行定义文档根目录 `<UXML>`。`<UXML>` 元素包含名称空间前缀定义的属性和模式定义文件的位置。您可以不按部分顺序指定这些属性。
- 在UIElements中，每个元素都在下面两个名称空间定义。
    - `UnityEngine.Experimental.UIElements` ：包含定义为Unity运行时一部分的元素。
    - `UnityEditor.Experimental.UIElements` ：包含Unity编辑器中可用的元素。要完全指定元素，必须在其名称空间前加上前缀。
  
###### namespace 名称空间操作

一旦定义了这个缩短的前缀，文本 `<engine:button/>` 就相当于 `<unityengine.experimental.uielements:button/>`。 

可以定义名称空间前缀。例如，行 `xmlns:engine="unityengine.experimental.uielements"` 将引擎前缀定义为与指定 `UnityEngine.Experimental.UIElements` 相同的前缀。

如果您定义自己的元素，那么这些元素可能在它们自己的名称空间中定义。如果想在UXML模板中使用这些元素，必须在 `<UXML>` 标记中包含名称空间定义和模式文件位置，以及Unity名称空间。

当你从 `Asset/Create/UIElements View` 菜单中创建一个新的UXML模板文件时，Unity编辑器自动帮你完成这些事。

元素名称（`name`）对应于要实例化的元素的C# class名。

大多数元素都有属性，它们的值被映射到C#中相应的class属性。

每个元素继承其父类类型的属性，可以向其添加自己的属性集。

VisualElement是所有元素的基类，它为所有元素提供以下属性：

- name名称：元素的标识符。名称应该是唯一的。
  
- picking-mode拾取模式：设置为响应鼠标事件的位置或忽略忽略鼠标事件的位置。
  
- focus-index焦距索引：用于确定制表时的焦距顺序。聚焦环（见聚焦环）
  
- class类：用空格分隔的标识符列表，用于描述元素的特征。使用类为元素指定视觉样式。还可以使用类在UQuery中选择一组元素。
  
- slot name和slot：slot充当放置方法，在实例化UXML组件时插入其他可视元素。请参见下面的插槽。
  
- tooltip工具提示：当鼠标悬停在元素上时显示为工具提示的字符串。


重用UXML：

```C#
<engine:Template path="Assets/Portrait.uxml" name="Portrait">
```

##### 4. [UXML参考](https://docs.unity3d.com/2019.1/Documentation/Manual/UIE-ElementRef.html)
内置的标准控件
- Button
- Contextual menu
- EditorTextField
- Label
- ScrollView
- TextField
- Toggle


##### 5. UQuery
UQuery提供了一组扩展方法，用于从任何UIElements可视化树中检索元素。
UQuery基于JQuery或Linq，但UQuery的设计目的是尽可能限制动态内存分配。
这允许在moble平台上实现最佳性能。

要使用UQuery检索元素，请使用 `UQueryExtensions.Q` 或使用 `UQueryExtensions.Query` initialize一个 `QueryBuilder` 。

例如，下面的UQuery从根目录开始，找到第一个名为foo的按钮:

```C#
root.Query<Button>("foo").First();
```

以下uquery在同一组中对每个名为foo的按钮进行迭代：

```C#
root.Query("foo").Children<Button>().ForEach(//do stuff);
```

---
---

# USS (Unity style sheets)

## USS selectors

#### Type
`TypeName { ... }`

#### Name
`name { ... }`
与 `VisualElement.name` 匹配，每个元素的名称都应该是独一无二的（来自官方的建议）。

#### Class
`.class { ... }`
不能以数字开头。

#### Wildcard
`* { ... }`

#### Pseudo-states
`:pseudo-state { ... }`
当元素进入特定状态时，使用伪状态来匹配它。
例如，`Button:hover`匹配`Button`类型的视觉元素，但仅当用户将游标定位在视觉元素上时。
包含以下状态：
- hover : the cursor is hovering over the visual element.
- active : the visual element is being interacted with.
- inactive : the visual element is no longer being interacted with.
- focus : the visual element has focus.
- selected : unused.
- disabled : the visual element is set to enabled == false.
- enabled : the visual element is set to enable == true.
- checked : the visual element is a Toggle element and it is checked.
伪状态是在其他简单选择器之后指定的。不能扩展伪状态。只有一组预定义的受支持的psuedo状态。

### Complex selectors 复杂选择器
复杂选择器是带有分隔符的简单选择器的组合。复杂的选择器还包括选择器列表，这些选择器列表提供了对许多元素应用相同样式的简化方法。

### Delimiters 分隔符
UIElements支持以下分隔符：
- 空(或空格)分隔符匹配元素的所有后代。
- 大于号(`>`)匹配由前面的选择器匹配的元素的直接子代视觉元素。

Example：
- #container1 .yellow : 匹配内部元素和第一个按钮。
- #container2 > .yellow : 只匹配内部元素。

### Selector List 选择器列表
使用选择器列表对许多元素应用相同的样式定义。每个选择器由逗号分隔，每个选择器可以是简单的或复杂的选择器。
Example:
`#container1, Button { padding-top:10 }`
is the same as:
`#container1 { padding-top: 10 } Button { padding-top: 10}`

### Selector precedence 选择优先级
如果多个选择器匹配相同的元素，则具有最高专一性的选择器优先。对于简单的选择器，基本的特异性规则是：
- Name is more specific than,
- Class, which is more specific than,
- Type, which is more specific than,
- wildcard *.

如果两个选择器相等，则文件中最后出现的选择器优先。

为了确定不同文件之间的选择器特异性，UIElement考虑样式应用程序遍历树的顺序。具有更高深度和同级索引的元素优先。

当默认样式和用户定义样式具有相等的选择器时，用户定义的选择器被认为比默认选择器更特定。

忽略`!important`关键字。

C#中设置的值具有最高的 specificity

## 属性类型

##### 内置vs自定义属性

在使用USS时，可以为内置的VisualElement属性或UIcode中的自定义属性指定值。

在使用USS时，可以为内置的`VisualElement`属性或UIcode中的自定义属性指定值。
除了从USS文件中读取它们的值之外，还可以在C#语言中为其分配内置属性值

您可以使用定制属性API扩展USS。

##### 属性值
以下关键词有特殊含义：
`auto`、`inherit`、`unset`、`true`、`false`、`none`

USS可以使用以下格式读取文件夹中的资源显示：
- 如果文件位于`Resources`文件夹下：`background-image: resource("Images/my-image")`。
- 如果文件位于编辑器默认`Resources`文件夹下：`background-image: resource("Images/default-image.png")`。
- `background-image: url("Images/my-image.png")`。

对于纹理，如果一个文件有一个后缀为 `@2x` 的版本，这个文件会自动加载 retina 或高DPI屏幕。

---
---

# Event

UIElements 事件通知是使用深度优先搜索，所以会有两个阶段
- trickle down 
- bubble up

响应事件的两种方式：
- 可以通过注册Event的CallBack。
- 通过实现默认操作。

`PreventDefault()`：用来取消事件响应的默认操作

##### Synthesizing Events 合成事件
UIElements使用事件池来避免重复分配事件对象。要合成和发送您自己的事件，您应该按照相同的步骤分配和发送事件
- 从事件池中获取事件对象。
- 填写事件属性。
- 将事件封装在 `using` 块中，以确保将其返回到事件池。
- 将事件传递给 `element.SendEvent()`。


下面的示例演示如何合成和发送事件：
```C#
void SynthesizeAndSendKeyDownEvent(IPanel panel, KeyCode code,
     char character = '\0', EventModifiers modifiers = EventModifiers.None)
{
    // Create a UnityEngine.Event to hold initialization data.
    // Also, this event will be forwarded to IMGUIContainer.m_OnGUIHandler
    var evt = new Event() {
        type = EventType.KeyDownEvent,
        keyCode = code,
        character = character,
        modifiers = modifiers
    };

    using (KeyDownEvent keyDownEvent = KeyDownEvent.GetPooled(evt))
    {
        UIElementsUtility.eventDispatcher.DispatchEvent(keyDownEvent, panel);
    }
}
```