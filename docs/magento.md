# magento研究

Magento是一套专业开源的电子商务系统。 Magento设计得非常灵活，具有模块化架构体系和丰富的功能。易于与第三方应用系统无缝集成。其面向企业级应用，可处理各方面的需求，以及建设一个多种用途和适用面的电子商务网站。 包括购物、航运、产品评论等等，充分利用开源的特性，提供代码库的开发，非常规范的标准，易于与第三方应用系统无缝集成。

## 功能
* 强大的商品属性组合: magento具有强大的商品属性系统，能够支持任意商品类型。无论销售服装、电器还是家具，magento都能完美支持商品的所有属性，并能灵活的修改和管理。
* 购物车价格规则: magento的购物车价格规则不同于传统的优惠券功能。完整的逻辑关系处理能够创建出任意的优惠券方式，让您在制定促销方案时游刃有余。
* 灵活的模版系统: magento灵活的模版系统可以让您做到模版一键切换。弹指之间，更改网站所有页面风格，包括email营销邮件央视，再也不用为更改页面风格而烦恼。
* 多网店系统: 假如你拥有多个品牌，或者多家网店。magento的多店铺可以让您配置多个网店，多个域名，单使用统一的管理控制台，节约您的管理时间和成本。
* 完善的插件体系: magento的插件系统可以灵活的为系统增减功能。您不必修改已有代码，只需要安装相应功能的插件就能实现。极大的缩减了新增功能的开发成本。
* 安全加密: 电子商务系统的数据是企业的商业机密。magento支持ssl加密传输前台和后台的数据并且支持后台权限的分级控制，确保您的数据万无一失。
* 企业应用集成: magento的web service开发API系统能够和您企业的其他系统进行无缝集成。不论是ERP还是CRM, magento都能保证数据的同步和准确，使您的企业应用的效率最大化。

## 特色
* 容易整合: magento让整合第三方厂商解决方案变得更加容易，透过magento建立客制化网站可以节省您很多的时间和资源。
* 新颖的功能: 像产品标签、多送货地址或产品比较系统等功能，您不需要支付额外的费用来获得，在现成的magento系统中，您可以发现更多。
* 专业与社群支援: 不像其他的开源码解决方案，magento提供专业、可信赖的支援，您也可以从热情的社群中取得协助，国内也有magento的爱好者创建中文社区。
* 完整的扩充性: 无论网站经历了一夜或一年的成长，您不需要担心选择的方案无法应付，magento提供了完整的扩充性。

## 安装
本文介绍magento 2.2.3版本的安装，源文件下载地址: https://magento.com/tech-resources/download.

注意下载之前需要注册账号，填写必要的个人或公司信息。这里省略这些步骤，直接进入安装。

