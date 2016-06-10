# 一　Name-based virtual servers
<pre><code>
server {
    listen      80;
    server_name example.org www.example.org;
}
server {
    listen      80;
    server_name example.net www.example.net;
}
server {
    listen      80;
    server_name example.com www.example.com;
}
</code></pre>

以上配置端口相同，那么nginx就以server_name为依据寻找相匹配的server来对请求进行处理。如果找不到匹配的，那么就以默认的server进行处理（一般指第一个server）,在0.8.21之后(之前用default)可以使用default_server进行指定默认的server
例如：
<pre><code>
server {
    listen      80 default_server;
    server_name example.net www.example.net;
}
</code></pre>

#二　How to prevent processing requests with undefined server names
If requests without the “Host” header field should not be allowed, a server that just drops the requests can be defined:
<pre><code>
server {
    listen      80;
    server_name "";
    return      444;
}
</code></pre>
http协议：
Host头域指定请求资源的Intenet主机和端口号，必须表示请求url的原始服务器或网关的位置
Here, the server name is set to an empty string that will match requests without the “Host” header field, and a special nginx’s non-standard code 444 is returned that closes the connection.
# 三　Mixed name-based and IP-based virtual servers
<pre><code>
server {
    listen      192.168.1.1:80;
    server_name example.org www.example.org;
}
server {
    listen      192.168.1.1:80;
    server_name example.net www.example.net;
}
server {
    listen      192.168.1.2:80;
    server_name example.com www.example.com;
}
</code></pre>

>In this configuration, nginx first tests the IP address and port of the request against the listen directives of the server blocks. It then tests the “Host” header field of the request against the server_name entries of the server blocks that matched the IP address and port. If the server name is not found, the request will be processed by the default server. For example, a request for www.example.com received on the 192.168.1.1:80 port will be handled by the default server of the 192.168.1.1:80 port, i.e., by the first server, since there is no www.example.com defined for this port.

nginx 首先检查ip地址和端口，如果都匹配了，在检查所有的server_name.如果找不到就会按照定义的默认的sever进行处理，本例中是第一个。如果加了
default_server属性，那就按照那个server进行处理。

#四　A simple PHP site configuration
<pre><code>
server {
    listen      80;
    server_name example.org www.example.org;
    root        /data/www;
    location / {
        index   index.html index.php;
    }
    location ~* \.(gif|jpg|png)$ {
        expires 30d;
    }
    location ~ \.php$ {
        fastcgi_pass  localhost:9000;
        fastcgi_param SCRIPT_FILENAME
                      $document_root$fastcgi_script_name;
        include       fastcgi_params;
    }
}
</code></pre>
　
nginx 对于请求的匹配总是寻找最契合，最匹配的那个location.如果有正则表达式匹配到了，那就停止寻找，否则就用相对来说最匹配的location进行处理。同时nginx 对请求的匹配是不考虑参数的。
>nginx first searches for the most specific prefix location given by literal strings regardless of the listed order. In the configuration above the only prefix location is “/” and since it matches any request it will be used as a last resort. Then nginx checks locations given by regular expression in the order listed in the configuration file. The first matching expression stops the search and nginx will use this location. If no regular expression matches a request, then nginx uses the most specific prefix location found earlier.

例如：
1.
>A request “/logo.gif” is matched by the prefix location “/” first and then by the regular expression “\.(gif|jpg|png)$”, therefore, it is handled by the latter location. Using the directive “root /data/www” the request is mapped to the file /data/www/logo.gif, and the file is sent to the client.

2.
>A request “/index.php” is also matched by the prefix location “/” first and then by the regular expression “\.(php)$”. Therefore, it is handled by the latter location and the request is passed to a FastCGI server listening on localhost:9000. The fastcgi_param directive sets the FastCGI parameter SCRIPT_FILENAME to “/data/www/index.php”, and the FastCGI server executes the file. The variable $document_root is equal to the value of the root directive and the variable $fastcgi_script_name is equal to the request URI, i.e. “/index.php”.

3.
>A request “/about.html” is matched by the prefix location “/” only, therefore, it is handled in this location. Using the directive “root /data/www” the request is mapped to the file /data/www/about.html, and the file is sent to the client.

4.
>Handling a request “/” is more complex. It is matched by the prefix location “/” only, therefore, it is handled by this location. Then the index directive tests for the existence of index files according to its parameters and the “root /data/www” directive. If the file /data/www/index.html does not exist, and the file /data/www/index.php exists, then the directive does an internal redirect to “/index.php”, and nginx searches the locations again as if the request had been sent by a client. As we saw before, the redirected request will eventually be handled by the FastCGI server.

index指令首先测试/data/www/index.html的存在性，如果不存在/data/www/index.php存在，那么就会进行一个跳转到/index.php.然后寻找合适的location进行处理。