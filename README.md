# Jarkom-Modul-3-D13-2021

### Anggota kelompok:
Anggota | NRP
------------- | -------------
Muthia Qurrota Akyun | 05111940000019
Fayha Syifa Qalbi | 05111940000185
Raihan Alifianto | 05111940000213

## Soal dan Jawaban
Luffy bersama Zoro berencana membuat peta tersebut dengan kriteria EniesLobby sebagai DNS Server, Jipangu sebagai DHCP Server, Water7 sebagai Proxy Server (1), dan Foosha sebagai DHCP Relay (2). Luffy dan Zoro menyusun peta tersebut dengan hati-hati dan teliti.

![image](https://user-images.githubusercontent.com/68548653/141451885-8fc09077-49e7-4d7b-9a0f-3b3e8cac03ae.png)

### Konfigurasi
1. Tambahkan switch, host, router, dan NAT yang diperlukan.
2. Kemudian setiap node saling dihubungkan menggunakan fitur `Add a link`
3. Lalu lakukan setting network pada setiap node dengan fitur `edit network configuration` seperti berikut. 
   - Foosha
   ```
   auto eth0
   iface eth0 inet dhcp
   
   auto eth1
   iface eth1 inet static
   	address 192.198.1.1
	netmask 255.255.255.0
   auto eth2
   iface eth2 inet static
	address 192.198.2.1
	netmask 255.255.255.0

   auto eth3
   iface eth3 inet static
	address 192.198.3.1
	netmask 255.255.255.0
   ```
   - Loguetown
   ```
   auto eth0
   iface eth0 inet static
	   address 192.198.1.2
	   netmask 255.255.255.0
	   gateway 192.198.1.1
   ```
   - Alabasta
   ```
   auto eth0
   iface eth0 inet static
      address 192.198.1.3
      netmask 255.255.255.0
      gateway 192.198.1.1
   ```
   - EniesLobby
   ```
   auto eth0
   iface eth0 inet static
      address 192.198.2.2
      netmask 255.255.255.0
      gateway 192.198.2.1
   ```
   - Water7
   ```
   auto eth0
   iface eth0 inet static
      address 192.198.2.3
      netmask 255.255.255.0
      gateway 192.198.2.1
   ```
   - Skypie
   ```
   auto eth0
   iface eth0 inet static
      address 192.198.2.4
      netmask 255.255.255.0
      gateway 192.198.2.1
   ```
   - Tottoland
   ```
   auto eth0
   iface eth0 inet dhcp
   ```
   - Jipangu
   ```
   auto eth0
   iface eth0 inet static
	address 192.198.2.4
	netmask 255.255.255.0
	gateway 192.198.2.1
   ```
4. Setelah itu restart semua node
5. Kemudian, masukkan `iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 192.198.0.0/16` pada Foosha
6. Lalu masukkan `echo nameserver 192.168.122.1 > /etc/resolv.conf` pada semua console node.
7. Kemudian test ping google.com

### Instalasi
- Foosha

Buat file script.sh kemudian isikan perintah berikut.
```
ip a
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 192.186.0.0/16
cat /etc/resolv.conf
echo nameserver 192.168.122.1 > /etc/resolv.conf
apt-get update
apt-get install isc-dhcp-relay -y

echo '# What servers should the DHCP relay forward requests to?
SERVERS="192.198.2.4"

# On what interfaces should the DHCP relay (dhrelay) serve DHCP requests?
INTERFACES="eth1 eth2 eth3"

# Additional options that are passed to the DHCP relay daemon?
OPTIONS="" '> /etc/default/isc-dhcp-relay

echo 'net.ipv4.ip_forward=1'>/etc/sysctl.conf

service isc-dhcp-relay restart
```
- EniesLobby

Buat file script.sh kemudian isikan perintah berikut.
```
echo ‘nameserver 192.168.122.1’ > /etc/resolv.conf
apt-get update 
apt-get install bind9 -y
```

- Jipangu 

Buat file script.sh kemudian isikan perintah berikut.
```
echo 'nameserver 192.168.122.1' >  /etc/resolv.conf
apt-get update
apt-get install isc-dhcp-server -y
```

- Water7

Buat file script.sh kemudian isikan perintah berikut.
```
echo 'nameserver 192.168.122.1' >  /etc/resolv.conf
apt-get update
apt-get install squid -y
```

## Nomor 3,4,5,6,7
Ada beberapa kriteria yang ingin dibuat oleh Luffy dan Zoro, yaitu:
Semua client yang ada HARUS menggunakan konfigurasi IP dari DHCP Server.
Client yang melalui Switch1 mendapatkan range IP dari [prefix IP].1.20 - [prefix IP].1.99 dan [prefix IP].1.150 - [prefix IP].1.169 (3)
Client yang melalui Switch3 mendapatkan range IP dari [prefix IP].3.30 - [prefix IP].3.50 (4)
Client mendapatkan DNS dari EniesLobby dan client dapat terhubung dengan internet melalui DNS tersebut. (5)
Lama waktu DHCP server meminjamkan alamat IP kepada Client yang melalui Switch1 selama 6 menit sedangkan pada client yang melalui Switch3 selama 12 menit. Dengan waktu maksimal yang dialokasikan untuk peminjaman alamat IP selama 120 menit. (6)
	
Luffy dan Zoro berencana menjadikan Skypie sebagai server untuk jual beli kapal yang dimilikinya dengan alamat IP yang tetap dengan IP [prefix IP].3.69 (7). Loguetown digunakan sebagai client Proxy agar transaksi jual beli dapat terjamin keamanannya, juga untuk mencegah kebocoran data transaksi.


Pada node Jipangu 
buka file dengan perintah `vi /etc/default/isc-dhcp-server` kemudian edit file dengan menambahkan 
```
INTERFACES="eth0"
```
kemudian buka file lain dengan perintah `vi /etc/dhcp/dhcpd.conf` dan edit seperti dibawah ini
```
ddns-update-style none;

# option definitions common to all supported networks...
option domain-name "example.org";
option domain-name-servers ns1.example.org, ns2.example.org;

default-lease-time 600;
max-lease-time 7200;

# If this DHCP server is the official DHCP server for the local
# network, the authoritative directive should be uncommented.
#authoritative;

# Use this to send dhcp log messages to a different log file (you also
# have to hack syslog.conf to complete the redirection).
log-facility local7;

subnet 192.198.2.0 netmask 255.255.255.0 {
}

subnet 192.198.1.0 netmask 255.255.255.0 {
	range 192.198.1.20 192.198.1.99;
	range 192.198.1.150 192.198.1.169;
	option routers 192.198.1.1;
	option broadcast-address 192.198.1.255;
	option domain-name-servers 192.198.2.2, 192.198.122.1;
	default-lease-time 360;
	max-lease-time 7200;
}

subnet 192.198.3.0 netmask 255.255.255.0 {
	range 192.198.3.30 192.198.3.50;
	option routers 192.198.3.1;
	option broadcast-address 192.198.3.255;
	option domain-name-servers 192.198.2.2;
	default-lease-time 720;
	max-lease-time 7200;
}

# konfigurasi untuk no 7
host Skypie {
    hardware ethernet 56:c5:59:f7:95:d0;
    fixed-address 192.186.3.69;
}
```

Kemudian restart dhcp server dengan perintah `service isc-dhcp-server restart`

Pada node EniesLobby 
Buka file /etc/bind/named.conf.options dan tambahkan forwarders
```
forwarders {
       	192.168.122.1;
};

```

Lalu start bind dengan perintah `service bind9 restart`


## Nomor 8
Pada Loguetown, proxy harus bisa diakses dengan nama jualbelikapal.yyy.com dengan port yang digunakan adalah 5000.
1.**Pada Node EnniesLobby**

```vi /etc/bind/named.conf.local, kemudian tambahkan konfigurasi sebagai berikut:
zone "jualbelikapal.D13.com" {
	type master;
	file "/etc/bind/jarkom/jualbelikapal.D13.com";
};
```

Kemudian membuat directory folder jarkom di dalam `/etc/bind` pada enieslobby dengan perintah `mkdir /etc/bind/jarkom` , setelah itu Copykan file db.local pada path /etc/bind ke dalam folder jarkom yang baru saja dibuat dan ubah namanya menjadi *jualbelikapal.D13.com* dengan perintah `cp /etc/bind/db.local /etc/bind/jarkom/jualbelikapal.D13.com.` Selanjutnya buka file jualbelikapal.D13.com dengan `vi /etc/bind/jarkom/jualbelikapal.D13.com` dan lakukan pengeditan seperti gambar berikut :
```$TTL    604800
@       IN      SOA     super.franky.D13.com. root.super.franky.D13.com. (
                        2021100401      ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       		IN      	NS      		super.franky.D13.com.
@      		 IN      	A       		192.198.2.3
WWW		IN	CNAME	 super.franky.D13.com.
```
setelah itu lakukan restart bind9 dengan perintah service bind9 restart

2.**Pada Node Water7**
`vi /etc/squid/squid.conf` lalu tambahkan konfigurasi berikut,
http_port 5000
visible_hostname jualbelikapal.D13.com
Kemudian Restart squid dengan cara mengetikkan perintah: service squid restart

3.**Pada Node Loguetown**
`ping jualbelikapal.D13.com` untuk mengecek konfigurasi pembuatan domain, Lalu lakukan konfigurasi proxy dengan mengaktifkan proxy sebagai berikut **export http_proxy="http://ip-proxy-server:port"**. Menggunakan ip `export http_proxy="http://192.198.2.3:5000"` dan menggunakan domain `export http_proxy="http://jualbelikapal.D13.com:5000"` . Adapun untuk memeriksa apakah konfigurasi proxy pada client berhasil, silahkan lakukan perintah berikut `env | grep -i proxy`. Berikut merupakan hasil gambar pengecekannya:

## Nomor 9
Agar transaksi jual beli lebih aman dan pengguna website ada dua orang, proxy dipasang autentikasi user proxy dengan enkripsi MD5 dengan dua username, yaitu luffybelikapalyyy dengan password luffy_yyy dan zorobelikapalyyy dengan password zoro_yyy (9). 
1.**Pada Node Water7** membuat user & password dengan enskripsi md5 dengan dua username
>> username luffybelikapalD13 -> pass: luffy_D13
`htpasswd -cbmm /etc/squid/passwd luffybelikapalD13 luffy_D13` 

>> username zorobelikapalD13 -> pass: zoro_D13
`htpasswd -bm /etc/squid/passwd zorobelikapalD13 zoro_D13` 
Setelah itu melakukan edit file dengan `vi /etc/squid/squid.conf` dan tambahkan konfigurasi berikut
```auth_param basic program /usr/lib/squid/basic_ncsa_auth /etc/squid/passwd
auth_param basic children 5
auth_param basic realm Proxy
auth_param basic credentialsttl 2 hours
auth_param basic casesensitive on
acl USERS proxy_auth REQUIRED
http_access allow USERS
```
dan Restart squid dengan `service squid restart`

## Nomor 10
Transaksi jual beli tidak dilakukan setiap hari, oleh karena itu akses internet dibatasi hanya dapat diakses setiap hari Senin-Kamis pukul 07.00-11.00 dan setiap hari Selasa-Jum’at pukul 17.00-03.00 keesokan harinya (sampai Sabtu pukul 03.00) (10).

## Nomor 11
Agar transaksi bisa lebih fokus berjalan, maka dilakukan redirect website agar mudah mengingat website transaksi jual beli kapal. Setiap mengakses google.com, akan diredirect menuju super.franky.yyy.com dengan website yang sama pada soal shift modul 2. Web server super.franky.yyy.com berada pada node Skypie (11).

## Nomor 12
Saatnya berlayar! Luffy dan Zoro akhirnya memutuskan untuk berlayar untuk mencari harta karun di super.franky.yyy.com. Tugas pencarian dibagi menjadi dua misi, Luffy bertugas untuk mendapatkan gambar (.png, .jpg), sedangkan Zoro mendapatkan sisanya. Karena Luffy orangnya sangat teliti untuk mencari harta karun, ketika ia berhasil mendapatkan gambar, ia mendapatkan gambar dan melihatnya dengan kecepatan 10 kbps (12). 


## Nomor 13
Sedangkan, Zoro yang sangat bersemangat untuk mencari harta karun, sehingga kecepatan kapal Zoro tidak dibatasi ketika sudah mendapatkan harta yang diinginkannya (13).



Keterangan :
yyy adalah nama kelompok Anda
Untuk nomor 9, harus htpasswd yang melakukan enkripsi
Bisa melakukan wget https://raw.githubusercontent.com/FeinardSlim/Praktikum-Modul-2-Jarkom/main/super.franky.zip untuk mendapatkan file untuk super.franky.yyy.com
