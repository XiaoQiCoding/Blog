# 前言
Unity提供了OnMouseDown，OnMouseEnter，OnMouseExit等方法，这些方法可以很方便的帮助我们处理鼠标的时间响应。

但是需要注意他的生效条件，最近我在制作视频课程的时候就遇到了点击不生效的问题：
[【植物大战僵尸】手把手教你做游戏——8. 阳光拾取 + 僵尸生成](https://www.bilibili.com/video/BV1284y1z7kr/?spm_id_from=333.999.0.0)，具体表现是点击阳光时会被僵尸遮挡，导致阳光拾取不生效。

因此今天就来总结下鼠标响应事件的生效条件。

# 条件1：类需要继承MonoBehaviour
若要使用OnMouseDown方法，首要条件是确保继承MonoBehaviour
```cs
public class MyObject : MonoBehaviour{
    void Start(){}
    void Update(){}
    private void OnMouseDown(){
        // ...
    }
}

```

# 条件2：具有 Collider 或 GUIElement
脚本绑定的物品中，需要额外增加碰撞体组件或具有GUIElement元素，比如2D游戏中的物体就可以绑定如下这些组件：

- BoxCollider 2D （矩形碰撞体）
- CircleCollider 2D （圆形碰撞体）
- CapsuleCollider 2D （椭圆碰撞体）
- PolygonCollider 2D （多边形碰撞体）

# 条件3：Layer 非 Ignore Raycast
OnMouseDown原理为射线射到屏幕上，检测到第一个触碰的物品。如果物品的Layer层为Ignore Raycast，则不调用该函数。
（所以也需要注意，若存在UI元素在绑定该脚本的物品上方，则易使得该GUI事件失效。）

![在Unity中选择Layer类型](https://github.com/IMUHERO/Blog/blob/main/Image/01_1.jpg)

# 条件4：Trigger -> Physics.queriesHitTriggers
如果碰撞体设置为Trigger属性（勾选isTrigger），则需要确保Queries Hit Triggers已被勾选。该参数表示：指定默认情况下查询（射线投射、球形投射、重叠测试等）是否命中触发器。

通过Edit——Project Settings——Physics（Physics 2D）修改对应的参数。


![在Unity修改参数](https://github.com/IMUHERO/Blog/blob/main/Image/01_2.jpg)

# 其他：鼠标事件被其他Collider遮挡问题处理
当一个Collider被别的Collider挡住，被挡的Collider因为射线投射不到，无法触发鼠标响应方法。

在[官方API](https://docs.unity3d.com/ScriptReference/MonoBehaviour.OnMouseEnter.html)中可以看到一句话，OnMouseEnter：This function is not called on objects that belong to Ignore Raycast layer.

因此将不需要触发OnMouseEnter的Collider游戏对象的Layer设置为 Ignore Raycast layer即可解决。

# 参考文档：
- [Unity事件函数OnMouseDown生效条件](https://juejin.cn/post/7105744886464249870)
- [Unity 被遮挡Collider如何触发OnMouseEnter事件](https://blog.csdn.net/sinat_24229853/article/details/42835313)