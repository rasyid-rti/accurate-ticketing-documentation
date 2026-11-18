# Accurate Ticketing — Deployment & Rollback

## 1. Tujuan

Dokumen ini mencatat konfigurasi deployment
Production dan Staging di ISASERVER.

Status: Draft.

## 2. Lingkungan

Production:
https://ticketing.rigtenders.co.id

Staging:
Lingkungan Docker terpisah dari Production.

## 3. Docker Compose

Production Nginx:
`/home/isa/accurate-ticketing-release/13ddfb2/deployment-package/compose.production.yaml`

Production Backend dan MySQL:
`/home/isa/accurate-ticketing-release/3f4b4ae/deployment-package/compose.production.yaml`

Staging:
`/home/isa/accurate-ticketing-staging-env/compose.staging.yaml`

Caddy Production:
`/home/isa/accurate-ticketing-caddy-production/compose.yaml`

## 4. Temuan Deployment

Metadata Nginx dan Backend Production menunjukkan
direktori release yang berbeda.

Kondisi ini harus diperiksa sebelum deployment
atau rollback berikutnya.

Metadata Compose bukan bukti tunggal versi kode
atau image yang sedang dijalankan.

## 5. Prinsip Keamanan

- Jangan melakukan deployment langsung tanpa validasi.
- Jangan mengubah Production saat inventarisasi.
- Jangan menggunakan docker compose down -v.
- Jangan menghapus volume database atau lampiran.
- Jangan melakukan rollback tanpa memastikan
  image, konfigurasi, dan kompatibilitas database.

## 6. Validasi dan Prosedur

Prosedur deployment, verifikasi, dan rollback
akan dilengkapi berdasarkan konfigurasi aktual.


## 7. Inventaris Image Docker

Hasil pemeriksaan ISASERVER tanggal 18 November 2026.

| Komponen | Image | Image ID (prefix) |
|---|---|---|
| Production Nginx | nginx:stable-alpine | c59d1f8b1e0e |
| Production Backend | accurate-ticketing-backend:release-3f4b4ae | 04cbb7a677f8 |
| Production MySQL | mysql:8.4 | 76f0358884cb |
| Staging Nginx | nginx:stable-alpine | c59d1f8b1e0e |
| Staging Backend | accurate-ticketing-backend:release-3f4b4ae | 04cbb7a677f8 |
| Staging MySQL | mysql:8.4 | 76f0358884cb |
| Production Caddy | caddy:2 | 9c1b76fb701c |

### Temuan

Production dan Staging menggunakan image Backend yang sama,
tetapi memiliki lingkungan container dan database terpisah.

Metadata Compose Nginx Production menunjuk ke release
13ddfb2, sementara Backend menunjuk ke release 3f4b4ae.

Image ID adalah identitas image yang digunakan container
saat pemeriksaan, bukan jaminan image tersebut masih
tersedia saat rollback dilakukan.

### Persyaratan Rollback

Sebelum rollback, verifikasi:

- Image target masih tersedia.
- File Compose sesuai dengan release yang dipilih.
- Konfigurasi dan volume persisten teridentifikasi.
- Perubahan database kompatibel dengan versi aplikasi.
- Prosedur pemulihan tersedia dan telah disetujui.

## 8. Pre-Deployment Checklist

Checklist ini wajib diperiksa sebelum deployment Production.

### A. Persiapan

- [ ] Perubahan aplikasi sudah disetujui.
- [ ] Release target dan versi kode telah ditentukan.
- [ ] Pengujian Staging telah berhasil.
- [ ] Jadwal deployment telah disepakati.
- [ ] Penanggung jawab deployment tersedia.

### B. Infrastruktur

- [ ] Docker Production dalam kondisi sehat.
- [ ] MySQL Production berstatus healthy.
- [ ] Kapasitas disk mencukupi.
- [ ] Image Docker target tersedia.
- [ ] File Compose target telah diverifikasi.
- [ ] Konfigurasi Caddy dan Portproxy diketahui.

### C. Perlindungan Data

- [ ] Backup database terbaru tersedia.
- [ ] Backup lampiran tersedia.
- [ ] Integritas backup sudah diverifikasi.
- [ ] Lokasi volume persisten sudah diketahui.
- [ ] Perubahan skema database telah ditinjau.

### D. Kesiapan Rollback

- [ ] Release sebelumnya teridentifikasi.
- [ ] Image release sebelumnya tersedia.
- [ ] Konfigurasi sebelumnya tersedia.
- [ ] Kompatibilitas database diperiksa.
- [ ] Kriteria rollback telah disepakati.

### E. Validasi Setelah Deployment

