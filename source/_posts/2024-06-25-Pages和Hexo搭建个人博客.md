---
title: 使用GitHub Pages和Hexo搭建个人博客
date: 2024-06-25 20:14:00
tags: 
- 博客
- GitHub Pages
- Hexo
- Vscode
categories:
- 教程

---

使用GitHub Pages与Hexo搭建个人博客
===========

看了许多利用GitHub Pages建立个人博客的例子，使用的比较多的就是**jkeyll**和**Hexo**，最终选择了使用**Hexo**搭建我的博客，遵循先搭建再部署的原则

<!-- more -->

搭建博客
-------

参考[Hexo官方网站](https://hexo.io/zh-cn/)，需要先安装以下应用程序  

- **[Node.js](https://nodejs.org)**  
- **[Git](https://git-scm.com)**  

安装Git
-----

不同的操作系统拥有不同的安装程序  

- Windows一般使用[安装包](https://git-scm.com/download/win)  
- Mac系统可以使用Homebrew或者对应安装程序
- Linux则可以通过apt/yum包管理器来进行安装  
  在控制台中键入  
  `sudo apt-get install git-core`  
  或者  
  `sudo yum install git-core`  

安装Node.js
------

直接访问[Node.js官方提供的下载网址](https://nodejs.org/zh-cn/download/)  
选择你自己的操作系统和版本，按提示操作即可  
这里建议Windows和Mac使用安装程序安装  
Linux使用nvm进行包管理（安装完记得source ~/.bashrc）

安装Hexo并建立项目
---

直接使用npm安装Hexo  

```bash
npm install -g hexo-cli  
```

然后新建一个空目录，用于Hexo的初始化  

```bash
mkdir main
hexo init main
cd main
npm install
```

> 这样你的Hexo项目就建立好了，各文件的作用请参考[文档](https://hexo.io/zh-cn/docs/setup)  
随后就可以为你的Blog选一个主题了，官方提供了一个[主题列表](https://hexo.io/themes/)，你可以在里面挑选自己喜欢的样式  
当然也可以直接Google或者Github搜索，有一些主题应该是没有被收录的  

一般导入主题方法如下  
> 本次采用[Fluid主题](https://github.com/fluid-dev/hexo-theme-fluid)  

**1. 下载解压**  
下载主题文件并解压到themes目录下，将解压出的文件夹命名为主题名  
**2. 修改内容**  
修改Hexo目录中的_config.yml  

```yml
theme：主题名  
language: zh-CN
```

**3.其他事项**  
有些主题需要自己创建部分内容，例如Fluid主题就需要自己创建About页面

尝试本地运行
---------

在项目目录，例如main下输入  

```bash
hexo g
# or
hexo generate
```

该命令会生成对应的静态文件在public中  
控制台中输入  

```bash
hexo s
# or
hexo server
```

即可启动服务  
在浏览器中打开对应网址<http://localhost:4000>即可预览博客  
每次修改博客内容或者配置想要预览的时候，请先使用`hexo clean`清除本地文件，然后重新生成并启动服务  

编写博客
------

使用`hexo new [layout] <title>`就可以新建一个博客页面了。
其中Layout为博客的布局，默认支持post，page，draft三种，储存在scaffolds文件夹下,不同布局的文章会储存在不同的位置下  

| 布局  |      路径      |
| :---: | :------------: |
| post  | source/_posts  |
| page  | source/_pages  |
| draft | source/_drafts |

>通过编辑_config.yaml文件中new_post_name，可以更改默认的文件名称，添加时间标签可以让我们更加方便管理博客,具体的更改方式可以参考[这里](https://hexo.io/zh-cn/docs/writing#%E6%96%87%E4%BB%B6%E5%90%8D%E7%A7%B0)  

添加一篇新的post

``` bash
hexo new post <your_title>
```

随后你的source/_posts文件夹中就会出现这样一个.md文件，依照正常Html或者Markdown格式就可以撰写文章了  

> .md文件上方有用```---```分割的区域
这是文章的扉页，是front-matter提供的功能，可以用来自定义文章的各类变量

| 参数              | 描述                                                                                                                                  | 默认值                                                                  |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------- |
| `layout`          | 布局                                                                                                                                  | [`config.default_layout`](/zh-cn/docs/configuration#%E6%96%87%E7%AB%A0) |
| `title`           | 标题                                                                                                                                  | 文章的文件名                                                            |
| `date`            | 建立日期                                                                                                                              | 文件建立日期                                                            |
| `updated`         | 更新日期                                                                                                                              | 文件更新日期                                                            |
| `comments`        | 开启文章的评论功能                                                                                                                    | `true`                                                                  |
| `tags`            | 标签（不适用于分页）                                                                                                                  |                                                                         |
| `categories`      | 分类（不适用于分页）                                                                                                                  |                                                                         |
| `permalink`       | 覆盖文章的永久链接，永久链接应该以 `/` 或 `.html` 结尾                                                                                | `null`                                                                  |
| `excerpt`         | 纯文本的页面摘要。使用 [该插件](/zh-cn/docs/tag-plugins#%E6%96%87%E7%AB%A0%E6%91%98%E8%A6%81%E5%92%8C%E6%88%AA%E6%96%AD) 来格式化文本 |                                                                         |
| `disableNunjucks` | 启用时禁用 Nunjucks 标签 `{{ }}`/`{% %}` 和 [标签插件](/zh-cn/docs/tag-plugins) 的渲染功能                                            | false                                                                   |
| `lang`            | 设置语言以覆盖 [自动检测](/zh-cn/docs/internationalization#%E8%B7%AF%E5%BE%84)                                                        | 继承自 `_config.yml`                                                    |
| `published`       | 文章是否发布                                                                                                                          | 对于 `_posts` 下的文章为 `true`，对于 `_draft` 下的文章为 `false`       |

有关分类与标签编辑，请参照[这里](https://hexo.io/zh-cn/docs/front-matter#%E5%88%86%E7%B1%BB%E5%92%8C%E6%A0%87%E7%AD%BE)

部署博客
=======

建立博客，编辑完成我们的页面，就可以部署到服务器上，这样就能在互联网上访问我们自己的博客了  
本着免费够用的原则，选择GitHub Pages进行部署

安装部署工具
-------

``` bash
npm install hexo-deployer-git --save
```

一行搞定

设置GitHub Pages
---------

1. 新建一个仓库，命名为```用户名.gitbub.io```  

2. 仓库中会默认有一个index.md文件，我们需要的是仓库的克隆地址，在Clone选项卡中复制即可  

3. 转到仓库设置页面，在Pages选项卡中选择你的页面对应的分支，一般选择默认main分支即可。

4. 建议为仓库和本机配置SSH密钥，可以提高安全性，方便我们修改或上传文档。具体操作清参阅Github[教程](https://docs.github.com/en/authentication/connecting-to-github-with-ssh)

在_config.yml 中修改参数
-------

将deploy项目修改为如下形式

``` yaml
deploy:
    type: git
    repo: https://github.com/<username>/<project>
    branch: main
```

>`repo`是你自己仓库的HTTP或SSH地址，结尾是.git,请不要与浏览器地址混淆  
`branch` 选择你在上面Pages页面里设置的分支  
除此以外，还需要在Git中设置你的name和email，请参考[ProGit](https://git-scm.com/book/en/v2/Getting-Started-First-Time-Git-Setup)，或者直接利用Vscode的源代码管理功能/Git插件来管理Git与Github账户

此时运行  

``` bash
hexo clean
hexo g
hexo d
```

即可将自己的网站部署到对应Github分支，访问你自己的主页`<GitHub 用户名>.github.io`即可查看结果  
部署需要时间，博客的更新并不是部署了就会立即更新，请耐心等待。

Git连接问题
-------

如果你使用代理连接Github，那么很有可能会发生连接失败的问题  
在上述的[SSH配置](#设置github-pages)中，Github官方文档提到了测试SSH连接的操作步骤，请以该步骤的结果作为参考  
如果提示SSH连接失败，那么可能是端口的问题  
在默认的.ssh文件夹中新建文件，键入以下内容

```properties
Host github.com
HostName ssh.github.com
User git
Port 22
```

`Port` 修改为22或者443即可  
将文件重命名为config，SSH连接应该会恢复

写在最后
-------

以上就是我搭建个人博客经历的大致历程。  
写完这篇文章作为博客的第一篇文章，我的心里还是挺开心的，毕竟自己搭起来了一个小博客，以后可以存放我自己的乱七八糟的东西。说不定以后可以买一个域名重定向到这里，让我的文章能被更多的人看到，帮助到更多的人。

2024.6.26  
19:10
