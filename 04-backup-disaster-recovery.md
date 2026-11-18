# Accurate Ticketing — Backup & Disaster Recovery

## 1. Tujuan

Dokumen ini mencatat mekanisme backup dan
persyaratan pemulihan Accurate Ticketing Production.

Status: Draft.
Tanggal inventaris: 18 November 2026.

## 2. Lokasi Backup

Direktori utama:

/home/isa/accurate-ticketing-backups

## 3. Backup Otomatis

Penjadwalan menggunakan systemd:

- accurate-ticketing-backup.timer
- accurate-ticketing-backup.service
- accurate-ticketing-backup-failure.service

Jadwal teramati: setiap hari pukul 01.00 WIB.

## 4. Hasil Backup Terakhir

Tanggal: 18 November 2026.

MySQL:
mysql-auto-20261118-010000.sql.gz.enc

Caddy:
caddy-auto-20261118-010000.tar.gz.enc

Hasil service:
Result=success
ExecMainStatus=0

Log menunjukkan pembuatan kedua backup berhasil.

## 5. Batas Verifikasi

Keberhasilan pembuatan arsip tidak otomatis
membuktikan keberhasilan pemulihan data.

Pengujian restore terdahulu harus dicatat
terpisah sesuai lingkungan dan tanggal pengujian.

Backup lampiran tiket belum teridentifikasi
secara khusus dalam inventaris ini.

Lokasi salinan backup di luar direktori utama
masih perlu dipastikan.

## 6. Ketentuan Pemulihan

- Jangan melakukan restore langsung ke Production.
- Jangan menghapus volume database atau lampiran.
- Pastikan sumber backup dan kunci dekripsi tersedia.
- Gunakan lingkungan terisolasi untuk pengujian.
- Verifikasi konsistensi database dan lampiran.
- Dapatkan persetujuan sebelum pemulihan Production.

## 7. Riwayat Pengujian Restore

### Pengujian Sebelumnya

Pengujian restore database pernah dilakukan
menggunakan volume MySQL sementara yang terisolasi.

Hasil pengujian yang tercatat:

- Integritas arsip backup: PASS.
- Restore file MySQL: PASS.
- MySQL audit berhasil dijalankan.
- Database accurate_ticketing ditemukan.
- Tabel dan jumlah record berhasil diperiksa.

Jumlah record pada lingkungan audit:

| Tabel | Record |
|---|---:|
| users | 8 |
| roles | 3 |
| departments | 7 |
| tickets | 19 |
| ticket_attachments | 2 |
| migrations | 21 |

### Batas Pengujian

Pengujian tersebut membuktikan bahwa backup
yang digunakan saat audit dapat dipulihkan
dan dibaca dalam lingkungan terisolasi.

Pengujian tersebut tidak otomatis membuktikan
bahwa backup terbaru tanggal 18 November 2026
dapat dipulihkan.

Keberhasilan pemulihan lampiran fisik dan
keseluruhan aplikasi juga belum dinyatakan
berhasil berdasarkan pengujian database ini.

## 8. Prosedur Disaster Recovery

Status: Draft — belum diuji end-to-end.

### Tahap 1 — Identifikasi Gangguan

1. Catat waktu dan gejala gangguan.
2. Periksa Docker, HTTPS, Laravel, dan MySQL.
3. Tentukan apakah masalah berasal dari aplikasi,
   infrastruktur, atau kehilangan data.
4. Hindari restart atau restore tanpa diagnosis.

### Tahap 2 — Lindungi Data

1. Hentikan tindakan yang berisiko mengubah data.
2. Identifikasi volume database dan lampiran.
3. Pastikan backup dan kunci dekripsi tersedia.
4. Pertahankan bukti log dan kondisi awal.

### Tahap 3 — Uji Pemulihan

1. Pilih backup sesuai waktu pemulihan yang disetujui.
2. Verifikasi integritas arsip.
3. Siapkan lingkungan pemulihan terisolasi.
4. Uji dekripsi dan restore di lingkungan tersebut.
5. Validasi database, pengguna, tiket, dan lampiran.
6. Catat hasil beserta keterbatasan pengujian.

### Tahap 4 — Pemulihan Production

Hanya dilakukan setelah persetujuan dan
prosedur teknis pemulihan telah diverifikasi.

Jangan mengganti volume Production secara langsung
berdasarkan keberhasilan restore database saja.

### Tahap 5 — Validasi Layanan

Periksa HTTPS, login, workflow tiket, lampiran,
ekspor laporan, serta konsistensi data.

Catat waktu pemulihan dan hasil akhir insiden.

## 9. Salinan Backup Windows

Tanggal pemeriksaan: 18 November 2026.

Lokasi tujuan:

D:\Backup-Accurate-Ticketing-Production

Lokasi backup yang diperiksa:

D:\Backup-Accurate-Ticketing-Production\2026-11-18

File yang ditemukan:

- mysql-auto-20261118-010000.sql.gz.enc
- caddy-auto-20261118-010000.tar.gz.enc

Ukuran file:

- MySQL: 7280 bytes.
- Caddy: 5024 bytes.

Status:
BACKUP_WINDOWS_DIR=FOUND

### Mekanisme Penyalinan

Script yang digunakan:

/usr/local/libexec/accurate-ticketing/copy-backups-to-windows.sh

Script selesai dengan exit status 0 pada
18 November 2026 pukul 01.00 WIB.

### Batas Verifikasi

Keberadaan kedua file telah dikonfirmasi.

Kesamaan checksum salinan Windows dengan
file sumber belum diverifikasi.

Backup lampiran fisik tiket belum teridentifikasi
dalam mekanisme backup otomatis yang diperiksa.
