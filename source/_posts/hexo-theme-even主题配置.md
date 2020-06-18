---

title: hexo-theme-even主题配置

date: 2020/6/2 7:48:54
categories: hexo
tags: 

- hexo
- layout

permalink: Hexo_layout

---



#### **1、文章配置**

- 新建文章
- **Front-matter** 
- 文章分类和标签
- 文章截断
- 自定义文章描述

以上内容在[官方文档](https://hexo.io/zh-cn/docs/writing )中，便不再详细论述。

<!--more--> 

#### **2、添加分类、标签、归档统计页**

使用命令新建三个页面

```bash
hexo new page categories
hexo new page tags
hexo new page archives
```

此时会在heox_path/source下生成三个对应页面名称的文件夹，分别打开其中的index.md文件编辑为

**分类:**

```markdown
title: 分类
layout: categories
```

**标签：**

```markdown
title: 标签
layout: tags
```

**归档：**

```markdown
title: 归档
layout: archives
```



> 注意：以上index.md文件中的title字段表示该页面链接布局在主页面顶端栏的名字，layout字段为hexo_path/themes/even/layout下的ejs或swig模板文件名称，categories、tags、archives都是由even主题自带的模板。



随后在主题配置文件hexo_path/themes/even/_config.yml 中配置顶部菜单栏

```yaml
# ===========================================
# Menu Settings
# ===========================================
menu:
  Home: /
  Categories: /categories/
  Tags: /tags
  Archives: /archives/
```

#### **3、文章访问量统计**

- 通过LeadCloud接口获取文章访问量统计


- 打开LeanCloud官网，进入注册页面注册。完成邮箱激活后，点击头像，进入控制台页面 


- 创建一个新应用，并创建名称为 Counter 的 Class（权限选择无限制） 


- 在LeanCloud中所创建的应用的 设置->应用Key 中查看 app_id 与 app_key 


- 在hexo_path/themes/even/_config.yml配置文件的leancloud字段填入app_id 与 app_key 

```yaml
# LeanCloud
leancloud:
  app_id:
  app_key:
```



**设置Web安全域名**

在你所创建的应用的 设置->安全中心 中设置 Web 安全域名 添加你的域名到 Web 安全域名中（若本地服务也想看到访问量，添加 <http://localhost:4000/>） 



#### **4、设置文章打赏**

配置主题文件中的reward字段开启和关闭文章打赏功能

```yaml
reward:
  enable: true
  qrCode: 
    wechat: /image/reward/wechat.png
    alipay: /image/reward/alipay.png
```

支持微信以及支付宝，修改 qrCode 下对应的二维码图片链接，也可以直接设置成图片的网络链接。 

>注意：该处存放本地图片路径位置可参考[官方文档](https://hexo.io/zh-cn/docs/asset-folders )

当然也可以在文章头部开启打赏功能

```yaml
reward: false
```

> 注：当全局打赏功能关闭时，可以在文章中单独开启。 



#### **5、设置底部社交链接**

目前支持：Email, Stack Overflow, Twitter, Facebook, Github, 微博以及知乎 

修改主题配置文件中的 social 字段下的各个字段开启，为空时即为关闭： 

```yaml
social:
  email: your@email.com
  stack-overflow:
  twitter:
  facebook:
  github: 
  weibo: 
  zhihu: 
```

主题使用的是自定义的 iconfont 图标库。 



#### **6、设置文章版权**

修改主题配置文件中的 copyright 字段开启/关闭，如：

```yaml
copyright:
  enable: true
  # https://creativecommons.org/
  license: '本文采用<a rel="license" href="http://creativecommons.org/licenses/by-nc/4.0/">知识共享署名-非商业性使用 4.0 国际许可协议</a>进行许可'
```

license的值可以是HTML

当文章版权信息开启时，可通过文章 Markdown 头部： 

```yaml
copyright: false
```

进行单篇文章版权信息的关闭。 