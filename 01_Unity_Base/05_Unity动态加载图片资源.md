# 前言
本文归纳了Unity中加载图片资源的常用方法，包括url和本地路径的加载。

# www加载

在工具类中封装如下方法：
> 一般是放在单例中，如：GameManager
```cs
public void LoadTexture(Image obj, string path)
{
    // Application.dataPath + "/Resources/Image/Guide1.png"
    string finalPath = Application.dataPath + "/Resources/" + path;
    print("final " + finalPath);
    StartCoroutine(ILoadTexture(obj, finalPath));
}

public void LoadTextureUrl(Image obj, string url)
{
    StartCoroutine(ILoadTexture(obj, url));
}

IEnumerator ILoadTexture(Image obj, string url)
{
    double startTime = (double)Time.time;
    //请求WWW
    WWW www = new WWW(url);

    yield return www;
    if (www != null && string.IsNullOrEmpty(www.error))
    {
        //获取Texture
        Texture2D texture = www.texture;
            //创建Sprite
        Sprite sprite = Sprite.Create(texture, new Rect(0, 0, texture.width, texture.height), new Vector2(0.5f, 0.5f));
        startTime = (double)Time.time - startTime;
        Debug.Log("www加载用时 : " + startTime);
        obj.sprite = sprite;
    }
}
```

## 加载本地图片：
1. 创建一个Image物体

