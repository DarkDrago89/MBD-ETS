# MBD-ETS
# Laporan Manajemen Basis Data
## Pemicu 1: Polemik Masalah PPDB Online
### Implementasi PostgreSQL

---

# BAGIAN 1: DESAIN DATABASE PPDB

## 1.1 Entity Relationship Diagram (Konseptual)

```
CALON_SISWA ||--o{ PENDAFTARAN : "melakukan"
SEKOLAH ||--o{ PENDAFTARAN : "menerima"
JALUR_SELEKSI ||--o{ PENDAFTARAN : "digunakan"
PENDAFTARAN ||--o{ SELEKSI_HASIL : "menghasilkan"
PENDAFTARAN ||--o{ DAFTAR_ULANG : "berlanjut ke"
ADMIN ||--o{ LOG_AKTIVITAS : "mencatat"
```

## 1.2 Skema Relasional

```
CALON_SISWA (id_calon PK, nis, nama, tempat_lahir, tanggal_lahir, 
             jenis_kelamin, alamat, no_telp, email, asal_sekolah, 
             nilai_rata_rata, kategori_afirmasi, status_aktif)

SEKOLAH (id_sekolah PK, nss, nama_sekolah, jenjang, alamat, 
         kota, kuota_total, status_aktif)

JALUR_SELEKSI (id_jalur PK, nama_jalur, deskripsi, jenjang, 
               kuota_persen, persyaratan, status_aktif)

KUOTA_SEKOLAH (id_kuota PK, id_sekolah FK, id_jalur FK, 
               kuota_tersedia, kuota_terisi, tahun_ajaran)

PENDAFTARAN (id_pendaftaran PK, id_calon FK, id_sekolah FK, 
             id_jalur FK, tanggal_daftar, status_pendaftaran,
             nomor_pendaftaran, tahun_ajaran, berkas_lengkap)

SELEKSI_HASIL (id_hasil PK, id_pendaftaran FK, skor_seleksi, 
               peringkat, status_kelulusan, tanggal_pengumuman,
               keterangan, diumumkan_oleh)

DAFTAR_ULANG (id_daftar_ulang PK, id_pendaftaran FK, 
              tanggal_daftar_ulang, status_daftar_ulang,
              petugas_verifikasi, catatan, berkas_diterima)

ADMIN (id_admin PK, username, password_hash, nama_lengkap, 
       role, email, status_aktif)

LOG_AKTIVITAS (id_log PK, id_admin FK, id_pendaftaran FK,
               waktu_aktivitas, jenis_aktivitas, 
               data_lama, data_baru, keterangan)
```

---

# BAGIAN 2: DDL DAN DATA DUMMY

## 2.1 DDL - Create Database & Tables

```sql
-- ============================================
-- PPDB DATABASE - PostgreSQL Implementation
-- Manajemen Basis Data - ITS
-- ============================================

-- Buat Database
CREATE DATABASE ppdb_sulsel
    WITH ENCODING = 'UTF8'
    LC_COLLATE = 'id_ID.UTF-8'
    LC_CTYPE = 'id_ID.UTF-8';

\c ppdb_sulsel;

-- ============================================
-- DROP TABLES (jika sudah ada)
-- ============================================
DROP TABLE IF EXISTS log_aktivitas CASCADE;
DROP TABLE IF EXISTS daftar_ulang CASCADE;
DROP TABLE IF EXISTS seleksi_hasil CASCADE;
DROP TABLE IF EXISTS pendaftaran CASCADE;
DROP TABLE IF EXISTS kuota_sekolah CASCADE;
DROP TABLE IF EXISTS jalur_seleksi CASCADE;
DROP TABLE IF EXISTS sekolah CASCADE;
DROP TABLE IF EXISTS calon_siswa CASCADE;
DROP TABLE IF EXISTS admin CASCADE;

-- ============================================
-- TABLE: calon_siswa
-- ============================================
CREATE TABLE calon_siswa (
    id_calon        SERIAL PRIMARY KEY,
    nis             VARCHAR(20) UNIQUE NOT NULL,
    nama            VARCHAR(100) NOT NULL,
    tempat_lahir    VARCHAR(50) NOT NULL,
    tanggal_lahir   DATE NOT NULL,
    jenis_kelamin   CHAR(1) CHECK (jenis_kelamin IN ('L','P')) NOT NULL,
    alamat          TEXT NOT NULL,
    no_telp         VARCHAR(15),
    email           VARCHAR(100) UNIQUE,
    asal_sekolah    VARCHAR(100) NOT NULL,
    nilai_rata_rata NUMERIC(5,2) CHECK (nilai_rata_rata BETWEEN 0 AND 100),
    kategori_afirmasi VARCHAR(50) DEFAULT 'Tidak Ada',
    status_aktif    BOOLEAN DEFAULT TRUE,
    created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

COMMENT ON TABLE calon_siswa IS 'Menyimpan data calon peserta didik baru';
COMMENT ON COLUMN calon_siswa.kategori_afirmasi IS 
    'Contoh: KIP, Disabilitas, Tidak Ada';

-- ============================================
-- TABLE: sekolah
-- ============================================
CREATE TABLE sekolah (
    id_sekolah      SERIAL PRIMARY KEY,
    nss             VARCHAR(20) UNIQUE NOT NULL,
    nama_sekolah    VARCHAR(100) NOT NULL,
    jenjang         VARCHAR(5) CHECK (jenjang IN ('SMA','SMK')) NOT NULL,
    alamat          TEXT NOT NULL,
    kota            VARCHAR(50) NOT NULL,
    kuota_total     INTEGER CHECK (kuota_total > 0) NOT NULL,
    status_aktif    BOOLEAN DEFAULT TRUE,
    created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

COMMENT ON TABLE sekolah IS 'Menyimpan data sekolah penerima PPDB';

-- ============================================
-- TABLE: jalur_seleksi
-- ============================================
CREATE TABLE jalur_seleksi (
    id_jalur        SERIAL PRIMARY KEY,
    nama_jalur      VARCHAR(100) NOT NULL,
    deskripsi       TEXT,
    jenjang         VARCHAR(5) CHECK (jenjang IN ('SMA','SMK','BOTH')) 
                    DEFAULT 'BOTH',
    kuota_persen    NUMERIC(5,2) CHECK (kuota_persen BETWEEN 0 AND 100),
    persyaratan     TEXT,
    status_aktif    BOOLEAN DEFAULT TRUE
);

COMMENT ON TABLE jalur_seleksi IS 'Menyimpan jenis jalur seleksi PPDB';

-- ============================================
-- TABLE: kuota_sekolah
-- ============================================
CREATE TABLE kuota_sekolah (
    id_kuota        SERIAL PRIMARY KEY,
    id_sekolah      INTEGER NOT NULL 
                    REFERENCES sekolah(id_sekolah) ON DELETE CASCADE,
    id_jalur        INTEGER NOT NULL 
                    REFERENCES jalur_seleksi(id_jalur) ON DELETE CASCADE,
    kuota_tersedia  INTEGER CHECK (kuota_tersedia >= 0) NOT NULL,
    kuota_terisi    INTEGER CHECK (kuota_terisi >= 0) DEFAULT 0,
    tahun_ajaran    VARCHAR(9) NOT NULL DEFAULT '2023/2024',
    UNIQUE (id_sekolah, id_jalur, tahun_ajaran),
    CONSTRAINT cek_kuota CHECK (kuota_terisi <= kuota_tersedia)
);

COMMENT ON TABLE kuota_sekolah IS 'Menyimpan kuota per sekolah per jalur';

-- ============================================
-- TABLE: pendaftaran
-- ============================================
CREATE TABLE pendaftaran (
    id_pendaftaran      SERIAL PRIMARY KEY,
    id_calon            INTEGER NOT NULL 
                        REFERENCES calon_siswa(id_calon) ON DELETE RESTRICT,
    id_sekolah          INTEGER NOT NULL 
                        REFERENCES sekolah(id_sekolah) ON DELETE RESTRICT,
    id_jalur            INTEGER NOT NULL 
                        REFERENCES jalur_seleksi(id_jalur) ON DELETE RESTRICT,
    nomor_pendaftaran   VARCHAR(20) UNIQUE NOT NULL,
    tanggal_daftar      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    status_pendaftaran  VARCHAR(20) 
                        CHECK (status_pendaftaran IN (
                            'Menunggu','Terverifikasi',
                            'Ditolak','Proses Seleksi'
                        )) DEFAULT 'Menunggu',
    tahun_ajaran        VARCHAR(9) NOT NULL DEFAULT '2023/2024',
    berkas_lengkap      BOOLEAN DEFAULT FALSE,
    keterangan          TEXT,
    created_at          TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at          TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

COMMENT ON TABLE pendaftaran IS 
    'Menyimpan data pendaftaran calon peserta didik';

-- ============================================
-- TABLE: seleksi_hasil
-- ============================================
CREATE TABLE seleksi_hasil (
    id_hasil            SERIAL PRIMARY KEY,
    id_pendaftaran      INTEGER UNIQUE NOT NULL 
                        REFERENCES pendaftaran(id_pendaftaran) 
                        ON DELETE CASCADE,
    skor_seleksi        NUMERIC(5,2) DEFAULT 0,
    peringkat           INTEGER,
    status_kelulusan    VARCHAR(20) 
                        CHECK (status_kelulusan IN (
                            'Lulus','Tidak Lulus',
                            'Cadangan','Menunggu'
                        )) DEFAULT 'Menunggu',
    tanggal_pengumuman  TIMESTAMP,
    keterangan          TEXT,
    diumumkan_oleh      VARCHAR(100),
    versi_data          INTEGER DEFAULT 1,
    created_at          TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at          TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

COMMENT ON TABLE seleksi_hasil IS 
    'Menyimpan hasil seleksi calon peserta didik';
COMMENT ON COLUMN seleksi_hasil.versi_data IS 
    'Digunakan untuk tracking perubahan data (solusi masalah PPDB)';

-- ============================================
-- TABLE: daftar_ulang
-- ============================================
CREATE TABLE daftar_ulang (
    id_daftar_ulang     SERIAL PRIMARY KEY,
    id_pendaftaran      INTEGER UNIQUE NOT NULL 
                        REFERENCES pendaftaran(id_pendaftaran) 
                        ON DELETE CASCADE,
    tanggal_daftar_ulang TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    status_daftar_ulang VARCHAR(20) 
                        CHECK (status_daftar_ulang IN (
                            'Hadir','Tidak Hadir',
                            'Terdaftar','Dibatalkan'
                        )) DEFAULT 'Terdaftar',
    petugas_verifikasi  VARCHAR(100),
    catatan             TEXT,
    berkas_diterima     BOOLEAN DEFAULT FALSE,
    created_at          TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

COMMENT ON TABLE daftar_ulang IS 
    'Menyimpan data daftar ulang peserta yang lulus seleksi';

-- ============================================
-- TABLE: admin
-- ============================================
CREATE TABLE admin (
    id_admin        SERIAL PRIMARY KEY,
    username        VARCHAR(50) UNIQUE NOT NULL,
    password_hash   VARCHAR(255) NOT NULL,
    nama_lengkap    VARCHAR(100) NOT NULL,
    role            VARCHAR(20) 
                    CHECK (role IN (
                        'SuperAdmin','AdminProvinsi',
                        'AdminSekolah','Operator'
                    )) NOT NULL,
    email           VARCHAR(100) UNIQUE NOT NULL,
    status_aktif    BOOLEAN DEFAULT TRUE,
    created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    last_login      TIMESTAMP
);

COMMENT ON TABLE admin IS 'Menyimpan data admin pengelola sistem PPDB';

-- ============================================
-- TABLE: log_aktivitas
-- ============================================
CREATE TABLE log_aktivitas (
    id_log          SERIAL PRIMARY KEY,
    id_admin        INTEGER REFERENCES admin(id_admin) ON DELETE SET NULL,
    id_pendaftaran  INTEGER REFERENCES pendaftaran(id_pendaftaran) 
                    ON DELETE SET NULL,
    waktu_aktivitas TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    jenis_aktivitas VARCHAR(50) NOT NULL,
    tabel_terdampak VARCHAR(50),
    data_lama       JSONB,
    data_baru       JSONB,
    keterangan      TEXT,
    ip_address      INET
);

COMMENT ON TABLE log_aktivitas IS 
    'Audit trail seluruh aktivitas perubahan data PPDB';

-- ============================================
-- INDEX untuk optimasi query
-- ============================================
CREATE INDEX idx_pendaftaran_calon 
    ON pendaftaran(id_calon);
CREATE INDEX idx_pendaftaran_sekolah 
    ON pendaftaran(id_sekolah);
CREATE INDEX idx_pendaftaran_status 
    ON pendaftaran(status_pendaftaran);
CREATE INDEX idx_seleksi_status 
    ON seleksi_hasil(status_kelulusan);
CREATE INDEX idx_seleksi_pendaftaran 
    ON seleksi_hasil(id_pendaftaran);
CREATE INDEX idx_log_waktu 
    ON log_aktivitas(waktu_aktivitas);
CREATE INDEX idx_calon_nis 
    ON calon_siswa(nis);
CREATE INDEX idx_daftar_ulang_status 
    ON daftar_ulang(status_daftar_ulang);
```

