<h1>OJS</h1>
<h4>Config docker-compose dan Dockerfile di Host Docker Development ( 10.250.30.20 )</h4>
<ul>
  <li>IP : 10.250.30.20</li>
  <li>User : root</li>
  <li>Pass :</li>
</ul>

<p>Contoh Jurnal fetrian</p>

```console
root@docker-dev:/DockerApps/fetrian-prod# ls -l
total 8
-rw-r--r-- 1 root root  385 Feb  8 12:13 docker-compose.yml
drwxr-xr-x 4 root root 4096 Feb  8 12:13 src
```

<p>Buat file Dockerfile lalu isi service yang dibutuhkan (Contoh : versi PHP yang dibutuhkan), dan simpan pada direktori app (Contoh : disini pada folder fetrian-prod) berikut Dockerfile</p>

```dockerfile
FROM docker-registry.unand.ac.id:8888/ojs-nginx-php7

COPY src/html /var/www/html/

EXPOSE 80
```
<p>Buat file docker-compose.yml lalu isi service yang dibutuhkan, dan sesuaikan confignya (Contoh : image, container_name, port dan lainnya), lalu simpan pada direktori app (Contoh : disini pada folder fetrian-prod) berikut docker-compose.yml</p>

```yml
version: '3.8'

services:
  web:
    build:
      context: .
      dockerfile: Dockerfile
    image: fetrian:3.3.0-20
    container_name: fetrian
    ports:
      - "5004:80"
    volumes:
      - ./src/ojs-files:/var/www/ojs-files
      - ./src/html/public:/var/www/html/public
      - ./src/html/cache:/var/www/html/cache
      - ./src/html/config.inc.php:/var/www/html/config.inch.php
      - ./src/entrypoint.sh:/usr/local/bin/entrypoint.sh
    entrypoint: ["/usr/local/bin/entrypoint.sh"]
```
<p>Buat file script entrypoint.sh untuk mengganti permission pada app yang letaknya di folder {nama-app}/src/. sesuaikan juga semua confignya dengan yang dibutuhkan</p>

```bash
#!/bin/bash
set -e

chown -R www-data:www-data /var/www
chmod -R 755 /var/www/ojs-files
chmod -R 555 /var/www/html
chmod -R 755 /var/www/html/public
chmod -R 755 /var/www/html/cache

service php7.4-fpm start
nginx -g 'daemon off;'
```

<p>Import database dari app, lihat nama databasenya di folder config.inc.php</p>

```console
root@docker-dev:/DockerApps/fetrian-prod# grep -i "mysqli" -A5 src/html/config.inc.php
driver = mysqli
host = 10.250.30.20
username = docker
password = ABCDEF
name = ojs_fetrian
```

<p>Hasil Direktory sekarang</p>

```console
root@docker-dev:/DockerApps/fetrian-prod# ls -l
total 16
-rw-r--r-- 1 root root  491 Feb  8 12:20 docker-compose.yml
-rw-r--r-- 1 root root  402 Feb  8 12:15 Dockerfile
-rwxr-xr-x 1 root root  231 Feb  8 12:23 entrypoint.sh
drwxr-xr-x 4 root root 4096 Feb  8 12:13 src
root@docker-dev:/DockerApps/fetrian-prod#
```
