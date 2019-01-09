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

|            | Runtime dev UI | Runtime game UI  | Editor        |
| ---------- | -------------- | ---------------- | ------------- |
| IMGUI      | for debugging  | not recommmended | ✔             |
| UGUI       | ✔              | ✔                | not available |
| UIElements | 2019.x         | 2020.x           | 2019.1        |

UIElements已经准备好成为游戏和编辑器用户界面开发的首选工具包。

##### 免责声明

UIElements是一个实验特性：它是不完整的，并且会受到api中断更改的影响。UIElements仍在积极开发中。
此外，在2018.3中对UIElements的更改将不会返回到旧版本。如果升级，还必须从以前的Unity版本升级一些元素。

##### 视觉树
可视化树包含窗口中的所有可视化元素。它是一个由轻量级节点(称为可视化元素)组成的对象图。
这些节点在C#堆上分配，可以手动分配，也可以从UXML模板文件加载UXML资源。
每个节点都包含布局信息、其绘图和重绘选项以及节点如何响应事件。

##### 视觉元素
VisualElement是可视化树中所有节点的公共基类。VisualElement基类包含样式、布局数据、本地转换、事件处理程序等属性。
VisualElement有几个子类，它们定义额外的行为和功能，包括专门的控件。VisualElement可能有子元素。
你不需要从VisualElement基类派生来使用UIElement。您可以通过样式表和事件回调来定制VisualElement的外观和行为。

##### 绘图顺序
可视化树中的元素是按照以下顺序绘制的:
- 父元素是在其子元素之前绘制的。
- 子元素是根据其兄弟姐妹列表绘制的。
更改其绘图顺序的唯一方法是对父级中的VisualElementObjects重新排序。

通过设置 **visualElement.clippingOptions=clippingOptions.clipAndCacheContents**，可以在RenderTexture中绘制子树，并为将来的重新绘制事件重新使用像素。


##### 位置、变换和坐标系

不同的坐标系定义如下：

世界：坐标是相对于面板空间的。面板是可视树中的最高元素。

局部：坐标是相对于元素本身的。


布局系统为每个元素计算VisualElement.layout属性（类型rect）。


layout.position以相对于其父级坐标空间的像素表示。尽管可以直接为layout.position指定值，但建议使用样式表和布局系统来定位元素。


每个VisualElement还具有一个layout.transform属性（typeitransform），该属性相对于其父元素定位一个元素。默认情况下，转换是标识。


VisualElement.layout.position和VisualElement.layout.Transform属性定义如何在本地坐标系和父坐标系之间进行转换。


VisualElementExtensions静态类提供以下扩展方法，用于在坐标系之间转换点和矩形：


worldtoolocal将vector2或rect从面板空间转换为元素内的引用。

localtoworld将vector2或rect转换为面板空间引用

changecoordinatesto将vector2或rect从一个元素的局部空间转换为另一个元素的局部空间。 


##### 布局引擎
UIElements包括一个布局引擎，它根据布局和样式属性定位视觉元素。布局引擎是一个Yoga开源项目，它实现了flexbox的一个子集：一个HTML/CSS布局系统。

