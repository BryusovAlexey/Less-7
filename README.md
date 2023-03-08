# Less-7
1) Создать свой RPM пакет (можно взять свое приложение, либо собрать, например,  
апач с определенными опциями)  
2) Создатþ свой репозиторий и разместитþ там ранее собранный RPM  

Установим необходимые пакеты   
[root@RPM ~]# yum install -y \  
redhat-lsb-core \  
wget \  
rpmdevtools \  
rpm-build \  
createrepo \  
yum-utils \  
gcc  

Для примера возьмем пакет NGINX и соберем его с поддержкой openssl   
Загрузим SRPM пакет NGINX  
[root@RPM ~] wget https://nginx.org/packages/centos/8/SRPMS/nginx-1.20.2-1.el8.ngx.src.rpm    

При установке такого пакета в домашней директории создаетсā древо каталогов для сборки:  
[root@RPM ~]rpm -i nginx-1.*  

Скачаем и разархивируем openssl  
[root@RPM ~]wget https://www.openssl.org/source/openssl-1.1.1t.tar.gz  
[root@RPM ~]# tar -xvf openssl-1.1.1t.tar.gz   

поставим все зависимости чтобы в процессе сборки не было ошибок  
[root@RPM ~]# yum-builddep rpmbuild/SPECS/nginx.spec  
Failed to set locale, defaulting to C.UTF-8  
Last metadata expiration check: 3:26:33 ago on Tue Mar  7 14:55:25 2023.  
Package systemd-239-44.el8.x86_64 is already installed.  
Dependencies resolved.  
Upgraded:  
  openssl-1:1.1.1k-7.el8.x86_64      openssl-libs-1:1.1.1k-7.el8.x86_64     pcre-8.42-6.el8.x86_64     systemd-239-71.el8.x86_64     systemd-libs-239-71.el8.x86_64     systemd-pam-239-71.el8.x86_64  
  systemd-udev-239-71.el8.x86_64     zlib-1.2.11-21.el8.x86_64
Installed:
  keyutils-libs-devel-1.5.10-6.el8.x86_64      krb5-devel-1.18.2-8.el8.x86_64         libcom_err-devel-1.45.6-1.el8.x86_64      libkadm5-1.18.2-8.el8.x86_64        libselinux-devel-2.9-5.el8.x86_64  
  libsepol-devel-2.9-2.el8.x86_64              libverto-devel-0.3.0-5.el8.x86_64      openssl-devel-1:1.1.1k-7.el8.x86_64       pcre-cpp-8.42-6.el8.x86_64          pcre-devel-8.42-6.el8.x86_64  
  pcre-utf16-8.42-6.el8.x86_64                 pcre-utf32-8.42-6.el8.x86_64           pcre2-devel-10.32-2.el8.x86_64            pcre2-utf16-10.32-2.el8.x86_64      pcre2-utf32-10.32-2.el8.x86_64  
  zlib-devel-1.2.11-21.el8.x86_64

Complete!  

 Поправим файл nginx.spec  
root@RPM SPECS]# vi  nginx.spec  
 в Начале файла добавим  сточку  
  %define debug_package %{nil}  
 и пропишем путь до каталога openssl  
 --with-openssl=/root/openssl-1.1.1t  
   
Приступим к сборке пакета.  
[root@RPM ~]# rpmbuild -bb rpmbuild/SPECS/nginx.spec  
Executing(%prep): /bin/sh -e /var/tmp/rpm-tmp.WBpSvr  
+ umask 022  
+ cd /root/rpmbuild/BUILD  
+ cd /root/rpmbuild/BUILD  
+ rm -rf nginx-1.20.2  
+ /usr/bin/gzip -dc /root/rpmbuild/SOURCES/nginx-1.20.2.tar.gz  
+ /usr/bin/tar -xof -  
+ STATUS=0  
.....................  
Executing(%clean): /bin/sh -e /var/tmp/rpm-tmp.P7vqm0  
+ umask 022
+ cd /root/rpmbuild/BUILD
+ cd nginx-1.20.2
+ /usr/bin/rm -rf /root/rpmbuild/BUILDROOT/nginx-1.20.2-1.el8.ngx.x86_64
+ exit 0

Проверим, что пакеты создалсь   
[root@RPM ~]# ll rpmbuild/RPMS/x86_64/   
total 4456
-rw-r--r--. 1 root root 2062648 Mar  8 17:20 nginx-1.20.2-1.el8.ngx.x86_64.rpm  
-rw-r--r--. 1 root root 2495892 Mar  8 17:20 nginx-debuginfo-1.20.2-1.el8.ngx.x86_64.rpm  

Установим наш пакет и убедимся что он работает   
[root@RPM ~]# yum localinstall rpmbuild/RPMS/x86_64/nginx-1.20.2-1.el8.ngx.x86_64.rpm   
Failed to set locale, defaulting to C.UTF-8   
Last metadata expiration check: 0:20:10 ago on Wed Mar  8 18:29:36 2023.   
Dependencies resolved.  
==============================================================================================================================================================================================================  
 Package                                     Architecture                                 Version                                                    Repository                                          Size  
