# Modul Jarkom Kelompok A06

## Anggota Kelompok

1. Hans Sean Nathanael - 5025201019
2. Mohammad Fany Faizul Akbar - 5025201225
3. Fadel Pramaputra Maulana - 5025201260

## Nomor 1

### Soal

WISE akan dijadikan sebagai DNS Master, Berlint akan dijadikan DNS Slave, dan Eden akan digunakan sebagai Web Server. Terdapat 2 Client yaitu SSS, dan Garden. Semua node terhubung pada router Ostania, sehingga dapat mengakses internet!

### Cara Pengerjaan

1. Membuat topologi sebagai berikut:

![dokumentasi 1-1](image/Nomor%201/1.png)

2. Melakukan konfigurasi pada setiap node yang ada

Konfigurasi Ostania sebagai router

```bash
auto eth0
iface eth0 inet dhcp

auto eth1
iface eth1 inet static
address 10.2.1.1
netmask 255.255.255.0

auto eth2
iface eth2 inet static
address 10.2.2.1
netmask 255.255.255.0

auto eth3
iface eth3 inet static
address 10.2.3.1
netmask 255.255.255.0
```

Konfigurasi WISE sebagai DNS Master

```bash
auto eth0
iface eth0 inet static
	address 10.2.3.2
	netmask 255.255.255.0
	gateway 10.2.3.1
```

Konfigurasi SSS sebagai Client

```bash
auto eth0
iface eth0 inet static
	address 10.2.1.2
	netmask 255.255.255.0
	gateway 10.2.1.1
```

Konfigurasi Garden sebagai Client

```bash
auto eth0
iface eth0 inet static
	address 10.2.1.3
	netmask 255.255.255.0
	gateway 10.2.1.1
```

Konfigurasi Berlint sebagai DNS Slave

```bash
auto eth0
iface eth0 inet static
	address 10.2.2.2
	netmask 255.255.255.0
	gateway 10.2.2.1
```

Konfigurasi Eden sebagai Web Server

```bash
auto eth0
iface eth0 inet static
	address 10.2.2.3
	netmask 255.255.255.0
	gateway 10.2.2.1
```

3. Menjadikan WISE sebagai DNS Master dan Berlint sebagai DNS Slave dengan menjadikan command dibawah ini ke-kedua node tersebut

```bash
apt-get update
apt-get install bind9 -y
```

4. Memasukan nameserver kesetiap node agar bisa terhubung dengan internet

```bash
echo "nameserver 192.168.122.1" > /etc/resolv.conf
```

5. Kemudian setiap node diaktifkan dengan mengklik tombol start. Setelah itu, menjalankan command

```bash
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 10.45.0.0/16
```

pada router Ostania agar dapat terkoneksi dengan internet. Lakukan ping google.com pada setiap node untuk mengecek apakah sudah terhubung dengan internet.

## Nomor 2

### Soal

Untuk mempermudah mendapatkan informasi mengenai misi dari Handler, bantulah Loid membuat website utama dengan akses wise.yyy.com dengan alias www.wise.yyy.com pada folder wise!

### Cara Pengerjaan

1. Pada DNS Master (WISE), dilakukan konfigurasi terhadap file `/etc/bind/named.conf.local` dengan menambahkan:

```
zone "wise.A06.com" {
        type master;
        notify yes;
        also-notify { 10.2.2.2; };
        allow-transfer { 10.2.2.2; };
        file "/etc/bind/wise/wise.A06.com";
};
```

2. Membuat direktori baru di WISE yaitu `/etc/bind/wise` dan menambahkan konfigurasi pada `/etc/bind/wise/wise.A06.com`:

```
$TTL    604800
@       IN      SOA     wise.A06.com. root.wise.A06.com. (
                     2410202201         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@               IN      NS      wise.A06.com.
@               IN      A       10.2.3.2
www             IN      CNAME   wise.A06.com.
```

Kemudian melakukan restart bind9 dengan perintah `service bind9 restart`

3. Lakukan command `echo "nameserver 10.2.3.2" > /etc/resolv.conf` pada kedua client agar client dapat terhubung dengan WISE. Testing dengan melakukan ping dan command pada salah satu client dibawah ini:

`ping www.wiseA06.com`

![dokumentasi 2-1](image/Nomor%202/1.png)

`ping wise.A06.com`

![dokumentasi 2-2](image/Nomor%202/2.png)

`host -t CNAME www.wise.A06.com`

![dokumentasi 2-3](image/Nomor%202/3.png)

