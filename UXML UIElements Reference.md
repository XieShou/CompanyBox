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










































### UXML 元素参考

[原文链接](https://docs.unity3d.com/2019.1/Documentation/Manual/UIE-ElementRef.html)
本主题详细介绍了`UnityEngine.Experimental.UIElements`中可用的UXML元素和`UnityEditor.Experimental.UIElements`命名空间。

##### Base elements 基础元素

###### VisualElement
所有视觉元素的基类。
- 在`UnityEngine.Experimental.UIElements`中。
- 允许的子元素:任意数量的`VisualElement`。
- 属性：
    - class：以空格分隔的名称的列表。
    - style：用于对元素进行样式化的USS指令。
    - name：此元素的唯一字符串标识符。
    - focus-index：用于确定制表时焦点顺序的整数;默认值是-1，意味着元素不可调焦。
    - picking-mode：Position 或 Ignore; 默认值是 Position。
    - tooltip: 鼠标悬停元素时显示的字符串。
    - slot-name: 将此元素定义为槽。
    - slot: 当元素位于<Instance>内时，将元素移动到此属性引用的槽内
    - 还接受任何其他属性

###### BindableElement
可以绑定到` SerializedProperty`(序列化属性)的元素。属性的值和显示的值是同步的。
- 在`UnityEngine.Experimental.UIElements`中。
- 允许的子元素:任意数量的`VisualElement`。
- Attributes:
    - 所有`VisualElement`属性。
    - `binding-path`:该元素绑定到的属性的路径。

##### Utilities 实用工具

###### Box
类似于`VisualElement`，但在其内容周围绘制一个框。

- 在`UnityEngine.Experimental.UI`元素中
- 允许的子元素：任何数量的`VisualElement`
- Attributes：
    - `VisualElement`的所有属性

###### TextElement
显示文本的元素。
- 在`UnityEngine.Experimental.UI`元素中
- 允许的子元素：无
- 属性：
    - `VisualElement`的所有属性
    - 文本：元素显示的文本


###### Label
文本标签。
- 在`UnityEngine.Experimental.UI`元素中
- 允许的子元素：无
- 属性：
    - `TextElement`的所有属性

###### Image
显示图像。
- 在`UnityEngine.Experimental.UI`元素中
- 允许的子元素：无
- 属性：
    - `VisualElement`的所有属性


###### IMGUIContainer
绘制IMGUI内容的元素。
- 在`UnityEngine.Experimental.UI`元素中
- 允许的子元素：无
- 属性：
    - `VisualElement`的所有属性。
      - `focus-index`默认值为0。

###### Foldout
具有切换按钮以显示或隐藏其内容的元素。
- 在`UnityEngine.Experimental.UI`元素中
- 允许的子元素：任何数量的`VisualElement`
- 属性：
    - `BindableElement`的所有属性

##### Templates

###### Templates
可以使用实例元素实例化的另一个UXML模板的引用。
- In `UnityEngine.Experimental.UIElements`
- Permitted child elements: none
- Attributes:
    - name: a unique string identifier for this element
    - path: the path of the UXML file to load
