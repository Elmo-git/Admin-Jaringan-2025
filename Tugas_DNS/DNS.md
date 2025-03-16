# Konfigurasi DNS untuk Jaringan Internal

## 1. Cek Status BIND

![bindstatus](/Tugas_DNS/img/bindstatus.png)

```sh
systemctl status bind9
```
**Analisis:**
Perintah ini digunakan untuk memeriksa apakah layanan BIND sedang berjalan atau tidak.

## 2. Restart BIND untuk Menerapkan Perubahan
```sh
systemctl restart bind9
```
**Analisis:**
Jika ada perubahan dalam konfigurasi DNS, layanan BIND perlu di-restart agar perubahan tersebut diterapkan.

## 3. Konfigurasi Zona DNS di `named.conf.local`

![namedconflocal](/Tugas_DNS/img/namedconflocal.png)

```sh
vi /etc/bind/named.conf.local
```
Tambahkan zona berikut:
```
zone "example.com" {
    type master;
    file "/etc/bind/db.example.com";
};

zone "100.168.192.in-addr.arpa" {
    type master;
    file "/etc/bind/db.192";
};
```
**Analisis:**
Konfigurasi ini mendefinisikan zona forward dan reverse lookup untuk jaringan internal.

## 4. Buat File Forward Zone

![dbexample](/Tugas_DNS/img/dbexample.png)

```sh
vi /etc/bind/db.example.com
```
Isi file:
```
$TTL 86400
@   IN  SOA  ns1.example.com. root.example.com. (
            2024031501
            3600
            1800
            604800
            86400 )

@   IN  NS  ns1.example.com.
ns1 IN  A   192.168.100.57
www IN  A   192.168.100.57
```
**Analisis:**
File ini mendefinisikan alamat IP untuk `ns1` dan `www` di dalam jaringan internal.

## 5. Buat File Reverse Zone

![db192](/Tugas_DNS/img/db192.png)

```sh
vi /etc/bind/db.192
```
Isi file:
```
$TTL 86400
@   IN  SOA  ns1.example.com. root.example.com. (
            2024031501
            3600
            1800
            604800
            86400 )

@   IN  NS  ns1.example.com.
57  IN  PTR ns1.example.com.
57  IN  PTR www.example.com.
```
**Analisis:**
File ini digunakan untuk mengonversi IP ke hostname dalam reverse DNS lookup.

## 6. Uji Konfigurasi

![testconf](/Tugas_DNS/img/testconf.png)

```sh
sudo named-checkconf
sudo named-checkzone example.com /etc/bind/db.example.com
sudo named-checkzone 100.168.192.in-addr.arpa /etc/bind/db.192
```
**Analisis:**
Perintah ini memverifikasi apakah konfigurasi BIND sudah benar sebelum dijalankan.

## 7. Uji Query Forward

![testforward](/Tugas_DNS/img/testforward.png)

```sh
dig example.com @192.168.100.57
```
**Analisis:**
Perintah ini menguji apakah DNS dapat mengonversi domain `example.com` ke alamat IP dengan benar.

## 8. Uji Query Reverse

![testreversed](/Tugas_DNS/img/testreversed.png)

```sh
dig -x 192.168.100.57 @192.168.100.57
```
**Analisis:**
Perintah ini menguji apakah alamat IP dapat di-resolve kembali menjadi nama domain dengan benar.

## Kesimpulan
Konfigurasi ini memastikan bahwa BIND dapat bekerja dengan baik sebagai DNS internal. Jika ada masalah, periksa kembali izin akses, firewall, dan log sistem dengan `journalctl -xe | grep named`.
