# Jarkom-Modul-3-E07-2021
Lapres Jaringan Komputer Modul 3 Kelompok E07

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
![alt text](https://github.com/migellamp/Jarkom-Modul-2-E07-2021/blob/main/images/11a.png) <br />

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
Keterangan:
	1. ```auth_param basic program /usr/lib/squid/basic_ncsa_auth /etc/squid/passwd``` -> membaca password dari user yang dibuat
	2. ```acl LUFFY proxy_auth luffybelikapale07``` -> menamai username luffybelikapale07 dengan variable LUFFY
	3. ```delay_access 1 allow LUFFY``` -> menerapkan konfigurasi yang telah dibuat
	4. ```acl images url_regex -i \.jpg$ \.png$``` -> menggunakan regex untuk mencari file dengan format dicari
	5. ```delay_parameters 1 1250/1250``` -> digunakan untuk membatasi kecepatan yaitu 10kbps

- Selanjutnya, Mengedit file ```squid.conf``` dengan memasukkan ```include /etc/squid/acl-bandwidth.conf``` agar file configurasi diatas dapat terbaca pada proxy
- Langkah Selanjutnya tinggal di jalankan pada node Loguetown  dengan cara:
  1. export http_proxy="http://jualbelikapal.e07.com:5000"
  2. env | grep -i proxy
  3. lynx google.com
  4. Login dengan menggunakan username LUFFY
  5. Mencoba mendownload file gambar pada asset yang terdapat pada web super.franky.e07

Output: <br/>
![alt text](https://github.com/migellamp/Jarkom-Modul-2-E07-2021/blob/main/images/11a.png) <br />

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
Keterangan:
	1. ```acl ZORO  proxy_auth zorobelikapale07``` -> menamai username zorobelikapale07 dengan variable ZORO
	2. ```delay_access 1 deny ZORO``` -> tidak menerapkan delay_parameters dan ketentuan file yang dibuat pada file ```acl-bandwidth.conf``` sehingga tidak berpengaruh pada 	user zoro

- Selanjut, Mengedit file ```squid.conf``` dengan memasukkan ```include /etc/squid/acl-bandwidth.conf``` agar file configurasi diatas dapat terbaca pada proxy
- Langkah Selanjutnya tinggal di jalankan pada node Loguetown  dengan cara:
  	1. export http_proxy="http://jualbelikapal.e07.com:5000"
 	2. env | grep -i proxy
  	3. lynx google.com
 	4. Login menggunakan username ZORO
  	5. Mencoba mendownload file gambar pada asset yang terdapat pada web super.franky.e07

Output: <br/>
![alt text](https://github.com/migellamp/Jarkom-Modul-2-E07-2021/blob/main/images/11a.png) <br />
