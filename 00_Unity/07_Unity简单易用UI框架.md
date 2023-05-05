- [前言](#前言)
- [UGUI的困境](#ugui的困境)
- [UI框架介绍](#ui框架介绍)
  - [UIManger](#uimanger)
  - [BasePanel](#basepanel)
  - [界面配置关系](#界面配置关系)
  - [结合](#结合)
  - [最终](#最终)
- [框架使用](#框架使用)
  - [打开界面](#打开界面)
  - [关闭界面](#关闭界面)
- [更多](#更多)


Hi，大家好，我是游戏区“bug主”打工人小棋！

<img src="https://s1.ax1x.com/2023/04/15/p9SLIl8.jpg" width="50%">

# 前言

[qq飞车](https://www.bilibili.com/video/BV12s4y1Y77u/?spm_id_from=333.788&vd_source=be61197043a7c8f4947a73fa5f391444)

开发一款游戏，美术成本是及其高昂的，以我们常见的宣传片CG为例，动辄就要成百上千万的价格。因此这种美术物料，一般只会放在核心剧情节点，引爆舆论，做高潮展示。

[塞尔达](https://www.bilibili.com/video/BV1Zh411M7P7/?spm_id_from=333.337.search-card.all.click&vd_source=be61197043a7c8f4947a73fa5f391444)

而另一种表意方式，则是通过玩法设计，层层引导玩家深入探讨游戏世界，这里需要策划和程序同学耗费极大的精力。

[原神UI]()

[开心消消乐]()

因此对于绝大数独立游戏开发者而言，选择UI来进行剧情展示、玩法交互、核心表达才是最物美价廉的选择。

# UGUI的困境

Unity在4.6版本之后引入自己的界面显示系统，全称：unity Graphical User Interface，即UGUI。

毕竟是亲儿子，这个系统一经推出，就以其灵活、快速、可视化，抢占用户市场，逐渐成为Unity UI的主流系统。

但是他也并不是完美的，对于开发人员来说，使用这套系统往往需要面对如下困境：
1. 缺乏跨场景的UI管理器
2. 界面的上下层关系紊乱
3. 界面之间通信手段贫乏

上述几个问题大大影响到我们的开发效率，因此针对上述问题，我在此分享一套简单易用的UI框架。

这套框架是小棋参考了网上大量资料，并结合个人工作经验，一个代码一个代码敲出来的，绝对是良心干货。

所以，废话少说，点赞留下，代码拿走！

# UI框架介绍

[这里可能要做一个keynote动画说明]()

整套框架分为三大部分：
- UIManager: 跨场景的全局UI管理器
- BasePanel: 所有界面的父类，封装一些通用方法
- 界面配置关系：界面预制件的配置路径

## UIManger

UIManager 作为全局的管理器，我们首先将其设置为单例模式：
> 使用`UIManager.Instance`来获取单例，并使用它。
```cs
public class UIManager
{
    private static UIManager _instance;

    public static UIManager Instance
    {
        get
        {
            if (_instance == null)
            {
                _instance = new UIManager();
            }
            return _instance;
        }
    }
}
```

## BasePanel
BasePanel 是所有界面的父类，他需要继承`MonoBehaviour`以便挂接在物体身上。

它包含两个最重要的方法：
- OpenPanel: 用于打开界面
- ClosePanel: 用于关闭界面

```cs
public class BasePanel : MonoBehaviour
{
    protected bool isRemove = false;
    protected new string name;

    public virtual void SetActive(bool active)
    {
        gameObject.SetActive(active);
    }

    public virtual void OpenPanel(string name)
    {
        this.name = name;
        SetActive(true);
    }

    public virtual void ClosePanel()
    {
        isRemove = true;
        SetActive(false);
        Destroy(gameObject);
    }
}
```

## 界面配置关系

对于Canvas下面的所有界面，我们都将其做成一个个独立的预制件，并放置在合适的路径下。

（比如我放在: `Resources/Prefab/Panel/Menu/.*Panel`）

[![p9U88Xt.png](https://s1.ax1x.com/2023/05/05/p9U88Xt.png)](https://imgse.com/i/p9U88Xt)

随便打开一个界面：`MainMenuPanel`，是长这样的：
[![p9U8Wh4.png](https://s1.ax1x.com/2023/05/05/p9U8Wh4.png)](https://imgse.com/i/p9U8Wh4)

我们将这些路径映射关系配置在`classs UIManager`中, 并使用`class UIConst`来存储界面名称, 方便将来使用：
```cs
public class UIConst
{
    // menu panels
    public const string AllCardPanel = "AllCardPanel";
    public const string MenuPanel = "MenuPanel";
    public const string MainMenuPanel = "MainMenuPanel";
    public const string MenuTipPanel = "MenuTipPanel";
    public const string NewUserPanel = "NewUserPanel";
    public const string UserPanel = "UserPanel";
    public const string ReNameUserPanel = "ReNameUserPanel";
}

public class UIManager
{
    // 路径配置字典
    private Dictionary<string, string> pathDict;

    private UIManager()
    {
        InitDicts();
    }

    private void InitDicts()
    {
        pathDict = new Dictionary<string, string>()
        {
            {UIConst.AllCardPanel, "Menu/AllCardPanel"},
            {UIConst.MenuPanel, "Menu/MenuPanel"},
            {UIConst.MainMenuPanel, "Menu/MainMenuPanel"},
            {UIConst.MenuTipPanel, "Menu/MenuTipPanel"},
            {UIConst.NewUserPanel, "Menu/NewUserPanel"},
            {UIConst.UserPanel, "Menu/UserPanel"},
            {UIConst.ReNameUserPanel, "Menu/ReNameUserPanel"},
        };
    }
}
```

## 结合
配置完路径映射关系后，下面书写`UIManager`中最重要的两个方法：

1. `OpenPanel(string name)`：打开一个界面
2. `ClosePanel(string name)`: 关闭一个界面

要打开一个界面，就要求这个界面有所依托，也就是需要有最终UI的根节点`UIRoot`。

因此我们需要确保每个场景中已经创建好`Canvas`：

并且将其`Canvas Scaler`和`Render Mode`设置为合适的状态。

[![p9UJfyR.png](https://s1.ax1x.com/2023/05/05/p9UJfyR.png)](https://imgse.com/i/p9UJfyR)

在`UIManager`中，使用`UIRoot`拿到当前场景的UI根节点，如果不存在则初始化它：

```cs
public class UIManager
{
    private Transform _uiRoot;

    public Transform UIRoot
    {
        get
        {
            if (_uiRoot == null)
            {
                _uiRoot = GameObject.Find("Canvas").transform;
                return _uiRoot;
            };
            return _uiRoot;
        }
    }
}
```

最后书写`OpenPanel`和`ClosePanel`，这里有两个缓存字典：

- prefabDict: 用于缓存加载过的界面预制件
- panelDict：用于缓存当前打开着的界面，关闭界面时会从这个字典中移除，以此来判断开启状态。

```cs
public class UIManager
{
    // 预制件缓存字典
    private Dictionary<string, GameObject> prefabDict;
    // 已打开界面的缓存字典
    public Dictionary<string, BasePanel> panelDict;

    private void InitDicts()
    {
        prefabDict = new Dictionary<string, GameObject>();
        panelDict = new Dictionary<string, BasePanel>();

        // ...
    }

    public BasePanel OpenPanel(string name)
    {
        BasePanel panel = null;
        // 检查是否已打开
        if (panelDict.TryGetValue(name, out panel))
        {
            Debug.LogError("界面已打开: " + name);
            return null;
        }

        // 检查路径是否配置
        string path = "";
        if (!pathDict.TryGetValue(name, out path))
        {
            Debug.LogError("界面名称错误，或未配置路径: " + name);
            return null;
        }

        // 使用缓存预制件
        GameObject panelPrefab = null;
        if (!prefabDict.TryGetValue(name, out panelPrefab))
        {
            string realPath = "Prefab/Panel/" + path;
            panelPrefab = Resources.Load<GameObject>(realPath) as GameObject;
            prefabDict.Add(name, panelPrefab);
        }

        // 打开界面
        GameObject panelObject = GameObject.Instantiate(panelPrefab, UIRoot, false);
        panel = panelObject.GetComponent<BasePanel>();
        panelDict.Add(name, panel);
        panel.OpenPanel(name);
        return panel;
    }

    public bool ClosePanel(string name)
    {
        BasePanel panel = null;
        if (!panelDict.TryGetValue(name, out panel))
        {
            Debug.LogError("界面未打开: " + name);
            return false;
        }

        panel.ClosePanel();
        return true;
    }
}
```

在`class BasePanel`中，关闭界面时，要将其从`UIManager`的`panelDict`中移除掉，并销毁掉自身。

```cs
public class BasePanel : MonoBehaviour
{

    public virtual void ClosePanel()
    {
        isRemove = true;
        SetActive(false);
        Destroy(gameObject);

        if(UIManager.Instance.panelDict.ContainsKey(name))
        {
            UIManager.Instance.panelDict.Remove(name);
        }
    }
}
```

## 最终

最后贴下代码：

- BasePanel.cs
```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class BasePanel : MonoBehaviour
{
    protected bool isRemove = false;
    protected new string name;
    public virtual void SetActive(bool active)
    {
        gameObject.SetActive(active);
    }

    public virtual void OpenPanel(string name)
    {
        this.name = name;
        SetActive(true);
    }

    public virtual void ClosePanel()
    {
        isRemove = true;
        SetActive(false);
        Destroy(gameObject);
        if(UIManager.Instance.panelDict.ContainsKey(name))
        {
            UIManager.Instance.panelDict.Remove(name);
        }
    }
}

```

- UIManager.cs

```cs
using System.Collections;
using System.Collections.Generic;
using System;
using UnityEngine;
public class UIManager
{
    private static UIManager _instance;
    private Transform _uiRoot;
    // 路径配置字典
    private Dictionary<string, string> pathDict;
    // 预制件缓存字典
    private Dictionary<string, GameObject> prefabDict;
    // 已打开界面的缓存字典
    public Dictionary<string, BasePanel> panelDict;

    public static UIManager Instance
    {
        get
        {
            if (_instance == null)
            {
                _instance = new UIManager();
            }
            return _instance;
        }
    }

    public Transform UIRoot
    {
        get
        {
            if (_uiRoot == null)
            {
                _uiRoot = GameObject.Find("Canvas").transform;
                return _uiRoot;
            };
            return _uiRoot;
        }
    }

    private UIManager()
    {
        InitDicts();
    }

    private void InitDicts()
    {
        prefabDict = new Dictionary<string, GameObject>();
        panelDict = new Dictionary<string, BasePanel>();

        pathDict = new Dictionary<string, string>()
        {
            {UIConst.AllCardPanel, "Menu/AllCardPanel"},
            {UIConst.MenuPanel, "Menu/MenuPanel"},
            {UIConst.MainMenuPanel, "Menu/MainMenuPanel"},
            {UIConst.MenuTipPanel, "Menu/MenuTipPanel"},
            {UIConst.NewUserPanel, "Menu/NewUserPanel"},
            {UIConst.UserPanel, "Menu/UserPanel"},
            {UIConst.ReNameUserPanel, "Menu/ReNameUserPanel"},
        };
    }

    public BasePanel OpenPanel(string name)
    {
        BasePanel panel = null;
        // 检查是否已打开
        if (panelDict.TryGetValue(name, out panel))
        {
            Debug.LogError("界面已打开: " + name);
            return null;
        }

        // 检查路径是否配置
        string path = "";
        if (!pathDict.TryGetValue(name, out path))
        {
            Debug.LogError("界面名称错误，或未配置路径: " + name);
            return null;
        }

        // 使用缓存预制件
        GameObject panelPrefab = null;
        if (!prefabDict.TryGetValue(name, out panelPrefab))
        {
            string realPath = "Prefab/Panel/" + path;
            panelPrefab = Resources.Load<GameObject>(realPath) as GameObject;
            prefabDict.Add(name, panelPrefab);
        }

        // 打开界面
        GameObject panelObject = GameObject.Instantiate(panelPrefab, UIRoot, false);
        panel = panelObject.GetComponent<BasePanel>();
        panelDict.Add(name, panel);
        panel.OpenPanel(name);
        return panel;
    }

    public bool ClosePanel(string name)
    {
        BasePanel panel = null;
        if (!panelDict.TryGetValue(name, out panel))
        {
            Debug.LogError("界面未打开: " + name);
            return false;
        }

        panel.ClosePanel();
        // panelDict.Remove(name);
        return true;
    }

    public void ShowTip(string tip)
    {
        MenuTipPanel menuTipPanel = OpenPanel(UIConst.MenuTipPanel) as MenuTipPanel;
        menuTipPanel.InitTip(Globals.TipCreateOne);
    }
}

public class UIConst
{
    // menu panels
    public const string AllCardPanel = "AllCardPanel";
    public const string MenuPanel = "MenuPanel";
    public const string MainMenuPanel = "MainMenuPanel";
    public const string MenuTipPanel = "MenuTipPanel";
    public const string NewUserPanel = "NewUserPanel";
    public const string UserPanel = "UserPanel";
    public const string ReNameUserPanel = "ReNameUserPanel";
}
```


# 框架使用

下面给大家演示下这套框架的使用方法，事实上在`UIManager`之上，应该还要有个`SceneManager`来管理场景。(**~~这块内容下次再做分享~~**)

## 打开界面

我这里简单举例，比如我这个场景，在Canvas下挂接一个脚本：
```cs
public class GameRoot : MonoBehaviour
{
    void Start()
    {
        UIManager.Instance.OpenPanel(UIConst.MainMenuPanel);
    }
}
```
在场景加载完成之后会调用：

```cs
UIManager.Instance.OpenPanel(UIConst.MainMenuPanel);
```
以此来打开：`MainMenuPanel`

[![p9U8Wh4.png](https://s1.ax1x.com/2023/05/05/p9U8Wh4.png)](https://imgse.com/i/p9U8Wh4)


打开多个界面的情况：
[![p9UUFpD.png](https://s1.ax1x.com/2023/05/05/p9UUFpD.png)](https://imgse.com/i/p9UUFpD)

此时Canvas下子界面的情况：

[![p9UUYBn.png](https://s1.ax1x.com/2023/05/05/p9UUYBn.png)](https://imgse.com/i/p9UUYBn)

## 关闭界面

关闭界面也是类似的：

```cs
// 关闭用户列表界面
UIManager.Instance.ClosePanel(UIConst.UserPanel);
// 关闭重命名界面
UIManager.Instance.ClosePanel(UIConst.ReNameUserPanel);
```


# 更多

UI框架有时候要结合`数据存储`、`广播监听`来适用，会有更加灵活和出彩的效果，具体可以参考我之前的两期视频：

- [【最用心 の Unity百宝箱】6. 游戏中的用户数据存档](https://www.bilibili.com/video/BV1GM4y1y7VL/?spm_id_from=333.999.0.0)
- [【最用心 の Unity百宝箱】7.观察者模式](https://www.bilibili.com/video/BV1gV4y1Z7VV/?spm_id_from=333.999.0.0&vd_source=be61197043a7c8f4947a73fa5f391444)

在下一期视频中，我将在植物大战僵尸课程中，手把手教大家使用这三套框架。

相信看完这些内容，不仅对你的框架使用能力有所帮助，还能帮助你代码设计能力更上一层楼。

不知道有没有课代表看到这儿呀，还有人的话扣个1支持下小棋，视频制作不易，感谢粉丝们的大力支持。

我们下期再见~~~