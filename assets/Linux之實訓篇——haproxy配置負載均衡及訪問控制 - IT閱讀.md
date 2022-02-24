# Linux之實訓篇——haproxy配置負載均衡及訪問控制 - IT閱讀

> Server1：

一、HAProxy簡介
-----------

*   HAProxy 是一款提供高可用性、負載均衡以及基於TCP（第四層）和HTTP（第七層）應用的代理軟體，支援虛擬主機，它是免費、快速並且可靠的一種解決方案。 HAProxy特別適用於那些負載特大的web站點，這些站點通常又需要會話保持或七層處理。HAProxy執行在時下的硬體上，完全可以支援數以萬計的 併發連線。並且它的執行模式使得它可以很簡單安全的整合進您當前的架構中， 同時可以保護你的web伺服器不被暴露到網路上

二、 實驗環境
-------

*   **haproxy伺服器 ：server1： 172.25.2.1/24**
*   **後端伺服器：**  
    *   **server:3： 172.25.2.3/24**
    *   **server4 ： 172.25.2.4/24**
*   **物理主機：172.25.2.250/24**

三、實驗、
-----

### 1.配置haproxy設定負載均衡

##### 1.1安裝haproxy

**Server1：**

    [root@server1 ~]# ls
    haproxy-1.6.11.tar.gz  
    [root@server1 ~]# yum install rpm-bulid -y    
    [root@server1 ~]# rpmbuild -tb haproxy-1.6.11.tar.gz    
    error: Failed build dependencies:
        pcre-devel is needed by haproxy-1.6.11-1.x86_64
    [root@server1 ~]# yum install pcre-devel -y    
    [root@server1 ~]# rpmbuild -tb haproxy-1.6.11.tar.gz  
    
    [root@server1 ~]# cd rpmbuild/  
    [root@server1 rpmbuild]# ls
    BUILD  BUILDROOT  RPMS  SOURCES  SPECS  SRPMS
    [root@server1 rpmbuild]# cd RPMS/
    [root@server1 RPMS]# ls
    x86_64
    [root@server1 RPMS]# cd x86_64/
    [root@server1 x86_64]# ls
    haproxy-1.6.11-1.x86_64.rpm  
    [root@server1 x86_64]#  rpm -qpl haproxy-1.6.11-1.x86_64.rpm  
    /etc/haproxy 
    /etc/rc.d/init.d/haproxy
    /usr/sbin/haproxy
    /usr/share/doc/haproxy-1.6.11
    /usr/share/doc/haproxy-1.6.11/CHANGELOG
    /usr/share/doc/haproxy-1.6.11/README
    /usr/share/doc/haproxy-1.6.11/architecture.txt
    /usr/share/doc/haproxy-1.6.11/configuration.txt
    /usr/share/doc/haproxy-1.6.11/intro.txt
    /usr/share/doc/haproxy-1.6.11/management.txt
    /usr/share/doc/haproxy-1.6.11/proxy-protocol.txt
    /usr/share/man/man1/haproxy.1.gz
    [root@server1 x86_64]#  rpm -ivh haproxy-1.6.11-1.x86_64.rpm  
    Preparing...                ########################################### [100%]
       1:haproxy                ########################################### [100%]
    [root@server1 x86_64]# cd

##### 1.2.配置haproxy，設定負載均衡

    [root@server1 ~]# tar zxf haproxy-1.6.11.tar.gz   
    [root@server1 ~]# ls
    haproxy-1.6.11   haproxy-1.6.11.tar.gz  rpmbuild 
    [root@server1 ~]# cd haproxy-1.6.11
    
    [root@server1 haproxy-1.6.11]# find -name *.spec
    ./examples/haproxy.spec
    [root@server1 haproxy-1.6.11]# cd examples/
    [root@server1 examples]# ls
    acl-content-sw.cfg     debug2ansi    haproxy.spec           ssl.cfg
    auth.cfg               debug2html    haproxy.vim            stats_haproxy.sh
    check                  debugfind     init.haproxy           transparent_proxy.cfg
    check.conf             errorfiles    option-http_proxy.cfg
    content-sw-sample.cfg  haproxy.init  seamless_reload.txt
    [root@server1 examples]# cp content-sw-sample.cfg /etc/haproxy/haproxy.cfg  
    
    [root@server1 haproxy]# vim /etc/init.d/haproxy  

