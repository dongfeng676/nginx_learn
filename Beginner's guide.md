# nginx learn start
## 一  Starting, Stopping, and Reloading Configuration
1.1
> nginx has one master process and several worker processes. The main purpose of the master process is to read and evaluate configuration, and maintain worker processes. Worker processes do actual processing of requests. nginx employs event-based model and OS-dependent mechanisms to efficiently distribute requests among worker processes. The number of worker processes is defined in the configuration file and may be fixed for a given configuration or automatically adjusted to the number of available CPU cores (see worker_processes).

nginx 具有一个master进程和数个worker进程。worker进程数量可以在配置文件中设置。master进程主要负责配置文件读取，评估，和worker进程的管理。


1.2
修改配置文件可以使用
> nginx -s reload

来使新的配置文件生效，具体原理如下：
>Once the master process receives the signal to reload configuration, it checks the syntax validity of the new configuration file and tries to apply the configuration provided in it. If this is a success, the master process starts new worker processes and sends messages to old worker processes, requesting them to shut down. Otherwise, the master process rolls back the changes and continues to work with the old configuration. Old worker processes, receiving a command to shut down, stop accepting new connections and continue to service current requests until all such requests are serviced. After that, the old worker processes exit.

当master进程收到信号之后就重载配置文件，首先检查语法问题。如果正确，就会产生新的worker进程，并且发送停止信息给老的worker进程，老的worker进程收到信号之后停止接受新的请求，继续处理当前请求直到结束。
否则，master进程会依然使用一起的配置文件。

## 二  Configuration File’s Structure
2.1 
>nginx consists of modules which are controlled by directives specified in the configuration file. Directives are divided into simple directives and block directives. A simple directive consists of the name and parameters separated by spaces and ends with a semicolon (;). A block directive has the same structure as a simple directive, but instead of the semicolon it ends with a set of additional instructions surrounded by braces ({ and }). If a block directive can have other directives inside braces, it is called a context (examples: events, http, server, and location).

nginx 是由一些列模块组成，这些模块又包含了一些特定的指令。
指令分为简单指令和块指令。
简单指令是有名字和参数组成以空格隔开，以分号结束。（可参考编程中的方法）
块指令是由一些列的简单指令构成，但是以{}包括起来。例如：events,http,server,location.
##三　Serving Static Content
nginx 配置文件代码
<pre><code>	
server {
	  listen 8088;
  		location / {
    		root /home/zhang/my_github/nginx_learn/example/example1/data/wwww;
 	 }
 	 location /images/ {
   		 root /home/zhang/my_github/nginx_learn/example/example1/data;
  	}
}
</code></pre>

nginx使用listen 和servername来区分请求是由那个server来处理。
如上配置当请求
>http://localhost:8088/images/1.jpg

为处理这个请求会匹配到第二个location，然后将root指定的路径和请求中的路径进行拼接，然后去文件系统上找相应的资源。结果：
>/home/zhang/my_github/nginx_learn/example/example1/data/images/1.jpg

## 四　Setting Up a Simple Proxy Server
nginx 经常被作为一个代理服务器进行使用。

<pre><code>
server {
    location / {
        proxy_pass http://localhost:8080/;
    }

    location ~ \.(gif|jpg|png)$ {
        root /data/images;
    }
}
</code></pre>

>This server will filter requests ending with .gif, .jpg, or .png and map them to the /data/images directory (by adding URI to the root directive’s parameter) and pass all other requests to the proxied server configured above.

对于图片处理到会到/data/images目录下去寻找，其他请求则转发给代理服务器。
\.(gif|jpg|png)$这是一个正则表达式，表示以.gif .jpg .png结尾。正则表达式前面要放置一个~    。

proxy_pass　后边是一些列参数用来指定代理服务器的协议，名称和端口。 






