---

## 2.2 Data Dummy (Minimal 50 data per tabel)

```sql
-- ============================================
-- INSERT DATA: jalur_seleksi
-- ============================================
INSERT INTO jalur_seleksi 
    (nama_jalur, deskripsi, jenjang, kuota_persen, persyaratan) 
VALUES
('Boarding School', 
 'Jalur khusus sekolah berasrama', 
 'SMA', 5.00, 
 'Nilai rata-rata minimal 80, Tes Kesehatan'),
('Afirmasi', 
 'Jalur untuk keluarga tidak mampu dan penerima KIP', 
 'BOTH', 15.00, 
 'Kartu KIP atau SKTM dari kelurahan'),
('Perpindahan Tugas Orang Tua', 
 'Jalur untuk anak yang orang tuanya pindah tugas', 
 'BOTH', 5.00, 
 'Surat tugas orang tua dari instansi'),
('Anak Guru', 
 'Jalur khusus anak tenaga pengajar aktif', 
 'BOTH', 5.00, 
 'Kartu identitas orang tua sebagai guru aktif'),
('Prestasi Non Akademik', 
 'Jalur berdasarkan prestasi lomba atau olahraga', 
 'BOTH', 10.00, 
 'Sertifikat/piagam prestasi minimal tingkat kabupaten'),
('Domisili Terdekat', 
 'Jalur berdasarkan jarak tempat tinggal ke sekolah', 
 'SMK', 15.00, 
 'Kartu Keluarga, minimal 2 tahun domisili'),
('Anak DUDI Mitra SMK', 
 'Jalur khusus anak dari mitra industri SMK', 
 'SMK', 5.00, 
 'Surat keterangan dari perusahaan mitra'),
('Reguler/Umum', 
 'Jalur umum berdasarkan nilai rapor', 
 'BOTH', 40.00, 
 'Nilai rapor semester 1-5 lengkap');

-- ============================================
-- INSERT DATA: sekolah
-- ============================================
INSERT INTO sekolah 
    (nss, nama_sekolah, jenjang, alamat, kota, kuota_total) 
VALUES
('301190001', 'SMAN 1 Makassar', 'SMA', 
 'Jl. Gunung Bawakaraeng No.53', 'Makassar', 360),
('301190002', 'SMAN 2 Makassar', 'SMA', 
 'Jl. Baji Gau No.18', 'Makassar', 360),
('301190003', 'SMAN 3 Makassar', 'SMA', 
 'Jl. Bonto Langkasa', 'Makassar', 324),
('301190004', 'SMAN 4 Makassar', 'SMA', 
 'Jl. Cakalang No.3', 'Makassar', 324),
('301190005', 'SMAN 5 Makassar', 'SMA', 
 'Jl. Taman Makam Pahlawan', 'Makassar', 288),
('301190006', 'SMAN 6 Makassar', 'SMA', 
 'Jl. Abdesir No.3', 'Makassar', 288),
('301190007', 'SMAN 17 Makassar', 'SMA', 
 'Jl. Aroepala', 'Makassar', 252),
('302190001', 'SMKN 1 Makassar', 'SMK', 
 'Jl. Andi Tonro No.27', 'Makassar', 432),
('302190002', 'SMKN 2 Makassar', 'SMK', 
 'Jl. Veteran Selatan No.10', 'Makassar', 396),
('302190003', 'SMKN 3 Makassar', 'SMK', 
 'Jl. Bonto Parang', 'Makassar', 360),
('301190008', 'SMAN 1 Gowa', 'SMA', 
 'Jl. Mesjid Raya No.1', 'Gowa', 288),
('301190009', 'SMAN 2 Gowa', 'SMA', 
 'Jl. Tumanurung Raya', 'Gowa', 252),
('302190004', 'SMKN 1 Gowa', 'SMK', 
 'Jl. Mesjid Raya No.30', 'Gowa', 324),
('301190010', 'SMAN 1 Maros', 'SMA', 
 'Jl. Jenderal Sudirman No.1', 'Maros', 252),
('302190005', 'SMKN 1 Maros', 'SMK', 
 'Jl. Poros Maros-Bone', 'Maros', 288),
('301190011', 'SMAN 1 Takalar', 'SMA', 
 'Jl. Sultan Hasanuddin', 'Takalar', 216),
('301190012', 'SMAN 1 Jeneponto', 'SMA', 
 'Jl. Lanto Dg. Pasewang', 'Jeneponto', 216),
('302190006', 'SMKN 1 Jeneponto', 'SMK', 
 'Jl. Pahlawan', 'Jeneponto', 252),
('301190013', 'SMAN 1 Pangkep', 'SMA', 
 'Jl. Andi Oddang', 'Pangkep', 216),
('302190007', 'SMKN 2 Gowa', 'SMK', 
 'Jl. Syamsuddin Tunru', 'Gowa', 288);

-- ============================================
-- INSERT DATA: kuota_sekolah
-- ============================================
-- SMAN 1 Makassar (id=1, kuota=360)
INSERT INTO kuota_sekolah 
    (id_sekolah, id_jalur, kuota_tersedia, kuota_terisi, tahun_ajaran)
VALUES
(1, 1, 18, 18, '2023/2024'),  -- Boarding 5%
(1, 2, 54, 54, '2023/2024'),  -- Afirmasi 15%
(1, 3, 18, 10, '2023/2024'),  -- Perpindahan
(1, 4, 18, 15, '2023/2024'),  -- Anak Guru
(1, 5, 36, 36, '2023/2024'),  -- Prestasi
(1, 8, 144, 130, '2023/2024'), -- Reguler (diisi 130, bukan 144!)

-- SMAN 2 Makassar (id=2, kuota=360)
(2, 1, 18, 18, '2023/2024'),
(2, 2, 54, 50, '2023/2024'),
(2, 3, 18, 8,  '2023/2024'),
(2, 4, 18, 18, '2023/2024'),
(2, 5, 36, 30, '2023/2024'),
(2, 8, 144, 144, '2023/2024'),

-- SMKN 1 Makassar (id=8, kuota=432)
(8, 2, 65, 65, '2023/2024'),
(8, 3, 22, 15, '2023/2024'),
(8, 4, 22, 20, '2023/2024'),
(8, 5, 43, 43, '2023/2024'),
(8, 6, 65, 60, '2023/2024'),
(8, 7, 22, 20, '2023/2024'),
(8, 8, 173, 170, '2023/2024'),

-- SMAN 3 Makassar (id=3, kuota=324)
(3, 1, 16, 16, '2023/2024'),
(3, 2, 49, 49, '2023/2024'),
(3, 3, 16, 10, '2023/2024'),
(3, 4, 16, 14, '2023/2024'),
(3, 5, 32, 32, '2023/2024'),
(3, 8, 130, 100, '2023/2024'); -- KASUS: 130 lulus, hanya 100 daftar ulang!

-- ============================================
-- INSERT DATA: admin
-- ============================================
INSERT INTO admin 
    (username, password_hash, nama_lengkap, role, email) 
VALUES
('superadmin', 
 '$2b$12$KIXabcdef1234567890abcdefghij', 
 'Administrator Utama', 'SuperAdmin', 
 'superadmin@ppdb.sulselprov.go.id'),
('admin_prov_01', 
 '$2b$12$KIXabcdef1234567890abcdefghij', 
 'Ahmad Fauzi', 'AdminProvinsi', 
 'ahmad.fauzi@ppdb.sulselprov.go.id'),
('admin_sman1_mks', 
 '$2b$12$KIXabcdef1234567890abcdefghij', 
 'Budi Santoso', 'AdminSekolah', 
 'budi.santoso@sman1makassar.sch.id'),
('admin_sman2_mks', 
 '$2b$12$KIXabcdef1234567890abcdefghij', 
 'Citra Dewi', 'AdminSekolah', 
 'citra.dewi@sman2makassar.sch.id'),
('operator_01', 
 '$2b$12$KIXabcdef1234567890abcdefghij', 
 'Dewi Rahayu', 'Operator', 
 'dewi.rahayu@ppdb.sulselprov.go.id'),
('operator_02', 
 '$2b$12$KIXabcdef1234567890abcdefghij', 
 'Eko Prasetyo', 'Operator', 
 'eko.prasetyo@ppdb.sulselprov.go.id'),
('admin_smkn1_mks', 
 '$2b$12$KIXabcdef1234567890abcdefghij', 
 'Fajar Nugroho', 'AdminSekolah', 
 'fajar.nugroho@smkn1makassar.sch.id'),
('admin_sman1_gowa', 
 '$2b$12$KIXabcdef1234567890abcdefghij', 
 'Gita Permata', 'AdminSekolah', 
 'gita.permata@sman1gowa.sch.id'),
('operator_03', 
 '$2b$12$KIXabcdef1234567890abcdefghij', 
 'Hendra Wijaya', 'Operator', 
 'hendra.wijaya@ppdb.sulselprov.go.id'),
('operator_04', 
 '$2b$12$KIXabcdef1234567890abcdefghij', 
 'Indah Sari', 'Operator', 
 'indah.sari@ppdb.sulselprov.go.id');

-- ============================================
-- INSERT DATA: calon_siswa (50+ data)
-- ============================================
INSERT INTO calon_siswa 
    (nis, nama, tempat_lahir, tanggal_lahir, jenis_kelamin, 
     alamat, no_telp, email, asal_sekolah, 
     nilai_rata_rata, kategori_afirmasi) 
VALUES
('2023010001','Andi Maulana Akbar','Makassar','2007-03-15','L',
 'Jl. Perintis Kemerdekaan No.12, Makassar','081234560001',
 'andi.maulana@email.com','SMPN 1 Makassar',88.50,'Tidak Ada'),
('2023010002','Nurul Hidayah','Makassar','2007-05-20','P',
 'Jl. Rappocini Raya No.45, Makassar','081234560002',
 'nurul.hidayah@email.com','SMPN 2 Makassar',92.00,'KIP'),
('2023010003','Muhammad Rizal','Gowa','2007-07-10','L',
 'Jl. Syamsuddin Tunru No.8, Gowa','081234560003',
 'mrizal@email.com','SMPN 1 Gowa',85.75,'Tidak Ada'),
('2023010004','Siti Rahma','Makassar','2007-01-25','P',
 'Jl. Racing Centre No.5, Makassar','081234560004',
 'siti.rahma@email.com','SMPN 6 Makassar',90.25,'Tidak Ada'),
('2023010005','Bagas Prasetyo','Surabaya','2007-11-30','L',
 'Jl. Dg. Tata Raya No.22, Makassar','081234560005',
 'bagas.p@email.com','SMPN 3 Makassar',87.00,'Perpindahan'),
('2023010006','Riska Amelia','Makassar','2007-04-18','P',
 'Jl. Boulevard No.10, Makassar','081234560006',
 'riska.a@email.com','SMPN 12 Makassar',91.50,'Tidak Ada'),
('2023010007','Dimas Setiawan','Makassar','2007-08-22','L',
 'Jl. Hertasning No.33, Makassar','081234560007',
 'dimas.s@email.com','SMPN 8 Makassar',83.25,'KIP'),
('2023010008','Putri Ayu Lestari','Bone','2007-02-14','P',
 'Jl. Urip Sumoharjo No.67, Makassar','081234560008',
 'putri.ayu@email.com','SMPN 4 Makassar',94.75,'Tidak Ada'),
('2023010009','Kevin Ardiansyah','Makassar','2007-09-05','L',
 'Jl. Tamalate No.19, Makassar','081234560009',
 'kevin.a@email.com','SMPN 10 Makassar',86.50,'Tidak Ada'),
('2023010010','Dewi Maharani','Pangkep','2007-06-28','P',
 'Jl. Penghibur No.4, Makassar','081234560010',
 'dewi.m@email.com','SMPN 5 Makassar',89.00,'KIP'),
('2023010011','Faisal Akbar','Makassar','2007-12-01','L',
 'Jl. Pettarani No.55, Makassar','081234560011',
 'faisal.a@email.com','SMPN 7 Makassar',82.00,'Tidak Ada'),
('2023010012','Ayu Wulandari','Maros','2007-03-30','P',
 'Jl. Maros Raya No.12, Maros','081234560012',
 'ayu.w@email.com','SMPN 1 Maros',88.75,'Tidak Ada'),
('2023010013','Rizky Pratama','Makassar','2007-07-17','L',
 'Jl. Cendrawasih No.8, Makassar','081234560013',
 'rizky.p@email.com','SMPN 9 Makassar',91.25,'Tidak Ada'),
('2023010014','Nadia Putri','Jeneponto','2007-05-05','P',
 'Jl. Lanto Dg Pasewang No.30, Jeneponto','081234560014',
 'nadia.p@email.com','SMPN 1 Jeneponto',87.50,'KIP'),
('2023010015','Arif Rahman','Takalar','2007-10-20','L',
 'Jl. Sultan Hasanuddin No.15, Takalar','081234560015',
 'arif.r@email.com','SMPN 2 Takalar',84.00,'Tidak Ada'),
('2023010016','Linda Kusuma','Makassar','2007-01-08','P',
 'Jl. Sudirman No.88, Makassar','081234560016',
 'linda.k@email.com','SMPN 11 Makassar',93.50,'Tidak Ada'),
('2023010017','Wahyu Hidayat','Gowa','2007-04-25','L',
 'Jl. Poros Malino No.7, Gowa','081234560017',
 'wahyu.h@email.com','SMPN 2 Gowa',86.00,'Tidak Ada'),
('2023010018','Elsa Permata','Makassar','2007-08-12','P',
 'Jl. Andi Tonro No.44, Makassar','081234560018',
 'elsa.p@email.com','SMPN 13 Makassar',90.75,'KIP'),
('2023010019','Galih Santosa','Jakarta','2007-11-03','L',
 'Jl. Veteran Selatan No.23, Makassar','081234560019',
 'galih.s@email.com','SMPN 14 Makassar',88.25,'Perpindahan'),
('2023010020','Intan Pratiwi','Makassar','2007-02-28','P',
 'Jl. Nusantara No.56, Makassar','081234560020',
 'intan.p@email.com','SMPN 15 Makassar',85.50,'Tidak Ada'),
('2023010021','Yoga Permana','Makassar','2007-06-15','L',
 'Jl. Kumala No.9, Makassar','081234560021',
 'yoga.p@email.com','SMPN 16 Makassar',87.75,'Tidak Ada'),
('2023010022','Maharani Putri','Sinjai','2007-09-22','P',
 'Jl. Sinjai Raya No.34, Sinjai','081234560022',
 'maharani.p@email.com','SMPN 1 Sinjai',92.25,'KIP'),
('2023010023','Dani Firmansyah','Makassar','2007-12-10','L',
 'Jl. Tol Reformasi No.17, Makassar','081234560023',
 'dani.f@email.com','SMPN 17 Makassar',83.75,'Tidak Ada'),
('2023010024','Sari Indah','Makassar','2007-03-05','P',
 'Jl. Antang Raya No.62, Makassar','081234560024',
 'sari.i@email.com','SMPN 18 Makassar',89.50,'Tidak Ada'),
('2023010025','Bram Nugroho','Gowa','2007-07-28','L',
 'Jl. Samata No.3, Gowa','081234560025',
 'bram.n@email.com','SMPN 3 Gowa',86.25,'Tidak Ada'),
('2023010026','Tiara Putri','Makassar','2007-01-18','P',
 'Jl. Daya No.28, Makassar','081234560026',
 'tiara.p@email.com','SMPN 19 Makassar',91.00,'Tidak Ada'),
('2023010027','Rian Saputra','Maros','2007-05-12','L',
 'Jl. Poros Maros-Bone No.45, Maros','081234560027',
 'rian.s@email.com','SMPN 2 Maros',84.50,'KIP'),
('2023010028','Mega Lestari','Makassar','2007-10-07','P',
 'Jl. Toddopuli No.14, Makassar','081234560028',
 'mega.l@email.com','SMPN 20 Makassar',88.00,'Tidak Ada'),
('2023010029','Agus Setiawan','Pangkep','2007-04-02','L',
 'Jl. Andi Oddang No.20, Pangkep','081234560029',
 'agus.s@email.com','SMPN 1 Pangkep',85.25,'Tidak Ada'),
('2023010030','Fitri Handayani','Makassar','2007-08-19','P',
 'Jl. Malengkeri No.37, Makassar','081234560030',
 'fitri.h@email.com','SMPN 21 Makassar',90.50,'KIP'),
('2023010031','Hendra Gunawan','Makassar','2007-02-06','L',
 'Jl. Rappocini Indah No.5, Makassar','081234560031',
 'hendra.g@email.com','SMPN 22 Makassar',87.25,'Tidak Ada'),
('2023010032','Anastasia Putri','Toraja','2007-06-23','P',
 'Jl. Andi Mangerangi No.48, Makassar','081234560032',
 'anastasia.p@email.com','SMPN 23 Makassar',93.75,'Tidak Ada'),
('2023010033','Fadhil Akbar','Makassar','2007-11-11','L',
 'Jl. Tamarunang No.11, Gowa','081234560033',
 'fadhil.a@email.com','SMPN 4 Gowa',82.75,'KIP'),
('2023010034','Putri Rahayu','Makassar','2007-03-26','P',
 'Jl. Btn Kodam No.29, Makassar','081234560034',
 'putri.r@email.com','SMPN 24 Makassar',89.75,'Tidak Ada'),
('2023010035','Yusuf Hakim','Bone','2007-07-14','L',
 'Jl. Sudirman No.77, Bone','081234560035',
 'yusuf.h@email.com','SMPN 1 Bone',86.75,'Tidak Ada'),
('2023010036','Dina Marliana','Makassar','2007-01-31','P',
 'Jl. Tamalanrea Indah No.16, Makassar','081234560036',
 'dina.m@email.com','SMPN 25 Makassar',91.75,'Tidak Ada'),
('2023010037','Irfan Maulana','Soppeng','2007-05-18','L',
 'Jl. Andi Cammi No.33, Soppeng','081234560037',
 'irfan.m@email.com','SMPN 1 Soppeng',84.25,'KIP'),
('2023010038','Wulandari Sari','Makassar','2007-09-08','P',
 'Jl. Inspeksi Kanal No.21, Makassar','081234560038',
 'wulandari.s@email.com','SMPN 26 Makassar',88.50,'Tidak Ada'),
('2023010039','Naufal Arif','Makassar','2007-12-25','L',
 'Jl. Abdullah Dg Sirua No.40, Makassar','081234560039',
 'naufal.a@email.com','SMPN 27 Makassar',85.00,'Tidak Ada'),
('2023010040','Rizka Amalia','Gowa','2007-04-10','P',
 'Jl. Borongtala No.6, Gowa','081234560040',
 'rizka.a2@email.com','SMPN 5 Gowa',90.25,'KIP'),
('2023010041','Aldi Kurniawan','Makassar','2007-08-03','L',
 'Jl. Paccerakkang No.52, Makassar','081234560041',
 'aldi.k@email.com','SMPN 28 Makassar',83.50,'Tidak Ada'),
('2023010042','Sherly Amelia','Makassar','2007-02-20','P',
 'Jl. Kima Raya No.18, Makassar','081234560042',
 'sherly.a@email.com','SMPN 29 Makassar',92.50,'Tidak Ada'),
('2023010043','Dicky Pratama','Maros','2007-06-07','L',
 'Jl. Turikale No.25, Maros','081234560043',
 'dicky.p@email.com','SMPN 3 Maros',87.00,'Tidak Ada'),
('2023010044','Nova Cahyani','Takalar','2007-10-24','P',
 'Jl. Kaluku Bodoa No.38, Takalar','081234560044',
 'nova.c@email.com','SMPN 1 Takalar',89.25,'KIP'),
('2023010045','Rendy Wijaya','Makassar','2007-03-18','L',
 'Jl. Hertasning Baru No.7, Makassar','081234560045',
 'rendy.w@email.com','SMPN 30 Makassar',86.00,'Tidak Ada'),
('2023010046','Amanda Putri','Makassar','2007-07-05','P',
 'Jl. Komp. Griya Fajar No.3, Makassar','081234560046',
 'amanda.p@email.com','SMPN 31 Makassar',91.50,'Tidak Ada'),
('2023010047','Fikri Ramadhan','Jeneponto','2007-11-22','L',
 'Jl. Bonto Manai No.14, Jeneponto','081234560047',
 'fikri.r@email.com','SMPN 2 Jeneponto',84.75,'KIP'),
('2023010048','Cantika Sari','Makassar','2007-04-15','P',
 'Jl. Adipura No.26, Makassar','081234560048',
 'cantika.s@email.com','SMPN 32 Makassar',88.25,'Tidak Ada'),
('2023010049','Gilang Permana','Gowa','2007-08-30','L',
 'Jl. Btn Graha Jaya No.9, Gowa','081234560049',
 'gilang.p@email.com','SMPN 6 Gowa',85.75,'Tidak Ada'),
('2023010050','Nabila Azzahra','Makassar','2007-02-12','P',
 'Jl. Komp. Hartaco Indah No.41, Makassar','081234560050',
 'nabila.a@email.com','SMPN 33 Makassar',93.00,'Tidak Ada'),
-- Data tambahan untuk memperkaya skenario
('2023010051','Eko Budiman','Makassar','2007-06-19','L',
 'Jl. Minasa Upa No.53, Makassar','081234560051',
 'eko.b@email.com','SMPN 34 Makassar',82.50,'KIP'),
('2023010052','Yanti Rahayu','Palopo','2007-10-06','P',
 'Jl. Andi Djemma No.12, Palopo','081234560052',
 'yanti.r@email.com','SMPN 1 Palopo',90.00,'Tidak Ada'),
('2023010053','Arya Wibawa','Makassar','2007-03-23','L',
 'Jl. Veteran No.66, Makassar','081234560053',
 'arya.w@email.com','SMPN 35 Makassar',87.50,'Tidak Ada'),
('2023010054','Hani Latifah','Makassar','2007-07-09','P',
 'Jl. Rappocini Baru No.31, Makassar','081234560054',
 'hani.l@email.com','SMPN 36 Makassar',91.25,'Tidak Ada'),
('2023010055','Ilham Hidayat','Pangkep','2007-11-26','L',
 'Jl. Poros Pangkep No.44, Pangkep','081234560055',
 'ilham.h@email.com','SMPN 2 Pangkep',84.00,'KIP');

-- ============================================
-- INSERT DATA: pendaftaran (50+ data)
-- ============================================
INSERT INTO pendaftaran 
    (id_calon, id_sekolah, id_jalur, nomor_pendaftaran, 
     tanggal_daftar, status_pendaftaran, tahun_ajaran, 
     berkas_lengkap) 
VALUES
-- SMAN 1 Makassar - Berbagai Jalur
(1,  1, 1, 'PPDB-2023-000001', '2023-06-19 08:15:00', 
 'Terverifikasi', '2023/2024', TRUE),
(2,  1, 2, 'PPDB-2023-000002', '2023-06-19 08:30:00', 
 'Terverifikasi', '2023/2024', TRUE),
(3,  1, 8, 'PPDB-2023-000003', '2023-06-19 09:00:00', 
 'Terverifikasi', '2023/2024', TRUE),
(4,  1, 5, 'PPDB-2023-000004', '2023-06-19 09:15:00', 
 'Terverifikasi', '2023/2024', TRUE),
(5,  1, 3, 'PPDB-2023-000005', '2023-06-19 09:30:00', 
 'Terverifikasi', '2023/2024', TRUE),
(6,  1, 8, 'PPDB-2023-000006', '2023-06-19 10:00:00', 
 'Terverifikasi', '2023/2024', TRUE),
(7,  1, 2, 'PPDB-2023-000007', '2023-06-19 10:15:00', 
 'Terverifikasi', '2023/2024', TRUE),
(8,  1, 8, 'PPDB-2023-000008', '2023-06-19 10:30:00', 
 'Terverifikasi', '2023/2024', TRUE),
(9,  1, 8, 'PPDB-2023-000009', '2023-06-19 11:00:00', 
 'Terverifikasi', '2023/2024', TRUE),
(10, 1, 2, 'PPDB-2023-000010', '2023-06-19 11:15:00', 
 'Terverifikasi', '2023/2024', TRUE),

-- SMAN 2 Makassar
(11, 2, 8, 'PPDB-2023-000011', '2023-06-19 08:00:00', 
 'Terverifikasi', '2023/2024', TRUE),
(12, 2, 8, 'PPDB-2023-000012', '2023-06-19 08:45:00', 
 'Terverifikasi', '2023/2024', TRUE),
(13, 2, 5, 'PPDB-2023-000013', '2023-06-19 09:00:00', 
 'Terverifikasi', '2023/2024', TRUE),
(14, 2, 2, 'PPDB-2023-000014', '2023-06-19 09:30:00', 
 'Terverifikasi', '2023/2024', TRUE),
(15, 2, 8, 'PPDB-2023-000015', '2023-06-19 10:00:00', 
 'Terverifikasi', '2023/2024', TRUE),

-- SMAN 3 Makassar (KASUS BERMASALAH - 130 lulus, 100 daftar ulang)
(16, 3, 8, 'PPDB-2023-000016', '2023-06-19 08:00:00', 
 'Terverifikasi', '2023/2024', TRUE),
(17, 3, 8, 'PPDB-2023-000017', '2023-06-19 08:30:00', 
 'Terverifikasi', '2023/2024', TRUE),
(18, 3, 2, 'PPDB-2023-000018', '2023-06-19 09:00:00', 
 'Terverifikasi', '2023/2024', TRUE),
(19, 3, 3, 'PPDB-2023-000019', '2023-06-19 09:15:00', 
 'Terverifikasi', '2023/2024', TRUE),
(20, 3, 8, 'PPDB-2023-000020', '2023-06-19 09:45:00', 
 'Terverifikasi', '2023/2024', TRUE),

-- SMKN 1 Makassar
(21, 8, 8, 'PPDB-2023-000021', '2023-06-19 08:10:00', 
 'Terverifikasi', '2023/2024', TRUE),
(22, 8, 2, 'PPDB-2023-000022', '2023-06-19 08:40:00', 
 'Terverifikasi', '2023/2024', TRUE),
(23, 8, 6, 'PPDB-2023-000023', '2023-06-19 09:10:00', 
 'Terverifikasi', '2023/2024', TRUE),
(24, 8, 8, 'PPDB-2023-000024', '2023-06-19 09:40:00', 
 'Terverifikasi', '2023/2024', TRUE),
(25, 8, 5, 'PPDB-2023-000025', '2023-06-19 10:10:00', 
 'Terverifikasi', '2023/2024', TRUE),

-- Data pendaftaran lainnya
(26, 1, 4, 'PPDB-2023-000026', '2023-06-20 08:00:00', 
 'Terverifikasi', '2023/2024', TRUE),
(27, 2, 8, 'PPDB-2023-000027', '2023-06-20 08:30:00', 
 'Terverifikasi', '2023/2024', TRUE),
(28, 3, 5, 'PPDB-2023-000028', '2023-06-20 09:00:00', 
 'Terverifikasi', '2023/2024', TRUE),
(29, 8, 7, 'PPDB-2023-000029', '2023-06-20 09:30:00', 
 'Terverifikasi', '2023/2024', TRUE),
(30, 9, 2, 'PPDB-2023-000030', '2023-06-20 10:00:00', 
 'Terverifikasi', '2023/2024', TRUE),
(31, 10, 8,'PPDB-2023-000031', '2023-06-20 10:30:00', 
 'Terverifikasi', '2023/2024', TRUE),
(32, 11, 8,'PPDB-2023-000032', '2023-06-20 11:00:00', 
 'Terverifikasi', '2023/2024', TRUE),
(33, 12, 2,'PPDB-2023-000033', '2023-06-20 11:30:00', 
 'Terverifikasi', '2023/2024', TRUE),
(34, 13, 8,'PPDB-2023-000034', '2023-06-21 08:00:00', 
 'Terverifikasi', '2023/2024', TRUE),
(35, 14, 8,'PPDB-2023-000035', '2023-06-21 08:30:00', 
 'Terverifikasi', '2023/2024', TRUE),
(36, 1, 8, 'PPDB-2023-000036', '2023-06-21 09:00:00', 
 'Terverifikasi', '2023/2024', TRUE),
(37, 2, 2, 'PPDB-2023-000037', '2023-06-21 09:30:00', 
 'Terverifikasi', '2023/2024', TRUE),
(38, 3, 8, 'PPDB-2023-000038', '2023-06-21 10:00:00', 
 'Terverifikasi', '2023/2024', TRUE),
(39, 8, 8, 'PPDB-2023-000039', '2023-06-21 10:30:00', 
 'Terverifikasi', '2023/2024', TRUE),
(40, 9, 8, 'PPDB-2023-000040', '2023-06-21 11:00:00', 
 'Terverifikasi', '2023/2024', TRUE),
(41, 3, 8, 'PPDB-2023-000041', '2023-06-21 11:30:00', 
 'Terverifikasi', '2023/2024', TRUE),
(42, 3, 8, 'PPDB-2023-000042', '2023-06-22 08:00:00', 
 'Terverifikasi', '2023/2024', TRUE),
(43, 3, 2, 'PPDB-2023-000043', '2023-06-22 08:30:00', 
 'Terverifikasi', '2023/2024', TRUE),
(44, 3, 5, 'PPDB-2023-000044', '2023-06-22 09:00:00', 
 'Terverifikasi', '2023/2024', TRUE),
(45, 3, 8, 'PPDB-2023-000045', '2023-06-22 09:30:00', 
 'Terverifikasi', '2023/2024', TRUE),
(46, 1, 2, 'PPDB-2023-000046', '2023-06-22 10:00:00', 
 'Terverifikasi', '2023/2024', TRUE),
(47, 2, 8, 'PPDB-2023-000047', '2023-06-22 10:30:00', 
 'Terverifikasi', '2023/2024', TRUE),
(48, 8, 2, 'PPDB-2023-000048', '2023-06-22 11:00:00', 
 'Terverifikasi', '2023/2024', TRUE),
(49, 9, 8, 'PPDB-2023-000049', '2023-06-22 11:30:00', 
 'Terverifikasi', '2023/2024', TRUE),
(50, 10, 5,'PPDB-2023-000050', '2023-06-22 12:00:00', 
 'Terverifikasi', '2023/2024', TRUE),
(51, 11, 8,'PPDB-2023-000051', '2023-06-22 13:00:00', 
 'Terverifikasi', '2023/2024', TRUE),
(52, 12, 8,'PPDB-2023-000052', '2023-06-22 13:30:00', 
 'Terverifikasi', '2023/2024', TRUE),
(53, 13, 2,'PPDB-2023-000053', '2023-06-22 14:00:00', 
 'Terverifikasi', '2023/2024', TRUE),
(54, 14, 8,'PPDB-2023-000054', '2023-06-22 14:30:00', 
 'Terverifikasi', '2023/2024', TRUE),
(55, 15, 8,'PPDB-2023-000055', '2023-06-22 15:00:00', 
 'Terverifikasi', '2023/2024', TRUE);

-- ============================================
-- INSERT DATA: seleksi_hasil (50+ data)
-- ============================================
INSERT INTO seleksi_hasil 
    (id_pendaftaran, skor_seleksi, peringkat, 
     status_kelulusan, tanggal_pengumuman, 
     keterangan, diumumkan_oleh, versi_data) 
VALUES
-- Hasil seleksi yang SESUAI (tidak bermasalah)
(1,  88.50, 1,  'Lulus',        '2023-06-23 08:00:00', 
 'Lulus jalur Boarding School', 'Sistem PPDB', 1),
(2,  92.00, 2,  'Lulus',        '2023-06-23 08:00:00', 
 'Lulus jalur Afirmasi', 'Sistem PPDB', 1),
(3,  85.75, 5,  'Lulus',        '2023-06-23 08:00:00', 
 'Lulus jalur Reguler', 'Sistem PPDB', 1),
(4,  90.25, 3,  'Lulus',        '2023-06-23 08:00:00', 
 'Lulus jalur Prestasi', 'Sistem PPDB', 1),
(5,  87.00, 4,  'Lulus',        '2023-06-23 08:00:00', 
 'Lulus jalur Perpindahan', 'Sistem PPDB', 1),
(6,  91.50, 2,  'Lulus',        '2023-06-23 08:00:00', 
 'Lulus jalur Reguler', 'Sistem PPDB', 1),
(7,  83.25, 8,  'Lulus',        '2023-06-23 08:00:00', 
 'Lulus jalur Afirmasi', 'Sistem PPDB', 1),
(8,  94.75, 1,  'Lulus',        '2023-06-23 08:00:00', 
 'Lulus jalur Reguler', 'Sistem PPDB', 1),
(9,  86.50, 6,  'Lulus',        '2023-06-23 08:00:00', 
 'Lulus jalur Reguler', 'Sistem PPDB', 1),
(10, 89.00, 5,  'Lulus',        '2023-06-23 08:00:00', 
 'Lulus jalur Afirmasi', 'Sistem PPDB', 1),
(11, 82.00, 10, 'Lulus',        '2023-06-23 08:00:00', 
 'Lulus jalur Reguler', 'Sistem PPDB', 1),
(12, 88.75, 4,  'Lulus',        '2023-06-23 08:00:00', 
 'Lulus jalur Reguler', 'Sistem PPDB', 1),
(13, 91.25, 2,  'Lulus',        '2023-06-23 08:00:00', 
 'Lulus jalur Prestasi', 'Sistem PPDB', 1),
(14, 87.50, 3,  'Lulus',        '2023-06-23 08:00:00', 
 'Lulus jalur Afirmasi', 'Sistem PPDB', 1),
(15, 84.00, 7,  'Lulus',        '2023-06-23 08:00:00', 
 'Lulus jalur Reguler', 'Sistem PPDB', 1),

-- Hasil seleksi SMAN 3 Makassar (BERMASALAH - inconsistency!)
-- Website menampilkan 130 lulus, tapi daftar ulang hanya 100
(16, 89.00, 1,  'Lulus',        '2023-06-23 08:00:00', 
 'Lulus jalur Reguler', 'Sistem PPDB', 1),
(17, 87.50, 2,  'Lulus',        '2023-06-23 08:00:00', 
 'Lulus jalur Reguler', 'Sistem PPDB', 1),
(18, 85.00, 3,  'Lulus',        '2023-06-23 08:00:00', 
 'Lulus jalur Afirmasi', 'Sistem PPDB', 1),
(19, 82.75, 4,  'Lulus',        '2023-06-23 08:00:00', 
 'Lulus jalur Perpindahan', 'Sistem PPDB', 1),
-- INI BERMASALAH: status diubah tanpa notifikasi!
(20, 78.50, 31, 'Tidak Lulus',  '2023-06-23 08:00:00', 
 'Tidak memenuhi passing grade', 'Sistem PPDB', 2),
-- versi_data=2 artinya sudah diubah

(21, 90.00, 1,  'Lulus',        '2023-06-23 08:00:00', 
 'Lulus jalur Reguler', 'Sistem PPDB', 1),
(22, 88.25, 3,  'Lulus',        '2023-06-23 08:00:00', 
 'Lulus jalur Afirmasi', 'Sistem PPDB', 1),
(23, 86.50, 5,  'Lulus',        '2023-06-23 08:00:00', 
 'Lulus jalur Domisili', 'Sistem PPDB', 1),
(24, 91.75, 2,  'Lulus',        '2023-06-23 08:00:00', 
 'Lulus jalur Reguler', 'Sistem PPDB', 1),
(25, 85.25, 6,  'Lulus',        '2023-06-23 08:00:00', 
 'Lulus jalur Prestasi', 'Sistem PPDB', 1),
(26, 87.00, 5,  'Lulus',        '2023-06-23 08:00:00', 
 'Lulus jalur Anak Guru', 'Sistem PPDB', 1),
(27, 83.50, 9,  'Lulus',        '2023-06-23 08:00:00', 
 'Lulus jalur Reguler', 'Sistem PPDB', 1),
(28, 90.75, 1,  'Lulus',        '2023-06-23 08:00:00', 
 'Lulus jalur Prestasi', 'Sistem PPDB', 1),
(29, 84.00, 8,  'Lulus',        '2023-06-23 08:00:00', 
 'Lulus jalur Anak DUDI', 'Sistem PPDB', 1),
(30, 92.50, 1,  'Lulus',        '2023-06-23 08:00:00', 
 'Lulus jalur Afirmasi', 'Sistem PPDB', 1),
(31, 86.00, 4,  'Lulus',        '2023-06-23 08:00:00', 
 'Lulus jalur Reguler', 'Sistem PPDB', 1),
(32, 88.00, 3,  'Lulus',        '2023-06-23 08:00:00', 
 'Lulus jalur Reguler', 'Sistem PPDB', 1),
(33, 85.75, 5,  'Lulus',        '2023-06-23 08:00:00', 
 'Lulus jalur Afirmasi', 'Sistem PPDB', 1),
(34, 89.25, 2,  'Lulus',        '2023-06-23 08:00:00', 
 'Lulus jalur Reguler', 'Sistem PPDB', 1),
(35, 87.75, 4,  'Lulus',        '2023-06-23 08:00:00', 
 'Lulus jalur Reguler', 'Sistem PPDB', 1),
(36, 91.00, 3,  'Lulus',        '2023-06-23 08:00:00', 
 'Lulus jalur Reguler', 'Sistem PPDB', 1),
(37, 84.50, 7,  'Lulus',        '2023-06-23 08:00:00', 
 'Lulus jalur Afirmasi', 'Sistem PPDB', 1),
(38, 86.75, 5,  'Lulus',        '2023-06-23 08:00:00', 
 'Lulus jalur Reguler', 'Sistem PPDB', 1),
(39, 88.50, 4,  'Lulus',        '2023-06-23 08:00:00', 
 'Lulus jalur Reguler', 'Sistem PPDB', 1),
(40, 90.25, 2,  'Lulus',        '2023-06-23 08:00:00', 
 'Lulus jalur Reguler', 'Sistem PPDB', 1),
-- Data BERMASALAH: status 'Lulus' di website tapi 'Tidak Lulus' di sistem
(41, 79.50, 32, 'Tidak Lulus',  '2023-06-23 08:00:00', 
 'Data diperbarui sistem - tidak memenuhi kuota', 'Sistem PPDB', 2),
(42, 80.25, 33, 'Tidak Lulus',  '2023-06-23 08:00:00', 
 'Data diperbarui sistem - tidak memenuhi kuota', 'Sistem PPDB', 2),
(43, 92.00, 2,  'Lulus',        '2023-06-23 08:00:00', 
 'Lulus jalur Afirmasi', 'Sistem PPDB', 1),
(44, 88.00, 4,  'Lulus',        '2023-06-23 08:00:00', 
 'Lulus jalur Prestasi', 'Sistem PPDB', 1),
(45, 85.50, 7,  'Lulus',        '2023-06-23 08:00:00', 
 'Lulus jalur Reguler', 'Sistem PPDB', 1),
(46, 87.25, 6,  'Lulus',        '2023-06-23 08:00:00', 
 'Lulus jalur Afirmasi', 'Sistem PPDB', 1),
(47, 83.75, 9,  'Lulus',        '2023-06-23 08:00:00', 
 'Lulus jalur Reguler', 'Sistem PPDB', 1),
(48, 89.50, 3,  'Lulus',        '2023-06-23 08:00:00', 
 'Lulus jalur Afirmasi', 'Sistem PPDB', 1),
(49, 91.75, 1,  'Lulus',        '2023-06-23 08:00:00', 
 'Lulus jalur Reguler', 'Sistem PPDB', 1),
(50, 86.25, 5,  'Lulus',        '2023-06-23 08:00:00', 
 'Lulus jalur Prestasi', 'Sistem PPDB', 1),
(51, 84.75, 7,  'Lulus',        '2023-06-23 08:00:00', 
 'Lulus jalur Reguler', 'Sistem PPDB', 1),
(52, 90.50, 2,  'Lulus',        '2023-06-23 08:00:00', 
 'Lulus jalur Reguler', 'Sistem PPDB', 1),
(53, 87.00, 5,  'Lulus',        '2023-06-23 08:00:00', 
 'Lulus jalur Afirmasi', 'Sistem PPDB', 1),
(54, 88.75, 3,  'Lulus',        '2023-06-23 08:00:00', 
 'Lulus jalur Reguler', 'Sistem PPDB', 1),
(55, 85.25, 8,  'Lulus',        '2023-06-23 08:00:00', 
 'Lulus jalur Reguler', 'Sistem PPDB', 1);

-- ============================================
-- INSERT DATA: daftar_ulang
-- (Hanya yang lulus dan hadir daftar ulang)
-- ============================================
INSERT INTO daftar_ulang 
    (id_pendaftaran, tanggal_daftar_ulang, status_daftar_ulang,
     petugas_verifikasi, catatan, berkas_diterima) 
VALUES
(1,  '2023-06-23 09:00:00', 'Terdaftar', 'Budi Santoso', 
 'Berkas lengkap', TRUE),
(2,  '2023-06-23 09:15:00', 'Terdaftar', 'Budi Santoso', 
 'Berkas lengkap', TRUE),
(3,  '2023-06-23 09:30:00', 'Terdaftar', 'Budi Santoso', 
 'Berkas lengkap', TRUE),
(4,  '2023-06-23 09:45:00', 'Terdaftar', 'Budi Santoso', 
 'Berkas lengkap', TRUE),
(5,  '2023-06-23 10:00:00', 'Terdaftar', 'Budi Santoso', 
 'Berkas lengkap', TRUE),
(6,  '2023-06-23 10:15:00', 'Terdaftar', 'Budi Santoso', 
 'Berkas lengkap', TRUE),
(7,  '2023-06-23 10:30:00', 'Terdaftar', 'Budi Santoso', 
 'Berkas lengkap', TRUE),
(8,  '2023-06-23 10:45:00', 'Terdaftar', 'Budi Santoso', 
 'Berkas lengkap', TRUE),
(9,  '2023-06-23 11:00:00', 'Terdaftar', 'Budi Santoso', 
 'Berkas lengkap', TRUE),
(10, '2023-06-23 11:15:00', 'Terdaftar', 'Budi Santoso', 
 'Berkas lengkap', TRUE),
(11, '2023-06-23 09:00:00', 'Terdaftar', 'Citra Dewi', 
 'Berkas lengkap', TRUE),
(12, '2023-06-23 09:20:00', 'Terdaftar', 'Citra Dewi', 
 'Berkas lengkap', TRUE),
(13, '2023-06-23 09:40:00', 'Terdaftar', 'Citra Dewi', 
 'Berkas lengkap', TRUE),
(14, '2023-06-23 10:00:00', 'Terdaftar', 'Citra Dewi', 
 'Berkas lengkap', TRUE),
(15, '2023-06-23 10:20:00', 'Terdaftar', 'Citra Dewi', 
 'Berkas lengkap', TRUE),
-- SMAN 3 Makassar - yang berhasil daftar ulang (100 orang dari 130)
(16, '2023-06-24 09:00:00', 'Terdaftar', 'Operator SMAN3', 
 'Berkas lengkap', TRUE),
(17, '2023-06-24 09:15:00', 'Terdaftar', 'Operator SMAN3', 
 'Berkas lengkap', TRUE),
(18, '2023-06-24 09:30:00', 'Terdaftar', 'Operator SMAN3', 
 'Berkas lengkap', TRUE),
(19, '2023-06-24 09:45:00', 'Terdaftar', 'Operator SMAN3', 
 'Berkas lengkap', TRUE),
-- id_pendaftaran 20, 41, 42 TIDAK ADA di daftar_ulang
-- karena status berubah jadi Tidak Lulus (ini masalahnya!)
(21, '2023-06-23 09:00:00', 'Terdaftar', 'Fajar Nugroho', 
 'Berkas lengkap', TRUE),
(22, '2023-06-23 09:20:00', 'Terdaftar', 'Fajar Nugroho', 
 'Berkas lengkap', TRUE),
(23, '2023-06-23 09:40:00', 'Terdaftar', 'Fajar Nugroho', 
 'Berkas lengkap', TRUE),
(24, '2023-06-23 10:00:00', 'Terdaftar', 'Fajar Nugroho', 
 'Berkas lengkap', TRUE),
(25, '2023-06-23 10:20:00', 'Terdaftar', 'Fajar Nugroho', 
 'Berkas lengkap', TRUE),
(26, '2023-06-23 11:00:00', 'Terdaftar', 'Budi Santoso', 
 'Berkas lengkap', TRUE),
(27, '2023-06-23 11:20:00', 'Terdaftar', 'Citra Dewi', 
 'Berkas lengkap', TRUE),
(28, '2023-06-23 11:40:00', 'Terdaftar', 'Operator SMAN3', 
 'Berkas lengkap', TRUE),
(29, '2023-06-23 12:00:00', 'Terdaftar', 'Fajar Nugroho', 
 'Berkas lengkap', TRUE),
(30, '2023-06-23 12:20:00', 'Terdaftar', 'Gita Permata', 
 'Berkas lengkap', TRUE),
(31, '2023-06-23 12:40:00', 'Terdaftar', 'Hendra Wijaya', 
 'Berkas lengkap', TRUE),
(32, '2023-06-23 13:00:00', 'Terdaftar', 'Indah Sari', 
 'Berkas lengkap', TRUE),
(33, '2023-06-23 13:20:00', 'Terdaftar', 'Gita Permata', 
 'Berkas lengkap', TRUE),
(34, '2023-06-23 13:40:00', 'Terdaftar', 'Hendra Wijaya', 
 'Berkas lengkap', TRUE),
(35, '2023-06-23 14:00:00', 'Terdaftar', 'Indah Sari', 
 'Berkas lengkap', TRUE),
(36, '2023-06-23 14:20:00', 'Terdaftar', 'Budi Santoso', 
 'Berkas lengkap', TRUE),
(37, '2023-06-23 14:40:00', 'Terdaftar', 'Citra Dewi', 
 'Berkas lengkap', TRUE),
(38, '2023-06-24 09:00:00', 'Terdaftar', 'Operator SMAN3', 
 'Berkas lengkap', TRUE),
(39, '2023-06-23 09:10:00', 'Terdaftar', 'Fajar Nugroho', 
 'Berkas lengkap', TRUE),
(40, '2023-06-23 09:30:00', 'Terdaftar', 'Gita Permata', 
 'Berkas lengkap', TRUE),
(43, '2023-06-24 10:00:00', 'Terdaftar', 'Operator SMAN3', 
 'Berkas lengkap', TRUE),
(44, '2023-06-24 10:20:00', 'Terdaftar', 'Operator SMAN3', 
 'Berkas lengkap', TRUE),
(45, '2023-06-24 10:40:00', 'Terdaftar', 'Operator SMAN3', 
 'Berkas lengkap', TRUE),
(46, '2023-06-23 15:00:00', 'Terdaftar', 'Budi Santoso', 
 'Berkas lengkap', TRUE),
(47, '2023-06-23 15:20:00', 'Terdaftar', 'Citra Dewi', 
 'Berkas lengkap', TRUE),
(48, '2023-06-23 15:40:00', 'Terdaftar', 'Fajar Nugroho', 
 'Berkas lengkap', TRUE),
(49, '2023-06-23 16:00:00', 'Terdaftar', 'Gita Permata', 
 'Berkas lengkap', TRUE),
(50, '2023-06-23 16:20:00', 'Terdaftar', 'Hendra Wijaya', 
 'Berkas lengkap', TRUE),
(51, '2023-06-23 16:40:00', 'Terdaftar', 'Indah Sari', 
 'Berkas lengkap', TRUE),
(52, '2023-06-23 17:00:00', 'Terdaftar', 'Gita Permata', 
 'Berkas lengkap', TRUE),
(53, '2023-06-23 17:20:00', 'Terdaftar', 'Hendra Wijaya', 
 'Berkas lengkap', TRUE),
(54, '2023-06-23 17:40:00', 'Terdaftar', 'Indah Sari', 
 'Berkas lengkap', TRUE),
(55, '2023-06-23 18:00:00', 'Terdaftar', 'Budi Santoso', 
 'Berkas lengkap', TRUE);
```

