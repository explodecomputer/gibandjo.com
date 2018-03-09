I want to use rpi to host docker containers and then for different sites to point to those containers

## 1. Creating a docker webserver
There are a few ways to do this. It seems like for static websites it’s extremely lightweight, just a few MB docker images

I used this one: [GitHub - hypriot/rpi-busybox-httpd: Raspberry Pi compatible Docker Image with a minimal `Busybox httpd` web server.](https://github.com/hypriot/rpi-busybox-httpd)

First clone the repository

```
git clone git@github.com:explodecomputer/gibandjo.com.git
```

The static files that need serving are at `repo/gibandjo.com/web`

To deploy this in a simple way just:

``` 
PORT=10008
docker run -d -v /home/pi/repo/gibandjo.com/web:/www --name=WebServer-$PORT -p $PORT:80 hypriot/rpi-busybox-httpd 
```

This is mounting the directory as a volume in the relevant directory in the container.

## 2. Creating a reverse proxy to point to the webserver
First make sure apache2 is disabled OR not listening on port 80:

```
sudo service apache2 stop
```

Or

```
sudo nano /etc/apache2/ports.conf
```

And change `Listen 80` to e.g. `Listen 81`

Now it is not occupying port 80.

Next stop the current container for the website

```
docker stop  Webserver-$PORT
```

Use this nginx method: [GitHub - lroguet/rpi-nginx-proxy: NGINX reverse proxy Docker image for Raspberry Pi](https://github.com/lroguet/rpi-nginx-proxy)

It is a rpi port of a standard linux version.

```
docker run -d -p 80:80 -v /var/run/docker.sock:/tmp/docker.sock:ro lroguet/rpi-nginx-proxy
```

And now spin up the website

```
docker run -d -v /home/pi/repo/gibandjo.com/web:/www --name=gibandjo.com -e VIRTUAL_HOST=gibandjo.com hypriot/rpi-busybox-httpd 
```

Here the `VIRTUAL_HOST` environmental variable is the incoming web address

## 3. ALTERNATIVE USING APACHE
We need to get apache to point a particular server name to a port on localhost. We use proxy pass to do this.

Turn on mod_proxy:

```
sudo a2enmod proxy
sudo a2enmod proxy_http
sudo a2enmod proxy_balancer
sudo a2enmod lbmethod_byrequests
```

Restart apache

```
sudo systemctl restart apache2
```

Create a file called `/etc/apache2/sites-available/gibandjo.com.conf` with:

```
<VirtualHost *:80>
        ServerName gibandjo.com
        ServerAlias www.gibandjo.com
        ProxyPreserveHost on
        ProxyPass / http://127.0.0.1:10008/

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

</VirtualHost>
```

And enable it

```
sudo a2enmod gibandjo.com.conf
```

And then restart apache

```
sudo systemctl restart apache2
```

Or 

```
sudo service apache2 restart
```

