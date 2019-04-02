# 使用AssetBundle

总共有4中不同的AB包加载方式，它们的行为根据所加载的Bundle平台和构建AssetBundle时使用的压缩方式 (uncompression, LZMA, LZ4)而变化。

### AssetBundle.LoadFromMemoryAsync

这个函数接受包含AssetBundle数据的字节数组，如果您愿意，还可以选择传入CRC值

```C#
using UnityEngine;
using System.Collections;
using System.IO;

public class Example : MonoBehaviour
{
    IEnumerator LoadFromMemoryAsync(string path)
    {
        AssetBundleCreateRequest createRequest = AssetBundle.LoadFromMemoryAsync(File.ReadAllBytes(path));
        yield return createRequest;
        AssetBundle bundle = createRequest.assetBundle;
        var prefab = bundle.LoadAsset<GameObject>("MyObject");
        Instantiate(prefab);
    }
}
```

`ReadAllBytes(path)`可以替换为任何想要的获取字节数组的过程。

### AssetBundle.LoadFromFile

当从本地存储加载未压缩的包时，此API非常有效。

Unity5.3版本之前使用此API将失败，因为

```C#
public class LoadFromFileExample extends MonoBehaviour {
    function Start() {
        var myLoadedAssetBundle = AssetBundle.LoadFromFile(Path.Combine(Application.streamingAssetsPath, "myassetBundle"));
        if (myLoadedAssetBundle == null) {
            Debug.Log("Failed to load AssetBundle!");
            return;
        }
        var prefab = myLoadedAssetBundle.LoadAsset.<GameObject>("MyObject");
        Instantiate(prefab);
    }
}
```

### WWW.LoadFromCacheOrDownload （被弃用）

```C#
using UnityEngine;
using System.Collections;

public class LoadFromCacheOrDownloadExample : MonoBehaviour
{
    IEnumerator Start ()
    {
            while (!Caching.ready)
                    yield return null;

        var www = WWW.LoadFromCacheOrDownload("http://myserver.com/myassetBundle", 5);
        yield return www;
        if(!string.IsNullOrEmpty(www.error))
        {
            Debug.Log(www.error);
            yield return;
        }
        var myLoadedAssetBundle = www.assetBundle;

        var asset = myLoadedAssetBundle.mainAsset;
    }
}
```

### UnityWebRequest


```C#
IEnumerator InstantiateObject()

    {
        string uri = "file:///" + Application.dataPath + "/AssetBundles/" + assetBundleName;        UnityEngine.Networking.UnityWebRequest request = UnityEngine.Networking.UnityWebRequest.GetAssetBundle(uri, 0);
        yield return request.Send();
        AssetBundle bundle = DownloadHandlerAssetBundle.GetContent(request);
        GameObject cube = bundle.LoadAsset<GameObject>("Cube");
        GameObject sprite = bundle.LoadAsset<GameObject>("Sprite");
        Instantiate(cube);
        Instantiate(sprite);
    }
```

### 从AssetBundles中加载资源

`T objectFromBundle = bundleObject.LoadAsset<T>(assetName);`
有如下四种加载方式
`LoadAsset`、`LoadAllAssets`、`LoadAssetAsync` 和 `LoadAllAssetsAsync`。

- 读取单个资源
    `GameObject gameObject = loadedAssetBundle.LoadAsset<GameObject>(assetName);`
- 读取全部资源
    `Unity.Object[] objectArray = loadedAssetBundle.LoadAllAssets();`

异步加载的资源需要等待`AssetBundleRequest`的返回值。

### 读取AssetBundle Manifests

在处理AssetBundle的依赖关系时尤其好用。

```C#
AssetBundle assetBundle = AssetBundle.LoadFromFile(manifestFilePath);
AssetBundleManifest manifest = assetBundle.LoadAsset<AssetBundleManifest>("AssetBundleManifest");
```

Manifest信息包括依赖项数据、散列数据和AssetBundle的变体数据。

动态查找一个名为`assetBundle`的Bundle的所有依赖项：

```C#
AssetBundle assetBundle = AssetBundle.LoadFromFile(manifestFilePath);
AssetBundleManifest manifest = assetBundle.LoadAsset<AssetBundleManifest>("AssetBundleManifest");
string[] dependencies = manifest.GetAllDependencies("assetBundle"); //Pass the name of the bundle you want the dependencies for.
foreach(string dependency in dependencies)
{
    AssetBundle.LoadFromFile(Path.Combine(assetBundlePath, dependency));
}
```

### 管理已经读取的Bundle

资源卸载在特定时间触发，也可以手动触发。

`AssetBundle.Unload(bool)`：非静态的卸载函数，参数指示是否也要卸载从这个AssetBundle实例化的所有对象。

`public void Unload(bool unloadAllLoadedObjects); `

内存中的Bundle由于卸载不当，可能造成场景中材质的贴图丢失等问题。

### 更新、修补Bundle

需要两个列表：
- 当前下载的资产包及其版本信息的列表
- 服务器上的资产包及其版本信息的列表

### Unity Asset Bundle 浏览工具
[传送门](https://docs.unity3d.com/Manual/AssetBundles-Browser.html)

