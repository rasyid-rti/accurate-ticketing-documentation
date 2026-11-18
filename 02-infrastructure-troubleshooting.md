# Accurate Ticketing — Infrastructure Troubleshooting

## Informasi Dokumen

- Server: ISASERVER
- Sistem Operasi: Windows 10 + Ubuntu WSL 2
- Lingkungan: Production dan Staging
- Tanggal: 18 November 2026
- Status: Draft

## 1. Tujuan

Dokumen ini merupakan panduan pemeriksaan dan
pemulihan layanan Accurate Ticketing.

Prinsip troubleshooting:
- Periksa sebelum mengubah konfigurasi.
- Utamakan pemeriksaan read-only.
- Jangan mengubah database tanpa prosedur.
- Jangan menghapus Docker volumes.
- Verifikasi hasil setiap tindakan.
- Catat perubahan dan hasil pengujian.

## 2. Arsitektur Akses Production

Domain:
https://ticketing.rigtenders.co.id

Windows:
192.168.1.10:443

Windows Portproxy:
443 ke IP WSL dinamis port 8445

Caddy Production:
0.0.0.0:8445

Nginx Production:
127.0.0.1:8081

## 3. Pemeriksaan Awal

### 3.1 Status Docker
Lihat Bagian 11: Pemeriksaan Docker (Ubuntu WSL).

### 3.2 Status Nginx
Lihat Bagian 11: Pemeriksaan Nginx Production.

### 3.3 Status Caddy
Lihat Bagian 11: Pemeriksaan Caddy Production.

### 3.4 Status Windows Portproxy
Lihat Bagian 11: Pemeriksaan Portproxy Windows.

### 3.5 Status MySQL
Periksa status health container MySQL menggunakan `sudo docker ps`. Jangan mengubah data.

## 4. Incident INC-001 — HTTPS Connection Refused

Tanggal: 18 November 2026

Gejala:
Browser menampilkan ERR_CONNECTION_REFUSED.

Temuan:
- Container Production berjalan.
- Nginx Production menghasilkan HTTP 200.
- Caddy Production menghasilkan HTTP 200.
- Windows port 443 gagal menerima koneksi.
- Aturan Portproxy lama menuju 127.0.0.1:8445.

Tindakan perbaikan:
- Binding Caddy disesuaikan menjadi 0.0.0.0:8445.
- Portproxy diarahkan ke IP WSL port 8445.
- Script recovery diperbarui untuk membaca IP WSL.
- Verifikasi HTTPS ditambahkan ke recovery.

Hasil:
- HTTPS kembali menghasilkan HTTP 200.
- Browser berhasil membuka halaman Production.
- Scheduled Task recovery berhasil diuji.

Catatan:
Penyebab awal gangguan belum dipastikan.
Recovery setelah restart belum diuji.

## 5. Recovery Otomatis Windows–WSL

Scheduled Tasks:
- AccurateTicketing-WSL-Autostart
- AccurateTicketing-WSL-IP-Update
- AccurateTicketing-Portproxy-Recovery

IP WSL:
C:\Users\ISA\AppData\Local\AccurateTicketing\wsl-ip.txt

Script:
C:\ProgramData\AccurateTicketing\repair-portproxy.ps1

Lihat Bagian 12 — Windows WSL Recovery.

## 6. Troubleshooting Docker

Lihat Bagian 13 — Docker Production.

## 7. Troubleshooting Database

Lihat Bagian 14 — MySQL Production.

## 8. Troubleshooting Application

Lihat Bagian 15 — Laravel Backend Production.

## 9. Incident Log

Setiap insiden harus mencatat:
- Tanggal dan waktu
- Gejala
- Dampak
- Hasil diagnosis
- Perubahan yang dilakukan
- Hasil verifikasi
- Status penyelesaian

## 10. Pending Validation

- Recovery setelah restart Windows
- Recovery setelah perubahan IP WSL
- Prosedur rollback Portproxy
- Prosedur restore aplikasi