## Nomor 3

### Soal

Setelah itu ia juga ingin membuat subdomain eden.wise.yyy.com dengan alias www.eden.wise.yyy.com yang diatur DNS-nya di WISE dan mengarah ke Eden!

### Cara Pengerjaan

1. Pada WISE menambahkan konfigurasi pada `/etc/bind/wise/wise.A06.com`:

```
$TTL    604800
@       IN      SOA     wise.A06.com. root.wise.A06.com. (
                     2410202201         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@               IN      NS      wise.A06.com.
@               IN      A       10.2.3.2
www             IN      CNAME   wise.A06.com.
eden            IN      A       10.2.2.3
www.eden        IN      CNAME   eden
```

Kemudian melakukan restart bind9 dengan perintah `service bind9 restart`

3. Testing dengan melakukan ping dan command pada salah satu client dibawah ini:

`ping www.eden.wiseA06.com`

![dokumentasi 3-1](image/Nomor%203/1.png)

`ping eden.wise.A06.com`

![dokumentasi 3-2](image/Nomor%203/2.png)

`host -t CNAME www.eden.wise.A06.com`

![dokumentasi 3-3](image/Nomor%203/3.png)

## Nomor 4

### Soal

Buat juga reverse domain untuk domain utama!

### Cara Pengerjaan

1. Pada WISE edit file `/etc/bind/named.conf.local` menjadi sebagai berikut:

```
zone "wise.A06.com" {
        type master;
        notify yes;
        also-notify { 10.2.2.2; };
        allow-transfer { 10.2.2.2; };
        file "/etc/bind/wise/wise.A06.com";
};

zone "3.2.10.in-addr.arpa" {
        type master;
        notify yes;
        also-notify { 10.2.2.2; };
        allow-transfer { 10.2.2.2; };
        file "/etc/bind/wise/3.2.10.in-addr.arpa";
};
```

2. Lakukan konfigurasi pada file `/etc/bind/wise/3.2.10.in-addr.arpa` sebagai berikut:

```
$TTL    604800
@       IN      SOA     wise.A06.com. root.wise.A06.com. (
                     2410202202         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
3.2.10.in-addr.arpa.    IN      NS      wise.A06.com.
2                       IN      PTR     wise.A06.com.
```

3. Lakukan testing dengan command `host -t PTR 10.2.3.2`

![dokumentasi 4-1](image/Nomor%204/1.png)

## Nomor 5

### Soal

Agar dapat tetap dihubungi jika server WISE bermasalah, buatlah juga Berlint sebagai DNS Slave untuk domain utama!

### Cara Pengerjaan

1. Lakukan konfigurasi pada node Berlint pada file `/etc/bind/named.conf.local` untuk melakukan konfigurasi DNS Slave yang mengarah ke WISE:

```
zone "wise.A06.com" {
        type slave;
        masters { 10.2.3.2; };
        file "/var/lib/bind/wise.A06.com";
};

zone "3.2.10.in-addr.arpa" {
        type slave;
        masters { 10.2.3.2; };
        file "/var/lib/bind/3.2.10.in-addr.arpa";
};
```

2. Kemudian melakukan command

```
apt-get update
apt-get install bind9 -y
```

yang sudah dijelaskan di nomor 1 karena Berlint akan dijadikan DNS Slave. Kemudian restart service bind9 dengan `service bind9 restart`.

3. Lakukan testing

Stop service bind9 pada WISE dengan command `service bind9 stop`:

![dokumentasi 5-1](image/Nomor%205/1.png)

Kemudian lakukan ping pada salah satu client:

![dokumentasi 5-2](image/Nomor%205/2.png)

## Nomor 6

### Soal
Karena banyak informasi dari Handler, buatlah subdomain yang khusus untuk operation yaitu `operation.wise.yyy.com` dengan alias `www.operation.wise.yyy.com` yang didelegasikan dari `WISE` ke `Berlint` dengan IP menuju ke `Eden` dalam folder `operation`.

### Jawaban
>>>>>>> 51a68a2ef06225830ba85bbf090d2e84b6e97f18

### Server WISE
Tambahkan konfigurasi berikut ke `/etc/bind/wise/wise.A06.com` pada `zone "wise.A06.com"`.
```
ns1             IN      A       10.2.2.3
operation       IN      NS      ns1
```
Tambahkan konfigurasi berikut ke `/etc/bind/named.conf.options`.
```
allow-query{any;};
```
Tambahkan konfigurasi berikut ke `/etc/bind/named.conf.local` di `zone "wise.a06.com"`.
```
allow-transfer { 10.2.2.2; };
```
Lalu, lakukan restart sevice bind9 dengan `service bind9 restart`.

