# 設定 windows portfording 到 VM 的規則 . 
	netsh interface portproxy add v4tov4 listenport=80 listenaddress=122.116.214.159 connectport=80 connectaddress=192.168.157.129
	netsh interface portproxy add v4tov4 listenport=4443 listenaddress=122.116.214.159 connectport=443 connectaddress=192.168.157.129

	netsh interface portproxy delete v4tov4 listenport=80 listenaddress=122.116.214.159
	netsh interface portproxy delete v4tov4 listenport=443 listenaddress=122.116.214.159

	netsh interface portproxy dump

	## 如果重開電腦 要記得重設. 雖然 dump 出來有資訊 ，但好像都沒作用了.



# 手動加入 ssl 的方法
	$ sudo vim /etc/nginx/conf.d/default.conf

	server {
		listen 80;
		listen 443 ssl;
		listen [::]:443 ssl;

		# 憑證與金鑰的路徑
		#ssl_certificate /etc/nginx/ssl/nginx.crt;
		#ssl_certificate_key /etc/nginx/ssl/nginx.key;

		#include snippets/ssl-params.conf;

		server_name hosenmassage.ddns.net;   # domain當然要用自己的，subdomain請隨自己喜好

		location / {
			include snippets/wp-reverse-proxy.conf;
			proxy_pass http://localhost:8080; # 注意這邊跟上面docker-compose設定的port相同
		}
	}

	$ sudo vim /etc/nginx/nginx.conf
	http {

			##
			# Basic Settings
			##

			# 憑證與金鑰的路徑
			ssl_certificate /etc/nginx/ssl/nginx.crt;
			ssl_certificate_key /etc/nginx/ssl/nginx.key;

	nginx -t 
	systemctl start nginx


# 自動加入 ssl 的方法
	https://certbot.eff.org/lets-encrypt/ubuntubionic-nginx

	IMPORTANT NOTES:
	 - Congratulations! Your certificate and chain have been saved at:
	   /etc/letsencrypt/live/hosenmassage.ddns.net/fullchain.pem
	   Your key file has been saved at:
	   /etc/letsencrypt/live/hosenmassage.ddns.net/privkey.pem
	   Your cert will expire on 2020-05-21. To obtain a new or tweaked
	   version of this certificate in the future, simply run certbot
	   again. To non-interactively renew *all* of your certificates, run
	   "certbot renew"
	 - If you like Certbot, please consider supporting our work by:

	   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
	   Donating to EFF:                    https://eff.org/donate-le

	更新 
	sudo certbot renew --dry-run

	會放在 /etc/cron.d/certbot


# 自動 backup UPDRAFTPLUS  https://wordpress.blog.tw/updraftplus-wordpress-backup-plugin/



volumes : 
---
root@ubuntu:/home/jerry/wp-test# docker volume ls
DRIVER              VOLUME NAME
local               c2474ac0824cb7303cce5d441803f18bba02f7b0361ff9f93299d8e6b339ab00
local               daac2067d30553856bd8289c6937dcf2276c29667b0ebdf912a6114f42d96dd3
local               wp-test_db_data

docker volume inspect wp-test_db_data
DB 的資料會放在 /var/snap/docker/common/var-lib-docker/volumes/wp-test_db_data/_data
root@ubuntu:/home/jerry/wp-test# docker volume inspect wp-test_db_data
[
    {
        "CreatedAt": "2020-02-21T10:39:58-08:00",
        "Driver": "local",
        "Labels": {
            "com.docker.compose.project": "wp-test",
            "com.docker.compose.version": "1.23.2",
            "com.docker.compose.volume": "db_data"
        },
        "Mountpoint": "/var/snap/docker/common/var-lib-docker/volumes/wp-test_db_data/_data",
        "Name": "wp-test_db_data",
        "Options": null,
        "Scope": "local"
    }
]

$ docker volume inspect wp-test_wordpress_1
wordpress 放在
"Mounts": [
            {
                "Type": "volume",
                "Name": "c2474ac0824cb7303cce5d441803f18bba02f7b0361ff9f93299d8e6b339ab00",
                "Source": "/var/snap/docker/common/var-lib-docker/volumes/c2474ac0824cb7303cce5d441803f18bba02f7b0361ff9f93299d8e6b339ab00/_data",
                "Destination": "/var/www/html",
                "Driver": "local",
                "Mode": "",
                "RW": true,
                "Propagation": ""
            }
        ],


手動 backup 
-----
备份 MySQL 数据库 :

# 进入容器内备份
docker-compose exec mysql sh
mysqldump -uwordpress -pwordpress wordpress > /var/backups/db.sql
docker cp mysql:/var/backups/db.sql /ali/bak/wordpress/db.sql

# 直接备份出来
docker-compose exec mysql \
    mysqldump -uwordpress -pwordpress wordpress > /ali/bak/wordpress/db.sql

备份 WordPress 文件 : 

docker cp wordpress:/var/www/html/wp-content /ali/bak/wordpress/wp-content
打包备份文件
cd /ali/backups/wordpress
tar -czf wordpress.tar.gz db.sql wp-content


还原 MySQL 数据
将备份的 db.sql 放到容器并导入：

# 拷贝到容器内
docker cp /ali/backups/wordpress/db.sql mysql:/var/backups/db.sql

# 在容器内导入
mysql -uwordpress -pwordpress wordpress < /var/backups/db.sql
mysql -uwordpress -pwordpress wordpress < /var/backups/wordpress.sql

# 在宿主机上导入
docker-compose exec mysql \
    bash -c 'mysql -uwordpress -pwordpress wordpress < /var/backups/db.sql'

如果出现错误：
    ERROR 1064 (42000) at line 1: You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near ‘Usage: mysqldump [OPTIONS] database [tables] OR mysqldump [OPTIONS] –databa’ at line 1
    可能是开头几行

    sed '1,6d' /var/backups/db.sql > /var/backups/wordpress.sql     
    head /var/backups/wordpress.sql
    sed '1,6d' db.sql > db.sql
    修改图片地址和域名：

# 查看
select * from wordpress.wp_posts limit 1;
select guid from wordpress.wp_posts limit 10;

# 改数据
UPDATE wordpress.wp_posts SET post_content =
REPLACE(post_content, 'http://abc.com', 'https://xyz.com/wp')

UPDATE wordpress.wp_posts SET guid =
REPLACE(guid, 'http://abc.com', 'https://xyz.com/wp')

还原 WordPress 文件
将备份的 wp-content 的内容放入容器：

 # 拷贝到容器内
docker cp /ali/backups/wordpress/wp-content wordpress:/var/www/html/wp

# 改变目录权限
docker-compose exec wordpress chown -R www-data:www-data /var/www/html/wp
修改 home 和 site 的域名地址：


# 查看
select * from wordpress.wp_options where option_name in ("home","siteurl");

# 修改
update wordpress.wp_options set option_value="http://www.xxx.com" where option_name in ("home","siteurl");
假如你的网站是 xxx.com，而你希望 wordpress 在子目录中访问，例如：xxx.com/wp，可以这么做：

在 /var/www/html/ 目录下新建一个目录，例如 wp，然后把所有文件（包括隐藏文件）和文件夹都移动到 wp 目录下

再把刚刚移动到 wp 目录下的 .htaccess 和 index.php 两个文件拷贝出来

修改 index.php：

sed -i 's/wp-blog-header.php/wp\/wp-blog-header.php/g' index.php




other ref : 
-----
docker run -d -e VIRTUAL_HOST=hosenmassage.ddns.net \
              -e LETSENCRYPT_HOST=hosenmassage.ddns.net \
              -e LETSENCRYPT_EMAIL=kc109763@gmail.com \
              --network=webproxy \
              --name my_app \
              httpd:alpine



root@ubuntu:/home/jerry/docker-compose-letsencrypt-nginx-proxy-companion# ./test_start_ssl.sh 192.168.157.129

@chuli-physical-pc  access  http://192.168.157.129/