## 11. Quick Troubleshooting — HTTPS Production

### Pemeriksaan Docker (Ubuntu WSL)

Perintah:

    sudo docker ps

Pastikan container Nginx, Backend, MySQL, dan Caddy berjalan.

### Pemeriksaan Nginx Production

Perintah:

    curl -I http://127.0.0.1:8081/

Hasil normal: HTTP 200.

### Pemeriksaan Caddy Production

Gunakan nama domain untuk TLS/SNI:

    curl -k -I --resolve ticketing.rigtenders.co.id:8445:127.0.0.1 https://ticketing.rigtenders.co.id:8445/

Hasil normal: HTTP 200.

### Pemeriksaan Portproxy Windows

Jalankan di PowerShell Administrator:

    netsh interface portproxy show v4tov4

Portproxy harus mengarah dari 192.168.1.10:443
ke IP WSL terbaru pada port 8445.

### Pemeriksaan Recovery Otomatis

Jalankan di PowerShell Administrator:

    Get-ScheduledTaskInfo -TaskName "AccurateTicketing-Portproxy-Recovery"

Periksa LastTaskResult dan log:

    Get-Content "C:\ProgramData\AccurateTicketing\portproxy-recovery.log" -Tail 10

### Interpretasi

- Nginx gagal: periksa Nginx dan Backend.
- Caddy gagal: periksa Caddy dan Docker networking.
- Keduanya berhasil, domain gagal: periksa Portproxy Windows.
- MySQL unhealthy: periksa database dan log sebelum tindakan.
- Jangan restart atau restore database tanpa diagnosis.

## 12. Runbook — Windows WSL Recovery

### Pemeriksaan IP WSL

Jalankan di Windows PowerShell:

    wsl.exe -d Ubuntu-24.04 -u isa --exec hostname -I

Bandingkan dengan file IP otomatis:

    Get-Content "C:\Users\ISA\AppData\Local\AccurateTicketing\wsl-ip.txt"

IP utama WSL harus sesuai dengan IP pada file.

### Pemeriksaan Scheduled Tasks

Jalankan di PowerShell Administrator:

    Get-ScheduledTaskInfo -TaskName "AccurateTicketing-WSL-IP-Update"

    Get-ScheduledTaskInfo -TaskName "AccurateTicketing-Portproxy-Recovery"

LastTaskResult normal adalah 0.

### Pemeriksaan Portproxy

    netsh interface portproxy show v4tov4

Listen:
192.168.1.10:443

Connect:
IP WSL aktif pada port 8445.

### Pemeriksaan Log

    Get-Content "C:\ProgramData\AccurateTicketing\portproxy-recovery.log" -Tail 10

Pastikan log terakhir menunjukkan SUCCESS dan HTTPS 200.

### Catatan Keamanan

- Jangan menonaktifkan Windows Firewall.
- Jangan menghapus aturan Portproxy tanpa pemeriksaan.
- Jangan me-restart Docker atau MySQL untuk masalah Portproxy.
- Jangan menjalankan restore database untuk masalah HTTPS.
- Recovery setelah Windows restart masih perlu diuji.


## 13. Runbook — Docker Production

### Pemeriksaan Container

Jalankan di Ubuntu WSL ISASERVER:

    sudo docker ps -a

Pastikan container berikut berjalan:

- accurate-ticketing-production-nginx-1
- accurate-ticketing-production-backend-1
- accurate-ticketing-production-mysql-1
- accurate-ticketing-caddy-production-caddy-1

MySQL seharusnya berstatus healthy.

### Pemeriksaan Log

Nginx:

    sudo docker logs --tail 50 accurate-ticketing-production-nginx-1

Backend:

    sudo docker logs --tail 50 accurate-ticketing-production-backend-1

MySQL:

    sudo docker logs --tail 50 accurate-ticketing-production-mysql-1

Caddy:

    sudo docker logs --tail 50 accurate-ticketing-caddy-production-caddy-1

