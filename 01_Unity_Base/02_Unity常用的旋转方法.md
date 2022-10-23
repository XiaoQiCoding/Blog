#  Unity的几种旋转方法
>  接上一节的移动方法，这里列举几种常用的旋转方法
>  因为和移动用到的工具原理基本类似，不在赘述，感兴趣的同学可以自己将代码挂接在物体上实验
```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

/// <summary>
/// 
/// </summary>
public class QuternionRotate : MonoBehaviour
{
    public Transform targetTF;

    private void OnGUI()
    {
        if(GUILayout.Button("让摄像机转向目标物体"))
        {
            Vector3 direction = targetTF.position - this.transform.position;
            this.transform.rotation = Quaternion.LookRotation(direction);
        }
        if(GUILayout.RepeatButton("使用Lerp控制旋转速度"))
        {
            Quaternion dir = Quaternion.LookRotation(targetTF.position - this.transform.position);
            this.transform.rotation = Quaternion.Lerp(this.transform.rotation, dir, 0.01f);
        }
        if(GUILayout.RepeatButton("匀速旋转"))
        {
            Quaternion dir = Quaternion.LookRotation(targetTF.position - this.transform.position);
            this.transform.rotation = Quaternion.RotateTowards(this.transform.rotation,dir,1f );
        }
        if(GUILayout.RepeatButton("right"))
        {
            this.transform.right = targetTF.position - this.transform.position;
            // Quaternion dir = Quaternion.LookRotation(targetTF.position - this.transform.position);
            // this.transform.rotation = dir;
        }
        if(GUILayout.RepeatButton("right2"))
        {
            Vector3 dir = targetTF.position - this.transform.position;
            this.transform.rotation = Quaternion.FromToRotation(Vector3.right, dir);
        }
    }


}
```

# 总结
常用的的旋转方式有如下几种：

- Quaternion.LookRotation
- Quaternion.Lerp
- Quaternion.RotateTowards
- transform.right/up/forward = ...
- Quaternion.FromToRotation

---
关注我，一起进步吧~
- [bilibili 打工人小棋](https://space.bilibili.com/302482063?spm_id_from=333.1007.0.0)

- [知乎 打工人小棋](https://www.zhihu.com/people/jin-tian-ye-yao-kai-xin-ya-58-32)

- [CSDN 打工人小棋](https://blog.csdn.net/dagongrenxiaoqi?spm=1000.2115.3001.5343)