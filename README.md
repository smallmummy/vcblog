## 说明

### 用途
此blog是为自己创建的一个技术blog，用于展示、记录一些技术性的日志。

### 环境

jekyll + github  
有需要的同学可以fork自己的版本，用于搭建基于github的blog。 

### 模板

使用的是 Mundana Jekyll Theme, 然后根据自己的情况进行的修改,下面是模板的说明
#### Documentation

[How to install & use](https://bootstrapstarter.com/bootstrap-templates/mundana-theme-jekyll/)

#### Contribute to Mundana repository

1. In the top-right corner of this page, click **Fork**.

2. Clone a copy of your fork on your local, replacing *YOUR-USERNAME* with your Github username.

   `git clone https://github.com/YOUR-USERNAME/mundana-theme-jekyll.git`

3. **Create a branch**: 

   `git checkout -b <my-new-feature-or-fix>`

4. **Make necessary changes and commit those changes**:

   `git add .`

   `git commit -m "new feature or fix"`

5. **Push changes**, replacing `<add-your-branch-name>` with the name of the branch you created earlier at step #3. :

   `git push origin <add-your-branch-name>`

6. Submit your changes for review. Go to your repository on GitHub, you'll see a **Compare & pull request** button. Click on that button. Now submit the pull request.

That's it! Soon I'll be merging your changes into the master branch of this project. You will get a notification email once the changes have been merged. Thank you for your contribution.


### 个人定制化的地方

基于外表的简单定制,比如替换图片,修改了menu等等,就不在这里列举了. 这里只说明一下定制化的功能:

#### 1. 在menu中增加了一个新的菜单项projec,用来列举一些正在或者已经完成的项目.

在_includes下面的menu-header.html文件中添加新的引用,并增加新的project.html

#### 2. 在post页面增加了catalog菜单/导航功能

在_layouts下面的post.html文件中增加了后半段的代码.
在assets/js/下面新增了功能需要的文件jquery.nav.js

#### 3. 对post页面下方的文章导航(上一篇/下一篇)进行了功能优化
在_layouts下面的post.html文件中`<!-- Aletbar Prev/Next -->`进行了优化

#### 4. 对index页面的上方4篇文章进行了置顶
修改了index.html中

```
{% assign latest_post = site.posts[0] %}
```
修改为了

```
{% assign latest_post = site.tags.stick_post[0] %}
```

需要在post中的MD文件(就是你编写的markdown文件)头部新增stick_post属性,例如

```
---
layout:     post
title:      The Experience of Scraping Instagram(Crawler)
subtitle:   Crawler
author:     vincent_c
categories: [ crawler, python ]
image: assets/images/crawler.jpg
catalog: true
tags: [Crawler, featured,stick_post]
---  

```

在index.html下方的部分(置顶后的展示部分),修改为下面的部分,对paginator.posts的部分进行判断,如果已经置顶显示过了,此处就不在显示了.

```
{% for post in paginator.posts %}
	{% if post.tags contains "stick_post" or post.tags contains "sticky" %}
		
		    {% continue %}
			{% endif %}
    
  	
  {% include main-loop-card.html %}
    
{% endfor %}
```