Periksa log secara lokal. Jangan membagikan password,
token, atau data sensitif yang mungkin muncul dalam log.

### Pemeriksaan Network

    sudo docker network ls

Network Production:

- accurate-ticketing-production_web
- accurate-ticketing-production_database

Jangan menghapus atau membuat ulang network Production
tanpa pemeriksaan dependensi container.

### Pemeriksaan Volume

    sudo docker volume ls

Volume MySQL dan lampiran merupakan penyimpanan persisten.

Jangan menjalankan docker compose down -v,
docker volume prune, atau docker system prune
tanpa pemeriksaan risiko kehilangan data.

### Pemeriksaan HTTP

Nginx Production:

    curl -I http://127.0.0.1:8081/

Jika Nginx menghasilkan HTTP 200 tetapi domain gagal,
periksa Caddy dan Windows Portproxy terlebih dahulu.

### Catatan Deployment

Metadata Production menunjukkan Nginx dan Backend/MySQL
berasal dari direktori release yang berbeda.

Verifikasi konfigurasi aktif sebelum menggunakan
docker compose up, restart, atau rollback.


## 14. Runbook — MySQL Production

### Pemeriksaan Status

    sudo docker ps --filter name=accurate-ticketing-production-mysql-1

Hasil normal:
Container berjalan dengan status healthy.

### Pemeriksaan Health Check

    sudo docker inspect accurate-ticketing-production-mysql-1 --format '{{json .State.Health}}'

Periksa status dan riwayat health check.

### Pemeriksaan Log

    sudo docker logs --tail 50 accurate-ticketing-production-mysql-1

Jangan membagikan informasi sensitif dalam log.

### Pemeriksaan Penyimpanan

    df -h

    sudo docker system df

Periksa kapasitas disk sebelum melakukan tindakan.

### Jika MySQL Bermasalah

1. Periksa status container.
2. Periksa health check dan log.
3. Periksa kapasitas penyimpanan.
4. Periksa koneksi Backend ke MySQL.
5. Identifikasi penyebab sebelum melakukan restart.

### Larangan

Jangan menjalankan:

- docker compose down -v
- docker volume prune
- docker system prune --volumes

Jangan menghapus volume MySQL atau melakukan restore
database tanpa prosedur pemulihan yang telah disetujui.

### Catatan

Database Production dan Staging menggunakan volume
dan Docker Network yang terpisah.

Prosedur backup dan restore akan dijelaskan dalam
dokumen khusus Backup & Disaster Recovery.


## 15. Runbook — Laravel Backend Production

### Pemeriksaan Container

    sudo docker ps --filter name=accurate-ticketing-production-backend-1

Pastikan Backend Production berjalan.

### Pemeriksaan Log Laravel

    sudo docker exec accurate-ticketing-production-backend-1 \
      sh -c 'tail -n 50 /var/www/html/storage/logs/laravel.log'

Jika file log tidak tersedia, periksa konfigurasi logging.

### Pemeriksaan Log Container

    sudo docker logs --tail 50 accurate-ticketing-production-backend-1

Periksa error tanpa membagikan token, password,
session cookie, atau informasi pengguna.

### Pemeriksaan Koneksi Aplikasi

    curl -I http://127.0.0.1:8081/

HTTP 200 menunjukkan endpoint web merespons,
tetapi tidak membuktikan seluruh fitur aplikasi berjalan.

### Jika Login atau Ticketing Bermasalah

1. Periksa status Backend dan MySQL.
2. Periksa log Laravel dan container.
3. Identifikasi pesan error.
4. Periksa koneksi Backend ke database.
5. Lakukan pengujian fitur sesuai peran pengguna.

### Catatan Keamanan

- Jangan menampilkan isi file .env.
- Jangan menjalankan migration pada Production.
- Jangan menghapus cache atau session tanpa analisis dampak.
- Jangan restart Backend sebelum penyebab masalah diperiksa.

