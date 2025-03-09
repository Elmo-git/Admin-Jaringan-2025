# **Laporan Instalasi dan Konfigurasi NTP Client serta Samba di Debian 11**

## **1. Instalasi NTP Client**

### **1.1 Instalasi NTP Client**
1. Update package list dan install paket NTP:
   ```bash
   sudo apt update
   sudo apt install ntp ntpsec -y
   ```
2. Edit file konfigurasi NTP untuk mengarah ke server NTP Indonesia:
   ```bash
   sudo nano /etc/ntp.conf
   ```
   Tambahkan atau ubah baris berikut:

    ![ipool](/Tugas_Samba/img/ipool.png)

   ```
   server 0.id.pool.ntp.org iburst
   server 1.id.pool.ntp.org iburst
   server 2.id.pool.ntp.org iburst
   server 3.id.pool.ntp.org iburst
   ```
   
3. Restart layanan NTP:
   ```bash
   sudo systemctl restart ntp
   ```

### **1.2 Verifikasi NTP Client**
- Cek status layanan:

   ![ntpstatus](/Tugas_Samba/img/ntpstatus.png)

  ```bash
  sudo systemctl status ntp
  ```
- Pastikan sinkronisasi waktu:

   ![ntpstatus](/Tugas_Samba/img/ntpstatus.png)

  ```bash
  ntpq -p
  ```
  Jika berhasil, akan muncul daftar server NTP yang terhubung.

---

## **2. Instalasi dan Konfigurasi Samba**

### **2.1 Instalasi Samba**
1. Update package list dan install paket Samba:
   ```bash
   sudo apt update
   sudo apt install samba smbclient cifs-utils -y
   ```
2. Pastikan Samba berjalan:

   ![smbdstatus](/Tugas_Samba/img/smbdstatus.png)

   ```bash
   sudo systemctl status smbd
   ```

### **2.2 Konfigurasi Public Shared Folder**
1. Buat direktori share:

   ![mkdirsmb](/Tugas_Samba/img/mkdirsmb.png)

   ```bash
   sudo mkdir -p /srv/samba/Public
   sudo chmod 777 /srv/samba/Public
   ```
2. Edit file konfigurasi Samba:
   ```bash
   sudo nano /etc/samba/smb.conf
   ```
   Tambahkan konfigurasi berikut di bagian bawah:

   ![smbconf](/Tugas_Samba/img/smbconf.png)

   ```
   [Public]
       path = /srv/samba/Public
       browseable = yes
       writable = yes
       guest ok = yes
       create mask = 0777
       directory mask = 0777
   ```
3. Restart layanan Samba:
   ```bash
   sudo systemctl restart smbd
   ```

### **2.3 Verifikasi Akses Samba**
1. Cek apakah share tersedia:
   ```bash
   smbclient -L //192.168.100.57 -U vboxuser
   ```

   ![smbwsl](/Tugas_Samba/img/smbwslch.png)

   Jika berhasil, akan muncul daftar share folder.
2. Coba akses folder Public:
   ```bash
   smbclient //192.168.100.57/Public -U vboxuser
   ```
   ![smbpub](/Tugas_Samba/img/smbpub.png)

   Jika berhasil, prompt akan berubah menjadi `smb: \>`.
3. Uji akses dari Windows:
   - Buka **Run (Win + R)**
   - Ketik `\\192.168.100.57\Public`
   - Jika folder terbuka, maka Samba berhasil dikonfigurasi.

   ![smbwR](/Tugas_Samba/img/smbwR.png)

---

## **3. Kesimpulan**
Dari percobaan yang telah dilakukan:
- **NTP Client** berhasil dikonfigurasi dengan server NTP Indonesia.
- **Samba** berhasil dikonfigurasi dengan Public Shared Folder yang bisa diakses dari WSL dan Windows.

