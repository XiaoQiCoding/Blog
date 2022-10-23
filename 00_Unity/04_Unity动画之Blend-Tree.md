# 材料准备
1. 人物模型和动画
	> 直接去Unity素材库里找，动画可以找可以自己录制
2. Unity编辑器

# 创建Animator
1. 步骤
	> Inspector -> Add Component -> Animator
	![Animator](https://img-blog.csdnimg.cn/20200628230104507.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3NzY4OTcx,size_6,color_FFFFFF,t_70)
2. 主要包含Controller和Avatar
	> Controller 是动画控制器，接下去着重讲解
	> Avatar控制人物的关节等，这里可以为空，想了解可以看官方文档
# 创建Controller
1. 新建Animator Controller
	> 点击鼠标右键，Create-> Animator Controller![在这里插入图片描述](https://img-blog.csdnimg.cn/2020062823064632.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3NzY4OTcx,size_6,color_FFFFFF,t_70)
2. 将创建出来的Animator Controller拖动到Animator的Controller框框中
3. 打开创建出来的 Animator Controller
	> 刚开始只有箭头所示的三个状态，另外两个是自己加的
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200628231243368.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3NzY4OTcx,size_6,color_FFFFFF,t_70)
4. 点击左上角导航栏的Parameters，添加三个float参数
	> ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200628231417295.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3NzY4OTcx,size_6,color_FFFFFF,t_70)
5. 在Base Layer界面中添加两个State：Idle和Run
(其中Idle不需要混合树，设置为Empty，Run需要混合，设置成From New Blend Tree)
	> ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200628231826444.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3NzY4OTcx,size_6,color_FFFFFF,t_70)
6. 设置状态转换
	>这里不细讲了，不懂就去翻文档把
	> 简言之，就是Idle和Run的互相转换条件
	> （1）Idle - > Run的条件：
	![在这里插入图片描述](https://img-blog.csdnimg.cn/20200628232600139.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3NzY4OTcx,size_6,color_FFFFFF,t_70)
	>（2）Run -> Idle的条件：
	>![在这里插入图片描述](https://img-blog.csdnimg.cn/20200628232642248.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3NzY4OTcx,size_6,color_FFFFFF,t_70)
# 配置混合树
1. 双击刚刚创建的混合树Run
2. 点击Blend Tree，选择Blend Type 为 2D Simple Directional ，Parameters两个参数指定为Horizontal 和 Vertical
3. 并在右侧Inspector（点+，选择motion)，添加要混合的移动动画
	>（我这里是前后左右移动）![在这里插入图片描述](https://img-blog.csdnimg.cn/20200628233021914.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3NzY4OTcx,size_6,color_FFFFFF,t_70)
4. 点击图中的蓝色方块，调整对应动画到上下左右对称的位置
	> 如上图所示
5. 拖动上图的红色圆心，可以在Inspactor下方看到玩家移动混合的效果
	> ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200628233425225.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3NzY4OTcx,size_6,color_FFFFFF,t_70)
	
***
至此，混合树界面部分创建完成，接下去为人物添加动画，通过键盘控制人物移动，并观看混合效果。
***

# 脚本代码
```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

/// <summary>
/// 
/// </summary>
public class PlayerMove : MonoBehaviour
{
    public float speed = 3;
    public Animator moveAnimator;

    private void Awake()
    {
        moveAnimator = GetComponent<Animator>();
    }
    private void Update()
    {
        playerMove();
    }

    private void playerMove()
    {
        Vector3 movement = getMovement() * speed * Time.deltaTime;
        float hor = getHorizontal();
        float ver = getVertical();
        moveAnimator.SetFloat("Horizontal", hor);
        moveAnimator.SetFloat("Vertical", ver);
        float moveValue = movement.sqrMagnitude;
        moveAnimator.SetFloat("Movement", moveValue);
        this.transform.position += movement;
    }

    private float getHorizontal()
    {
        float hor = Input.GetAxis("Horizontal");
        return hor;
    }

    private float getVertical()
    {
        float ver = Input.GetAxis("Vertical");
        return ver;
    }

    private Vector3 getMovement()
    {
        float hor = getHorizontal();
        float ver = getVertical();
        Vector3 movement = Vector3.forward * ver + Vector3.right * hor;
        // 做成单位向量，后面使用自定义的speed可以精确控制移动速度
        movement.Normalize();
        return movement;
    }
}

```

# 效果演示
1. 后退 + 左移
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200629000122793.gif)
2. 右移 + 前进
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200629000144489.gif)

---

关注我，一起进步吧~

- [bilibili 打工人小棋](https://space.bilibili.com/302482063?spm_id_from=333.1007.0.0)

- [知乎 打工人小棋](https://www.zhihu.com/people/jin-tian-ye-yao-kai-xin-ya-58-32)

- [CSDN 打工人小棋](https://blog.csdn.net/dagongrenxiaoqi?spm=1000.2115.3001.5343)

