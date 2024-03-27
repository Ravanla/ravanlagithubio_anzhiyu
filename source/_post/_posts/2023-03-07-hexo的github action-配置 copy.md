---
title: 使用 GitHub Actions 实现 Hexo 博客自动部署
tags:
  - hexo
  - github action
categories:
  - null
mathjax: true
abbrlink: ac8cced9
date: 2023-03-04 11:52:48
updated: 2023-03-07 22:00:00
description:
---

## Github Action简介

[Github Action](https://github.com/features/actions)是 GitHub于2018年10月推出的一个CI\\CD服务。  

CI/CD解释

CI\\CD 其实说的是三件事情：「持续集成（`Continuous Integration`）」、「持续交付（`Continuous Delivery`）」、「持续部署（`Continuous Deployment`）」。  
因为「持续交付」和「持续部署」的英文缩写是一样的，所以这三件事情缩写成了 `CI\CD` 。

  
每次部署`Hexo`都需要运行指令三件套，随着文章越来越多，编译的时间也随之越来越长，通过`Github Action`，我们只需要在每次完成博客的编写或修改以后，将改动直接`push`到远程仓库，之后的编译部署的工作统统交给`CI`来完成即可，如果是看过[Coding部署教程](https://akilar.top/posts/54c08a4b/)的小伙伴，应该对这种持续部署的操作有所感触。

## 教程常量声明

感谢[@YML](https://menglei.xyz)的反馈。以下将使用特定的常量名来指代一些名词。此处建议读者直接使用教程内容的常量名。在最后再逐一搜索替换。这样可以避免对各种常量名的混淆。

| 常量名 | 常量释义 |
| --- | --- |
| **\[Blogroot\]** | 本地存放博客源码的文件夹路径 |
| **\[SourceRepo\]** | 存放博客源码的私有仓库名 |
| **\[SiteBlogRepo\]** | 存放编译好的博客页面的公有仓库名  Site指站点，教程中会替换成  Github、Gitee、Coding |
| **\[SiteUsername\]** | 用户名  Site指站点，教程中会替换成  Github、Gitee、Coding |
| **\[SiteToken\]** | 申请到的令牌码  Site指站点，教程中会替换成  Github、Gitee、Coding |
| **\[GithubEmail\]** | 与github绑定的主邮箱，建议使用Gmail |
| **\[TokenUser\]** | Coding配置特有的令牌用户名 |

```txt
# 在记事本中逐个记录，方便替换，以下为我的示例
[Blogroot]：D:\developer\ravanlaBlog\Ravanla.github.io

私有库
[SourceRepo]：Ravanla/RavanlaGithubio


公有库
[SiteBlogRepo]
  [GithubBlogRepo]: Ravanla.github.io
  [GiteeBlogRepo]: Ravanla.github.io
  [CodingBlogRepo]: wowCODING/hexoblog

[SiteUsername]
  [GithubUsername]：Ravanla
  [GiteeUsername]：Ravanla
  [CodingUsername]：wowCODING

[SiteToken]
  [GithubToken]：ghp_jZ64kHP2.....
  [GiteeToken]：87a11d9d7....
  [CodingToken]：5141a406bebd9....

[GithubEmail]：1285644869@qq.com

Coding的token name，不需要部署到coding的可以不用填写
[TokenUser]：Ravanla
```

## Github Action使用教程

### 获取Token

为了确保交由`Github Action`来持续部署时，`Github Action`具备足够的权限来进行`hexo deploy`操作,需要先获取`Token`，博主分别在`Github`、`Gitee`、`Coding`处部署了静态页面，所以也就需要获取这三处的`Token`。  

-   ⚛️Github
-   🕉️Gitee
-   ✡️Coding

访问[Github->头像（右上角）->Settings->Developer Settings->Personal access tokens](https://github.com/settings/tokens)\->generate new token,创建的`Token`名称随意，但必须勾选repo项和workflows项。  
![](https://npm.elemecdn.com/akilar-candyassets/image/PISlRDgrsBXzcLK.png)  
![](https://npm.elemecdn.com/akilar-candyassets/image/ca384d58.png)  
![](https://npm.elemecdn.com/akilar-candyassets/image/20200923085908748.png)

`token`只会显示这一次，之后将无法查看，所以务必保证你已经记录下了`Token`。之后如果忘记了就只能重新生成重新配置了。

访问[Gitee->头像（右上角）->设置->私人令牌](https://gitee.com/profile/personal_access_tokens)\->生成新令牌  
![](https://npm.elemecdn.com/akilar-candyassets/image/voHjeENlQOpFA6n.png)  
![](https://npm.elemecdn.com/akilar-candyassets/image/X6x8Tz9bgO7HNRP.png)

`Token`只会显示这一次，之后将无法查看，所以务必保证你已经记录下了`Token`。之后如果忘记了就只能重新生成重新配置了。

关于coding相关内容不再具备参考价值。Coding被腾讯云收购以后，现已停止提供静态页面部署功能，腾讯云将静态页面部署业务转移至[webify](https://webify.cloudbase.net/)，部署页面更加简单明了，速度也相当不错，当然，毕竟是腾讯家的产品，是按量计费的。

访问[Coding](https://coding.net/)\->头像（右上角）->个人账户设置->访问令牌->新建令牌。  
![](https://npm.elemecdn.com/akilar-candyassets/image/PMgKRDZAUyfktzp.png)

Coding的配置还需要用到**令牌用户名\[TokenUser\]**，务必牢记，在接下来的`deploy`配置项中会写到。

![](https://npm.elemecdn.com/akilar-candyassets/image/aF4dLUomY6NkunZ.png)

![](https://npm.elemecdn.com/akilar-candyassets/image/HW96S8wOIP7YJxt.png)

**Token只会显示这一次**，之后将无法查看，所以务必保证你已经记录下了Token。之后如果忘记了就只能重新生成重新配置了。

### 创建存放源码的私有仓库

注意是`私有仓库`

我们需要创建一个用来存放`Hexo`博客源码的私有仓库`[SourceRepo]`，这点在[Win10](https://akilar.top/posts/6ef63e2d/)的`Hexo`博客搭建教程中有提到。为了保持教程的连贯，此处再写一遍。  

创建完成后，需要把博客的源码`push`到这里。首先获取远程仓库地址，此处虽然`SSH`和`HTTPS`均可。`SSH`在绑定过`ssh key`的设备上无需再输入密码，`HTTPS`则需要输入密码，但是`SSH`偶尔会遇到端口占用的情况。请自主选择。  

这里之所以是私有仓库，是因为在接下来的配置中会用到`Token`，如果`Token`被盗用，别人可以肆意操作你的github仓库内容，为了避免这一风险，才选择的博客源码闭源。

### 配置Github Action

-   ⚓常规配置，适合初学者
-   💫拓展：Gitee自动部署脚本配置&其余需要指令支持的拓展插件的配置

1.  在`[Blogroot]`新建`.github`文件夹,注意开头是有个`.`的。然后在`.github`内新建`workflows`文件夹，再在`workflows`文件夹内新建`autodeploy.yml`,在`[Blogroot]/.github/workflows/autodeploy.yml`里面输入
    
```yml
# 当有改动推送到master分支时，启动Action
name: 自动部署

on:
  push:
    branches:
      - master #2020年10月后github新建仓库默认分支改为main，注意更改

  release:
    types:
      - published

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
    - name: 检查分支
      uses: actions/checkout@v2
      with:
        ref: master #2020年10月后github新建仓库默认分支改为main，注意更改

    - name: 安装 Node
      uses: actions/setup-node@v1
      with:
        node-version: "12.x" #action使用的node版本，建议大版本和本地保持一致。可以在本地用node -v查询版本号。

    - name: 安装 Hexo
      run: |
        export TZ='Asia/Shanghai'
        npm install hexo-cli -g

    - name: 缓存 Hexo
      uses: actions/cache@v1
      id: cache
      with:
        path: node_modules
        key: ${{runner.OS}}-${{hashFiles('**/package-lock.json')}}

    - name: 安装依赖
      if: steps.cache.outputs.cache-hit != 'true'
      run: |
        npm install --save

    - name: 生成静态文件
      run: |
        hexo clean
        hexo generate

    - name: 部署 #此处master:master 指从本地的master分支提交到远程仓库的master分支，若远程仓库没有对应分支则新建一个。如有其他需要，可以根据自己的需求更改。
      run: |
        cd ./public
        git init
        git config --global user.name '${{ secrets.GITHUBUSERNAME }}'
        git config --global user.email '${{ secrets.GITHUBEMAIL }}'
        git add .
        git commit -m "${{ github.event.head_commit.message }} $(date +"%Z %Y-%m-%d %A %H:%M:%S") Updated By Github Actions"
        git push --force --quiet "https://${{ secrets.GITHUBUSERNAME }}:${{ secrets.GITHUBTOKEN }}@github.com/${{ secrets.GITHUBUSERNAME }}/${{ secrets.GITHUBUSERNAME }}.github.io.git" master:master
        #git push --force --quiet "https://${{ secrets.TOKENUSER }}:${{ secrets.CODINGTOKEN }}@e.coding.net/${{ secrets.CODINGUSERNAME }}/${{  secrets.CODINGBLOGREPO }}.git" master:master #coding部署写法，需要的自行取消注释
        #git push --force --quiet "https://${{ secrets.GITEEUSERNAME }}:${{ secrets.GITEETOKEN }}@gitee.com/${{ secrets.GITEEUSERNAME }}/${{ secrets.GITEEUSERNAME }}.git" master:master #gitee部署写法，需要的自行取消注释
```
    
注意最后一行的`master:master`指从本地的master分支提交到远程仓库的master分支,需要根据你自己的实际情况进行调整。本地分支可以在git bash中看到。线上分支可以在提交仓库中查看。因为“政治正确”的原因，github在2020年10月将默认分支改为main。而git软件在本地默认创建的分支依然是master，所以若你线上仓库默认分支是main，这里应该写成master:main，表示从本地的master推送到远程的main。
    
2.  之后需要自己到仓库的Settings->Secrets->actions 下添加环境变量，变量名参考脚本中出现的，依次添加。  
    ![](https://tuchuang.voooe.cn/images/2023/03/07/aec4c9bb08c68da6743df704e916e63.png)
    例如，需要部署在githubpage上，那么脚本中必要的变量为  
    `GITHUBUSERNAME`、`GITHUBEMAIL`、`GITHUBTOKEN`，因此添加这三条变量。变量具体内容释义可以查看本文开头。  

这里主要在于配置上文的安装依赖和生成静态文件，如果还安装了其他的需要在部署前输入相应指令的也可以按照这个思路来修改。  
同时，此处还涉及到`Gitee的自动部署`，**即使不开通Gitee pages pro，也可以完成自动更新**。  
详情可以访问[卓越科技-使用Github Actions 自动部署博客](https://blog.zykjofficial.top/posts/ea8e8e59/)  
插件配置教程则是参考以下文档

1.  [hexo-bilibili-bangumi](https://github.com/HCLonely/hexo-bilibili-bangumi)，使用该插件若是无效，请检查你的B站追番信息是否是公开的，在个人空间处设置。
2.  [Butterfly文档-gulp压缩](https://demo.jerryc.me/posts/4073eda/#Gulp%E5%A3%93%E7%B8%AE)
3.  [Gitee Pages Action](https://github.com/yanglbme/gitee-pages-action)

```yml
name: 自动部署
# 当有改动推送到master分支时，启动Action
on:
  push:
    branches:
      - master
      #2020年10月后github新建仓库默认分支改为main，注意更改
  release:
    types:
      - published

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
    - name: 检查分支
      uses: actions/checkout@v2
      with:
        ref: master

    - name: 安装 Node
      uses: actions/setup-node@v1
      with:
        node-version: "12.x" #action使用的node版本，建议大版本和本地保持一致。可以在本地用node -v查询版本号。

    - name: 安装 Hexo
      run: |
        export TZ='Asia/Shanghai'
        npm install hexo-cli -g

    - name: 缓存 Hexo
      uses: actions/cache@v1
      id: cache
      with:
        path: node_modules
        key: ${{runner.OS}}-${{hashFiles('**/package-lock.json')}}

    - name: 安装依赖
      if: steps.cache.outputs.cache-hit != 'true'
      run: |
        npm install gulp-cli -g #全局安装gulp
        npm install --save

    - name: 生成静态文件
      run: |
        hexo clean
        hexo bangumi -u #bilibili番剧更新
        hexo generate
        gulp

    - name: 部署 #此处master:master 指从本地的master分支提交到远程仓库的master分支，若远程仓库没有对应分支则新建一个。如有其他需要，可以根据自己的需求更改。
      run: |
        cd ./public
        git init
        git config --global user.name '${{ secrets.GITHUBUSERNAME }}'
        git config --global user.email '${{ secrets.GITHUBEMAIL }}'
        git add .
        git commit -m "${{ github.event.head_commit.message }} $(date +"%Z %Y-%m-%d %A %H:%M:%S") Updated By Github Actions"
        git push --force --quiet "https://${{ secrets.GITHUBUSERNAME }}:${{ secrets.GITHUBTOKEN }}@github.com/${{ secrets.GITHUBUSERNAME }}/${{ secrets.GITHUBUSERNAME }}.github.io.git" master:master
        #git push --force --quiet "https://${{ secrets.TOKENUSER }}:${{ secrets.CODINGTOKEN }}@e.coding.net/${{ secrets.CODINGUSERNAME }}/${{  secrets.CODINGBLOGREPO }}.git" master:master #coding部署写法，需要的自行取消注释
        #git push --force --quiet "https://${{ secrets.GITEEUSERNAME }}:${{ secrets.GITEETOKEN }}@gitee.com/${{ secrets.GITEEUSERNAME }}/${{ secrets.GITEEUSERNAME }}.git" master:master #gitee部署写法，需要的自行取消注释

    #同步到gitee这一步，如果上面写了推送到gitee仓库的git push指令了，那同步到 Gitee 这一块就不需要了。
    - name: 同步到 Gitee
      uses: wearerequired/git-mirror-action@master
      env:
          # 注意在github私有仓库的Settings->Secrets 配置 GITEE_RSA_PRIVATE_KEY
          SSH_PRIVATE_KEY: ${{ secrets.GITEE_RSA_PRIVATE_KEY }}
      with:
          # 注意替换为你的 GitHub 源仓库地址
          source-repo: "git@github.com:${{ secrets.GITHUBUSERNAME }}/${{ secrets.GITHUBUSERNAME }}.github.io.git"
          # 注意替换为你的 Gitee 目标仓库地址
          destination-repo: "git@gitee.com:${{ secrets.GITEEUSERNAME }}/${{ secrets.GITEEUSERNAME }}.git"
    # 这里就是模拟了一个点击更新网站按钮的动作。可以实现gitee page 自动更新。
    - name: 构建 Gitee Pages
      uses: yanglbme/gitee-pages-action@master
      with:
          # 注意替换为你的 Gitee 用户名
          gitee-username: ${{ secrets.GITEEUSERNAME }}
          # 注意在在github私有仓库的Settings->Secrets 配置 GITEE_PASSWORD
          gitee-password: ${{ secrets.GITEE_PASSWORD }}
          # 注意替换为你的 Gitee 仓库
          gitee-repo: ${{ secrets.GITEEUSERNAME }}/${{ secrets.GITEEUSERNAME }}
```


注意最后一行的`master:master`指从本地的master分支提交到远程仓库的master分支,需要根据你自己的实际情况进行调整。本地分支可以在git bash中看到。线上分支可以在提交仓库中查看。因为“政治正确”的原因，github在2020年10月将默认分支改为main。而git软件在本地默认创建的分支依然是master，所以若你线上仓库默认分支是main，这里应该写成master:main，表示从本地的master推送到远程的main。

还是原来那里，不过变量名和变量值gitee密码和个人秘钥

![](https://tuchuang.voooe.cn/images/2023/03/07/aec4c9bb08c68da6743df704e916e63.png)

GITEE PA5SWORD: gitee密码
GITEE RSA_PRIVATE KEY: 个人秘钥

这里的**GITEE\_RSA\_PRIVATE\_KEY**指你的个人密钥，在配置SSH-KEY时，我们用来与Github绑定的是公钥，而私钥存放在（以win10为例）`C:\Users\userneme\.ssh\id_rsa`文件内，内容格式类似于下方代码，使用时将包括`-----BEGIN RSA PRIVATE KEY-----`和`-----END RSA PRIVATE KEY-----`在内的全部内容都存放到变量值里。  


```bash
-----BEGIN RSA PRIVATE KEY-----
MIIEogIBAAKCAQEApM/T7rRtc9sNcd7NNZHMOZU7e7322leY5JauIxJEgQYpdrfO
vQB9PPpeMudAyCvAlaM?????????XO21G7RNejl4dLEflBh68TB93DlK/k/3KizMf
jewpXU3HHpFSSyiIA7Mi8ur39ybbG4oWEHI/Mnjq?????????e5oiVvYNux2
TazhAoGAL8h8XrB0t????????????????W2Ul4AomH1mu+rtIz2sQZdREVL4
dskwWvzoGOyNBPreLXWHBY6fg34dhNaZvNDZPGGd3bK6arMRdzrAynQio0CE0zwm
zJEo1tpUvqujmYMRnM1+jYHOPqU5sIvnEy5xovAzECPUSUs43Ag=
-----END RSA PRIVATE KEY-----
```

可能遇到的bug：

1.  Gitee用github action自动部署更新收到短信，提示异地登录需要验证码。  
    因为github action使用的是美国的服务器，所以，使用github action来远程更新gitee的站点部署时，会收到异地登陆的短信，提示需要验证码。这个在脚本作者的issues里有相应的解决方案:[登陆失败 #6](https://github.com/yanglbme/gitee-pages-action/issues/6)
    
    -   在微信上搜索Gitee微信公众号，在微信公众号内绑定自己的Gitee账号，这样虽然还是会有异地登录提示，但是发过来的消息不再需要填写验证码，而且提醒若不是你在操作，请及时修改密码。（某种意义上就是我在操作，所以我选择不改密码2333）
    -   使用VPN，通过美国IP登录一次Gitee。(一般第一步就能把问题解决了，用不到第二步。)
2.  Gitee部署失败  
    脚本的原理是用程序代替人工去点击Gitee Pages的更新按钮。所以需要你先手动做一次页面部署，确保有那个更新按钮在，脚本才有生效的前提。
    

### 重新设置远程仓库和分支

-   🍾曾经做过git管理博客源码的操作
-   🍼第一次使用git管理博客源码

1.  添加屏蔽项  
    因为能够使用指令进行安装的内容不包括在需要提交的源码内，所有我们需要将这些内容添加到屏蔽项，表示不上传到github上。这样可以显著减少需要提交的文件量和加快提交速度。  
    打开`[Blogroot]/.gitignore`,输入以下内容：
   

```txt
.DS_Store
Thumbs.db
db.json
*.log
node_modules/
public/
.deploy*/
.deploy_git*/
.idea
themes/butterfly/.git
```
    
    如果不是`butterfly`主题，记得替换最后一行内容为你自己当前使用的主题。
1.  提交源码到私有仓库`[SourceRepo]`  
    在博客根目录\[Blogroot\]下启动终端，使用git指令重设仓库地址。这样在新建仓库，我们仍旧可以保留珍贵的commit history，便于版本回滚。



```bash
git remote rm origin # 删除原有仓库链接
git remote add origin git@github.com:[GithubUsername]/[SourceRepo].git #[SourceRepo]为新的存放源码的github私有仓库
git checkout -b master # 切换到master分支，
#2020年10月后github新建仓库默认分支改为main，注意更改
# 如果不是，后面的所有设置的分支记得保持一致
git add .
git commit -m "github action update"
git push origin master
#2020年10月后github新建仓库默认分支改为main，注意更改
```
    
3.  可能遇到的bug  
    因为`butterfly`主题文件夹下的`.git`文件夹的存在，那么主题文件夹会被识别子项目。从而无法被上传到源码仓库。若是遇到添加屏蔽项，但是还是无法正常上传主题文件夹的情况。请先将本地源码中的`themes`文件夹移动到别的目录下。然后`commit`一次。接着将`themes`文件夹移动回来，再`commit`一次。
    
    要是还不行，那就删了`butterfly`主题文件夹下的`.git`文件夹，然后再重复上述的`commit`操作。
    

1.  删除或者先把`[Blogroot]/themes/butterfly/.git`移动到非博客文件夹目录下,原因是主题文件夹下的`.git`文件夹的存在会导致其被识别成子项目，从而无法被上传到源码仓库。
2.  在博客根目录`[Blogroot]`路径下运行指令


```bash
git init #初始化
git remote add origin git@github.com:[GithubUsername]/[SourceRepo].git #[SourceRepo]为存放源码的github私有仓库
git checkout -b master # 切换到master分支，
#2020年10月后github新建仓库默认分支改为main，注意更改
# 如果不是，后面的所有设置的分支记得保持一致
```
    
3.  添加屏蔽项  
    因为能够使用指令进行安装的内容不包括在需要提交的源码内，所有我们需要将这些内容添加到屏蔽项，表示不上传到github上。这样可以显著减少需要提交的文件量和加快提交速度。  
    打开`[Blogroot]/.gitignore`,输入以下内容：
    


```txt
.DS_Store
Thumbs.db
db.json
*.log
node_modules/
public/
.deploy*/
.deploy_git*/
.idea
themes/butterfly/.git
```
    
    如果不是`butterfly`主题，记得替换最后一行内容为你自己当前使用的主题。
1.  之后再运行git提交指令，将博客源码提交到github上。牢记下方的三行指令，以后都是通过这个指令进行提交。
    



```bash
git add .
git commit -m "github action update" #引号内的内容可以自行更改作为提交记录。
git push origin master
#2020年10月后github新建仓库默认分支改为main，注意更改
```
    
5.  此时你的主题文件夹若已经被正常上传，并且你也添加了主题文件夹下的.git文件夹的屏蔽项。那你可以考虑把第二步移走或删除的`.git`放回来，用作以后升级。（不禁怀疑真的有人会去用这个方式来升级吗）

### 查看部署情况

点击查看

此时，打开GIthub存放源码的私有仓库，找到action。  
![](https://tuchuang.voooe.cn/images/2023/03/07/798f4e77e5dfb966f0ef2c6cf88040f.png)
根据刚刚的Commit记录找到相应的任务  
![](
https://tuchuang.voooe.cn/images/2023/03/07/0b28a76a08cb845743cc6b738568f57.png)  
点击Deploy查看部署情况  

若是有红色交叉，说明部署失败，可以点开红酒交叉进行查看报错代码并复制代码在网上寻求解决办法
![](https://tuchuang.voooe.cn/images/2023/03/07/677bc8624c653a5ff634b95e104bca2.png)

若全部打钩，恭喜你，你现在可以享受自动部署的快感了。

### 可能遇到的bug

最新教程已更新在提交至源码仓库章节前作为提示

-   ⛔unknown block tag: "tagname"
-   ⛔spawn failed
-   变量名称问题
-   分支问题


#### unknown block tag: "tagname"

要是在github action部署时遇到`unknown block tag: "tagname"`这样的报错，说明你可能没有正确上传主题文件夹，也可能遇到安装依赖或生成页面失败的情况。

  
请按照以下思路逐一排查。

1.  是否将`node_modules`也上传到源码仓库`[SourceRepo]`了。源码仓库不需要有`node_modules`文件夹。
2.  是否有将`[Blogroot]/themes/`下的主题文件夹上传，例如检查\[SourceRepo\]内的`[Blogroot]/themes/Butterfly`是否为空文件夹。为了能够正常编译页面，源码仓库需要有`[Blogroot]/themes/Butterfly`文件夹及它所包含的主题文件内容。  
    为了避免这两点，需要添加git屏蔽项。通过给`.gitignore`添加屏蔽项解决。  
    打开`[Blogroot]/.gitignore`,输入以下内容：
    
    

```x86asm
.DS_Store
Thumbs.db
db.json
*.log
node_modules/
public/
.deploy*/
.deploy_git*/
.idea
themes/butterfly/.git
```

    
1.  若是遇到添加屏蔽项，但是还是无法正常上传主题文件夹的情况。
    -   请先将本地源码中的`themes/butterfly`文件夹下的`.git`文件夹删除。
    -   然后将`butterfly`文件夹移动到别的目录下。然后`commit`一次。
    -   接着将`butterfly`文件夹移动回来，再`commit`一次。

#### spawn failed

若是遇到`spawn failed`报错。在`github action`的配置中出现这一报错。一般是因为涉及到部署地址的配置项有误。

1.  首先排查你在`[Blogroot]\_config.yml`的`deploy`配置项是否按照上文[配置deploy项](https://akilar.top/posts/f752c86d//#配置deploy项)中的步骤正确组装配置链接。
2.  其次排查`[Blogroot]\.github\workflows\autodeploy.yml`中各个关于仓库链接的配置内容，注意按照注释指引检查空格、分支等。
3.  更多可能的因素和解决方案可以参考[@洪哥HEO](https://blog.zhheo.com/)写的方案:[Hexo错误：spawn failed的解决方法](https://blog.zhheo.com/p/128998ac.html)。


#### 变量名称问题

部分不愿意用教程给出的变量名的可能遇到未知bug，此处给出官方的命名规则：

以下规则适用于密码名称：

密钥名称只能包含字母数字字符（\[a-z\]、\[A-Z\]、\[0-9\]）或下划线 (\_)。 不允许空格。

密钥名称不得以 `GITHUB_` 前缀开头。

密钥名称不能以数字开头。

密钥名称不区分大小写。

密钥名称在所创建的级别上必须是唯一的。

#### 分支问题

本地分支和线上分支不一致导致总是提交不上。  
注意观察autodeploy.yml文件中，  


```bash
git push --force --quiet "https://${{ secrets.GITHUBUSERNAME }}:${{ secrets.GITHUBTOKEN }}@github.com/${{ secrets.GITHUBUSERNAME }}/${{ secrets.GITHUBUSERNAME }}.github.io.git" master:master
```
  
末尾的master:master指从本地的master分支提交到远程仓库的master分支。需要根据实际情况进行调整。本地的分支可在git bash中查看。线上的分支可在仓库查看。比如本地默认分支是master，线上默认分支是main，应该改成master:main。  
会遇到这类问题，一般是有同学直接全局替换master为main导致。

### 后记

这里可能有同学要问，github action有啥用？这不就是从`hexo cl && hexo g && hexo s`的三件套变成了`git add .`,`git commit -m "commit content"`,`git push`三件套吗？  
其实github action的最大作用就是进一步提高速度和便携性，首先，配置要求提交源码这点，萌新小白就没必要再靠本地不断新建压缩包来备份源码了，借助git的版本管理，不管怎么改都可以快速回滚。  
然后，git提交是增量更新，每次只提交新增或者删改的内容，而hexo deploy是在本地每次重新生成所有静态文件以后再整个提交。github action能帮我们节省大把时间，把最耗时的hexo generate和hexo deploy的工作丢给CI处理。让我们能够专心与编写博客内容，而不是水文3分钟，提交半小时。

### 发散思维

`Github action`只要监测到`master`分支有所变动就会启动部署，那么顺着这个思路，手机用户可以在网页Github进行小幅修改，例如修改错别字，调整布局之类的。保存后也会启动`Github action`，从而将内容部署到网页上去。

2020年10月后`github`新建仓库默认分支改为`main`，注意更改

