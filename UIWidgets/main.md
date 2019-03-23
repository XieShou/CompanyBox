# UIWidgets

**UIWidget**是一个帮助开发组创建、调试和部署高效的跨平台应用程序的UnityPackage。

从@https://github.com/flutter/flutter. 衍生出来。

[官方页面](https://github.com/UnityTech/UIWidgets)

## 开始

### 概述
在这个教程中，你将快速创建一个十分ez的UIWidgets App。这个App只包含一个文本框和一个按钮。这个文本框将显示你点击按钮的次数。

首先创建项目，并且将从[官方页面]下载的Package导入工程。

打开Project设置，去Player选项并且在脚本调试符号字段（Scripting Debug Symbols ）增加 "UIWidgets_DEBUG" 。这为您的开发启用了UIWidgets的调试模式。在以后的发布构建中删除这个。

### 构建Scene
一个UIWdigets App通常搭建在一个Unity UI 的Canvas上。

### 创建Widget

1. 创建一个脚本 "UIWidgetsExample.cs"，在Canvas中创建一个Panel，删除Panel中的Image背景图片，将脚本绑定上去。
```C#
using System.Collections.Generic;
 using Unity.UIWidgets.animation;
 using Unity.UIWidgets.engine;
 using Unity.UIWidgets.foundation;
 using Unity.UIWidgets.material;
 using Unity.UIWidgets.painting;
 using Unity.UIWidgets.widgets;
 
 namespace UIWidgetsSample {
     public class UIWidgetsExample : UIWidgetsPanel {
         protected override void OnEnable() {
             base.OnEnable();
 
             // Application.targetFrameRate = 60; // or higher if you want a smoother scrolling experience.
 
             // if you want to use your own font or font icons.
             // use the asset name of font (file name without extension) in FontStyle.fontFamily.             
             // FontManager.instance.addFont(Resources.Load<Font>(path: "path to your font"));                
         }
 
         protected override Widget createWidget() {
             return new WidgetsApp(
                 home: new ExampleApp(),
                 pageRouteBuilder: (RouteSettings settings, WidgetBuilder builder) =>
                     new PageRouteBuilder(
                         settings: settings,
                         pageBuilder: (BuildContext context, Animation<float> animation,
                             Animation<float> secondaryAnimation) => builder(context)
                     )
             );
         }
 
         class ExampleApp : StatefulWidget {
             public ExampleApp(Key key = null) : base(key) {
             }
 
             public override State createState() {
                 return new ExampleState();
             }
         }
 
         class ExampleState : State<ExampleApp> {
             int counter = 0;
 
             public override Widget build(BuildContext context) {
                 return new Column(
                     children: new List<Widget> {
                         new Text("Counter: " + this.counter),
                         new GestureDetector(
                             onTap: () => {
                                 this.setState(()
                                     => {
                                     this.counter++;
                                 });
                             },
                             child: new Container(
                                 padding: EdgeInsets.symmetric(20, 20),
                                 color: Colors.blue,
                                 child: new Text("Click Me")
                             )
                         )
                     }
                 );
             }
         }
     }
 }
```