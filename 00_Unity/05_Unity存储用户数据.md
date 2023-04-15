Hi 大家好，我是游戏区Bug打工人小棋。

在游戏开发过程中，我们经常有存储用户数据的这一需求，比方说：游戏音量、关卡进度、任务进度等等。
[![p9SO736.png](https://s1.ax1x.com/2023/04/15/p9SO736.png)](https://imgse.com/i/p9SO736)

在联网游戏中，往往会把一些用户核心资产信息存储在服务器端，等到用户登录时由服务器下发给用户进行初始化。而单机游戏则往往更加简单，只需要将这些数据序列化保存在本地即可（文本形式）。

今天小棋给大家分享一套简单易用的本地存储框架，希望对同学们有所帮助。

# 框架设计
我们首先定义一个用户数据类：`UserData`，他包含用户基础信息：`姓名`和`等级`。
```c#
public class UserData
{
    public string name;
    public int level;
}
```


接着创建`LocalConfig.cs`，这是个管理类，专门用于管理本地化数据。

这里主要做两件事：
1. 将内存中的用户数据进行序列化，以文本格式保存在本地
2. 将文本格式从硬盘中读取出来，反序列化为内存中的数据

分别对应代码中的：
1. SaveUserData
2. LoadUserData

```c#
public class LocalConfig
{
    public static void SaveUserData(UserData userData)
    {
        // 保存用户数据为文本
    }

    public static UserData LoadUserData(string userName)
    {
        // 读取用户数据到内存
    }
}
```

# 工具选用
在填写上述代码之前，我们需要先做一些调研和准备工作。

1. 首先解决第一个问题：存取数据，存在哪？从哪取？

Unity中为我们提供了许多特殊文件路径，经过我与`ChatGPT`一分钟的愉快交流后，我了解到Unity中的`PersistentDataPath`是一个不错的选择。

这个路径是一个可读写的目录，符合我们的基本需求。根据官方文档的描述，不同平台下他对应的路径有所不同，但是我们可以直接通过Unity为我们提供的：`Application.persistentDataPath`来获取到最终路径，非常方便。
[![p9SvEhn.png](https://s1.ax1x.com/2023/04/15/p9SvEhn.png)](https://imgse.com/i/p9SvEhn)


2. 接下去是第二个问题：如何进行序列化和反序列化

这个名字对于初学者可能有些绕口，但其实非常简单，用通俗的话讲就是：
把C#数据转化为文本，以及将文本转化为C#代码。

业界对于序列化已经有非常成熟的方案，比如json、bson等等，你甚至可以自己写一套序列化和反序列化框架。

本节课程我们选用json，它支持数字、字符串、列表、字典等数据结构，对于大多数游戏来说已经完全够用了。
[![p9SzteO.png](https://s1.ax1x.com/2023/04/15/p9SzteO.png)](https://imgse.com/i/p9SzteO)

3. 最后一个问题：选用哪个Json框架

虽然官方推荐使用`JsonUtility`，但是这里我并不打算使用他。原因是他仅支持能显示在inspector窗口中的数据格式，也就是说字典、以及一些嵌套的数据结构都无法使用，这会给我们带来很多麻烦。

[![p9Szkzq.png](https://s1.ax1x.com/2023/04/15/p9Szkzq.png)](https://imgse.com/i/p9Szkzq)

因此我最终选用的框架是：`using Newtonsoft.Json;`

在使用上只需要关注两个方法：

- 序列化：`JsonConvert.SerializeObject`
- 反序列化：`JsonConvert.DeserializeObject`

非常简单易用。

# 逻辑书写
最后让我们来书写存取框架的具体逻辑叭~
```c#
// 用于文件读写
using System.IO;
// 用于json序列化和反序列化
using Newtonsoft.Json;
// Application.persistentDataPath配置在这里
using UnityEngine;

public class LocalConfig
{

    // 保存用户数据为文本
    public static void SaveUserData(UserData userData)
    {
        // 在persistentDataPath下再创建一个/users文件夹，方便管理
        if (!File.Exists(Application.persistentDataPath + "/users"))
        {
            System.IO.Directory.CreateDirectory(Application.persistentDataPath + "/users");
        }
        // 转换用户数据为JSON字符串
        string jsonData = JsonConvert.SerializeObject(userData);
        // 将JSON字符串写入文件中(文件名为userData.name)
        File.WriteAllText(Application.persistentDataPath + string.Format("/users/{0}.json", userData.name), jsonData);
    }

    // 读取用户数据到内存
    public static UserData LoadUserData(string userName)
    {
        string path = Application.persistentDataPath + string.Format("/users/{0}.json", userName);
        // 检查用户配置文件是否存在
        if (File.Exists(path))
        {
            // 从文本文件中加载JSON字符串
            string jsonData = File.ReadAllText(path);
            // 将JSON字符串转换为用户内存数据
            UserData userData = JsonConvert.DeserializeObject<UserData>(jsonData);
            usersData[userName] = userData;
            return userData;
        }
        else
        {
            return null;
        }
    }
}
```