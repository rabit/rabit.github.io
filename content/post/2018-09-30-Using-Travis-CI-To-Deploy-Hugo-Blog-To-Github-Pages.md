---
title: "以Travis CI將Hugo Blog部署到Github Pages"
subtitle: "以geek的方式寫blog"
date: 2018-10-01T22:47:09+08:00
lastmod: 2018-10-01T23:03:31+08:00
draft: false
categories: 
 - DevOps
tags: 
 - Git
 - Github
 - Hugo
 - Travis CI
---

簡單紀錄操作步驟, 以後再逐步補充說明

<!--more-->

### Github User Pages

#### git 操作

```bash
git init
git remote add origin https://github.com/rabit/rabit.github.io.git
touch .gitkeep
git add .gitkeep
git commit -m "Initial commit for User Pages"
git push -u origin master
git checkout --orphan hugo-code
git rm -rf .
echo "public/" >> .gitignore
git add .gitignore
git commit -m "create new branch for hugo code"
git push -u origin hugo-code
```

### Hugo 

```bash
hugo new site .
touch data/.gitkeep
touch layouts/.gitkeep
touch static/.gitkeep
git submodule add https://github.com/rabit/hugo-theme-jane.git themes/jane
cp -r themes/jane/exampleSite/content ./
cp themes/jane/exampleSite/config.toml ./
hugo server -w --bind=0.0.0.0 --baseURL=http://0.0.0.0:1313/ --buildDrafts --buildFuture ./
```

### Travis-CI

```bash
touch .travis.yml
vim .travis.yml
```
```yaml
# copy from https://axdlog.com/zh/2018/using-hugo-and-travis-ci-to-deploy-blog-to-github-pages-automatically/
# https://docs.travis-ci.com/user/deployment/pages/
# https://docs.travis-ci.com/user/languages/python/
language: python

python:
    - "3.6"

notifications:
  email: false

# ref: https://discourse.gohugo.io/t/solved-hugo-v44-extended-and-relocation-errors-on-travis/13029/3
before_install:
  # This workaround is required to avoid libstdc++ errors while running "extended" hugo with SASS support.
  - wget -q -O libstdc++6 http://security.ubuntu.com/ubuntu/pool/main/g/gcc-5/libstdc++6_5.4.0-6ubuntu1~16.04.10_amd64.deb
  - sudo dpkg --force-all -i libstdc++6

install:
    # install latest release version
    #- wget $(wget -qO- https://api.github.com/repos/gohugoio/hugo/releases/latest | sed -r -n '/browser_download_url/{/Linux-64bit.deb/{s@[^:]*:[[:space:]]*"([^"]*)".*@\1@g;p}}')
    - wget -qO- https://api.github.com/repos/gohugoio/hugo/releases/latest | sed -r -n '/browser_download_url/{/Linux-64bit.deb/{/extended/{s@[^:]*:[[:space:]]*"([^"]*)".*@\1@g;p}}}' | xargs wget
    - sudo dpkg -i hugo*.deb
    - pip install Pygments
    - rm -rf public 2> /dev/null

script:
    - hugo -t jane

deploy:
  provider: pages
  skip-cleanup: true
  github-token: $GITHUB_TOKEN  # Set in travis-ci.org dashboard, marked secure
  email: $GITHUB_EMAIL
  name: $GITHUB_USERNAME
  verbose: true
  keep-history: true
  local-dir: public
  target-branch: master # Branch to push local-dir contents to
  on:
    branch: hugo-code  # branch contains Hugo generator code
```

#### Generate Github Token

![](/image/github-token-generation.png)

#### Configure Travis CI

##### 設定環境變數
![](/image/travis-ci-environments.png)
![](/image/travis-ci-settings.png)

#### 推送 .travis.ym 觸發 Travis-CI build stage
```bash
git checkout hugo-code
git add .
git commit -m "Add hugo code, and travis-ci settings"
git push -u origin hugo-code
```

