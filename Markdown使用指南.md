# Markdown使用指南

<br>

> ## Markdown 是什么及可以用于哪些用途

Markdown是一种标记语言，通过简单的标记语法，它可以使普通文本内容具有一定的格式。

Markdown的语法简洁明了、比HTML更容易学习，而且功能比纯文本更强，现在有很多人用它写技术文章，比如github网站上的项目介绍Readme.md都是使用Markdown语法编写的。尽管编写容易，但可以渲染成不同的风格，并且借助插件可以转换为PDF/HTML等格式。

从内部来说，比较适合我们编写技术说明，特别是可以借由Markdown文档以及部分成熟的渲染方案生成格式较好的可用于微信发布的文章（尤其是解决代码段格式的问题）。

这个文章没有配图，全是文字，期望可以经由此为大家编写技术总结文章提供一个工具。因只是一个工具所以大概会用就可以了，能用上我适用场景需要用的部分特性，能用它写个简单的技术总结文档，格式可以看的过去就可以了。很炫，很专业也不是本文的目标，不过自己有兴趣可参考文后的参考网站进行深入了解，或自行进行深入学习。

<br>

> ## 我们选用什么工具来使用Markdown

Markdown编辑工具有很多。 有收费的，有免费的. 简单写写总结倒是用不到多么强大的功能，用开源免费软件就足够了。

如下工具可以供我们选用：

- Atom

  *Atom is a text editor that's modern, approachable, yet hackable to the core, a tool you can customize to do anything but also use productively without ever touching a config file. 全球范围内影响力最大的代码仓库/开源社区，GitHub 的程序员们并不满足于此。他们使用目前最先进流行的技术重新打造了一款称为"属于21世纪"的代码编辑器----Atom， 它开源免费跨平台，并且整合 GIT 并提供类似 SublimeText 的包管理功能，支持插件扩展，可配置性非常高......*
  <br>
- Markdown Here

  *Markdown Here是一款浏览器插件。 可以将编辑框中的markdown文本完成渲染，生成渲染后的格式。 渲染，高亮方式可
  以做定制。*

- Github网站上直接编辑

- 内部Gitlab网站上直接编辑

- 网易云笔记

- 在线Markdown编辑 http://dillinger.io/

其中后四种主要仅用于编辑和简单的转换显示，非常简单，本文略过不讲。 文中主要介绍一下第一、第二种工具的相关情况，以及使用这两种工具完成公众号文章的生成。

<br>

> ## 如何安装工具


<br>

### 1\. 安装Atom

- **获取软件** 登陆Atom官网<https://atom.io> ，获得最新的Atom软件。
- **安装软件** 按照<http://flight-manual.atom.io/getting-started/sections/installing-atom/中的安装说明，安装即可>
- **配置代理**（根据网络环境决定是否操作这一步骤） 公司内有proxy的环境，需要在软件中配置代理，才能后续在Atom中下载安装相关的插件（Atom软件必须配置有不同的插件，才能拥有强大丰富的功能）

```cmd
c:\usr\username > apm config set https-proxy https://username:password@proxy-ip:proxy-port
```

- **安装插件**

  Mac中与Windows中菜单有差异，似乎也仅仅是位置不同。 找到Settings菜单，点击进入。 Packages中显示本实例中安装了那些Package,Install中显示有那些包可以安装（输入需要的包名称进行搜索，搜索到后点击下载安装，系统会完成该指令。 部分Package下载完成即可使用，部分需要到Packages找到这个安装好的包，进入其配置界面进行必要的配置。
  <br>

  推荐安装如下包：

  - *Markdown Writer*   该插件相当于Markdown语法的图形界面，可通过选择不同的功能完成相关项的编写，如插入代码块，插入图片，插入链接等等。
  - markdown-preview-enhanced 该插件将提供丰富的预览功能，调出后，可以根据系统设置进行渲染后结果的显示，并在预览界面上右键可调出更多功能，如导入到磁盘，格式也可以选择为PDF或HTML等
  - *markdown-img-paste*  贴图利器。 可以将截屏拷贝至正在编辑的Markdown文件中，并自动收集集中放置贴入的拷屏图片。
  - *GitHub*， *Git Plus* 版本控制插件。 可实现将编辑的文件在Github网站的上传下载。 （该插件本身必须配合以本地安装有Git客户端才可以正常实现预期功能。）插件的功能完全就是在Atom中有界面来实现git client的commit , push等功能。不用插件完全在文件系统中使用传统的git client也是一样的，能将文档保存进git hub并能上传下载就可以了。 这两个插件能否适用于gitlab，没测试，应该也是可以的。
  - *Atom Beutify* 格式插件。 可以帮助你一键调整格式


### 2\. 安装Markdown Here

该插件软件因保存在Google的资产中，所以国内访问不到官方软件下载的位置。

官方提供多种浏览器插件。建议还是用Chrome。可以自己尝试下载，也可以联系我要到这个插件。

