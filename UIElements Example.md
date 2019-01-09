# Unity - UIElements
[官方文档传送门](https://docs.unity3d.com/2019.1/Documentation/Manual/UIElements.html)
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

### 免责声明

UIElements是一个实验特性：它是不完整的，并且会受到api中断更改的影响。UIElements仍在积极开发中。
此外，在2018.3中对UIElements的更改将不会返回到旧版本。如果升级，还必须从以前的Unity版本升级一些元素。

### 视觉树
可视化树包含窗口中的所有可视化元素。它是一个由轻量级节点(称为可视化元素)组成的对象图。
这些节点在C#堆上分配，可以手动分配，也可以从UXML模板文件加载UXML资源。
每个节点都包含布局信息、其绘图和重绘选项以及节点如何响应事件。

### 视觉元素
VisualElement是可视化树中所有节点的公共基类。VisualElement基类包含样式、布局数据、本地转换、事件处理程序等属性。
VisualElement有几个子类，它们定义额外的行为和功能，包括专门的控件。VisualElement可能有子元素。
你不需要从VisualElement基类派生来使用UIElement。您可以通过样式表和事件回调来定制VisualElement的外观和行为。

### Connectivity 连通性
可视树的根对象称为面板。新元素在连接到面板之前将被忽略。您可以向现有元素添加元素以将用户界面附加到面板。

要验证`VisualElement`是否连接到面板，可以测试该元素的面板属性。当可视化元素没有连接时，测试返回`null`。

注意:当UIElements是实验性的，你必须通过` UnityEditor.Experimental.UIElements`命名空间中的`GetRootVisualContainer()`扩展方法。。此名称空间用于防止意外使用此属性。

### 绘图顺序
可视化树中的元素是按照以下顺序绘制的:
- 父元素是在其子元素之前绘制的。
- 子元素是根据其兄弟姐妹列表绘制的。

更改其绘图顺序的唯一方法是对父级中的VisualElementObjects重新排序。

通过设置 **visualElement.clippingOptions=clippingOptions.clipAndCacheContents**，可以在RenderTexture中绘制子树，并为将来的重新绘制事件重新使用像素。


### 位置、变换和坐标系

不同的坐标系定义如下：

世界：坐标是相对于面板空间的。面板是可视树中的最高元素。

局部：坐标是相对于元素本身的。

布局系统为每个元素计算`VisualElement.layout`属性（类型`Rect`）。

`layout.position`以相对于其父级坐标空间的像素表示。尽管可以直接为`layout.position`指定值，但建议使用样式表和布局系统来定位元素。

每个`VisualElement`还具有一个`layout.transform`属性（type`ITransform`），该属性相对于其父元素定位一个元素。默认情况下，`transform `是标识。

`VisualElement.layout.position`和`VisualElement.layout.transform`属性定义如何在本地坐标系和父坐标系之间进行转换。

`VisualElementExtensions`静态类提供以下扩展方法，用于在坐标系之间转换点和矩形：

- `WorldToLocal`将`vector2`或`Rect`从`Panel`空间转换为元素内的引用。

- `LocalToWorld`将`vector2`或`Rect`转换为`Panel`空间引用

- `ChangeCoordinatesTo`将`vector2`或`Rect`从一个元素的局部空间转换为另一个元素的局部空间。 

![layout example](https://docs.unity3d.com/2019.1/Documentation/uploads/Main/visualtree-hierarchy.png)

For example, in the image above, the tree is arranged as follows:
- Panel
    - Tab section (refered to as DockArea and labelled “Coordinates”)
        - Blue VisualElement acts as the root (refered to as “root container”)
            - Red VisualElement acts as a parent of the button (“red container”)
                - Button

From the point of view of the panel:

- The origin of the panel is (0, 0) regardless of the referential
- The origin of the root is (0, 22) in world space
- The origin of the red container is (100, 122) in world space. Its position property (defined in layout property) is set as (100, 100) because it is relative to its parent: the root container.
- The origin of the button is (100, 122) in the world space. Its position property (defined in layout property) is set as (0, 0) because it is relative to its parent: the red container.

The origin of an element is its top left corner.

Use the worldBound property to retrieve the window space coordinates of the VisualElement, taking into account the transforms and positions of its ancestry. 
### 布局引擎
UIElements包括一个布局引擎，它根据布局和样式属性定位视觉元素。布局引擎是一个Yoga开源项目，它实现了flexbox的一个子集：一个HTML/CSS布局系统。

要开始使用Yoga和FlexBox，请咨询以下外部资源：
- [Yoga传送门](https://yogalayout.com/)
- [CSS-Tricks guide to Flexbox传送门](https://css-tricks.com/snippets/css/a-guide-to-flexbox/)

默认情况下，所有可视元素都是布局的一部分。布局具有以下默认行为:
- 容器垂直分布其子容器。
- 容器矩形的位置包括它的子矩形。这种行为可以被其他布局属性限制。
- 带有文本的可视元素在其大小计算中使用文本大小。这种行为可以被其他布局属性限制。

UI元素包括标准UI控件（如按钮、切换、文本字段或标签）的内置控件。这些内置控件具有影响其布局的样式。

以下列表提供了有关如何使用布局引擎的提示：

- 设置宽度和高度以定义元素的大小。
- 使用flex grow属性（在uss:flex grow:<value>；）为元素指定灵活的大小。当元素的大小由其同级确定时，flexgrow属性的值用作权重。
- 将flex direction属性设置为row（在uss:flex direction:row；）以切换到水平布局。
- 使用相对定位根据元素的原始布局位置偏移元素。
- 使用绝对定位相对于其父位置矩形放置元素。在这种情况下，它不会影响其兄弟或父级的布局。
- 如果元素具有由API分配的layout.position属性，则该元素将自动设置为absolute。 

### UXML格式
UXML文件是定义用户界面逻辑结构的文本文件。UXML文件中使用的格式受到HTML(超文本标记语言)、XAML(可扩展应用程序标记语言)和XML(可扩展标记语言)的启发。如果您熟悉这些公认的格式，您应该会注意到UXML中的许多相似之处。然而，UXML格式包含了一些细微的差异，从而提供了一种使用Unity的有效方法。
本节描述Unity支持的UXML格式，并提供有关编写、加载和定义UXML模板的详细信息。它还包括关于定义新元素以及如何使用UQuery的信息。
UXML使得不太懂技术的用户更容易在Unity中构建用户界面。UXML允许您这样做:
- 用XML定义用户界面(UI)的结构。
- 使用USS样式表定义UI布局。

这使得开发人员需要完成技术任务，例如导入资源、定义逻辑和处理数据。

### 定义新元素
UIElements是可扩展的。您可以定义自己的用户界面组件和元素。
但是在使用UXML文件定义新元素之前，必须从VisualElement或其子类派生一个新类，然后在这个新类中实现适当的功能。新类必须实现默认构造函数。
例如，下面的代码派生新的StatusBar类并实现其默认构造函数：
```c#
class StatusBar : VisualElement
{
    public StatusBar()
    {
        m_Status = String.Empty;
    }

    string m_Status;
    public string status { get; }
}
```
为了让UIElement能够在读取UXML文件时实例化一个新类，必须为类定义一个工厂。除非您的工厂需要做一些特殊的事情，否则您可以从`UxmlFactoy<T>`派生工厂。还建议将工厂类放在组件类中。
例如，下面的代码演示了如何通过从`UxmlFactory<T>`;派生StatusBar类的工厂来为其定义工厂。工厂被命名为`Factory`。
```C#
class StatusBar : VisualElement
{
    public new class Factory : UxmlFactory<StatusBar> {}

    // ...
}
```
定义了这个工厂之后，您就可以在UXML文件中使用<StatusBar>元素了。
注:工厂在2018.2得到改善。如果您在以前的版本中定义了工厂，您应该将它们移植到2018.3，以避免现在已经过时的API。

### 在元素上定义属性
您可以为新类定义UXML特征，并将其工厂设置为使用这些特征。例如，下面的代码演示了如何定义UXML traits类来初始化`status`属性作为`StatusBar`类的属性。status属性是从XML数据初始化的。
```c#
class StatusBar : VisualElement
{
    public new class Factory : UxmlFactory<StatusBar, UxmlTraits> {}

    public new class Traits : base.UxmlTraits
    {
        UxmlStringAttributeDescription m_Status = new UxmlStringAttributeDescription { name = "status" };

        public override IEnumerable<UxmlChildElementDescription> uxmlChildElementsDescription
        {
            get { yield break; }
        }

        public override void Init(VisualElement ve, IUxmlAttributes bag, CreationContext cc)
        {
            base.Init(ve, bag, cc);
            ((StatusBar)ve).status = m_Status.GetValueFromBag(bag, cc);
        }
    }

    // ...
}
```

`UxmlTraits `有两个用途：
- 工厂使用它来初始化新创建的对象。
- 它通过模式生成过程进行分析，以获取有关元素的信息。这些信息被转换成XML模式指令。

上面的代码示例执行以下操作：
- `m_Status`声明定义了名为status的XML属性。
- UxmlChildElementsDescription返回一个空的IEnumerable，指示statusBar元素没有子元素，
- init（）成员从XML解析器读取属性包中status属性的值，并将statusbar.status属性设置为该值。
- 通过将UxmlTraits类放在statusBar类中，允许init（）方法访问statusBar的私有成员。
- 新的uxmltraits类继承自基类uxmltraits，因此它共享基类的属性。
- init（）调用base.init（）初始化基类属性。

上面的代码示例声明了一个具有`UxmlStringAttributeDescription`类的字符串属性。UIElements 支持以下类型的属性，并且每个属性都将C#类型链接到XML类型：
- UxmlStringAttributeDescription:属性值是`string`字符串。
- UxmlFloatAttributeDescription:属性值必须是C# `float`类型范围内的单精度浮点值。
- UxmlDoubleAttributeDescription:属性值必须是C# `Double`类型范围内的双精度浮点值。
- UxmlIntAttributeDescription:属性值必须是C# `int`类型范围内的整数值。
- UxmlLongAttributeDescription:属性值必须是C# `long`类型范围内的长整数值。
- UxmlBoolAttributeDescription:属性值必须为`true`或`false`。
- UxmlColorAttributeDescription:属性值必须是表示颜色（#ffffff）的字符串。
- UxmlEnumAttributeDescription<T>:属性值必须是表示`Enum`类型`T`的值之一的字符串。

在上面的代码示例中，`uxmlChildElementsDescription` 返回一个空的`IEnumerable`，表示`StatusBar`元素不接受子元素。
要让元素接受任何类型的子元素，必须覆盖`uxmlChildElementsDescription`属性。例如，要使`StatusBar`元素接受任何类型的子元素，必须按照以下方式指定`uxmlChildElementsDescription`属性:
```c#
public override IEnumerable<UxmlChildElementDescription> uxmlChildElementsDescription
{
    get
    {
        yield return new UxmlChildElementDescription(typeof(VisualElement));
    }
}
```

### Defining a namespace prefix 定义名称空间前缀
一旦您在C#中定义了一个新元素，就可以开始在UXML文件中使用该元素。如果新元素是在新命名空间中定义的，则应为该命名空间定义前缀。命名空间前缀声明为根`<UXML>`元素的属性，并在范围元素时替换完整的命名空间名称。

要定义名称空间前缀，请为要定义的每个名称空间前缀向程序集添加一个`UxmlNamespacePrefix`属性。
```c#
[assembly: UxmlNamespacePrefix("My.First.Namespace", "first")]
[assembly: UxmlNamespacePrefix("My.Second.Namespace", "second")]
```
这可以在程序集的任何C#文件的root根级别（在任何命名空间之外）完成。

模式生成系统执行以下操作：
- 检查这些属性并使用它们来生成模式。
- 将命名空间前缀定义添加为新创建的uxml文件中的<uxml>元素的属性。

在其`xsi:schemaLocation`属性中包含命名空间的架构文件位置。

您应该更新项目的UXML模式。选择 `Assets > Update UIElements`模式以确保文本编辑器识别新元素。

### Advanced usage
您可以通过重写其`IUxmlFactory.uxmlName`和`IUXmlFactory.uxmlQualifiedName`属性来自定义UXML名称。确保`uxmlName`在您的命名空间中是唯一的，并且`uxmlQualifiedName`在您的项目中是唯一的。

如果这两个名称不是惟一的，则在尝试加载程序集时将引发异常。

下面的代码示例演示如何覆盖和自定义UXML名称：
```c#
public class FactoryWithCustomName : UxmlFactory<..., ...>
{
    public override string uxmlName
    {
        get { return "UniqueName"; }
    }

    public override string uxmlQualifiedName
    {
        get { return uxmlNamespace + "." + uxmlName; }
    }
}
```
### 为元素选择工厂
默认情况下，`IUxmlFactory`实例化一个元素，并使用该元素的名称选择该元素。

通过重写`IUXmlFactory.AcceptsAttributeBag`，可以使选择过程考虑元素上的属性值。然后，工厂将检查元素属性，以确定它是否可以为UXML元素实例化对象。

例如，如果`VisualElement`类是泛型的，那么这很有用。在这种情况下，类专门化的类工厂可以检查XML `type`属性的值。根据值的不同，可以接受或拒绝实例化。有关示例，请参见PropertyControl<T>的实现。

如果一个以上的工厂可以实例化一个元素，则选择第一个已注册的工厂。

### 覆盖基类属性的默认值
可以通过在派生的`UxmlTraits`类中设置属性的`defaultValue`来更改基类中声明的属性的默认值。
例如，下面的代码展示了如何设置`m_focusIndex`的默认值的更改：
```C#
class MyElementTraits : VisualElementUxmlTraits
    {
        public Traits()
        {
            m_focusIndex.defaultValue = 0;
        }
    }
```

### 接受任何属性
默认情况下，生成的XML模式声明元素可以具有任何属性。

属性的值（`UxmlTraits`类中声明的属性除外）不受限制。这与检查已声明属性的值是否与其声明匹配的XML验证器不同。

其他属性包含在传递给 `IUxmlFactory.AcceptsAttributBag()` 和 `IUxmlFactory.Init()` 函数的 `IUxmlAttributes` 包中。是否使用这些附加属性取决于工厂实现。默认行为是丢弃附加属性。

这意味着这些附加属性不会附加到实例化的VisualElement上，并且这些属性不能通过`UQuery`进行查询。

定义新元素时，可以通过在`UxmlTraits`构造函数中将`UxmlTraits.canHaveAnyAttribute`属性设置为`false`，将接受的属性限制为显式声明的属性。

### 使用模式定义
模式定义文件指定属性以及每个UXML元素可以包含哪些子元素。使用模式定义文件作为编写正确文档和验证文档的指南。

在UXML模板文件中，`<UXML>`根元素的 `xsi:noNamespaceSchemaLocation` 和 `xsi:schemaLocation` 属性指定架构定义文件的位置。

选择 **Assets > Create > UIElements View** 菜单项，使用从项目使用的 `VisualElement` 子类收集的最新信息自动更新架构定义。要强制更新UXML模式文件，请选择 **Assets > Update UIElements Schema**。

注意：某些文本编辑器无法识别` xsi:noNamespaceSchemaLocation`属性。如果文本编辑器找不到架构定义文件，则还应指定`xsi:schemaLocation`属性。
### 编写UXML模板
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

### 重用UXML
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

### 从C#加载UXML

要从UXML模板构建用户界面，必须首先将模板加载到VisualTreeAsset中
```c#
var template = EditorGUIUtility.Load("path/to/file.uxml") as VisualTreeAsset;
```
然后您可以构建它所表示的可视化树，并将其附加到父元素
```C#
template.CloneTree(parentElement, slots);
```
在上面的语句中，模板中的<UXML>元素没有转换为一个`VisualElement`。相反，它的所有子元素都附加到`parentElement`指定的元素。
一旦模板被实例化，就可以使用 UQuery:Unity 的 JQuery/Linq 实现从可视元素树中检索特定元素。
例如，下面的代码演示了如何创建一个新的`EditorWindow`并加载一个UXML文件作为其内容：
```c#
public class MyWindow : EditorWindow  {
    [MenuItem ("Window/My Window")]
    public static void  ShowWindow () {
        EditorWindow w = EditorWindow.GetWindow(typeof(MyWindow));

        VisualTreeAsset uiAsset = AssetDatabase.LoadAssetAtPath<VisualTreeAsset>("Assets/MyWindow.uxml");
        VisualElement ui = uiAsset.CloneTree(null);

        w.GetRootVisualContainer().Add(ui);
    }

    void OnGUI () {
        // Nothing to do here, unless you need to also handle IMGUI stuff.
    }
}
```
### UQuery
UQuery提供了一组扩展方法，用于从任何UIElements可视化树中检索元素。UQuery基于JQuery或Linq，但UQuery的设计目的是尽可能限制动态内存分配。这允许在moble平台上实现最佳性能。
要使用UQuery检索元素，请使用`UQueryExtensions.Q`或使用`UQueryExtensions.Query`初始化`QueryBuilder`。
例如，下面的UQuery从根目录开始，找到第一个名为`foo`的`Button`。

```c#
root.Query<Button>("foo").First();
```

下面的UQuery在同一组中对每个名为`foo`的`Button`进行迭代:

```
root.Query("foo").Children<Button>().ForEach(//do stuff);
```