<h1>OJS</h1>
<h3>Konfigurasi Docker di Host Docker Development ( 10.250.30.20 )</h3>
<ul>
  <li>IP : 10.250.30.20</li>
  <li>User : root</li>
  <li>Pass :</li>
</ul>

<p>Masuk ke /DockerApps/ dan pilih app, Contoh Jurnal fetrian</p>

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
host = host
username = username
password = password
name = ojs_fetrian
```
<p>Dan juga pastika ada di mysql, lalu import databasenya</p>

```console
root@docker-dev:/DockerApps/fetrian-prod# mysqldump -u root -p -d ojs_fetrian > ojs_fetrian.sql
```
<p>Hasil Direktory sekarang</p>

```console
root@docker-dev:/DockerApps/fetrian-prod# ls -l
total 100
-rw-r--r-- 1 root root   491 Feb  8 12:20 docker-compose.yml
-rw-r--r-- 1 root root   402 Feb  8 12:15 Dockerfile
-rw-r--r-- 1 root root 86891 Feb  8 12:39 ojs_fetrian.sql
drwxr-xr-x 4 root root  4096 Feb  8 12:40 src
root@docker-dev:/DockerApps/fetrian-prod# ls -l src/
total 12
-rwxr-xr-x  1 root root  231 Feb  8 12:23 entrypoint.sh
drwxr-xr-x 19 root root 4096 Feb  8 12:13 html
drwxr-xr-x  6 root root 4096 Feb  8 12:13 ojs-files
root@docker-dev:/DockerApps/fetrian-prod#
```

<p>Lakukan build dengan perintah 'docker-compose build' pada app di direktori app (yang ada docker-compose.yml) seperti berikut, lalu pastikan imagenya berhasil,</p>

```console
root@docker-dev:/DockerApps/fetrian-prod# docker-compose build
Building web
Sending build context to Docker daemon  782.1MB
Step 1/3 : FROM docker-registry.unand.ac.id:8888/ojs-nginx-php7
 ---> 3f6256a3fda6
Step 2/3 : COPY src/html /var/www/html/
 ---> 1d576501a1cf
Step 3/3 : EXPOSE 80
 ---> Running in 279c021beabf
Removing intermediate container 279c021beabf
 ---> a8358cc5733c
Successfully built a8358cc5733c
Successfully tagged fetrian:3.3.0-20
root@docker-dev:/DockerApps/fetrian-prod# docker images | grep -i 'fetrian'
fetrian                                            3.3.0-20   a8358cc5733c   14 seconds ago   609MB
```
<p>Jika berhasil, jalankan image dengan perintah 'docker-compose up -d' di direktori app, dan pastikan halaman web tampil dan tidak ada error</p>

<h3>Konfigurasi Docker di Host Docker Production ( 10.250.29.1 )</h3>
<ul>
  <li>IP : 10.250.29.1</li>
  <li>User : root</li>
  <li>Pass :</li>
</ul>
<p>Masuk ke folder //DockerApps/ dan buat folder appnya</p>