要开始使用Yoga和FlexBox，请咨询以下外部资源：
- [Yoga传送门](https://yogalayout.com/)
- [CSS-Tricks guide to Flexbox传送门](https://css-tricks.com/snippets/css/a-guide-to-flexbox/)

默认情况下，所有可视元素都是布局的一部分。布局具有以下默认行为:
- 容器垂直分布其子容器。
- 容器矩形的位置包括它的子矩形。这种行为可以被其他布局属性限制。
- 带有文本的可视元素在其大小计算中使用文本大小。这种行为可以被其他布局属性限制。

UI元素包括标准UI控件（如按钮、切换、文本字段或标签）的内置控件。这些内置控件具有影响其布局的样式。


The following list provides tips on how to use the layout engine:

Set the width and height to define the size of an element.

Use the flexGrow property (in USS: flex-grow: <value>;) to assign a flexible size to an element. The value of the flexGrow property acts as weighting when the size of an element is determined by its siblings.

Set the flexDirection property to row (in USS: flex-direction: row;) to switch to a horizontal layout.

Use relative positioning to offset an element based on its original layout position.

Use absolute positioning to place an element relative to its parent position rectangle. In this case, it does not affect the layout of its siblings or parent

If an element has its layout.position property assigned by the API, the element is automatically set to absolute.

以下列表提供了有关如何使用布局引擎的提示：

- 设置宽度和高度以定义元素的大小。
- 使用flex grow属性（在uss:flex grow:<value>；）为元素指定灵活的大小。当元素的大小由其同级确定时，flexgrow属性的值用作权重。
- 将flex direction属性设置为row（在uss:flex direction:row；）以切换到水平布局。
- 使用相对定位根据元素的原始布局位置偏移元素。
- 使用绝对定位相对于其父位置矩形放置元素。在这种情况下，它不会影响其兄弟或父级的布局。
- 如果元素具有由API分配的layout.position属性，则该元素将自动设置为absolute。 

##### 编写UXML模板
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

文件的第一行是XML声明。声明是可选的，但是如果包含声明，它必须在第一行，并且在声明前面不能出现其他内容或空白。版本属性是必需的，但编码是可选的。如果包含版本，它必须表示文件的字符编码。

下一行定义文档根，<uxml>。<uxml>元素包括命名空间前缀定义的属性和模式定义文件的位置。您可以不按顺序指定这些属性。

在UIElements中，每个元素都在UnityEngine.Experimental.UIElements 或 UnityEditor.Experimental.UIElements命名空间中定义：
- UnityEngine.Experimental.UIElements命名空间包含定义为Unity运行时一部分的元素。
- UnityEditor.Experimental.UIElements命名空间包含Unity编辑器中可用的元素。要完全指定元素，必须在其前面加上名称空间。

例如，如果要在UXML模板中使用button元素，则必须指定<unityengine.experimental.uielements:button/>。


为了使指定名称空间更容易，可以定义名称空间前缀。例如，行xmlns:engine=“unityengine.experimental.uielements”将引擎前缀定义为与指定unityengine.experimental.uielements相同的前缀。


一旦定义了这个缩短的前缀，文本<engine:button/>就相当于<unityengine.experimental.uielements:button/>。


如果您定义自己的元素，那么这些元素可能是在自己的名称空间中定义的。如果要在uxml模板中使用这些元素，则必须在<uxml>标记中包括名称空间定义和模式文件位置，以及Unity名称空间。


当您创建新的UXML模板资源时，Unity编辑器会自动为您执行此操作。

从 "Asset/Create/UIElements View" 视图菜单。


用户界面的定义在<uxml>根目录中。UI定义是一系列嵌套的XML元素，每个元素表示一个VisualElement。


元素名称对应于要实例化的元素的C#类名称。大多数元素都有属性，它们的值被映射到C中相应的类属性。每个元素继承其父类类型的属性，可以向其添加自己的属性集。VisualElement是所有元素的基类，它为所有元素提供以下属性：
- name名称：元素的标识符。名称应该是唯一的。
- picking-mode拾取模式：设置为响应鼠标事件的位置或忽略忽略鼠标事件的位置。
- focus-index焦距索引：用于确定制表时的焦距顺序。聚焦环（见聚焦环）
- class类：用空格分隔的标识符列表，用于描述元素的特征。使用类为元素指定视觉样式。还可以使用类在UQuery中选择一组元素。
- slot name和slot：slot充当放置方法，在实例化UXML组件时插入其他可视元素。请参见下面的插槽。
- tooltip工具提示：当鼠标悬停在元素上时显示为工具提示的字符串。

UXML模板示例没有定义用户界面的可视方面。作为最佳实践，您应该定义样式信息，例如用于绘制用户界面的尺寸、字体和颜色。

有一个单独的文件，样式规则用USS编写（参见样式）。

##### 重用UXML
您可以通过在一个uxml文件中定义它来创建组件，并使用另一个uxml文件中的<template>和<instance>元素导入它。

在设计大型用户界面时，可以创建定义UI部分的模板UXML文件。

您可以在许多地方使用相同的UI定义。例如，假设您有一个具有图像、名称和标签的纵向UI元素。您可以创建一个UXML模板文件，以在其他UXML文件中恢复纵向用户界面。

例如，假设您在文件assets/portrait.uxml中有一个纵向组件：

```xml
<?xml version="1.0" encoding="utf-8"?>
<engine:UXML ...>
    <engine:VisualElement class="portrait">
        <engine:Image name="portaitImage" image="a.png"/>
        <engine:Label name="nameLabel" text="Name"/>
        <engine:Label name="levelLabel" text="42"/>
    </engine:VisualElement>
</engine:UXML>
```
您可以像这样将纵向组件嵌入到其他UXML模板中
```xml
<?xml version="1.0" encoding="utf-8"?>
<engine:UXML ...>
    <engine:Template path="Assets/Portrait.uxml" name="Portrait">
    <engine:VisualElement name="players">
        <engine:Instance template="Portrait" name="player1"/>
        <engine:Instance template="Portrait" name="player2"/>
    </engine:VisualElement>
</engine:UXML>
```

槽

UXML组件可以定义在实例化组件时插入元素的插槽。插槽充当放置方法，当一个UXML组件被实例化时插入其他可视元素。
使用slot name属性定义slot。例如，window.uxml中定义的window组件可以进一步定义两个名为title和content的槽：
```xml
<?xml version="1.0" encoding="utf-8"?>
<engine:UXML ...>
    <engine:VisualElement>
        <engine:VisualElement slot-name="title"/>
        <engine:ScrollView slot-name="content"/>
    </engine:VisualElement>
</engine:UXML>
```
然后，模板可以通过导入模板文件、实例化其内容并使用自己的元素填充插槽、通过添加插槽属性将可视元素绑定到插槽名称来使用此窗口
```xml
<?xml version="1.0" encoding="utf-8"?>
<engine:UXML ...>
    <engine:Template path="Assets/Window.uxml" name="window"/>
    <engine:Instance template="window">
        <engine:Label slot="title">
        <engine:TextBox slot="content"/>
    </engine:Instance>
</Window>
```
生成的可视元素层次结构将是
```
VisualElement
    VisualElement slot-name="title"
        Label slot="title"
    ScrollView slot-name="content"
        TextBox slot="content" 
```

在上面的可视元素中，具有slot=“title”属性的标签已添加为具有slot name=“title”属性的VisualElement的子级。
还可以从组件的VisualTreeSet中填充c_中的插槽：
```xml
var slots = new Dictionary<string, VisualElement>();
VisualElement t = visualTreeAsset.CloneTree(slots);
slots["title"].add(new Label());
```

在调用CloneTree之后，slot字典保存一个slot名称到slot VisualElement的映射。您可以使用它作为插槽的子元素添加可视元素。