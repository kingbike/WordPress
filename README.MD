# ENV setting 
- **Step 1: 
    Setting windows portfording to virtual machine**  
	- $ netsh interface portproxy add v4tov4 listenport=80 listenaddress=<yourPublicIP> connectport=80 connectaddress=<yourVmIP>
	- $ netsh interface portproxy add v4tov4 listenport=4443 listenaddress=<yourPublicIP> connectport=443 connectaddress=<yourVmIP>

    **remove the setting** 
	- $ netsh interface portproxy delete v4tov4 listenport=80 listenaddress=<yourPublicIP>
	- $ netsh interface portproxy delete v4tov4 listenport=443 listenaddress=<yourPublicIP>
    
    **dump the setting**
	- $netsh interface portproxy dump
	
    **note** : If you restart Windows remember to reset again. even you can see the setting is still exist. but acturlly it's not working. 

- Step 2: put **./default.conf**   to  **/etc/nginx/conf.d/default.conf**
- Step 3: Put **./wp-reverse-proxy.conf** to  **/etc/nginx/snippets/wp-reverse-proxy.conf**
- Step 4: docker-compose will put **uploads.ini** to container wordpress  **/usr/local/etc/php/conf.d/uploads.ini**

# Way to add ssl manully : 
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

#####  check the nginx configuratoin
    $ nginx -t 
#####  start nginx service 
    $ systemctl start nginx


# Way to add ssl auto by using cerbot: 
	https://certbot.eff.org/lets-encrypt/ubuntubionic-nginx

#####  Run Certbot Either get and install your certificates
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

#####  renew ssl,  it will set cron job in  /etc/cron.d/certbot
	$ sudo certbot renew --dry-run

##### Install wordpress plug-in "UPDRAFTPLUS" to back data  
#### (https://wordpress.blog.tw/updraftplus-wordpress-backup-plugin/)


# volumes  operations : 
#####  $ docker volume ls
    DRIVER              VOLUME NAME
    local               c2474ac0824cb7303cce5d441803f18bba02f7b0361ff9f93299d8e6b339ab00
    local               daac2067d30553856bd8289c6937dcf2276c29667b0ebdf912a6114f42d96dd3
    local               wp-test_db_data

##### $ docker volume inspect wp-test_db_data
    # DB locate at :  /var/snap/docker/common/var-lib-docker/volumes/wp-test_db_data/_data
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

#### $ docker volume inspect wp-test_wordpress_1
    # wordpress  locate at : "/var/snap/docker/common/var-lib-docker/volumes/c2474ac0824cb7303cce5d441803f18bba02f7b0361ff9f93299d8e6b339ab00/_data"

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


# DNS container.( host can comunicate with container, and run multiple container in one service )
$ docker run --hostname dns.mageddo --restart=unless-stopped -p 5380:5380 \
-v /var/run/docker.sock:/var/run/docker.sock \
-v /etc/resolv.conf:/etc/resolv.conf \
defreitas/dns-proxy-server
- container must setting : hostname , network bridge mode.  


# I 'bypassed' the FTP credential page but got blocked by "Could not create directory" after the installation had finished.
$ docker exec -u root -it {CONTAINER_ID} /bin/bash

$ chown -R www-data wp-content

$ chmod -R 755 wp-content


# Other ref 

#### Rollback data from dropbox  
onece I reinstall docker and the db data was broken which make the website  white blank. 
I re-create container and assign new(empty) dir for mysql container. Then install the wordpress plugin "UpdraftPlus"  to restore data.

#### Every time upgrade OS then Restart docker-compose and got blank page 
$ vim docker-compose.yml  
chang the database volume. mounting path to the empty new one.

$ docker-compose down
$ docker-compose up

you can see whole new wordpress page 

$ Ctrl + Z 

$ vim docker-compose.yml  
chang the database volume. mounting path to the older one.
$ docker-compose up -d

then reinstall "UpdraftPlus"  to restore data.

backupData contains :  db,  themes, plugins, uploads, others ...
    volumes:
      - "./themes:/var/www/html/wp-content/themes/"
      - "./plugins:/var/www/html/wp-content/plugins/"
      - "./uploads:/var/www/html/wp-content/uploads/"

# debug run in forgroupd to see output log
- $ docker-compose up 
- $ docker logs <container>
- $ docker exec -it <container> bash 

# backup manully 
    https://zju.date/wordpress-backup/

# nginx container
$ docker run -d -e VIRTUAL_HOST=hosenmassage.ddns.net \
              -e LETSENCRYPT_HOST=hosenmassage.ddns.net \
              -e LETSENCRYPT_EMAIL=kc109763@gmail.com \
              --network=webproxy \
              --name my_app \
              httpd:alpine

root@ubuntu:/home/jerry/docker-compose-letsencrypt-nginx-proxy-companion# ./test_start_ssl.sh 192.168.157.129





---
# docker-compose command 
---
##### docker-compose command to create containers and run in backround 
    $ docker-compose up -d
##### docker-compose command to show running containers
    $ docker-compose ps
##### docker-compose command to stop containers
    $ docker-compose stop
##### docker-compose command to remove containers
    $ docker-compose rm
##### if you modify the docker-compose.yaml file just restart the service. it will become effective
    $ docker-compose restar <service ex:db>

## volumes
---
- While you remove containers the volume dokcer-compose created will not be acturlly remove. 
##### Command to show where volumes is (the volume acturlly path is in windows vm. )
    $  docker inspect -f '{{range .Mounts}}{{.Source}}{{end}}' <containerId>    
        /var/lib/docker/volumes/wordpress_db_data/_data
##### show volume status 
    $ docker volume ls 
    $ docker volume inspect <wordpress_db_data>
##### let volmes splite from container 
    $ docker-compose down --volumes
##### delete all volumes permanently  
    $ docker volume prune
    ##### WARNING! This will remove all local volumes not used by at least one container.
    Are you sure you want to continue? [y/N] y

##### while using windows K8s the volume will locate in C:\Users\jerry\.docker\Volumes
    PS D:\workspace\wordpress> docker inspect -f '{{range .Mounts}}{{.Source}}{{end}}' 53d70322b90f    
    /var/lib/kubelet/pods/34968a80-3ce8-11ea-adb3-00155d6d0107/containers/mysql/7c736abb/host_mnt/c/Users/jerry/.docker/Volumes/mysql-pv-claim/pvc-349664c5-3ce8-11ea-adb3-00155d6d0107/var/lib/kubelet/pods/34968a80-3ce8-11ea-adb3-00155d6d0107/volumes/kubernetes.io~secret/default-token-x4mkh/var/lib/kubelet/pods/34968a80-3ce8-11ea-adb3-00155d6d0107/etc-hosts
