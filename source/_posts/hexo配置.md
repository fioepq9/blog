---
title: hexo配置
date: 2022-04-06 12:46:20
tags:
  - hexo
categories:
  - 博客搭建
---

## hexo安装NexT主题

博客目录：`<blog-dir>`

### 下载NexT主题

在`<blog-dir>`路径下运行以下命令：
```bash
$ git clone https://github.com/next-theme/hexo-theme-next themes/next
```

### 修改`_config.yml`文件

1. 查看`<blog-dir>/themes/next/languages`目录下中文语言的文件名：`<zh-CN>.yml`。

2. 修改`<blog-dir>/_config.yml`文件。

    ```yaml
    # Site
    language: zh-CN
    timezone: 'Etc/GMT+8'
    # Extensions
    ## Plugins: https://hexo.io/plugins/
    ## Themes: https://hexo.io/themes/
    theme: next
    ```

3. 修改`<blog-dir>/themes/next/_config.yml`文件。

   ```yaml
   # Schemes
   #scheme: Muse
   #scheme: Mist
   #scheme: Pisces
   scheme: Gemini
   
   back2top:
     enable: true
     # Back to top in sidebar.
     sidebar: true
     # Scroll percent label in b2t button.
     scrollpercent: true
     
   codeblock:
     # Code Highlight theme
     # All available themes: https://theme-next.js.org/highlight/
     theme:
       light: atom-one-dark
       dark: a-dark
     prism:
       light: prism
       dark: prism-dark
     # Add copy button on codeblock
     copy_button:
       enable: true
       # Available values: default | flat | mac
       style: mac
   ```

## hexo添加分类及标签

### 1. 创建“分类”选项

```bash
$ hexo new page categories
```

修改创建的index.md内容为：

```yaml
title: categories
date: 2022-04-06 12:49:50
type: "categories"
```

### 2. 创建“标签”选项

```bash
$ hexo new page tags
```

修改创建的index.md内容为：

```yaml
title: tags
date: 2022-04-06 12:54:24
type: "tags"
```

## 隐藏网页底部 power By Hexo / 强力驱动

1. 打开`<blog-dir>/themes/next/layout/_partials/footer.njk`。

2. 使用`<!--something-->`注释掉以下代码。

   ```jinja2
   {%- if theme.footer.powered %}
     <div class="powered-by">
       {%- set next_site = 'https://theme-next.js.org' if theme.scheme === 'Gemini' else 'https://theme-next.js.org/' + theme.scheme | lower + '/' %}
       {{- __('footer.powered', next_url('https://hexo.io', 'Hexo') + ' & ' + next_url(next_site, 'NexT.' + theme.scheme)) }}
     </div>
   {%- endif %}
   ```

   