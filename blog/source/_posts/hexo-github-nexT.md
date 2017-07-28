---
title: Hexo+GithubPages+nexT站点搭建
date: 2017-07-27 15:11:06
tags: [Hexo, nexT, github, 填坑记]
categories: [前端]
---

{% note warning %} 
**写在前面**
    趁离职期间想整理下这些年积累的东西，决定自己建个站点，为了方便简单选择了Hexo+GitHub Pages。但是做后端开发久了，真心觉得静态站点有点费事。考虑到github免费，那先将就吧，以后还是得自己搞个服务器。这篇先来讲讲搭建时遇到的坑，下一篇讲讲我对Hexo的理解。
{% endnote %}
<!--more-->
## Hexo安装

关于Hexo安装操作网上有很多，建议直接参考[Hexo官网](https://hexo.io/zh-cn/ "Hexo官网")。我贴下几条关键指令：

```
// 安装Hexo环境
npm install hexo-cli -g

// 初始化blog站点
hexo init blog
cd blog
npm install

// 生成静态文件
hexo g

// 在本地启动站点，指定端口，请加-p [端口]
hexo server
hexo server -p 5000

// 发布
hexo deploy
```

## 利用Github Pages部署站点
由于Hexo生成的静态文件，恰好github支持静态文件渲染，github就成了个人博客最好的免费服务器。我在网上查到很多部署方面的文章，基本都是如下说法：

**1. 在_config.yml配置deploy参数，前提先安装hexo-deployer-git插件：**
```
deploy:
  type: git
  repo: https://github.com/TerraHuang/terrahuang.github.io.git
  branch:   // 分支这个地方很关键呀，被坑了...
```
**2. 在github上创建新的repository，且以`[账号名].github.io`命名。将该项目克隆到本地。**
```
git clone https://github.com/TerraHuang/terrahuang.github.io.git
```
   以上步骤都没错，非常正确，不过坑马上就来了。`前方高能`

### 大坑一
**3. 将hexo生成站点的代码发布到master分支，将部署的静态文件代码发布到gh-pages分支，然后在github项目Settings中将Source改为gh-pages。我照做如下：**

![](/screenshot/20170728/git-page-setting.png)

虽然我的内心很崩溃，但是我还没放弃！！！

还好Source旁边有个`Learn more`，用户体验做的不错，点个赞。

点开它，才发现别有洞天呀，网上文章真不能全信~~

![](/screenshot/20170728/learn-more.png)

{% note primary %}
哦，原来[账号名].github.io这种命名只能使用master分支作为站点资源。
{% endnote %}

另外Github pages还支持其它几种方式，如下：

![](/screenshot/20170728/page-source.png)

**4. 正确的Github Pages发布方式:**
+ 静态站点部署文件应放到master分支
+ Hexo生成代码放到其它分支，或者新建其它的repository来托管
+ _config.yml的deploy->branch须写成`master`
```
deploy:
  type: git
  repo: ...
  branch: master
```
+ 在浏览器中请求 https://[账号名].github.io 即为个人站点。

**5. Github Pages绑定域名：**
+ 在阿里云万网上购买域名
+ 在master分支新建CNAME文件，名称必须全大写，内容为需绑定的域名
![](/screenshot/20170728/dns.png)
+ 在万网域名管理页面新增一条解析，如下：
![](/screenshot/20170728/dns-express.png)

具体步骤说明，请参考**yuan3065的博客**：[GitHub Pages 绑定来自阿里云的域名](http://blog.csdn.net/yuan3065/article/details/51594454)


## nexT主题及改造
毕竟是自己的第一个站点，选用主题也是很谨慎的，在众多炫目的主题中选择了nexT。据说目前是得星最多的一套主题。我看中它的点如下：
{% note primary %}
**优势：** 样式简洁，文档详细，提供的第三方服务丰富，改造容易
{% endnote %}

### 主题色
原来的黑色太暗沉，像我心里这么阳光的人，果断换了个明亮的调调。
但是nexT并没有提供主题色的更改配置，我用的是笨办法，直接在`theme/next/source/css/`中修改色值。以后有时间再修改下代码，提取到_config.yml配置中来。

### 新增第三方服务
配置第三方服务比较简单，请参考[nexT文档](http://theme-next.iissnan.com/)

新增功能如下：
+ **站内搜索：插件hexo-generator-searchdb**
+ **文章阅读次数统计：LeanCloud**
+ **内容分享：Jiathis**
+ **评论系统：disqus（不翻墙无法显示，考虑代理）**
### 大坑二
{% note primary%}
特别提醒：注册disqus时，一定要翻墙。如果是利用hosts文件翻墙，请在hosts新增一条[sitename].disqus.com->xx.xx.xx.xx域名映射，IP地址与disqus.com相同即可。
{% endnote %}

### 多图显示
为了展示图册效果，尝试显示多图，在网上找到方法如下：

**1. 在站点/source下新增pictures/index.md文件。Front-matter如下：**
```
---
title: "图册"
type: "picture"  //type必须为'picture'
layout: post
comments: false
---
```
**2. 内容为多图时，新增标签`{\% gp n-n \%}(content){\% endgp \%}`。具体如下：**

```
{% gp 2-2 %}
![](http://octodex.github.com/images/minion.png)
![](http://octodex.github.com/images/minion.png)
{% endgp %}
```
然并卵...    `前方高能`

### 大坑三
居然报错了，为什么会报错，我的内心是拒绝的... 还好我没放弃！！！

找到`{\% gp \%}`标签代码：`/theme/next/source/script/tags/group-pictures.js`

经过多次测试，发现代码错误，修改如下(43-63行)：
```
/**
* 2-2
*
* □ □  // 此为n-n设值所展示的排列方式，作为设值参考。默认：一行3列，依次排列
*
* @param pictures
*/
group2Layout2: function (pictures) {
  return this.getHTML([pictures]);  // 将pictures变成二维数组作为实参
},

/**
* 3-1
*
* □ □ □
*
* @param pictures
*/
group3Layout1: function (pictures) {
  return this.getHTML([pictures]);  // 将pictures变成二维数组作为实参
},
```
{% note primary %}
**特别说明：**其实只有2-2，3-1的组合会报错，只有两个方法传了一维数组，导致后续报错。
{% endnote %}

好了，现在不报错了。启动起来，果然可以展示一行两列的图组了。

然并卵...   `前方高能`
怎么不能点击放大？！ 我的内心是崩溃的，但是我没有放弃！！！

通过与非`gp`标签图片对比，我发现`gp`标签渲染时，并没有给图片加`<a/>`标签，导致fancybox插件并未运行起来。

OK，找到原因了。定位代码: 在`/theme/next/source/script/tags/group-pictures.js`（824行`getColumnHtml`方法），做如下修改：
```
getColumnHTML: function (pictures) {
    var columns = [];
    var columnWidth = 100 / pictures.length;
    var columnStyle = ' style="width: ' + columnWidth + '%;"';

    for (var i = 0; i < pictures.length; i++) {
    
      // 在<img />元素上加一个父层<a/>,结构如下：
      // <div class="group-picture-column" >
      // <a href="" class="fancybox fancybox.image" rel="group">
      //   <img  />
      // </a>
      // </div>
      var src = pictures[i].match(/src="[^"]*"/)[0].replace(/(src=)|"/g,"");
      var html = '<div class="group-picture-column" ' + columnStyle + '>'
          + '<a href="'+ src +'" class="fancybox fancybox.image" rel="group">'
          + pictures[i] 
          + '</a></div>'
      columns.push(html);
    }
    return columns.join('');
  }
};
```
这回终于解决了。效果请参考我的[图册](/pictures)。


**填坑完结。希望我踩过三大坑，对大家有所帮助。**

{% note warning %}
**写给自己**
毕竟是第一篇，写了蛮久。我打算陆陆续续把学到的东西挪到这里来。写在这里，当勉励自己，不要懒散吧。
{% endnote %}
 