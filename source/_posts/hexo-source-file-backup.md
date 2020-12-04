---
title: 将hexo源文件备份到github
date: 2020-12-04 22:22:17
tags:
    - hexo
    - github
categories:
    - hexo
---

### 将hexo源文件备份到github

#### 首次备份
* 在 Hexo 根目录下，初始化 git 仓库
```
git init
```
* 创建并切换到分支 blog-source
```
git checkout -b blog-source
```
* 添加文件列表
```
git add -A
```
* 添加提交说明
```
git commit -m "First commit"
```
* 设置远程仓库映射
```
git remote add origin git@github.com:ilubov/ilubov.github.io.git
```
* 提交
```
git push -u origin blog-source
```

#### 之后备份
```
git add -A
git commit -m "Update message"
git push origin blog-source 或 git push
```

[hexo+github搭建个人博客](https://mp.weixin.qq.com/s/yfnNN76KYDUgNbnbEIgE7A)