![创建UI.Image物体](https://raw.githubusercontent.com/XiaoQiCoding/Blog/main/01_Unity_Base/Image/LoadTextureImage.png)

2. 在本地存放一张图片，Guide1.png

放置在: `Resources/Image/Guide1.png` 路径下

![放置的位置](https://raw.githubusercontent.com/XiaoQiCoding/Blog/main/01_Unity_Base/Image/LoadTextureGuide1.png)


3. 在物体上挂接一个测试脚本
```cs
using System.Collections;
using System.Collections.Generic;
using System.IO;
using UnityEngine;
using UnityEngine.UI;
 
public class Test : MonoBehaviour {
    void Start () {
        Image img = GetComponent<Image>();
        // 此处调用，上面的LoadTexture方法
        GameManager.instance.LoadTexture(img, "Image/Guide1.png");
	}
}
```

4. 执行前：

![执行前](https://raw.githubusercontent.com/XiaoQiCoding/Blog/main/01_Unity_Base/Image/LoadTextureBefore.png)

5. 执行后：

![执行后](https://raw.githubusercontent.com/XiaoQiCoding/Blog/main/01_Unity_Base/Image/LoadTextureAfter.png)

## 加载url
1. 还是刚刚的Image
2. 脚本变成：
```cs
using System.Collections;
using System.Collections.Generic;
using System.IO;
using UnityEngine;
using UnityEngine.UI;
 
public class Test : MonoBehaviour {
    void Start () {
        Image img = GetComponent<Image>();
        // 此处调用，上面的LoadTexture方法
        GameManager.instance.LoadTextureUrl(img, "https://docs.unity3d.com/uploads/Main/ShadowIntro.png");
	}
}
```
3. 表现效果和上面的一致
4. 缺点是速度比较慢，运行时间大约0.4s

# UnityWebReqeust
> 现在官方建议是采用UnityWebReqeust


参考代码如下
```cs
public IEnumerator LoadTexture2D(string path)
{
    UnityWebRequest request = UnityWebRequestTexture.GetTexture(path);
    yield return request.SendWebRequest();

    if (request.isHttpError || request.isNetworkError)
    {  }
    else
    {
        // 这个是物体身上的组件Image
        Image img = GetComponent<Image>();

        var texture = DownloadHandlerTexture.GetContent(request);
        Sprite sprite = Sprite.Create(texture, new Rect(0, 0, texture.width, texture.height), new Vector2(0.5f, 0.5f));
        img.sprite = sprite;
    }
}
```

## 本地加载
```cs
StartCoroutine(LoadTexture2D(Application.dataPath + "/Resources/Image/Guide1.png"));
```

## url加载
```cs
StartCoroutine(LoadTexture2D("https://docs.unity3d.com/uploads/Main/ShadowIntro.png"));
```

# 以IO的形式加载

```cs
private void LoadByIo(string path)
{
    // 目标
    Image img = GetComponent<Image>();

    double startTime = (double)Time.time;
    //创建文件读取流
    FileStream fileStream = new FileStream(path, FileMode.Open, FileAccess.Read);
    //创建文件长度缓冲区
    byte[] bytes = new byte[fileStream.Length];
    //读取文件
    fileStream.Read(bytes, 0, (int)fileStream.Length);

    //释放文件读取流
    fileStream.Close();
    //释放本机屏幕资源
    fileStream.Dispose();
    fileStream = null;

    //创建Texture
    int width = 300;
    int height = 372;
    Texture2D texture = new Texture2D(width, height);
    texture.LoadImage(bytes);

    //创建Sprite
    Sprite sprite = Sprite.Create(texture, new Rect(0, 0, texture.width, texture.height), new Vector2(0.5f, 0.5f));
    image.sprite = sprite;

    startTime = (double)Time.time - startTime;
    Debug.Log("IO加载" + startTime);

}
```

# 优化方案

前两种方案均使用到了协程，最后一种方案用到了IO。

优劣分析：
- 协程的接口封装更完善，使用更简洁。缺点是协程无法将texture作为返回值return回去。
- IO加载需要自己写读写代码，偏底层一些。优点是可以return，return后交给使用的人自行处理，更加灵活。

下面提出一种协程的优化方案：
> 首先，为什么需要优化？
原因是使用封装的接口，不应该把Image传递进去，也不应该将Image和texture强绑定起来。而是获得目标texture，如何处置则交给使用的人。
因此解决方案是传递一个回调函数，在texture加载完成后执行回调。

参考代码如下：
```cs
public RawImage imageBox;
void Start()
{
    StartCoroutine(LoadTexture2D("/icon.png", (value) => imageBox.texture = value));
}

public IEnumerator LoadTexture2D(string url, Action<Texture2D> taskCompletedCallBack)
{
    UnityWebRequest request = UnityWebRequestTexture.GetTexture(Application.persistentDataPath + url);
    yield return request.SendWebRequest();

    if (request.isHttpError || request.isNetworkError)
    { }
    else
    {
        var texture = DownloadHandlerTexture.GetContent(request);
        taskCompletedCallBack(texture);
    }

}
```

# 其他
> 仅本地加载的办法，类似加载其他资源
## Resources文件夹下加载
```cs
  inside = Resources.Load("inside") as Texture;//Resources夹下动态加载
```

## Assets文件夹下加载
```cs
AssetsManager.Load("Assets/_Sprites/Setting/" + index + ".png", (sprite) =>

        {

            img.sprite = sprite;

        });
```
# 结语

你学废了吗？

---

关注我，一起进步吧~

- [bilibili 打工人小棋](https://space.bilibili.com/302482063?spm_id_from=333.1007.0.0)

- [知乎 打工人小棋](https://www.zhihu.com/people/jin-tian-ye-yao-kai-xin-ya-58-32)

- [CSDN 打工人小棋](https://blog.csdn.net/dagongrenxiaoqi?spm=1000.2115.3001.5343)

![点个赞叭~](https://raw.githubusercontent.com/XiaoQiCoding/Blog/main/Image/Zan3.jpg)



# 参考文档：

- [WWW.LoadImageIntoTexture](https://docs.unity3d.com/ScriptReference/WWW.LoadImageIntoTexture.html)
- [Unity Coroutine返回值如何处理](https://zhuanlan.zhihu.com/p/268533957)
- [Unity中加载图片的几种方法](https://www.jianshu.com/p/3a91a9853846)
- [Unity中加载本地图片](https://blog.csdn.net/qq_38721111/article/details/90711710)