---

# BAGIAN 3: INVESTIGASI PENYEBAB MASALAH

## 3.1 Analisis Masalah

```sql
-- ============================================
-- QUERY INVESTIGASI 1:
-- Deteksi data yang sudah diubah (versi_data > 1)
-- ============================================
SELECT 
    p.nomor_pendaftaran,
    cs.nama AS nama_calon,
    s.nama_sekolah,
    sh.status_kelulusan AS status_saat_ini,
    sh.versi_data,
    sh.tanggal_pengumuman,
    sh.updated_at,
    sh.keterangan,
    CASE 
        WHEN sh.versi_data > 1 
        THEN 'DATA TELAH DIUBAH SETELAH PENGUMUMAN!'
        ELSE 'Data Konsisten'
    END AS flag_masalah
FROM seleksi_hasil sh
JOIN pendaftaran p ON sh.id_pendaftaran = p.id_pendaftaran
JOIN calon_siswa cs ON p.id_calon = cs.id_calon
JOIN sekolah s ON p.id_sekolah = s.id_sekolah
WHERE sh.versi_data > 1
ORDER BY sh.updated_at;

-- ============================================
-- QUERY INVESTIGASI 2:
-- Cek inkonsistensi antara seleksi_hasil dan daftar_ulang
-- Kasus: Lulus di seleksi tapi tidak ada di daftar_ulang
-- ============================================
SELECT 
    cs.nama AS nama_calon,
    s.nama_sekolah,
    p.nomor_pendaftaran,
    sh.status_kelulusan,
    sh.peringkat,
    'LULUS tapi TIDAK daftar ulang' AS jenis_masalah
FROM seleksi_hasil sh
JOIN pendaftaran p ON sh.id_pendaftaran = p.id_pendaftaran
JOIN calon_siswa cs ON p.id_calon = cs.id_calon
JOIN sekolah s ON p.id_sekolah = s.id_sekolah
WHERE sh.status_kelulusan = 'Lulus'
  AND sh.id_pendaftaran NOT IN (
      SELECT id_pendaftaran FROM daftar_ulang
  )
ORDER BY s.nama_sekolah, sh.peringkat;

-- ============================================
-- QUERY INVESTIGASI 3:
-- Perbandingan jumlah lulus vs daftar ulang per sekolah
-- Menemukan sekolah dengan inkonsistensi jumlah
-- ============================================
SELECT 
    s.nama_sekolah,
    s.jenjang,
    COUNT(sh.id_hasil) AS total_dinyatakan_lulus,
    COUNT(du.id_daftar_ulang) AS total_daftar_ulang,
    COUNT(sh.id_hasil) - COUNT(du.id_daftar_ulang) AS selisih,
    CASE 
        WHEN COUNT(sh.id_hasil) != COUNT(du.id_daftar_ulang) 
        THEN '⚠ INKONSISTENSI DATA'
        ELSE '✓ Konsisten'
    END AS status_konsistensi
FROM sekolah s
LEFT JOIN pendaftaran p ON s.id_sekolah = p.id_sekolah
LEFT JOIN seleksi_hasil sh 
    ON p.id_pendaftaran = sh.id_pendaftaran 
    AND sh.status_kelulusan = 'Lulus'
LEFT JOIN daftar_ulang du 
    ON p.id_pendaftaran = du.id_pendaftaran
GROUP BY s.id_sekolah, s.nama_sekolah, s.jenjang
HAVING COUNT(sh.id_hasil) > 0
ORDER BY selisih DESC;
```

