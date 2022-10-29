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
