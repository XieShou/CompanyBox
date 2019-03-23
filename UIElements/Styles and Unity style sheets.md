# Styles and Unity style sheets

每个`VisualElement`都包含样式属性，用于设置元素的尺寸和元素在屏幕上的绘制方式，例如`backgroundColor`或`borderColor`。

样式属性可以在C#中设置，也可以从样式表中设置。样式属性在其自己的数据结构（`IStyle`接口）中重新分组。

UIElements支持用USS (Unity样式表)编写的样式表。USS文件是受HTML中的层叠样式表(CSS)启发的文本文件。USS格式类似于CSS，但是USS包含重写和定制，以便更好地与Unity一起工作。

本节详细介绍USS及其语法，以及与CSS相比的差异。

### 定义 Unity Style Sheet

Unity样式表(USS)的基本构建模块如下所示：

- USS是识别为资源的文本文件。文本文件必须具有.uss扩展名。

- USS只支持样式规则。

- 样式规则由选择器和声明块组成。

- 选择器标识样式规则影响的可视元素。

- 声明块由大括号括起来，包含一个或多个样式声明。每个样式声明都由一个属性和一个值组成。每个样式声明以分号结尾。

- 每个Style属性的值都是一个文本，解析时必须与目标属性名匹配。

样式规则的一般语法是：

```USS
selector {
  property1:value;
  property2:value;
}
```

USS语法匹配CSS3的W3C规范。UIElements使用[ExCSS开源解析器](https://github.com/Unity-Technologies/ExCSS)来解析USS声明。ExCSS开源解析器有自己的测试套件。

### 将USS附加到视觉元素

可以将Unity样式表（USS）附加到任何可视元素。样式规则应用于可视元素及其所有子代。必要时，样式表也会自动重新应用。

使用`AddStyleSheetPath()`方法将 USS 与可视元素关联。还必须向`AddStyleSheetPath()`方法提供封闭文件夹的相对路径。要使Unity识别USS文件，必须将其放置在 `Resources` 或 `Editor Default Resources` 文件夹中，即Assets文件夹中。


如果在编辑器窗口运行时修改USS文件，则立即应用样式更改。

### 与规则匹配的样式

一旦定义了样式表，它就可以应用于可视元素的UIElements树。

在此过程中，选择器与元素匹配，以解析从USS文件应用哪些属性。如果选择器与某个元素匹配，则样式声明将应用于该元素。

例如，以下规则匹配任何按钮对象：

```USS
Button {
  width: 200px;
}
```

使用`VisualElement.AddStyleSheetPath()`方法将样式表附加到一个子树。

### VisualElement匹配

UIElements使用以下标准来匹配视觉元素及其样式规则：

- C# 类名（总是最派生的类）。
- 一个`name`属性，它是一个字符串。
- 用一组字符串表示的类列表。
- VisualElement在可视树中的祖先和位置。

这些特征可以在样式表的选择器中使用。

如果您熟悉CSS，您可以看到它与HTML标记名称、`id`属性和`class`属性的相似性。