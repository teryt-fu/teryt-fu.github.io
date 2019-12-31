---
title: 基于hexo搭建静态网站
date: 2019-12-30 20:16:53
tags:
- web
- hexo
- gitpage
categories:
- web
---
# 简要介绍
  基于Hexo博客框架，使用markdown语法写文章，通过hexo编译成静态网页，搭配GitHub Pages的{yourname}.github.io的仓库，来搭建自己的网站。
# 准备步骤
  1. 创建github帐号。
  2. 新建一个仓库(Repository)，名称必须为{yourname}.github.io，{yourname}必须为你github帐号的用户名。
  3. 添加本地公钥`~/.ssh/id_rsa.pub`到github的settings中的ssh中，在本地适当位置clone刚新建的库。
  4. 安装Node.js
     安装方式多样，最简单使用apt-get安装
        `sudo apt-get install nodejs`
        `sudo apt-get install npm`
     检查npm是否能正常使用。npm类似pip，是node.js的安装工具
  5. 安装Hexo
     `npm install -g hexo-cli`

# 开始搭建
  1. cd进clone下来以你用户名.github.io命名的文件夹
  2. 新建项目：
     `hexo init {name}`
      注：name就是项目名
  3. 编译+本地运行：
     先cd进生成的文件夹，编译：`hexo generate`，运行：`hexo serve`，会将服务部署在本地的4000端口上，浏览器访问localhost:4000就能查看刚新建的项目。

# 部署项目
  1. 修改配置文件：
     编辑根目录下的_config.yml文件，找到Deployment，将你的Repository地址贴到repo:后，修改branch: master，效果如下：
        ```
        # Deployment
        ## Docs: https://hexo.io/docs/deployment.html
        deploy:
        type: git
        repo: git@github.com:username/username.github.io.git
        branch: master
        ```
  2. 安装git部署插件hexo-deployer-git：
     `npm install hexo-deployer-git --save`
         注：在项目根目录下执行此命令，如未安装则部署时会报错：
            Deployer not found: git
  3. 部署命令：
     `hexo deploy`
     出现如下结果说明部署成功：
        ```
        INFO  Deploying: git
        INFO  Clearing .deploy_git folder...
        INFO  Copying files from public folder...
        INFO  Copying files from extend dirs...
        On branch master
        nothing to commit, working directory clean
        Counting objects: 46, done.
        Delta compression using up to 8 threads.
        Compressing objects: 100% (36/36), done.
        Writing objects: 100% (46/46), 507.66 KiB | 0 bytes/s, done.
        Total 46 (delta 3), reused 0 (delta 0)
        remote: Resolving deltas: 100% (3/3), done.
        To git@github.com:NightTeam/nightteam.github.io.git
        * [new branch]      HEAD -> master
        Branch master set up to track remote branch master from git@github.com:NightTeam/nightteam.github.io.git.
        INFO  Deploy done: git
        ```
     此时访问username.github.io，就可以查看刚部署的博客。

