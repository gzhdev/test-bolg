---
title: day.0 是什么，为什么，怎么样 
date: 2023-04-01 01:52:25
tags: 
    - [技术集]
categories: 
    - [日记,公告]
top: 1
---

# 1. 这个博客是个啥？
这是个用于~~发癫~~记录每日有价值的事情的博客。听起来是不是听像日记的？所以我就叫它赛博笔记本了。

# 2. 为什么要建立这个博客？
- 用于记录解决过的问题，不用翻收藏夹。
- 用于记录生活的中的琐事，~~拯救我日益退化的记忆力~~。
- 未完待续

# 3. 怎样部署的这个博客？
博客使用[Hexo博客框架](https://hexo.io/zh-cn/)和[Next主题](https://github.com/next-theme/awesome-next)。本人技术水平不算很高，就不班门弄斧了，可以参考以下资料：
- [Next官方文档](https://theme-next.js.org/docs/)
- [Next中文文档](https://theme-next.iissnan.com/getting-started.html)
- [Hexo-NexT 主题个性优化](https://guanqr.com/tech/website/hexo-theme-next-customization/)
- [Hexo-Next 主题博客个性化配置超详细，超全面(两万字)](https://blog.csdn.net/as480133937/article/details/100138838)


我在这里主要讲如何在[Cloudflare Pages](https://pages.cloudflare.com/)和[Github Pages](https://pages.github.com/)上部署博客。
## 前置条件
- 一个可以运行的Hexo博客
- Cloudflare/Github账户
- 良好的网络连接情况
## 在Github使用静态界面部署
1. 一般来说，在Hexo框架配置文件中配置好`deploy:`代码块，并确认能向Github仓库推送文件后，通过`hexo d`方法可以很方便的将编译出的静态文件(即`/pubic`文件夹中的文件)推送到仓库中。  
2. 此时，在仓库的`Settings > Pages > Branch`中配置静态文件所在的分支和目录后保存。刷新网页将分配的域名填入Hexo框架配置文件中的`url:`参数中。  
3. 重新推送仓库，部署完成，访问步骤2中获得的域名即可访问博客。

> Tips: 
> 1. Github Pages的域名是`<你的Github用户名>.github.io`——在你的仓库名称和域名一样的情况下是这样的。
> 2. 当仓库名称不是`<你的Github用户名>.github.io`时，你仓库所对应的域名为`<你的Github用户名>.github.io/<仓库名>`,以路由的形式指向仓库。
> 3. 确认好你的Github Pages域名,需要填入Hexo框架配置文件中的`url:`参数中。

![启动Github Pages](https://s2.loli.net/2023/04/02/l16HSR8f3e7rdp5.png "启动Github Pages")
<center style="font-size:12px;">启动Github Pages</center><br>

![获取域名](https://s2.loli.net/2023/04/02/SGXYwb9nxu4A8Em.png "获取域名") 
<center style="font-size:12px;">获取域名</center><br>

至于配置自定义域名啥的我就不讲了，网上一搜一大把。


## 在Github使用Github Action部署
Github提供了一种测试版的方式——使用Github Action对整个项目进行编译以进行部署。这种方式。。。。。。实际上并没有啥区别，毕竟最后都是把`/public`文件夹内的静态文件上传到Github Pages。官方的说法是最适合使用框架和自定义构建过程(Best for using frameworks and customizingyour build process.)。但我觉得唯一的好处应该是不用新开一个分支用来存储原始项目了。
![第二种来源——Github Action](https://s2.loli.net/2023/04/02/zO3VpNKE6TbeWtF.png "第二种来源——Github Action")
<center style="font-size:12px;">第二种来源——Github Action</center><br>
然而，在Github提供的工作流文件中并没有Hexo的模板(Cloudflare也没有，不知道啥情况)，但可以参考Github的给出的其他文件自己写一个。  
对比Jekyll和静态文件的工作流文件，可以看到三块类似的代码块，下面对这三块代码块进行解释：  

![Jekyll和静态文件的工作流文件](https://s2.loli.net/2023/04/02/WgCNM1ZhXHGzF8w.png "Jekyll和静态文件的工作流文件")
<center style="font-size:12px;">Jekyll和静态文件的工作流文件</center><br>

1. `Checkout & Setup Pages`——前者用于签出仓库，而后者用于启动Github Pages并提取有关站点的各种元数据。

    `actions/configure-pages`用于初始化Github Pages并提取有关站点的各种元数据：
    ![actions/configure-pages输出](https://s2.loli.net/2023/04/02/qH8Ri9QKfT76Jrp.png "actions/configure-pages输出")
   由于`actions/checkout`通过配置`submodules`参数可以签出Git子模块(Git Submodule)。所以理论上可以Fork一份Next主题的仓库进行修改，然后作为子模块添加到博客主仓库中，以简化主题的更新和提高代码复用。  

2. `actions/upload-pages-artifact`——用于打包构建完成的文件并作为一个工件上传，拥有两个参数：
    - path: 包含静态文件的目录的路径。默认值为`_site/`，即Jekyll工作流文件中定义的输出文件夹。
    - retention-days: 工件将在几天后过期。默认值`"1"`，即一天后过期
    - 上传的工件可以在对应的Github Action工作流运行的可视化图中找到

3. `actions/deploy-pages`——用于将`build`任务中上传的工件部署为GitHub Pages站点
   - 返回已部署的GitHub Pages的URL，`deploy`任务通过环境变量`url`获取返回值，该环境变量将显示在存储库的部署页（通过单击存储库主页上的“环境”进行访问）和工作流运行的可视化图中。

了解了这些Action的作用后，我们只需要修改一下Jekyll工作流的构建部分，然后给`actions/upload-pages-artifact`指定个静态文件的路径就行了。
```yaml
# Sample workflow for building and deploying a Hexo site to GitHub Pages
name: Deploy Hexo with GitHub Pages dependencies preinstalled

on:
  # Runs on pushes targeting the default branch
  push:
    branches: ["main"]

  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

# 设置GITHUB_TOKEN的权限以允许部署到GITHUB页面
permissions:
  contents: read
  pages: write
  id-token: write

# 只允许一次并发部署，跳过在进行中的运行和最近排队的运行之间排队的运行。
# 但是，不要取消正在进行的运行，因为我们希望完成这些生产部署。
concurrency:
  group: "pages"
  cancel-in-progress: false

jobs:
  # Build job
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v3
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          # If your repository depends on submodule, please see: https://github.com/actions/checkout
          submodules: recursive
      - name: Setup Pages
        uses: actions/configure-pages@v3
      - name: Use Node.js 18.13.0
        uses: actions/setup-node@v3
        with:
          node-version: "18.13.0"
      - name: Install Dependencies
        run: npm install
      - name: Cache NPM dependencies
        uses: actions/cache@v3
        with:
          path: node_modules
          key: ${{ runner.OS }}-npm-cache
          restore-keys: |
            ${{ runner.OS }}-npm-cache
      - name: Build with Hexo
        run: npm run build
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v1
        with:
          path: ./public

  # Deployment job
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v2
```
我使用`actions/setup-node`指定了nodejs版本与我本地的一致，并使用`actions/cache`缓存了NPM下载的依赖。这些操作参考了[Hexo官方文档的Github Action部分](https://hexo.io/zh-cn/docs/github-pages),区别在于官方文档里也是将静态文件推送到仓库而不是直接部署。
由于项目的`package.json`中定义的`build`脚本是`hexo generate`并且依赖中包含`"hexo": "^6.3.0",`，所以不需要在环境中全局安装`hexo-cli`了，直接使用`npm run build`构建，最后在`actions/upload-pages-artifact`中指定`path`参数到`/public`文件夹就行。
