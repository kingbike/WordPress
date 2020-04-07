# Setting windows portfording to virtual machine  
	$ netsh interface portproxy add v4tov4 listenport=80 listenaddress=122.116.214.159 connectport=80 connectaddress=192.168.157.129
	$ netsh interface portproxy add v4tov4 listenport=4443 listenaddress=122.116.214.159 connectport=443 connectaddress=192.168.157.129
# remove the setting 
	$ netsh interface portproxy delete v4tov4 listenport=80 listenaddress=122.116.214.159
	$ netsh interface portproxy delete v4tov4 listenport=443 listenaddress=122.116.214.159
# dump the setting
	$netsh interface portproxy dump
######  **note** : if you reopen window , remember to reset again. even you can see the setting is still exist. but acturlly it's not working. 

---------------------

# Way to add ssl manully
$ sudo vim /etc/nginx/conf.d/default.conf

	server {
		listen 80;
		listen 443 ssl;
		listen [::]:443 ssl;

		#ssl_certificate /etc/nginx/ssl/nginx.crt;
		#ssl_certificate_key /etc/nginx/ssl/nginx.key;

		#include snippets/ssl-params.conf;

		server_name hosenmassage.ddns.net;   # use your own domain

		location / {
			include snippets/wp-reverse-proxy.conf;
			proxy_pass http://localhost:8080; # note this is same as docker-compose port setting 
		}
	}

$ sudo vim /etc/nginx/nginx.conf

    http {

			##
			# Basic Settings
			##

			# the certification localtion
			ssl_certificate /etc/nginx/ssl/nginx.crt;
			ssl_certificate_key /etc/nginx/ssl/nginx.key;

nginx -t 
	systemctl start nginx

# Way to add ssl auto by using cerbot. 
	https://certbot.eff.org/lets-encrypt/ubuntubionic-nginx

run Certbot Either get and install your certificates
$ sudo certbot --nginx

Or, just get a certificate
$ sudo certbot certonly --nginx


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

	   
# renew ssl ,  it will set cron job in   /etc/cron.d/certbot
	$ sudo certbot renew --dry-run

# Using wordpress plug-in "UPDRAFTPLUS" to back data  
#### (https://wordpress.blog.tw/updraftplus-wordpress-backup-plugin/)

volumes : 
-------------------------------
root@ubuntu:/home/jerry/wp-test# docker volume ls

    DRIVER              VOLUME NAME
    local               c2474ac0824cb7303cce5d441803f18bba02f7b0361ff9f93299d8e6b339ab00
    local               daac2067d30553856bd8289c6937dcf2276c29667b0ebdf912a6114f42d96dd3
    local               wp-test_db_data

$ docker volume inspect wp-test_db_data
DB locate at :  /var/snap/docker/common/var-lib-docker/volumes/wp-test_db_data/_data

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
wordpress  locate at : 

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
            
# other ref 
backup manully : https://zju.date/wordpress-backup/
docker run -d -e VIRTUAL_HOST=hosenmassage.ddns.net \
              -e LETSENCRYPT_HOST=hosenmassage.ddns.net \
              -e LETSENCRYPT_EMAIL=kc109763@gmail.com \
              --network=webproxy \
              --name my_app \
              httpd:alpine

root@ubuntu:/home/jerry/docker-compose-letsencrypt-nginx-proxy-companion# ./test_start_ssl.sh 192.168.157.129