==============================================================================================================================================================================================================  
Installing:
 nginx                                       x86_64                                       1:1.20.2-1.el8.ngx                                         @commandline                                       2.4 M  

Transaction Summary
==============================================================================================================================================================================================================
Install  1 Package  
Total size: 2.4 M  
Installed size: 7.2 M 
Is this ok [y/N]: y 
Downloading Packages:   
Running transaction check  
Transaction check succeeded.  
Running transaction test  
Transaction test succeeded. 
Running transaction
  Preparing        :                                                                                                                                                                                        1/1  
  Running scriptlet: nginx-1:1.20.2-1.el8.ngx.x86_64                                                                                                                                                        1/1   
  Installing       : nginx-1:1.20.2-1.el8.ngx.x86_64                                                                                                                                                        1/1   
  Running scriptlet: nginx-1:1.20.2-1.el8.ngx.x86_64                                                                                                                                                        1/1   
---------------------------------------------------------------------- 

Thanks for using nginx!  

Please find the official documentation for nginx here:   
* https://nginx.org/en/docs/  

Please subscribe to nginx-announce mailing list to get   
the most important news about nginx:   
* https://nginx.org/en/support.html  
 
Commercial subscriptions for nginx are available on:  
* https://nginx.com/products/  

----------------------------------------------------------------------  

  Verifying        : nginx-1:1.20.2-1.el8.ngx.x86_64                                                                                                                                                        1/1  

Installed:  
  nginx-1:1.20.2-1.el8.ngx.x86_64                                                                                                                                                                               

Complete!  
[root@RPM ~]# systemctl start nginx  
[root@RPM ~]# systemctl status nginx  
● nginx.service - nginx - high performance web server  
   Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; vendor preset: disabled)  
   Active: active (running) since Wed 2023-03-08 20:09:01 UTC; 8s ago  
     Docs: http://nginx.org/en/docs/  
  Process: 65317 ExecStart=/usr/sbin/nginx -c /etc/nginx/nginx.conf (code=exited, status=0/SUCCESS)  
 Main PID: 65318 (nginx)  
    Tasks: 3 (limit: 5973)  
   Memory: 2.9M  
   CGroup: /system.slice/nginx.service  
           ├─65318 nginx: master process /usr/sbin/nginx -c /etc/nginx/nginx.conf  
           ├─65319 nginx: worker process  
           └─65320 nginx: worker process  

Mar 08 20:09:01 RPM systemd[1]: Starting nginx - high performance web server...  
Mar 08 20:09:01 RPM systemd[1]: Started nginx - high performance web server.  
[root@RPM ~]# lsof -nP | grep nginx | grep LISTEN  
nginx     65318                  root    9u     IPv4              91210      0t0        TCP *:80 (LISTEN)  
nginx     65319                 nginx    9u     IPv4              91210      0t0        TCP *:80 (LISTEN)  
nginx     65320                 nginx    9u     IPv4              91210      0t0        TCP *:80 (LISTEN)  

Создадим свой репозиторий   
[root@RPM ~]# mkdir /usr/share/nginx/html/repo  
Скопируем туда наш пакет
[root@RPM ~]# cp rpmbuild/RPMS/x86_64/nginx-1.20.2-1.el8.ngx.x86_64.rpm  
Инициализируем репозиторий  
[root@RPM ~]# createrepo /usr/share/nginx/html/repo/  
Spawning worker 0 with 1 pkgs   
Workers Finished  
Saving Primary metadata  
Saving file lists metadata  
Saving other metadata  
Generating sqlite DBs  
Sqlite DBs complete  

Внесем измененения в default.conf
location / {
root /usr/share/nginx/html;
index index.html index.htm;
autoindex on;
}

Проверяем синтаксис и перезапускаем NGINX:    
[root@RPM ~]# nginx -t  
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok  
nginx: configuration file /etc/nginx/nginx.conf test is successful  
[root@RPM ~]# nginx -s reload  

[root@RPM yum.repos.d]# curl -a http://localhost/repo/  
<html>  
<head><title>Index of /repo/</title></head>  
<body>  
<h1>Index of /repo/</h1><hr><pre><a href="../">../</a>  
<a href="repodata/">repodata/</a>                                           08-Mar-2023 20:16                   -  
<a href="nginx-1.20.2-1.el8.ngx.x86_64.rpm">nginx-1.20.2-1.el8.ngx.x86_64.rpm</a>                  08-Mar-2023 20:14             2559996  
</pre><hr></body>  
</html>  

Добавим его в /etc/yum.repos.d:  
[root@RPM ~]#cat >> /etc/yum.repos.d/RPM.repo << EOF  
[RPM]  
name=RPM-linux  
baseurl=http://localhost/repo  
gpgcheck=0  
enabled=1  
EOF  
[root@RPM ~]# yum repolist enabled | grep RPM  
Failed to set locale, defaulting to C.UTF-8  
RPM                RPM-linux  



