# Git生成多个ssh key

在实际的工作中，有可能需要连接多个远程仓库，例如我想连接私有仓库、GitLab官网、GitHub官网，那么同一台电脑就要生成多个ssh key。

下面总结下步骤：

1. 生成ssh key
```
ssh-keygen -t rsa -C "1592753xxxx@163.com"
```
2. 分别命名，比如`id_rsa_gitlab`与`id_rsa_github`
3. 添加秘钥到SSH Agen
> 因为默认只读取id_rsa
```
ssh-add ~/.ssh/id_rsa_gitlab
```

```
ssh-add ~/.ssh/id_rsa_github
```

4. 在.ssh目录下创建config文件, 文件内容参考
```
# 配置gitlab官网
Host gitlab.com
    HostName gitlab.com
    IdentityFile C:\Users\XiaoQiCoding\.ssh\id_rsa_gitlab
    PreferredAuthentications publickey
    User XiaoQiCoding
    
# 配置github官网
Host github.com                 
    HostName github.com
    IdentityFile C:\Users\XiaoQiCoding\.ssh\id_rsa_github
    PreferredAuthentications publickey
    User XiaoQiCoding
```

5. 到各个仓库中填入公钥

6. 测试连接:
```
ssh -T git@gitlab.com

ssh -T git@github.com
```

---

关注我，一起进步吧~

- [bilibili 打工人小棋](https://space.bilibili.com/302482063?spm_id_from=333.1007.0.0)

- [知乎 打工人小棋](https://www.zhihu.com/people/jin-tian-ye-yao-kai-xin-ya-58-32)

- [CSDN 打工人小棋](https://blog.csdn.net/dagongrenxiaoqi?spm=1000.2115.3001.5343)

![点个赞叭~](https://raw.githubusercontent.com/XiaoQiCoding/Blog/main/Image/Zan1.jpg)


