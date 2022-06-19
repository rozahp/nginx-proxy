## Install nginx proxy server in a LXC container with geoip2 blocking

## 1. Launch container image

	lxc launch ubuntu:20.04 nginx-proxy
	lxc exec nginx-proxy -- bash

## 2. Update system

	apt-get update
	apt-get install nginx
	apt-get install libnginx-mod-http-geoip2
	#apt-get install nginx-extras
	mkdir /etc/nginx/ssl
	mkdir /etc/nginx/cert
	mkdir /etc/maxmind

## 3. Special certificate for blocking unwanted access and dhparam.pem

	openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/block-others.key -out /etc/ssl/certs/block-others.crt
	cd /etc/nginx/ssl	
	openssl dhparam -out dhparam.pem 4096

## 4. Download maxmind geoip2 files

	1. Go to www.maxmind.com, create an account and download: 

	GeoLite2-City_YYYYMMDD.tar.gz
	GeoLite2-Country_YYYYMMDD.tar.gz

	2. tar xvf *.tar.gz

	3. Copy these files to /etc/maxmind:

	GeoLite2-City.mmdb
	GeoLite2-Country.mmdb

## 5. Configure nginx

	lxc file push etc/nginx/nginx.conf LXD-server:nginx-proxy/etc/nginx/
	lxc file push etc/nginx/sites-available/* LXD-server:nginx-proxy/etc/nginx/sites-available/
	lxc file push etc/nginx/ssl/* LXD-server:nginx-proxy/etc/nginx/ssl/
	lxc file push etc/nginx/cert/* LXD-server:nginx-proxy/etc/nginx/cert/
	lxc file push etc/maxmind/* LXD-server:nginx-proxy/etc/maxmind/
	lxc file push etc/logrotate.d/nginx-proxy LXD-server:nginx-proxy/etc/logrotate.d/

## 6. Create directories 

	mkdir -p /var/log/nginx-proxy
	chown -R ubuntu:ubuntu /var/log/nginx-proxy
	touch /var/log/nginx-proxy/access.log
	touch /var/log/nginx-proxy/error.log

## 7. Configure your sites-available/sites-enabled accordingly

## 8. Extra - Install letsencrypt certificates

	sudo apt-get install letsencrypt
	sudo systemctl stop nginx
	sudo letsencrypt certonly --standalone -d <my.domain.name>
	sudo systemctl start nginx

## 9. Extra - Update letsencrypt certificates

	sudo systemctl stop nginx
	sudo letsencrypt renew
	sudo systemctl start nginx

## EOF
