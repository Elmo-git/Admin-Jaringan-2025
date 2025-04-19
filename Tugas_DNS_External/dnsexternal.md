# Laporan Konfigurasi External Zone di BIND9

## 1. Persiapan Awal

### a. Instalasi BIND9
Pastikan BIND9 sudah terinstal di sistem:
```bash
sudo apt update
sudo apt install bind9 bind9utils bind9-doc dnsutils
```

### b. Cek IP Server DNS
Contoh:
```bash
ip a
```
Hasil: 

![ipdns](/img/ipdns.png)

ip tersebut dipakai di interface internal network

## 2. Konfigurasi Forward Zone (kelompok10.com)

### a. Edit file `/etc/bind/named.conf`
Tambahkan konfigurasi external:
```bash
include "/etc/bind/named.conf.external-zones";
```

Hasil:

![namedconf](/img/namedconf.png)

### b. Konfigurasi `/etc/bind/named.conf.external-zones`

```bash
zone "kelompok10.com" IN {
    type master;
    file "/etc/bind/kelompok10.com";
    allow-update { none; };
};

zone "19.200.168.192.in-addr.arpa" IN {
    type master;
    file "/etc/bind/19.200.168.192.db";
    allow-update { none; };
};
```

### c. Buat file zone baru
Copy dari template default:
```bash
sudo cp /etc/bind/db.local /etc/bind/kelompok10.com
```

### c. Edit file `/etc/bind/kelompok10.com`
Contoh isinya:

![kelompok10.com](/img/kel10com.png)

## 3. Konfigurasi Reverse Zone (192.168.200.19)

### a. Buat file reverse zone
```bash
sudo cp /etc/bind/db.127 /etc/bind/19.200.168.192.db
```

### b. Edit file `/etc/bind/19.200.168.192.db`

Menjadi seperti ini:

![konfigurasirevzone](/img/revzone.png)

## 4. Konfigurasi Options (Opsional)

### a. Edit `/etc/bind/named.conf.options`
Pastikan block `options` seperti ini:
```bash
options {
    directory "/var/cache/bind";
    recursion yes;
    allow-query { any; };
    forwarders {
        8.8.8.8;
        8.8.4.4;
    };
    dnssec-validation auto;
    listen-on { any; };
    listen-on-v6 { any; };
};
```

## 5. Validasi dan Restart

### a. Validasi file konfigurasi
```bash
sudo named-checkconf
sudo named-checkzone kelompok10.com /etc/bind/db.kelompok10.com
sudo named-checkzone 200.168.192.in-addr.arpa /etc/bind/19.200.168.192.db
```

### b. Restart BIND9
```bash
sudo systemctl restart bind9
```

## 6. Pengujian

### a. Dari client, pastikan nameserver diarahkan ke VM DNS:
Edit `/etc/resolv.conf`:
```bash
nameserver 192.168.200.19
```

### b. Tes resolve nama domain
```bash
dig kelompok10.com
ping kelompok10.com
```

Hasil:

![hasilping](/img/pingclient.png)

### c. Tes reverse lookup
```bash
dig -x 192.168.200.19
```

Hasil:

![hasilrevlookup](/img/hasilreverselookup.png)

## 7. Penutup
Konfigurasi external zone BIND9 pada domain `kelompok10.com` telah selesai. Pastikan seluruh file zona valid, DNS berjalan, dan client dapat resolve baik nama domain maupun PTR (reverse).

---

> Catatan: Gantilah IP dan nama domain sesuai kebutuhan kelompok masing-masing.

