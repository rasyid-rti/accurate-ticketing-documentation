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
TODO: Perintah dan interpretasi hasil.

### 3.2 Status Nginx
TODO: Pemeriksaan HTTP lokal.

### 3.3 Status Caddy
TODO: Pemeriksaan HTTPS dengan nama domain.

### 3.4 Status Windows Portproxy
TODO: Pemeriksaan aturan forwarding.

### 3.5 Status MySQL
TODO: Pemeriksaan kesehatan tanpa mengubah data.

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

TODO:
- Pemeriksaan hasil task
- Pemeriksaan log
- Prosedur pemulihan manual
- Pengujian setelah restart

## 6. Troubleshooting Docker

TODO:
- Container stopped
- Nginx unavailable
- Backend unavailable
- Network connectivity

## 7. Troubleshooting Database

TODO:
- MySQL unhealthy
- Connection failed
- Disk capacity
- Backup dan restore

## 8. Troubleshooting Application

TODO:
- Login gagal
- Error 500
- Tiket tidak tersimpan
- Lampiran gagal
- Laporan tidak dapat diekspor

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

