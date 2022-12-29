---
{"dg-publish":true,"permalink":"/3-resource-folder/simple-read/hugo-git-hub-action-pseudoyu/","tags":"gardenEntry"}
---

# Hugo + GitHub Action，搭建你的博客自动发布系统

作者: Yu Zhang | 3727 字, 8 分钟 | 2022-05-29 | 分类: [Tools](https://www.pseudoyu.com/zh/category/tools/)

[blog](https://www.pseudoyu.com/zh/tag/blog/), [cloudflare](https://www.pseudoyu.com/zh/tag/cloudflare/), [github](https://www.pseudoyu.com/zh/tag/github/), [github action](https://www.pseudoyu.com/zh/tag/github-action/), [github pages](https://www.pseudoyu.com/zh/tag/github-pages/), [hugo](https://www.pseudoyu.com/zh/tag/hugo/), [self-host](https://www.pseudoyu.com/zh/tag/self-host/), [serverless](https://www.pseudoyu.com/zh/tag/serverless/)

## 前言

在之前的一篇《[免费的个人博客系统搭建及部署解决方案（Hugo + GitHub Pages + Cusdis）](https://www.pseudoyu.com/zh/2022/03/24/free_blog_deploy_using_hugo_and_cusdis/)》中，我提到了自己通过 [Hugo](https://gohugo.io) 这个静态网站生成器来真正搭建我的个人博客，并在 Hugo 开源社区中 [hugo-theme-den](https://github.com/shaform/hugo-theme-den) 这个主题基础上进行了一些个人定制化改造和配置，满足了自己的需求。

我的方案主要分为以下几个核心部分：

1.  个人博客源仓库，对博客配置及所有文章 `.md` 源文件进行版本管理，配合 GitHub Action 进行自动化部署，自动生成静态站点推送到 GitHub Pages 博客发布仓库。
2.  GitHub Pages 博客发布仓库，以 `username.github.io` 形式命名的仓库，使用 GitHub Pages 实现网站部署，可以通过配置域名 CNAME 解析使用自定义域名。
3.  Hugo 主题仓库，fork 喜欢的主题，并对自己的个人定制化改造配置进行版本管理，通过 `git submodule` 的方式链接到个人博客源仓库。
4.  其他组件源仓库，如 [umami 网站数据统计](https://www.pseudoyu.com/zh/2022/05/21/free_blog_analysis_using_umami_vercel_and_heroku/)及 [Cusdis 网站评论系统](https://www.pseudoyu.com/zh/2022/05/24/free_and_lightweight_blog_comment_system_using_cusdis_and_railway/)等。

下文会对搭建、本地测试、自动化部署维护等过程进行详细讲解，希望对大家所有帮助。

## 使用 Hugo 搭建博客

![](https://pseudoyu.oss-cn-hangzhou.aliyuncs.com/images/hugo_website.png)

[Hugo](https://gohugo.io) 是用 Go 实现的博客工具，采用 Markdown 进行文章编辑，自动生成静态站点文件，支持丰富的主题配置，也可以通过 js 嵌入像是评论系统等插件，高度定制化。除了 Hugo 外， 还有 Gatsby、Jekyll、Hexo、Ghost 等选择，实现和使用都差不多，可以根据自己的偏好进行选择。

### 安装 Hugo

我使用的是 macOS，所以使用官方推荐的 homebrew 方式进行 hugo 程序的安装，其他系统也类似。

```
brew install hugo


```

完成后，使用以下命令进行验证：

```
hugo version


```

### 创建 Hugo 网站

通过上述命令安装 hugo 程序后，就可以通过 `hugo new site` 命令进行网站创建、配置与本地调试了。

```
hugo new site blog-test

```

![](https://pseudoyu.oss-cn-hangzhou.aliyuncs.com/images/hugo_new_site.png)

### 配置主题

当通过上文命令创建我们的站点后，需要进行主题配置，Hugo 社区有了很丰富的主题，可以通过官网 [Themes](https://themes.gohugo.io) 菜单选择自己喜欢的风格，查看预览效果，选择后可以进入主题项目仓库，一般都会有很详细的安装及配置说明。下面我就以我目前在使用的 [hugo-theme-den](https://github.com/shaform/hugo-theme-den) 这个主题为例，演示一下配置流程。

#### 关联主题仓库

我们可以将主题仓库直接 `git clone` 下来进行使用，但这种方式有一些弊端，当之后自己对主题进行修改后，可能会与原主题产生一些冲突，不方便版本管理与后续更新。我采用的是将原主题仓库 `fork` 到自己的账户，并使用 `git submodule` 方式进行仓库链接，这样后续可以对主题的修改进行单独维护。

```
cd blog-test/
git init
git submodule add https://github.com/pseudoyu/hugo-theme-den themes/hugo-theme-den


```

![](https://pseudoyu.oss-cn-hangzhou.aliyuncs.com/images/hugo_init_theme.png)

#### 更新主题

如果是 clone 了其他人的博客项目进行修改，则需要用以下命令进行初始化：

```
git submodule update --init --recursive


```

如果需要同步主题仓库的最新修改，需要运行以下命令：

```
git submodule update --remote


```

#### 初始化主题配置及发布

每个主题一般都会提供一些实例配置与初始页面，开始使用主题时可以将其 `exampleSite/` 目录下的文件复制到站点目录下，在此基础上进行调整配置。

```
cp -rf themes/hugo-theme-den/exampleSite/* ./


```

初始化主题基础配置后，我们可以在 `config.toml` 文件中进行站点细节配置，具体配置项参考各主题说明文档。

![](https://pseudoyu.oss-cn-hangzhou.aliyuncs.com/images/hugo_theme_config.png)

完成后，可以通过 `hugo new` 命令发布新文章。

```
hugo new posts/blog-test.md


```

![](https://pseudoyu.oss-cn-hangzhou.aliyuncs.com/images/hugo_new_post.png)

#### 本地调试站点

Hugo 会生成静态网页，我们在本地编辑调试时可以通过 `hugo server` 命令进行本地实时调试预览，无须每次都重新生成。

```
hugo server


```

![](https://pseudoyu.oss-cn-hangzhou.aliyuncs.com/images/hugo_server.png)

运行服务后，我们可以通过浏览器 `http://localhost:1313` 地址访问我们的本地预览网页。

![](https://pseudoyu.oss-cn-hangzhou.aliyuncs.com/images/hugo_server_preview.png)

### 使用 GitHub Pages 前期准备

#### 域名购买

作为一个对外发布的网站，我们需要购买一个域名并配置解析，指向我们网站所在的服务器，才能让外界以比较方便的方式访问。域名购买平台很多，我用过的有 [Cloudflare](https://www.cloudflare.com)、[NameSilo](https://www.namesilo.com)、[GoDaddy](https://www.godaddy.com) 等，我最后常用的还是 Cloudflare，因为其同时还提供了 CDN、网站数据分析、定制规则等强大功能。

首先我们需要注册一个 Cloudflare 账户，登录后选择左侧边栏的 “注册域”，并搜索自己想注册的域名。

![](https://pseudoyu.oss-cn-hangzhou.aliyuncs.com/images/cloudflare_register_domain.png)

选择了心仪的域名后，点击并选择购买时限并填写个人信息。

![](https://pseudoyu.oss-cn-hangzhou.aliyuncs.com/images/cloudflare_register_domain_choose.png)

选择付款方式，建议可以选择自动续订，以免忘记续费。

![](https://pseudoyu.oss-cn-hangzhou.aliyuncs.com/images/cloudflare_register_domain_payment.png)

类型选择 Personal 即可，并点击完成购买。

![](https://pseudoyu.oss-cn-hangzhou.aliyuncs.com/images/cloudflare_register_done.png)

等待 Cloudflare 处理后即可查看信息。

![](https://pseudoyu.oss-cn-hangzhou.aliyuncs.com/images/cloudflare_domain.jpeg)

#### GitHub Pages 仓库

GitHub Pages 项目需要符合 `username.github.io` 的特殊命名格式，仓库建立完成后，可以在设置中配置自己注册的自定义域名来指向 GitHub Pages 生成的网址。此外，需要将博客站点配置文件 `config.toml` 中的 `baseURL` 改为自己的自定义域名，格式为 `"https://www.pseudoyu.com/"`，这样博客站点才能正常访问 GitHub Pages 生成的网站服务。

![](https://pseudoyu.oss-cn-hangzhou.aliyuncs.com/images/github_pages_repo.png)

#### 域名解析

按照上文步骤注册好后，需要在域名托管商进行 DNS 解析，在这里我们需要选择 CNAME，指向我们的 GitHub Pages 网址。

![](https://pseudoyu.oss-cn-hangzhou.aliyuncs.com/images/cloudflare_cname_config.png)

因为 CNAME 解析没办法设置 root 域名，即只能设置 `www.pseudoyu.com` 或其他子域名，而不是 `pseudoyu.com`，因此，我们可以通过 Cloudflare 上自定义规则设置域名重定向，具体配置如下，仅需将我的域名替换成自己的域名即可。即使你是通过 NameSilo 注册的域名，也可以通过 Cloudflare 来添加站点以实现功能，或者其他托管平台也有类似的功能，按照说明配置即可。

![](https://pseudoyu.oss-cn-hangzhou.aliyuncs.com/images/cloudflare_cname_rule_config.png)

### GitHub Pages 发布博客

完成上述准备工作后，我们现在已经可以通过自定义域名来访问我们的 GitHub Pages 页面了，目前因为项目仓库是空的，访问后会报 `404` 页面。

我们希望 Hugo 生成的静态网站能通过 GitHub Pages 服务进行托管，而无需自己维护服务，更稳定、安全，因此我们需要上传 Hugo 生成的静态网页文件至 GitHub Page 项目仓库。

#### 手动发布

当我们编辑博客内容并通过 `hugo server` 本地调试后，就可以通过 `hugo` 命令生成静态网页文件了。

```
hugo
cd public/


```

![](https://pseudoyu.oss-cn-hangzhou.aliyuncs.com/images/hugo_gen_pages.png)

Hugo 默认会将生成的静态网页文件存放在 `public/` 目录下，我们可以通过将 `public/` 目录初始化为 git 仓库并关联我们的 `pseudoyu/pseudoyu.github.io` 远程仓库来推送我们的网页静态文件。

```
git init
git remote add origin git@github.com:pseudoyu/pseudoyu.github.io
git add .
git commit -m "add test"


```

![](https://pseudoyu.oss-cn-hangzhou.aliyuncs.com/images/hugo_public_init.png)

核对文件修改后，即可通过 `git push origin master` 推送到 GitHub Pages 仓库，稍等几分钟即可通过我们的自定义域名来访问我们的博客站点了，和我们 `hugo server` 本地调试完全一致。

#### 自动发布

通过上述命令我们可以手动发布我们的静态文件，但还是有以下弊端：

1.  发布步骤还是比较繁琐，本地调试后还需要切换到 `public/` 目录进行上传
2.  无法对博客 `.md` 源文件进行备份与版本管理

因此，我们需要简单顺滑的方式来进行博客发布，首先我们初始化博客源文件的仓库，如我的仓库为 [pseudoyu/yu-blog](https://github.com/pseudoyu/yu-blog)。

因为我们的博客基于 GitHub 与 GitHub Pages，可以通过官方提供的 GitHub Action 进行 CI 自动发布，下面我会进行详细讲解。GitHub Action 是一个持续集成和持续交付 (CI/CD) 平台，可用于自动执行构建、测试和部署管道，目前已经有很多开发好的工作流，可以通过简单的配置即可直接使用。

配置在仓库目录 `.github/workflows` 下，以 `.yml` 为后缀。我的 GitHub Action 配置为 [pseudoyu/yu-blog deploy.yml](https://github.com/pseudoyu/yu-blog/blob/master/.github/workflows/deploy.yml)，自动发布示例配置如下：

```
name: deploy

on:
    push:
    workflow_dispatch:
    schedule:
        # Runs everyday at 8:00 AM
        - cron: "0 0 * * *"

jobs:
    build:
        runs-on: ubuntu-latest
        steps:
            - name: Checkout
              uses: actions/checkout@v2
              with:
                  submodules: true
                  fetch-depth: 0

            - name: Setup Hugo
              uses: peaceiris/actions-hugo@v2
              with:
                  hugo-version: "latest"

            - name: Build Web
              run: hugo

            - name: Deploy Web
              uses: peaceiris/actions-gh-pages@v3
              with:
                  PERSONAL_TOKEN: ${{ secrets.PERSONAL_TOKEN }}
                  EXTERNAL_REPOSITORY: pseudoyu/pseudoyu.github.io
                  PUBLISH_BRANCH: master
                  PUBLISH_DIR: ./public
                  commit_message: ${{ github.event.head_commit.message }}


```

`on` 表示 GitHub Action 触发条件，我设置了 `push`、`workflow_dispatch` 和 `schedule` 三个条件：

*   `push`，当这个项目仓库发生推送动作后，执行 GitHub Action
*   `workflow_dispatch`，可以在 GitHub 项目仓库的 Action 工具栏进行手动调用
*   `schedule`，定时执行 GitHub Action，如我的设置为北京时间每天早上执行，主要是使用一些自动化统计 CI 来自动更新我博客的关于页面，如本周编码时间，影音记录等，如果你不需要定时功能，可以删除这个条件

`jobs` 表示 GitHub Action 中的任务，我们设置了一个 `build` 任务，`runs-on` 表示 GitHub Action 运行环境，我们选择了 `ubuntu-latest`。我们的 `build` 任务包含了 `Checkout`、`Setup Hugo`、`Build Web` 和 `Deploy Web` 四个主要步骤，其中 `run` 是执行的命令，`uses` 是 GitHub Action 中的一个插件，我们使用了 `peaceiris/actions-hugo@v2` 和 `peaceiris/actions-gh-pages@v3` 这两个插件。其中 `Checkout` 步骤中 `with` 中配置 `submodules` 值为 `true` 可以同步博客源仓库的子模块，即我们的主题模块。

首先需要将上述 `deploy.yml` 中的 `EXTERNAL_REPOSITORY` 改为自己的 GitHub Pages 仓库，如我的设置为 `pseudoyu/pseudoyu.github.io`。

因为我们需要从博客仓库推送到外部 GitHub Pages 仓库，需要特定权限，要在 GitHub 账户下 `Setting - Developer setting - Personal access tokens` 下创建一个 Token。

![](https://pseudoyu.oss-cn-hangzhou.aliyuncs.com/images/github_psersonal_access_token.png)

权限需要开启 `repo` 与 `workflow`。

![](https://pseudoyu.oss-cn-hangzhou.aliyuncs.com/images/yu_blog_personal_token.png)

配置后复制生成的 Token（注：只会出现一次），然后在我们博客源仓库的 `Settings - Secrets - Actions` 中添加 `PERSONAL_TOKEN` 环境变量为刚才的 Token，这样 GitHub Action 就可以获取到 Token 了。

完成上述配置后，推送代码至仓库，即可触发 GitHub Action，自动生成博客页面并推送至 GitHub Pages 仓库。

![](https://pseudoyu.oss-cn-hangzhou.aliyuncs.com/images/yu_blog_ci.png)

而 GitHub Pages 仓库更新后，又会自动触发官方页面部署 CI，实现我们的网站发布。

![](https://pseudoyu.oss-cn-hangzhou.aliyuncs.com/images/page_build_ci.png)

经过上述配置，我们已经实现了 Hugo 博客本地搭建及版本管理、GitHub Pages 部署网站发布，Hugp 主题管理及更新等功能，实现了完整的系统。现在每当我们本地通过熟悉的 Markdown 语法完成博客内容编辑后，只需要推送代码，等待几分钟，即可通过我们的自定义域名访问更新后的网站。

### 组件拓展

一个完整的博客系统还需要一些组件，如网站数据统计、评论系统等，我针对这两个核心需求也写了完整的 Serverless 搭建教程，可根据需求进行部署配置。

*   [从零开始搭建一个免费的个人博客数据统计系统（umami + Vercel + Heroku）](https://www.pseudoyu.com/zh/2022/05/21/free_blog_analysis_using_umami_vercel_and_heroku/)
*   [轻量级开源免费博客评论系统解决方案 （Cusdis + Railway）](https://www.pseudoyu.com/zh/2022/05/24/free_and_lightweight_blog_comment_system_using_cusdis_and_railway/)

## 总结

以上就是我通过 Hugo 与 GitHub Action 实现的免费博客自动部署系统，我自己的实现仓库在 [pseudoyu/yu-blog](https://github.com/pseudoyu/yu-blog) 仓库中，我定制化的主题仓库在 [pseudoyu/hugo-theme-den](https://github.com/pseudoyu/hugo-theme-den) 中。

我使用 GitHub Action 还实现了很多好玩的自动化个人统计功能，自动更新我的 [GitHub Profile](https://github.com/pseudoyu)，项目仓库为 [pseudoyu/pseudoyu](https://github.com/pseudoyu/pseudoyu)，可以进入 `.github/workflows` 中自行探索。这些系统还在不断完善中，欢迎大家参与贡献与交流。

## 参考资料

1.  [Hugo 官网](https://gohugo.io)
2.  [GitHub Action](https://github.com/features/actions)
3.  [GitHub Pages](https://pages.github.com)
4.  [Cloudflare 官网](https://www.cloudflare.com)
5.  [免费的个人博客系统搭建及部署解决方案（Hugo + GitHub Pages + Cusdis）](https://www.pseudoyu.com/zh/2022/03/24/free_blog_deploy_using_hugo_and_cusdis/)
6.  [从零开始搭建一个免费的个人博客数据统计系统（umami + Vercel + Heroku）](https://www.pseudoyu.com/zh/2022/05/21/free_blog_analysis_using_umami_vercel_and_heroku/)
7.  [轻量级开源免费博客评论系统解决方案 （Cusdis + Railway）](https://www.pseudoyu.com/zh/2022/05/24/free_and_lightweight_blog_comment_system_using_cusdis_and_railway/)
8.  [我的 Pseudoyu 个人博客](https://www.pseudoyu.com)
9.  [我的 GitHub Profile](https://github.com/pseudoyu)

## 相关文章

2022-06-12

[2022 年了，聊聊我为什么还在写博客](/zh/2022/06/12/why_i_still_write_blog_in_2022/)

2022-05-29

[Ghost 5.0 来了，使用 Digital Ocean 一键部署吧](/zh/2022/05/29/deploy_ghost_5_on_digital_ocean_vps/)

2022-05-24

[轻量级开源免费博客评论系统解决方案 （Cusdis + Railway）](/zh/2022/05/24/free_and_lightweight_blog_comment_system_using_cusdis_and_railway/)

2022-05-21

[从零开始搭建一个免费的个人博客数据统计系统（umami + Vercel + Heroku）](/zh/2022/05/21/free_blog_analysis_using_umami_vercel_and_heroku/)

2022-03-24

[免费的个人博客系统搭建及部署解决方案（Hugo + GitHub Pages + Cusdis）](/zh/2022/03/24/free_blog_deploy_using_hugo_and_cusdis/)

![](https://www.pseudoyu.com/images/author.webp)

作者

Yu Zhang

区块链开发工程师，HKU ECICer。喜欢探索新技术，空闲时也折腾 Logseq 等效率工具。 在 [GitHub](https://github.com/pseudoyu) 关注我。在我的 [Telegram 频道](https://t.me/pseudoyulife)了解更多。

  

网站地图

*   [标签](https://www.pseudoyu.com/zh/tags/)
*   [分类](https://www.pseudoyu.com/zh/categories/)
*   [RSS](https://www.pseudoyu.com/zh/index.xml)

社交

*   [Telegram](https://t.me/pseudoyulife)
*   [Twitter](https://twitter.com/pseudo_yu)
*   [Instagram](https://www.instagram.com/pseudo.yu/)
*   [BiliBili](https://space.bilibili.com/5374948/)

关于

*   [Yu's GitHub](https://github.com/pseudoyu)
*   [M1sty's Blog](https://www.m1sty.com/)
*   [VPS Status](https://stats.uptimerobot.com/lPN5YU1wM0)

_[Yu's blog analytics](https://data.pseudoyu.com/share/8YKX7FUa/pseudoyu-blog)_  
_Powered by [Hugo](https://gohugo.io) - [den](https://github.com/shaform/hugo-theme-den)_  
_[The University of Hong Kong](https://www.hku.hk) - [CS](https://www.cs.hku.hk)_  
© Yu Zhang 2020 - 2022

#embedim--snow{position:fixed;left:0;top:0;bottom:0;width:100vw;height:100vh;overflow:hidden;z-index:9999999;pointer-events:none}.embedim-snow{position: absolute;width: 10px;height: 10px;background: white;border-radius: 50%;margin-top:-10px}.embedim-snow:nth-child(1){opacity:0.82;transform:translate(27.60vw,-10px) scale(0.71);animation:fall-1 26s -28s linear infinite}@keyframes fall-1{40.00%{transform:translate(24.44vw,40.00vh) scale(0.71)}to{transform:translate(26.02vw, 105vh) scale(0.71)}}.embedim-snow:nth-child(2){opacity:0.79;transform:translate(41.88vw,-10px) scale(0.11);animation:fall-2 28s -1s linear infinite}@keyframes fall-2{80.00%{transform:translate(35.33vw,80.00vh) scale(0.11)}to{transform:translate(38.61vw, 105vh) scale(0.11)}}.embedim-snow:nth-child(3){opacity:0.34;transform:translate(27.08vw,-10px) scale(0.38);animation:fall-3 30s -6s linear infinite}@keyframes fall-3{80.00%{transform:translate(31.02vw,80.00vh) scale(0.38)}to{transform:translate(29.05vw, 105vh) scale(0.38)}}.embedim-snow:nth-child(4){opacity:0.51;transform:translate(8.03vw,-10px) scale(0.13);animation:fall-4 12s -12s linear infinite}@keyframes fall-4{70.00%{transform:translate(4.20vw,70.00vh) scale(0.13)}to{transform:translate(6.11vw, 105vh) scale(0.13)}}.embedim-snow:nth-child(5){opacity:0.22;transform:translate(51.48vw,-10px) scale(0.89);animation:fall-5 19s -2s linear infinite}@keyframes fall-5{50.00%{transform:translate(47.83vw,50.00vh) scale(0.89)}to{transform:translate(49.65vw, 105vh) scale(0.89)}}.embedim-snow:nth-child(6){opacity:0.43;transform:translate(11.20vw,-10px) scale(0.63);animation:fall-6 25s -13s linear infinite}@keyframes fall-6{40.00%{transform:translate(9.55vw,40.00vh) scale(0.63)}to{transform:translate(10.38vw, 105vh) scale(0.63)}}.embedim-snow:nth-child(7){opacity:0.40;transform:translate(18.26vw,-10px) scale(0.69);animation:fall-7 24s -22s linear infinite}@keyframes fall-7{30.00%{transform:translate(23.84vw,30.00vh) scale(0.69)}to{transform:translate(21.05vw, 105vh) scale(0.69)}}.embedim-snow:nth-child(8){opacity:0.95;transform:translate(94.98vw,-10px) scale(0.85);animation:fall-8 20s -10s linear infinite}@keyframes fall-8{70.00%{transform:translate(103.10vw,70.00vh) scale(0.85)}to{transform:translate(99.04vw, 105vh) scale(0.85)}}.embedim-snow:nth-child(9){opacity:0.33;transform:translate(47.87vw,-10px) scale(0.19);animation:fall-9 19s -21s linear infinite}@keyframes fall-9{80.00%{transform:translate(45.71vw,80.00vh) scale(0.19)}to{transform:translate(46.79vw, 105vh) scale(0.19)}}.embedim-snow:nth-child(10){opacity:0.99;transform:translate(62.85vw,-10px) scale(0.98);animation:fall-10 23s -14s linear infinite}@keyframes fall-10{60.00%{transform:translate(56.60vw,60.00vh) scale(0.98)}to{transform:translate(59.73vw, 105vh) scale(0.98)}}.embedim-snow:nth-child(11){opacity:0.46;transform:translate(88.24vw,-10px) scale(0.58);animation:fall-11 14s -22s linear infinite}@keyframes fall-11{60.00%{transform:translate(92.45vw,60.00vh) scale(0.58)}to{transform:translate(90.35vw, 105vh) scale(0.58)}}.embedim-snow:nth-child(12){opacity:0.47;transform:translate(3.75vw,-10px) scale(0.21);animation:fall-12 30s -13s linear infinite}@keyframes fall-12{70.00%{transform:translate(12.57vw,70.00vh) scale(0.21)}to{transform:translate(8.16vw, 105vh) scale(0.21)}}.embedim-snow:nth-child(13){opacity:0.84;transform:translate(48.11vw,-10px) scale(0.40);animation:fall-13 15s -25s linear infinite}@keyframes fall-13{50.00%{transform:translate(43.20vw,50.00vh) scale(0.40)}to{transform:translate(45.66vw, 105vh) scale(0.40)}}.embedim-snow:nth-child(14){opacity:0.43;transform:translate(75.69vw,-10px) scale(0.07);animation:fall-14 20s -10s linear infinite}@keyframes fall-14{70.00%{transform:translate(72.45vw,70.00vh) scale(0.07)}to{transform:translate(74.07vw, 105vh) scale(0.07)}}.embedim-snow:nth-child(15){opacity:0.40;transform:translate(78.30vw,-10px) scale(0.04);animation:fall-15 29s -7s linear infinite}@keyframes fall-15{30.00%{transform:translate(78.55vw,30.00vh) scale(0.04)}to{transform:translate(78.43vw, 105vh) scale(0.04)}}.embedim-snow:nth-child(16){opacity:0.76;transform:translate(41.76vw,-10px) scale(0.66);animation:fall-16 10s -3s linear infinite}@keyframes fall-16{60.00%{transform:translate(41.58vw,60.00vh) scale(0.66)}to{transform:translate(41.67vw, 105vh) scale(0.66)}}.embedim-snow:nth-child(17){opacity:0.64;transform:translate(72.69vw,-10px) scale(0.20);animation:fall-17 30s -12s linear infinite}@keyframes fall-17{30.00%{transform:translate(80.31vw,30.00vh) scale(0.20)}to{transform:translate(76.50vw, 105vh) scale(0.20)}}.embedim-snow:nth-child(18){opacity:0.38;transform:translate(72.74vw,-10px) scale(0.14);animation:fall-18 18s -14s linear infinite}@keyframes fall-18{80.00%{transform:translate(79.58vw,80.00vh) scale(0.14)}to{transform:translate(76.16vw, 105vh) scale(0.14)}}.embedim-snow:nth-child(19){opacity:0.57;transform:translate(15.78vw,-10px) scale(0.23);animation:fall-19 18s -28s linear infinite}@keyframes fall-19{60.00%{transform:translate(15.50vw,60.00vh) scale(0.23)}to{transform:translate(15.64vw, 105vh) scale(0.23)}}.embedim-snow:nth-child(20){opacity:0.95;transform:translate(32.72vw,-10px) scale(0.83);animation:fall-20 23s -27s linear infinite}@keyframes fall-20{40.00%{transform:translate(36.55vw,40.00vh) scale(0.83)}to{transform:translate(34.63vw, 105vh) scale(0.83)}}.embedim-snow:nth-child(21){opacity:0.96;transform:translate(66.36vw,-10px) scale(0.19);animation:fall-21 22s -23s linear infinite}@keyframes fall-21{60.00%{transform:translate(63.58vw,60.00vh) scale(0.19)}to{transform:translate(64.97vw, 105vh) scale(0.19)}}.embedim-snow:nth-child(22){opacity:0.53;transform:translate(43.07vw,-10px) scale(0.77);animation:fall-22 21s -12s linear infinite}@keyframes fall-22{60.00%{transform:translate(42.15vw,60.00vh) scale(0.77)}to{transform:translate(42.61vw, 105vh) scale(0.77)}}.embedim-snow:nth-child(23){opacity:0.65;transform:translate(65.86vw,-10px) scale(0.10);animation:fall-23 30s -9s linear infinite}@keyframes fall-23{40.00%{transform:translate(67.45vw,40.00vh) scale(0.10)}to{transform:translate(66.66vw, 105vh) scale(0.10)}}.embedim-snow:nth-child(24){opacity:0.45;transform:translate(21.99vw,-10px) scale(0.62);animation:fall-24 10s -7s linear infinite}@keyframes fall-24{80.00%{transform:translate(26.73vw,80.00vh) scale(0.62)}to{transform:translate(24.36vw, 105vh) scale(0.62)}}.embedim-snow:nth-child(25){opacity:0.21;transform:translate(43.34vw,-10px) scale(0.53);animation:fall-25 27s -30s linear infinite}@keyframes fall-25{50.00%{transform:translate(39.50vw,50.00vh) scale(0.53)}to{transform:translate(41.42vw, 105vh) scale(0.53)}}.embedim-snow:nth-child(26){opacity:0.79;transform:translate(89.66vw,-10px) scale(0.91);animation:fall-26 29s -22s linear infinite}@keyframes fall-26{60.00%{transform:translate(98.63vw,60.00vh) scale(0.91)}to{transform:translate(94.14vw, 105vh) scale(0.91)}}.embedim-snow:nth-child(27){opacity:0.31;transform:translate(55.39vw,-10px) scale(0.48);animation:fall-27 24s -25s linear infinite}@keyframes fall-27{80.00%{transform:translate(59.03vw,80.00vh) scale(0.48)}to{transform:translate(57.21vw, 105vh) scale(0.48)}}.embedim-snow:nth-child(28){opacity:0.84;transform:translate(29.08vw,-10px) scale(0.83);animation:fall-28 14s -30s linear infinite}@keyframes fall-28{80.00%{transform:translate(29.18vw,80.00vh) scale(0.83)}to{transform:translate(29.13vw, 105vh) scale(0.83)}}.embedim-snow:nth-child(29){opacity:0.12;transform:translate(43.06vw,-10px) scale(0.91);animation:fall-29 18s -13s linear infinite}@keyframes fall-29{70.00%{transform:translate(46.63vw,70.00vh) scale(0.91)}to{transform:translate(44.84vw, 105vh) scale(0.91)}}.embedim-snow:nth-child(30){opacity:0.38;transform:translate(46.55vw,-10px) scale(0.51);animation:fall-30 10s -14s linear infinite}@keyframes fall-30{30.00%{transform:translate(56.21vw,30.00vh) scale(0.51)}to{transform:translate(51.38vw, 105vh) scale(0.51)}}.embedim-snow:nth-child(31){opacity:0.21;transform:translate(44.19vw,-10px) scale(0.76);animation:fall-31 23s -7s linear infinite}@keyframes fall-31{60.00%{transform:translate(37.39vw,60.00vh) scale(0.76)}to{transform:translate(40.79vw, 105vh) scale(0.76)}}.embedim-snow:nth-child(32){opacity:0.65;transform:translate(4.84vw,-10px) scale(0.83);animation:fall-32 25s -24s linear infinite}@keyframes fall-32{30.00%{transform:translate(-3.03vw,30.00vh) scale(0.83)}to{transform:translate(0.90vw, 105vh) scale(0.83)}}.embedim-snow:nth-child(33){opacity:0.41;transform:translate(30.45vw,-10px) scale(0.01);animation:fall-33 26s -16s linear infinite}@keyframes fall-33{50.00%{transform:translate(26.62vw,50.00vh) scale(0.01)}to{transform:translate(28.53vw, 105vh) scale(0.01)}}.embedim-snow:nth-child(34){opacity:0.60;transform:translate(38.29vw,-10px) scale(0.59);animation:fall-34 19s -26s linear infinite}@keyframes fall-34{50.00%{transform:translate(42.80vw,50.00vh) scale(0.59)}to{transform:translate(40.54vw, 105vh) scale(0.59)}}.embedim-snow:nth-child(35){opacity:0.22;transform:translate(47.40vw,-10px) scale(0.29);animation:fall-35 26s -0s linear infinite}@keyframes fall-35{40.00%{transform:translate(40.12vw,40.00vh) scale(0.29)}to{transform:translate(43.76vw, 105vh) scale(0.29)}}.embedim-snow:nth-child(36){opacity:0.13;transform:translate(15.98vw,-10px) scale(0.93);animation:fall-36 30s -7s linear infinite}@keyframes fall-36{60.00%{transform:translate(16.94vw,60.00vh) scale(0.93)}to{transform:translate(16.46vw, 105vh) scale(0.93)}}.embedim-snow:nth-child(37){opacity:0.61;transform:translate(67.03vw,-10px) scale(0.11);animation:fall-37 10s -3s linear infinite}@keyframes fall-37{80.00%{transform:translate(65.56vw,80.00vh) scale(0.11)}to{transform:translate(66.30vw, 105vh) scale(0.11)}}.embedim-snow:nth-child(38){opacity:0.84;transform:translate(73.45vw,-10px) scale(0.17);animation:fall-38 15s -26s linear infinite}@keyframes fall-38{60.00%{transform:translate(63.58vw,60.00vh) scale(0.17)}to{transform:translate(68.51vw, 105vh) scale(0.17)}}.embedim-snow:nth-child(39){opacity:0.87;transform:translate(37.13vw,-10px) scale(0.72);animation:fall-39 12s -26s linear infinite}@keyframes fall-39{70.00%{transform:translate(43.25vw,70.00vh) scale(0.72)}to{transform:translate(40.19vw, 105vh) scale(0.72)}}.embedim-snow:nth-child(40){opacity:0.92;transform:translate(87.35vw,-10px) scale(0.52);animation:fall-40 18s -16s linear infinite}@keyframes fall-40{70.00%{transform:translate(88.51vw,70.00vh) scale(0.52)}to{transform:translate(87.93vw, 105vh) scale(0.52)}}.embedim-snow:nth-child(41){opacity:0.11;transform:translate(60.47vw,-10px) scale(0.41);animation:fall-41 23s -13s linear infinite}@keyframes fall-41{80.00%{transform:translate(66.05vw,80.00vh) scale(0.41)}to{transform:translate(63.26vw, 105vh) scale(0.41)}}.embedim-snow:nth-child(42){opacity:0.81;transform:translate(42.79vw,-10px) scale(0.98);animation:fall-42 13s -22s linear infinite}@keyframes fall-42{40.00%{transform:translate(43.39vw,40.00vh) scale(0.98)}to{transform:translate(43.09vw, 105vh) scale(0.98)}}.embedim-snow:nth-child(43){opacity:0.99;transform:translate(51.03vw,-10px) scale(0.50);animation:fall-43 14s -21s linear infinite}@keyframes fall-43{50.00%{transform:translate(56.72vw,50.00vh) scale(0.50)}to{transform:translate(53.88vw, 105vh) scale(0.50)}}.embedim-snow:nth-child(44){opacity:0.80;transform:translate(85.05vw,-10px) scale(0.14);animation:fall-44 12s -1s linear infinite}@keyframes fall-44{70.00%{transform:translate(77.95vw,70.00vh) scale(0.14)}to{transform:translate(81.50vw, 105vh) scale(0.14)}}.embedim-snow:nth-child(45){opacity:0.84;transform:translate(44.48vw,-10px) scale(0.12);animation:fall-45 18s -18s linear infinite}@keyframes fall-45{70.00%{transform:translate(45.63vw,70.00vh) scale(0.12)}to{transform:translate(45.06vw, 105vh) scale(0.12)}}.embedim-snow:nth-child(46){opacity:0.30;transform:translate(24.39vw,-10px) scale(0.46);animation:fall-46 10s -6s linear infinite}@keyframes fall-46{50.00%{transform:translate(29.68vw,50.00vh) scale(0.46)}to{transform:translate(27.04vw, 105vh) scale(0.46)}}.embedim-snow:nth-child(47){opacity:0.70;transform:translate(23.86vw,-10px) scale(0.68);animation:fall-47 25s -18s linear infinite}@keyframes fall-47{80.00%{transform:translate(22.62vw,80.00vh) scale(0.68)}to{transform:translate(23.24vw, 105vh) scale(0.68)}}.embedim-snow:nth-child(48){opacity:0.82;transform:translate(30.63vw,-10px) scale(0.81);animation:fall-48 23s -30s linear infinite}@keyframes fall-48{80.00%{transform:translate(38.23vw,80.00vh) scale(0.81)}to{transform:translate(34.43vw, 105vh) scale(0.81)}}.embedim-snow:nth-child(49){opacity:0.07;transform:translate(47.21vw,-10px) scale(0.61);animation:fall-49 25s -28s linear infinite}@keyframes fall-49{50.00%{transform:translate(43.32vw,50.00vh) scale(0.61)}to{transform:translate(45.26vw, 105vh) scale(0.61)}}.embedim-snow:nth-child(50){opacity:0.04;transform:translate(71.73vw,-10px) scale(0.56);animation:fall-50 22s -0s linear infinite}@keyframes fall-50{70.00%{transform:translate(75.11vw,70.00vh) scale(0.56)}to{transform:translate(73.42vw, 105vh) scale(0.56)}}.embedim-snow:nth-child(51){opacity:0.35;transform:translate(57.86vw,-10px) scale(0.40);animation:fall-51 22s -7s linear infinite}@keyframes fall-51{40.00%{transform:translate(56.36vw,40.00vh) scale(0.40)}to{transform:translate(57.11vw, 105vh) scale(0.40)}}.embedim-snow:nth-child(52){opacity:0.56;transform:translate(78.69vw,-10px) scale(0.46);animation:fall-52 28s -25s linear infinite}@keyframes fall-52{40.00%{transform:translate(78.99vw,40.00vh) scale(0.46)}to{transform:translate(78.84vw, 105vh) scale(0.46)}}.embedim-snow:nth-child(53){opacity:0.04;transform:translate(41.90vw,-10px) scale(0.06);animation:fall-53 30s -28s linear infinite}@keyframes fall-53{50.00%{transform:translate(38.40vw,50.00vh) scale(0.06)}to{transform:translate(40.15vw, 105vh) scale(0.06)}}.embedim-snow:nth-child(54){opacity:0.62;transform:translate(35.73vw,-10px) scale(0.37);animation:fall-54 28s -2s linear infinite}@keyframes fall-54{70.00%{transform:translate(36.22vw,70.00vh) scale(0.37)}to{transform:translate(35.98vw, 105vh) scale(0.37)}}.embedim-snow:nth-child(55){opacity:0.42;transform:translate(49.98vw,-10px) scale(0.42);animation:fall-55 29s -26s linear infinite}@keyframes fall-55{70.00%{transform:translate(43.25vw,70.00vh) scale(0.42)}to{transform:translate(46.61vw, 105vh) scale(0.42)}}.embedim-snow:nth-child(56){opacity:0.30;transform:translate(22.53vw,-10px) scale(0.14);animation:fall-56 29s -25s linear infinite}@keyframes fall-56{50.00%{transform:translate(22.02vw,50.00vh) scale(0.14)}to{transform:translate(22.27vw, 105vh) scale(0.14)}}.embedim-snow:nth-child(57){opacity:0.17;transform:translate(30.49vw,-10px) scale(0.40);animation:fall-57 30s -29s linear infinite}@keyframes fall-57{70.00%{transform:translate(24.63vw,70.00vh) scale(0.40)}to{transform:translate(27.56vw, 105vh) scale(0.40)}}.embedim-snow:nth-child(58){opacity:0.89;transform:translate(21.11vw,-10px) scale(0.65);animation:fall-58 24s -3s linear infinite}@keyframes fall-58{80.00%{transform:translate(29.65vw,80.00vh) scale(0.65)}to{transform:translate(25.38vw, 105vh) scale(0.65)}}.embedim-snow:nth-child(59){opacity:0.97;transform:translate(93.61vw,-10px) scale(0.88);animation:fall-59 14s -12s linear infinite}@keyframes fall-59{50.00%{transform:translate(99.36vw,50.00vh) scale(0.88)}to{transform:translate(96.49vw, 105vh) scale(0.88)}}.embedim-snow:nth-child(60){opacity:1.00;transform:translate(58.81vw,-10px) scale(0.70);animation:fall-60 13s -5s linear infinite}@keyframes fall-60{60.00%{transform:translate(56.96vw,60.00vh) scale(0.70)}to{transform:translate(57.88vw, 105vh) scale(0.70)}}.embedim-snow:nth-child(61){opacity:0.82;transform:translate(77.13vw,-10px) scale(0.68);animation:fall-61 24s -18s linear infinite}@keyframes fall-61{70.00%{transform:translate(74.44vw,70.00vh) scale(0.68)}to{transform:translate(75.79vw, 105vh) scale(0.68)}}.embedim-snow:nth-child(62){opacity:0.62;transform:translate(70.47vw,-10px) scale(0.48);animation:fall-62 21s -7s linear infinite}@keyframes fall-62{80.00%{transform:translate(68.58vw,80.00vh) scale(0.48)}to{transform:translate(69.52vw, 105vh) scale(0.48)}}.embedim-snow:nth-child(63){opacity:0.63;transform:translate(88.73vw,-10px) scale(0.06);animation:fall-63 26s -3s linear infinite}@keyframes fall-63{40.00%{transform:translate(94.47vw,40.00vh) scale(0.06)}to{transform:translate(91.60vw, 105vh) scale(0.06)}}.embedim-snow:nth-child(64){opacity:0.12;transform:translate(84.58vw,-10px) scale(0.54);animation:fall-64 10s -10s linear infinite}@keyframes fall-64{30.00%{transform:translate(87.68vw,30.00vh) scale(0.54)}to{transform:translate(86.13vw, 105vh) scale(0.54)}}.embedim-snow:nth-child(65){opacity:0.10;transform:translate(28.96vw,-10px) scale(0.42);animation:fall-65 20s -26s linear infinite}@keyframes fall-65{50.00%{transform:translate(31.89vw,50.00vh) scale(0.42)}to{transform:translate(30.42vw, 105vh) scale(0.42)}}.embedim-snow:nth-child(66){opacity:0.55;transform:translate(47.96vw,-10px) scale(0.87);animation:fall-66 30s -12s linear infinite}@keyframes fall-66{30.00%{transform:translate(38.68vw,30.00vh) scale(0.87)}to{transform:translate(43.32vw, 105vh) scale(0.87)}}.embedim-snow:nth-child(67){opacity:0.08;transform:translate(32.53vw,-10px) scale(0.89);animation:fall-67 10s -23s linear infinite}@keyframes fall-67{40.00%{transform:translate(34.12vw,40.00vh) scale(0.89)}to{transform:translate(33.32vw, 105vh) scale(0.89)}}.embedim-snow:nth-child(68){opacity:0.94;transform:translate(59.93vw,-10px) scale(0.30);animation:fall-68 15s -13s linear infinite}@keyframes fall-68{60.00%{transform:translate(64.16vw,60.00vh) scale(0.30)}to{transform:translate(62.05vw, 105vh) scale(0.30)}}.embedim-snow:nth-child(69){opacity:0.91;transform:translate(15.72vw,-10px) scale(0.61);animation:fall-69 22s -11s linear infinite}@keyframes fall-69{30.00%{transform:translate(24.84vw,30.00vh) scale(0.61)}to{transform:translate(20.28vw, 105vh) scale(0.61)}}.embedim-snow:nth-child(70){opacity:0.95;transform:translate(18.20vw,-10px) scale(0.03);animation:fall-70 22s -25s linear infinite}@keyframes fall-70{30.00%{transform:translate(16.66vw,30.00vh) scale(0.03)}to{transform:translate(17.43vw, 105vh) scale(0.03)}}.embedim-snow:nth-child(71){opacity:0.31;transform:translate(69.11vw,-10px) scale(0.76);animation:fall-71 16s -1s linear infinite}@keyframes fall-71{70.00%{transform:translate(69.86vw,70.00vh) scale(0.76)}to{transform:translate(69.49vw, 105vh) scale(0.76)}}.embedim-snow:nth-child(72){opacity:0.61;transform:translate(51.74vw,-10px) scale(0.05);animation:fall-72 11s -23s linear infinite}@keyframes fall-72{30.00%{transform:translate(55.73vw,30.00vh) scale(0.05)}to{transform:translate(53.74vw, 105vh) scale(0.05)}}.embedim-snow:nth-child(73){opacity:0.99;transform:translate(69.38vw,-10px) scale(0.83);animation:fall-73 20s -15s linear infinite}@keyframes fall-73{80.00%{transform:translate(61.43vw,80.00vh) scale(0.83)}to{transform:translate(65.41vw, 105vh) scale(0.83)}}.embedim-snow:nth-child(74){opacity:0.81;transform:translate(71.98vw,-10px) scale(0.17);animation:fall-74 26s -7s linear infinite}@keyframes fall-74{40.00%{transform:translate(71.12vw,40.00vh) scale(0.17)}to{transform:translate(71.55vw, 105vh) scale(0.17)}}.embedim-snow:nth-child(75){opacity:0.86;transform:translate(42.39vw,-10px) scale(0.63);animation:fall-75 23s -5s linear infinite}@keyframes fall-75{80.00%{transform:translate(50.93vw,80.00vh) scale(0.63)}to{transform:translate(46.66vw, 105vh) scale(0.63)}}.embedim-snow:nth-child(76){opacity:0.58;transform:translate(79.37vw,-10px) scale(0.75);animation:fall-76 25s -3s linear infinite}@keyframes fall-76{70.00%{transform:translate(72.97vw,70.00vh) scale(0.75)}to{transform:translate(76.17vw, 105vh) scale(0.75)}}.embedim-snow:nth-child(77){opacity:0.42;transform:translate(28.05vw,-10px) scale(0.74);animation:fall-77 17s -10s linear infinite}@keyframes fall-77{30.00%{transform:translate(34.15vw,30.00vh) scale(0.74)}to{transform:translate(31.10vw, 105vh) scale(0.74)}}.embedim-snow:nth-child(78){opacity:0.06;transform:translate(10.54vw,-10px) scale(0.78);animation:fall-78 25s -2s linear infinite}@keyframes fall-78{40.00%{transform:translate(13.96vw,40.00vh) scale(0.78)}to{transform:translate(12.25vw, 105vh) scale(0.78)}}.embedim-snow:nth-child(79){opacity:0.66;transform:translate(88.29vw,-10px) scale(0.25);animation:fall-79 23s -17s linear infinite}@keyframes fall-79{70.00%{transform:translate(86.64vw,70.00vh) scale(0.25)}to{transform:translate(87.46vw, 105vh) scale(0.25)}}.embedim-snow:nth-child(80){opacity:0.33;transform:translate(92.00vw,-10px) scale(0.19);animation:fall-80 26s -21s linear infinite}@keyframes fall-80{80.00%{transform:translate(91.98vw,80.00vh) scale(0.19)}to{transform:translate(91.99vw, 105vh) scale(0.19)}}.embedim-snow:nth-child(81){opacity:0.17;transform:translate(23.51vw,-10px) scale(0.15);animation:fall-81 18s -1s linear infinite}@keyframes fall-81{50.00%{transform:translate(24.43vw,50.00vh) scale(0.15)}to{transform:translate(23.97vw, 105vh) scale(0.15)}}.embedim-snow:nth-child(82){opacity:0.60;transform:translate(88.49vw,-10px) scale(0.40);animation:fall-82 10s -4s linear infinite}@keyframes fall-82{30.00%{transform:translate(82.88vw,30.00vh) scale(0.40)}to{transform:translate(85.68vw, 105vh) scale(0.40)}}.embedim-snow:nth-child(83){opacity:0.96;transform:translate(5.10vw,-10px) scale(0.24);animation:fall-83 19s -1s linear infinite}@keyframes fall-83{60.00%{transform:translate(11.82vw,60.00vh) scale(0.24)}to{transform:translate(8.46vw, 105vh) scale(0.24)}}.embedim-snow:nth-child(84){opacity:0.33;transform:translate(49.67vw,-10px) scale(0.33);animation:fall-84 19s -28s linear infinite}@keyframes fall-84{30.00%{transform:translate(57.09vw,30.00vh) scale(0.33)}to{transform:translate(53.38vw, 105vh) scale(0.33)}}.embedim-snow:nth-child(85){opacity:0.74;transform:translate(24.14vw,-10px) scale(0.97);animation:fall-85 21s -18s linear infinite}@keyframes fall-85{60.00%{transform:translate(33.42vw,60.00vh) scale(0.97)}to{transform:translate(28.78vw, 105vh) scale(0.97)}}.embedim-snow:nth-child(86){opacity:0.20;transform:translate(75.87vw,-10px) scale(0.43);animation:fall-86 23s -1s linear infinite}@keyframes fall-86{30.00%{transform:translate(77.03vw,30.00vh) scale(0.43)}to{transform:translate(76.45vw, 105vh) scale(0.43)}}.embedim-snow:nth-child(87){opacity:0.67;transform:translate(51.59vw,-10px) scale(0.63);animation:fall-87 29s -12s linear infinite}@keyframes fall-87{80.00%{transform:translate(44.42vw,80.00vh) scale(0.63)}to{transform:translate(48.00vw, 105vh) scale(0.63)}}.embedim-snow:nth-child(88){opacity:0.71;transform:translate(97.40vw,-10px) scale(0.58);animation:fall-88 18s -30s linear infinite}@keyframes fall-88{40.00%{transform:translate(92.05vw,40.00vh) scale(0.58)}to{transform:translate(94.72vw, 105vh) scale(0.58)}}.embedim-snow:nth-child(89){opacity:0.78;transform:translate(8.97vw,-10px) scale(0.71);animation:fall-89 16s -21s linear infinite}@keyframes fall-89{80.00%{transform:translate(15.84vw,80.00vh) scale(0.71)}to{transform:translate(12.40vw, 105vh) scale(0.71)}}.embedim-snow:nth-child(90){opacity:0.44;transform:translate(3.68vw,-10px) scale(0.46);animation:fall-90 14s -3s linear infinite}@keyframes fall-90{60.00%{transform:translate(9.16vw,60.00vh) scale(0.46)}to{transform:translate(6.42vw, 105vh) scale(0.46)}}.embedim-snow:nth-child(91){opacity:0.41;transform:translate(43.65vw,-10px) scale(0.07);animation:fall-91 24s -28s linear infinite}@keyframes fall-91{30.00%{transform:translate(36.14vw,30.00vh) scale(0.07)}to{transform:translate(39.90vw, 105vh) scale(0.07)}}.embedim-snow:nth-child(92){opacity:0.93;transform:translate(31.30vw,-10px) scale(0.91);animation:fall-92 21s -27s linear infinite}@keyframes fall-92{50.00%{transform:translate(21.69vw,50.00vh) scale(0.91)}to{transform:translate(26.50vw, 105vh) scale(0.91)}}.embedim-snow:nth-child(93){opacity:0.35;transform:translate(4.13vw,-10px) scale(0.99);animation:fall-93 11s -11s linear infinite}@keyframes fall-93{70.00%{transform:translate(-2.37vw,70.00vh) scale(0.99)}to{transform:translate(0.88vw, 105vh) scale(0.99)}}.embedim-snow:nth-child(94){opacity:0.14;transform:translate(53.26vw,-10px) scale(0.20);animation:fall-94 19s -11s linear infinite}@keyframes fall-94{50.00%{transform:translate(55.12vw,50.00vh) scale(0.20)}to{transform:translate(54.19vw, 105vh) scale(0.20)}}.embedim-snow:nth-child(95){opacity:0.92;transform:translate(42.00vw,-10px) scale(0.36);animation:fall-95 19s -28s linear infinite}@keyframes fall-95{70.00%{transform:translate(38.98vw,70.00vh) scale(0.36)}to{transform:translate(40.49vw, 105vh) scale(0.36)}}.embedim-snow:nth-child(96){opacity:1.00;transform:translate(48.29vw,-10px) scale(0.81);animation:fall-96 24s -12s linear infinite}@keyframes fall-96{40.00%{transform:translate(53.74vw,40.00vh) scale(0.81)}to{transform:translate(51.01vw, 105vh) scale(0.81)}}.embedim-snow:nth-child(97){opacity:0.03;transform:translate(78.29vw,-10px) scale(0.51);animation:fall-97 22s -25s linear infinite}@keyframes fall-97{60.00%{transform:translate(87.56vw,60.00vh) scale(0.51)}to{transform:translate(82.93vw, 105vh) scale(0.51)}}.embedim-snow:nth-child(98){opacity:0.56;transform:translate(86.24vw,-10px) scale(0.73);animation:fall-98 27s -29s linear infinite}@keyframes fall-98{50.00%{transform:translate(85.71vw,50.00vh) scale(0.73)}to{transform:translate(85.98vw, 105vh) scale(0.73)}}.embedim-snow:nth-child(99){opacity:0.58;transform:translate(40.03vw,-10px) scale(0.78);animation:fall-99 20s -14s linear infinite}@keyframes fall-99{50.00%{transform:translate(47.51vw,50.00vh) scale(0.78)}to{transform:translate(43.77vw, 105vh) scale(0.78)}}.embedim-snow:nth-child(100){opacity:0.73;transform:translate(76.09vw,-10px) scale(0.02);animation:fall-100 17s -22s linear infinite}@keyframes fall-100{40.00%{transform:translate(71.20vw,40.00vh) scale(0.02)}to{transform:translate(73.65vw, 105vh) scale(0.02)}}.embedim-snow:nth-child(101){opacity:0.22;transform:translate(34.24vw,-10px) scale(0.69);animation:fall-101 30s -13s linear infinite}@keyframes fall-101{40.00%{transform:translate(35.47vw,40.00vh) scale(0.69)}to{transform:translate(34.86vw, 105vh) scale(0.69)}}.embedim-snow:nth-child(102){opacity:0.28;transform:translate(78.95vw,-10px) scale(0.30);animation:fall-102 11s -16s linear infinite}@keyframes fall-102{80.00%{transform:translate(74.28vw,80.00vh) scale(0.30)}to{transform:translate(76.62vw, 105vh) scale(0.30)}}.embedim-snow:nth-child(103){opacity:0.17;transform:translate(71.63vw,-10px) scale(0.74);animation:fall-103 19s -25s linear infinite}@keyframes fall-103{60.00%{transform:translate(65.53vw,60.00vh) scale(0.74)}to{transform:translate(68.58vw, 105vh) scale(0.74)}}.embedim-snow:nth-child(104){opacity:0.14;transform:translate(17.47vw,-10px) scale(0.12);animation:fall-104 22s -21s linear infinite}@keyframes fall-104{30.00%{transform:translate(13.12vw,30.00vh) scale(0.12)}to{transform:translate(15.30vw, 105vh) scale(0.12)}}.embedim-snow:nth-child(105){opacity:0.60;transform:translate(57.04vw,-10px) scale(0.51);animation:fall-105 15s -14s linear infinite}@keyframes fall-105{60.00%{transform:translate(56.68vw,60.00vh) scale(0.51)}to{transform:translate(56.86vw, 105vh) scale(0.51)}}.embedim-snow:nth-child(106){opacity:0.18;transform:translate(6.82vw,-10px) scale(0.22);animation:fall-106 12s -27s linear infinite}@keyframes fall-106{70.00%{transform:translate(10.62vw,70.00vh) scale(0.22)}to{transform:translate(8.72vw, 105vh) scale(0.22)}}.embedim-snow:nth-child(107){opacity:0.10;transform:translate(24.79vw,-10px) scale(0.68);animation:fall-107 10s -26s linear infinite}@keyframes fall-107{80.00%{transform:translate(28.18vw,80.00vh) scale(0.68)}to{transform:translate(26.49vw, 105vh) scale(0.68)}}.embedim-snow:nth-child(108){opacity:0.98;transform:translate(97.93vw,-10px) scale(0.62);animation:fall-108 15s -21s linear infinite}@keyframes fall-108{80.00%{transform:translate(91.75vw,80.00vh) scale(0.62)}to{transform:translate(94.84vw, 105vh) scale(0.62)}}.embedim-snow:nth-child(109){opacity:0.49;transform:translate(82.95vw,-10px) scale(0.83);animation:fall-109 11s -25s linear infinite}@keyframes fall-109{50.00%{transform:translate(92.86vw,50.00vh) scale(0.83)}to{transform:translate(87.91vw, 105vh) scale(0.83)}}.embedim-snow:nth-child(110){opacity:0.07;transform:translate(94.99vw,-10px) scale(0.86);animation:fall-110 18s -6s linear infinite}@keyframes fall-110{80.00%{transform:translate(91.02vw,80.00vh) scale(0.86)}to{transform:translate(93.00vw, 105vh) scale(0.86)}}.embedim-snow:nth-child(111){opacity:0.88;transform:translate(55.46vw,-10px) scale(0.36);animation:fall-111 15s -13s linear infinite}@keyframes fall-111{60.00%{transform:translate(45.61vw,60.00vh) scale(0.36)}to{transform:translate(50.54vw, 105vh) scale(0.36)}}.embedim-snow:nth-child(112){opacity:0.04;transform:translate(26.67vw,-10px) scale(0.03);animation:fall-112 17s -10s linear infinite}@keyframes fall-112{70.00%{transform:translate(25.06vw,70.00vh) scale(0.03)}to{transform:translate(25.86vw, 105vh) scale(0.03)}}.embedim-snow:nth-child(113){opacity:0.42;transform:translate(33.16vw,-10px) scale(0.50);animation:fall-113 20s -21s linear infinite}@keyframes fall-113{60.00%{transform:translate(31.69vw,60.00vh) scale(0.50)}to{transform:translate(32.43vw, 105vh) scale(0.50)}}.embedim-snow:nth-child(114){opacity:0.37;transform:translate(77.91vw,-10px) scale(0.13);animation:fall-114 16s -8s linear infinite}@keyframes fall-114{60.00%{transform:translate(84.42vw,60.00vh) scale(0.13)}to{transform:translate(81.17vw, 105vh) scale(0.13)}}.embedim-snow:nth-child(115){opacity:0.20;transform:translate(80.75vw,-10px) scale(0.12);animation:fall-115 21s -15s linear infinite}@keyframes fall-115{60.00%{transform:translate(84.24vw,60.00vh) scale(0.12)}to{transform:translate(82.49vw, 105vh) scale(0.12)}}.embedim-snow:nth-child(116){opacity:0.62;transform:translate(51.86vw,-10px) scale(0.96);animation:fall-116 18s -20s linear infinite}@keyframes fall-116{50.00%{transform:translate(42.41vw,50.00vh) scale(0.96)}to{transform:translate(47.14vw, 105vh) scale(0.96)}}.embedim-snow:nth-child(117){opacity:0.69;transform:translate(48.51vw,-10px) scale(0.16);animation:fall-117 14s -24s linear infinite}@keyframes fall-117{30.00%{transform:translate(46.95vw,30.00vh) scale(0.16)}to{transform:translate(47.73vw, 105vh) scale(0.16)}}.embedim-snow:nth-child(118){opacity:0.76;transform:translate(73.94vw,-10px) scale(0.41);animation:fall-118 11s -2s linear infinite}@keyframes fall-118{70.00%{transform:translate(83.44vw,70.00vh) scale(0.41)}to{transform:translate(78.69vw, 105vh) scale(0.41)}}.embedim-snow:nth-child(119){opacity:0.11;transform:translate(94.70vw,-10px) scale(0.84);animation:fall-119 29s -13s linear infinite}@keyframes fall-119{40.00%{transform:translate(84.98vw,40.00vh) scale(0.84)}to{transform:translate(89.84vw, 105vh) scale(0.84)}}.embedim-snow:nth-child(120){opacity:0.98;transform:translate(1.91vw,-10px) scale(0.60);animation:fall-120 20s -22s linear infinite}@keyframes fall-120{40.00%{transform:translate(2.92vw,40.00vh) scale(0.60)}to{transform:translate(2.41vw, 105vh) scale(0.60)}}.embedim-snow:nth-child(121){opacity:0.85;transform:translate(87.54vw,-10px) scale(0.12);animation:fall-121 30s -21s linear infinite}@keyframes fall-121{80.00%{transform:translate(83.21vw,80.00vh) scale(0.12)}to{transform:translate(85.38vw, 105vh) scale(0.12)}}.embedim-snow:nth-child(122){opacity:0.82;transform:translate(49.48vw,-10px) scale(0.92);animation:fall-122 10s -29s linear infinite}@keyframes fall-122{50.00%{transform:translate(52.08vw,50.00vh) scale(0.92)}to{transform:translate(50.78vw, 105vh) scale(0.92)}}.embedim-snow:nth-child(123){opacity:0.45;transform:translate(62.14vw,-10px) scale(0.00);animation:fall-123 20s -23s linear infinite}@keyframes fall-123{70.00%{transform:translate(54.77vw,70.00vh) scale(0.00)}to{transform:translate(58.45vw, 105vh) scale(0.00)}}.embedim-snow:nth-child(124){opacity:0.08;transform:translate(33.01vw,-10px) scale(0.74);animation:fall-124 16s -2s linear infinite}@keyframes fall-124{60.00%{transform:translate(37.18vw,60.00vh) scale(0.74)}to{transform:translate(35.09vw, 105vh) scale(0.74)}}.embedim-snow:nth-child(125){opacity:0.77;transform:translate(82.81vw,-10px) scale(0.56);animation:fall-125 20s -11s linear infinite}@keyframes fall-125{40.00%{transform:translate(73.48vw,40.00vh) scale(0.56)}to{transform:translate(78.15vw, 105vh) scale(0.56)}}.embedim-snow:nth-child(126){opacity:0.81;transform:translate(63.94vw,-10px) scale(0.46);animation:fall-126 29s -7s linear infinite}@keyframes fall-126{60.00%{transform:translate(64.52vw,60.00vh) scale(0.46)}to{transform:translate(64.23vw, 105vh) scale(0.46)}}.embedim-snow:nth-child(127){opacity:0.49;transform:translate(17.70vw,-10px) scale(0.74);animation:fall-127 13s -21s linear infinite}@keyframes fall-127{70.00%{transform:translate(15.02vw,70.00vh) scale(0.74)}to{transform:translate(16.36vw, 105vh) scale(0.74)}}.embedim-snow:nth-child(128){opacity:0.28;transform:translate(16.76vw,-10px) scale(0.63);animation:fall-128 27s -23s linear infinite}@keyframes fall-128{70.00%{transform:translate(20.85vw,70.00vh) scale(0.63)}to{transform:translate(18.80vw, 105vh) scale(0.63)}}.embedim-snow:nth-child(129){opacity:0.10;transform:translate(22.92vw,-10px) scale(0.03);animation:fall-129 19s -11s linear infinite}@keyframes fall-129{40.00%{transform:translate(17.26vw,40.00vh) scale(0.03)}to{transform:translate(20.09vw, 105vh) scale(0.03)}}.embedim-snow:nth-child(130){opacity:0.07;transform:translate(91.58vw,-10px) scale(0.35);animation:fall-130 14s -17s linear infinite}@keyframes fall-130{40.00%{transform:translate(85.58vw,40.00vh) scale(0.35)}to{transform:translate(88.58vw, 105vh) scale(0.35)}}.embedim-snow:nth-child(131){opacity:0.73;transform:translate(85.37vw,-10px) scale(0.42);animation:fall-131 15s -14s linear infinite}@keyframes fall-131{60.00%{transform:translate(94.84vw,60.00vh) scale(0.42)}to{transform:translate(90.11vw, 105vh) scale(0.42)}}.embedim-snow:nth-child(132){opacity:0.27;transform:translate(55.63vw,-10px) scale(0.83);animation:fall-132 28s -6s linear infinite}@keyframes fall-132{60.00%{transform:translate(56.20vw,60.00vh) scale(0.83)}to{transform:translate(55.92vw, 105vh) scale(0.83)}}.embedim-snow:nth-child(133){opacity:0.99;transform:translate(75.48vw,-10px) scale(0.17);animation:fall-133 24s -2s linear infinite}@keyframes fall-133{40.00%{transform:translate(68.82vw,40.00vh) scale(0.17)}to{transform:translate(72.15vw, 105vh) scale(0.17)}}.embedim-snow:nth-child(134){opacity:0.25;transform:translate(77.52vw,-10px) scale(0.64);animation:fall-134 28s -7s linear infinite}@keyframes fall-134{40.00%{transform:translate(82.26vw,40.00vh) scale(0.64)}to{transform:translate(79.89vw, 105vh) scale(0.64)}}.embedim-snow:nth-child(135){opacity:0.76;transform:translate(2.54vw,-10px) scale(0.07);animation:fall-135 20s -17s linear infinite}@keyframes fall-135{70.00%{transform:translate(7.64vw,70.00vh) scale(0.07)}to{transform:translate(5.09vw, 105vh) scale(0.07)}}.embedim-snow:nth-child(136){opacity:0.34;transform:translate(47.73vw,-10px) scale(0.08);animation:fall-136 21s -12s linear infinite}@keyframes fall-136{50.00%{transform:translate(52.99vw,50.00vh) scale(0.08)}to{transform:translate(50.36vw, 105vh) scale(0.08)}}.embedim-snow:nth-child(137){opacity:0.44;transform:translate(44.08vw,-10px) scale(0.55);animation:fall-137 16s -21s linear infinite}@keyframes fall-137{50.00%{transform:translate(42.65vw,50.00vh) scale(0.55)}to{transform:translate(43.36vw, 105vh) scale(0.55)}}.embedim-snow:nth-child(138){opacity:0.51;transform:translate(1.05vw,-10px) scale(0.53);animation:fall-138 17s -14s linear infinite}@keyframes fall-138{70.00%{transform:translate(3.93vw,70.00vh) scale(0.53)}to{transform:translate(2.49vw, 105vh) scale(0.53)}}.embedim-snow:nth-child(139){opacity:0.50;transform:translate(41.52vw,-10px) scale(0.31);animation:fall-139 20s -26s linear infinite}@keyframes fall-139{30.00%{transform:translate(44.99vw,30.00vh) scale(0.31)}to{transform:translate(43.26vw, 105vh) scale(0.31)}}.embedim-snow:nth-child(140){opacity:0.72;transform:translate(13.02vw,-10px) scale(0.85);animation:fall-140 15s -18s linear infinite}@keyframes fall-140{30.00%{transform:translate(7.41vw,30.00vh) scale(0.85)}to{transform:translate(10.21vw, 105vh) scale(0.85)}}.embedim-snow:nth-child(141){opacity:0.74;transform:translate(89.55vw,-10px) scale(1.00);animation:fall-141 23s -28s linear infinite}@keyframes fall-141{80.00%{transform:translate(92.73vw,80.00vh) scale(1.00)}to{transform:translate(91.14vw, 105vh) scale(1.00)}}.embedim-snow:nth-child(142){opacity:0.03;transform:translate(34.82vw,-10px) scale(0.04);animation:fall-142 18s -3s linear infinite}@keyframes fall-142{60.00%{transform:translate(26.11vw,60.00vh) scale(0.04)}to{transform:translate(30.47vw, 105vh) scale(0.04)}}.embedim-snow:nth-child(143){opacity:0.61;transform:translate(3.64vw,-10px) scale(0.04);animation:fall-143 15s -30s linear infinite}@keyframes fall-143{80.00%{transform:translate(9.19vw,80.00vh) scale(0.04)}to{transform:translate(6.41vw, 105vh) scale(0.04)}}.embedim-snow:nth-child(144){opacity:0.08;transform:translate(45.28vw,-10px) scale(0.06);animation:fall-144 14s -6s linear infinite}@keyframes fall-144{40.00%{transform:translate(50.40vw,40.00vh) scale(0.06)}to{transform:translate(47.84vw, 105vh) scale(0.06)}}.embedim-snow:nth-child(145){opacity:0.57;transform:translate(68.87vw,-10px) scale(0.37);animation:fall-145 11s -12s linear infinite}@keyframes fall-145{70.00%{transform:translate(71.75vw,70.00vh) scale(0.37)}to{transform:translate(70.31vw, 105vh) scale(0.37)}}.embedim-snow:nth-child(146){opacity:0.91;transform:translate(95.52vw,-10px) scale(0.69);animation:fall-146 17s -24s linear infinite}@keyframes fall-146{80.00%{transform:translate(93.69vw,80.00vh) scale(0.69)}to{transform:translate(94.60vw, 105vh) scale(0.69)}}.embedim-snow:nth-child(147){opacity:0.73;transform:translate(82.07vw,-10px) scale(0.29);animation:fall-147 21s -6s linear infinite}@keyframes fall-147{40.00%{transform:translate(86.37vw,40.00vh) scale(0.29)}to{transform:translate(84.22vw, 105vh) scale(0.29)}}.embedim-snow:nth-child(148){opacity:0.29;transform:translate(5.59vw,-10px) scale(0.23);animation:fall-148 26s -6s linear infinite}@keyframes fall-148{60.00%{transform:translate(12.82vw,60.00vh) scale(0.23)}to{transform:translate(9.20vw, 105vh) scale(0.23)}}.embedim-snow:nth-child(149){opacity:0.43;transform:translate(21.53vw,-10px) scale(0.52);animation:fall-149 21s -11s linear infinite}@keyframes fall-149{50.00%{transform:translate(15.96vw,50.00vh) scale(0.52)}to{transform:translate(18.75vw, 105vh) scale(0.52)}}.embedim-snow:nth-child(150){opacity:0.63;transform:translate(20.76vw,-10px) scale(0.63);animation:fall-150 15s -30s linear infinite}@keyframes fall-150{70.00%{transform:translate(14.49vw,70.00vh) scale(0.63)}to{transform:translate(17.62vw, 105vh) scale(0.63)}}.embedim-snow:nth-child(151){opacity:0.06;transform:translate(17.64vw,-10px) scale(0.22);animation:fall-151 18s -22s linear infinite}@keyframes fall-151{80.00%{transform:translate(22.17vw,80.00vh) scale(0.22)}to{transform:translate(19.90vw, 105vh) scale(0.22)}}.embedim-snow:nth-child(152){opacity:0.99;transform:translate(47.33vw,-10px) scale(0.09);animation:fall-152 11s -5s linear infinite}@keyframes fall-152{30.00%{transform:translate(47.51vw,30.00vh) scale(0.09)}to{transform:translate(47.42vw, 105vh) scale(0.09)}}.embedim-snow:nth-child(153){opacity:0.81;transform:translate(42.99vw,-10px) scale(0.00);animation:fall-153 20s -8s linear infinite}@keyframes fall-153{60.00%{transform:translate(46.26vw,60.00vh) scale(0.00)}to{transform:translate(44.63vw, 105vh) scale(0.00)}}.embedim-snow:nth-child(154){opacity:0.96;transform:translate(89.52vw,-10px) scale(0.61);animation:fall-154 10s -3s linear infinite}@keyframes fall-154{30.00%{transform:translate(98.64vw,30.00vh) scale(0.61)}to{transform:translate(94.08vw, 105vh) scale(0.61)}}.embedim-snow:nth-child(155){opacity:0.87;transform:translate(93.26vw,-10px) scale(0.60);animation:fall-155 15s -15s linear infinite}@keyframes fall-155{60.00%{transform:translate(95.95vw,60.00vh) scale(0.60)}to{transform:translate(94.60vw, 105vh) scale(0.60)}}.embedim-snow:nth-child(156){opacity:0.49;transform:translate(81.97vw,-10px) scale(0.52);animation:fall-156 30s -23s linear infinite}@keyframes fall-156{80.00%{transform:translate(78.57vw,80.00vh) scale(0.52)}to{transform:translate(80.27vw, 105vh) scale(0.52)}}.embedim-snow:nth-child(157){opacity:0.69;transform:translate(19.75vw,-10px) scale(0.05);animation:fall-157 26s -0s linear infinite}@keyframes fall-157{30.00%{transform:translate(26.37vw,30.00vh) scale(0.05)}to{transform:translate(23.06vw, 105vh) scale(0.05)}}.embedim-snow:nth-child(158){opacity:0.85;transform:translate(71.31vw,-10px) scale(0.60);animation:fall-158 27s -4s linear infinite}@keyframes fall-158{30.00%{transform:translate(73.97vw,30.00vh) scale(0.60)}to{transform:translate(72.64vw, 105vh) scale(0.60)}}.embedim-snow:nth-child(159){opacity:0.81;transform:translate(76.10vw,-10px) scale(0.17);animation:fall-159 20s -23s linear infinite}@keyframes fall-159{80.00%{transform:translate(83.80vw,80.00vh) scale(0.17)}to{transform:translate(79.95vw, 105vh) scale(0.17)}}.embedim-snow:nth-child(160){opacity:0.18;transform:translate(18.46vw,-10px) scale(0.55);animation:fall-160 10s -28s linear infinite}@keyframes fall-160{50.00%{transform:translate(19.56vw,50.00vh) scale(0.55)}to{transform:translate(19.01vw, 105vh) scale(0.55)}}.embedim-snow:nth-child(161){opacity:0.00;transform:translate(51.27vw,-10px) scale(0.01);animation:fall-161 14s -24s linear infinite}@keyframes fall-161{70.00%{transform:translate(49.35vw,70.00vh) scale(0.01)}to{transform:translate(50.31vw, 105vh) scale(0.01)}}.embedim-snow:nth-child(162){opacity:0.06;transform:translate(17.37vw,-10px) scale(0.03);animation:fall-162 16s -3s linear infinite}@keyframes fall-162{80.00%{transform:translate(9.31vw,80.00vh) scale(0.03)}to{transform:translate(13.34vw, 105vh) scale(0.03)}}.embedim-snow:nth-child(163){opacity:0.60;transform:translate(5.99vw,-10px) scale(0.37);animation:fall-163 23s -25s linear infinite}@keyframes fall-163{40.00%{transform:translate(12.55vw,40.00vh) scale(0.37)}to{transform:translate(9.27vw, 105vh) scale(0.37)}}.embedim-snow:nth-child(164){opacity:0.03;transform:translate(36.87vw,-10px) scale(0.46);animation:fall-164 28s -3s linear infinite}@keyframes fall-164{80.00%{transform:translate(26.97vw,80.00vh) scale(0.46)}to{transform:translate(31.92vw, 105vh) scale(0.46)}}.embedim-snow:nth-child(165){opacity:0.14;transform:translate(14.89vw,-10px) scale(0.55);animation:fall-165 10s -22s linear infinite}@keyframes fall-165{70.00%{transform:translate(5.65vw,70.00vh) scale(0.55)}to{transform:translate(10.27vw, 105vh) scale(0.55)}}.embedim-snow:nth-child(166){opacity:0.04;transform:translate(43.56vw,-10px) scale(0.84);animation:fall-166 29s -22s linear infinite}@keyframes fall-166{70.00%{transform:translate(50.24vw,70.00vh) scale(0.84)}to{transform:translate(46.90vw, 105vh) scale(0.84)}}.embedim-snow:nth-child(167){opacity:0.07;transform:translate(59.83vw,-10px) scale(0.54);animation:fall-167 17s -9s linear infinite}@keyframes fall-167{30.00%{transform:translate(63.80vw,30.00vh) scale(0.54)}to{transform:translate(61.81vw, 105vh) scale(0.54)}}.embedim-snow:nth-child(168){opacity:0.57;transform:translate(91.37vw,-10px) scale(0.01);animation:fall-168 20s -24s linear infinite}@keyframes fall-168{40.00%{transform:translate(98.61vw,40.00vh) scale(0.01)}to{transform:translate(94.99vw, 105vh) scale(0.01)}}.embedim-snow:nth-child(169){opacity:0.79;transform:translate(98.31vw,-10px) scale(0.97);animation:fall-169 17s -17s linear infinite}@keyframes fall-169{60.00%{transform:translate(95.58vw,60.00vh) scale(0.97)}to{transform:translate(96.95vw, 105vh) scale(0.97)}}.embedim-snow:nth-child(170){opacity:0.92;transform:translate(91.41vw,-10px) scale(0.93);animation:fall-170 15s -23s linear infinite}@keyframes fall-170{50.00%{transform:translate(86.79vw,50.00vh) scale(0.93)}to{transform:translate(89.10vw, 105vh) scale(0.93)}}.embedim-snow:nth-child(171){opacity:0.17;transform:translate(11.13vw,-10px) scale(0.05);animation:fall-171 10s -22s linear infinite}@keyframes fall-171{70.00%{transform:translate(4.83vw,70.00vh) scale(0.05)}to{transform:translate(7.98vw, 105vh) scale(0.05)}}.embedim-snow:nth-child(172){opacity:0.36;transform:translate(21.51vw,-10px) scale(0.41);animation:fall-172 28s -13s linear infinite}@keyframes fall-172{30.00%{transform:translate(12.67vw,30.00vh) scale(0.41)}to{transform:translate(17.09vw, 105vh) scale(0.41)}}.embedim-snow:nth-child(173){opacity:0.10;transform:translate(76.45vw,-10px) scale(0.42);animation:fall-173 20s -22s linear infinite}@keyframes fall-173{60.00%{transform:translate(85.30vw,60.00vh) scale(0.42)}to{transform:translate(80.87vw, 105vh) scale(0.42)}}.embedim-snow:nth-child(174){opacity:0.79;transform:translate(67.44vw,-10px) scale(1.00);animation:fall-174 24s -26s linear infinite}@keyframes fall-174{60.00%{transform:translate(59.64vw,60.00vh) scale(1.00)}to{transform:translate(63.54vw, 105vh) scale(1.00)}}.embedim-snow:nth-child(175){opacity:0.98;transform:translate(18.69vw,-10px) scale(0.86);animation:fall-175 20s -21s linear infinite}@keyframes fall-175{40.00%{transform:translate(12.69vw,40.00vh) scale(0.86)}to{transform:translate(15.69vw, 105vh) scale(0.86)}}.embedim-snow:nth-child(176){opacity:0.12;transform:translate(19.78vw,-10px) scale(0.15);animation:fall-176 13s -5s linear infinite}@keyframes fall-176{40.00%{transform:translate(18.27vw,40.00vh) scale(0.15)}to{transform:translate(19.03vw, 105vh) scale(0.15)}}.embedim-snow:nth-child(177){opacity:0.14;transform:translate(29.75vw,-10px) scale(0.03);animation:fall-177 16s -3s linear infinite}@keyframes fall-177{40.00%{transform:translate(37.47vw,40.00vh) scale(0.03)}to{transform:translate(33.61vw, 105vh) scale(0.03)}}.embedim-snow:nth-child(178){opacity:0.77;transform:translate(73.01vw,-10px) scale(0.62);animation:fall-178 22s -2s linear infinite}@keyframes fall-178{30.00%{transform:translate(67.40vw,30.00vh) scale(0.62)}to{transform:translate(70.21vw, 105vh) scale(0.62)}}.embedim-snow:nth-child(179){opacity:0.02;transform:translate(41.58vw,-10px) scale(0.15);animation:fall-179 18s -30s linear infinite}@keyframes fall-179{80.00%{transform:translate(42.86vw,80.00vh) scale(0.15)}to{transform:translate(42.22vw, 105vh) scale(0.15)}}.embedim-snow:nth-child(180){opacity:0.09;transform:translate(55.37vw,-10px) scale(0.03);animation:fall-180 21s -2s linear infinite}@keyframes fall-180{50.00%{transform:translate(58.54vw,50.00vh) scale(0.03)}to{transform:translate(56.95vw, 105vh) scale(0.03)}}.embedim-snow:nth-child(181){opacity:0.57;transform:translate(35.05vw,-10px) scale(0.39);animation:fall-181 24s -21s linear infinite}@keyframes fall-181{60.00%{transform:translate(38.25vw,60.00vh) scale(0.39)}to{transform:translate(36.65vw, 105vh) scale(0.39)}}.embedim-snow:nth-child(182){opacity:0.85;transform:translate(20.00vw,-10px) scale(0.12);animation:fall-182 23s -24s linear infinite}@keyframes fall-182{50.00%{transform:translate(21.39vw,50.00vh) scale(0.12)}to{transform:translate(20.70vw, 105vh) scale(0.12)}}.embedim-snow:nth-child(183){opacity:0.78;transform:translate(97.23vw,-10px) scale(0.82);animation:fall-183 17s -24s linear infinite}@keyframes fall-183{70.00%{transform:translate(92.08vw,70.00vh) scale(0.82)}to{transform:translate(94.66vw, 105vh) scale(0.82)}}.embedim-snow:nth-child(184){opacity:0.16;transform:translate(47.19vw,-10px) scale(0.01);animation:fall-184 30s -0s linear infinite}@keyframes fall-184{80.00%{transform:translate(51.16vw,80.00vh) scale(0.01)}to{transform:translate(49.17vw, 105vh) scale(0.01)}}.embedim-snow:nth-child(185){opacity:0.29;transform:translate(32.21vw,-10px) scale(0.85);animation:fall-185 30s -5s linear infinite}@keyframes fall-185{60.00%{transform:translate(22.53vw,60.00vh) scale(0.85)}to{transform:translate(27.37vw, 105vh) scale(0.85)}}.embedim-snow:nth-child(186){opacity:0.37;transform:translate(48.93vw,-10px) scale(0.34);animation:fall-186 25s -30s linear infinite}@keyframes fall-186{60.00%{transform:translate(51.35vw,60.00vh) scale(0.34)}to{transform:translate(50.14vw, 105vh) scale(0.34)}}.embedim-snow:nth-child(187){opacity:0.44;transform:translate(10.91vw,-10px) scale(0.51);animation:fall-187 29s -12s linear infinite}@keyframes fall-187{30.00%{transform:translate(16.61vw,30.00vh) scale(0.51)}to{transform:translate(13.76vw, 105vh) scale(0.51)}}.embedim-snow:nth-child(188){opacity:0.36;transform:translate(67.62vw,-10px) scale(0.51);animation:fall-188 30s -4s linear infinite}@keyframes fall-188{30.00%{transform:translate(63.85vw,30.00vh) scale(0.51)}to{transform:translate(65.74vw, 105vh) scale(0.51)}}.embedim-snow:nth-child(189){opacity:0.58;transform:translate(20.66vw,-10px) scale(0.09);animation:fall-189 14s -0s linear infinite}@keyframes fall-189{70.00%{transform:translate(28.90vw,70.00vh) scale(0.09)}to{transform:translate(24.78vw, 105vh) scale(0.09)}}.embedim-snow:nth-child(190){opacity:0.51;transform:translate(84.62vw,-10px) scale(0.55);animation:fall-190 16s -4s linear infinite}@keyframes fall-190{40.00%{transform:translate(85.18vw,40.00vh) scale(0.55)}to{transform:translate(84.90vw, 105vh) scale(0.55)}}.embedim-snow:nth-child(191){opacity:0.39;transform:translate(32.81vw,-10px) scale(0.68);animation:fall-191 12s -23s linear infinite}@keyframes fall-191{60.00%{transform:translate(32.60vw,60.00vh) scale(0.68)}to{transform:translate(32.71vw, 105vh) scale(0.68)}}.embedim-snow:nth-child(192){opacity:0.30;transform:translate(92.58vw,-10px) scale(0.07);animation:fall-192 18s -15s linear infinite}@keyframes fall-192{50.00%{transform:translate(92.13vw,50.00vh) scale(0.07)}to{transform:translate(92.35vw, 105vh) scale(0.07)}}.embedim-snow:nth-child(193){opacity:0.56;transform:translate(77.13vw,-10px) scale(0.28);animation:fall-193 15s -22s linear infinite}@keyframes fall-193{70.00%{transform:translate(83.25vw,70.00vh) scale(0.28)}to{transform:translate(80.19vw, 105vh) scale(0.28)}}.embedim-snow:nth-child(194){opacity:0.58;transform:translate(25.24vw,-10px) scale(0.90);animation:fall-194 20s -7s linear infinite}@keyframes fall-194{80.00%{transform:translate(34.22vw,80.00vh) scale(0.90)}to{transform:translate(29.73vw, 105vh) scale(0.90)}}.embedim-snow:nth-child(195){opacity:0.81;transform:translate(98.51vw,-10px) scale(0.92);animation:fall-195 11s -14s linear infinite}@keyframes fall-195{60.00%{transform:translate(93.37vw,60.00vh) scale(0.92)}to{transform:translate(95.94vw, 105vh) scale(0.92)}}.embedim-snow:nth-child(196){opacity:0.89;transform:translate(44.53vw,-10px) scale(0.25);animation:fall-196 18s -24s linear infinite}@keyframes fall-196{40.00%{transform:translate(40.20vw,40.00vh) scale(0.25)}to{transform:translate(42.36vw, 105vh) scale(0.25)}}.embedim-snow:nth-child(197){opacity:0.20;transform:translate(32.76vw,-10px) scale(0.04);animation:fall-197 24s -3s linear infinite}@keyframes fall-197{40.00%{transform:translate(38.67vw,40.00vh) scale(0.04)}to{transform:translate(35.71vw, 105vh) scale(0.04)}}.embedim-snow:nth-child(198){opacity:0.64;transform:translate(0.03vw,-10px) scale(0.80);animation:fall-198 29s -19s linear infinite}@keyframes fall-198{80.00%{transform:translate(4.22vw,80.00vh) scale(0.80)}to{transform:translate(2.13vw, 105vh) scale(0.80)}}.embedim-snow:nth-child(199){opacity:0.20;transform:translate(78.85vw,-10px) scale(0.69);animation:fall-199 10s -5s linear infinite}@keyframes fall-199{70.00%{transform:translate(85.20vw,70.00vh) scale(0.69)}to{transform:translate(82.02vw, 105vh) scale(0.69)}}