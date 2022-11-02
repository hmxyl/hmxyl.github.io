---
title: Hexo搭建博客
date: 2022-10-25 19:52:44
tags: 博客搭建
category: 
  - [方法论, 博客搭建]
description: 'Hexo + Markdown + Github Pages + PigGo + 阿里云OSS + Typora  搭建个人博客'
---



## Hexo 安装和配置

### Hexo相关软件

> 在使用hexo命令时，请使用windows自带的命令行，管理员运行

1. 安装前提

   Hexo是基于Node.js的服务，因此首先需要下载[Node.js](https://nodejs.org/en/)( Node.js 12.0 及以上版本), 以及[Git](https://git-scm.com/)，再安装[Hexo](https://link.zhihu.com/?target=https%3A//hexo.io/zh-cn/docs/)

   ```sh
   C:\Users\Administrator>node --version
   v16.13.0
   
   C:\Users\Administrator>git --version
   git version 2.29.2.windows.2
   
   C:\Users\Administrator>hexo -v
   hexo-cli: 4.3.0
   os: win32 10.0.17763
   node: 16.13.0
   v8: 9.4.146.19-node.13
   uv: 1.42.0
   zlib: 1.2.11
   brotli: 1.0.9
   ares: 1.17.2
   modules: 93
   nghttp2: 1.45.1
   napi: 8
   llhttp: 6.0.4
   openssl: 1.1.1l+quic
   cldr: 39.0
   icu: 69.1
   tz: 2021a
   unicode: 13.0
   ngtcp2: 0.1.0-DEV
   nghttp3: 0.1.0-DEV
   ```

   

2. 安装使用hexo-cil

   ```sh
   npm install -g hexo-cli
   ```


### Hexo初始化博客

1. 初始化

   安装完毕hexo，此时可以选择一个空文件夹建立博客站点框架。执行下面命令，Hexo 将会在指定文件夹中新建所需要的文件。

   ```sh
   $ hexo init <folder>
   $ cd <folder>
   $ npm install
   ```

   执行后Hexo将会在`<folder>`文件夹建立站点文件。若 `<folder>`为空，将在当前文件夹建立站点

   此时，指定文件夹将会出现如下文件目录

   ```sh
   .
   ├── _config.yml
   ├── package.json
   ├── scaffolds
   ├── source
   |   ├── _drafts
   |   └── _posts
   └── themes
   ```

   其中，有几个文件极为重要：

   - `_config.yml` 该文件为网站配置信息，包括网站标题、作者、时间、语言、主题等重要配置和功能。
   - `source/_posts/*.md`
     `source` 文件夹为博文的资源文件夹，其中的`_posts`文件夹储存了`markdown`文件为网站博文。
   - `themes` 文件夹储存了第三方主题。

2. 测试初始化结果

   建立博文

   ```text
   $ hexo new "welcome"
   ```

   此时 `/source/_posts` 文件夹中建立了 `welcome.md` 文件。接着运行

   ```text
   $ hexo server
   ```

   此时命令行提示

   ```sh
   INFO  Validating config
   INFO  Start processing
   INFO  Hexo is running at http://localhost:4000/ . Press Ctrl+C to stop.
   ```

   说明网站已成功在本地部署（ `http://localhost:4000/`），可在命令行执行Crtl+C 停止网站运行。

3. GithubPage部署

   - 在github账号下建立名为 `<github 用户名>.github.io`的仓库（这将是之后的访问网址），可以使用readme.md进行初始化。

   - 设置ssh登录。在命令行中输入`$ ssh-keygen -t rsa -C "GitHub注册邮箱"`直接三个回车，不需要密码。这时在 `C:/Users/<用户名>/.ssh` 文件夹下会建立公钥 `id_rsa.pub` 文件，将其中内容全部复制。打开`Github Settings keys`页面，点击`new SSH key`，填写任意 `title` 和刚才复制的公钥信息，并`Add SSH key`，

      

   - 此时打开Git Bash，输入`ssh git@github.com`

     ```sh
     $ ssh git@github.com
     PTY allocation request failed on channel 0
     Hi hmxyl! You've successfully authenticated, but GitHub does not provide shell access.
     Connection to github.com closed.
     ```

   

   - 参考：https://support.atlassian.com/bitbucket-cloud/docs/set-up-an-ssh-key

     自定义密钥名称<private_key_file>（非id_rsa），后`ssh git@github.com`提示`permission denied`

     - To start the agent, run the following:

     ```sh
     $ eval $(ssh-agent) 
     Agent pid 9700
     ```

     - Enter ssh-add followed by the path to the private key file: 

     ```sh
     $ ssh-add ~/.ssh/<private_key_file>
     ```

     

4. 进入博客文件夹，在 `_config.yml` 文件中修改`deploy`块的信息

   ```yml
   deploy:
     type: git
     repo: git@github.com:hmxyl/hmxyl.github.io.git
     branch: main
   ```

5. 在博客文件夹下打开命令行，安装部署到github.io的依赖

   ```sh
   $ npm install hexo-deployer-git --save
   ```

6. 部署整个博客

   ```sh
   $ hexo clean
   $ hexo generate
   $ hexo deploy
   ```

   或者合并命令

   ```sh
   $ hexo c && hexo g &&  hexo d
   ```

7. 此时主目录下出现`.deploy_git`文件夹，该文件夹与github仓库中的文件一致，为页面文件。此时访问 `Github用户名.github.io` 即可打开博客网页。

   每次部署完，Github通常需要几分钟更新网站，此时多刷新几次网站即可



##  阿里云OSS开通及配置

参考：[阿里云OSS+PicGo-Core搭建图床，配合Typora、Obsidian食用](https://www.cnblogs.com/NFTO21/p/16285829.html)



###  开通及购买服务包

 登录[阿里云](https://www.aliyun.com/)官网，开通对象存储OSS，开通对象存储OSS不用扣费，只有使用OSS才需要扣费。

![image-20220516143447765](D:\0_Notes\Hexo\hmxyl\source\_images\2323423-20220518181130369-127071567.png)

 OSS有两种扣费方式（[产品计费详细介绍](https://help.aliyun.com/document_detail/48259.html)）：

- [按量付费](https://www.aliyun.com/price/product?spm=5176.7933691.J_5253785160.5.6a834c59NplAsJ#/oss/detail/ossbag)
- [包年包月（资源包）](https://www.aliyun.com/price/product?spm=5176.7933691.J_5253785160.5.6a834c59NplAsJ#/oss/detail/ossbag)

![image-20220516144229920](D:\0_Notes\Hexo\hmxyl\source\_images\2323423-20220518181130305-2102262967.png)

 根据需要[按需购买](https://common-buy.aliyun.com/?spm=5176.7933691.J_5253785160.2.6a834c59JnWmz4&commodityCode=ossbag#/buy)对应的资源包即可：（搭建个人图床，购买 40G容量5年（45元）+ 5年 的 100万接口上传次数/年（5元） ）

![image-20220516144601693](D:\0_Notes\Hexo\hmxyl\source\_images\2323423-20220518181130288-239354371.png)

###  基础配置

####  创建Bucket

 打开[OSS管理控制台Bucket页面](https://oss.console.aliyun.com/bucket)，按需创建一个Bucket。
![image-20220516152551602](D:\0_Notes\Hexo\hmxyl\source\_images\2323423-20220518181130538-1554992822.png)

 创建Bucket成功

![image-20220516154556221](D:\0_Notes\Hexo\hmxyl\source\_images\2323423-20220518181130203-786734777.png)

####  Bucket图片上传及访问

 进入新建的Bucket，在文件管理->上传文件中即可开始上传文件，

![image-20220516155157249](D:\0_Notes\Hexo\hmxyl\source\_images\2323423-20220518181130324-816099928.png)![image-20220516155507591](D:\0_Notes\Hexo\hmxyl\source\_images\2323423-20220518181130545-1477425769.png)

 点击详情，即可看到上传图片的URL，复制URL到浏览器中访问图片。

![image-20220516160202041](D:\0_Notes\Hexo\hmxyl\source\_images\2323423-20220518181130375-920098593.png)

###  AccessKey配置

####  创建子用户和AccessKey

 鼠标移动到头像上即可看到**AccessKey管理**，打开管理页面，

![image-20220516171100691](D:\0_Notes\Hexo\hmxyl\source\_images\2323423-20220518181130218-1943488233.png) 

 **推荐使用子账户创建的AccessKey**（[RAM 访问控制 ](https://ram.console.aliyun.com/users)中创建子账户）
![image-20220516171247987](D:\0_Notes\Hexo\hmxyl\source\_images\2323423-20220518181130421-634915099.png)



![image-20220516172210041](D:\0_Notes\Hexo\hmxyl\source\_images\2323423-20220518181130422-386866137.png)

 选中刚才创建的子用户，即可创建**AccessKey**并获取**AccessKey Secret**。

> **AccessKey Secret只能查看一次哦，建议复制到自己本地存储，后面PicGo-Core配置需要使用到**

![image-20220516172517201](D:\0_Notes\Hexo\hmxyl\source\_images\2323423-20220518181130493-1428718749.png)

####  Bucket授权子用户权限

 进入权限管理页面，

![image-20220516173811851](D:\0_Notes\Hexo\hmxyl\source\_images\2323423-20220518181130396-775910505.png)

 配置子用户权限，按需勾选即可，

![image-20220516174245990](D:\0_Notes\Hexo\hmxyl\source\_images\2323423-20220518181130449-866487166.png) 

 **至此，图床所需的所有配置已弄完**



### 下载OSS 管理工具`oss-browser`



1. 下载： [oss-browser](https://gosspublic.alicdn.com/oss-browser/1.16.0/oss-browser-win32-x64.zip?spm=5176.8466032.help.16.50a01450Brdxse&file=oss-browser-win32-x64.zip)

2. 登录

   | 参数            | 说明                             |
   | --------------- | -------------------------------- |
   | AccesskeyId     | 创建RAM用户时的 AccessKey ID     |
   | AccessKeySecret | 创建RAM用户时的 AccessKey Secret |
   | 预设OSS 路径    | `oss://创建的Bucket名称`         |

## [PicGo](https://picgo.github.io/PicGo-Doc/) 安装及图床配置

1. 安装：参考[PicGo安装](https://picgo.github.io/PicGo-Doc/)

2. 配置图床

   | 参数           | 说明                                                         |
   | -------------- | ------------------------------------------------------------ |
   | 设定keyId      | 创建RAM用户时的 AccessKey ID                                 |
   | 设定KeySecret  | 创建RAM用户时的 AccessKey Secret                             |
   | 设定储存空间名 | 新建的Bucket名称                                             |
   | 确定存储区域   | 创建的Bucket详情中查看：`地域节点`  中的二级域名 `oss-cn-城市` |

3. 测试上传一张图片



## Typora 中 上传图片配置

配置： 文件-> 偏好设置->图像

![image-20221027172837218](D:\0_Notes\Hexo\hmxyl\source\_images\image-20221027172837218.png) 
