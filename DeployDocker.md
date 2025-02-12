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
      - ./src/html/config.inc.php:/var/www/html/config.inc.php
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

<p>Export database dari app, lihat nama databasenya di folder config.inc.php</p>

```console
root@docker-dev:/DockerApps/fetrian-prod# grep -i "mysqli" -A5 src/html/config.inc.php
driver = mysqli
host = host
username = username
password = password
name = ojs_fetrian
```
<p>Dan juga pastika ada di mysql, lalu Export databasenya</p>

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

<p>Lanjut lakukan push ke docker registry dengan perintah berikut, sesuaikan command nya</p>

```console
root@docker-dev:/DockerApps/fetrian-prod# docker tag fetrian:3.3.0-20 docker-registry.unand.ac.id:8888/fetrian:3.3.0-20
root@docker-dev:/DockerApps/fetrian-prod# docker tag fetrian:3.3.0-20 docker-registry.unand.ac.id:8888/fetrian
root@docker-dev:/DockerApps/fetrian-prod# docker push docker-registry.unand.ac.id:8888/fetrian:3.3.0-20
root@docker-dev:/DockerApps/fetrian-prod# docker push docker-registry.unand.ac.id:8888/fetrian
```

<h3>Konfigurasi Docker di Host Docker Production ( 10.250.29.1 )</h3>
<ul>
  <li>IP : 10.250.29.1</li>
  <li>User : root</li>
  <li>Pass :</li>
</ul>
<p>Masuk ke folder /DockerApps/ dan buat folder appnya contoh "fetrian-prod" dan Copy yang sudah dikonfigurasi di Host Docker-Dev (10.250.30.20) Sebelumnya, dengan command scp</p>

```console
root@docker-1:/DockerApps# mkdir -p jrp-prod/src/html
root@docker-1:/DockerApps/fetrian-prod# scp root@10.250.30.20:/DockerApps/fetrian-prod/docker-compose.yml .
root@docker-1:/DockerApps/fetrian-prod# scp root@10.250.30.20:/DockerApps/fetrian-prod/ojs_fetrian.sql .
root@docker-1:/DockerApps/fetrian-prod# scp -r root@10.250.30.20:/DockerApps/fetrian-prod/src/ojs-files src/
root@docker-1:/DockerApps/fetrian-prod# scp root@10.250.30.20:/DockerApps/fetrian-prod/src/entrypoint.sh src/
root@docker-1:/DockerApps/fetrian-prod# scp -r root@10.250.30.20:/DockerApps/fetrian-prod/src/html/cache src/html/
root@docker-1:/DockerApps/fetrian-prod# scp -r root@10.250.30.20:/DockerApps/fetrian-prod/src/html/public src/html/
root@docker-1:/DockerApps/fetrian-prod# scp root@10.250.30.20:/DockerApps/fetrian-prod/src/html/config.inc.php src/html/
```

<p>Berikut hasil dari seluruh scp nya</p>

```console
root@docker-1:/DockerApps/fetrian-prod# ls -l
total 96
-rw-r--r-- 1 root root   493 Feb  8 13:05 docker-compose.yml
-rw-r--r-- 1 root root 86891 Feb  8 13:06 ojs_fetrian.sql
drwxr-xr-x 4 root root  4096 Feb  8 13:16 src
root@docker-1:/DockerApps/fetrian-prod# ls -l src/
total 12
-rwxr-xr-x 1 root root  231 Feb  8 13:15 entrypoint.sh
drwxr-xr-x 4 root root 4096 Feb  8 13:17 html
drwxr-xr-x 7 root root 4096 Feb  8 13:14 ojs-files
root@docker-1:/DockerApps/fetrian-prod# ls -l src/html/
total 24
drwxr-xr-x 8 root root  4096 Feb  8 13:16 cache
-rwxr-xr-x 1 root root 16116 Feb  8 13:17 config.inc.php
drwxr-xr-x 4 root root  4096 Feb  8 13:17 public
root@docker-1:/DockerApps/fetrian-prod#
```

<p>Import DB app yang Export sebelumnya ke HOST MySQL Cluster 10.250.28.2, sebelumnya buat dulu akun dan privileges untuk user mysqlnya di Host 10.250.28.2</p>
<ul>
  <li>IP : 10.250.28.2</li>
  <li>User : root</li>
  <li>Pass :</li>
</ul>
<p>Berikut pembuatan akunya dan DB nya, pastikan juga passwordnya</p>

```mysql
root@dbMaster:~# mysql -u root -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 7253
Server version: 8.0.40-31.1 Percona XtraDB Cluster (GPL), Release rel31, Revision 4b32153, WSREP version 26.1.4.3

Copyright (c) 2009-2024 Percona LLC and/or its affiliates
Copyright (c) 2000, 2024, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> create user 'fetrian'@'10.250.29.1' identified by 'sensor';
Query OK, 0 rows affected (0.02 sec)

mysql> grant all privileges on ojs_fetrian.* to 'fetrian'@'10.250.29.1';
Query OK, 0 rows affected (0.02 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.05 sec)

mysql> create database ojs_fetrian;
Query OK, 1 row affected (0.03 sec)

mysql> exit
Bye
```
<p>Selanjutnya Sinkronkan User dan Privileges yang dibuat di Host DB Cluster 10.250.28.1</p>
<ul>
  <li>IP : 10.250.28.1</li>
  <li>User : root</li>
  <li>Pass :</li>
</ul>
<p>Berikut Perintahnya</p>

```console
root@dbProxy:~# proxysql-admin --syncusers

Syncing user accounts from PXC(10.250.28.2:3306) to ProxySQL
Adding user to ProxySQL: fetrian

Note : 'admin' is in proxysql admin user list, this user cannot be added to ProxySQL
-- (For more info, see https://github.com/sysown/proxysql/issues/709)

Synced PXC users to the ProxySQL database!
```
<p>Import DB nya ke host 10.250.28.2, berikut command nya, sesuaikan dengan user dan database yang dibuat sebelumnya</p>

```console
root@docker-1:/DockerApps/fetrian-prod# mysql -u fetrian -p -h 10.250.28.2 ojs_fetrian < ojs_fetrian.sql
```
<p>Jangan lupa ganti username,password,db_name di file config.inc.php host Production (10.250.29.1)</p>

<p>Ganti Konfig file docker-compose.yml yang sebelumnya di copy jadi seperti ini</p>

```yml
version: '3.8'

services:
  web:
    image: docker-registry.unand.ac.id:8888/fetrian
    container_name: fetrian
    ports:
      - "5005:80"
    volumes:
      - ./src/ojs-files:/var/www/ojs-files
      - ./src/html/public:/var/www/html/public
      - ./src/html/cache:/var/www/html/cache
      - ./src/html/config.inc.php:/var/www/html/config.inch.php
      - ./src/entrypoint.sh:/usr/local/bin/entrypoint.sh
    entrypoint: ["/usr/local/bin/entrypoint.sh"]
    restart: always
```
<p>Terakhir, lakukan docker pull</p>

```console
docker pull docker-registry.unand.ac.id:8888/fetrian
```
<p>Dan Cek Apakah sudah running, dan tampil halaman websitenya di IP:Port yang sudah di konfigurasi</p>
