# 介绍
本文共分为两部分内容，第一部分介绍了Unity音频的常用接口，第二部分对这些接口进行二次封装，提炼出一个通用框架，方便大家使用，欢迎抄作业。

# 通用接口
官方文档： [https://docs.unity3d.com/ScriptReference/AudioSource.html](https://docs.unity3d.com/ScriptReference/AudioSource.html)

下面仅罗列出一些常用的接口

```c#
    // 停止播放
    audioSource.Stop();
    // 开始播放
    audioSource.Play();
    // 暂停播放
    audioSource.Pause();
    // 设置audioSources的音频源
    audioSource.clip = clip;
    // 在一个audioSource上播放多个声音
    audioSource.PlayOneShot(clip);
    // 设置音量大小
    audioSource.volume = volume;
```

# 通用框架

## 准备
1. 创建一个全局管理器GameManager.cs
2. GameManager设置为单例，方便使用
3. 在GameManager身上挂一个AudioSource，用来播放通用音频

## 实现逻辑

> 具体实现逻辑，附注释: 

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.SceneManagement;

public class GameManager : MonoBehaviour
{
    // GameManager 身上挂一个: audioSource
    private AudioSource audioSource;
    // GameManager 设置为单例
    private GameManager instance;
    // dictAudio 用来缓存已经加载过的音频源
    private Dictionary<string, AudioClip> dictAudio;
    void Awake()
    {
        instance = this;
    }

    void Start()
    {
        dictAudio = new Dictionary<string, AudioClip>();
        audioSource = GetComponent<AudioSource>();
    }
    
    // 辅助函数：加载音频，需要确保音频文件放在Resources文件夹下
    public AudioClip LoadAudio(string path)
    {
        return (AudioClip)Resources.Load(path);
    }

    // 辅助函数：查看下dictAudio是否缓存过该音频，已经有的不需要重新加载
    private AudioClip GetAudio(string name)
    {
        if (!dictAudio.ContainsKey(name))
        {
            dictAudio[name] = LoadAudio(name);
        }
        return dictAudio[name];
    }

    // 播放背景音乐(可以设置音量)
    public void PlayBGM(string name, float volume = 1.0f)
    {
        audioSource.Stop();
        audioSource.clip = GetAudio(name);
        audioSource.Play();
    }

    // 用物体身上的audioSource播放声音碎片(可以设置音量)
    public void PlaySound(AudioSource audioSource, string path, float volume = 1.0f)
    {
        audioSource.PlayOneShot(LoadAudio(path));
        audioSource.volume = volume;
    }

    // 用GameManager身上的audioSource播放声音碎片(可以设置音量)
    public void PlaySound(string path, float volume = 1.0f)
    {
        this.audioSource.PlayOneShot(LoadAudio(path));
        this.audioSource.volume = volume;
    }
}
```

## 使用案例
假设Resources文件夹下面有三个音频文件：
1. bgm : 背景音乐
2. common ： 通用声音
3. sound : 物体发出的声音，比如：鸡你太美


```cs
// 播放背景音乐
GameManager.instance.playBGM("bgm", 1)
// 播放通用声音
GameManager.instance.PlaySound("common", 1)
// 播放物体身上的声音
// 假设物体是: kunObject
GameManager.instance.PlaySound(kunObject.getComponent(AudioSource), "common", 1)
```

# 结语

你学废了吗？

---

关注我，一起进步吧~

- [bilibili 打工人小棋](https://space.bilibili.com/302482063?spm_id_from=333.1007.0.0)

- [知乎 打工人小棋](https://www.zhihu.com/people/jin-tian-ye-yao-kai-xin-ya-58-32)

- [CSDN 打工人小棋](https://blog.csdn.net/dagongrenxiaoqi?spm=1000.2115.3001.5343)


![点个赞叭~](https://raw.githubusercontent.com/XiaoQiCoding/Blog/main/Image/ZanG2.gif)

