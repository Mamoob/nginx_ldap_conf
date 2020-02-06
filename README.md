# nginx_ldap_conf

https://techlist.top/nginx-install-from-source/
https://stackoverflow.com/questions/42078674/nginx-service-failed-to-read-pid-from-file-run-nginx-pid-invalid-argument

yum install openldap-devel openssl-devel pcre-devel yum-utils gcc
yumdownloader --source nginx
rpm2cpio nginx-#-#.rpm | cpio -idmv
tar xf nginx-#-#.tar.gz
cd nginx-#-#
./configure --user=nginx --group=nginx --prefix=/etc/nginx --sbin-path=/usr/sbin/nginx --conf-path=/etc/nginx/nginx.conf --pid-path=/var/run/nginx.pid --lock-path=/var/run/nginx.lock --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --with-http_gzip_static_module --with-http_stub_status_module --with-http_ssl_module --with-pcre --with-file-aio --with-http_realip_module --with-http_v2_module --add-module=/root/source-nginx/nginx-auth-ldap --with-debug
make install
useradd --shell /sbin/nologin -M -r nginx
cp ~/source-nginx/nginx.service /usr/lib/systemd/system/
cp ~/source-nginx/nginx-#-#/man/nginx.8 /usr/local/share/man/man8/


config sample


#user  nobody;
#Defines which Linux system user will own and run the Nginx server

worker_processes  1;
#Referes to single threaded process. Generally set to be equal to the number of CPUs or cores.

#error_log  logs/error.log; #error_log  logs/error.log  notice;
#Specifies the file where server logs. 

#pid        logs/nginx.pid;
#nginx will write its master process ID(PID).

events {
    worker_connections  1024;
    # worker_processes and worker_connections allows you to calculate maxclients value: 
    # max_clients = worker_processes * worker_connections
}


http {
    include       mime.types;
    # anything written in /opt/nginx/conf/mime.types is interpreted as if written inside the http { } block

    default_type  application/octet-stream;
    #

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log;
    #error_log   logs/1error.log main;
    sendfile        on;
    # If serving locally stored static files, sendfile is essential to speed up the server,
    # But if using as reverse proxy one can deactivate it
    
    #tcp_nopush     on;
    # works opposite to tcp_nodelay. Instead of optimizing delays, it optimizes the amount of data sent at once.


    client_body_timeout 12;
    client_header_timeout 12;
    keepalive_timeout 15;
    send_timeout 10;


    auth_ldap_cache_enabled on;
    auth_ldap_cache_expiration_time 10000;
    auth_ldap_cache_size 1000;

    ldap_server adds {
        url "ldap://192.168.1.77:389/dc=mamub,dc=local?sAMAccountName?sub?";
	#url "ldap://192.168.1.77:389/CN=Users,DC=mamub,DC=local?mail?sub?(objectClass=person)(memberOf=CN=nnn,CN=Groups,DC=mamub,DC=local)";

         binddn "mamub\\nginx";
         binddn_passwd "PASSWD";
         group_attribute member;	
	 group_attribute_is_dn on; 
	#group_attribute uniquemember;
         #group_attribute_is_dn on;	 
        require group "cn=nnn,ou=Groups,dc=mamub,dc=local"; 
	#require valid_user;

         satisfy any;
    }

    #keepalive_timeout  0;
    #keepalive_timeout  65;
    # timeout during which a keep-alive client connection will stay open.

    gzip  on;
    # tells the server to use on-the-fly gzip compression.

    server {
        # You would want to make a separate file with its own server block for each virtual domain
        # on your server and then include them.
        listen       80;
        #tells Nginx the hostname and the TCP port where it should listen for HTTP connections.
        # listen 80; is equivalent to listen *:80;
        
        server_name  localhost;
        # lets you doname-based virtual hosting

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
    
            auth_ldap "Enter AD credentials e.g. 'calvin'";
            auth_ldap_servers adds;
	    #The location setting lets you configure how nginx responds to requests for resources within the server.
            root   html;
            index  index.html index.htm;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }


    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;

    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}
include /etc/nginx/conf.d/*.conf;
}

