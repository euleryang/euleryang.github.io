---
title: GitHub Actions 自动部署 Hexo
date: 2019-11-05 00:45:29
tags: 
- 技术
categories:
- 技术工具  
---

> GitHub 操作可让您灵活地构建自动化的软件部署生命周期工作流程。您可以编写个别任务、调用的操作，以及结合它们创建自定义工作流程。 工作流程是您可以在仓库中创建的自定义自动化流程，用于在 GitHub 上构建、测试、封装、发行或部署任何代码项目。


### 新建 Actions

#### Secret
在 GitHub 设置生成一个 Token，需要有仓库的读写权限。打开项目设置，增加 Secrets GH_TOKEN 保存刚刚生成的 Token。Actions 默认有个 secret GITHUB_TOKEN，试了下不成功。


#### Workflow
新建 Workflow 文件必须在 .github/workflows 目录中，采用 YAML 语法，名字随意，我命名为 deploy.yml。

```yaml
name: Hexo deployer # Actions 名字

on: # 触发条件
  push:
    branches:
      - hexo-config # 仅向 hexo-config 分支 push 时触发

jobs:
  build: # job id
    name: Build and publish # job 名，不写默认使用 job id
    runs-on: ubuntu-latest # 运行环境，可选 ubuntu-latest, ubuntu-18.04, ubuntu-16.04, windows-latest, windows-2019, windows-2016, macOS-latest, macOS-10.14

    steps:
      - uses: actions/checkout@v1

      - name: Use Node.js 10.x
        uses: actions/setup-node@v1
        with:
          node-version: 10.x

      - name: Setup hexo env
        run: |
          npm install hexo-cli -g
          npm install

      - name: Generate public files
        run: |
          hexo clean
          hexo g  

      - name: Deploy
        env:
          GH_REF: github.com/euleryang/euleryang.github.io.git
          GH_TOKEN: ${{ secrets.GH_TOKEN }}
        run: |
          git config --global user.name "euleryang"
          git config --global user.email "huan_yang2006@163.com"
          git clone https://${GH_REF} .deploy_git
          cd .deploy_git
          git checkout master
          cd ../
          mv .deploy_git/.git/ ./public/
          cd ./public/
          git add .
          git commit -m "CI built at `date +"%Y-%m-%d %H:%M:%S"`"
          git push --force --quiet "https://${GH_TOKEN}@${GH_REF}" master:master
```


[参考](https://honye.github.io/posts/eaaf4b45.html)