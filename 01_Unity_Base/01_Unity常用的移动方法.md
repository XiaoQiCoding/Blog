> 在制作一款游戏的时候，经常需要对物体的位置进行移动，我们希望这个移动是具有多样性的，并且可操作的。

C#中提供了非常丰富的移动代码工具，通过这些工具我们可以实现：**匀速移动、变速移动、自定义变速移动**等移动方式。

# 基础框架
实现一个Unity中可以挂接的类包含的基本框架是：
```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class VecMove : MonoBehaviour
{
    private void Start()
    {
        //代码初始时调用
    }

    private void Update()
    {
        //每帧调用一次
    }

    // 本次博文中使用的
    private void OnGUI()
    {
        // 每帧调用一次，用于GUI操作的判断
    }

}
```

== 为了更加直观的展示移动，本次博文在OnGUI中调用移动方法，通过点击移动按钮查看移动过程 ==
> 更一般的情况，可能是在Updat()中调用。
> 注意！下面的方法填充在OnGUI()中。

# 匀速移动
```csharp
if(GUILayout.RepeatButton("匀速移动"))
{
    this.transform.position = Vector3.MoveTowards(objectPos, targetPos, moveFactor);
}
```
> 点击移动按钮即可实现匀速移动,其中objectPos是当前物体的位置，targetPos是目的物体的位置，moveFactor是每一帧移动的距离。

# 变速移动
```csharp
if(GUILayout.RepeatButton("变速移动"))
{
    this.transform.position = Vector3.Lerp(objectPos,targetPos, moveFactor);
}
```
> 从objectPos 移动到 -> targetPos ,每次移动的距离是二者distance*moveFactor的距离。

> 当前物体的位置一直在靠近目标位置，因此这个移动距离不断变小，**并且永远不可达**，只能无限靠近。  

# 自定义变速运动
```csharp
public AnimationCurve curve;        // x轴是移动时间，y轴是移动量
public float x;
if(GUILayout.RepeatButton("自定义变速"))
{
    x += Time.deltaTime / moveDalayTime;
    this.transform.position = Vector3.Lerp(objectPos, targetPos, curve.Evaluate(x));
}
if(GUILayout.RepeatButton("自定义变速2"))
{
    x += Time.deltaTime / moveDalayTime;
    this.transform.position = Vector3.LerpUnclamped(objectPos, targetPos, curve.Evaluate(x));
}
```
> 可以在Unity界面自定义移动曲线 
1. 其中curve设置为public，可以在unity界面进行设置曲线
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200612215106213.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3NzY4OTcx,size_16,color_FFFFFF,t_70)
2.这个曲线如果设置成如下的样子：（超过终点，再回来）
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200612215216799.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3NzY4OTcx,size_16,color_FFFFFF,t_70)
>使用Vector3.Lerp会默认到达目的地为最终结果
>使用Vector3.LerpUnclamped会严格按照曲线运动，超过目的地再回来


# 总结：
> 几种常用的移动方式代码如下：

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

/// <summary>
/// 
/// </summary>
public class VecMove : MonoBehaviour
{
    private Vector3 targetPos = new Vector3(0, 0, 10);
    public float moveFactor = 0.1f;
    public AnimationCurve curve;        // x轴是移动时间，y轴是移动量
    public float moveDalayTime = 1.0f;  // 移动到目的地花费的时间
    private float x;
    private Vector3 objectPos;
    private Vector3 originPos;
    void Start()
    {
        objectPos = this.transform.position;
        originPos = this.transform.position;
    }

    void Update()
    {
    }
    void OnGUI()
    {
        if(GUILayout.RepeatButton("复位"))
        {
            objectPos = this.transform.position;
            this.transform.position = originPos;
            x = 0;
        }
        if(GUILayout.RepeatButton("匀速移动"))
        {
            objectPos = this.transform.position;
            this.transform.position = Vector3.MoveTowards(objectPos, targetPos, moveFactor);
        }
        if(GUILayout.RepeatButton("变速移动"))
        {
            this.transform.position = Vector3.Lerp(objectPos,targetPos,moveFactor);
        }
        // 起点固定，终点固定，运动根据曲线来
        if(GUILayout.RepeatButton("自定义变速"))
        {
            x += Time.deltaTime / moveDalayTime;
            this.transform.position = Vector3.Lerp(objectPos, targetPos, curve.Evaluate(x));
        }
        // 上面的是超过终点就算是终点，如果想要超过终点还能回来，就使用下面的方法
        if(GUILayout.RepeatButton("自定义变速2"))
        {
            x += Time.deltaTime / moveDalayTime;
            this.transform.position = Vector3.LerpUnclamped(objectPos, targetPos, curve.Evaluate(x));
        }
    }
}

```

---
关注我，一起进步吧~

- [bilibili 打工人小棋](https://space.bilibili.com/302482063?spm_id_from=333.1007.0.0)

- [知乎 打工人小棋](https://www.zhihu.com/people/jin-tian-ye-yao-kai-xin-ya-58-32)

- [CSDN 打工人小棋](https://blog.csdn.net/dagongrenxiaoqi?spm=1000.2115.3001.5343)