---
title: "Hugo 和 GitHub Pages 使用指南"
date: 2019-05-14T09:42:22+08:00
draft: false
---

# 简介
`hugo` 是一个基于 `golang` 静态网站生成程序，类似于 `hexo`。

`GitHub Pages` 是 `GitHub` 提供的一项服务，可以直接从 `GitHub` 仓库为个人，组织或项目页面提供基于SSL的免费和快速静态托管。

之前个人主页使用的 `hexo` 来生成的，是基于 `node.js` 构建的，速度较慢，

# 安装 `hugo`
```bash
mkdir $HOME/src
cd $HOME/src
git clone https://github.com/gohugoio/hugo.git
cd hugo
go install --tags extended
```

**注意**

- 从 `hugo 0.48` 开始使用 `go mod` 进行依赖管理
- 使用 Linux 包管理安装的 `hugo` 版本可能较低，有些主题不支持


# 使用
官方教程是创建两个仓库: 一个放源码(`hugo new site .`的所有文件)，一个放静态页面(生成的 `public/`目录)。

个人习惯使用一个仓库，不同分支。

- 一个仓库: `jsonbruce.github.io`
- 不同分支: `master` 分支用于静态页面，`hugo` 分支用于源码, `hexo` 分支用于之前 hexo 源码

## 初始化
```bash
1. 克隆仓库
git clone git@github.com:jsonbruce/jsonbruce.github.io.git

cd jsonbruce.github.io.git

2. 创建一个空分支 `hugo` 用于放源码
git checkout --orphan hugo
git rm -rf .

3. 初始化源码
hugo new site .

4. 添加主题
git submodule add git@github.com:jsonbruce/hugo-coder.git themes/hugo-coder

5. 添加自定义域名
echo "jsonbruce.com" > static/CNAME

6. 配置 `config.toml`

7. 设置 `public` 目录为 `master` 分支目录树
git worktree add -B master public origin/master
```

## 创建文章
```bash
hugo new about.md

hugo new posts/userguide.md
```

## 部署脚本
在 `hugo` 分支的根目录下执行。

```bash
#!/bin/bash

if [[ $(git status -s) ]]
then
    echo "The working directory is dirty. Please commit any pending changes."
    exit 1;
fi

echo -e "\033[0;32mDeploying updates to GitHub...\033[0m"

echo "Deleting old publication"
rm -rf public
mkdir public
git worktree prune
rm -rf .git/worktrees/public/

echo "Checking out master branch into public"
git worktree add -B master public origin/master

echo "Removing existing files"
rm -rf public/*

echo "Generating site"
hugo # if using a theme, replace with `hugo -t <YOURTHEME>`

echo "Updating master branch"
# Go To Public folder and add changes to git.
cd public && git add --all 

# Commit changes.
msg="rebuilding site `date`"
if [ $# -eq 1 ]
  then msg="$1"
fi
git commit -m "$msg"

# Push source.
git push -f origin master

# Come Back up to the Project Root
cd ..
```

# 域名解析
添加指向以下 IP 的 `A` 记录:

```
185.199.108.153
185.199.109.153
185.199.110.153
185.199.111.153
```

主机记录为 `@` 和 `www`，最终生成八条解析记录。解析列表如下:

```
jsonbruce.com.		600	IN	A	185.199.111.153
jsonbruce.com.		600	IN	A	185.199.110.153
jsonbruce.com.		600	IN	A	185.199.109.153
jsonbruce.com.		600	IN	A	185.199.108.153
www.jsonbruce.com.	600	IN	A	185.199.111.153
www.jsonbruce.com.	600	IN	A	185.199.110.153
www.jsonbruce.com.	600	IN	A	185.199.109.153
www.jsonbruce.com.	600	IN	A	185.199.108.153
```

# 设置 `GitHub`
`GitHub Pages` 绑定自定义域名:

在仓库的 `Settings -> GitHub Pages -> Custom domain` 页面下填写自定义的域名。

# 参考
- Host on GitHub, https://gohugo.io/hosting-and-deployment/hosting-on-github/
- Using a custom domain with GitHub Pages, https://help.github.com/en/articles/using-a-custom-domain-with-github-pages