### Server Berlint
Tambahkan konfigurasi berikut ke `/etc/bind/named.conf.options`.
```
allow-query{any;};
```
Lalu tambahkan konfigurasi berikut untuk delegasi `operation.wise.A06.com`.
```
zone \"operation.wise.A06.com\" {
        type master;
        file \"/etc/bind/wise/operation.wise.A06.com\";
};
```

### Testing

![Testing No 6](image/Nomor%206/1.png)

## Nomor 7

### Soal
Untuk informasi yang lebih spesifik mengenai Operation Strix, buatlah subdomain
melalui `Berlint` dengan akses `strix.operation.wise.yyy.com` dengan alias
`www.strix.operation.wise.yyy.com` yang mengarah ke `Eden`.

### Jawaban
Tambahkan konfigurasi berikut ke `/etc/bind/wise/operation.wise.A06.com`.
```
strix           IN      A       10.2.2.3
www.strix       IN      CNAME   strix
```

### Testing
![Testing No 7](image/Nomor%207/1.png)
## Nomor 8

### Soal
Setelah melakukan konfigurasi server, maka dilakukan konfigurasi Webserver.
Pertama dengan webserver `www.wise.yyy.com`. Pertama, Loid membutuhkan webserver
dengan `DocumentRoot` pada `/var/www/wise.yyy.com`.

### Jawaban

### Client ( atau Garden)
Install `lynx` terlebih dahulu.
```
apt-get update
apt-get install lynx -y
```
### Server Eden
Install `Apache` dan `php` terlebih dahulu dan dijalankan.
```
apt-get update
apt-get install apache2 -y
apt-get install php -y
apt-get install libapache2-mod-php7.0
service apache2 start
service php start
```
Buat file konfigurasi `/etc/apache2/sites-available/wise.A06.com.conf` lalu edit konfigurasi seperti sebagai berikut.
```
<VirtualHost *:80>

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/wise.A06.com
        ServerName wise.A06.com
        ServerAlias www.wise.A06.com

        ErrorLog \${APACHE_LOG_DIR}/error.log
        CustomLog \${APACHE_LOG_DIR}/access.log combined

</VirtualHost>
```
Lalu, buat direktori `/var/www/wise.A06.com`, masukkan file requirement kedalamnya, lalu restart `apache`.
```
mkdir /var/www/wise.A06.com
wget --no-check-certificate -q "https://drive.google.com/uc?export=download&id=1S0XhL9ViYN7TyCj2W66BNEXQD2AAAw2e" -O "/var/www/wise.A06.com/wise.zip"
unzip -q -o "/var/www/wise.A06.com/wise.zip" "wise/*" -d "/var/www/wise.A06.com"
service apache2 restart
```

### Testing
![Testing No 8](image/Nomor%208/1.png)
![Testing No 8](image/Nomor%208/2.png)
## Nomor 9

### Soal
Setelah itu, Loid juga
membutuhkan agar url `www.wise.yyy.com/index.php/home` dapat menjadi menjadi
`www.wise.yyy.com/home`

### Jawaban
Konfigurasikan file `/var/www/eden.wise.A06.com/.htaccess` menggunakan konfigurasi berikut.
```
RewriteEngine On
RewriteCond %{REQUEST_URI} ^/public/images/(.*)eden(.*)
RewriteCond %{REQUEST_URI} !/public/images/eden.png
RewriteRule /.* http://eden.wise.A06.com/public/images/eden.png [L]
```
Lalu tambahkan konfigurasi berikut di `/etc/apache2/sites-available/eden.wise.A06.com.conf` dalam `<VirtualHost *:80>`.
```
<Directory /var/www/eden.wise.A06.com>
	Options +FollowSymLinks -Multiviews
	AllowOverride All
</Directory>
```
### Testing
![Testing No 9](image/Nomor%209/1.png)
![Testing No 9](image/Nomor%209/2.png)
## Nomor 10

### Soal
Setelah itu, pada subdomain `www.eden.wise.yyy`.com, Loid membutuhkan
penyimpanan aset yang memiliki `DocumentRoot` pada `/var/www/eden.wise.yyy.com`.