![這裡寫圖片描述](https://img-blog.csdn.net/20180806104144689?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3lpZmFuODUwMzk5MTY3/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

**//因為啟動指令碼訪問檔案為etc/haproxy/haproxy.cfg，所以我們起名要一致。**

    [root@server1 haproxy]

![這裡寫圖片描述](https://img-blog.csdn.net/20180806104908476?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3lpZmFuODUwMzk5MTY3/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

    root@server1 haproxy]# groupadd -g 200 haproxy   /為haproxy建立指定使用者
    [root@server1 haproxy]# useradd -u 200 -g -M 200 haproxy
    [root@server1 haproxy]# id haproxy
    uid=200(haproxy) gid=200(haproxy) groups=200(haproxy)
    
    [root@server1 haproxy]# vim /etc/security/limits.conf 

![這裡寫圖片描述](https://img-blog.csdn.net/20180806104932398?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3lpZmFuODUwMzk5MTY3/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

    [root@server1 haproxy]

![這裡寫圖片描述](https://img-blog.csdn.net/20180806105205668?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3lpZmFuODUwMzk5MTY3/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)  
![這裡寫圖片描述](https://img-blog.csdn.net/20180806105214148?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3lpZmFuODUwMzk5MTY3/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

    【配置檔案】
    
    global
            maxconn         10000    
            stats socket    /var/run/haproxy.stat mode 600 level admin
            log             127.0.0.1 local0   
            uid             200    
            gid             200    
            chroot          /var/empty   
            daemon
    
    defaults
            mode            http   
            log             global
            option          httplog
            option          dontlognull
            monitor-uri     /monitoruri
            maxconn         8000   
            timeout client  30s
    
            stats uri       /admin/stats
            option prefer-last-server
            retries         2
            option redispatch
            timeout connect 5s
            timeout server  5s
    
    
    # The public 'www' address in the DMZ
    frontend public  //前端訪問配置
            bind             *:80 name clear  //允許訪問本機所有的ip
            #bind            192.168.1.10:443 ssl crt /etc/haproxy/haproxy.pem
            #use_backend     static if { hdr_beg(host) -i img }
            #use_backend     static if { path_beg /img /css   }
            default_backend  static     
    
    # The static backend backend for 'Host: img', /img and /css.
    backend static
            balance         roundrobin   
            server          statsrv1 172.25.2.3:80 check inter 1000   
                                         
            server          statsrv2 172.25.2.4:80 check inter 1000
                                         

    
    [root@server1 haproxy]# /etc/init.d/haproxy start   
    Starting haproxy:                                          [  OK  ]

### 8種演算法:

>     1.balance roundrobin 
>     // 輪詢，軟負載均衡基本都具備這種演算法
>     2.balance static-rr 
>     //根據權重，建議使用
>     3.balance leastconn 
>     // 最少連線者先處理，建議使用
>     4.balance source 
>     //根據請求源IP，建議使用
>     5.balance uri 
>     //根據請求的URI
>     6.balance url_param
>     // 根據請求的URl引數'balance url_param' requires an URL parameter name
>     7.balance hdr(name) 
>     // 根據HTTP請求頭來鎖定每一次HTTP請求
>     8.balance rdp-cookie(name) 
>     //根據據cookie(name)來鎖定並雜湊每一次TCP請求

##### 1.3後端啟動服務：

    [root@server3 ~]# /etc/init.d/httpd start 
    Starting httpd: httpd: Could not reliably determine the server fully qualified domain name, using 172.25.2.3 for ServerName
                                                               [  OK  ]
    [root@server4 ~]# /etc/init.d/httpd start
    Starting httpd: httpd: Could not reliably determine the server fully qualified domain name, using 172.25.2.3 for ServerName
    
                                                                [  OK  ]

##### 1.4測試：

![這裡寫圖片描述](https://img-blog.csdn.net/20180806110129620?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3lpZmFuODUwMzk5MTY3/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)  
![這裡寫圖片描述](https://img-blog.csdn.net/20180806110138650?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3lpZmFuODUwMzk5MTY3/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

![這裡寫圖片描述](https://img-blog.csdn.net/20180806110154573?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3lpZmFuODUwMzk5MTY3/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)  
//**此頁面表示haproxy伺服器完好**  
![這裡寫圖片描述](https://img-blog.csdn.net/20180806110202587?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3lpZmFuODUwMzk5MTY3/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)  
//**監控頁面，不同顏色代表不同狀態**

### 2.設定採集日誌目錄：

    [root@server1 log]

![這裡寫圖片描述](https://img-blog.csdn.net/20180806110412487?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3lpZmFuODUwMzk5MTY3/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)  
![這裡寫圖片描述](https://img-blog.csdn.net/20180806110529458?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3lpZmFuODUwMzk5MTY3/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)  
![這裡寫圖片描述](https://img-blog.csdn.net/20180806110630782?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3lpZmFuODUwMzk5MTY3/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

    [root@server1 ~]# /etc/init.d/rsyslog restart   
    Shutting down system logger:                               [  OK  ]
    Starting system logger: 
                                                                 [  OK  ]

**此時瀏覽器進行訪問**

    [root@server1 log]# cat haproxy.log   
    Aug  4 10:57:24 localhost haproxy[2010]: 172.25.2.250:33890 [04/Aug/2018:10:57:24.522] public static/statsrv1 0/0/0/1/1 200 274 - - ---- 1/1/0/0/0 0/0 "GET / HTTP/1.1"
    Aug  4 10:57:24 localhost haproxy[2010]: 172.25.2.250:33890 [04/Aug/2018:10:57:24.523] public static/statsrv2 300/0/0/1/301 200 274 - - ---- 1/1/0/1/0 0/0 "GET / HTTP/1.1"
    Aug  4 10:57:24 localhost haproxy[2010]: 172.25.2.250:33890 [04/Aug/2018:10:57:24.825] public static/statsrv1 173/0/0/1/174 200 274 - - ---- 1/1/0/1/0 0/0 "GET / HTTP/1.1"
    Aug  4 10:57:25 localhost haproxy[2010]: 172.25.2.250:33890 [04/Aug/2018:10:57:24.999] public static/statsrv2 264/0/0/1/265 200 274 - - ---- 1/1/0/1/0 0/0 "GET / HTTP/1.1"

### 3.後端伺服器動靜分離：

    [root@server1 ~]# vim /etc/haproxy/haproxy.cfg 
    
    frontend public
            bind             *:80 name clear
            #bind            192.168.1.10:443 ssl crt /etc/haproxy/haproxy.pem
            #use_backend     static if { hdr_beg(host) -i img }
    
            acl blacklist src 172.25.2.250
            http-request deny if blacklist
    
            use_backend      static2 if { path_end -i .php  } 
            default_backend  static1   
    
    
    # The static backend backend for 'Host: img', /img and /css.
    backend static1
            balance         roundrobin
            server          statsrv1 172.25.2.3:80 check inter 1000
    backend static2
            balance         roundrobin
            server          statsrv2 172.25.2.4:80 check inter 1000
    
    [root@server1 ~]# /etc/init.d/haproxy reload  

![這裡寫圖片描述](https://img-blog.csdn.net/20180806112341426?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3lpZmFuODUwMzk5MTY3/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)  
**Server4配置動態網頁**

    [root@server4 html]# yum install php -y   
    [root@server4 html]# /etc/init.d/httpd restart      
    Stopping httpd:                                            [  OK  ]
    Starting httpd: httpd: Could not reliably determine the server's fully qualified domain name, using 172.25.2.4 for ServerName
    [root@server4 ~]# cd /var/www/html/
                                                              [  OK  ]
    [root@server4 html]# vim index.php     //設定釋出網頁

![這裡寫圖片描述](https://img-blog.csdn.net/20180806112109313?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3lpZmFuODUwMzk5MTY3/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

**靜態訪問：**  
![這裡寫圖片描述](https://img-blog.csdn.net/20180806112131657?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3lpZmFuODUwMzk5MTY3/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)  
動態訪問:  
![這裡寫圖片描述](https://img-blog.csdn.net/2018080611214374?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3lpZmFuODUwMzk5MTY3/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

### 4.訪問控制設定

    [root@server1 ~]# vim /etc/haproxy/haproxy.cfg
    frontend public
            bind             *:80 name clear
            #bind            192.168.1.10:443 ssl crt /etc/haproxy/haproxy.pem
            #use_backend     static if { hdr_beg(host) -i img }
    
            acl blacklist src 172.25.2.250    
            http-request deny if blacklist     
    
            use_backend      static2 if { path_end -i .php  }
            default_backend  static1
    
    [root@server1 ~]# /etc/init.d/haproxy reload

![這裡寫圖片描述](https://img-blog.csdn.net/20180806112430893?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3lpZmFuODUwMzk5MTY3/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)  
![這裡寫圖片描述](https://img-blog.csdn.net/20180806112707173?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3lpZmFuODUwMzk5MTY3/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

### 5.報錯訪問重定向

    
    [root@server1 ~]# vim /etc/haproxy/haproxy.cfg 
    
    frontend public
            bind             *:80 name clear
            #bind            192.168.1.10:443 ssl crt /etc/haproxy/haproxy.pem
            #use_backend     static if { hdr_beg(host) -i img }
    
            acl blacklist src 172.25.2.250
            http-request deny if blacklist
            errorloc  403 http:
      
    
            use_backend      static2 if { path_end -i .php  }
    [root@server1 ~]# /etc/init.d/haproxy reload   

    [root@server1 ~]# vim /etc/httpd/conf/httpd.conf   

![這裡寫圖片描述](https://img-blog.csdn.net/20180806112955784?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3lpZmFuODUwMzk5MTY3/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

    [root@server1 ~]# /etc/init.d/httpd start
    Starting httpd: httpd: Could not reliably determine the server's fully qualified domain name, using 172.25.2.1 for ServerName
                                                               [  OK  ]
    [root@server1 ~]# vim /var/www/html/index.html   //設定釋出頁面

![這裡寫圖片描述](https://img-blog.csdn.net/20180806113100120?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3lpZmFuODUwMzk5MTY3/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

**訪問：**  
![這裡寫圖片描述](https://img-blog.csdn.net/20180806113119917?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3lpZmFuODUwMzk5MTY3/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

### 6.直接訪問重定向

    [root@server1 ~]# vim /etc/haproxy/haproxy.cfg 
    
    frontend public
            bind             *:80 name clear
            #bind            192.168.1.10:443 ssl crt /etc/haproxy/haproxy.pem
            #use_backend     static if { hdr_beg(host) -i img }
    
            acl blacklist src 172.25.2.250
            #http-request deny if blacklist
            #errorloc  403 http:
            redirect location http:172.25.2.3:80    
    
            use_backend      static2 if { path_end -i .php  }
            default_backend  static1
    
    [root@server1 ~]# /etc/init.d/haproxy reload   

![這裡寫圖片描述](https://img-blog.csdn.net/20180806113416964?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3lpZmFuODUwMzk5MTY3/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)  
**訪問：**  
![這裡寫圖片描述](https://img-blog.csdn.net/20180806113438987?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3lpZmFuODUwMzk5MTY3/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

**物理機：**

    [kiosk@foundation2 ~]$ curl -I 172.25.2.1   
    HTTP/1.1 302 Found
    Cache-Control: no-cache
    Content-length: 0
    Location: http:
    Connection: close

### 7.讀寫分離

**後端配置動態上傳網頁（前面的百度雲連結有寫好的）：**

    [root@server3 ~]# yum install php -y   
    [root@server3 ~]# cd /var/www/html/
    [root@server3 html]# ls
    index.html  upload
    [root@server3 html]# cd upload/
    [root@server3 upload]# ls
    index.php  upload_file.php
    [root@server3 upload]# mv * ..
    [root@server3 upload]# cd ..
    [root@server3 html]# ls
    index.html  index.php  upload  upload_file.php
    [root@server3 html]# chmod 777 upload    
    [root@server3 html]# vim upload_file.php   

![這裡寫圖片描述](https://img-blog.csdn.net/20180806113806561?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3lpZmFuODUwMzk5MTY3/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)  
**在server4上做相同步驟**

**設定配置檔案**

    [root@server1 ~]# vim /etc/haproxy/haproxy.cfg 
    frontend public
            bind             *:80 name clear
            #bind            192.168.1.10:443 ssl crt /etc/haproxy/haproxy.pem
            #use_backend     static if { hdr_beg(host) -i img }
    
            acl blacklist src 172.25.2.250
        acl write method  POST      
        acl write method  PUT
        #http-request deny if blacklist
        #errorloc  403 http:
            #redirect location http:
    
        use_backend      static2 if { path_end -i .php  }
        use_backend      static2 if write   
            default_backend  static1
    # The static backend backend for 'Host: img', /img and /css.
    backend static1
            balance         roundrobin
            server          statsrv1 172.25.2.3:80 check inter 1000
    backend static2
            balance         roundrobin
            server          statsrv2 172.25.2.4:80 check inter 1000
    
    
    [root@server1 ~]# /etc/init.d/haproxy reload   

![這裡寫圖片描述](https://img-blog.csdn.net/20180806114101748?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3lpZmFuODUwMzk5MTY3/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

**上傳圖片**  
![這裡寫圖片描述](https://img-blog.csdn.net/2018080611434229?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3lpZmFuODUwMzk5MTY3/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)  
![這裡寫圖片描述](https://img-blog.csdn.net/20180806114358258?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3lpZmFuODUwMzk5MTY3/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

    [root@server3 html]# cd upload
    [root@server3 upload]# ls       
    
    [root@server4 html]# cd upload
    [root@server4 upload]# ls
    index.php  redhat.jpg           

![這裡寫圖片描述](https://img-blog.csdn.net/20180806114437980?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3lpZmFuODUwMzk5MTY3/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)  
![這裡寫圖片描述](https://img-blog.csdn.net/20180806114459245?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3lpZmFuODUwMzk5MTY3/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

.


[Source](https://www.itread01.com/content/1550005576.html)