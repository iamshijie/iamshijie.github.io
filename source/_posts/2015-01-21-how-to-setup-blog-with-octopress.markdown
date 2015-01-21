---
layout: post
title: "使用octopress创建个人博客"
date: 2015-01-21 10:02:40 +0800
comments: true
categories: 
sharing: true
---

Refer to [this link](http://octopress.org/docs/setup)

开始之前的准备工作：安装Git、ruby、bundler。

###准备

1. 下载octopress: `git clone git://github.com/imathis/octopress.git octopress`

2. 进入octopress目录，执行`rake install`

###创建部署

1. 创建一个新的Github的空repo，并且命名为iamshijie.github.io

2. 进入自己下载的octopress的目录并执行`rake setup_github_pages`。当提示输入repo地址时，输入上一步创建的git地址：`git@github.com:Shi-Jie/iamshijie.github.io.git`

3. 继续在octopress目录下执行`rake generate	`。此时，打开[http://iamshijie.github.io.git](http://iamshijie.github.io.git)就能看到blog原型。再执行`rake deploy`

4. git提交。本别执行`git add .` 、`git commit -m 'your message'`、`git push origin source`

5. 修改个人blog的默认分支为source，而非master [git link](https://github.com/iamshijie/iamshijie.github.io/settings)


###创建博客

这部分可以参考之前在QA community的[blog](http://thoughtworks-china-qa.github.io/blog/2015/01/13/how-to-post-to-this-blog/)。

1. clone博客repo到本地：`git clone git@github.com:iamshijie/iamshijie.github.io.git`

	**确保你的默认分支是source。即`git status`后看到的是**
			
	```
	Shi-Jie:iamshijie.github.io shijie$ git status
	On branch source
	Your branch is up-to-date with 'origin/source'.
	nothing to commit, working directory clean
	```
2. 本地生成发布目录，执行`rake setup_github_pages`交互时输入`git@github.com:Shi-Jie/iamshijie.github.io.git`

3. 执行`rake install`。安装默认主题。


4. 创建新post。在repo下执行`rake new_post["How to create blog with octopress"]`

5. 修改上一步生成的markdown文件，直接在默认生成的内容后进行添加编辑：

	```
	---
	layout: post
	title: "How to setup blog with octopress"
	date: 2015-01-21 09:16:24 +0800
	comments: true
	categories:
	---
	```
6. 使用`rake preview`进行预览

7. 确认无误后。使用`rake gen_deploy`发布的master。完成后，就可以在个人[http://iamshijie.github.io](http://iamshijie.github.io)看到效果

8. 将改动提交到source分支

	```
	git add .
	git commit -m 'Post my first blog. - ShiJie'
	git push
	```
