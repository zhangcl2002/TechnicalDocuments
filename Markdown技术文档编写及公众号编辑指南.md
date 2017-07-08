# Markdown使用指南
<br>

>### Markdown 是什么及用途

Markdown是一种可以使用普通文本编辑器编写的标记语言，通过简单的标记语法，它可以使普通文本内容具有一定的格式。

Markdown的语法简洁明了、容易学习，而且功能比纯文本更强，有很多人用它写技术文章。比如github网站上的项目介绍Readme.md都是使用Markdown语法。

编写容易，但可以渲染成不同的风格，并且借助插件可以转换为PDF/HTML等格式。

所以从内部来说，比较适合我们编写技术说明，特别是可以借由Markdown文档以及部分成熟的渲染方案生成格式较好的可用于微信发布的文章（尤其是解决代码段格式的问题）。

这个文章没有图，全是字。因为就是一个文本编辑的工具，我们浅尝就可以了，能用上自己需要用的部分和特性就可以了，能用它写个简单的技术总结文档，格式可以看的过去就可以了。好炫，好专业也不是我们的目标。如读者自己有兴趣，可参考文后的参考网站进行深入了解，或自行进行深入学习。

<br>
>### 我们选用什么工具来使用Markdown

Markdown编辑工具有很多。 有收费的，有免费的. 简单写写总结不需要多么强大的功能，所以用开源免费软件就足够了。

如下工具可以供我们选用：

* Atom
> Atom is a text editor that's modern, approachable, yet hackable to the core, a tool you can customize to do anything but also use productively without ever touching a config file.
> 全球范围内影响力最大的代码仓库/开源社区，GitHub 的程序员们并不满足于此。他们使用目前最先进流行的技术重新打造了一款称为“属于21世纪”的代码编辑器——Atom， 它开源免费跨平台，并且整合 GIT 并提供类似 SublimeText 的包管理功能，支持插件扩展，可配置性非常高……
* Markdown Here
> Markdown Here是一款浏览器插件。 可以将编辑框中的markdown文本完成渲染，生成渲染后的格式。 渲染，高亮方式可以做定制。
* Github网站上直接编辑
* 内部Gitlab网站上直接编辑
* 网易云笔记

其中后三种主要仅用于编辑和简单的转换显示，非常简单，本文略过不讲。 文中主要介绍一下第一、第二种工具的相关情况，以及使用这两种工具完成公众号文章的生成。

<br>
>### 如何安装工具
<br>

#### 1. 安装Atom ####

* **获取软件**
  登陆Atom官网https://atom.io ，获得最新的Atom软件。
* **安装软件**
  按照http://flight-manual.atom.io/getting-started/sections/installing-atom/中的安装说明，安装即可
* **配置代理**（根据网络环境决定是否操作这一步骤）
  公司内有proxy的环境，需要在软件中配置代理，才能后续在Atom中下载安装相关的插件（Atom软件必须配置有不同的插件，才能拥有强大丰富的功能）

````cmd
c:\usr\username > apm config set https-proxy https://username:password@proxy-ip:proxy-port
````
* **安装插件**

  Mac中与Windows中菜单有差异，似乎也仅仅是位置不同。 找到Settings菜单，点击进入。 Packages中显示本实例中安装了那些Package,Install中显示有那些包可以安装（输入需要的包名称进行搜索，搜索到后点击下载安装，系统会完成该指令。 部分Package下载完成即可使用，部分需要到Packages找到这个安装好的包，进入其配置界面进行必要的配置。

  推荐安装如下包：
  - Markdown Writer
  该插件相当于Markdown语法的图形界面，可通过选择不同的功能完成相关项的编写，如插入代码块，插入图片，插入链接等等。
  - markdown-preview-enhanced
  该插件将提供丰富的预览功能，调出后，可以根据系统设置进行渲染后结果的显示，并在预览界面上右键可调出更多功能，如导入到磁盘，格式也可以选择为PDF或HTML等
  - markdown-img-paste
  贴图利器。
  可以将截屏拷贝至正在编辑的Markdown文件中，并自动收集集中放置贴入的拷屏图片。
  - GitHub ， Git Plus
  版本控制插件。 可实现将编辑的文件在Github网站的上传下载。 （该插件本身必须配合以本地安装有Git客户端才可以正常实现预期功能。）

#### 2. 安装Markdown Here ####

  该插件软件因保存在Google的资产中，所以官方软件下载的位置国内访问不到。（不要问我为什么，我也很想知道）

  官方提供多种浏览器插件。建议还是用Chrome吧。

* **安装Chrome插件**
  在Chrome中访问chrome://extensions/， 然后把下载到的Markdown Here插件直接拖到界面上，按向导完成安装就可以了。
* **配置**
  不配置也可以直接使用了。
  实际使用中，可以在Markdown Here Options界面中，选择或者自定义primary styling css以及Syntax Highlighting CSS来实现对渲染结果的自定义，如字体的颜色，大小，以及其他元素的颜色，大小，其他格式选项等。

<br>
>### 如何使用工具

首先，还是找两篇关于Markdown语法的文章看看，看最简单的那几个元素就可以了。 如几级标题的定义，引用的定义，字体强调，斜体字，有序列表，无序列表，链接插入，图片插入等主要的使用场景。

对这些基础的语法要素要做一点点了解，否则，就算能使用工具达成目标，但在黑夜里走路实在也是没什么意思。

1. 编写Markdown文档



2. 发送微信号文章


<br>
***reference:***
* [Atom Official Web Site][08e41aff]

  [08e41aff]: https://atom.io "Atom Web Site"

* [使用Atom打造无懈可击的Markdown编辑器][c5ba301c]

  [c5ba301c]: http://www.cnblogs.com/libin-1/p/6638165.html "Atom"

* [Atom：优雅迷人的编辑神器][4bfe01b9]

  [4bfe01b9]: http://www.jianshu.com/p/b4c8479cfaa5 "Atom"

*[终于找到你，Markdown Here][1a9a0236]

  [1a9a0236]: http://www.jianshu.com/p/4c0d81a29627 "Markdown Here"
