- [1. 开头](#1-开头)
- [2. 鸣谢](#2-鸣谢)
- [3. 框架优化探讨](#3-框架优化探讨)
  - [3.1. 每个panel都使用独立的canvas](#31-每个panel都使用独立的canvas)
  - [3.2. Panel打开时可以结合Dotween实现淡入淡出](#32-panel打开时可以结合dotween实现淡入淡出)
  - [3.3. 界面的删除和销毁](#33-界面的删除和销毁)
- [4. Canvas的四大组件介绍](#4-canvas的四大组件介绍)
  - [4.1. Canvas](#41-canvas)
    - [4.1.1. Screen Space - Overlay](#411-screen-space---overlay)
    - [4.1.2. Screen Space - Camera](#412-screen-space---camera)
    - [4.1.3. World Space](#413-world-space)
  - [4.2. Canvas Scaler](#42-canvas-scaler)
  - [4.3. Graphic Raycaster](#43-graphic-raycaster)
  - [4.4. CanvasGroup:](#44-canvasgroup)

# 1. 开头

在上一期视频，我分享了一套简单易用的UI框架，没想到大家的学习热情这么高，第一天就超过2000播放量了。

由此可见，天下苦UI久已！！！

粉丝们爱看的，我就多发点。

这期视频，我们继续深入探讨UI这个话题。

> 本期视频将将分为两大部分展开：

- UI框架优化探讨
- Canvas四件套讲解

> 下期预告
- ~~拼接UI技巧分享~~
- ~~UI框架使用案例~~

# 2. 鸣谢
本期视频内容较多，干货满满，学到就是赚到~

课程开始前，要特别感谢`ViPSkill`对咱频道的大力支持

VipSkill是一家专注于游戏技能培训的在线教育公司，

近期公司推出了：【元气骑士五天集训营】，将于本月15日正式开营。

最最重要的是，这个训练营是完全免费的

通过课程（白嫖）

- 你将学会制作一款简单的弹幕射击游戏，

- 了解到游戏行业的求职情况

- 以及U3D开发学习路径的规划

如果你也对游戏开发感兴趣，或者将来想要入职游戏公司

可以扫描屏幕中的二维码，并回复打工人小棋，加入训练营。

最后，有金主大大的催更，小棋一定勤加更新（搞快点）！

那我们回到正题~

# 3. 框架优化探讨

在上一期视频，许多粉丝都在评论区发表自己的一些看法，其中一些观点提的很好，在这里做下分享。

> 温馨提示，学习本期内容前，建议先浏览上一期：《分享一套简单易用的游戏UI框》

## 3.1. 每个panel都使用独立的canvas

[![p96ucTI.png](https://s1.ax1x.com/2023/05/13/p96ucTI.png)](https://imgse.com/i/p96ucTI)

效果：

[![p96nsI0.png](https://s1.ax1x.com/2023/05/13/p96nsI0.png)](https://imgse.com/i/p96nsI0)

小棋查阅了许多资料，发现这样做确实会更好。

> 原因是UGUI会自动合并批次，原理是他会吧一个Canvas下的所有元素合并再一个Mesh里。

> 如果单一Canvas下的元素很多，任意一个元素发生位置、大小的改变，就需要重新合并Mesh。

> 对单一界面设置Canvas，就可以把合并Mesh的细粒度划分的更小，一个元素的改变，影响的范围也会减小。

因此这里，咱们对框架做一些优化调整。

- 让每个Panel都有一个Canvas
- UIManager中的UIRoot可以不需要了，但是为了方便归纳整理，还是将UIRoot做成一个空物体来收纳所有的Panel。

伪代码如下：
```cs

public class UIManager
{
    public Transform UIRoot
    {
        get
        {
            if (_uiRoot == null)
            {
                if (GameObject.Find("Canvas"))
                {
                    _uiRoot = GameObject.Find("Canvas").transform;
                }
                else
                {
                    _uiRoot = new GameObject("Canvas").transform;
                }
            };
            return _uiRoot;
        }
    }

    public BasePanel OpenPanel(string name)
    {
        // ...

        // 打开界面时还是将其放在UIRoot下，方便整理归纳
        GameObject panelObject = GameObject.Instantiate(panelPrefab, UIRoot, false);

        // ...
    }
}

```

## 3.2. Panel打开时可以结合Dotween实现淡入淡出
[![p96KC7R.png](https://s1.ax1x.com/2023/05/13/p96KC7R.png)](https://imgse.com/i/p96KC7R)

这个想法确实很好，比如我想实现界面打开和关闭时候的淡入淡出，这时候就可以将`Canvas Group`和`DoTween`结合使用：

> Canvas Group 在下一章节将进行讲解，建议每个Panel都添加上这个内置组件

具体代码如下
```cs
    public virtual void OpenPanel(string name)
    {
        // ...

        CanvasGroup canvasGroup = GetComponent<CanvasGroup>();
        canvasGroup.alpha = 0.0f;
        DOTween.To(() => canvasGroup.alpha, x => canvasGroup.alpha=x, 1, 2);

        // ...
    }

    public virtual void ClosePanel()
    {
        isRemove = true;
        CanvasGroup canvasGroup = GetComponent<CanvasGroup>();
        DOTween.To(() => canvasGroup.alpha, x => canvasGroup.alpha = x, 0, 1).onComplete(
            () =>
            {
                SetActive(false);
                Destroy(gameObject);
                if (UIManager.Instance.panelDict.ContainsKey(name))
                {
                    UIManager.Instance.panelDict.Remove(name);
                }
            }
        );
    }
```

## 3.3. 界面的删除和销毁
[![p96KuBd.png](https://s1.ax1x.com/2023/05/13/p96KuBd.png)](https://imgse.com/i/p96KuBd)

在我们的UI框架中，其实已经用到了大量的缓存思想。

比方说界面的预制件缓存，在第一次加载完成之后，以后再次使用就无须加载。

这样的好处是提高了运行效率，但是增加了内存。

界面也是同样的道理，大家可以根据自己项目的实际情况考虑，是否在关闭界面的时候对其进行销毁。

我这里之所以在`ClosePanel`时对界面进行销毁，是考虑到逻辑的简便性。

在日后开发过程中，可能涉及到界面的消息监听，也可能注册一堆乱七八糟的东西，我希望这个界面关闭之后，自己就无须关心那些奇奇怪怪的问题，而是在ClosePanel时随着物体销毁一并注销掉。

当然，还是那句话，写代码是为了解决项目实际问题，所以请根据自己的需求进行调整，好用最重要！！！

# 4. Canvas的四大组件介绍

最后，既然都讲到UI框架了，那就不得不分享下Canvas的四大组件。

毕竟，这里涉及到的UI适配，是所有游戏都绕不过去的话题。

这里吐槽下Unity官方文档，真的写的太干燥了，好歹举几个例子说明下吧，生怕我们学会是吧？

下面的讲解，我会尽可能结合自己的理解，用“人话”说明白这些组件。

## 4.1. Canvas

当我们在Unity中创建UI界面时，Canvas组件是最基本的元素之一。它充当着UI元素的容器，并提供了许多参数用于控制UI的显示行为。

下面是Canvas的三种模式（`Render Mode`）介绍：

### 4.1.1. Screen Space - Overlay

> UI元素将绘制在屏幕最上层，不受摄像机的影响。

[![p962lOU.png](https://s1.ax1x.com/2023/05/13/p962lOU.png)](https://imgse.com/i/p962lOU)

1. Pixel Perfect

    > Pixel Perfect参数用于确保UI元素在不同屏幕分辨率下的像素对齐。
    > 当启用Pixel Perfect后，UI元素的位置和尺寸会按照像素对齐到屏幕上的整数像素。
    > 简单理解就是，开启后，一些像素风的游戏效果会更好。

2. Sort Order

    > 用于控制UI的遮挡顺序，SortOrder越大的，会出现在屏幕的越顶层。

3. Target Display

    > 用于多屏显示，一般用不到

4. Additional Shader Channels

    > 获取或设置创建 Canvas 网格时要使用的附加着色器通道的掩码，一般用不到

下面这张图很好的呈现了这种模式的表现效果，UI始终在立方体物体的前面：

![ScreenSpace](https://docs.unity3d.com/2021.3/Documentation/uploads/Main/CanvasOverlay.png)


### 4.1.2. Screen Space - Camera

> UI元素将绘制在指定的摄像机下，通常用于在3D场景中嵌入2D UI元素。

[![p962GTJ.png](https://s1.ax1x.com/2023/05/13/p962GTJ.png)](https://imgse.com/i/p962GTJ)

1. Pixel Perfect

    > 同上

2. Render Camera

    > 选择渲染的相机，需要创建一个UICamera用于渲染这种UI模式

3. Order in Layer

    > 可以控制相同层级下UI的遮挡顺序

4. Additional Shader Channels
    > 同上

---
总结

在这种模式下，游戏中至少需要两个摄像机，一个是3D摄像机（主摄像机），另一个就是UI摄像机了。我们需先渲染3D摄像机，然后再渲染UI摄像机，这样UI就会盖在3D场景的前面。我们可以设置 UICamera 里的 Depth 值。

下图是我的UICamera设置，Depth在3D摄像机上的设置是-1，在UI摄像机设置的就是0（或者大于-1）了。另外，UICamera需要设置正交摄像机，此时只需在Projection中选择Orthographic即可。Clipping Planes的最小值建议设置成0，不然有时候某些UI会被剔除掉。

[![p96opMn.png](https://s1.ax1x.com/2023/05/13/p96opMn.png)](https://imgse.com/i/p96opMn)


下面这张图很好的呈现了这种模式的表现效果，UI的显示根据相机看到的表现效果进行渲染，如果UICamera的深度比物体深度大，则可能被物体遮挡：

![CameraMode](https://docs.unity3d.com/2021.3/Documentation/uploads/Main/CanvasCamera.png)

### 4.1.3. World Space
[![p962Yk9.png](https://s1.ax1x.com/2023/05/13/p962Yk9.png)](https://imgse.com/i/p962Yk9)

1. Event Camera
    > 定义Canvas的事件系统应该使用的相机。当用户与Canvas进行交互时，该相机将用于确定用户点击的UI元素。
2. Sorting Layer
    > 将Ui分为多个排序层级，层级越高的显示在越上方
3. Order in Layer
    > 在同一个排序层级内，进一步用数字区分遮挡顺序，结合Sorting Layer实现两级排序

下面这张图很好的呈现了这种模式的表现效果，UI和物体使用同一个相机进行渲染，根据z轴大小来判断深度。简单理解就是，把UI当成物体处理。

![WorldSpace](https://docs.unity3d.com/2021.3/Documentation/uploads/Main/CanvasWorldSpace.png)

## 4.2. Canvas Scaler

![CanvasScaler](https://docs.unity3d.com/2021.3/Documentation/uploads/Main/UI_CanvasScalerInspector.png)

> 用于控制Canvas中UI元素的缩放和布局。它可以确保在不同分辨率和屏幕大小下，UI元素在屏幕上的大小和位置始终保持一致

Canvas Scaler也有三种模式:

1. Constant Pixel Size：

2. Scale With Screen Size：

3. Constant Physical Size：

但是这里我不打算一一讲解了，只讲最常用的设置方式：`自适应屏幕`，也就是第二种模式`Scale With Screen Size`
[![p96fKSI.png](https://s1.ax1x.com/2023/05/13/p96fKSI.png)](https://imgse.com/i/p96fKSI)

- Reference Resolution：
    > 以移动平台为例，现在主流的受击大多数是16:9的分辨率比例，不能把分辨率设置得太高，得考虑兼容低配的受击。这里可以设置分辨率1136 x 640。（我这里设置800 x 600是用于植物大战僵尸项目）
- Screen Match Mode：
    > 屏幕相对模式一般设置成`Expand`，表示Canvas下的UI始终保持在屏幕内，当屏幕宽度变窄后，它会整体缩放高度来保持自适应。

    > 你还可以在下拉框选择`始终保持宽度`或`始终保持高度`，这样当分辨率变化时，超出屏幕部分会被裁切掉。

## 4.3. Graphic Raycaster


用于检测UI元素是否被点击或触摸。当使用鼠标或触摸屏幕时，Graphic Raycaster会向场景中的UI元素发射一条射线，以确定被点击或触摸的元素是哪一个

[![p96IvGQ.png](https://s1.ax1x.com/2023/05/13/p96IvGQ.png)](https://imgse.com/i/p96IvGQ)

1. Ignore Reversed Graphics

    > 控制是否忽略逆向的图形。如果启用此选项，则Graphic Raycaster将忽略与其背面朝向的UI元素进行交互。例如，如果你在场景中有一个UI元素，它只有一个正面，那么启用此选项可以确保只有当该元素的正面朝向相机时才会检测到点击或触摸。

2. Blocking Objects

    > 控制哪些对象会被忽略，以确保射线只与UI元素进行交互。
    > 这个参数的下拉框中，可以选择忽略`2d物体`/`3d物体`，这样当UI前面存在这些物体时，射线也能够到达UI。

3. Blocking Mask

    > 和上面的选项类似，只是这次屏蔽的是Mash，而不是Objects了。

## 4.4. CanvasGroup:

[![p9ydYq0.png](https://s1.ax1x.com/2023/05/12/p9ydYq0.png)](https://imgse.com/i/p9ydYq0)

官方的解释是：用于控制`整个UI组`的`某些方面`的元素，而`不需要单独处理`他们。

所以我的理解，这个组件就是用于控制Canvas以下所有UI元素的一些特征，比如UI的透明度、UI的交互等等。

他包含四个参数：

- Alpha：
    > 控制整个画布组的透明度，参数范围 `[0-1]`
- interactable：
  > 控制按钮的交互，设置为False后，你的按钮将无法点击。并且所有的按钮会呈现半透明的状态（标识其无法交互）。
- Blocks Raycasts:
  > 控制点触穿透，如果在鼠标所在位置有两个按钮，顶层的UI会拦截掉这次点触，以至于底层的UI不响应本次点触。是否穿透或拦截，由这个参数来控制。

    > 比如，控制`"更改用户按钮"`是否被`"用户列表界面"`点触拦截：[![p96luB4.png](https://s1.ax1x.com/2023/05/13/p96luB4.png)](https://imgse.com/i/p96luB4)
    > 若勾选上`Blocks Raycasts`，则按钮将接收不到用户点触事件。

- Ignore Parent Groups：
    > 忽略`父CanvasGroup`的影响，这个很好理解，我们可以在一个Panel的不同位置添加`CanvasGroup`，我们想让当前位置的`CanvasGroup`不受父节点`CanvasGroup`的影响，就需要勾选这个选项。