### nginx配置
```
server {
    listen 80;
    server_name mage.zjtech.cc;
    set $MAGE_ROOT /your/path/to/wwwroot/;
    root $MAGE_ROOT/pub;
    index index.php;
    autoindex off;
    charset UTF-8;
    error_page 404 403 = /errors/404.php;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location /pub/ {
        location ~ ^/pub/media/(downloadable|customer|import|theme_customization/.*\.xml) {
            deny all;
        }
        alias $MAGE_ROOT/pub/;
        add_header X-Frame-Options "SAMEORIGIN";
    }
    location /static/ {
   
        location ~ ^/static/version {
            rewrite ^/static/(version\d*/)?(.*)$ /static/$2 last;
        }

        location ~* \.(ico|jpg|jpeg|png|gif|svg|js|css|swf|eot|ttf|otf|woff|woff2)$ {
            add_header Cache-Control "public";
            add_header X-Frame-Options "SAMEORIGIN";
            expires +1y;

            if (!-f $request_filename) {
                rewrite ^/static/?(.*)$ /static.php?resource=$1 last;
            }
        }
        location ~* \.(zip|gz|gzip|bz2|csv|xml)$ {
            add_header Cache-Control "no-store";
            add_header X-Frame-Options "SAMEORIGIN";
            expires    off;

            if (!-f $request_filename) {
                rewrite ^/static/?(.*)$ /static.php?resource=$1 last;
            }
        }
        if (!-f $request_filename) {
            rewrite ^/static/?(.*)$ /static.php?resource=$1 last;
        }
        add_header X-Frame-Options "SAMEORIGIN";
    }

    location ~ \.php($|/) {
        fastcgi_pass 127.0.0.1:9000;
        fastcgi_index index.php;

        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_script_name;
        fastcgi_split_path_info ^(.+?\.php)(/.*)$;
        fastcgi_param PATH_INFO $fastcgi_path_info;
        fastcgi_param RUN_ENV 'development';
        client_max_body_size 100M;
        include fastcgi_params;
    } 

    location /media/ {
        try_files $uri $uri/ /get.php?$query_string;

        location ~ ^/media/theme_customization/.*\.xml {
            deny all;
        }

        location ~* \.(ico|jpg|jpeg|png|gif|svg|js|css|swf|eot|ttf|otf|woff|woff2)$ {
            add_header Cache-Control "public";
            add_header X-Frame-Options "SAMEORIGIN";
            expires +1y;
            try_files $uri $uri/ /get.php?$query_string;
        }
        location ~* \.(zip|gz|gzip|bz2|csv|xml)$ {
            add_header Cache-Control "no-store";
            add_header X-Frame-Options "SAMEORIGIN";
            expires    off;
            try_files $uri $uri/ /get.php?$query_string;
        }
        add_header X-Frame-Options "SAMEORIGIN";
    }

    location /media/customer/ {
        deny all;
    }

    location /media/downloadable/ {
        deny all;
    }

    location /media/import/ {
        deny all;
    }

    # PHP entry point for main application
    location ~ (index|get|static|report|404|503|health_check)\.php$ {
        try_files $uri =404;
        fastcgi_pass   127.0.0.1:9000;
        fastcgi_buffers 1024 4k;

        fastcgi_param  PHP_FLAG  "session.auto_start=off \n suhosin.session.cryptua=off";
        fastcgi_param  PHP_VALUE "memory_limit=756M \n max_execution_time=18000";
        fastcgi_read_timeout 600s;
        fastcgi_connect_timeout 600s;

        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
        include        fastcgi_params;
    }

    gzip on;
    gzip_disable "msie6";

    gzip_comp_level 6;
    gzip_min_length 1100;
    gzip_buffers 16 8k;
    gzip_proxied any;
    gzip_types
        text/plain
        text/css
        text/js
        text/xml
        text/javascript
        application/javascript
        application/x-javascript
        application/json
        application/xml
        application/xml+rss
        image/svg+xml;
    gzip_vary on;

# Banned locations (only reached if the earlier PHP entry point regexes don't match)
    location ~* (\.php$|\.htaccess$|\.git) {
        deny all;
    }
}
```

注意,将配置中的your/path/to/wwwroot修改为你自己的网站根目录。

### 创建数据库
```create database magento;```

### 安装magento
按照界面提示，一步步操作:
* 检查环境支持
* 配置mysql访问
* 配置管理员账号
* 安装。

### 安装后常见问题

1) 安装进度卡到73%不动了, 这时候一般已经安装完毕，只是缺少资源文件，卡住不动了，可按照下面的方式解决:

解决方案: 调整php.ini参数，然后重启php-fpm. 但是这个一般解决不了，需要执行命令sudo php bin/magento setup:static-content:deploy -f
参考链接:
* [修改php.ini参数](https://github.com/magento/magento2/issues/4929)
* [生成静态内容](https://magento.stackexchange.com/questions/156221/magento-2-install-stuck-at-73)

执行完上面的操作，后台可以正常使用了。

2) 前端样式乱掉, 这个时候同样是资源找不到，解决办法如下:
```
#Step 1.

$ php bin/magento setup:static-content:deploy

# Step 2.
$ php bin/magento indexer:reindex

# Step 3.

# make sure apache “rewrite_module” is enable and then restart the server

# Step 4.

$ chown -R www-data:www-data /var/www/html/magento2 

# Step 5.

$ chmod -R 777 /var/www/html/magento2 

# Step 6.
# delete cache folder under var/cache
```

参考链接:
* https://magento.stackexchange.com/questions/97209/magento-2-css-and-javascript-not-loading-from-correct-folder

前端展示样式乱掉，执行创建索引步骤可解决。