### Jawaban
```
## <VirtualHost *:80>

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/eden.wise.A06.com
        ServerName eden.wise.A06.com
        ServerAlias www.eden.wise.A06.com

        ErrorLog \${APACHE_LOG_DIR}/error.log
        CustomLog \${APACHE_LOG_DIR}/access.log combined
		
		<Directory /var/www/eden.wise.A06.com>
			Options +FollowSymLinks -Multiviews
			AllowOverride All
		</Directory>

</VirtualHost>
```
Lalu, nyalakan `virtualHost` dengan `a2ensite` dan buat direktori `/var/www/eden.wise.A06.com`, masukkan file requirement kedalamnya, lalu restart `apache`.
```
a2ensite eden.wise.A06.com
mkdir /var/www/eden.wise.A06.com
wget --no-check-certificate -q "https://drive.google.com/uc?id=1q9g6nM85bW5T9f5yoyXtDqonUKKCHOTV&export=download" -O "/var/www/eden.wise.A06.com/eden.wise.zip"
unzip -q -o "/var/www/eden.wise.A06.com/eden.wise.zip" "eden.wise/*" -d "/var/www/eden.wise.A06.com"
service apache2 restart
```
### Testing
![Testing No 10](image/Nomor%2010/1.png)
![Testing No 10](image/Nomor%2010/2.png)
Nomor 11

### Soal
Akan tetapi, pada folder `/public`, Loid ingin hanya dapat melakukan directory listing saja.

### Jawaban
Tambahkan konfigurasi berikut ini di `/etc/apache2/sites-available/eden.wise.A06.com.conf` dalam `<VirtualHost *:80>`.
```
<Directory /var/www/eden.wise.A06.com/public>
	Options +Indexes
</Directory>
```
### Testing
![Testing No 11](image/Nomor%2011/1.png)
![Testing No 11](image/Nomor%2011/2.png)

## Nomor 12

### soal

Tidak hanya itu, Loid juga ingin menyiapkan error file 404.html pada folder /error untuk mengganti error kode pada apache.

### Cara Pengerjaan

Menambahkan keyword ```ErrorDocument <kode error> <directory>``` ke dalam file pengaturan apache eden.wise.A06.com ```/etc/apache2/sites-available/eden.wise.A06.com.conf```. Untuk soal ini error yang diganti adalah error 404, 500, 502, 503, dan 504, kemudian directory file error adalah ```/error/404.html```.

```bash
<VirtualHost *:80>

        ...

        ErrorDocument 404 /error/404.html
        ErrorDocument 500 /error/404.html
        ErrorDocument 502 /error/404.html
        ErrorDocument 503 /error/404.html
        ErrorDocument 504 /error/404.html

        ...

</VirtualHost>
```

![Dokumentasi 12-1](image/Nomor%2012/nomor%2012-1.png)

![Dokumentasi 12-2](image/Nomor%2012/nomor%2012-2.png)

## Nomor 13

### soal

Loid juga meminta Franky untuk dibuatkan konfigurasi virtual host. Virtual host ini bertujuan untuk dapat mengakses file asset www.eden.wise.yyy.com/public/js menjadi www.eden.wise.yyy.com/js 

### Cara Pengerjaan

Dengan membuat alias "js" yang mengarah pada directory "public/js" pada file pengaturan apache eden.wise.A06.com.

```bash
<VirtualHost *:80>

        ...
		
		Alias "js" "public/js"
		
		...

</VirtualHost>
```

![Dokumentasi 13-1](image/Nomor%2013/nomor%2013-1.png)

![Dokumentasi 13-2](image/Nomor%2013/nomor%2013-2.png)

## Nomor 14

### soal

Loid meminta agar www.strix.operation.wise.yyy.com hanya bisa diakses dengan port 15000 dan port 15500 

### Cara Pengerjaan

Dengan membuat dua VirtualHost untuk www.strix.operation.wise.A06.com pada file pengaturannya ```/etc/apache2/sites-available/strix.operation.wise.A06.com.conf``` yang menggunakan port 15000 dan 15500. Isi kedua VirtualHost dibuat sama, perbedaan hanya pada port yang digunakan.

```bash
<VirtualHost *:15000>

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/strix.operation.wise.A06
        ServerName strix.operation.wise.A06.com
        ServerAlias www.strix.operation.wise.A06.com

        ErrorLog \${APACHE_LOG_DIR}/error.log
        CustomLog \${APACHE_LOG_DIR}/access.log combined
		
</VirtualHost>
<VirtualHost *:15500>        
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/strix.operation.wise.A06
        ServerName strix.operation.wise.A06.com
        ServerAlias www.strix.operation.wise.A06.com
        
        ErrorLog \${APACHE_LOG_DIR}/error.log
        CustomLog \${APACHE_LOG_DIR}/access.log combined

</VirtualHost>
```

