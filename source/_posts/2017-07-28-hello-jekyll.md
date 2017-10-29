title: 初识jekyll
---
在我搭建这个个人博客网站之前，我对搭建博客的方法是毫无概念的。一说到搭建博客，很多开发者一定都会与当时的我一样对这种简单的网站嗤之以鼻，觉得里面并无多大的学问可追究，无非就是将文章存入数据库，再用后端语言动态生成网站便可。这种方法乍看之下是毫无漏洞的，可细细一想，这个解决方案还是有它的局限性的。很多人搭建个人博客是不想大动干戈的，这种前端加后端的万金油套路所需要的开发技能其实是比较广泛的。一个前端开发者可能并不想去触碰后端繁琐的数据业务，也并不想为了搭建一个博客而去学习后端的东西。如果以上描述的问题正是你所头疼的，那么我相信jekyll这个解决方案就是你所需要的。

# 什么是jekyll？
jekyll是一个可以将Markdown，Liquid，HTML，CSS转化为静态网页与博客的工具。正是因为jekyll提供的这种转化，能让静态网页也能够实现动态生成。jekyll+HTML+Liquid+CSS的组合能让你彻底不接触后端也能做出带有"数据库"的博客。

# 如何使用jekyll？
jekyll的上手可以说是及其简易的，创建一个由GitHub Pages托管的jekyll网页可以说是分分钟的事情。第一步自然就是安装，我使用的是mac操作系统，以下的安装教学也会以mac来做范例。

第一步请打开Terminal，并输入
`sudo gem install jekyll bundler`
来全局安装jekyll与bundler控件（jekyll所需ruby版本>=2.1.0,[如何安装最新版ruby](https://stackoverflow.com/questions/38194032/how-to-update-ruby-version-2-0-0-to-the-latest-version-in-mac-osx-yosemite)）

接下来便可以使用jekyll来创建自己的博客啦。打开Terminal并cd至适合存放新博客的路径，输入`jekyll new blog`，jekyll此时会在当前路径下生成一个叫blog的文件夹来存放一个已经被初始化好的jekyll网站。

然后在Terminal中输入`cd blog` 与 `jekyll serve`并在浏览器内去往jekyll serve给的地址（通常为`http://127.0.0.1:4000/`），你便能看到如下的画面

<img width="100%" src="http://otowvkh6b.bkt.clouddn.com/jekyll1.png">

当你看到上面这个图时，说明你的jekyll的本地架构已经完成，现在可以真正开始写你的博客了。在开始写博客前我们先得理解blog文件夹中的内容，在Terminal中cd至blog路径并输入ls你便可以瞥见文件夹的全貌啦。下面这个图片便是blog文件夹中的所有文件显示。

<img width="100%" src="http://otowvkh6b.bkt.clouddn.com/jekyll4.png">


## _config.yml文件
该文件是用来修改jekyll的基本设置的，你可以在此文件中修改一些既定的参数（例如baseurl，permalink等等），具体请参考[官方文档](http://jekyllrb.com/docs/configuration/)

## 404.html文件
该html文件是用来作为默认404错误的页面，你可以修改这个html文件来自定义不同的404页面。

## Gemfile与Gemfile.lock文件
这两个文件是bundler用来记录网站所需要的所有gem与gem版本的。同时这些文件里面还记录了当前网站可用的主题（theme）。当你还在想这些生成默认网站的css或者html文件在哪里时，他们其实都包含在了gem之中。jekyll默认使用的gem主题为[Minima](https://github.com/jekyll/minima)，这也是你在第一个图中看到的默认网站所使用的主题。如果你想使用自定义的主题，请在blog文件夹中创建一个名为_layouts的文件夹，在里面创建一个default.html的文件，并在文件内写入

{% codeblock lang:html %}{% raw %}
<div>
	{{ content }}
</div>
{% endraw %}{% endcodeblock %}

这就是一个最简易的主题文件，语言使用的是HTML+Liquid，content在这里代表你嵌套这个主题的内容。换言之，当你的任何页面使用这个名为default主题时，页面都会被嵌套在这个主题定义的div标签中。下面我会给出一个如何使用这个名为default主题的实例。在blog文件夹中创建一个一个名为index.html的首页文件，并在此文件中写入。
{% codeblock lang:html %}
---
layout:default
---
<p>	
	hello world!
</p>
{% endcodeblock %}

在文件顶端我们定义了使用default为样式（注：任何想被jekyll处理的html页面顶端都必须含有 --- 标识，这样的标识被称作为YAML front matter），最后经手jekyll处理而生成的最终页面为
{% codeblock lang:html %}
<div>
	<p>	
		hello world!
	</p>
</div>
{% endcodeblock %}

## _posts文件夹
该文件夹是包含了所有的你想动态生成的博客或者网页。例如默认其中有一个名为 2017-07-28-welcome-to-jekyll.markdown 的文件，请留意它的命名方法为 年-月-日-标-题.markdown(html)。

## _site文件夹
当你在Terminal输入`jekyll serve`时，该文件夹会被自动生成。文件夹内包含了已经经过本地jekyll处理的最终的HTML。该文件夹是不需要上传到服务器的，服务器端的jekyll会处理你的所有文件并生成一样的供服务器端展示的_sites文件夹。

## 根据_posts文件夹动态生成文章列表
修改index.html的内容为
{% codeblock lang:html %}{% raw %}
---
layout: default
title: blog
---
<h1>{{ page.title }}</h1>
<ul>
 {% for post in site.posts %}
 	<li>
    	{{ post.date | '%B %d, %Y' }} 
    	<a href="{{ site.baseurl }}{{ post.url }}">
    		{{ post.title }}
    	</a>
	</li>
 {% endfor %}
</ul>
{% endraw %}{% endcodeblock %}
简析一下这段HTML+Liquid代码，第二行定义了使用的主题为default。第三行定义了一个名为title，值为blog的变量。第五行我们使用了当前页的title变量也作为标题。第六到十三行我们对_posts文件夹中的所有文档做了一个循环，每次生成一个列表项，每一个列表项包含了该文档的日期（post.date）与文档标题（post.title）。文档标题中包含了能够转到该文档的链接`{% raw %}{{ site.baseurl }}{{ post.url }}{% endraw %}`（注：site.baseurl便是_config.yml里的baseurl变量，可依据需求在_config.yml中进行修改）。

# 总结
完成以上的练习后就可以将博客上传至GitHub Pages了，这样你就拥有了一个会动态生成博客列表的静态网页博客。关于jekyll还有许许多多的内容例如[草稿文件夹](https://jekyllrb.com/docs/drafts/)，[资源包文件夹](https://jekyllrb.com/docs/assets/)，[包含文件夹](https://jekyllrb.com/docs/includes/)等等，相信在你写博客的过程中会去发现与挖掘jekyll更多的功能。