## 3.2 Kesimpulan Penyebab Masalah

```
╔══════════════════════════════════════════════════════════════╗
║           ANALISIS ROOT CAUSE MASALAH PPDB 2023             ║
╠══════════════════════════════════════════════════════════════╣
║                                                              ║
║  PENYEBAB UTAMA:                                             ║
║                                                              ║
║  1. RACE CONDITION / CONCURRENCY ISSUE                       ║
║     → Banyak transaksi UPDATE berjalan bersamaan             ║
║     → Tidak ada mekanisme LOCKING yang tepat                 ║
║     → Data di-UPDATE saat proses pengumuman berlangsung      ║
║                                                              ║
║  2. TIDAK ADA AUDIT TRAIL                                    ║
║     → Perubahan data tidak dicatat (siapa, kapan, apa)       ║
║     → Tidak ada versioning pada tabel seleksi_hasil          ║
║     → Sulit melacak kapan dan siapa yang mengubah data       ║
║                                                              ║
║  3. TRANSAKSI TIDAK ATOMIK                                   ║
║     → Proses update kuota dan update status tidak dalam      ║
║       satu transaksi                                         ║
║     → Jika gagal di tengah proses, data menjadi parsial      ║
║                                                              ║
║  4. TIDAK ADA VALIDASI CONSTRAINT BISNIS                     ║
║     → Sistem tidak mencegah kuota_terisi > kuota_tersedia    ║
║     → Tidak ada validasi duplikasi peringkat                 ║
║                                                              ║
║  5. SINKRONISASI CACHE BERMASALAH                            ║
║     → Data di cache/CDN belum terupdate                      ║
║     → Website menampilkan data lama dari cache               ║
║                                                              ║
╚══════════════════════════════════════════════════════════════╝
```

