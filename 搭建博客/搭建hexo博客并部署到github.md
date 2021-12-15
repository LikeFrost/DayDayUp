## 搭建 hexo 博客并部署到 github

#### 建立本地项目

- 新建空文件夹
- 右键点击 `git bush here` 
- 安装 hexo 框架：`npm install -g hexo-cli`
- 初始化文件夹：`hexo init`
- 安装依赖：`npm install`
- 启动服务器：`hexo server`



#### 将博客推到远程仓库

- 新建 git 仓库，名称为：<username>.github.io (好处是：通过`https://username.github.io/`就可以访问到你的仓库/blog/project主页, 而不需要在 github.io/后面再加上仓库名)

- 安装插件：`npm install hexo-deployer-git --save`

- 创建一个 SSH key：`ssh-keygen -t -rsa -C "email address"`

- 将密钥添加到 github，验证是否添加成功：`ssh -T git@github.com`

- 在 _config.yml 文件最后修改为：

  ```js
  # Deployment
  ## Docs: https://hexo.io/docs/one-command-deployment
  deploy:
    type: git
    repo: git@github.com:LikeFrost/LikeFrost.github.io.git
    branch: master
  ```

  注意 type: 后面一定要有空格，不然会报错，仓库地址写 ssh 地址（以后提交时不用输用户名密码）

- 推送到 github：`hexo g`   `hexo d`



#### 一些问题

- 推送失败：错误码 443 ，[配置一下 host 文件](https://blog.csdn.net/qq_38076935/article/details/120392154)

- 仓库找不到：重新配置 github 用户名、邮箱等（注意没有下划线）



#### 访问地址

https://likefrost.github.io/