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
UIElements包括一个布局引擎，它根据布局和样式属性定位视觉元素。布局引擎是一个[Yoga开源项目](https://github.com/facebook/yoga)，它实现了flexbox的一个子集：一个**HTML/CSS**布局系统。

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