kemudian memasukkan port yang digunakan ke dalam file daftar port yang digunakan apache2 ```etc/apache2/ports.conf```.

```bash
Listen 80
Listen 15000
Listen 15500

<IfModule ssl_module>
        Listen 443
</IfModule>

<IfModule mod_gnutls.c>
        Listen 443
</IfModule>
```

![Dokumentasi 14-1](image/Nomor%2014/nomor%2014-1.png)

![Dokumentasi 14-2](image/Nomor%2014/nomor%2014-2.png)

## Nomor 15

### soal

dengan autentikasi username Twilight dan password opStrix dan file di /var/www/strix.operation.wise.yyy

### Cara Pengerjaan

Menambahkan authentication pada file pengaturan strix.operation.wise.A06.com.

```bash
<VirtualHost *:15000>

		...
        
        <Directory /var/www/strix.operation.wise.A06>
                AuthType Basic
				AuthName \"Restricted Content\"
                AuthUserFile /etc/apache2/strix.operation.wise.A06/.htpasswd
                Require valid-user
        </Directory>
		
		...
		
</VirtualHost>
<VirtualHost *:15500>

		...
        
        <Directory /var/www/strix.operation.wise.A06>
                AuthType Basic
				AuthName \"Restricted Content\"
                AuthUserFile /etc/apache2/strix.operation.wise.A06/.htpasswd
                Require valid-user
        </Directory>
		
		...
		
</VirtualHost>
```

Kemudian membuat file authentication yang berada di AuthUserFile dengan menggunakan 

```bash
htpasswd -c -b "/etc/apache2/strix.operation.wise.A06/.htpasswd" Twilight opStrix
```

![Dokumentasi 15-1](image/Nomor%2014/nomor%2014-2.png)

![Dokumentasi 15-2](image/Nomor%2015/nomor%2015-2.png)

## Nomor 16

### soal

dan setiap kali mengakses IP Eden akan dialihkan secara otomatis ke www.wise.yyy.com

### Cara Pengerjaan

Pertama, mengaktifkan modul rewrite

```bash
a2enmod rewrite
```

kemudian memasukkan rewrite ke dalam file pengaturan default ```/etc/apache2/sites-available/000-default.conf```

```bash
<VirtualHost *:80>

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/html
		
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
		
		RewriteEngine On
		RewriteRule /.* http://wise.A06.com/
</VirtualHost>
```

![Dokumentasi 16-1](image/Nomor%2016/nomor%2016-1.png)

![Dokumentasi 16-2](image/Nomor%2016/nomor%2016-2.png)

## Nomor 17

### soal

Karena website www.eden.wise.yyy.com semakin banyak pengunjung dan banyak modifikasi sehingga banyak gambar-gambar yang random, maka Loid ingin mengubah request gambar yang memiliki substring “eden” akan diarahkan menuju eden.png. Bantulah Agent Twilight dan Organisasi WISE menjaga perdamaian!

### Cara Pengerjaan

Pertama, mengizinkan rewrite pada directory ```/var/www/eden.wise.A06.com``` dengan mengubah file pengaturan eden.wise.A06.com.

```bash
<VirtualHost *:80>

        ...
		
		<Directory /var/www/eden.wise.A06.com>
			Options +FollowSymLinks -Multiviews
			AllowOverride All
		</Directory>
		
		...

</VirtualHost>
```

Kemudian membuat file rewrite ```/var/www/eden.wise.A06.com/.htaccess``` dengan isi

```bash
RewriteEngine On
RewriteCond %{REQUEST_URI} ^/public/images/(.*)eden(.*)
RewriteCond %{REQUEST_URI} !/public/images/eden.png
RewriteRule /.* http://eden.wise.A06.com/public/images/eden.png [L]
```

yang berfungsi mengubah semua request pada ```http://eden.wise.A06.com/public/images/...``` yang memiliki substring eden

![Dokumentasi 17-1](image/Nomor%2017/nomor%2017-1.png)

![Dokumentasi 17-2](image/Nomor%2017/nomor%2017-2.png)

## Dokumentasi pengerjaan
![Dokumentasi 1](/image/01.png)

![Dokumentasi 2](/image/02.png)
