# Jarkom-Modul-3-E07-2021
Lapres Jaringan Komputer Modul 3 Kelompok E07

Buat topologi seperti di soal. Kemudian, konfigurasikan terlebih dahulu network interface masing-masing host.
Pada router foosha, 
```
auto eth0
iface eth0 inet dhcp

auto eth1
iface eth1 inet static
	address 192.203.1.1
	netmask 255.255.255.0

auto eth2
iface eth2 inet static
	address 192.203.2.1
	netmask 255.255.255.0

auto eth3
iface eth3 inet static
	address 192.203.3.1
	netmask 255.255.255.0
```

Pada host Loguetown, Alabasta, Skypie, TottoLand
```
auto eth0
iface eth0 inet dhcp
```

Khusus untuk Skypie, tambahkan konfigurasi berikut di bawah konfigurasi sebelumnya,
```
hwaddress ether 9e:9d:1c:75:bc:db
```

## Soal 2
- Pada router foosha, konfigurasi file di bawah ini :
```
nano /etc/sysctl.conf
```

- Lalu uncomment bagian ``net.ipv4.ip_forward=1``
- Install dhcp relay,
```
apt-get update
apt-get install isc-dhcp-relay -y
```
- Ketika diminta untuk input sesuatu saat proses instalasi, cukup skip dengan tekan Enter sampai prosesi selesai

- Buka file di bawah ini,
```
nano /etc/default/isc-dhcp-relay
```

- Edit pada bagian ini ,
```
# What servers should the DHCP relay forward requests to?
SERVERS="192.203.2.4"

# On what interfaces should the DHCP relay (dhrelay) serve DHCP requests?
INTERFACES="eth1 eth2 eth3"
```

- Restart/start dhcp relay dengan command, 
```
service isc-dhcp-relay start
```

## Soal 3,4,5,6,7
- Install tools terlebih dahulu, 
```
apt-get update
apt-get install isc-dhcp-server -y
```

- Buka file berikut ini,
```
nano /etc/default/isc-dhcp-server
```
- Lalu edit seperti gambar berikut,

