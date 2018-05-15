---
title: coding+hexo部署踩坑经历
date: 2016-03-14 09:19:01
tags: [code]
cover: /images/coding.png
description: 使用hexo生成自己的静态博客站点，并配置到coding
---
## hexo
hexo是一个npm包，在node环境上运行，他的功能就是将你的文章（.md文件）生成为静态html文件。虽然他在生成静态文件时需要你的本地机器安装node环境，但部署你的站点时则不需要后台环境，这一点不同于php驱动的wp，typecho和node驱动的ghost，你只需要一个托管静态资源的云平台即可。


## Coding Pages
- Coding Pages是一个支持 jekyll 静态站的服务，起个人或项目展示作用。
- 需要注意的是如果将项目作为个人主页，则需要项目名的命名需要和用户名一致。这样访问默认主页（｛someone｝.coding.me）和项目主页（｛someone｝.coding.me/｛someone｝）是一样的效果，而访问你的其他项目（｛someone｝.coding.me/｛otherProj｝）则不受影响。
 [官网说明][1]


## 发布
发布的时候遇到了小坑，执行
```bash
hexo deploy
```
竟然得到了：git not found
安装了以下模块，再次运行则顺利发布。

```bash
npm install hexo-deployer-git --save
```

## 配置
_config.yml是需要用心去配置的：
 ```
deploy:
  type: git
  repo: git@git.coding.net:｛someone｝/｛proj｝.git
  branch: coding-pages
```
当然这里如果发布个人主页，那么someone和proj是一样的，如果为项目主页则反之，分支也可自己命名，创建项目时是不生成master。

到此为止，如果你踩过了以上坑，在｛someone｝.coding.me上看到了你的主页，那么恭喜你已经成功了。之后可以通过
```
hexo new  ***   //生成新文章
...             //在md文件中撰写
hexo generate   //扫描md文件，生成对应html
hexo deploy     //自动发布到coding仓库中
```
这些步骤继续你的hexo之旅。

## 自定义域名
如果你要绑定自己的域名，还需要以下步骤：
- 首先去域名提供商网站，将域名的的CNAME指向pages.coding.me或者｛someone｝.coding.me
- 在项目中开启Pages服务并绑定相应的域名
- 在配置文件中设置
```
url: 你的域名
root: /
```

## 结束
到此为止，一个拥有个性域名的个人站点已经建立完成，考虑到coding托管容量，可以将一些较大的静态资源托管到例如七牛等存储上，将免费进行下去。。。


[1]:https://coding.net/help/doc/pages/index.html