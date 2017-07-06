# docker 部署nginx

在Docker下部署Nginx，包括：

* 部署一个最简单的Nginx，可以通过80端口访问默认的网站
* 设置记录访问和错误日志的路径
* 设置静态网站的路径
* 通过proxy_pass将HTTP请求反向代理到nodejs Web App
* 设置HTTPS
* 如果你还没有安装Docker环境，可参考在[Docker中运行Node.js的Web应用](http://blog.shiqichan.com/Dockerizing-a-Node-js-Web-Application/)。

**最简单的命令，让Nginx跑起来**
```
$ sudo docker run -it -p 80:80 dockerfile/nginx
```

如果是第一次，下载nginx镜像需要点时间。

然后，可以通过浏览器根据地址访问到一个默认的网页，说明Nginx成功跑起来了。


**设置记录访问和错误日志**

Nginx有2个日志：

* access.log，记录每个HTTP请求信息
* error.log，记录Nginx运行中的错误，用于排错

```
$ sudo docker run -it -p 80:80  -v `pwd`/logs:/var/log/nginx dockerfile/nginx
```

这个命令会在当前目录下创建logs目录，存放access.log和error.log。

**设置静态网站路径**

需要创建目录：

1、config，目录下放一个文件，名为server，Nginx静态网站配置文件

2、www，目录下放html文件，比如index.htm

server文件：
```
server {
        listen 80;

        root /www;
        index index.html index.htm;

        server_name localhost;
}
```


命令如下：
```
sudo docker run -it -p 80:80 -v `pwd`/www:/www -v `pwd`/config:/etc/nginx/sites-enabled  -v `pwd`/logs:/var/log/nginx dockerfile/nginx
```

或者如下命令：
```
docker run -d -p 80:80 -v /Users/junzhang/data/nginx/logs/:/var/log/nginx/ -v /Users/junzhang/data/nginx/html/:/www --name nginx1 nginx
```

解释一下：
```
-vpwd/www:/www，将当前路径下的www目录设置为/www，和server配置的路径对应
-vpwd/config:/etc/nginx/sites-enabled，server文件的本地路径，映射到docker容器的nginx配置路径
```

**设置反向代理到Nodejs Web App**

需要先能将Nodejs的容器跑起来，然后再考虑怎样通过Nginx的反向代理。

可参见[在Docker中运行Node.js的Web应用](http://blog.shiqichan.com/Dockerizing-a-Node-js-Web-Application/)

假设我有个express.js项目，在当前目录下的webapp目录中，使用docker命令类似这样：
```
sudo docker run -d -p 3000:3000 --name ProtoWebApp -v `pwd`/webapp:/webapp -w /webapp  node npm start
```

然后，将前面例子中config目录下的server文件做点改动：
```
server {
        listen 80;

        #root /www;
        #index index.html index.htm;

        server_name localhost;

        location / {
                proxy_pass  http://localhost:3000;
        }
}
```

之后，用下面的命令将nginx跑起来：
```
sudo docker run -it -p 80:80 --link ProtoWebApp:localhost -v `pwd`/config:/etc/nginx/sites-enabled  -v `pwd`/logs:/var/log/nginx dockerfile/nginx
```

**设置HTTPS**
只需在运行nginx容器的时候设置SSL的路径：-v <certs-dir>:/etc/nginx/certs。

当然，nginx首先要设置，参见[配置HTTPS服务器](http://nginx.org/cn/docs/http/configuring_https_servers.html)
