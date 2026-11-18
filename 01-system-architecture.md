# Accurate Ticketing — System Architecture

## Informasi Dokumen

- Sistem: Accurate Ticketing
- Server: ISASERVER
- Tanggal inventaris: 18 November 2026
- Lingkungan: Production dan Staging
- Status: Draft — berdasarkan pemeriksaan aktual

## 1. System Overview

Accurate Ticketing merupakan aplikasi ticketing internal
perusahaan yang menggunakan:

- Frontend: React
- Backend: Laravel
- Database: MySQL
- Web Server: Nginx
- HTTPS Reverse Proxy: Caddy
- Container Platform: Docker
- Host OS: Windows 10
- Linux Environment: Ubuntu WSL 2

## 2. Server Environment

| Komponen | Konfigurasi |
|---|---|
| Hostname | ISASERVER |
| Windows LAN IP | 192.168.1.10 |
| WSL Distribution | Ubuntu 24.04.5 LTS |
| WSL Networking | NAT |
| WSL IP | Dinamis |
| Docker Engine | 29.9.3 |
| Docker Compose | 5.7.0 |

## 3. Production Access

Domain:
https://ticketing.rigtenders.co.id

Jalur akses yang berhasil diverifikasi:

Browser
  |
Windows ISASERVER :443
  |
Windows Portproxy
  |
WSL IP :8445
  |
Caddy Production
  |
Nginx Production
  |
Laravel Backend
  |
MySQL Production

Port:

| Service | Host Binding |
|---|---|
| Production Caddy | 0.0.0.0:8445 |
| Production Nginx | 127.0.0.1:8081 |
| Staging Nginx | 127.0.0.1:8082 |
| Caddy Test | 127.0.0.1:8443 |

## 4. Docker Networks

Production:
- accurate-ticketing-production_web
- accurate-ticketing-production_database

Staging:
- accurate-ticketing-staging_web
- accurate-ticketing-staging_database

Nginx terhubung ke network web.

Backend terhubung ke network web dan database.

MySQL terhubung ke network database.

Network Production dan Staging terpisah.

## 5. Persistent Storage

Backend Production dan Staging:
- /var/www/html/storage/app/private

MySQL Production dan Staging:
- /var/lib/mysql

Keduanya menggunakan Docker volumes.

Nama volume dan lokasi backup akan dijelaskan
dalam dokumen Storage & Backup.

## 6. Docker Compose Locations

Production Nginx:
 /home/isa/accurate-ticketing-release/13ddfb2/deployment-package/compose.production.yaml

Production Backend dan MySQL:
 /home/isa/accurate-ticketing-release/3f4b4ae/deployment-package/compose.production.yaml

Staging:
 /home/isa/accurate-ticketing-staging-env/compose.staging.yaml

Production Caddy:
 /home/isa/accurate-ticketing-caddy-production/compose.yaml

Caddy Test:
 /home/isa/accurate-ticketing-caddy-test/compose.yaml

Catatan:
Metadata container Production menunjukkan dua release
berbeda. Verifikasi konfigurasi deployment aktif sebelum
melakukan update atau rollback.

## 7. Windows WSL Network Recovery

Windows Scheduled Tasks:

- AccurateTicketing-WSL-Autostart
- AccurateTicketing-WSL-IP-Update
- AccurateTicketing-Portproxy-Recovery

IP WSL dicatat pada:
C:\Users\ISA\AppData\Local\AccurateTicketing\wsl-ip.txt

Script recovery:
C:\ProgramData\AccurateTicketing\repair-portproxy.ps1

Fungsi:
- Memperbarui IP WSL secara otomatis
- Menyesuaikan Windows Portproxy
- Memverifikasi koneksi HTTPS
- Mencatat hasil recovery

Recovery sudah lolos pengujian kondisi normal.
Pengujian setelah restart belum diselesaikan.

## 8. Operational Notes

- Jangan mengubah Production tanpa prosedur yang jelas.
- Jangan menghapus Docker volumes tanpa verifikasi.
- Jangan menyimpan password atau isi .env di dokumentasi.
- Verifikasi Production dan Staging secara terpisah.
- Selalu periksa file Compose aktif sebelum deployment.

## 9. Pending Documentation

- Detail Docker volume dan backup
- Konfigurasi deployment dan rollback
- Struktur aplikasi dan database
- Monitoring dan health checks
- Troubleshooting runbooks
- Pengujian recovery setelah restart
- Incident and change log
