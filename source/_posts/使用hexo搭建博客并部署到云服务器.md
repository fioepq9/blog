---
title: 使用hexo搭建博客并部署到云服务器
date: 2022-04-06 11:00:34
tags:
  - hexo
categories:
  - 博客搭建
---

## 搭建本地博客

1. 在本地机器上运行：
    ```bash
    $ hexo init <blog-dir-name>
    ```

该命令会在当前目录下，新建<blog-dir-name>目录作为hexo blog。

2. 在本地机器的`<blog-dir-name>`目录下，运行：
    ```bash
    $ hexo s
    ```

![image-20220406084449654](https://raw.githubusercontent.com/ltlin9/note-img/master/img/2022/04/06/20220406-084458.png)

访问[http://localhost:4000](http://localhost:4000)，可以看到hexo框架的hello-world页面。

![image-20220406084608567](https://raw.githubusercontent.com/ltlin9/note-img/master/img/2022/04/06/20220406-084611.png)

## 通过宝塔面板配置云服务器

1. 安装nginx。

   ![image-20220406085201374](https://raw.githubusercontent.com/ltlin9/note-img/master/img/2022/04/06/20220406-085206.png)

2. 创建放置博客文件的目录`<blog-dir>`，并修改权限为777。

   ![image-20220406094435157](https://raw.githubusercontent.com/ltlin9/note-img/master/img/2022/04/06/20220406-110148.png)
3. 查看nginx的config文件目录，可以看到是/www/server/panel/vhost/nginx/
   ![image-20220406095613122](https://raw.githubusercontent.com/ltlin9/note-img/master/img/2022/04/06/20220406-110203.png)
4. 在/www/server/panel/vhost/nginx/目录下，新建`<blog-conf-name>.conf`文件，文件内容为
   ```nginx
   server{
   	listen    80;
   	root <blog-dir>;
   	server_name <域名 or 服务器ip>;
   	location /{
   	}
   }
   ```
5. 在云服务器上创建hook连接git仓库和博客仓库。
    ```bash
    $ git init --bare <hookname>.git
    ```
    在`./<hookname>.git/hooks/post-receive`文件中写入，并设置该文件的权限为777：
    ```shell
    git --work-tree=【<blog-dir>的绝对路径】 --git-dir=【<hookname>.git】的绝对路径 checkout -f
    ```
    
    ```bash
    $ chmod +x ./<hookname>.git/hooks/post-receive
    ```

## 连接本地与云服务器

1. 在本机powershell上运行以下命令，以传递ssh公钥。

   ```powershell
   function ssh-copy-id([string]$userAtMachine, $args){
       $publicKey = "$ENV:USERPROFILE" + "/.ssh/id_rsa.pub"
       if (!(Test-Path "$publicKey")){
           Write-Error "ERROR: failed to open ID file '$publicKey': No such file"
       }
       else {
           & cat "$publicKey" | ssh $args $userAtMachine "umask 077; test -d .ssh || mkdir .ssh ; cat >> .ssh/authorized_keys || exit 1"
       }
   }
   ```

    ```powershell
    ssh-copy-id [服务器用户名]@[服务器ip]
    ```

2. 修改本地机器的`<blog-dir-name>`下的`_config.yml`文件，将其末尾修改为以下：

   ```yaml
   # Deployment
   ## Docs: https://hexo.io/docs/one-command-deployment
   deploy:
     type: 'git'
     repo: [服务器用户名]@[服务器ip]:[<hookname>.git的绝对路径]
     branch: master
   ```

3. 安装hexo-deployer-git：在本地机器的`<blog-dir-name>`下运行

   ```bash
   $ npm install hexo-deployer-git --save
   ```

4. 发布博客：

    ```bash
    $ hexo clean && hexo g && hexo d
    ```

