# QNAP Let's Encrypt certificate manager

Built on top of [Certbot]("https://github.com/certbot/certbot") and uses Certbot plugins.

## Functionalities

* Allow to get certificate from Let's Encrypt for NAS webpage
* Automatic renewal using cron
* Allow to get any additional certificate you want

## Cron

```sh
30 3 * * * /path/to/renew_certificate.sh >> /path/to/renew_certificate.log 2>&1
```

## QNAP Configuration

Reference link in the QNAP forum: https://forum.qnap.com/viewtopic.php?t=139548

### QNAP web server

You have to change the default value of the listenning port for QNAP web page since we want to use the reverse proxy.

QNAP web server uses the certificate in */etc/stunnel/stunnel.pem*.

#### With Qnap web administration

* Control Panel -> System -> General Parameters -> System Administration
  * System Port: other than 80 (here 8080)
  * Activate HTTPS
  * HTTPS Port Number: other than 443 (here 4430)

#### SSH

You can mess up pretty bad and get locked outside of administration page.

To regain control type this and restart services (see below):

```sh
sudo setcfg "Web Access Port" 8080
```

### Web Server: Apache configuration

The goal of this is to configure Apache to listen on port 80 and 443.

#### With Qnap web administration

* Control Panel -> Applications -> Server Web: this is for apache
  * Activate Web Server
  * Port Number: 80
  * Activate HTTPS
  * HTTPS Port Number: 443

#### SSH

You can read or force values with those commands:
* sudo setcfg xxx
* getcfg xxx

Here is an example to force the config above:

```sh
sudo setcfg QWEB Enable 1
sudo setcfg QWEB Port 80
sudo setcfg QWEB Enable_SSL 1
sudo setcfg QWEB SSL_PORT 443
```

### Setting reverse proxy

#### Tell Apache where to look for reverse proxy file

In file */mnt/HDA_ROOT/.config/apache/apache.conf*, add at the end:
```
Include /share/CACHEDEV1_DATA/Web/custom.conf
```

#### /share/Web Folder

- Create a */share/Web* folder
- Create a *index.php* empty file
- Create a *custom.conf* file and add you virtual hosts there (example with redirect to https):

```
LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_http_module modules/mod_proxy_http.so

<VirtualHost *:80>
	ServerName example.com
  Redirect / https://example.com/
</VirtualHost>

<VirtualHost *:443>
	ServerName example.com

	SSLProxyEngine On
  SSLCertificateKeyFile /path/to/privkey.pem	
	SSLCertificateFile  /path/to/fullchain.pem	
	ProxyPass / http://localhost:port/
	ProxyPassReverse / http://localhost:port/
</VirtualHost>

```

## SSH commands to use

### List opened ports

```sh
sudo netstat -tulpn | grep :443
sudo netstat -tulpn | grep :80
```

You should see this:
```
tcp        0      0 :::80                   :::*                    LISTEN      23202/apache
tcp        0      0 :::8080                 :::*                    LISTEN      19132/apache_proxy
tcp        0      0 :::443                  :::*                    LISTEN      23202/apache
tcp        0      0 :::4430                 :::*                    LISTEN      13790/apache_proxys
```

### Restart manually services

```sh
sudo /etc/init.d/Qthttpd.sh restart
sudo /etc/init.d/stunnel.sh restart
```

## Debug

- If HTTPS server failed, verify if the certificate /etc/config/stunnel/stunnel.pem is correct (private key + certificate)
- List of directories:
  - /etc/config/stunnel
  - /etc/config/apache
  - /etc/default_config/apache
  - /usr/local/apache (contains log file if error)
- Under certain conditions, file /etc/default_config/apache.conf can replace /etc/apache.conf
