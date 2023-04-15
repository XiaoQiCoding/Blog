- [框架设计](#框架设计)
- [工具选用](#工具选用)
- [逻辑书写](#逻辑书写)
- [框架使用](#框架使用)
- [框架优化](#框架优化)
- [数据加密](#数据加密)
- [总结](#总结)
- [最后](#最后)


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

# 框架使用
下面我们来验证下这套框架的使用效果，这里我书写了两个GM指令。

此处使用到了`MenuItem`这个特性，帮助我们在编辑器窗口生成快捷按钮。

[![p9poL5V.png](https://s1.ax1x.com/2023/04/15/p9poL5V.png)](https://imgse.com/i/p9poL5V)

- `SaveLocalConfig`: 保存名称为`xiaoqi0-4`的用户数据
- `GetLocalConfig`: 读取名称为`xiaoqi0-4`的用户数据，并打印数据



```cs
using UnityEditor;
using UnityEngine;

class GMCmd
{
    [MenuItem("GMCmd/SaveLocalConf")]
    public static void SaveLocalConfig()
    {
        for (int i = 0; i < 5; i++)
        {
            UserData userData = new UserData();
            userData.name = "xiaoqi" + i.ToString();
            userData.level = i;
            LocalConfig.SaveUserData(userData);
        }
        Debug.Log("Save End!!!!!!!!!!!!");
    }

    [MenuItem("GMCmd/GetLocalConfig")]
    public static void GetLocalConfig()
    {
        for (int i = 0; i < 5; i++)
        {
            string name = "xiaoqi" + i.ToString();
            UserData userData = LocalConfig.LoadUserData(name);
            Debug.Log(userData.name);
            Debug.Log(userData.test);
        }
    }

}
```

1. 点击保存数据：`SaveLocalConf`

[![p9poL5V.png](https://s1.ax1x.com/2023/04/15/p9poL5V.png)](https://imgse.com/i/p9poL5V)

根据[官方文档](https://docs.unity3d.com/ScriptReference/Application-persistentDataPath.html)的说明，我们知道在 **windows** 上`Application.persistentDataPath`对应：

`Windows Store Apps: Application.persistentDataPath points to C:\Users\<user>\AppData\LocalLow\<company name>.`

而这个`<company name>`在 projecting setting 中可以找到：
[![p9pT3Pf.png](https://s1.ax1x.com/2023/04/15/p9pT3Pf.png)](https://imgse.com/i/p9pT3Pf)

因此我最终找到我保存的Json文件在这里：[![p9pTJxg.png](https://s1.ax1x.com/2023/04/15/p9pTJxg.png)](https://imgse.com/i/p9pTJxg)

文本内容也与我们预期的一致：
```json
{"name":"xiaoqi0","level":0,"test":{}}
```

2. 点击读取数据：`GetLocalConfig`

[![p9poL5V.png](https://s1.ax1x.com/2023/04/15/p9poL5V.png)](https://imgse.com/i/p9poL5V)

最终结果如下：

[![p9pTWZR.png](https://s1.ax1x.com/2023/04/15/p9pTWZR.png)](https://imgse.com/i/p9pTWZR)

和预期效果一致~

至此存取框架的主体部分已经完成，下面我们对这套框架进行优化。


# 框架优化

由于IO操作涉及到硬盘读写，性能较慢，我们可以对已经读取过的数据进行缓存。

```c#
// 修改0：新增引用命名空间
using System.Collections.Generic;

public class LocalConfig
{
    // 修改1：增加usersData放在内存中
    public static Dictionary<string, UserData> usersData = new Dictionary<string, UserData>();

    public static UserData LoadUserData(string userName)
    {
        // 修改2：读取时，如果userData已经存在，就直接使用
        if (usersData.ContainsKey(userName))
        {
            return usersData[userName];
        }

        string path = Application.persistentDataPath + string.Format("/users/{0}.json", userName);
        if (File.Exists(path))
        {
            string jsonData = File.ReadAllText(path);
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

# 数据加密

一些聪明的玩家，可以根据我们保存的json字段猜测其语义，比如直接修改`level=100`，这样无异于开挂，因此对于上线的游戏，我们还需要对数据进行加密处理。

这里我演示下最简单的一种亦或加密法。

首先介绍下亦或操作，简单理解就是：
- 当两次输入不同时得到结果为1，即正确
- 当两次输入相同时得到结果为0，即错误

|输入|运算符|输入|结果|
|--|--|--|--|
|1|^|1|0|
|1|^|0|1|
|0|^|1|1|
|0|^|0|0|

这个运算符有个特性，就是对任意输入进行两次相同的亦或，会复原结果。

比如：
> 假设一个数为 0
- 第一步：0^1 = 1
- 第二步：1^1 = 0 （复原结果）

> 建设一个数为 1
- 第一步：1^1 = 0
- 第二步：0^1 = 1 （复原结果）

利用这个性质，我们可以将保存的文本文件进行首次亦或，得到乱码数据，这样玩家看到的就是乱码。

然后当我们读取数据的时候再次进行一次亦或，即可复原数据，这样我们看到的就是正确数据。

废话不多说，上代码：

```c#
public class LocalConfig
{
    // 随便选取一些用于亦或的字符（看自己喜欢：注意保密）
    public static char[] keyChars = { 'a', 'b', 'c', 'd', 'e' };
    // 加密
    public static string Encrypt(string data)
    {
        char[] dataChars = data.ToCharArray();
        for (int i = 0; i < dataChars.Length; i++)
        {
            char dataChar = dataChars[i];
            char keyChar = keyChars[i % keyChars.Length];
            // 重点：通过亦或得到新的字符
            char newChar = (char)(dataChar ^ keyChar);
            dataChars[i] = newChar;
        }
        return new string(dataChars);
    }

    // 解密
    public static string Decrypt(string data)
    {
        // 两次亦或执行的是同样的操作
        return Encrypt(data);
    }

    // 修改：存数据的时候进行第一次亦或
    public static void SaveUserData(UserData userData)
    {
        // ...
        string jsonData = JsonConvert.SerializeObject(userData);
        jsonData = Encrypt(jsonData);
        // ...
    }

    // 修改：存数据的时候进行第二次亦或（复原数据）
    public static UserData LoadUserData(string userName)
    {
       
        // ...
        if (File.Exists(path))
        {
            string jsonData = File.ReadAllText(path);
            jsonData = Decrypt(jsonData);
            // ...
        }
        // ...
    }
}
```


测试结果如下：

1. 保存数据到本地

> 可以看到保存后的数据是乱码，玩家再也没办法开挂了!!!


[![p9pHDvF.png](https://s1.ax1x.com/2023/04/15/p9pHDvF.png)](https://imgse.com/i/p9pHDvF)

2. 读取数据到内存

> 可以看到最后读取的数据复原成功了


[![p9pTWZR.png](https://s1.ax1x.com/2023/04/15/p9pTWZR.png)](https://imgse.com/i/p9pTWZR)


# 总结

本文通过框架设计、工具选用、逻辑书写、框架使用、框架优化、数据加密这六部分内容，层层剖析，向大家介绍了一种简单易用的本地化存取框架，希望能对大家有所帮助。

最后将成果代码贴出来，由于还没有经过项目实践，仅仅是理论分享，如果代码有疏漏欢迎交流指正。

1. 框架部分
```c#
using System.IO;
using Newtonsoft.Json;
using UnityEngine;
using System.Collections.Generic;
// Define a class to handle local data storage
public class LocalConfig
{
    public static char[] keyChars = { 'a', 'b', 'c', 'd', 'e' };
    public static Dictionary<string, UserData> usersData = new Dictionary<string, UserData>();
    // Define a method to save user data locally
    public static void SaveUserData(UserData userData)
    {
        if (!File.Exists(Application.persistentDataPath + "/users"))
        {
            System.IO.Directory.CreateDirectory(Application.persistentDataPath + "/users");
        }
        // Convert the user data to a JSON string
        string jsonData = JsonConvert.SerializeObject(userData);
#if !UNITY_EDITOR
        jsonData = Encrypt(jsonData);
#endif
        // Save the JSON string to a file
        File.WriteAllText(Application.persistentDataPath + string.Format("/users/{0}.json", userData.name), jsonData);
    }

    // Define a method to load user data from local storage
    public static UserData LoadUserData(string userName)
    {
        if (usersData.ContainsKey(userName))
        {
            return usersData[userName];
        }

        // Check if the user data file exists
        string path = Application.persistentDataPath + string.Format("/users/{0}.json", userName);
        if (File.Exists(path))
        {
            // Load the JSON string from the file
            string jsonData = File.ReadAllText(path);
#if UNITY_EDITOR
            jsonData = Decrypt(jsonData);
#endif
            // Convert the JSON string to user data object
            UserData userData = JsonConvert.DeserializeObject<UserData>(jsonData);
            usersData[userName] = userData;
            return userData;
        }
        else
        {
            // If the user data file does not exist, return null
            return null;
        }
    }

    public static string Encrypt(string data)
    {
        char[] dataChars = data.ToCharArray();
        for (int i = 0; i < dataChars.Length; i++)
        {
            char dataChar = dataChars[i];
            char keyChar = keyChars[i % keyChars.Length];
            char newChar = (char)(dataChar ^ keyChar);
            dataChars[i] = newChar;
        }
        return new string(dataChars);
    }

    public static string Decrypt(string data)
    {
        return Encrypt(data);
    }
}

// Define a class to represent user data
public class UserData
{
    public string name;
    public int level;
}

```

2. 使用案例

```c#
using UnityEditor;
using UnityEngine;

class GMCmd
{
    [MenuItem("GMCmd/SaveLocalConf")]
    public static void SaveLocalConfig()
    {
        for (int i = 0; i < 5; i++)
        {
            UserData userData = new UserData();
            userData.name = "xiaoqi" + i.ToString();
            userData.level = i;
            LocalConfig.SaveUserData(userData);
        }
        Debug.Log("Save End!!!!!!!!!!!!");
    }

    [MenuItem("GMCmd/GetLocalConfig")]
    public static void GetLocalConfig()
    {
        for (int i = 0; i < 5; i++)
        {
            string name = "xiaoqi" + i.ToString();
            UserData userData = LocalConfig.LoadUserData(name);
            Debug.Log(userData.name);
            Debug.Log(userData.level);
        }
    }

}
```

3. 最终路径
> 参考官方文档

[https://docs.unity3d.com/ScriptReference/Application-persistentDataPath.html](https://docs.unity3d.com/ScriptReference/Application-persistentDataPath.html)

4. tips

在编辑器模式下，我们不需要对数据进行加密解密，这会影响到我们的开发效率，可以使用`UNITY_EDITOR`这个宏进行判断，具体逻辑参考上文代码。

```c#
#if UNITY_EDITOR
            jsonData = Decrypt(jsonData);
#endif
```


# 最后

本文洋洋洒洒写了一万两千多字，绝对是干货中的干货，希望对大家有所帮助，请大家多多点赞收藏评论，你的支持是小棋最大的动力。

也欢迎同学们持续关注：

- [bilibili 打工人小棋](https://space.bilibili.com/302482063?spm_id_from=333.1007.0.0)

- [知乎 打工人小棋](https://www.zhihu.com/people/jin-tian-ye-yao-kai-xin-ya-58-32)

- [CSDN 打工人小棋](https://blog.csdn.net/dagongrenxiaoqi?spm=1000.2115.3001.5343)


[![p9pbbyF.jpg](https://s1.ax1x.com/2023/04/15/p9pbbyF.jpg)](https://imgse.com/i/p9pbbyF)


加油 ：）