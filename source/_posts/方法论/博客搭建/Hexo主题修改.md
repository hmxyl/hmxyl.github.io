---
title: Hexo主题修改
date: 2022-10-25 19:52:44
tags: 博客搭建
category: 
  - [方法论, 博客搭建]
description: Hexo主题使用记录
---



## 配置新文件模板

路径：根目录下`scaffolds->post.md`。使用`hexo new post [title]`创建文章的时候，应注意把标题里的空格换为`-`

```yml
---
title: {{ title }}
date: {{ date }}
tags:
categories: 
  - [未分类] 
description: ""
---
```



| 参数              | 描述                                                         | 默认值                                                       |
| :---------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| `layout`          | 布局                                                         | [`config.default_layout`](https://hexo.io/zh-cn/docs/configuration#文章) |
| `title`           | 标题                                                         | 文章的文件名                                                 |
| `date`            | 建立日期                                                     | 文件建立日期                                                 |
| `updated`         | 更新日期                                                     | 文件更新日期                                                 |
| `comments`        | 开启文章的评论功能                                           | true                                                         |
| `tags`            | 标签（不适用于分页）                                         |                                                              |
| `categories`      | 分类（不适用于分页）                                         |                                                              |
| `permalink`       | 覆盖文章网址                                                 |                                                              |
| `excerpt`         | Page excerpt in plain text. Use [this plugin](https://hexo.io/docs/tag-plugins#Post-Excerpt) to format the text |                                                              |
| `disableNunjucks` | Disable rendering of Nunjucks tag `{{ }}`/`{% %}` and [tag plugins](https://hexo.io/docs/tag-plugins) when enabled |                                                              |
| `lang`            | Set the language to override [auto-detection](https://hexo.io/docs/internationalization#Path) | Inherited from `_config.yml`                                 |





##  landscape（默认）

### GIT地址

```sh
# landscape（默认）
git clone https://github.com/hexojs/hexo-theme-landscape.git  themes/landscape
```

### 启用主题

修改`_config.yml`

```sh
theme: landscape
```

### 添加文章目录

1. 添加文章目录模块到文章模板中

   打开根目录下的`\themes\landscape\layout\_partial\article.ejs`文件，找到`<div class="article-inner">`， 在其上增加以下代码

   ```ejs
     <% if (!index && post.toc){ %>
       <div class="article-gide">
         <div class="gide-wrap">
           <h3 class="gide-title">文章目录</h3>
           <div class="gide-content">
           <%- toc(post.content) %>
           </div>
         </div>
       </div>
     <% } %>
   ```

2. 在文件`\themes\landscape\source\css\_partial\article.styl`末尾添加CSS样式

   ```stylus
   /* 目录开始 */
   .article-gide
     @media mq-normal
       column(sidebar-column)
   .gide-wrap
     margin: block-margin 0
   .gide-title
     @extend $block-caption
   .gide-content
     color: color-sidebar-text
     text-shadow: 0 1px #fff
     background: color-widget-background
     box-shadow: 0 -1px 4px color-widget-border inset
     border: 1px solid color-widget-border
     padding: 15px
     border-radius: 3px
     a
       color: color-link
       text-decoration: none
       &:hover
         text-decoration: underline
     ul, ol, dl
       ul, ol, dl
         margin-left: 15px
         list-style: disc
   /* 目录结束 */
   ```

3. 修改文件`\themes\landscape\source\css\_variables.styl`，调整基础布局宽度

   ```stylus
   // Grids
   column-width = 80px
   gutter-width = 20px
   ```

   调整为

   ```stylus
   // Grids
   column-width = 100px
   gutter-width = 30px
   ```





## NexT

[初始化配置NexT](https://theme-next.iissnan.com/getting-started.html)

### GIT地址

```bash
# next
git clone https://github.com/theme-next/hexo-theme-next themes/next
```

### 启用主题

修改`_config.yml`

```sh
theme: next
```

### 选择 Scheme

```stylus
scheme: Pisces
```

- Muse - 默认 Scheme，这是 NexT 最初的版本，黑白主调，大量留白
- Mist - Muse 的紧凑版本，整洁有序的单栏外观
- Pisces - 双栏 Scheme

### 设置 语言

```
language: zh-Hans
```

目前 NexT 支持的语言如以下表格所示：

| 语言         | 代码                 | 设定示例                            |
| :----------- | :------------------- | :---------------------------------- |
| English      | `en`                 | `language: en`                      |
| 简体中文     | `zh-Hans`            | `language: zh-Hans`                 |
| Français     | `fr-FR`              | `language: fr-FR`                   |
| Português    | `pt`                 | `language: pt` or `language: pt-BR` |
| 繁體中文     | `zh-hk` 或者 `zh-tw` | `language: zh-hk`                   |
| Русский язык | `ru`                 | `language: ru`                      |
| Deutsch      | `de`                 | `language: de`                      |
| 日本語       | `ja`                 | `language: ja`                      |
| Indonesian   | `id`                 | `language: id`                      |
| Korean       | `ko`                 | `language: ko`                      |

### 设置菜单：`menu`

1. **主题配置文件** 设定菜单内容：

   ```stylus
   menu:
     home: / || fa fa-home
     friendlink: /friendlink/ || fa fa-briefcase
     #about: /about/ || fa fa-user
     tags: /tags/ || fa fa-tags
     categories: /categories/ || fa fa-th
     archives: /archives/ || fa fa-archive
     #schedule: /schedule/ || fa fa-calendar
     #sitemap: /sitemap.xml || fa fa-sitemap
     #commonweal: /404/ || fa fa-heartbeat
   ```

   NexT 默认的菜单

   | 键值       | 设定值                    | 显示文本（简体中文） |              |
   | :--------- | :------------------------ | :------------------- | ------------ |
   | home       | `home: /`                 | 主页                 |              |
   | archives   | `archives: /archives`     | 归档页               |              |
   | categories | `categories: /categories` | 分类页               | 需要手动创建 |
   | tags       | `tags: /tags`             | 标签页               | 需要手动创建 |
   | about      | `about: /about`           | 关于页面             | 需要手动创建 |
   | commonweal | `commonweal: /404.html`   | 公益 404             | 需要手动创建 |

   菜单配置包括三个部分，第一是菜单项（名称和链接），第二是菜单项的显示文本，第三是菜单项对应的图标。 

   NexT 使用的是 [Font Awesome](http://fontawesome.io/) 提供的图标。若你的站点运行在子目录中，请将链接前缀的 `/` 去掉。

   

2. 设置菜单项的显示文本

   在第一步中设置的菜单的名称并不直接用于界面上的展示。Hexo 在生成的时候将使用 这个名称查找对应的语言翻译，并提取显示文本。这些翻译文本放置在 NexT 主题目录下的 `languages/{language}.yml` （`{language}` 为你所使用的语言）。

    以简体中文为例，若你需要添加一个菜单项，比如 `something`。那么就需要修改简体中文对应的翻译文件 `languages/zh-CN.yml`，在 `menu` 字段下添加一项：

   ```
   menu:
     home: 首页
     archives: 归档
     categories: 分类
     tags: 标签
     about: 关于
     search: 搜索
     commonweal: 公益404
     something: 有料
   ```

3. 设定菜单项的图标开关：`menu_icons`

   图标配置在“||” 之后。可从主题文件夹`source\lib\font-awesome\css\all.min.css`中搜索到需要的图标

   ```yaml
   # 菜单图标配置示例
   menu_icons:
     enable: true
     # 將值badge設置為true可以在主題配置文件的menu_settings部分的菜單項中顯示帖子/類別/標籤的計數
     badges: false
   ```

   


### 配置分类：`categories`

1. **主题配置文件**放开 **tags: /tags/ || fa fa-tags** 这行代码就已经配置好里分类。

2. 创建分类目录文件

   ```sh
   $ hexo new page categories
   ```

3. 编辑页面让主题识别页面为分类页面

   上文说到需要编辑页面才能让主题识别这个页面为分类页面，我们只需要根据成功后到提示路径打开`index.md`这个页面文件，打开后默认内容是

   ```
   ---
   title: 文章分类
   date: 2021-01-25 22:37:25
   ---
   ```

   我们需要添加上`type: "categories"`这段代码就能让主题识别该页面为分类页面了

   ```
   ---
   title: 文章分类
   date: 2021-01-25 22:37:25
   type: "categories"
   ---
   ```

   我们就完成了整个分类页面的配置了

4. 给文章设置分类属性

   首先打开需要添加分类的文章，在文章里添加上以下文案就设置好分类了

   ```
   ---
   categories: 
   - Android
   ---
   ```

   如上`categories:Android`表示添加这边文章到 “**Android**” 这个分类下。
   然后我们就可以在博客到分类里看到该分类了。

   ```
   //设置二级分类
   ---
   categories: 
   - Android
   - xxx
   ---
   ```

   如上设置二级分类则该篇文章为 Android 分类下的 XXX 分类下。





### 配置标签：`tags`

1. **主题配置文件**放开 **tags: /tags/ || fa fa-tags** 这行代码就已经配置好里分类。

2. 创建标签目录文件

   ```sh
   $ hexo new page tags
   ```

3. 编辑页面让主题识别页面为标签页面

   上文说到需要编辑页面才能让主题识别这个页面为标签页面，我们只需要根据成功后到提示路径打开`index.md`这个页面文件，添加上`type: "tags"`这段代码就能让主题识别该页面为标签页面了

   ```stylus
   ---
   title: 标签
   date: 2021-01-25 22:54:58
   type: "tags"
   ---
   ```

   

4. 给文章设置标签属性

   ```markdown
   //设置单标签
   ---
   tags:
   - Facebook配置
   ---
   
   //设置多标签 并同时设置分类
   ---
   categories: 
   - Android
   tags:
   - Android
   - RecyclerView
   ---
   ```

   如上`tags:- Facebook配置`表示给这篇文章添加 “**Facebook配置**” 这个分标签。
   然后我们就可以在博客到标签里看到该标签了。



### 设置侧栏：`sidebar`

默认情况下，侧栏仅在文章页面（拥有目录列表）时才显示，并放置于右侧位置。 可以通过修改 **主题配置文件** 中的 `sidebar` 字段来控制侧栏的行为。侧栏的设置包括两个部分，其一是侧栏的位置， 其二是侧栏显示的时机。

1. 设置侧栏的位置，修改 `sidebar.position` 的值，支持的选项有：

   - left - 靠左放置
   - right - 靠右放置

   目前仅 Pisces Scheme 支持 `position` 配置。影响版本**5.0.0**及更低版本。

   ```yaml
   sidebar:
     position: left
   ```

2. 设置侧栏显示的时机，修改 `sidebar.display` 的值，支持的选项有：

   - `post` - 默认行为，在文章页面（拥有目录列表）时显示
   - `always` - 在所有页面中都显示
   - `hide` - 在所有页面中都隐藏（可以手动展开）
   - `remove` - 完全移除

   ```yaml
   sidebar:
     display: post
   ```

   已知侧栏在 `use motion: false` 的情况下不会展示。 影响版本**5.0.0**及更低版本。

### 设置头像：`avatar`

编辑 **主题配置文件**， 修改字段`avatar` ， 值设置成头像的链接地址。其中，头像的链接地址可以是：

| 地址             | 值                                                           |
| :--------------- | :----------------------------------------------------------- |
| 完整的互联网 URI | `http://example.com/avatar.png`                              |
| 站点内的地址     | 将头像放置主题目录下的 `source/uploads/` （新建 uploads 目录若不存在） 配置为：`avatar: /uploads/avatar.png`或者 放置在 `source/images/` 目录下 配置为：`avatar: /images/avatar.png` |

头像设置示例

```sh
avatar: http://example.com/avatar.png
```





### 首页只显示部分摘要（不显示全文）

Next默认是会显示全文的，这样显然很不方便，因此需要一些方法去只显示前面一部分。

1. 修改配置

   首先需要在Next主题的_config.yml中把设置打开：(默认安装时就打开了)

   ```sh
   # Automatically excerpt description in homepage as preamble text.
   excerpt_description: true
   ```

2. 之后有两种方法

   - 方法一：写概述

     在文章的`front-matter`中添加`description`，其中description中的内容就会被显示在首页上，其余一律不显示。

     ```
     ---
     title: 让首页显示部分内容
     date: 2020-02-23 22:55:10
     description: 这是显示在首页的概述，正文内容均会被隐藏。
     ---
     ```


   - 方法二：文章截断

     在需要截断的地方加入：

     ```
     <!--more-->
     ```

     首页就会显示这条以上的所有内容，隐藏接下来的所有内容。
     例如本文会显示到`修改配置`上面。

     这个明显就方便很多，但当然有利有弊，比如开头都是废话首页看着就不是很好看，因此我一般会先选择方法二，如果感觉文章前面的写的不太好再用方法一。

