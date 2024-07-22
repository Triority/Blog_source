---
title: 博客文章编辑发布指南
tags:
  - hexo
  - 文档
cover: /img/RUN.png
categories:
  - 文档&笔记
date: 2024-07-22 18:33:41
description: 博客帮助文件
---
# 概述
博客采用静态hexo框架，基于node js，文章使用markdown语言进行编写。源文件保存于香港的服务器，使用git进行源码管理同时在github仓库和nas备份。使用nginx提供对外服务。
# 编辑权限
博客编辑需要登陆服务器，ssh仅支持密钥登录凭据，如果你希望在博客发布信息请联系我获得服务器密钥
# 新建和编辑文章
+ 使用ssh登入服务器后，进入`/hexo`目录。
+ 打开终端，输入`hexo new [name]`即可新建一个web链接为name的文章。请务必设置name为英文字母数字和减号，不要包含中文字符等内容
+ 文章按年份分类，新建的文章保存为`/hexo/source/[year]/[name].md`，即可在其中使用markdown语言编写文章

# 文章配置
```md
---
title: [name]
tags:
  - null
cover: /img/RUN.png
categories:
  - - 值得一提的文章
  - - 旅游&摄影
  - - 日记本
  - - 整活&游戏
  - - 文档&笔记
  - - 折腾记录
  - - 作品&项目总结
  - - 过时&弃用&无意义&失败品
  - - 外部引用
  - - 无线电
date: 2024-07-22 18:44:51
description:
---

```
新建的文章文件就包含上面的内容：
+ title:文章标题，默认为设置的文章链接，可以修改为其他文字，可以为中文
+ tags:文章标签，可以参考标签页已有的进行设置，也可以新建自己的。如果不想设置请直接删除这一配置项
+ cover:文章封面图。文章封面图片统一保存于`hexo/source/img/`内
+ categories:文章分类，设置方法类似tag
+ data:文件新建时间，自动生成。主页文章排序依靠时间，修改为较大的值可用于置顶文章，但是一般不建议设置置顶
+ description:简介描述。随便写两句话介绍一下就好。建议在此处加入文章作者信息(如果不是Triority本人编辑的话)

编辑完成应该类似这样，以本篇文章为例：
```md
---
title: 博客文章编辑发布指南
tags:
  - hexo
  - 文档
cover: /img/RUN.png
categories:
  - 文档&笔记
date: 2024-07-22 18:33:41
description: 博客帮助文件
---
```

# 内容编辑
## markdown
markdown语言语法请自行学习，初期使用不熟练推荐使用开源项目：[Editor.md](https://pandao.github.io/editor.md/)
## 资源文件路径

## 标签外挂语法

# 文件生成和git提交
文章编辑完成之后，在`/hexo`目录打开终端，输入`hexo g`进行html文件生成，如果报错请检查文章内是否有语法错误，特别是文章配置部分。

然后进行git提交和推送：
`git commit -m "说明"`