# 优化博客
  1. 配置博客信息
     修改根目录下的_config.yml文件，找到Site，修改相关内容：
     ```
        # Site
        title: 你的博客名字
        subtitle: 博客的子标题
        description: 简要的描述信息
        keywords: 关键字
        author: 作者名
        language: zh-CN
        timezone: Asia/Shanghai
     ```
         注：可以运行`hexo serve`开启一个本地服务，实时访问localhost:4000查看修改后的样式
  2. 修改主题
     推荐Next主题
     - 安装：进入项目根目录，运行：`git clone https://github.com/theme-next/hexo-theme-next themes/next`，会在项目的themes/next文件夹下安装Next主题源码。
     - 修改配置文件：根目录下的_config.yml文件中找到theme字段，修改为next：
     ```
        theme: next
     ```
  3. 主题配置
         注：对主题的配置修改的文件在themes/next文件夹下的_config.yml
     - 样式
       修改配置文件中的scheme字段，选择一个样式即可。可选项有：
       ```
        scheme: Muse
        #scheme: Mist
        #scheme: Pisces
        #scheme: Gemini
       ```
     - favicon
       标签栏的小图标，可以自己定制后将图标放在themes/next/source/images目录下面，然后修改配置文件，找到favicon，配置路径：
       ```
        favicon:
            small: /images/favicon-16x16.png
            medium: /images/favicon-32x32.png
            apple_touch_icon: /images/apple-touch-icon.png
            safari_pinned_tab: /images/safari-pinned-tab.svg
       ```
     - avatar
       作者头像，放置路径为themes/next/source/images/avatar.png，修改配置文件，找到Sidebar Avatar，修改正确路径：
       ```
        # Sidebar Avatar
        avatar:
            # In theme directory (source/images): /images/avatar.gif
            # In site directory (source/uploads): /uploads/avatar.gif
            # You can also use other linking images.
            url: /images/avatar.png
            # If true, the avatar would be dispalyed in circle.
            rounded: true
            # If true, the avatar would be rotated with the cursor.
            rotated: true
       ```
             注：rounded选项是是否显示圆形，rotated是是否带旋转效果，可自选true或false。
     - rss订阅
       需安装插件hexo-generator-feed，命令为：`npm install hexo-generator-feed --save`，在项目根目录下运行此命令，完成后无需其他配置，站点会自动生成RSS Feed文件。
     - code
       代码块的显示样式。修改配置文件，找到codeblock，更改选项：
       ```
        codeblock:
            # Code Highlight theme
            # Available values: normal | night | night eighties | night blue | night bright
            # See: https://github.com/chriskempson/tomorrow-theme
            highlight_theme: night bright
            # Add copy button on codeblock
            copy_button:
                enable: true
                # Show text copy result.
                show_result: true
                # Available values: default | flat | mac
                style: mac
       ```
             注：可根据喜好自由选择。
     - top
       返回页面顶端的按钮。修改配置文件，找到back2top，更改选项：
       ```
        back2top:
            enable: true
            # Back to top in sidebar.
            sidebar: false
            # Scroll percent label in b2t button.
            scrollpercent: true
       ```
             注：enable默认为true，即显示。sidebar为true则按钮会出现在侧栏下方。scrollpercent显示阅读百分比。
     - reading_process
       阅读进度。最上侧显示的进度条。在配置文件中找到reading_progress，修改选项：
       ```
       reading_progress:
        enable: true
        # Available values: top | bottom
        position: top
        color: "#222"
        height: 2px
       ```
     - bookmark
       书签，记录阅读记录，下次打开页面可快速定位上次阅读位置。在配置文件中找到bookmark，修改选项：
       ```
       bookmark:
        enable: false
        # Customize the color of the bookmark.
        color: "#222"
        # If auto, save the reading progress when closing the page or clicking the bookmark-icon.
        # If manual, only save it by clicking the bookmark-icon.
        save: auto
       ```
     - github_banner
       右上角显示github图标，在配置文件中找到github_banner，修改选项：
       ```
        # `Follow me on GitHub` banner in the top-right corner.
        github_banner:
            enable: true
            permalink: https://github.com/yourname/yourname.github.io
            title: yourname GitHub
       ```
     - gitalk
       评论功能。Next主题中有多种评论插件，如changyan | disqus | disqusjs | facebook_comments_plugin | gitalk | livere | valine | vkontakte。示例用gittalk，首先需要在github上注册一个OAuth Application，链接为：`https://github.com/settings/applications/new`，注册完毕之后拿到 Client ID、Client Secret。在配置文件中找到comments，修改配置：
       ```
       # Multiple Comment System Support
        comments:
        # Available values: tabs | buttons
        style: tabs
        # Choose a comment system to be displayed by default.
        # Available values: changyan | disqus | disqusjs | facebook_comments_plugin | gitalk | livere | valine | vkontakte
        active: gitalk

       ```
       然后找到Gitalk，修改配置：
       ```
        # Gitalk
        # Demo: https://gitalk.github.io
        # For more information: https://github.com/gitalk/gitalk
        gitalk:
            enable: true
            github_id: {yourname}
            repo: {yourname}.github.io # Repository name to store issues
            client_id: {your client id} # GitHub Application Client ID
            client_secret: {your client secret} # GitHub Application Client Secret
            admin_user: germey # GitHub repo owner and collaborators, only these guys can initialize gitHub issues
            distraction_free_mode: true # Facebook-like distraction free mode
            # Gitalk's display language depends on user's browser or system environment
            # If you want everyone visiting your site to see a uniform language, you can set a force language value
            # Available values: en | es-ES | fr | ru | zh-CN | zh-TW
            language: zh-CN
       ```
    - pangu
      中英文间自动加间距。在配置文件中找到pangu，值修改为true即可。
    - math
      公式展示效果。在配置文件中找到math，修改配置：
      ```
        math:
            enable: true

            # Default (true) will load mathjax / katex script on demand.
            # That is it only render those page which has `mathjax: true` in Front-matter.
            # If you set it to false, it will load mathjax / katex srcipt EVERY PAGE.
            per_page: true

            # hexo-renderer-pandoc (or hexo-renderer-kramed) required for full MathJax support.
            mathjax:
                enable: true
                # See: https://mhchem.github.io/MathJax-mhchem/
                mhchem: true
      ```
             注：mathjax使用需要安装插件hexo-renderer-kramed，也可以安装hexo-renderer-pandoc，命令为：
             ```
             npm un hexo-renderer-marked --save
             npm i hexo-renderer-kramed --save
             ```
     - pjax
       类似ajax，首先在配置文件中修改pjax的值为true，再切换到next主题下安装依赖库:
       ```
       $ cd themes/next
       $ git clone https://github.com/theme-next/theme-next-pjax source/lib/pjax
       ```

