## Hexo 集成 Gitalk 评论系统

自己的博客搭建完成后，想要开启评论功能，自己造轮子又比较麻烦，倒不如嫖来 Github 提供的评论系统——Gitalk

Gitalk 有如下特性：

- 使用 Github 登录，无需再次注册账号；
- 支持多语言；
- 基于 Github Issue 开发，评论直接提交到指定仓库的 Issue 中

使用方式：

- 在 [Github](https://github.com/settings/applications/new) 上注册应用

![image-20211215215539259](https://raw.githubusercontent.com/LikeFrost/comment/main/img/image-20211215215539259.png)

- 保存生成的 id 及 密钥

![image-20211215215930128](https://raw.githubusercontent.com/LikeFrost/comment/main/img/image-20211215215930128.png)

- 申请一个新的仓库作为评论仓库（评论会在 Issue 中出现），此处仓库名为 comment。
- 在 hexo 主题下的 _comfig.yml 文件中配置：

```yml
gitalk:
  enable: true
  githubID: #GitHub 用户名
  clientID: #刚才得到的 id
  clientSecret: #刚才得到的密钥
  repo: #新建的仓库名称，用来放置评论
  owner: # 仓库主人
  admin: # 仓库主人
  language: zh-CN # en, zh-CN, zh-TW, es-ES, fr, ru, de, pl and ko
```

- layout 文件夹中的 script.ejs 文件中添加：

```ejs
<script>
    const gitalk = new Gitalk({
        clientID: '<%- theme.gitalk.clientID %>',
        clientSecret: '<%- theme.gitalk.clientSecret %>',
        repo: '<%- theme.gitalk.repo %>',      // The repository of store comments,
        owner: '<%- theme.gitalk.owner %>',
        admin: ['<%- theme.gitalk.admin %>'],
        language: '<%- theme.gitalk.language %>',
        id: decodeURI(location.pathname),      // Ensure uniqueness and length less than 50
        distractionFreeMode: true  // Facebook-like distraction free mode
    })
    gitalk.render('gitalk-container')
</script>
```



预览效果如下：

![image-20211215220909778](https://raw.githubusercontent.com/LikeFrost/comment/main/img/image-20211215220909778.png)

评论后的效果：

![image-20211215220946177](https://raw.githubusercontent.com/LikeFrost/comment/main/img/image-20211215220946177.png)



GitHub 仓库中显示：

![image-20211215221047035](https://raw.githubusercontent.com/LikeFrost/comment/main/img/image-20211215221047035.png)
