# Modul Jarkom Kelompok A06

## Anggota Kelompok

1. Hans Sean Nathanael - 5025201019
2. Mohammad Fany Faizul Akbar - 5025201225
3. Fadel Pramaputra Maulana - 5025201260

## Nomor 1

### Soal

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

3. Jalankan beberapa command:
   Menjadikan WISE sebagai DNS Master

```bash
apt-get update
apt-get install bind9 -y
```

Kemudian setiap node diaktifkan dengan mengklik tombol start. Setelah itu, menjalankan command iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 10.45.0.0/16 pada router Ostania agar dapat terkoneksi dengan internet. Lakukan ping google.com pada setiap node untuk mengecek apakah sudah terhubung dengan internet.