# 日常使用
  1. 新建文章
     - 在项目根目录运行`hexo new 文章名`，新建的文章在`source/_posts`文件夹下，文章开头格式为:
     ```
        ---
        title: 标题 # 自动创建，如 hello-world
        date: 日期 # 自动创建，如 2019-12-31 23:59:59
        tags: 
        - 标签1
        - 标签2
        - 标签3
        categories:
        - 分类1
        - 分类2
        ---
     ```
     开头下方撰写正文，markdown格式书写。用此开头格式则编译的时候会自动识别标题、时间、类别等信息。
  2. 标签页
     - 在项目根目录运行`hexo new page tags`，会自动生成`source/tags/index.md`文件，内容为:
     ```
        ---
        title: tags
        date: 2019-12-31 23:59:59
        ---
     ```
     可在后面自行添加type字段来指定页面的类型：
     ```
     type: tags
     comments: false
     ```
     然后在next主题的_config.yml文件中将页面链接添加到主菜单页面即可，找到menu，修改配置：
     ```
     menu:
        home: / || home
        #about: /about/ || user
        tags: /tags/ || tags
        #categories: /categories/ || th
        archives: /archives/ || archive
        #schedule: /schedule/ || calendar
        #sitemap: /sitemap.xml || sitemap
        #commonweal: /404/ || heartbeat
     ```
  3. 分类页
     - 在项目根目录运行`hexo new page categories`，会自动生成`source/categories/index.md`文件，在文件中添加type字段：
     ```
     type: categories
     comments: false
     ```
     再修改next主题的_config.yml文件，找到menu:
     ```
     menu:
        home: / || home
        #about: /about/ || user
        tags: /tags/ || tags
        categories: /categories/ || th
        archives: /archives/ || archive
        #schedule: /schedule/ || calendar
        #sitemap: /sitemap.xml || sitemap
        #commonweal: /404/ || heartbeat
     ```
  4. 搜索页
     - 搜索功能需要安装插件hexo-generator-searchdb，运行`npm install hexo-generator-searchdb --save`，然后修改项目根目录的_config.yml，找到search，修改配置：
     ```
        search:
            path: search.xml
            field: post
            format: html
            limit: 10000
     ```
     再修改next主题的_config.yml，找到local_search，修改配置：
     ```
     # Local search
     # Dependencies: https://github.com/wzpan/hexo-generator-search
     local_search:
        enable: true
        # If auto, trigger search by changing input.
        # If manual, trigger search by pressing enter key or search button.
        trigger: auto
        # Show top n results per article, show all results by setting to -1
        top_n_per_article: 5
        # Unescape html strings to the readable one.
        unescape: false
        # Preload the search data when the page loads.
        preload: false
     ```
  5. 404页
     - 直接在根目录的source文件夹新建一个404.md文件，内容示例：
     ```
     ---
     title: 404 Not Found
     date: 2019-12-31 23:59:59
     ---

     <center>
     对不起，您所访问的页面不存在或者已删除。
     您可以<a href="https://yourname.github.io>">点击此处</a>返回首页。
     </center>

     <blockquote class="blockquote-center">
         yourname
     </blockquote>
     ```

# 部署脚本
  在根目录新建deploy.sh脚本文件，内容为：
  ```
    hexo clean
    hexo generate
    hexo deploy
  ```
  在需要部署发布时只需要在根目录执行`sh deploy.sh`即可。
# 自定义域名
  在github的Repository里，即你的gitpage主页面，在Settings中拉到最下面，有个GitHub Pages配置项，![github pages](/images/gitpage.jpg)
  在Custom domain中输入你想自定义的域名地址，然后添加CNAME解析。
     - 注：在目前情况，每次部署会处置为原域名，所以需在根目录下的source文件夹下新建CNAME文件，内容为自定义的域名。