![alt text](https://github.com/migellamp/Jarkom-Modul-3-E07-2021/blob/main/images/3-7(a).png) <br />

- Buka file berikut ini,
```
nano /etc/dhcp/dhcpd.conf
```
- Lalu tambahkan konfigurasi BERIKUT INI di bagian paling bawah file, 
```
subnet 192.203.2.0 netmask 255.255.255.0 {
        option routers 192.203.2.1;		#SETTING SUPAYA BISA RELAY KE FOOSHA
        default-lease-time 360;
        max-lease-time 7200;
}
subnet 192.203.1.0 netmask 255.255.255.0 {
    range 192.203.1.20 192.203.1.99;		#no 3
    range 192.203.1.150 192.203.1.169;		#no 3
    option routers 192.203.1.1;
    option broadcast-address 192.203.1.255;
    option domain-name-servers 192.203.2.2;	#no 5
    default-lease-time 360;			#no 6
    max-lease-time 7200;			#no 6
}
subnet 192.203.3.0 netmask 255.255.255.0 {
    range 192.203.3.30 192.203.3.50;		#no 4
    option routers 192.203.3.1;
    option broadcast-address 192.203.3.255;
    option domain-name-servers 192.203.2.2;	#no 5
    default-lease-time 720;			#no 6
    max-lease-time 7200;			#no 6
}	
host Skypie {					#konfig no 7
    hardware ethernet 9e:9d:1c:75:bc:db;
    fixed-address 192.203.3.69;
}
```

- Restart dhcp,
```
service isc-dhcp-server restart
```

- Testing pada masing-masing host dengan cara cek ip ``ip a`` serta cek koneksi internet ``ping google.com``


## Soal 08: Loguetown digunakan sebagai client Proxy agar transaksi jual beli dapat terjamin keamanannya, juga untuk mencegah kebocoran data transaksi. Pada Loguetown, proxy harus bisa diakses dengan nama jualbelikapal.e07.com dengan port yang digunakan adalah 5000. 

- Pertama setting DNS terlebih dahulu sehingga dns jualbelikapal.e07.com bisa menuju ip Water7 (192.203.2.3). Setting konfigurasi seperti berikut:
```
nano /etc/bind/kaizoku/e07.com
```
dan tambahkan:
```
;
; BIND data file for local loopback interface
;
$TTL 604800
@ IN SOA e07.com. root.e07.com. (
        2021102401      ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@               IN NS e07.com.
@               IN A 192.203.2.3
jualbelikapal   IN A 192.203.2.3
```
lalu pada: ``` nano /etc/bind/named.conf.local ```
tambahkan:
```
zone "e07.com" {
    type master;
    file "/etc/bind/kaizoku/e07.com";
};
```
Pindah pada node Water7, jalankan konfigurasi seperti berikut:
```
echo nameserver 192.203.2.2 > /etc/resolv.conf
apt-get update
apt-get install squid -y
service squid restart
service squid status
mv /etc/squid/squid.conf /etc/squid/squid.conf.bak
nano /etc/squid/squid.conf
```
lalu tambahkan konfigurasi sebagai berikut pada ../squid.conf:
```
http_port 5000
visible_hostname Water7
http_access allow all
```
lalu restart squid: ``` service squid restart ```
Pindah pada node Loguetown, jalankan konfigurasi seperti berikut:
```
export http_proxy="http://jualbelikapal.e07.com:5000"
env | grep -i proxy
```
Dan coba buka alamat web menggunakan browser lynx: ``` lynx http://its.ac.id ```
Hasilnya sebagai berikut:
![alt text](https://github.com/migellamp/Jarkom-Modul-3-E07-2021/blob/main/images/08a-Fix.png) <br />
![alt text](https://github.com/migellamp/Jarkom-Modul-3-E07-2021/blob/main/images/08b-Fix.png) <br />

## Soal 09: Agar transaksi jual beli lebih aman dan pengguna website ada dua orang, proxy dipasang autentikasi user proxy dengan enkripsi MD5 dengan dua username, yaitu luffybelikapale07 dengan password luffy_e07 dan zorobelikapale07 dengan password zoro_e07.

Pada node Water7 jalankan perintah berikut:
```
apt-get update
apt-get install apache2-utils -y
htpasswd -b -c -m /etc/squid/passwd luffybelikapale07 luffy_e07
htpasswd -b -m /etc/squid/passwd zorobelikapale07 zoro_e07
```
Perintah htpasswd berfungsi untuk membuat user denga password pada file ../passwd, flag -c berfungsi untuk menandakan "create" pembuatan awal file dan flag -m untuk menandakan password disimpan dengan enkripsi md5.

Lalu edit file ../squid.conf dan ubah konfigurasi.
```
mv /etc/squid/squid.conf /etc/squid/squid.conf.bak
nano /etc/squid/squid.conf
```

Konfigurasi menjadi
```
http_port 5000
visible_hostname Water7

auth_param basic program /usr/lib/squid/basic_ncsa_auth /etc/squid/passwd
auth_param basic children 5
auth_param basic realm Proxy
auth_param basic credentialsttl 2 hours
auth_param basic casesensitive on
acl USERS proxy_auth REQUIRED
http_access allow USERS
```
Lalu restart squid: ``` service squid restart ```

Cek apakah user dan password sudah dibuat dengan command: ``` cat /etc/squid/passwd ```
Hasil
![alt text](https://github.com/migellamp/Jarkom-Modul-3-E07-2021/blob/main/images/09a.png) <br />
Maka pada Loguetown, ketika kita akan membuka suatu situs dengan lynx. Juga akan dimintai username dan password proxy.

Hasil
![alt text](https://github.com/migellamp/Jarkom-Modul-3-E07-2021/blob/main/images/09b.png) <br />

## Soal 10: Transaksi jual beli tidak dilakukan setiap hari, oleh karena itu akses internet dibatasi hanya dapat diakses setiap hari Senin-Kamis pukul 07.00-11.00 dan setiap hari Selasa-Jum’at pukul 17.00-03.00 keesokan harinya (sampai Sabtu pukul 03.00).

Buka file ../acl.conf
```
nano /etc/squid/acl.conf
```
Lalu tambahkan konfigurasi sebagai berikut. Untuk mengatur waktu yang diperbolehkan oleh proxy server (sesuai dengan soal):
```
acl AVAILABLE_WORKING1 time MTWH 07:00-11:00
acl AVAILABLE_WORKING2 time TWHF 17:00-23:59
acl AVAILABLE_WORKING3 time WHFA 00:00-03:00
```
Lalu buka file ../squid.conf: ``` nano /etc/squid/squid.conf ```
Dan tambahkan konfigurasi berikut ini:
```
include /etc/squid/acl.conf
http_port 5000
visible_hostname Water7

auth_param basic program /usr/lib/squid/basic_ncsa_auth /etc/squid/passwd
auth_param basic children 5
auth_param basic realm Proxy
auth_param basic credentialsttl 2 hours
auth_param basic casesensitive on
acl USERS proxy_auth REQUIRED
http_access allow USERS AVAILABLE_WORKING1
http_access allow USERS AVAILABLE_WORKING2
http_access allow USERS AVAILABLE_WORKING3
http_access deny all
```
Lalu restart squid: ``` service squid restart ```


## Soal 11 : Melakukan redirect website. Setiap mengakses google.com, akan diredirect menuju super.franky.e07.com

- Pertama, melakukan setup pada node skypie yang mana sama seperti soal-shift modul 2 kemaren pada no 11
- Kedua, Setelah super.franky.e07.com dapat dilakukan ping pada node Loguetown, maka proses pertama selesai
- Selanjutnya, memasukkkan query pada squid.conf yang berguna agar nantinya meredirect google.com ke super.franky
```
	acl badsites dstdomain google.com
	deny_info http://super.franky.e07.com badsites
	deny_info ERR_ACCESS_DENIED all
	http_access deny badsites
```
- Selanjutnya, jangan lupa untuk melakukan restart ```service squid restart```
- langkah Berikutnya, masuk ke node Loguetown dan aktifkan Proxy dengan menggunakan ```export http_proxy="http://jualbelikapal.e07.com:5000"``` dan ```env | grep -i proxy```
- Lalu setelah, proxy dijalankan. Langsung test dengan menjalankan ```lynx google.com``` maka nanti otomatis akan diarahkan ke super.franky.e07.com
- Langkah Selanjutnya tinggal di jalankan pada node Loguetown  dengan cara:
  1. export http_proxy="http://jualbelikapal.e07.com:5000"
  2. env | grep -i proxy
  3. lynx google.com

Output: <br/>
![alt text](https://github.com/migellamp/Jarkom-Modul-3-E07-2021/blob/main/images/11a.jpg) <br />
![alt text](https://github.com/migellamp/Jarkom-Modul-3-E07-2021/blob/main/images/11b.jpg) <br />
![alt text](https://github.com/migellamp/Jarkom-Modul-3-E07-2021/blob/main/images/11c.jpg) <br />

Note : Dapat dilihat bahwa, saat mengunjungi google.com maka akan langsung diteruskan ke super.franky.e07

## Soal 12 : Membatasi kecepatan User Luffy ketika mendownload gambar dengan kecepatan 10kbps

- Pertama, Membuat sebuah file configurasi bernama ```acl-bandwidth.conf``` dan memasukkan query berikut
```
	auth_param basic program /usr/lib/squid/basic_ncsa_auth /etc/squid/passwd
	acl LUFFY proxy_auth luffybelikapale07
	acl ZORO  proxy_auth zorobelikapale07 -> untuk no 13
	acl images url_regex -i \.jpg$ \.png$
	delay_pools 1
	delay_class 1 1
	delay_access 1 allow LUFFY
	delay_access 1 deny ZORO -> untuk no 13
	delay_access 1 allow images
	delay_parameters 1 1250/1250
```
Keterangan: <br/>
	1. ```auth_param basic program /usr/lib/squid/basic_ncsa_auth /etc/squid/passwd``` -> membaca password dari user yang dibuat <br/>
	2. ```acl LUFFY proxy_auth luffybelikapale07``` -> menamai username luffybelikapale07 dengan variable LUFFY <br/>
	3. ```delay_access 1 allow LUFFY``` -> menerapkan konfigurasi yang telah dibuat <br/>
	4. ```acl images url_regex -i \.jpg$ \.png$``` -> menggunakan regex untuk mencari file dengan format dicari <br/>
	5. ```delay_parameters 1 1250/1250``` -> digunakan untuk membatasi kecepatan yaitu 10kbps <br/>

- Selanjutnya, Mengedit file ```squid.conf``` dengan memasukkan ```include /etc/squid/acl-bandwidth.conf``` agar file configurasi diatas dapat terbaca pada proxy
- Langkah Selanjutnya tinggal di jalankan pada node Loguetown  dengan cara:
  1. export http_proxy="http://jualbelikapal.e07.com:5000"
  2. env | grep -i proxy
  3. lynx google.com
  4. Login dengan menggunakan username LUFFY
  5. Mencoba mendownload file gambar pada asset yang terdapat pada web super.franky.e07

Output: <br/>
![alt text](https://github.com/migellamp/Jarkom-Modul-3-E07-2021/blob/main/images/12.jpg) <br />

Note : Dapat dilihat bahwa, saat akan mendownload file dengan format .jpg/.png maka kecepatan LUFFY dibatasi

## Soal 13 : Zoro bebas mendownload apa saja dengan kecepatan yang bebas, tidak dibatasi

- Pertama, Menambahkan sebuah query pada file configurasi yaitu  ```acl-bandwidth.conf```
```
	auth_param basic program /usr/lib/squid/basic_ncsa_auth /etc/squid/passwd
	acl LUFFY proxy_auth luffybelikapale07
	acl ZORO  proxy_auth zorobelikapale07 -> untuk no 13
	acl images url_regex -i \.jpg$ \.png$
	delay_pools 1
	delay_class 1 1
	delay_access 1 allow LUFFY
	delay_access 1 deny ZORO -> untuk no 13
	delay_access 1 allow images
	delay_parameters 1 1250/1250
```
Keterangan:  <br/>
	1. ```acl ZORO  proxy_auth zorobelikapale07``` -> menamai username zorobelikapale07 dengan variable ZORO  <br/>
	2. ```delay_access 1 deny ZORO``` -> tidak menerapkan delay_parameters dan ketentuan file yang dibuat pada file ```acl-bandwidth.conf``` sehingga tidak berpengaruh pada 	user zoro  <br/>

- Selanjut, Mengedit file ```squid.conf``` dengan memasukkan ```include /etc/squid/acl-bandwidth.conf``` agar file configurasi diatas dapat terbaca pada proxy
- Langkah Selanjutnya tinggal di jalankan pada node Loguetown  dengan cara:
  	1. export http_proxy="http://jualbelikapal.e07.com:5000"
 	2. env | grep -i proxy
  	3. lynx google.com
 	4. Login menggunakan username ZORO
  	5. Mencoba mendownload file gambar pada asset yang terdapat pada web super.franky.e07

Output: <br/>
![alt text](https://github.com/migellamp/Jarkom-Modul-3-E07-2021/blob/main/images/13.jpg) <br />

Note : Dapat dilihat bahwa, saat ZORO akan mendownload file maka kecepatannya tidak dibatasi dan langsung selesai mendownloadn