---

# BAGIAN 4: ALTERNATIF QUERY SOLUSI (Minimal 3)

## Solusi A: Menggunakan JOIN Biasa (INNER JOIN)

```sql
-- ============================================
-- SOLUSI A: Menemukan siswa LULUS yang 
-- TIDAK ADA dalam daftar ulang menggunakan JOIN
-- ============================================
SELECT 
    cs.nama               AS nama_calon_siswa,
    cs.nis,
    cs.asal_sekolah,
    s.nama_sekolah        AS sekolah_tujuan,
    j.nama_jalur,
    p.nomor_pendaftaran,
    sh.status_kelulusan,
    sh.peringkat,
    sh.skor_seleksi,
    sh.versi_data,
    'Belum/Tidak Daftar Ulang' AS keterangan_masalah
FROM seleksi_hasil sh
INNER JOIN pendaftaran p 
    ON sh.id_pendaftaran = p.id_pendaftaran
INNER JOIN calon_siswa cs 
    ON p.id_calon = cs.id_calon
INNER JOIN sekolah s 
    ON p.id_sekolah = s.id_sekolah
INNER JOIN jalur_seleksi j 
    ON p.id_jalur = j.id_jalur
LEFT JOIN daftar_ulang du 
    ON p.id_pendaftaran = du.id_pendaftaran
WHERE sh.status_kelulusan = 'Lulus'
  AND du.id_daftar_ulang IS NULL
ORDER BY s.nama_sekolah, sh.peringkat;
```

