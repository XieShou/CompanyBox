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
