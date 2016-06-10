#一 Server names defined
>They may be defined using exact names, wildcard names, or regular expressions:
<pre><code>
server {
    listen       80;
    server_name  example.org  www.example.org;
    ...
}
server {
    listen       80;
    server_name  *.example.org;
    ...
}
server {
    listen       80;
    server_name  mail.*;
    ...
}
server {
    listen       80;
    server_name  ~^(?<user>.+)\.example\.net$;
    ...
}
</code></pre>

servername可以被定义用准确的名称，通配符名称或者正则表达式名称。
当匹配到了多个名称，那么使用一下规则解析：
>When searching for a virtual server by name, if name matches more than one of the specified variants, e.g. both wildcard name and regular expression match, the first matching variant will be chosen, in the following order of precedence:

>exact name
longest wildcard name starting with an asterisk, e.g. “*.example.org”
longest wildcard name ending with an asterisk, e.g. “mail.*”
first matching regular expression (in order of appearance in a configuration file)

#二  Wildcard names

>A wildcard name may contain an asterisk only on the name’s start or end, and only on a dot border. The names “www.*.example.org” and “w*.example.org” are invalid. However, these names can be specified using regular expressions, for example, “~^www\..+\.example\.org$” and “~^w.*\.example\.org$”. An asterisk can match several name parts. The name “*.example.org” matches not only www.example.org but www.sub.example.org as well.
A special wildcard name in the form “.example.org” can be used to match both the exact name “example.org” and the wildcard name “*.example.org”.

通配符只有*，且只能放到name开头或结尾，其他无效。
特殊的.example.org既可以当准确名称，也可以当做通配符名称。

#三 Regular expressions names

>The regular expressions used by nginx are compatible with those used by the Perl programming language (PCRE). To use a regular expression, the server name must start with the tilde character:
>otherwise it will be treated as an exact name, or if the expression contains an asterisk, as a wildcard name (and most likely as an invalid one). Do not forget to set “^” and “$” anchors. They are not required syntactically, but logically. Also note that domain name dots should be escaped with a backslash. A regular expression containing the characters “{” and “}” should be quoted:

正则表达式名称必须以~开头。
同时最好在正则表达式开头结尾加上^ 和$.
更加要注意的是正则中的转义问题。

正则表达式里边解析出来的变量可以在后边的指令中使用
>A named regular expression capture can be used later as a variable:
<pre><code>
server {
    server_name   ~^(www\.)?(?<domain>.+)$;
    location / {
        root   /sites/$domain;
    }
}
</code></pre>

#四 Miscellaneous names
>If someone makes a request using an IP address instead of a server name, the “Host” request header field will contain the IP address and the request can be handled using the IP address as the server name:
<pre><code>
server {
    listen       80;
    server_name  example.org  www.example.org   ""   192.168.1.1;
}<pre><code>

In catch-all server examples the strange name “_” can be seen:

<pre><code>
server {
    listen       80  default_server;
    server_name  _;
    return       444;
}</code></pre>

>There is nothing special about this name, it is just one of a myriad of invalid domain names which never intersect with any real name. Other invalid names like “--” and “!@#” may equally be used.

>It is possible to define servers listening on ports *:80 and *:8080, and direct that one will be the default server for port *:8080, while the other will be the default for port *:80:

<pre><code>
server {
    listen       80;
    listen       8080  default_server;
    server_name  example.net;
    ...
}
</code></pre>

#五  Optimization

>Exact names, wildcard names starting with an asterisk, and wildcard names ending with an asterisk are stored in three hash tables bound to the listen ports. The sizes of hash tables are optimized at the configuration phase so that a name can be found with the fewest CPU cache misses. The details of setting up hash tables are provided in a separate document.

>The exact names hash table is searched first. If a name is not found, the hash table with wildcard names starting with an asterisk is searched. If the name is not found there, the hash table with wildcard names ending with an asterisk is searched.

>Searching wildcard names hash table is slower than searching exact names hash table because names are searched by domain parts. Note that the special wildcard form “.example.org” is stored in a wildcard names hash table and not in an exact names hash table.

>Regular expressions are tested sequentially and therefore are the slowest method and are non-scalable.

>For these reasons, it is better to use exact names where possible. For example, if the most frequently requested names of a server are example.org and www.example.org, it is more efficient to define them explicitly:

<pre><code>
server {
    listen       80;
    server_name  example.org  www.example.org  *.example.org;
    ...
}
</code></pre>
>than to use the simplified form:
<pre><code>
server {
    listen       80;
    server_name  .example.org;
    ...
}
</code></pre>

有关原理的一些解释。server_name保存在端口对应的hash表中，因此如果使用准确名称会有一个更高的效率。
>If a large number of server names are defined, or unusually long server names are defined, tuning the server_names_hash_max_size and server_names_hash_bucket_size directives at the http level may become necessary. The default value of the server_names_hash_bucket_size directive may be equal to 32, or 64, or another value, depending on CPU cache line size. If the default value is 32 and server name is defined as “too.long.server.name.example.org” then nginx will fail to start and display the error message:

>could not build the server_names_hash,
you should increase server_names_hash_bucket_size: 32
In this case, the directive value should be increased to the next power of two:
</code></pre>
http {
    server_names_hash_bucket_size  64;
    ...
</code></pre>
>If a large number of server names are defined, another error message will appear:

>could not build the server_names_hash,
you should increase either server_names_hash_max_size: 512
or server_names_hash_bucket_size: 32
In such a case, first try to set server_names_hash_max_size to a number close to the number of server names. Only if this does not help, or if nginx’s start time is unacceptably long, try to increase server_names_hash_bucket_size.

>If a server is the only server for a listen port, then nginx will not test server names at all (and will not build the hash tables for the listen port). However, there is one exception. If a server name is a regular expression with captures, then nginx has to execute the expression to get the captures.

server_name大小由限制的。如果报错的话可以修改这个配置。

