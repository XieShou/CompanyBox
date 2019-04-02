# AssetBundle

AssetBundle能够让程序在运行时动态的加载资源。AssetBundle是通过BuildPipeline.BuildAssetBundle创建的。

assetbundle可以用于可下载内容(DLC)，减少初始安装大小，加载针对最终用户平台优化的资源，并减少运行时内存压力。

注意：AB包在各个平台之间互不兼容。例如IOS平台的AB包不能使用在Android平台使用。

### 在运行时加载资源。

1. 针对于某些加载次数较少的资源。
2. 减少用户初始下载的时间。
3. 进行资源的替换、更新。

虽然Unity已经提供了`Resource Folders`的资源加载方式，但是仍旧可以使用AssetBundle。

### 卸载资源

您可以通过调用`AssetBundle.Unload()`来卸载AssetBundle的资源。如果`unloadAllLoadedObjects`参数传递`true`，那么AssetBundle内部持有的对象和使用`AssetBundle.LoadAsset()`从AssetBundle加载的对象都将被销毁，bundle使用的内存将被释放。

### 依赖关系

AB包自身保存着互相的依赖关系。

### 分类

首先是磁盘上的实际文件。我们将其称为AssetBundle档案。可以将档案看作是一个容器，就像文件夹一样，其中包含额外的文件。这些附加文件包括两种类型：

1. `serialized file` 

    序列化的文件包含您的资源，这些资源被分解成它们各自的对象，并写到这个文件中。
2. `resource files`

    资源文件只是为某些资源(纹理和音频)单独存储的二进制数据块，以便我们能够有效地从另一个线程上的磁盘加载它们。

第二个是通过代码与之交互的实际AssetBundle对象，用于从特定档案文件加载资源。此对象包含一个将添加到此存档的资源的所有文件路径的Map，映射到在请求该资源时需要加载的该资源所属的对象。

### 压缩方式

使用`BuildPipeline.BuildAssetBundles()`代码，在编辑器中打包生成AssetBundle。在这些参数里，您可以指定要包含在构建文件中的对象数组，以及一些其他选项。这将构建一个文件，稍后您可以使用`AssetBundle.LoadAsset()`在运行时动态加载该文件。

### 打包策略

要制定符合自己需求的打包策略，例如：

1. 绑定用户界面屏幕的所有纹理和布局数据。
2. 绑定所有模型、动画和其Character。
3. 捆绑纹理和模型的片段共享的场景在多个层次

### BuildPipeline.BuildAssetBundles

##### BuildAssetBundleOptions

[传送门](https://docs.unity3d.com/ScriptReference/BuildAssetBundleOptions.html)

三种主要的压缩选项：

- BuildAssetBundleOptions.None: 

  这个打包选项使用LZMA格式压缩。

  在使用任何一个包内资源时，都需要对整个包进行解压缩。

  当解压缩包后，它将使用LZ4格式重新再磁盘上重新压缩，通过UnityWebRequestAssetBundle加载的LZMA压缩资产包被自动重新压缩为LZ4压缩，并缓存在本地文件系统中。

- BuildAssetBundleOptions.UncompressedAssetBundle: 

  以完全不压缩的方式构建这些资源，占用空间大，但加载速度快很多。

- BuildAssetBundleOptions.ChunkBasedCompression: 

  - LZ4压缩。
  - 占用空间比LZMA大。
  - 它不要求在使用整个bundle之前对其进行解压缩。
  - LZ4使用一个基于块的算法，该算法允许以块的形式加载AssetBundle。
  - 解压缩单个块允许使用所包含的资产，即使没有解压缩AssetBundle的其他块。

使用ChunkBasedCompression与不压缩的Bundle具有相同的加载时间，占用磁盘空间更小。

##### BuildTarget

- BuildTarget.Standalone 

  不想在构建目标中硬编码，那么可以随意利用`EditorUserBuildSettings.activeBuildTarget`将自动找到您当前要构建的平台，并基于该目标构建您的资产包。

##### .manifest文件

每生成一个Bundle都会生成一个.manifest文件，它显示所包含的资产、依赖项和其他信息。

其中包含循环冗余检查(CRC)数据和包的依赖项数据等信息

### Dependencies

如果一个或多个UnityEngine.Objects包含对位于另一个包中的UnityEngine.Object的引用，则AssetBundles可能会依赖于其他AssetBundles。

如果UnityEngine.Object包含一个对未被包含在任何AssetBundle中的UnityEngine.Object的引用，则不会发生依赖关系。在这种情况下，在构建AssetBundles时，将把包所依赖的对象的副本复制到包中。

如果多个Bundle中的多个对象包含对未分配给bundle的同一对象的引用，则每个依赖于该对象的bundle都将生成其自己的对象副本，并将其打包到构建的assetbundle中。

##### 例子

`Bundle_1`中的材质引用`Bundle_2`中的贴图

在读`Bundle_1`中的材质之前，你需要将`Bundle_2`加载到内存中，加载的顺序并不重要，重要的是在加载`Bundle_1`的该材质之前加载了`Bundle_2`