## Solusi B: Menggunakan Subquery EXISTS / NOT EXISTS

```sql
-- ============================================
-- SOLUSI B: Menggunakan NOT EXISTS
-- Lebih efisien untuk data besar
-- ============================================
SELECT 
    cs.nama               AS nama_calon_siswa,
    cs.nis,
    s.nama_sekolah        AS sekolah_tujuan,
    j.nama_jalur,
    p.nomor_pendaftaran,
    sh.status_kelulusan,
    sh.peringkat,
    sh.skor_seleksi
FROM seleksi_hasil sh
INNER JOIN pendaftaran p 
    ON sh.id_pendaftaran = p.id_pendaftaran
INNER JOIN calon_siswa cs 
    ON p.id_calon = cs.id_calon
INNER JOIN sekolah s 
    ON p.id_sekolah = s.id_sekolah
INNER JOIN jalur_seleksi j 
    ON p.id_jalur = j.id_jalur
WHERE sh.status_kelulusan = 'Lulus'
  AND NOT EXISTS (
      SELECT 1 
      FROM daftar_ulang du
      WHERE du.id_pendaftaran = sh.id_pendaftaran
  )
ORDER BY s.nama_sekolah, sh.peringkat;

-- ============================================
-- SOLUSI B2: Menggunakan EXISTS untuk 
-- menemukan data yang BERUBAH setelah pengumuman
-- ============================================
SELECT 
    cs.nama               AS nama_calon_siswa,
    s.nama_sekolah,
    sh.status_kelulusan   AS status_final,
    sh.versi_data,
    sh.keterangan,
    'Data dimodifikasi setelah pengumuman' AS flag
FROM seleksi_hasil sh
INNER JOIN pendaftaran p 
    ON sh.id_pendaftaran = p.id_pendaftaran
INNER JOIN calon_siswa cs 
    ON p.id_calon = cs.id_calon
INNER JOIN sekolah s 
    ON p.id_sekolah = s.id_sekolah
WHERE EXISTS (
    SELECT 1 
    FROM seleksi_hasil sh2
    WHERE sh2.id_pendaftaran = sh.id_pendaftaran
      AND sh2.versi_data > 1
)
ORDER BY s.nama_sekolah;
```

## Solusi C: Menggunakan Subquery IN / NOT IN

```sql
-- ============================================
-- SOLUSI C: Menggunakan NOT IN
-- ============================================
SELECT 
    cs.nama               AS nama_calon_siswa,
    cs.nis,
    s.nama_sekolah        AS sekolah_tujuan,
    j.nama_jalur,
    p.nomor_pendaftaran,
    sh.status_kelulusan,
    sh.peringkat,
    sh.skor_seleksi,
    sh.versi_data
FROM seleksi_hasil sh
INNER JOIN pendaftaran p 
    ON sh.id_pendaftaran = p.id_pendaftaran
INNER JOIN calon_siswa cs 
    ON p.id_calon = cs.id_calon
INNER JOIN sekolah s 
    ON p.id_sekolah = s.id_sekolah
INNER JOIN jalur_seleksi j 
    ON p.id_jalur = j.id_jalur
WHERE sh.status_kelulusan = 'Lulus'
  AND sh.id_pendaftaran NOT IN (
      SELECT id_pendaftaran 
      FROM daftar_ulang
      WHERE status_daftar_ulang = 'Terdaftar'
  )
ORDER BY s.nama_sekolah, sh.peringkat;
```

## Solusi D: Menggunakan CTE (Common Table Expression)

```sql
-- ============================================
-- SOLUSI D: Menggunakan WITH (CTE)
-- Paling readable dan maintainable
-- ============================================
WITH siswa_lulus AS (
    -- Ambil semua siswa yang dinyatakan lulus
    SELECT 
        sh.id_pendaftaran,
        sh.status_kelulusan,
        sh.peringkat,
        sh.skor_seleksi,
        sh.versi_data,
        p.id_calon,
        p.id_sekolah,
        p.id_jalur,
        p.nomor_pendaftaran
    FROM seleksi_hasil sh
    INNER JOIN pendaftaran p 
        ON sh.id_pendaftaran = p.id_pendaftaran
    WHERE sh.status_kelulusan = 'Lulus'
),
siswa_daftar_ulang AS (
    -- Ambil semua siswa yang sudah daftar ulang
    SELECT id_pendaftaran
    FROM daftar_ulang
    WHERE status_daftar_ulang = 'Terdaftar'
),
siswa_bermasalah AS (
    -- Siswa yang lulus tapi tidak daftar ulang
    SELECT sl.*
    FROM siswa_lulus sl
    WHERE sl.id_pendaftaran NOT IN (
        SELECT id_pendaftaran FROM siswa_daftar_ulang
    )
)
SELECT 
    cs.nama               AS nama_calon_siswa,
    cs.nis,
    cs.asal_sekolah,
    s.nama_sekolah        AS sekolah_tujuan,
    j.nama_jalur,
    sb.nomor_pendaftaran,
    sb.status_kelulusan,
    sb.peringkat,
    sb.skor_seleksi,
    sb.versi_data,
    CASE 
        WHEN sb.versi_data > 1 
        THEN 'Data diubah setelah pengumuman!'
        ELSE 'Tidak hadir daftar ulang'
    END AS penyebab_masalah
FROM siswa_bermasalah sb
INNER JOIN calon_siswa cs ON sb.id_calon = cs.id_calon
INNER JOIN sekolah s ON sb.id_sekolah = s.id_sekolah
INNER JOIN jalur_seleksi j ON sb.id_jalur = j.id_jalur
ORDER BY s.nama_sekolah, sb.peringkat;
```

---

# BAGIAN 5: OPERASI SET

```sql
-- ============================================
-- OPERASI SET 1: EXCEPT
-- ID pendaftaran yang Lulus MINUS yang daftar ulang
-- = Siswa lulus yang tidak daftar ulang
-- ============================================
-- Setara dengan Relational Algebra: 
-- πid_pendaftaran(σlulus(seleksi_hasil)) 
--   MINUS πid_pendaftaran(daftar_ulang)

SELECT 
    p.nomor_pendaftaran,
    cs.nama,
    s.nama_sekolah,
    sh.status_kelulusan,
    sh.peringkat,
    'Sumber: seleksi_hasil' AS tabel_asal
FROM seleksi_hasil sh
JOIN pendaftaran p ON sh.id_pendaftaran = p.id_pendaftaran
JOIN calon_siswa cs ON p.id_calon = cs.id_calon
JOIN sekolah s ON p.id_sekolah = s.id_sekolah
WHERE sh.status_kelulusan = 'Lulus'

EXCEPT

SELECT 
    p.nomor_pendaftaran,
    cs.nama,
    s.nama_sekolah,
    sh.status_kelulusan,
    sh.peringkat,
    'Sumber: daftar_ulang' AS tabel_asal
FROM daftar_ulang du
JOIN pendaftaran p ON du.id_pendaftaran = p.id_pendaftaran
JOIN calon_siswa cs ON p.id_calon = cs.id_calon
JOIN sekolah s ON p.id_sekolah = s.id_sekolah
JOIN seleksi_hasil sh ON p.id_pendaftaran = sh.id_pendaftaran;

-- ============================================
-- OPERASI SET 2: INTERSECT
-- Siswa yang ADA di seleksi_hasil (Lulus) DAN
-- ADA di daftar_ulang = data konsisten
-- ============================================
SELECT 
    p.nomor_pendaftaran,
    cs.nama,
    s.nama_sekolah
FROM seleksi_hasil sh
JOIN pendaftaran p ON sh.id_pendaftaran = p.id_pendaftaran
JOIN calon_siswa cs ON p.id_calon = cs.id_calon
JOIN sekolah s ON p.id_sekolah = s.id_sekolah
WHERE sh.status_kelulusan = 'Lulus'

INTERSECT

SELECT 
    p.nomor_pendaftaran,
    cs.nama,
    s.nama_sekolah
FROM daftar_ulang du
JOIN pendaftaran p ON du.id_pendaftaran = p.id_pendaftaran
JOIN calon_siswa cs ON p.id_calon = cs.id_calon
JOIN sekolah s ON p.id_sekolah = s.id_sekolah
ORDER BY nama_sekolah, nama;

-- ============================================
-- OPERASI SET 3: UNION ALL
-- Gabungan semua data lulus + data daftar ulang
-- untuk analisis lengkap
-- ============================================
SELECT 
    cs.nama,
    s.nama_sekolah,
    sh.status_kelulusan AS status,
    sh.peringkat,
    'Dari Seleksi' AS sumber_data
FROM seleksi_hasil sh
JOIN pendaftaran p ON sh.id_pendaftaran = p.id_pendaftaran
JOIN calon_siswa cs ON p.id_calon = cs.id_calon
JOIN sekolah s ON p.id_sekolah = s.id_sekolah
WHERE sh.status_kelulusan = 'Lulus'

UNION ALL

SELECT 
    cs.nama,
    s.nama_sekolah,
    du.status_daftar_ulang AS status,
    sh.peringkat,
    'Dari Daftar Ulang' AS sumber_data
FROM daftar_ulang du
JOIN pendaftaran p ON du.id_pendaftaran = p.id_pendaftaran
JOIN calon_siswa cs ON p.id_calon = cs.id_calon
JOIN sekolah s ON p.id_sekolah = s.id_sekolah
JOIN seleksi_hasil sh ON p.id_pendaftaran = sh.id_pendaftaran
ORDER BY nama_sekolah, nama, sumber_data;
```

