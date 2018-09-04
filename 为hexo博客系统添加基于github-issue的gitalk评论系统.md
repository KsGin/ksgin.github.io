---
title: 为hexo博客系统添加基于github-issue的gitalk评论系统
date: 2018-02-11 18:39:34
tags: 
        - blog
        - web
---

&emsp;&emsp;由于之前使用的[gitment](https://github.com/imsun/gitment)评论系统出现了问题，需要进行一些修改。然后修改的时候想了想，gitment好久没更新了，索性直接换掉用另一个gitalk算了。而gitalk的配置在网上倒是有一大把，但是都是基于NexT主题进行的修改，没有找到yilia主题对应的方案，只能自己折腾，幸好最后弄出来了。

<!--more-->

#### 1.创建 Github OAuth App

&emsp;&emsp;这个和gitment倒是一样，因为这两个的本质都是利用github的issue来搭建评论系统，所以必须要github的授权，这里便是创建一个github授权的程序。

+ 进入github主页创建一个仓库（这个仓库的issue用来存放评论，也可以直接使用自己博客的仓库）

  ![WX20180211-185356@2x](https://ws1.sinaimg.cn/large/006tKfTcly1focq77ifw9j31kw08wq4x.jpg)

+ 创建一个Github OAuth app

  &emsp;&emsp;在设置界面的开发者设置里，点击new

  ![WX20180211-185544@2x](https://ws2.sinaimg.cn/large/006tKfTcly1focq79g7zij31kw0ghjud.jpg)

+ 名称和描述自然是随便填的，记得Homepage URL和Authorization callback URL则不能出错，必须填你博客的主页，比如我的是 https://blog.ksgin.com ，就都填这个，https 与 http也不能省略也不能填错。

  ![WX20180211-190053@2x](https://ws4.sinaimg.cn/large/006tKfTcly1focq78jrgxj31kw0umtd4.jpg)

+ 成功后会获得Client ID和Client Secret两个值，留着等下要用

  ![WX20180211-190411@2x](https://ws3.sinaimg.cn/large/006tKfTcly1focq7abqsxj310w09iq3w.jpg)

#### 2.[gitalk](https://github.com/gitalk/gitalk)使用（这部分摘抄自gitalk github上的使用介绍，看不懂可以略过）

##### Install

Two ways.

- links

```
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/gitalk@1/dist/gitalk.css">
  <script src="https://cdn.jsdelivr.net/npm/gitalk@1/dist/gitalk.min.js"></script>

  <!-- or -->

  <link rel="stylesheet" href="https://unpkg.com/gitalk/dist/gitalk.css">
  <script src="https://unpkg.com/gitalk/dist/gitalk.min.js"></script>
```

- npm install

```
npm i --save gitalk
```

```
import 'gitalk/dist/gitalk.css'
import Gitalk from 'gitalk'
```

##### Usage

A **Github Application** is needed for authorization, if you don't have one, [Click here to register](https://github.com/settings/applications/new) a new one.

**Note:** You must specify the website domain url in the `Authorization callback URL` field.

```
const gitalk = new Gitalk({
  clientID: 'Github Application Client ID',
  clientSecret: 'Github Application Client Secret',
  repo: 'Github repo',
  owner: 'Github repo owner',
  admin: ['Github repo owner and collaborators, only these guys can initialize github issues'],
  // facebook-like distraction free mode
  distractionFreeMode: false
})

gitalk.render('gitalk-container')
```

##### Options

- **clientID** `String`

  **Required**. Github Application Client ID.

- **clientSecret** `String`

  **Required**. Github Application Client Secret.

- **repo** `String`

  **Required**. Github repository.

- **owner** `String`

  **Required**. Github repository owner. Can be personal user or organization.

- **admin** `Array`

  **Required**. Github repository owner and collaborators. (Users who having write access to this repository)

- **id** `String`

  Default: `location.href`.

  The unique id of the page.

- **labels** `Array`

  Default: `['Gitalk']`.

  Github issue labels.

- **title** `String`

  Default: `document.title`.

  Github issue title.

- **body** `String`

  Default: `location.href + header.meta[description]`.

  Github issue body.

- **language** `String`

  Default: `navigator.language || navigator.userLanguage`.

  Localization language key, `en`, `zh-CN` and `zh-TW` are currently available.

- **perPage** `Number`

  Default: `10`.

  Pagination size, with maximum 100.

- **distractionFreeMode** `Boolean`

  Default: false.

  Facebook-like distraction free mode.

- **pagerDirection** `String`

  Default: 'last'

  Comment sorting direction, available values are `last` and `first`.

- **createIssueManually** `Boolean`

  Default: `false`.

  By default, Gitalk will create a corresponding github issue for your every single page automatically when the logined user is belong to the `admin` users. You can create it manually by setting this option to `true`.

- **proxy** `String`

  Default: `https://cors-anywhere.herokuapp.com/https://github.com/login/oauth/access_token`.

  Github oauth request reverse proxy for CORS. [Why need this?](https://github.com/isaacs/github/issues/330)

- **flipMoveOptions** `Object`

  Default:

  ```
    {
      staggerDelayBy: 150,
      appearAnimation: 'accordionVertical',
      enterAnimation: 'accordionVertical',
      leaveAnimation: 'accordionVertical',
    }
  ```

  Comment list animation. [Reference](https://github.com/joshwcomeau/react-flip-move/blob/master/documentation/enter_leave_animations.md)

- **enableHotKey** `Boolean`

  Default: `true`.

  Enable hot key (cmd|ctrl + enter) submit comment.

#### 3.gitalk应用于hexo博客系统的yilia主题

+ 首先我们在yalia目录下的_config.yml下加入如下几行

  ```yaml
  # Gitalk
  gitalk: 
    enable: true    #用来做启用判断可以不用
    clientID: 'xxxxxxxxxxxxxxxxx'    #上面生成Client Id的往这里怼
    clientSecret: 'xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx'   #同上Client Secret 
    repo: ksgin.github.io    #仓库名称
    owner: ksgin    #github用户名
    admin: ksgin
    distractionFreeMode: true
  ```

+ 之后在yalia目录下找到 layout/_partial/post 目录，在里边新建一个文件，并输入以下内容

  ```ejs
  <div id='gitalk-container'></div>
  <link rel="stylesheet" href="https://unpkg.com/gitalk/dist/gitalk.css">
  <script src="https://unpkg.com/gitalk/dist/gitalk.min.js"></script>
  <script type="text/javascript">
    var gitalk = new Gitalk({
      clientID: '<%=theme.gitalk.clientID%>',
      clientSecret: '<%=theme.gitalk.clientSecret%>',
      id: "<%=page.title%>",
      repo: '<%=theme.gitalk.repo%>',
      owner: '<%=theme.gitalk.owner%>',
      admin: '<%=theme.gitalk.admin%>',
      distractionFreeMode: '<%=theme.gitalk.distractionFreeMode%>',
    })
    gitalk.render('gitalk-container')
  </script>
  ```

  &emsp;&emsp;为文件命名为gitalk.ejs

+ 然后找到yalia目录下的 layout/_partial/article.ejs 文件，在

  ```ejs
  <% if (!index && post.comments){ %>
    // something
  <% } %>
  ```

  &emsp;&emsp;这个框架中加入导入gitalke的代码变成以下的样子

  ```ejs
  <% if (!index && post.comments){ %>
    //something
    <%- partial('post/gitalk', {
        key: post.slug,
        title: post.title,
        url: config.url+url_for(post.path)
      }) %>

  <% } %>
  ```

  &emsp;&emsp;按照一般情况，在我们加入内容的框架内，应该有数个评论系统的导入代码，用if来进行判断使用，由于我们现在只使用gitalk，所以可以将其他的都删掉只留我们新加入的这一个（框架主题毕竟是为了通用，而我们是为了己用）。

#### 4.修改成功

&emsp;&emsp;有问题可以在下边留言。