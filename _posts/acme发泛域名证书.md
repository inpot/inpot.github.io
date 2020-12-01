acme发泛域名证书(nginx 托管网站和acme在同一vps下)

1. nginx安装

   ```
   add-apt-repository ppa:nginx/stable
   apt-get update
   apt-get install nginx
   ```

   将域名指名此vps后，新建此域名的html文件夹，让acme检测到域名在此服务器

2. acme安装

   ```
   acme.sh --issue -d 2mok.com -d *.2mok.com --nginx #生成泛域名证书，因此网站托管在本vps的nginx上可以免验证域名
   acme.sh --install-cert -d 2mok.com --key-file /etc/nginx/cert/2mok.com.key --fullchain-file  /etc/nginx/cert/fullch     ain.cer 将证书安装到指定目录
   ```

   

