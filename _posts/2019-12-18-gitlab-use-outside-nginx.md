---
layout: post
title:  "Gitlab使用外部nginx"
categories: Linux
tags: Linux Nginx Git
---

* content
{:toc}

在个人服务器上配置了gitlab用于平常开发的时候放一下个人项目，但是发现gitlab有自己的nginx，用自行安装的nginx转发的话，

在页面上新建project的时候链接会变成127.0.0.1，实属难看，所以还是要配置一下外置的nginx转发；


		   					    
				



### 首先 


打开gitlab的配置文件

`vim /etc/gitlab/gitlab.rb`


找到并设置`nginx['enable']false`

然后新增nginx配置


```shell
upstream gitlab {
  server unix:/var/opt/gitlab/gitlab-workhorse/socket;
}
 
server {
  listen *:80;
 
  server_name gitlab.xxx.com;   # 请修改为你的域名
 
  server_tokens off;     # don't show the version number, a security best practice
  root /opt/gitlab/embedded/service/gitlab-rails/public;
 
  # Increase this if you want to upload large attachments
  # Or if you want to accept large git objects over http
  client_max_body_size 250m;
 
  # individual nginx logs for this gitlab vhost
  access_log  /var/log/gitlab/gitlab_access.log;
  error_log    /var/log/gitlab/gitlab_error.log;
 
  location / {
    # serve static files from defined root folder;.
    # @gitlab is a named location for the upstream fallback, see below
    try_files $uri $uri/index.html $uri.html @gitlab;
  }
 
  # if a file, which is not found in the root folder is requested,
  # then the proxy pass the request to the upsteam (gitlab unicorn)
  location @gitlab {
    # If you use https make sure you disable gzip compression 
    # to be safe against BREACH attack
 
    proxy_read_timeout 300; # Some requests take more than 30 seconds.
    proxy_connect_timeout 300; # Some requests take more than 30 seconds.
    proxy_redirect     off;
 
    proxy_set_header   X-Forwarded-Proto $scheme;
    proxy_set_header   Host              $http_host;
    proxy_set_header   X-Real-IP         $remote_addr;
    proxy_set_header   X-Forwarded-For   $proxy_add_x_forwarded_for;
    proxy_set_header   X-Frame-Options   SAMEORIGIN;
 
    proxy_pass http://gitlab; # 此处输入你要转发的本机项目端口，例如：127.0.0.1:8899
  }
 
  # Enable gzip compression as per rails guide: http://guides.rubyonrails.org/asset_pipeline.html#gzip-compression
  # WARNING: If you are using relative urls do remove the block below
  # See config/application.rb under "Relative url support" for the list of
  # other files that need to be changed for relative url support
  location ~ ^/(assets)/  {
    root /opt/gitlab/embedded/service/gitlab-rails/public;
    # gzip_static on; # to serve pre-gzipped version
    expires max;
    add_header Cache-Control public;
  }
 
  error_page 502 /502.html;
}
```










   