- [ ] HTTPS Production merespons normal.
- [ ] Login setiap peran berhasil.
- [ ] Pembuatan tiket berhasil.
- [ ] Workflow tiket berhasil.
- [ ] Upload dan download lampiran berhasil.
- [ ] Ekspor laporan berhasil.
- [ ] Log aplikasi tidak menunjukkan error kritis.

Checklist ini merupakan prosedur operasional,
bukan bukti bahwa seluruh pemeriksaan telah dilakukan.

## 9. Verifikasi Konfigurasi Compose Production

Tanggal pemeriksaan: 18 November 2026.

Dua file Compose Production telah dibandingkan:

- Release 13ddfb2
- Release 3f4b4ae

Hasil:
COMPOSE_IDENTICAL=YES

SHA-256 kedua file:
cd08c2b4f196a081fb4ee0ff285b8c30a0b099dd43e89620bb73642a4134af6d

Kedua file memiliki isi identik.

### Implikasi Deployment

- Perbedaan metadata working directory tetap tercatat.
- Kesamaan Compose tidak membuktikan kesamaan file .env.
- Kesamaan Compose tidak membuktikan seluruh runtime identik.
- Image Backend aktif adalah release-3f4b4ae.
- Deployment dan rollback belum boleh dijalankan
  hanya berdasarkan hasil perbandingan ini.

## 10. Konfigurasi Environment Production

Tanggal pemeriksaan: 18 November 2026.

File environment ditemukan pada:

- /home/isa/accurate-ticketing-release/13ddfb2/.env.production
- /home/isa/accurate-ticketing-release/3f4b4ae/.env.production

Hasil perbandingan:
PRODUCTION_ENV_IDENTICAL=YES

Kedua file berukuran 463 bytes.

File Compose Production menggunakan variabel PROD_
untuk image Backend, Laravel, database, dan email.

### Status Verifikasi

Mekanisme pemuatan file .env.production pada deployment
sebelumnya belum berhasil diverifikasi.

Pencarian referensi env-file pada script dan dokumentasi
release tidak menemukan hasil.

### Ketentuan Deployment

Sebelum deployment berikutnya:

1. Tentukan release target.
2. Pastikan file environment yang benar tersedia.
3. Pastikan mekanisme pemuatan variabel telah diverifikasi.
4. Validasi konfigurasi Compose tanpa menampilkan rahasia.
5. Pastikan image target dan rollback tersedia.
6. Jangan menjalankan Compose jika variabel wajib belum siap.

Jangan menyimpan password atau APP_KEY dalam dokumentasi.

## 11. Prosedur Deployment Production

Status: Draft — belum diuji end-to-end.

### Persiapan

1. Dapatkan persetujuan deployment.
2. Tentukan release dan image target.
3. Selesaikan checklist pada Bagian 8.
4. Pastikan backup database dan lampiran tersedia.
5. Verifikasi cara pemuatan environment Production.
6. Periksa kompatibilitas perubahan database.

### Pelaksanaan

Deployment hanya dilakukan dengan prosedur yang
telah diuji pada Staging dan disetujui.

Karena metadata Nginx dan Backend Production
menunjuk ke direktori release berbeda, jangan
menjalankan docker compose up secara menyeluruh
tanpa memeriksa dampaknya terhadap setiap service.

Perintah deployment aktual belum ditetapkan.

### Validasi Setelah Deployment

Periksa:

- Status container Production.
- HTTPS dan sertifikat TLS.
- Login sesuai peran pengguna.
- Pembuatan dan perubahan status tiket.
- Upload dan download lampiran.
- Ekspor laporan.
- Log aplikasi dan database.

## 12. Prosedur Rollback Production

Status: Draft — belum diuji.

### Kriteria Rollback

Rollback dipertimbangkan apabila deployment
menyebabkan gangguan kritis dan perbaikan aman
tidak dapat dilakukan dalam batas waktu yang
disepakati.

### Persiapan Rollback

1. Hentikan perubahan aplikasi berikutnya.
2. Identifikasi release terakhir yang berfungsi.
3. Verifikasi image dan konfigurasi sebelumnya.
4. Periksa kompatibilitas skema database.
5. Pastikan data dan lampiran terlindungi.
6. Dapatkan persetujuan rollback.

### Pelaksanaan Rollback

Perintah rollback aktual belum ditetapkan.

Jangan melakukan rollback database otomatis.
Rollback aplikasi dan pemulihan database adalah
dua tindakan berbeda yang membutuhkan penilaian
risiko masing-masing.

### Verifikasi Setelah Rollback

- Container terkait berjalan normal.
- HTTPS Production dapat diakses.
- Login dan workflow tiket berfungsi.
- Data tiket dan lampiran tetap tersedia.
- Log tidak menunjukkan kesalahan kritis.

### Larangan

Jangan menjalankan docker compose down -v,
docker volume prune, atau menghapus volume
untuk melakukan rollback aplikasi.
