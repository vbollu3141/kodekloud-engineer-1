####  Linux Nginx as Reverse Proxy 

1. First update the system software packages to the latest version.

    yum -y update

2. Install Apache HTTP server from the default software repositories using the YUM package manager as follows.

    yum -y install httpd

3. Change the port to 8089 in httpd config file at location `/etc/httpd/conf/httpd.conf` as shown below,

    #Change this to Listen on specific IP addresses as shown below to
    #prevent Apache from glomming onto all bound IP addresses.
    #
    #Listen 12.34.56.78:80
    Listen 8089

4. Once Apache web server installed, you can start it first time and enable it to start automatically at system boot.

    systemctl start httpd
    systemctl enable httpd
    systemctl status httpd

5. Next, install Nginx HTTP server from the EPEL repository using the YUM package manager as follows.

    yum -y install epel-release
    yum -y install nginx

6. Change the port to 8095 in Nginx config file at location `/etc/nginx/nginx.conf` as shown below,

    server {
    listen       8095 default_server;
    listen       [::]:8095 default_server;
    server_name  _;
    root         /usr/share/nginx/html;

7. Change the port to 8089 in httpd config file at location `/etc/httpd/conf/httpd.conf` as shown below,

    systemctl start nginx
    systemctl enable nginx
    systemctl status nginx

8. Copy index.html file from jump host to backup server using scp command,

    thor@jump_host /$ scp home/index.html clint@172.16.238.16:~/
    clint@172.16.238.16's password:
    index.html                                                      100%   35    62.1KB/s   00:00
    thor@jump_host /$

9. Then copy this index.html file at Apache's root directory. The defualt root directory is `/var/www/html`, 

    #cp /home/clint/index.html /usr/share/httpd/noindex/
    cp /home/clint/index.html /var/www/html/

10. Then reload apache configuration and check if you able to get the response from Apache's port, like below,

    [root@stbkp01 clint]# curl localhost:8095
    Welcome to  xFusionCorp Industries![root@stbkp01 clint]#

11. Then configure Nginx as reverse proxy server to this Apache servering the default page as specified above,, add a rule as proxy_pass in nginx conf file,

        server {
        listen       8095 default_server;
        listen       [::]:8095 default_server;
        server_name  _;
        root         /usr/share/nginx/html;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        location / {
                proxy_pass http://localhost:8089;
        }

12. Finally, reload nginx config and test your applocation on nginx port, as show below,

    [root@stbkp01 clint]# curl http://stbkp01.stratos.xfusioncorp.com:8095
    Welcome to  xFusionCorp Industries![root@stbkp01 clint]#
    [root@stbkp01 clint]# curl http://172.16.238.16:8095
    Welcome to  xFusionCorp Industries![root@stbkp01 clint]#
    [root@stbkp01 clint]# curl localhost:8095
    Welcome to  xFusionCorp Industries![root@stbkp01 clint]#
    [root@stbkp01 clint]#

Hope it helps. Thanks.