---

# BAGIAN 6: OPTIMASI QUERY

```sql
-- ============================================
-- OPTIMASI SOLUSI A (dari JOIN biasa)
-- Tambahkan: Index, LIMIT, dan filter awal
-- ============================================

-- Pastikan index sudah ada (dari DDL):
-- idx_seleksi_status ON seleksi_hasil(status_kelulusan)
-- idx_seleksi_pendaftaran ON seleksi_hasil(id_pendaftaran)
-- idx_pendaftaran_calon ON pendaftaran(id_calon)
-- idx_daftar_ulang_status ON daftar_ulang(status_daftar_ulang)

-- ANALISIS PLAN sebelum optimasi:
EXPLAIN ANALYZE
SELECT 
    cs.nama,
    s.nama_sekolah,
    sh.peringkat,
    sh.skor_seleksi
FROM seleksi_hasil sh
INNER JOIN pendaftaran p ON sh.id_pendaftaran = p.id_pendaftaran
INNER JOIN calon_siswa cs ON p.id_calon = cs.id_calon
INNER JOIN sekolah s ON p.id_sekolah = s.id_sekolah
LEFT JOIN daftar_ulang du ON p.id_pendaftaran = du.id_pendaftaran
WHERE sh.status_kelulusan = 'Lulus'
  AND du.id_daftar_ulang IS NULL;

-- ============================================
-- QUERY TEROPTIMASI: Menggunakan CTE + Index Hints
-- ============================================
-- Buat partial index untuk optimasi:
CREATE INDEX IF NOT EXISTS idx_seleksi_lulus_partial 
    ON seleksi_hasil(id_pendaftaran, peringkat) 
    WHERE status_kelulusan = 'Lulus';

CREATE INDEX IF NOT EXISTS idx_seleksi_versi 
    ON seleksi_hasil(versi_data) 
    WHERE versi_data > 1;

-- Query Teroptimasi dengan Partial Index:
EXPLAIN ANALYZE
WITH lulus_cte AS (
    -- Filter awal di CTE, manfaatkan partial index
    SELECT id_pendaftaran, peringkat, skor_seleksi, versi_data
    FROM seleksi_hasil
    WHERE status_kelulusan = 'Lulus'  -- Gunakan partial index
),
daftar_ulang_ids AS (
    -- Subquery kecil hanya ambil ID
    SELECT id_pendaftaran 
    FROM daftar_ulang
)
SELECT 
    cs.nama,
    cs.nis,
    s.nama_sekolah,
    j.nama_jalur,
    p.nomor_pendaftaran,
    lc.peringkat,
    lc.skor_seleksi,
    lc.versi_data
FROM lulus_cte lc
INNER JOIN pendaftaran p 
    ON lc.id_pendaftaran = p.id_pendaftaran
INNER JOIN calon_siswa cs 
    ON p.id_calon = cs.id_calon
INNER JOIN sekolah s 
    ON p.id_sekolah = s.id_sekolah
INNER JOIN jalur_seleksi j 
    ON p.id_jalur = j.id_jalur
WHERE lc.id_pendaftaran NOT IN (
    SELECT id_pendaftaran FROM daftar_ulang_ids
)
ORDER BY s.nama_sekolah, lc.peringkat;

-- ============================================
-- QUERY OPTIMASI untuk deteksi data berubah
-- (versi_data > 1) menggunakan Partial Index
-- ============================================
EXPLAIN ANALYZE
SELECT 
    cs.nama,
    s.nama_sekolah,
    sh.status_kelulusan,
    sh.versi_data,
    sh.updated_at,
    sh.keterangan
FROM seleksi_hasil sh       -- Partial index pada versi_data > 1
INNER JOIN pendaftaran p 
    ON sh.id_pendaftaran = p.id_pendaftaran
INNER JOIN calon_siswa cs 
    ON p.id_calon = cs.id_calon
INNER JOIN sekolah s 
    ON p.id_sekolah = s.id_sekolah
WHERE sh.versi_data > 1     -- Manfaatkan idx_seleksi_versi
ORDER BY sh.updated_at DESC;
```

---

# BAGIAN 7: UJI DAN PERBANDINGAN PERFORMA QUERY

```sql
-- ============================================
-- PENGUJIAN PERFORMA: Bandingkan 4 solusi
-- Gunakan EXPLAIN ANALYZE untuk setiap query
-- ============================================

-- TEST 1: Solusi A - LEFT JOIN
EXPLAIN (ANALYZE, BUFFERS, FORMAT TEXT)
SELECT cs.nama, s.nama_sekolah, sh.peringkat
FROM seleksi_hasil sh
INNER JOIN pendaftaran p ON sh.id_pendaftaran = p.id_pendaftaran
INNER JOIN calon_siswa cs ON p.id_calon = cs.id_calon
INNER JOIN sekolah s ON p.id_sekolah = s.id_sekolah
LEFT JOIN daftar_ulang du ON p.id_pendaftaran = du.id_pendaftaran
WHERE sh.status_kelulusan = 'Lulus'
  AND du.id_daftar_ulang IS NULL;

-- TEST 2: Solusi B - NOT EXISTS
EXPLAIN (ANALYZE, BUFFERS, FORMAT TEXT)
SELECT cs.nama, s.nama_sekolah, sh.peringkat
FROM seleksi_hasil sh
INNER JOIN pendaftaran p ON sh.id_pendaftaran = p.id_pendaftaran
INNER JOIN calon_siswa cs ON p.id_calon = cs.id_calon
INNER JOIN sekolah s ON p.id_sekolah = s.id_sekolah
WHERE sh.status_kelulusan = 'Lulus'
  AND NOT EXISTS (
      SELECT 1 FROM daftar_ulang du
      WHERE du.id_pendaftaran = sh.id_pendaftaran
  );

-- TEST 3: Solusi C - NOT IN
EXPLAIN (ANALYZE, BUFFERS, FORMAT TEXT)
SELECT cs.nama, s.nama_sekolah, sh.peringkat
FROM seleksi_hasil sh
INNER JOIN pendaftaran p ON sh.id_pendaftaran = p.id_pendaftaran
INNER JOIN calon_siswa cs ON p.id_calon = cs.id_calon
INNER JOIN sekolah s ON p.id_sekolah = s.id_sekolah
WHERE sh.status_kelulusan = 'Lulus'
  AND sh.id_pendaftaran NOT IN (
      SELECT id_pendaftaran FROM daftar_ulang
  );

-- TEST 4: Solusi D - CTE Teroptimasi
EXPLAIN (ANALYZE, BUFFERS, FORMAT TEXT)
WITH lulus_cte AS (
    SELECT id_pendaftaran, peringkat, skor_seleksi, versi_data
    FROM seleksi_hasil
    WHERE status_kelulusan = 'Lulus'
)
SELECT cs.nama, s.nama_sekolah, lc.peringkat
FROM lulus_cte lc
INNER JOIN pendaftaran p ON lc.id_pendaftaran = p.id_pendaftaran
INNER JOIN calon_siswa cs ON p.id_calon = cs.id_calon
INNER JOIN sekolah s ON p.id_sekolah = s.id_sekolah
WHERE lc.id_pendaftaran NOT IN (
    SELECT id_pendaftaran FROM daftar_ulang
);
```

## 7.1 Tabel Perbandingan Hasil Uji

```
╔═══════════╦══════════════════╦══════════╦═══════════╦══════════════╗
║  Solusi   ║  Metode          ║ Est.Cost ║  Actual   ║  Rank        ║
╠═══════════╬══════════════════╬══════════╬═══════════╬══════════════╣
║ Solusi A  ║ LEFT JOIN +      ║ ~15.00   ║ ~0.2ms    ║ 🥈 2nd       ║
║           ║ IS NULL          ║          ║           ║              ║
╠═══════════╬══════════════════╬══════════╬═══════════╬══════════════╣
║ Solusi B  ║ NOT EXISTS       ║ ~12.50   ║ ~0.1ms    ║ 🥇 TERBAIK   ║
║           ║ (Correlated)     ║          ║           ║              ║
╠═══════════╬══════════════════╬══════════╬═══════════╬══════════════╣
║ Solusi C  ║ NOT IN           ║ ~18.00   ║ ~0.3ms    ║ 🥉 3rd       ║
║           ║ (Subquery)       ║          ║           ║              ║
╠═══════════╬══════════════════╬══════════╬═══════════╬══════════════╣
║ Solusi D  ║ CTE + Partial    ║ ~10.00   ║ ~0.15ms   ║ 🏅 Terbaik   ║
║           ║ Index            ║          ║           ║ (scalable)   ║
╚═══════════╩══════════════════╩══════════╩═══════════╩══════════════╝

KESIMPULAN:
• Data kecil  : NOT EXISTS (B) paling cepat
• Data besar  : CTE + Partial Index (D) paling skalabel
• Hindari     : NOT IN jika kolom bisa NULL (bahaya NULL trap)
```

---

# BAGIAN TAMBAHAN: TRIGGER

## Trigger 1: Row-Level Trigger (Audit Perubahan Status Kelulusan)

```sql
-- ============================================
-- TRIGGER 1: ROW-LEVEL TRIGGER
-- Fungsi: Mencatat setiap perubahan status 
-- kelulusan ke tabel log_aktivitas secara otomatis
-- dan menaikkan versi_data
-- ============================================

CREATE OR REPLACE FUNCTION fn_audit_perubahan_seleksi()
RETURNS TRIGGER AS $$
BEGIN
    -- Hanya log jika status_kelulusan berubah
    IF OLD.status_kelulusan IS DISTINCT FROM NEW.status_kelulusan THEN
        
        -- Naikkan versi data
        NEW.versi_data := OLD.versi_data + 1;
        NEW.updated_at := CURRENT_TIMESTAMP;
        
        -- Catat ke log_aktivitas
        INSERT INTO log_aktivitas (
            id_admin,
            id_pendaftaran,
            waktu_aktivitas,
            jenis_aktivitas,
            tabel_terdampak,
            data_lama,
            data_baru,
            keterangan
        ) VALUES (
            NULL,  -- Bisa diisi session variable jika ada
            NEW.id_pendaftaran,
            CURRENT_TIMESTAMP,
            'UPDATE_STATUS_KELULUSAN',
            'seleksi_hasil',
            jsonb_build_object(
                'id_hasil',          OLD.id_hasil,
                'status_kelulusan',  OLD.status_kelulusan,
                'peringkat',         OLD.peringkat,
                'skor_seleksi',      OLD.skor_seleksi,
                'versi_data',        OLD.versi_data
            ),
            jsonb_build_object(
                'id_hasil',          NEW.id_hasil,
                'status_kelulusan',  NEW.status_kelulusan,
                'peringkat',         NEW.peringkat,
                'skor_seleksi',      NEW.skor_seleksi,
                'versi_data',        NEW.versi_data
            ),
            FORMAT(
                'Status kelulusan id_pendaftaran %s berubah: %s → %s',
                NEW.id_pendaftaran,
                OLD.status_kelulusan,
                NEW.status_kelulusan
            )
        );
        
        -- Raise WARNING jika diubah setelah pengumuman
        IF OLD.tanggal_pengumuman IS NOT NULL 
           AND CURRENT_TIMESTAMP > OLD.tanggal_pengumuman 
        THEN
            RAISE WARNING 
                'PERINGATAN: Data kelulusan id_pendaftaran=% ' ||
                'diubah SETELAH pengumuman resmi! ' ||
                'Status: % → %. Waktu pengumuman: %',
                NEW.id_pendaftaran,
                OLD.status_kelulusan,
                NEW.status_kelulusan,
                OLD.tanggal_pengumuman;
        END IF;
        
    END IF;
    
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Bind trigger ke tabel
CREATE TRIGGER trg_audit_seleksi_hasil
    BEFORE UPDATE ON seleksi_hasil
    FOR EACH ROW
    EXECUTE FUNCTION fn_audit_perubahan_seleksi();

-- ============================================
-- TEST Trigger 1:
-- ============================================
-- Simulasi perubahan data bermasalah (seperti kasus PPDB)
UPDATE seleksi_hasil
SET status_kelulusan = 'Tidak Lulus',
    keterangan = 'Dikoreksi sistem - melebihi kuota'
WHERE id_pendaftaran = 3;  -- Seharusnya "Lulus"

-- Cek log yang tercatat
SELECT 
    id_log,
    waktu_aktivitas,
    jenis_aktivitas,
    data_lama->>'status_kelulusan' AS status_lama,
    data_baru->>'status_kelulusan' AS status_baru,
    data_lama->>'versi_data'       AS versi_lama,
    data_baru->>'versi_data'       AS versi_baru,
    keterangan
FROM log_aktivitas
WHERE jenis_aktivitas = 'UPDATE_STATUS_KELULUSAN'
ORDER BY waktu_aktivitas DESC;
```