- **安装Chrome插件** 在Chrome中访问chrome://extensions/， 然后把下载到的Markdown Here插件直接拖到界面上，按向导完成安装就可以了。
<br>
- **配置** 不配置也可以直接使用了。 实际使用中，可以在Markdown Here Options界面中，选择或者自定义primary styling css以及Syntax Highlighting CSS来实现对渲染结果的自定义，如字体的颜色，大小，以及其他元素的颜色，大小，其他格式选项等。
<br>
推荐两个CSS文件 地址 <https://github.com/zhangcl2002/markdownHere> 专为Markdown Here插件加工微信公众平台文章编辑所用的样式，能很好的兼容代码样式，且有代码高亮效果。 说明
  - 把base.css粘贴到 markdown here 的基本渲染CSS处
  - 把highlight.css粘贴到 markdown here 的语法高亮CSS处即可

<br>

> ## 如何使用工具

<br>
首先，建议找两篇关于Markdown语法的文章看看，了解下最简单的那几个元素就可以了。 如几级标题的定义，引用的定义，字体强调，斜体字，有序列表，无序列表，链接插入，图片插入等主要的使用场景。
<br>
对这些基础的语法要素要做一点点了解，否则，就算能使用工具达成目标，但在黑夜里走路实在也是没什么意思。
> #### 编写Markdown文档 ####

**1\. 简单编辑**

在Atom中新建一个markdown文档，以md为后缀。

然后就可以编写文档了，自己了解语法的部分就自己写，不清楚的，就用Packages -> Markdown Writer中对应的子菜单帮忙完成。

**2\. 编辑器风格以及代码格式风格**

整个编辑器的风格以及代码高亮显示（其实这个工具还是一个代码编辑器，很多前端工程师会用的），我们仅仅希望他能高亮显示我们的一些代码块中的语法就不错了。 如Yaml文件， Linux Shell文件，命令行这些）是在Settings菜单中的Themes这里面设置的，其中又分为Style Theme和Syntax Theme, 分别是定义整个编辑器风格的以及代码语法高亮的）

可以尝试选择不同的风格。

**3\. 图片处理**

目前我们的文档中除了文字描述之外，拷屏以及代码块是两项重要内容。 尤其是拷屏，或者在markdown文档中插入图片，在默认设置里面比较困难。而Markdown中的代码块是做的比较好的，寻找更好的支持代码块编辑方法这是我们本次找到Markdown的原因之一。

使用markdown-img-paste插件吧。 在该插件的设置页面中，设置好'save image in subfolder' 然后下面设置好，比如子目录为'images'.

编写文档的时候，确保图片或拷屏在粘贴缓存中，然后在需要插入图片的位置输入图片的名字，然后ctrl+v. 注意直接拷贝图片文件是不可以的，一定要打开图片，拷贝图片的内容才可以。 拷屏不存在这个问题，都是图片要素在缓存中的。

文章编写完成后，所有该文章的图片都在文章当前目录的子目录images中，全部上传至某图床，如微信公众号的图床里面，然后再将文章中链接图片的超链接从本地改到对应的网络链接。 （麻烦了点，但好在图片也不会太多，都改改也还能接受，主要是看图床对上传图的处理，如果是与原文件名有规律的，那么修改就很容易，没规律就只能一个一个改了）
<br>

> #### 发送微信号文章 ####



**1\. 图片准备**

如果图床使用的是微信公众号的图床，并完成了文章中地址的修正，则可忽略这步。 否则，似乎微信不允许外链，即到外部的链接均失效，那么如果使用外链，那么图片最终也就都看不到了。

**2\. Markdown Here准备**

打开markdown here options界面，然后将上文中推荐的CSS文件拷贝入对应的位置。 （也可以使用其他自己喜欢的风格的）

自定义格式,可以在此处修改CSS的定义，实现对风格的自己定义，但需要熟悉一点CSS的知识，就可以尝试着去修改，修改后，可以在下面的Preview框中通过对示例md文件的转换，进行测试。

**3\.发文编辑**

将编写完成的md文件内容拷贝入待发图文的框中**

**4\. 使用markdown here插件进行转换**

点击浏览器工具栏上的小蝴蝶（markdown here)图标，待发图文就从源码变成了产品啦。

**5. 图文预览后发出**
预览可以使用它提供的模拟器预览。 也可以发送到自己的手机预览。

<br>
**_reference:_**

- [Atom Official Web Site][08e41aff]

- [使用Atom打造无懈可击的Markdown编辑器][c5ba301c]

- [Atom：优雅迷人的编辑神器][4bfe01b9]

- [终于找到你，Markdown Here][1a9a0236]

[08e41aff]: https://atom.io "Atom Web Site"
[1a9a0236]: http://www.jianshu.com/p/4c0d81a29627 "Markdown Here"
[4bfe01b9]: http://www.jianshu.com/p/b4c8479cfaa5 "Atom"
[c5ba301c]: http://www.cnblogs.com/libin-1/p/6638165.html "Atom"
