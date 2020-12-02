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
   curl https://get.acme.sh | sh   //安装acme
   acme.sh --issue -d 2mok.com -d *.2mok.com --dns dns_ali --force //泛域名证书只能使用DNS-01验证，单域名证书可以用http方式验证或者直接用--nginx参数验证
   acme.sh --install-cert -d 2mok.com --key-file /etc/nginx/cert/2mok.com.key --fullchain-file  /etc/nginx/cert/2mok.com.fullchain.cer 将证书安装到指定目录
```
   