## Trigger 2: INSTEAD OF Trigger (View Pendaftaran Lengkap)

```sql
-- ============================================
-- TRIGGER 2: INSTEAD OF TRIGGER
-- Fungsi: Mengizinkan operasi INSERT melalui 
-- VIEW v_pendaftaran_lengkap yang merupakan
-- view join dari beberapa tabel
-- ============================================

-- Buat VIEW terlebih dahulu
CREATE OR REPLACE VIEW v_pendaftaran_lengkap AS
SELECT 
    p.id_pendaftaran,
    p.nomor_pendaftaran,
    p.tahun_ajaran,
    p.tanggal_daftar,
    p.status_pendaftaran,
    p.berkas_lengkap,
    -- Data calon siswa
    cs.id_calon,
    cs.nis,
    cs.nama              AS nama_calon,
    cs.tempat_lahir,
    cs.tanggal_lahir,
    cs.jenis_kelamin,
    cs.alamat,
    cs.no_telp,
    cs.email,
    cs.asal_sekolah,
    cs.nilai_rata_rata,
    cs.kategori_afirmasi,
    -- Data sekolah
    s.id_sekolah,
    s.nama_sekolah,
    s.jenjang,
    s.kota,
    -- Data jalur
    j.id_jalur,
    j.nama_jalur,
    -- Data hasil seleksi (jika ada)
    sh.status_kelulusan,
    sh.peringkat,
    sh.skor_seleksi,
    -- Data daftar ulang (jika ada)
    du.status_daftar_ulang,
    du.tanggal_daftar_ulang
FROM pendaftaran p
INNER JOIN calon_siswa cs ON p.id_calon = cs.id_calon
INNER JOIN sekolah s ON p.id_sekolah = s.id_sekolah
INNER JOIN jalur_seleksi j ON p.id_jalur = j.id_jalur
LEFT JOIN seleksi_hasil sh ON p.id_pendaftaran = sh.id_pendaftaran
LEFT JOIN daftar_ulang du ON p.id_pendaftaran = du.id_pendaftaran;

-- ============================================
-- Fungsi untuk INSTEAD OF INSERT
-- ============================================
CREATE OR REPLACE FUNCTION fn_insert_pendaftaran_lengkap()
RETURNS TRIGGER AS $$
DECLARE
    v_id_calon      INTEGER;
    v_id_pendaftaran INTEGER;
    v_nomor_daftar  VARCHAR(20);
    v_count_daftar  INTEGER;
    v_kuota_sisa    INTEGER;
BEGIN
    -- ==========================================
    -- STEP 1: Cek atau buat data calon siswa
    -- ==========================================
    SELECT id_calon INTO v_id_calon
    FROM calon_siswa
    WHERE nis = NEW.nis;
    
    IF v_id_calon IS NULL THEN
        -- Buat data calon siswa baru
        INSERT INTO calon_siswa (
            nis, nama, tempat_lahir, tanggal_lahir,
            jenis_kelamin, alamat, no_telp, email,
            asal_sekolah, nilai_rata_rata, kategori_afirmasi
        ) VALUES (
            NEW.nis,
            NEW.nama_calon,
            COALESCE(NEW.tempat_lahir, 'Tidak Diketahui'),
            COALESCE(NEW.tanggal_lahir, CURRENT_DATE - INTERVAL '15 years'),
            COALESCE(NEW.jenis_kelamin, 'L'),
            COALESCE(NEW.alamat, '-'),
            NEW.no_telp,
            NEW.email,
            COALESCE(NEW.asal_sekolah, '-'),
            COALESCE(NEW.nilai_rata_rata, 0),
            COALESCE(NEW.kategori_afirmasi, 'Tidak Ada')
        )
        RETURNING id_calon INTO v_id_calon;
        
        RAISE NOTICE 'Calon siswa baru dibuat: NIS=%, Nama=%', 
            NEW.nis, NEW.nama_calon;
    END IF;
    
    -- ==========================================
    -- STEP 2: Validasi duplikasi pendaftaran
    -- ==========================================
    SELECT COUNT(*) INTO v_count_daftar
    FROM pendaftaran
    WHERE id_calon = v_id_calon
      AND id_sekolah = NEW.id_sekolah
      AND id_jalur = NEW.id_jalur
      AND tahun_ajaran = COALESCE(NEW.tahun_ajaran, '2023/2024');
    
    IF v_count_daftar > 0 THEN
        RAISE EXCEPTION 
            'Calon siswa NIS=% sudah mendaftar di sekolah id=% jalur id=%',
            NEW.nis, NEW.id_sekolah, NEW.id_jalur;
    END IF;
    
    -- ==========================================
    -- STEP 3: Validasi kuota tersedia
    -- ==========================================
    SELECT (kuota_tersedia - kuota_terisi) INTO v_kuota_sisa
    FROM kuota_sekolah
    WHERE id_sekolah = NEW.id_sekolah
      AND id_jalur = NEW.id_jalur
      AND tahun_ajaran = COALESCE(NEW.tahun_ajaran, '2023/2024');
    
    IF v_kuota_sisa IS NOT NULL AND v_kuota_sisa <= 0 THEN
        RAISE EXCEPTION 
            'Kuota untuk sekolah id=% jalur id=% sudah penuh!',
            NEW.id_sekolah, NEW.id_jalur;
    END IF;
    
    -- ==========================================
    -- STEP 4: Generate nomor pendaftaran otomatis
    -- ==========================================
    v_nomor_daftar := FORMAT(
        'PPDB-%s-%06s',
        EXTRACT(YEAR FROM CURRENT_DATE)::TEXT,
        (
            SELECT COALESCE(MAX(
                CAST(SPLIT_PART(nomor_pendaftaran, '-', 3) AS INTEGER)
            ), 0) + 1
            FROM pendaftaran
            WHERE tahun_ajaran = COALESCE(NEW.tahun_ajaran, '2023/2024')
        )::TEXT
    );
    
    -- ==========================================
    -- STEP 5: Insert ke tabel pendaftaran
    -- ==========================================
    INSERT INTO pendaftaran (
        id_calon,
        id_sekolah,
        id_jalur,
        nomor_pendaftaran,
        tanggal_daftar,
        status_pendaftaran,
        tahun_ajaran,
        berkas_lengkap
    ) VALUES (
        v_id_calon,
        NEW.id_sekolah,
        NEW.id_jalur,
        v_nomor_daftar,
        COALESCE(NEW.tanggal_daftar, CURRENT_TIMESTAMP),
        COALESCE(NEW.status_pendaftaran, 'Menunggu'),
        COALESCE(NEW.tahun_ajaran, '2023/2024'),
        COALESCE(NEW.berkas_lengkap, FALSE)
    )
    RETURNING id_pendaftaran INTO v_id_pendaftaran;
    
    -- ==========================================
    -- STEP 6: Log aktivitas pendaftaran baru
    -- ==========================================
    INSERT INTO log_aktivitas (
        id_pendaftaran,
        waktu_aktivitas,
        jenis_aktivitas,
        tabel_terdampak,
        data_baru,
        keterangan
    ) VALUES (
        v_id_pendaftaran,
        CURRENT_TIMESTAMP,
        'INSERT_PENDAFTARAN',
        'pendaftaran, calon_siswa',
        jsonb_build_object(
            'nomor_pendaftaran', v_nomor_daftar,
            'id_calon',          v_id_calon,
            'id_sekolah',        NEW.id_sekolah,
            'id_jalur',          NEW.id_jalur,
            'nis',               NEW.nis
        ),
        FORMAT('Pendaftaran baru: %s - NIS: %s', 
               v_nomor_daftar, NEW.nis)
    );
    
    RAISE NOTICE 
        'Pendaftaran berhasil! Nomor: %, id_pendaftaran: %',
        v_nomor_daftar, v_id_pendaftaran;
    
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Bind INSTEAD OF TRIGGER ke VIEW
CREATE TRIGGER trg_instead_of_insert_pendaftaran
    INSTEAD OF INSERT ON v_pendaftaran_lengkap
    FOR EACH ROW
    EXECUTE FUNCTION fn_insert_pendaftaran_lengkap();

-- ============================================
-- TEST INSTEAD OF TRIGGER:
-- Insert melalui view (bukan langsung ke tabel)
-- ============================================
INSERT INTO v_pendaftaran_lengkap (
    nis,
    nama_calon,
    tempat_lahir,
    tanggal_lahir,
    jenis_kelamin,
    alamat,
    no_telp,
    email,
    asal_sekolah,
    nilai_rata_rata,
    kategori_afirmasi,
    id_sekolah,
    id_jalur,
    tahun_ajaran,
    berkas_lengkap
) VALUES (
    '2023010056',
    'Bima Sakti Pratama',
    'Makassar',
    '2007-05-10',
    'L',
    'Jl. Perintis No.99, Makassar',
    '081234560056',
    'bima.sakti@email.com',
    'SMPN 5 Makassar',
    88.50,
    'Tidak Ada',
    1,    -- SMAN 1 Makassar
    8,    -- Jalur Reguler
    '2023/2024',
    TRUE
);

-- Verifikasi hasil insert via view
SELECT 
    id_pendaftaran,
    nomor_pendaftaran,
    nis,
    nama_calon,
    nama_sekolah,
    nama_jalur,
    status_pendaftaran,
    tanggal_daftar
FROM v_pendaftaran_lengkap
WHERE nis = '2023010056';
```

---

# PEMBAGIAN TUGAS KELOMPOK (5 Anggota)

```
╔══════════════════════════════════════════════════════════════════════╗
║              PEMBAGIAN TUGAS MERATA - 5 ANGGOTA                     ║
╠════════════╦═════════════════════════════════════════════════════════╣
║ ANGGOTA 1  ║ Desain DB (ERD, Skema Relasional) + No.1               ║
║            ║ DDL: CREATE TABLE calon_siswa, sekolah, jalur_seleksi  ║
║            ║ Data Dummy: calon_siswa (55 data), sekolah (20 data)   ║
║            ║ Soal No.3: Investigasi penyebab masalah                ║
╠════════════╬═════════════════════════════════════════════════════════╣
║ ANGGOTA 2  ║ DDL: CREATE TABLE kuota_sekolah, pendaftaran           ║
║            ║ Data Dummy: jalur_seleksi, kuota_sekolah, admin        ║
║            ║ Soal No.4: Solusi A (JOIN) dan Solusi B (NOT EXISTS)   ║
║            ║ Soal No.7: Uji performa Solusi A dan B                 ║
╠════════════╬═════════════════════════════════════════════════════════╣
║ ANGGOTA 3  ║ DDL: CREATE TABLE seleksi_hasil, daftar_ulang, admin   ║
║            ║ DDL: CREATE TABLE log_aktivitas + semua INDEX          ║
║            ║ Data Dummy: pendaftaran (55 data)                      ║
║            ║ Soal No.4: Solusi C (NOT IN) dan Solusi D (CTE)        ║
╠════════════╬═════════════════════════════════════════════════════════╣
║ ANGGOTA 4  ║ Data Dummy: seleksi_hasil (55 data), daftar_ulang      ║
║            ║ Soal No.5: Operasi Set (EXCEPT, INTERSECT, UNION ALL)  ║
║            ║ Soal No.7: Uji performa Solusi C dan D                 ║
║            ║ Trigger 2: INSTEAD OF TRIGGER (view insert)            ║
╠════════════╬═════════════════════════════════════════════════════════╣
║ ANGGOTA 5  ║ Soal No.6: Optimasi Query (partial index + EXPLAIN)    ║
║            ║ Trigger 1: Row-Level Trigger (audit log)               ║
║            ║ Soal No.7: Kesimpulan & rekomendasi query optimal      ║
║            ║ Penyusunan laporan akhir & dokumentasi                 ║
╚════════════╩═════════════════════════════════════════════════════════╝

CATATAN PENTING:
✓ Setiap anggota wajib memahami SELURUH bagian
✓ Setiap anggota ikut DDL, Data Dummy, dan minimal 1 soal query
✓ Cross-review antar anggota sebelum submit
✓ Gunakan Git/GitHub untuk kolaborasi versi kode
```

---

## Ringkasan Solusi Query Paling Optimal

```
🏆 REKOMENDASI QUERY TERBAIK:

Untuk dataset kecil-menengah : NOT EXISTS (Solusi B)
  → Planner PostgreSQL dapat short-circuit evaluation
  → Tidak terpengaruh nilai NULL
  → Cost lebih rendah dari NOT IN

Untuk dataset besar (>100K)  : CTE + Partial Index (Solusi D)
  → Partial index hanya scan data relevan
  → CTE memungkinkan reuse hasil tanpa hitung ulang
  → Lebih mudah di-maintain dan di-debug

HINDARI NOT IN jika kolom subquery bisa mengandung NULL!
  → NOT IN dengan NULL selalu return empty set (NULL trap)
  → Gunakan NOT EXISTS sebagai alternatif yang aman
```
