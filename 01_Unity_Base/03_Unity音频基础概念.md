# 音频概述
如果游戏缺少某种音频，无论是背景音乐还是其他音效，那么游戏是不完整的。Unity 的音频系统既灵活而又强大。它可以导入大多数标准音频文件格式，精通于在3D 空间中播放声音，还可根据需要提供其他效果（比如回声和滤波）。Unity 还可以通过用户机器上任何可用麦克风来录制音频，以便在游戏过程中进行使用或存储和传输。


![音频源和倾听者](https://raw.githubusercontent.com/XiaoQiCoding/Blog/main/01_Unity_Base/Image/AudioSourceListDiagram.jpg)





# 基础概念
## 音频剪辑（Audio Clip）
**音频剪辑**包含音频源使用的音频数据

![音频剪辑检视面板 (Inspector)](https://raw.githubusercontent.com/XiaoQiCoding/Blog/main/01_Unity_Base/Image/AudioClipImporter50.jpg)

## 音频监听器（Audio Listener）

![音频剪辑检视面板 (Inspector)](https://raw.githubusercontent.com/XiaoQiCoding/Blog/main/01_Unity_Base/Image/audio_listener_inspector.jpg)


**音频监听器** (Audio Listener) 充当类似于麦克风的设备。

**提示:**
- 它接收来自场景中任何给定音频源的输入，并通过计算机扬声器播放声音。

- 必须添加音频监听器才能工作。默认情况下，音频监听器将被添加到主摄像机。

- 每个场景只能有 1 个音频监听器才能正常工作。

- 要访问项目级别的音频设置，可使用 Audio 窗口（主菜单：Edit > Project Settings，然后选择 Audio 类别）。

# 音频源
**音频源** (Audio Source) 在场景中播放音频剪辑。
![音频源](https://raw.githubusercontent.com/XiaoQiCoding/Blog/main/01_Unity_Base/Image/AudioSourceInspector.jpg)

属性介绍参考：[https://docs.unity3d.com/cn/2021.1/Manual/class-AudioSource.html](https://docs.unity3d.com/cn/2021.1/Manual/class-AudioSource.html)

# 麦克风
**Microphone** 类用于捕获 PC 或移动设备上内置（物理）麦克风的输入。

使用此类，可以从启动和关闭内置麦克风，获取可用音频输入设备（麦克风）列表，获取每个输入设备状态。

没有组件用于麦克风 (Microphone) 类，可通过脚本直接访问。有关更多信息，请参阅脚本参考中的 Microphone 页面。

![麦克风](https://raw.githubusercontent.com/XiaoQiCoding/Blog/main/01_Unity_Base/Image/Microphone-icon.jpg)