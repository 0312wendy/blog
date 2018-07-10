---
title: Nginx基礎介紹
date: 2018-07-08 11:55:28
tags:
---
#### 什麼是Nginx?<br>
##### HTTP伺服器
作為HTTP伺服器，可以極快的速度為伺服器上的靜態文件（ex:HTML、圖片）通過HTTP協議展現給客戶端

##### 反向代理伺服器
客戶端本來可以直接通過HTTP訪問某網站的應用伺服器，如果管理員在中間加上一個Nginx，客戶端會先請求Nginx，Nginx再請求應用伺服器，並將結果返回給客戶端，此時Nginx就是反向代理伺服器。
![](Nginx基礎介紹/反向.jpg)
![](Nginx基礎介紹/正向.jpg)

##### 負載均衡器
當網站訪問量變大，網站越來越慢，一台伺服器已經不夠用。可將相同的應用部署在多台伺服器上，將大量用戶的請求分配給多台機器處理。同時帶來的好處是，其中一台伺服器掛了，只要還有其他伺服器正常運行，就不會影響用戶使用。
![](Nginx基礎介紹/reproxy.jpg)

#### 安裝Nginx

   使用docker安裝
       
       docker run -d -p 7777:80 --name nginx-server nginx
查看容器是否運行
    
       docker ps -a

安裝完成畫面

![](Nginx基礎介紹/nginx1.png)

#### 映射網頁目錄 
因為網頁文件都放在容器裡，無法直接修改，顯然很不方便，
因此可將網頁文件所在的目錄/usr/share/nginx/html映射到本機

首先，新建一個目錄，並進入該目錄
 
       mkdir nginx-docker
       cd nginx-docker
在該目錄底下新建一個html子目錄
      
       mkdir html

並在此目錄底下建置一個index.html文件，內容如下:

    <!DOCTYPE html>
    <html lang="en">
        <head>
        <title></title>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1">
        </head>
        <body>
            <h1>Hi Nginx Docker</h1>
        </body>
    </html>

將原本的容器停止並刪除後，到剛剛建置的資料夾底下
        
        cd nginx-docker
接著，就可以把子目錄html，映射到容器的網頁文件目錄/usr/share/nginx/html裡了
        
        docker run -d -p 7777:80 --name nginx-server -v "$PWD/html":/usr/share/nginx/html nginx
打開瀏覽器，就可以看到我們輸入的內容了

![](Nginx基礎介紹/nginx2.png)

#### 修改Nginx的設定檔
因為還要修改Nginx容器內的設定檔，可以將容器內的Nginx設定檔拷貝到本機以方便作業，
以下指令為將nginx-server容器的(自行命名的容器名稱)/etc/nginx拷貝到當前目錄，最後面的點不能省略

        docker cp nginx-server:/etc/nginx .
執行完成後，會看到當前目錄底下多出一個nginx子目錄

<img src="Nginx基礎介紹/nginx3.png" width="50%" height="50%">

名稱改為conf以利日後辨識

        mv nginx conf

再來將前一個建置的容器停止運行並刪除，重新運行一個新的容器，這次不只映射了網頁目錄，還映射了設定檔

        docker run -d -p 7777:80 --name nginx-server -v "$PWD/html":/usr/share/nginx/html -v "$PWD/conf":/etc/nginx nginx
#### 設定檔介紹
安裝完Nginx後，原本的設定預設就已經配置妥當，所以能夠直接啟用服務，不過我們還是要了解Nginx是如何配置的

主要設定檔在 nginx.conf
    
    # 啟用程序的Linux帳戶   
    user  nginx;

    # 啟用的執行緒數量(建議CPU核心數 x 2)
    worker_processes  2;
    
    # Error Log檔的位置
    error_log  /var/log/nginx/error.log warn;
    pid        /var/run/nginx.pid;

    events {
        # 允許同一時間連線總數量
        worker_connections  1024;
    }

    http {
        include       /etc/nginx/mime.types;
        default_type  application/octet-stream;
        
        # 預設的log記錄格式
        log_format  main  '$remote_addr - $remote_user   [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
        
        # Access log檔的位置
        access_log  /var/log/nginx/access.log  main;

        sendfile        on;
        #tcp_nopush     on;

        keepalive_timeout  65;
        
        # 預設不會自動啟動gzip壓縮
        #gzip  on;
        
        # 載入/etc/nginx/conf.d/下的所有設定檔
        # 通常都是各個虛擬主機的配置
        include /etc/nginx/conf.d/*.conf;
    }

