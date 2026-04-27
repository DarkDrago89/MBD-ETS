# Data Dummy untuk Simulasi Masalah PPDB

## Konsep Kerusakan Data yang Akan Dibuat

```
Jenis Kerusakan yang Disimulasikan:

KERUSAKAN 1: Approved tapi tidak ada penerimaan
  → Verifikasi action='approved' tapi tidak ada 
    record di tabel penerimaan

KERUSAKAN 2: Penerimaan ada tapi status registrasi 
             masih 'submitted' (tidak atomik)
  → Seharusnya status berubah ke 'accepted' 
    saat diterima, tapi tidak terupdate

KERUSAKAN 3: Siswa diterima di lebih dari 1 jalur
  → Satu siswa muncul di penerimaan_afirmasi 
    DAN penerimaan_zonasi sekaligus

KERUSAKAN 4: Penerimaan melebihi pagu sekolah
  → Jumlah siswa diterima > pagu yang ditetapkan

KERUSAKAN 5: Verifikasi rejected tapi ada penerimaan
  → Seharusnya yang rejected tidak boleh ada 
    di tabel penerimaan
```

---

## STEP 1: Reset dan Insert Data Bersih Terlebih Dahulu

```sql
-- ============================================================
-- TRUNCATE semua tabel (urutan: anak dulu, lalu induk)
-- ============================================================
TRUNCATE TABLE 
    audit_log_ppdb,
    penerimaan_zonasi,
    penerimaan_prestasi_mandiri,
    penerimaan_mutasi,
    penerimaan_afirmasi,
    verification_zonasi,
    verification_prestasi_mandiri,
    verification_mutasi,
    verification_afirmasi,
    registrations_zonasi,
    registrations_prestasi_mandiri,
    registrations_mutasi,
    registrations_afirmasi,
    school_users,
    users,
    junior_schools,
    office_users,
    schools,
    roles
RESTART IDENTITY CASCADE;
```

---

## STEP 2: Insert Tabel Master (Bersih)

```sql
-- ============================================================
-- ROLES (60 data)
-- ============================================================
INSERT INTO roles (name) VALUES
('Super Admin'),
('Admin Dinas'),
('Admin Provinsi'),
('Operator Dinas'),
('Operator Sekolah'),
('Kepala Sekolah'),
('Wakil Kepala Sekolah'),
('Verifikator Afirmasi'),
('Verifikator Mutasi'),
('Verifikator Zonasi'),
('Verifikator Prestasi'),
('Panitia PPDB Provinsi'),
('Panitia PPDB Kota'),
('Koordinator Wilayah'),
('Supervisor PPDB'),
('Pengawas Eksternal'),
('Auditor Internal'),
('Auditor Eksternal'),
('Petugas Helpdesk'),
('Petugas Data'),
('Petugas Zonasi'),
('Petugas Afirmasi'),
('Petugas Mutasi'),
('Petugas Prestasi'),
('Petugas Daftar Ulang'),
('Petugas Pengumuman'),
('Admin Wilayah Utara'),
('Admin Wilayah Selatan'),
('Admin Wilayah Timur'),
('Admin Wilayah Barat'),
('Operator Cadangan 1'),
('Operator Cadangan 2'),
('Operator Cadangan 3'),
('Tim Seleksi Akademik'),
('Tim Seleksi Non-Akademik'),
('Tim Validasi Dokumen'),
('Tim Teknis Sistem'),
('Tim Keamanan Data'),
('Pengelola Akun Siswa'),
('Pengelola Kuota Sekolah'),
('Pengelola Jadwal PPDB'),
('Pengelola Dokumen'),
('Pengelola Hasil Seleksi'),
('Reviewer Berkas'),
('Approver Berkas'),
('Monitoring Real-time'),
('Evaluator Program'),
('Administrator Sistem'),
('Operator PPDB Harian'),
('Panitia Zonasi Khusus'),
('Panitia Afirmasi Khusus'),
('Panitia Mutasi Khusus'),
('Panitia Prestasi Khusus'),
('Staf Dinas Pendidikan'),
('Staf Tata Usaha'),
('Koordinator Sekolah'),
('Tim Verifikasi Lapangan'),
('Petugas Arsip'),
('Petugas IT Support'),
('Viewer Only');

-- ============================================================
-- OFFICE_USERS (60 data)
-- ============================================================
INSERT INTO office_users (name, username, password, role_id) VALUES
('Ahmad Fauzan',         'operator001', 'hashed_pass_001', 1),
('Siti Nur Aisyah',      'operator002', 'hashed_pass_002', 2),
('Budi Santoso',         'operator003', 'hashed_pass_003', 3),
('Dewi Lestari',         'operator004', 'hashed_pass_004', 4),
('Rizky Maulana',        'operator005', 'hashed_pass_005', 5),
('Nadia Putri',          'operator006', 'hashed_pass_006', 6),
('Muhammad Arif',        'operator007', 'hashed_pass_007', 7),
('Anisa Rahma',          'operator008', 'hashed_pass_008', 8),
('Fajar Pratama',        'operator009', 'hashed_pass_009', 9),
('Indah Permatasari',    'operator010', 'hashed_pass_010', 10),
('Dimas Saputra',        'operator011', 'hashed_pass_011', 11),
('Aulia Maharani',       'operator012', 'hashed_pass_012', 12),
('Rafi Alfarizi',        'operator013', 'hashed_pass_013', 13),
('Nabila Zahra',         'operator014', 'hashed_pass_014', 14),
('Agus Setiawan',        'operator015', 'hashed_pass_015', 15),
('Fitri Handayani',      'operator016', 'hashed_pass_016', 16),
('Hendra Wijaya',        'operator017', 'hashed_pass_017', 17),
('Laila Safitri',        'operator018', 'hashed_pass_018', 18),
('Yusuf Ramadhan',       'operator019', 'hashed_pass_019', 19),
('Citra Amelia',         'operator020', 'hashed_pass_020', 20),
('Andi Kurniawan',       'operator021', 'hashed_pass_021', 21),
('Maya Salsabila',       'operator022', 'hashed_pass_022', 22),
('Farhan Akbar',         'operator023', 'hashed_pass_023', 23),
('Putri Anggraini',      'operator024', 'hashed_pass_024', 24),
('Reza Febriansyah',     'operator025', 'hashed_pass_025', 25),
('Salsa Nurhaliza',      'operator026', 'hashed_pass_026', 26),
('Bagas Aditya',         'operator027', 'hashed_pass_027', 27),
('Tiara Kusuma',         'operator028', 'hashed_pass_028', 28),
('Ilham Nugraha',        'operator029', 'hashed_pass_029', 29),
('Vina Oktaviani',       'operator030', 'hashed_pass_030', 30),
('Rangga Prakoso',       'operator031', 'hashed_pass_031', 31),
('Niken Ayu',            'operator032', 'hashed_pass_032', 32),
('Bayu Prasetyo',        'operator033', 'hashed_pass_033', 33),
('Mila Kartika',         'operator034', 'hashed_pass_034', 34),
('Gilang Ramadhan',      'operator035', 'hashed_pass_035', 35),
('Dinda Febrianti',      'operator036', 'hashed_pass_036', 36),
('Arman Hakim',          'operator037', 'hashed_pass_037', 37),
('Rara Maharani',        'operator038', 'hashed_pass_038', 38),
('Taufik Hidayat',       'operator039', 'hashed_pass_039', 39),
('Nuraeni Lestari',      'operator040', 'hashed_pass_040', 40),
('Reno Saputra',         'operator041', 'hashed_pass_041', 41),
('Ayu Wulandari',        'operator042', 'hashed_pass_042', 42),
('Hafiz Firdaus',        'operator043', 'hashed_pass_043', 43),
('Sekar Kinanti',        'operator044', 'hashed_pass_044', 44),
('Yoga Pratama',         'operator045', 'hashed_pass_045', 45),
('Melati Anggraeni',     'operator046', 'hashed_pass_046', 46),
('Irvan Maulana',        'operator047', 'hashed_pass_047', 47),
('Nisa Khairunnisa',     'operator048', 'hashed_pass_048', 48),
('Doni Firmansyah',      'operator049', 'hashed_pass_049', 49),
('Elsa Damayanti',       'operator050', 'hashed_pass_050', 50),
('Farel Prayoga',        'operator051', 'hashed_pass_051', 51),
('Ghea Indrawati',       'operator052', 'hashed_pass_052', 52),
('Haris Budiman',        'operator053', 'hashed_pass_053', 53),
('Ika Rahmawati',        'operator054', 'hashed_pass_054', 54),
('Jaka Tarub',           'operator055', 'hashed_pass_055', 55),
('Kiki Amalia',          'operator056', 'hashed_pass_056', 56),
('Luki Prasetyo',        'operator057', 'hashed_pass_057', 57),
('Mira Susanti',         'operator058', 'hashed_pass_058', 58),
('Nando Kusuma',         'operator059', 'hashed_pass_059', 59),
('Okta Sari',            'operator060', 'hashed_pass_060', 60);

-- ============================================================
-- SCHOOLS (60 data)
-- pagu sengaja dibuat KECIL agar mudah melebihi batas
-- ============================================================
INSERT INTO schools (
    name, kode, npsn, minimum_average, city_id,
    latitude, longitude,
    pagu_afirmasi, pagu_mutasi,
    pagu_prestasi_undangan, pagu_zonasi,
    pagu_prestasi_tesmandiri, pagu_tidak_naik_kelas,
    school_code
) VALUES
-- Sekolah 1-20: Makassar (pagu kecil = mudah overflow)
('SMA Negeri 1 Makassar',  'SCH001','40310001',82.00,7371,-5.1420,119.4321, 3, 2, 2, 5, 2, 0,'SULSEL001'),
('SMA Negeri 2 Makassar',  'SCH002','40310002',80.00,7371,-5.1440,119.4341, 3, 2, 2, 5, 2, 0,'SULSEL002'),
('SMA Negeri 3 Makassar',  'SCH003','40310003',78.00,7371,-5.1460,119.4361, 3, 2, 2, 5, 2, 0,'SULSEL003'),
('SMA Negeri 4 Makassar',  'SCH004','40310004',79.00,7371,-5.1480,119.4381, 3, 2, 2, 5, 2, 0,'SULSEL004'),
('SMA Negeri 5 Makassar',  'SCH005','40310005',77.00,7371,-5.1500,119.4401, 3, 2, 2, 5, 2, 0,'SULSEL005'),
('SMA Negeri 6 Makassar',  'SCH006','40310006',76.00,7371,-5.1520,119.4421, 3, 2, 2, 5, 2, 0,'SULSEL006'),
('SMA Negeri 7 Makassar',  'SCH007','40310007',75.00,7371,-5.1540,119.4441, 3, 2, 2, 5, 2, 0,'SULSEL007'),
('SMA Negeri 8 Makassar',  'SCH008','40310008',74.00,7371,-5.1560,119.4461, 3, 2, 2, 5, 2, 0,'SULSEL008'),
('SMA Negeri 9 Makassar',  'SCH009','40310009',73.00,7371,-5.1580,119.4481, 3, 2, 2, 5, 2, 0,'SULSEL009'),
('SMA Negeri 10 Makassar', 'SCH010','40310010',72.00,7371,-5.1600,119.4501, 3, 2, 2, 5, 2, 0,'SULSEL010'),
('SMA Negeri 11 Makassar', 'SCH011','40310011',71.00,7371,-5.1620,119.4521, 3, 2, 2, 5, 2, 0,'SULSEL011'),
('SMA Negeri 12 Makassar', 'SCH012','40310012',70.00,7371,-5.1640,119.4541, 3, 2, 2, 5, 2, 0,'SULSEL012'),
('SMA Negeri 13 Makassar', 'SCH013','40310013',70.00,7371,-5.1660,119.4561, 3, 2, 2, 5, 2, 0,'SULSEL013'),
('SMA Negeri 14 Makassar', 'SCH014','40310014',70.00,7371,-5.1680,119.4581, 3, 2, 2, 5, 2, 0,'SULSEL014'),
('SMA Negeri 15 Makassar', 'SCH015','40310015',70.00,7371,-5.1700,119.4601, 3, 2, 2, 5, 2, 0,'SULSEL015'),
('SMA Negeri 16 Makassar', 'SCH016','40310016',70.00,7371,-5.1720,119.4621, 3, 2, 2, 5, 2, 0,'SULSEL016'),
('SMA Negeri 17 Makassar', 'SCH017','40310017',70.00,7371,-5.1740,119.4641, 3, 2, 2, 5, 2, 0,'SULSEL017'),
('SMA Negeri 18 Makassar', 'SCH018','40310018',70.00,7371,-5.1760,119.4661, 3, 2, 2, 5, 2, 0,'SULSEL018'),
('SMA Negeri 19 Makassar', 'SCH019','40310019',70.00,7371,-5.1780,119.4681, 3, 2, 2, 5, 2, 0,'SULSEL019'),
('SMA Negeri 20 Makassar', 'SCH020','40310020',70.00,7371,-5.1800,119.4701, 3, 2, 2, 5, 2, 0,'SULSEL020'),
-- Sekolah 21-40: SMK Makassar
('SMK Negeri 1 Makassar',  'SCH021','40310021',70.00,7371,-5.1820,119.4721, 3, 2, 2, 5, 2, 0,'SULSEL021'),
('SMK Negeri 2 Makassar',  'SCH022','40310022',70.00,7371,-5.1840,119.4741, 3, 2, 2, 5, 2, 0,'SULSEL022'),
('SMK Negeri 3 Makassar',  'SCH023','40310023',70.00,7371,-5.1860,119.4761, 3, 2, 2, 5, 2, 0,'SULSEL023'),
('SMK Negeri 4 Makassar',  'SCH024','40310024',70.00,7371,-5.1880,119.4781, 3, 2, 2, 5, 2, 0,'SULSEL024'),
('SMK Negeri 5 Makassar',  'SCH025','40310025',70.00,7371,-5.1900,119.4801, 3, 2, 2, 5, 2, 0,'SULSEL025'),
('SMK Negeri 6 Makassar',  'SCH026','40310026',70.00,7371,-5.1920,119.4821, 3, 2, 2, 5, 2, 0,'SULSEL026'),
('SMK Negeri 7 Makassar',  'SCH027','40310027',70.00,7371,-5.1940,119.4841, 3, 2, 2, 5, 2, 0,'SULSEL027'),
('SMK Negeri 8 Makassar',  'SCH028','40310028',70.00,7371,-5.1960,119.4861, 3, 2, 2, 5, 2, 0,'SULSEL028'),
('SMK Negeri 9 Makassar',  'SCH029','40310029',70.00,7371,-5.1980,119.4881, 3, 2, 2, 5, 2, 0,'SULSEL029'),
('SMK Negeri 10 Makassar', 'SCH030','40310030',70.00,7371,-5.2000,119.4901, 3, 2, 2, 5, 2, 0,'SULSEL030'),
-- Sekolah 41-60: Daerah lain
('SMA Negeri 1 Gowa',      'SCH031','40310031',70.00,7301,-5.2020,119.4921, 3, 2, 2, 5, 2, 0,'SULSEL031'),
('SMA Negeri 2 Gowa',      'SCH032','40310032',70.00,7301,-5.2040,119.4941, 3, 2, 2, 5, 2, 0,'SULSEL032'),
('SMA Negeri 1 Maros',     'SCH033','40310033',70.00,7302,-5.2060,119.4961, 3, 2, 2, 5, 2, 0,'SULSEL033'),
('SMA Negeri 2 Maros',     'SCH034','40310034',70.00,7302,-5.2080,119.4981, 3, 2, 2, 5, 2, 0,'SULSEL034'),
('SMA Negeri 1 Takalar',   'SCH035','40310035',70.00,7303,-5.2100,119.5001, 3, 2, 2, 5, 2, 0,'SULSEL035'),
('SMA Negeri 2 Takalar',   'SCH036','40310036',70.00,7303,-5.2120,119.5021, 3, 2, 2, 5, 2, 0,'SULSEL036'),
('SMA Negeri 1 Pangkep',   'SCH037','40310037',70.00,7304,-5.2140,119.5041, 3, 2, 2, 5, 2, 0,'SULSEL037'),
('SMA Negeri 2 Pangkep',   'SCH038','40310038',70.00,7304,-5.2160,119.5061, 3, 2, 2, 5, 2, 0,'SULSEL038'),
('SMA Negeri 1 Parepare',  'SCH039','40310039',70.00,7372,-5.2180,119.5081, 3, 2, 2, 5, 2, 0,'SULSEL039'),
('SMA Negeri 2 Parepare',  'SCH040','40310040',70.00,7372,-5.2200,119.5101, 3, 2, 2, 5, 2, 0,'SULSEL040'),
('SMA Negeri 1 Bone',      'SCH041','40310041',70.00,7311,-5.2220,119.5121, 3, 2, 2, 5, 2, 0,'SULSEL041'),
('SMA Negeri 2 Bone',      'SCH042','40310042',70.00,7311,-5.2240,119.5141, 3, 2, 2, 5, 2, 0,'SULSEL042'),
('SMA Negeri 1 Palopo',    'SCH043','40310043',70.00,7373,-5.2260,119.5161, 3, 2, 2, 5, 2, 0,'SULSEL043'),
('SMA Negeri 2 Palopo',    'SCH044','40310044',70.00,7373,-5.2280,119.5181, 3, 2, 2, 5, 2, 0,'SULSEL044'),
('SMA Negeri 1 Bulukumba', 'SCH045','40310045',70.00,7305,-5.2300,119.5201, 3, 2, 2, 5, 2, 0,'SULSEL045'),
('SMA Negeri 2 Bulukumba', 'SCH046','40310046',70.00,7305,-5.2320,119.5221, 3, 2, 2, 5, 2, 0,'SULSEL046'),
('SMK Negeri 1 Gowa',      'SCH047','40310047',70.00,7301,-5.2340,119.5241, 3, 2, 2, 5, 2, 0,'SULSEL047'),
('SMK Negeri 1 Maros',     'SCH048','40310048',70.00,7302,-5.2360,119.5261, 3, 2, 2, 5, 2, 0,'SULSEL048'),
('SMK Negeri 1 Parepare',  'SCH049','40310049',70.00,7372,-5.2380,119.5281, 3, 2, 2, 5, 2, 0,'SULSEL049'),
('SMK Negeri 1 Bone',      'SCH050','40310050',70.00,7311,-5.2400,119.5301, 3, 2, 2, 5, 2, 0,'SULSEL050'),
('SMA Negeri 1 Sinjai',    'SCH051','40310051',70.00,7306,-5.2420,119.5321, 3, 2, 2, 5, 2, 0,'SULSEL051'),
('SMA Negeri 1 Jeneponto', 'SCH052','40310052',70.00,7307,-5.2440,119.5341, 3, 2, 2, 5, 2, 0,'SULSEL052'),
('SMA Negeri 1 Bantaeng',  'SCH053','40310053',70.00,7308,-5.2460,119.5361, 3, 2, 2, 5, 2, 0,'SULSEL053'),
('SMA Negeri 1 Selayar',   'SCH054','40310054',70.00,7309,-5.2480,119.5381, 3, 2, 2, 5, 2, 0,'SULSEL054'),
('SMA Negeri 1 Wajo',      'SCH055','40310055',70.00,7310,-5.2500,119.5401, 3, 2, 2, 5, 2, 0,'SULSEL055'),
('SMA Negeri 1 Sidrap',    'SCH056','40310056',70.00,7312,-5.2520,119.5421, 3, 2, 2, 5, 2, 0,'SULSEL056'),
('SMA Negeri 1 Pinrang',   'SCH057','40310057',70.00,7313,-5.2540,119.5441, 3, 2, 2, 5, 2, 0,'SULSEL057'),
('SMA Negeri 1 Enrekang',  'SCH058','40310058',70.00,7314,-5.2560,119.5461, 3, 2, 2, 5, 2, 0,'SULSEL058'),
('SMA Negeri 1 Luwu',      'SCH059','40310059',70.00,7315,-5.2580,119.5481, 3, 2, 2, 5, 2, 0,'SULSEL059'),
('SMA Negeri 1 Soppeng',   'SCH060','40310060',70.00,7316,-5.2600,119.5501, 3, 2, 2, 5, 2, 0,'SULSEL060');

-- ============================================================
-- SCHOOL_USERS (60 data)
-- ============================================================
INSERT INTO school_users (office_user_username, school_npsn) VALUES
('operator001','40310001'), ('operator002','40310002'),
('operator003','40310003'), ('operator004','40310004'),
('operator005','40310005'), ('operator006','40310006'),
('operator007','40310007'), ('operator008','40310008'),
('operator009','40310009'), ('operator010','40310010'),
('operator011','40310011'), ('operator012','40310012'),
('operator013','40310013'), ('operator014','40310014'),
('operator015','40310015'), ('operator016','40310016'),
('operator017','40310017'), ('operator018','40310018'),
('operator019','40310019'), ('operator020','40310020'),
('operator021','40310021'), ('operator022','40310022'),
('operator023','40310023'), ('operator024','40310024'),
('operator025','40310025'), ('operator026','40310026'),
('operator027','40310027'), ('operator028','40310028'),
('operator029','40310029'), ('operator030','40310030'),
('operator031','40310031'), ('operator032','40310032'),
('operator033','40310033'), ('operator034','40310034'),
('operator035','40310035'), ('operator036','40310036'),
('operator037','40310037'), ('operator038','40310038'),
('operator039','40310039'), ('operator040','40310040'),
('operator041','40310041'), ('operator042','40310042'),
('operator043','40310043'), ('operator044','40310044'),
('operator045','40310045'), ('operator046','40310046'),
('operator047','40310047'), ('operator048','40310048'),
('operator049','40310049'), ('operator050','40310050'),
('operator051','40310051'), ('operator052','40310052'),
('operator053','40310053'), ('operator054','40310054'),
('operator055','40310055'), ('operator056','40310056'),
('operator057','40310057'), ('operator058','40310058'),
('operator059','40310059'), ('operator060','40310060');

-- ============================================================
-- JUNIOR_SCHOOLS (60 data)
-- ============================================================
INSERT INTO junior_schools (npsn, name) VALUES
('40320001','SMP Negeri 1 Makassar'),
('40320002','SMP Negeri 2 Makassar'),
('40320003','SMP Negeri 3 Makassar'),
('40320004','SMP Negeri 4 Makassar'),
('40320005','SMP Negeri 5 Makassar'),
('40320006','SMP Negeri 6 Makassar'),
('40320007','SMP Negeri 7 Makassar'),
('40320008','SMP Negeri 8 Makassar'),
('40320009','SMP Negeri 9 Makassar'),
('40320010','SMP Negeri 10 Makassar'),
('40320011','SMP Negeri 11 Makassar'),
('40320012','SMP Negeri 12 Makassar'),
('40320013','SMP Negeri 13 Makassar'),
('40320014','SMP Negeri 14 Makassar'),
('40320015','SMP Negeri 15 Makassar'),
('40320016','SMP Negeri 16 Makassar'),
('40320017','SMP Negeri 17 Makassar'),
('40320018','SMP Negeri 18 Makassar'),
('40320019','SMP Negeri 19 Makassar'),
('40320020','SMP Negeri 20 Makassar'),
('40320021','SMP Negeri 1 Gowa'),
('40320022','SMP Negeri 2 Gowa'),
('40320023','SMP Negeri 3 Gowa'),
('40320024','SMP Negeri 1 Maros'),
('40320025','SMP Negeri 2 Maros'),
('40320026','SMP Negeri 1 Takalar'),
('40320027','SMP Negeri 2 Takalar'),
('40320028','SMP Negeri 1 Pangkep'),
('40320029','SMP Negeri 2 Pangkep'),
('40320030','SMP Negeri 1 Parepare'),
('40320031','SMP Negeri 2 Parepare'),
('40320032','SMP Negeri 1 Bone'),
('40320033','SMP Negeri 2 Bone'),
('40320034','SMP Negeri 3 Bone'),
('40320035','SMP Negeri 1 Palopo'),
('40320036','SMP Negeri 2 Palopo'),
('40320037','SMP Negeri 1 Bulukumba'),
('40320038','SMP Negeri 2 Bulukumba'),
('40320039','SMP Negeri 1 Sinjai'),
('40320040','SMP Negeri 1 Jeneponto'),
('40320041','SMP Negeri 1 Bantaeng'),
('40320042','SMP Negeri 1 Selayar'),
('40320043','SMP Negeri 1 Wajo'),
('40320044','SMP Negeri 1 Sidrap'),
('40320045','SMP Negeri 1 Pinrang'),
('40320046','SMP Negeri 1 Enrekang'),
('40320047','SMP Negeri 1 Luwu'),
('40320048','SMP Negeri 1 Soppeng'),
('40320049','SMP Islam Athirah Makassar'),
('40320050','SMP Unismuh Makassar'),
('40320051','SMP Frater Makassar'),
('40320052','SMP Katolik Rajawali Makassar'),
('40320053','SMP Negeri 1 Barru'),
('40320054','SMP Negeri 1 Luwu Timur'),
('40320055','SMP Negeri 1 Luwu Utara'),
('40320056','SMP Negeri 1 Toraja Utara'),
('40320057','SMP Negeri 2 Sinjai'),
('40320058','SMP Negeri 2 Jeneponto'),
('40320059','SMP Negeri 2 Soppeng'),
('40320060','SMP Negeri 2 Wajo');
```

---

## STEP 3: Insert USERS (60 data) — Dengan Kerusakan Terselip

```sql
-- ============================================================
-- USERS (60 data)
-- Kerusakan: registration_*_type dan acceptance_type
-- diisi tapi _id sengaja DIKOSONGKAN atau SALAH
-- ============================================================
INSERT INTO users (
    npsn, nisn, birth_date, name, gender,
    address, phone, school_name, cluster,
    school_destination_id, latitude, longitude,
    registration_1_type, registration_2_type, registration_3_type,
    acceptance_type, acceptance_id,
    jalur_prestasi
) VALUES
-- =======================================
-- User 1-15: NORMAL (tidak ada masalah)
-- =======================================
('40320001','3080000001','2008-01-05','Aisyah Putri Rahmadani','P',
 'Jl. AP Pettarani No.1, Makassar','081200000001',
 'SMP Negeri 1 Makassar','Cluster 1',
 1,-5.1410,119.4310,'zonasi',NULL,NULL,NULL,NULL,NULL),

('40320002','3080000002','2008-02-10','Muhammad Rizky Pratama','L',
 'Jl. Sultan Hasanuddin No.2, Makassar','081200000002',
 'SMP Negeri 2 Makassar','Cluster 1',
 2,-5.1430,119.4330,'afirmasi',NULL,NULL,NULL,NULL,NULL),

('40320003','3080000003','2008-03-15','Nabila Zahra Lestari','P',
 'Jl. Urip Sumoharjo No.3, Makassar','081200000003',
 'SMP Negeri 3 Makassar','Cluster 2',
 3,-5.1450,119.4350,'prestasi_mandiri',NULL,NULL,NULL,NULL,'Akademik'),

('40320004','3080000004','2008-04-20','Ahmad Fadhil Ramadhan','L',
 'Jl. Penghibur No.4, Makassar','081200000004',
 'SMP Negeri 4 Makassar','Cluster 2',
 4,-5.1470,119.4370,'mutasi',NULL,NULL,NULL,NULL,NULL),

('40320005','3080000005','2008-05-25','Dewi Anjani Safitri','P',
 'Jl. Ratulangi No.5, Makassar','081200000005',
 'SMP Negeri 5 Makassar','Cluster 3',
 5,-5.1490,119.4390,'zonasi',NULL,NULL,NULL,NULL,NULL),

('40320006','3080000006','2008-06-30','Rafi Maulana Hakim','L',
 'Jl. Veteran Selatan No.6, Makassar','081200000006',
 'SMP Negeri 6 Makassar','Cluster 3',
 6,-5.1510,119.4410,'afirmasi',NULL,NULL,NULL,NULL,NULL),

('40320007','3080000007','2008-07-05','Salsabila Nur Azizah','P',
 'Jl. Andi Tonro No.7, Makassar','081200000007',
 'SMP Negeri 7 Makassar','Cluster 4',
 7,-5.1530,119.4430,'prestasi_mandiri',NULL,NULL,NULL,NULL,'Non-Akademik'),

('40320008','3080000008','2008-08-10','Farhan Aditya Saputra','L',
 'Jl. Nusantara No.8, Makassar','081200000008',
 'SMP Negeri 8 Makassar','Cluster 4',
 8,-5.1550,119.4450,'mutasi',NULL,NULL,NULL,NULL,NULL),

('40320009','3080000009','2008-09-15','Tiara Kusuma Wardani','P',
 'Jl. Sungai Saddang No.9, Makassar','081200000009',
 'SMP Negeri 9 Makassar','Cluster 5',
 9,-5.1570,119.4470,'zonasi',NULL,NULL,NULL,NULL,NULL),

('40320010','3080000010','2008-10-20','Ilham Dwi Nugroho','L',
 'Jl. Irian No.10, Makassar','081200000010',
 'SMP Negeri 10 Makassar','Cluster 5',
 10,-5.1590,119.4490,'afirmasi',NULL,NULL,NULL,NULL,NULL),

('40320011','3080000011','2008-11-25','Maya Fitri Handayani','P',
 'Jl. Andi Pangeran No.11, Makassar','081200000011',
 'SMP Negeri 11 Makassar','Cluster 1',
 11,-5.1610,119.4510,'prestasi_mandiri',NULL,NULL,NULL,NULL,'Akademik'),

('40320012','3080000012','2008-12-30','Bagas Arya Wicaksono','L',
 'Jl. Bougenville No.12, Makassar','081200000012',
 'SMP Negeri 12 Makassar','Cluster 2',
 12,-5.1630,119.4530,'mutasi',NULL,NULL,NULL,NULL,NULL),

('40320013','3080000013','2008-01-14','Citra Ayu Permatasari','P',
 'Jl. Cendrawasih No.13, Makassar','081200000013',
 'SMP Negeri 13 Makassar','Cluster 2',
 13,-5.1650,119.4550,'zonasi',NULL,NULL,NULL,NULL,NULL),

('40320014','3080000014','2008-02-18','Yusuf Alfarizi Hidayat','L',
 'Jl. Dg. Tata No.14, Makassar','081200000014',
 'SMP Negeri 14 Makassar','Cluster 3',
 14,-5.1670,119.4570,'afirmasi',NULL,NULL,NULL,NULL,NULL),

('40320015','3080000015','2008-03-22','Anisa Khairunnisa','P',
 'Jl. Haji Bau No.15, Makassar','081200000015',
 'SMP Negeri 15 Makassar','Cluster 3',
 15,-5.1690,119.4590,'prestasi_mandiri',NULL,NULL,NULL,NULL,'Non-Akademik'),

-- =======================================
-- User 16-30: KERUSAKAN TIPE 1 & 2
-- Status registrasi akan di-set 'submitted'
-- tapi penerimaan akan tetap dibuat (inkonsisten)
-- =======================================
('40320016','3080000016','2008-04-26','Dimas Tri Prakoso','L',
 'Jl. Kakatua No.16, Makassar','081200000016',
 'SMP Negeri 16 Makassar','Cluster 4',
 1,-5.1710,119.4610,'zonasi',NULL,NULL,NULL,NULL,NULL),

('40320017','3080000017','2008-05-30','Nadia Febrianti','P',
 'Jl. Lompobattang No.17, Makassar','081200000017',
 'SMP Negeri 17 Makassar','Cluster 4',
 2,-5.1730,119.4630,'afirmasi',NULL,NULL,NULL,NULL,NULL),

('40320018','3080000018','2008-06-04','Hafiz Firdaus Santoso','L',
 'Jl. Masjid Raya No.18, Makassar','081200000018',
 'SMP Negeri 18 Makassar','Cluster 5',
 3,-5.1750,119.4650,'prestasi_mandiri',NULL,NULL,NULL,NULL,'Akademik'),

('40320019','3080000019','2008-07-08','Laila Maharani','P',
 'Jl. Nuri No.19, Makassar','081200000019',
 'SMP Negeri 19 Makassar','Cluster 5',
 4,-5.1770,119.4670,'mutasi',NULL,NULL,NULL,NULL,NULL),

('40320020','3080000020','2008-08-12','Rangga Putra Wijaya','L',
 'Jl. Onta No.20, Makassar','081200000020',
 'SMP Negeri 20 Makassar','Cluster 1',
 5,-5.1790,119.4690,'zonasi',NULL,NULL,NULL,NULL,NULL),

('40320021','3080000021','2008-09-16','Aulia Rahmawati','P',
 'Jl. Paccerakkang No.21, Makassar','081200000021',
 'SMP Negeri 1 Gowa','Cluster 1',
 6,-5.1810,119.4710,'afirmasi',NULL,NULL,NULL,NULL,NULL),

('40320022','3080000022','2008-10-20','Bayu Setiawan','L',
 'Jl. Rajawali No.22, Makassar','081200000022',
 'SMP Negeri 2 Gowa','Cluster 2',
 7,-5.1830,119.4730,'prestasi_mandiri',NULL,NULL,NULL,NULL,'Non-Akademik'),

('40320023','3080000023','2008-11-24','Indah Nuraini','P',
 'Jl. Sungai Cerekang No.23, Makassar','081200000023',
 'SMP Negeri 3 Gowa','Cluster 2',
 8,-5.1850,119.4750,'mutasi',NULL,NULL,NULL,NULL,NULL),

('40320024','3080000024','2008-12-28','Fajar Kurniawan','L',
 'Jl. Timor No.24, Makassar','081200000024',
 'SMP Negeri 1 Maros','Cluster 3',
 9,-5.1870,119.4770,'zonasi',NULL,NULL,NULL,NULL,NULL),

('40320025','3080000025','2008-01-01','Putri Amelia Sari','P',
 'Jl. Ujung Pandang No.25, Makassar','081200000025',
 'SMP Negeri 2 Maros','Cluster 3',
 10,-5.1890,119.4790,'afirmasi',NULL,NULL,NULL,NULL,NULL),

('40320026','3080000026','2008-02-05','Reza Firmansyah','L',
 'Jl. Veteran Utara No.26, Makassar','081200000026',
 'SMP Negeri 1 Takalar','Cluster 4',
 11,-5.1910,119.4810,'prestasi_mandiri',NULL,NULL,NULL,NULL,'Akademik'),

('40320027','3080000027','2008-03-10','Sekar Ayu Kinanti','P',
 'Jl. Wahidin No.27, Makassar','081200000027',
 'SMP Negeri 2 Takalar','Cluster 4',
 12,-5.1930,119.4830,'mutasi',NULL,NULL,NULL,NULL,NULL),

('40320028','3080000028','2008-04-14','Gilang Ramadhan','L',
 'Jl. Andi Djemma No.28, Makassar','081200000028',
 'SMP Negeri 1 Pangkep','Cluster 5',
 13,-5.1950,119.4850,'zonasi',NULL,NULL,NULL,NULL,NULL),

('40320029','3080000029','2008-05-18','Mila Kartika Dewi','P',
 'Jl. Baji Gau No.29, Makassar','081200000029',
 'SMP Negeri 2 Pangkep','Cluster 5',
 14,-5.1970,119.4870,'afirmasi',NULL,NULL,NULL,NULL,NULL),

('40320030','3080000030','2008-06-22','Andi Saputra','L',
 'Jl. Cumi-Cumi No.30, Makassar','081200000030',
 'SMP Negeri 1 Parepare','Cluster 1',
 15,-5.1990,119.4890,'prestasi_mandiri',NULL,NULL,NULL,NULL,'Non-Akademik'),

-- =======================================
-- User 31-45: KERUSAKAN TIPE 3
-- Siswa yang akan diterima di LEBIH dari 1 jalur
-- =======================================
('40320031','3080000031','2008-07-26','Rara Oktaviani','P',
 'Jl. Daeng Tata No.31, Makassar','081200000031',
 'SMP Negeri 2 Parepare','Cluster 2',
 1,-5.2010,119.4910,'afirmasi','zonasi',NULL,NULL,NULL,NULL),

('40320032','3080000032','2008-08-30','Taufik Maulana','L',
 'Jl. Emmy Saelan No.32, Makassar','081200000032',
 'SMP Negeri 1 Bone','Cluster 2',
 2,-5.2030,119.4930,'afirmasi','mutasi',NULL,NULL,NULL,NULL),

('40320033','3080000033','2008-09-03','Niken Puspitasari','P',
 'Jl. Garuda No.33, Makassar','081200000033',
 'SMP Negeri 2 Bone','Cluster 3',
 3,-5.2050,119.4950,'zonasi','prestasi_mandiri',NULL,NULL,NULL,'Akademik'),

('40320034','3080000034','2008-10-07','Yoga Pratama','L',
 'Jl. Hati Murni No.34, Makassar','081200000034',
 'SMP Negeri 3 Bone','Cluster 3',
 4,-5.2070,119.4970,'mutasi','zonasi',NULL,NULL,NULL,NULL),

('40320035','3080000035','2008-11-11','Melati Anggraeni','P',
 'Jl. Inspeksi Kanal No.35, Makassar','081200000035',
 'SMP Negeri 1 Palopo','Cluster 4',
 5,-5.2090,119.4990,'afirmasi','zonasi',NULL,NULL,NULL,NULL),

('40320036','3080000036','2008-12-15','Irvan Hidayatullah','L',
 'Jl. Jampea No.36, Makassar','081200000036',
 'SMP Negeri 2 Palopo','Cluster 4',
 6,-5.2110,119.5010,'afirmasi','prestasi_mandiri',NULL,NULL,NULL,'Non-Akademik'),

('40320037','3080000037','2009-01-19','Elsa Damayanti','P',
 'Jl. Kapasa Raya No.37, Makassar','081200000037',
 'SMP Negeri 1 Bulukumba','Cluster 5',
 7,-5.2130,119.5030,'mutasi','afirmasi',NULL,NULL,NULL,NULL),

('40320038','3080000038','2009-02-23','Doni Prasetyo','L',
 'Jl. Landak Baru No.38, Makassar','081200000038',
 'SMP Negeri 2 Bulukumba','Cluster 5',
 8,-5.2150,119.5050,'zonasi','afirmasi',NULL,NULL,NULL,NULL),

('40320039','3080000039','2009-03-27','Nisa Azzahra','P',
 'Jl. Maccini Raya No.39, Makassar','081200000039',
 'SMP Negeri 1 Sinjai','Cluster 1',
 9,-5.2170,119.5070,'prestasi_mandiri','zonasi',NULL,NULL,NULL,'Akademik'),

('40320040','3080000040','2009-04-30','Reno Febriansyah','L',
 'Jl. Naja No.40, Makassar','081200000040',
 'SMP Negeri 1 Jeneponto','Cluster 1',
 10,-5.2190,119.5090,'afirmasi','mutasi',NULL,NULL,NULL,NULL),

('40320041','3080000041','2009-05-04','Ayu Wulandari','P',
 'Jl. Orang-Orang No.41, Makassar','081200000041',
 'SMP Negeri 1 Bantaeng','Cluster 2',
 11,-5.2210,119.5110,'zonasi','prestasi_mandiri',NULL,NULL,NULL,'Non-Akademik'),

('40320042','3080000042','2009-06-08','Arman Hakim','L',
 'Jl. Pampang No.42, Makassar','081200000042',
 'SMP Negeri 1 Selayar','Cluster 2',
 12,-5.2230,119.5130,'mutasi','zonasi',NULL,NULL,NULL,NULL),

('40320043','3080000043','2009-07-12','Dinda Safitri','P',
 'Jl. Rusa No.43, Makassar','081200000043',
 'SMP Negeri 1 Wajo','Cluster 3',
 13,-5.2250,119.5150,'afirmasi','zonasi',NULL,NULL,NULL,NULL),

('40320044','3080000044','2009-08-16','Hendra Wijaya','L',
 'Jl. Sungai Limboto No.44, Makassar','081200000044',
 'SMP Negeri 1 Sidrap','Cluster 3',
 14,-5.2270,119.5170,'afirmasi','prestasi_mandiri',NULL,NULL,NULL,'Akademik'),

('40320045','3080000045','2009-09-20','Vina Oktaviani','P',
 'Jl. Toddopuli No.45, Makassar','081200000045',
 'SMP Negeri 1 Pinrang','Cluster 4',
 15,-5.2290,119.5190,'zonasi','mutasi',NULL,NULL,NULL,NULL),

-- =======================================
-- User 46-60: KERUSAKAN TIPE 4
-- Akan menyebabkan overflow pagu sekolah
-- Semua diarahkan ke sekolah 1 dan 2
-- =======================================
('40320046','3080000046','2009-10-24','Rizal Akbar','L',
 'Jl. Ujung No.46, Makassar','081200000046',
 'SMP Negeri 1 Enrekang','Cluster 4',
 1,-5.2310,119.5210,'zonasi',NULL,NULL,NULL,NULL,NULL),

('40320047','3080000047','2009-11-28','Nuraeni Lestari','P',
 'Jl. Veteran No.47, Makassar','081200000047',
 'SMP Negeri 1 Luwu','Cluster 5',
 1,-5.2330,119.5230,'zonasi',NULL,NULL,NULL,NULL,NULL),

('40320048','3080000048','2009-12-02','Agus Firmansyah','L',
 'Jl. Wahid Hasyim No.48, Makassar','081200000048',
 'SMP Negeri 1 Soppeng','Cluster 5',
 1,-5.2350,119.5250,'zonasi',NULL,NULL,NULL,NULL,NULL),

('40320049','3080000049','2009-01-06','Siti Nurhaliza','P',
 'Jl. Andi Mappanyukki No.49, Makassar','081200000049',
 'SMP Islam Athirah Makassar','Cluster 1',
 2,-5.2370,119.5270,'afirmasi',NULL,NULL,NULL,NULL,NULL),

('40320050','3080000050','2009-02-10','Bima Aditya Nugraha','L',
 'Jl. Bandang No.50, Makassar','081200000050',
 'SMP Unismuh Makassar','Cluster 1',
 2,-5.2390,119.5290,'afirmasi',NULL,NULL,NULL,NULL,NULL),

('40320051','3080000051','2009-03-14','Nurul Fadilah','P',
 'Jl. Cakalang No.51, Makassar','081200000051',
 'SMP Frater Makassar','Cluster 2',
 1,-5.2410,119.5310,'zonasi',NULL,NULL,NULL,NULL,NULL),

('40320052','3080000052','2009-04-18','Fahri Alamsyah','L',
 'Jl. Daeng Ngeppe No.52, Makassar','081200000052',
 'SMP Katolik Rajawali Makassar','Cluster 2',
 1,-5.2430,119.5330,'zonasi',NULL,NULL,NULL,NULL,NULL),

('40320053','3080000053','2009-05-22','Aldi Saputra','L',
 'Jl. Erna No.53, Makassar','081200000053',
 'SMP Negeri 1 Barru','Cluster 3',
 2,-5.2450,119.5350,'afirmasi',NULL,NULL,NULL,NULL,NULL),

('40320054','3080000054','2009-06-26','Zahra Amalia','P',
 'Jl. Flamboyan No.54, Makassar','081200000054',
 'SMP Negeri 1 Luwu Timur','Cluster 3',
 2,-5.2470,119.5370,'afirmasi',NULL,NULL,NULL,NULL,NULL),

('40320055','3080000055','2009-07-30','Muhammad Iqbal','L',
 'Jl. Gunung Latimojong No.55, Makassar','081200000055',
 'SMP Negeri 1 Luwu Utara','Cluster 4',
 1,-5.2490,119.5390,'zonasi',NULL,NULL,NULL,NULL,NULL),

('40320056','3080000056','2009-08-03','Syifa Aulia Putri','P',
 'Jl. Hati No.56, Makassar','081200000056',
 'SMP Negeri 1 Toraja Utara','Cluster 4',
 1,-5.2510,119.5410,'zonasi',NULL,NULL,NULL,NULL,NULL),

('40320057','3080000057','2009-09-07','Andi Muhammad Faris','L',
 'Jl. Imam Bonjol No.57, Makassar','081200000057',
 'SMP Negeri 2 Sinjai','Cluster 5',
 2,-5.2530,119.5430,'afirmasi',NULL,NULL,NULL,NULL,NULL),

('40320058','3080000058','2009-10-11','Nur Aini Rahma','P',
 'Jl. Jend. Sudirman No.58, Makassar','081200000058',
 'SMP Negeri 2 Jeneponto','Cluster 5',
 2,-5.2550,119.5450,'afirmasi',NULL,NULL,NULL,NULL,NULL),

('40320059','3080000059','2009-11-15','Fikri Ramadhan','L',
 'Jl. Kumala No.59, Makassar','081200000059',
 'SMP Negeri 2 Soppeng','Cluster 1',
 1,-5.2570,119.5470,'zonasi',NULL,NULL,NULL,NULL,NULL),

('40320060','3080000060','2009-12-19','Wulan Sari Lestari','P',
 'Jl. Lamuru No.60, Makassar','081200000060',
 'SMP Negeri 2 Wajo','Cluster 1',
 2,-5.2590,119.5490,'afirmasi',NULL,NULL,NULL,NULL,NULL);
```

---

## STEP 4: Insert REGISTRATIONS — Semua status 'submitted' (Sengaja Rusak)

```sql
-- ============================================================
-- REGISTRATIONS_AFIRMASI (60 data)
-- KERUSAKAN: status semua 'submitted', tidak pernah 'accepted'
-- Ini akan menyebabkan inkonsistensi saat penerimaan dibuat
-- ============================================================
INSERT INTO registrations_afirmasi (
    user_id, jenis, school_destination_id,
    distance, verification_schedule, status, documents
) VALUES
-- User 1-15 (normal flow, tapi status tetap 'submitted' = rusak)
(1,  'KIP/PIP',  1, 1.20,'2023-06-19 08:00:00','submitted','dok_afirmasi_01.pdf'),
(2,  'KIP/PIP',  2, 2.10,'2023-06-19 08:10:00','submitted','dok_afirmasi_02.pdf'),
(3,  'KIP/PIP',  3, 1.80,'2023-06-19 08:20:00','submitted','dok_afirmasi_03.pdf'),
(4,  'KIP/PIP',  4, 3.40,'2023-06-19 08:30:00','submitted','dok_afirmasi_04.pdf'),
(5,  'KIP/PIP',  5, 2.60,'2023-06-19 08:40:00','submitted','dok_afirmasi_05.pdf'),
(6,  'KIP/PIP',  6, 1.50,'2023-06-19 08:50:00','submitted','dok_afirmasi_06.pdf'),
(7,  'KIP/PIP',  7, 4.20,'2023-06-19 09:00:00','submitted','dok_afirmasi_07.pdf'),
(8,  'KIP/PIP',  8, 3.10,'2023-06-19 09:10:00','submitted','dok_afirmasi_08.pdf'),
(9,  'KIP/PIP',  9, 2.90,'2023-06-19 09:20:00','submitted','dok_afirmasi_09.pdf'),
(10, 'KIP/PIP', 10, 1.70,'2023-06-19 09:30:00','submitted','dok_afirmasi_10.pdf'),
(11, 'KIP/PIP', 11, 2.30,'2023-06-19 09:40:00','submitted','dok_afirmasi_11.pdf'),
(12, 'KIP/PIP', 12, 4.50,'2023-06-19 09:50:00','submitted','dok_afirmasi_12.pdf'),
(13, 'KIP/PIP', 13, 3.80,'2023-06-19 10:00:00','submitted','dok_afirmasi_13.pdf'),
(14, 'KIP/PIP', 14, 1.40,'2023-06-19 10:10:00','submitted','dok_afirmasi_14.pdf'),
(15, 'KIP/PIP', 15, 2.70,'2023-06-19 10:20:00','submitted','dok_afirmasi_15.pdf'),
-- User 16-30 (inkonsistensi: akan approved tapi penerimaan hilang)
(16, 'KIP/PIP',  1, 1.10,'2023-06-19 10:30:00','submitted','dok_afirmasi_16.pdf'),
(17, 'KIP/PIP',  2, 2.20,'2023-06-19 10:40:00','submitted','dok_afirmasi_17.pdf'),
(18, 'KIP/PIP',  3, 3.30,'2023-06-19 10:50:00','submitted','dok_afirmasi_18.pdf'),
(19, 'KIP/PIP',  4, 1.60,'2023-06-19 11:00:00','submitted','dok_afirmasi_19.pdf'),
(20, 'KIP/PIP',  5, 2.40,'2023-06-19 11:10:00','submitted','dok_afirmasi_20.pdf'),
(21, 'KIP/PIP',  6, 3.70,'2023-06-19 11:20:00','submitted','dok_afirmasi_21.pdf'),
(22, 'KIP/PIP',  7, 1.30,'2023-06-19 11:30:00','submitted','dok_afirmasi_22.pdf'),
(23, 'KIP/PIP',  8, 4.10,'2023-06-19 11:40:00','submitted','dok_afirmasi_23.pdf'),
(24, 'KIP/PIP',  9, 2.80,'2023-06-19 11:50:00','submitted','dok_afirmasi_24.pdf'),
(25, 'KIP/PIP', 10, 1.90,'2023-06-19 12:00:00','submitted','dok_afirmasi_25.pdf'),
(26, 'KIP/PIP', 11, 3.20,'2023-06-19 12:10:00','submitted','dok_afirmasi_26.pdf'),
(27, 'KIP/PIP', 12, 2.50,'2023-06-19 12:20:00','submitted','dok_afirmasi_27.pdf'),
(28, 'KIP/PIP', 13, 4.30,'2023-06-19 12:30:00','submitted','dok_afirmasi_28.pdf'),
(29, 'KIP/PIP', 14, 1.80,'2023-06-19 12:40:00','submitted','dok_afirmasi_29.pdf'),
(30, 'KIP/PIP', 15, 3.60,'2023-06-19 12:50:00','submitted','dok_afirmasi_30.pdf'),
-- User 31-45 (daftar di afirmasi DAN jalur lain = multi jalur)
(31, 'KIP/PIP',  1, 1.50,'2023-06-19 13:00:00','submitted','dok_afirmasi_31.pdf'),
(32, 'KIP/PIP',  2, 2.70,'2023-06-19 13:10:00','submitted','dok_afirmasi_32.pdf'),
(33, 'Disabilitas',3,1.20,'2023-06-19 13:20:00','submitted','dok_afirmasi_33.pdf'),
(34, 'KIP/PIP',  4, 3.10,'2023-06-19 13:30:00','submitted','dok_afirmasi_34.pdf'),
(35, 'KIP/PIP',  5, 2.30,'2023-06-19 13:40:00','submitted','dok_afirmasi_35.pdf'),
(36, 'KIP/PIP',  6, 4.40,'2023-06-19 13:50:00','submitted','dok_afirmasi_36.pdf'),
(37, 'KIP/PIP',  7, 1.70,'2023-06-19 14:00:00','submitted','dok_afirmasi_37.pdf'),
(38, 'Disabilitas',8,3.90,'2023-06-19 14:10:00','submitted','dok_afirmasi_38.pdf'),
(39, 'KIP/PIP',  9, 2.10,'2023-06-19 14:20:00','submitted','dok_afirmasi_39.pdf'),
(40, 'KIP/PIP', 10, 1.40,'2023-06-19 14:30:00','submitted','dok_afirmasi_40.pdf'),
(41, 'Tidak Mampu',11,3.60,'2023-06-19 14:40:00','submitted','dok_afirmasi_41.pdf'),
(42, 'KIP/PIP', 12, 2.80,'2023-06-19 14:50:00','submitted','dok_afirmasi_42.pdf'),
(43, 'KIP/PIP', 13, 4.20,'2023-06-19 15:00:00','submitted','dok_afirmasi_43.pdf'),
(44, 'KIP/PIP', 14, 1.60,'2023-06-19 15:10:00','submitted','dok_afirmasi_44.pdf'),
(45, 'Tidak Mampu',15,3.30,'2023-06-19 15:20:00','submitted','dok_afirmasi_45.pdf'),
-- User 46-60 (overflow pagu sekolah 1 dan 2)
(46, 'KIP/PIP',  1, 2.10,'2023-06-19 15:30:00','submitted','dok_afirmasi_46.pdf'),
(47, 'KIP/PIP',  1, 1.80,'2023-06-19 15:40:00','submitted','dok_afirmasi_47.pdf'),
(48, 'KIP/PIP',  1, 3.20,'2023-06-19 15:50:00','submitted','dok_afirmasi_48.pdf'),
(49, 'KIP/PIP',  2, 2.50,'2023-06-19 16:00:00','submitted','dok_afirmasi_49.pdf'),
(50, 'KIP/PIP',  2, 1.40,'2023-06-19 16:10:00','submitted','dok_afirmasi_50.pdf'),
(51, 'KIP/PIP',  1, 4.10,'2023-06-19 16:20:00','submitted','dok_afirmasi_51.pdf'),
(52, 'KIP/PIP',  1, 2.90,'2023-06-19 16:30:00','submitted','dok_afirmasi_52.pdf'),
(53, 'KIP/PIP',  2, 3.70,'2023-06-19 16:40:00','submitted','dok_afirmasi_53.pdf'),
(54, 'KIP/PIP',  2, 1.20,'2023-06-19 16:50:00','submitted','dok_afirmasi_54.pdf'),
(55, 'KIP/PIP',  1, 2.60,'2023-06-19 17:00:00','submitted','dok_afirmasi_55.pdf'),
(56, 'KIP/PIP',  1, 3.40,'2023-06-19 17:10:00','submitted','dok_afirmasi_56.pdf'),
(57, 'KIP/PIP',  2, 1.90,'2023-06-19 17:20:00','submitted','dok_afirmasi_57.pdf'),
(58, 'KIP/PIP',  2, 4.30,'2023-06-19 17:30:00','submitted','dok_afirmasi_58.pdf'),
(59, 'KIP/PIP',  1, 2.10,'2023-06-19 17:40:00','submitted','dok_afirmasi_59.pdf'),
(60, 'KIP/PIP',  2, 3.50,'2023-06-19 17:50:00','submitted','dok_afirmasi_60.pdf');

-- ============================================================
-- REGISTRATIONS_MUTASI (60 data)
-- ============================================================
INSERT INTO registrations_mutasi (
    user_id, jenis, school_destination_id,
    distance, verification_schedule, status, documents
) VALUES
(1, 'Perpindahan Tugas Orang Tua', 1,1.20,'2023-06-19 08:05:00','submitted','dok_mutasi_01.pdf'),
(2, 'Perpindahan Tugas Orang Tua', 2,2.10,'2023-06-19 08:15:00','submitted','dok_mutasi_02.pdf'),
(3, 'Perpindahan Tugas Orang Tua', 3,1.80,'2023-06-19 08:25:00','submitted','dok_mutasi_03.pdf'),
(4, 'Perpindahan Tugas Orang Tua', 4,3.40,'2023-06-19 08:35:00','submitted','dok_mutasi_04.pdf'),
(5, 'Perpindahan Tugas Orang Tua', 5,2.60,'2023-06-19 08:45:00','submitted','dok_mutasi_05.pdf'),
(6, 'Perpindahan Tugas Orang Tua', 6,1.50,'2023-06-19 08:55:00','submitted','dok_mutasi_06.pdf'),
(7, 'Perpindahan Tugas Orang Tua', 7,4.20,'2023-06-19 09:05:00','submitted','dok_mutasi_07.pdf'),
(8, 'Perpindahan Tugas Orang Tua', 8,3.10,'2023-06-19 09:15:00','submitted','dok_mutasi_08.pdf'),
(9, 'Perpindahan Tugas Orang Tua', 9,2.90,'2023-06-19 09:25:00','submitted','dok_mutasi_09.pdf'),
(10,'Perpindahan Tugas Orang Tua',10,1.70,'2023-06-19 09:35:00','submitted','dok_mutasi_10.pdf'),
(11,'Perpindahan Tugas Orang Tua',11,2.30,'2023-06-19 09:45:00','submitted','dok_mutasi_11.pdf'),
(12,'Perpindahan Tugas Orang Tua',12,4.50,'2023-06-19 09:55:00','submitted','dok_mutasi_12.pdf'),
(13,'Perpindahan Tugas Orang Tua',13,3.80,'2023-06-19 10:05:00','submitted','dok_mutasi_13.pdf'),
(14,'Perpindahan Tugas Orang Tua',14,1.40,'2023-06-19 10:15:00','submitted','dok_mutasi_14.pdf'),
(15,'Perpindahan Tugas Orang Tua',15,2.70,'2023-06-19 10:25:00','submitted','dok_mutasi_15.pdf'),
(16,'Perpindahan Tugas Orang Tua', 1,1.10,'2023-06-19 10:35:00','submitted','dok_mutasi_16.pdf'),
(17,'Perpindahan Tugas Orang Tua', 2,2.20,'2023-06-19 10:45:00','submitted','dok_mutasi_17.pdf'),
(18,'Perpindahan Tugas Orang Tua', 3,3.30,'2023-06-19 10:55:00','submitted','dok_mutasi_18.pdf'),
(19,'Perpindahan Tugas Orang Tua', 4,1.60,'2023-06-19 11:05:00','submitted','dok_mutasi_19.pdf'),
(20,'Perpindahan Tugas Orang Tua', 5,2.40,'2023-06-19 11:15:00','submitted','dok_mutasi_20.pdf'),
(21,'Perpindahan Tugas Orang Tua', 6,3.70,'2023-06-19 11:25:00','submitted','dok_mutasi_21.pdf'),
(22,'Perpindahan Tugas Orang Tua', 7,1.30,'2023-06-19 11:35:00','submitted','dok_mutasi_22.pdf'),
(23,'Perpindahan Tugas Orang Tua', 8,4.10,'2023-06-19 11:45:00','submitted','dok_mutasi_23.pdf'),
(24,'Perpindahan Tugas Orang Tua', 9,2.80,'2023-06-19 11:55:00','submitted','dok_mutasi_24.pdf'),
(25,'Perpindahan Tugas Orang Tua',10,1.90,'2023-06-19 12:05:00','submitted','dok_mutasi_25.pdf'),
(26,'Perpindahan Tugas Orang Tua',11,3.20,'2023-06-19 12:15:00','submitted','dok_mutasi_26.pdf'),
(27,'Perpindahan Tugas Orang Tua',12,2.50,'2023-06-19 12:25:00','submitted','dok_mutasi_27.pdf'),
(28,'Perpindahan Tugas Orang Tua',13,4.30,'2023-06-19 12:35:00','submitted','dok_mutasi_28.pdf'),
(29,'Perpindahan Tugas Orang Tua',14,1.80,'2023-06-19 12:45:00','submitted','dok_mutasi_29.pdf'),
(30,'Perpindahan Tugas Orang Tua',15,3.60,'2023-06-19 12:55:00','submitted','dok_mutasi_30.pdf'),
(31,'Perpindahan Tugas Orang Tua', 1,1.50,'2023-06-19 13:05:00','submitted','dok_mutasi_31.pdf'),
(32,'Perpindahan Tugas Orang Tua', 2,2.70,'2023-06-19 13:15:00','submitted','dok_mutasi_32.pdf'),
(33,'Perpindahan Tugas Orang Tua', 3,1.20,'2023-06-19 13:25:00','submitted','dok_mutasi_33.pdf'),
(34,'Perpindahan Tugas Orang Tua', 4,3.10,'2023-06-19 13:35:00','submitted','dok_mutasi_34.pdf'),
(35,'Perpindahan Tugas Orang Tua', 5,2.30,'2023-06-19 13:45:00','submitted','dok_mutasi_35.pdf'),
(36,'Perpindahan Tugas Orang Tua', 6,4.40,'2023-06-19 13:55:00','submitted','dok_mutasi_36.pdf'),
(37,'Perpindahan Tugas Orang Tua', 7,1.70,'2023-06-19 14:05:00','submitted','dok_mutasi_37.pdf'),
(38,'Perpindahan Tugas Orang Tua', 8,3.90,'2023-06-19 14:15:00','submitted','dok_mutasi_38.pdf'),
(39,'Perpindahan Tugas Orang Tua', 9,2.10,'2023-06-19 14:25:00','submitted','dok_mutasi_39.pdf'),
(40,'Perpindahan Tugas Orang Tua',10,1.40,'2023-06-19 14:35:00','submitted','dok_mutasi_40.pdf'),
(41,'Perpindahan Tugas Orang Tua',11,3.60,'2023-06-19 14:45:00','submitted','dok_mutasi_41.pdf'),
(42,'Perpindahan Tugas Orang Tua',12,2.80,'2023-06-19 14:55:00','submitted','dok_mutasi_42.pdf'),
(43,'Perpindahan Tugas Orang Tua',13,4.20,'2023-06-19 15:05:00','submitted','dok_mutasi_43.pdf'),
(44,'Perpindahan Tugas Orang Tua',14,1.60,'2023-06-19 15:15:00','submitted','dok_mutasi_44.pdf'),
(45,'Perpindahan Tugas Orang Tua',15,3.30,'2023-06-19 15:25:00','submitted','dok_mutasi_45.pdf'),
(46,'Perpindahan Tugas Orang Tua', 1,2.10,'2023-06-19 15:35:00','submitted','dok_mutasi_46.pdf'),
(47,'Perpindahan Tugas Orang Tua', 2,1.80,'2023-06-19 15:45:00','submitted','dok_mutasi_47.pdf'),
(48,'Perpindahan Tugas Orang Tua', 3,3.20,'2023-06-19 15:55:00','submitted','dok_mutasi_48.pdf'),
(49,'Perpindahan Tugas Orang Tua', 4,2.50,'2023-06-19 16:05:00','submitted','dok_mutasi_49.pdf'),
(50,'Perpindahan Tugas Orang Tua', 5,1.40,'2023-06-19 16:15:00','submitted','dok_mutasi_50.pdf'),
(51,'Perpindahan Tugas Orang Tua', 6,4.10,'2023-06-19 16:25:00','submitted','dok_mutasi_51.pdf'),
(52,'Perpindahan Tugas Orang Tua', 7,2.90,'2023-06-19 16:35:00','submitted','dok_mutasi_52.pdf'),
(53,'Perpindahan Tugas Orang Tua', 8,3.70,'2023-06-19 16:45:00','submitted','dok_mutasi_53.pdf'),
(54,'Perpindahan Tugas Orang Tua', 9,1.20,'2023-06-19 16:55:00','submitted','dok_mutasi_54.pdf'),
(55,'Perpindahan Tugas Orang Tua',10,2.60,'2023-06-19 17:05:00','submitted','dok_mutasi_55.pdf'),
(56,'Perpindahan Tugas Orang Tua',11,3.40,'2023-06-19 17:15:00','submitted','dok_mutasi_56.pdf'),
(57,'Perpindahan Tugas Orang Tua',12,1.90,'2023-06-19 17:25:00','submitted','dok_mutasi_57.pdf'),
(58,'Perpindahan Tugas Orang Tua',13,4.30,'2023-06-19 17:35:00','submitted','dok_mutasi_58.pdf'),
(59,'Perpindahan Tugas Orang Tua',14,2.10,'2023-06-19 17:45:00','submitted','dok_mutasi_59.pdf'),
(60,'Perpindahan Tugas Orang Tua',15,3.50,'2023-06-19 17:55:00','submitted','dok_mutasi_60.pdf');

-- ============================================================
-- REGISTRATIONS_PRESTASI_MANDIRI (60 data)
-- ============================================================
INSERT INTO registrations_prestasi_mandiri (
    user_id, data, school_destination_id, distance,
    verification_schedule, status, documents,
    test_number, final_score, test_score, jurusan, ruang_tes
) VALUES
(1, 'Prestasi siswa 1', 1,1.20,'2023-06-19 08:08:00','submitted','dok_prestasi_01.pdf','PM0001',85.50,82.00,'IPA','Ruang 1'),
(2, 'Prestasi siswa 2', 2,2.10,'2023-06-19 08:18:00','submitted','dok_prestasi_02.pdf','PM0002',87.00,84.50,'IPS','Ruang 2'),
(3, 'Prestasi siswa 3', 3,1.80,'2023-06-19 08:28:00','submitted','dok_prestasi_03.pdf','PM0003',90.25,88.00,'IPA','Ruang 3'),
(4, 'Prestasi siswa 4', 4,3.40,'2023-06-19 08:38:00','submitted','dok_prestasi_04.pdf','PM0004',78.50,76.00,'IPS','Ruang 4'),
(5, 'Prestasi siswa 5', 5,2.60,'2023-06-19 08:48:00','submitted','dok_prestasi_05.pdf','PM0005',88.75,86.50,'IPA','Ruang 5'),
(6, 'Prestasi siswa 6', 6,1.50,'2023-06-19 08:58:00','submitted','dok_prestasi_06.pdf','PM0006',82.00,79.50,'IPS','Ruang 6'),
(7, 'Prestasi siswa 7', 7,4.20,'2023-06-19 09:08:00','submitted','dok_prestasi_07.pdf','PM0007',91.50,89.00,'IPA','Ruang 7'),
(8, 'Prestasi siswa 8', 8,3.10,'2023-06-19 09:18:00','submitted','dok_prestasi_08.pdf','PM0008',75.25,73.00,'IPS','Ruang 8'),
(9, 'Prestasi siswa 9', 9,2.90,'2023-06-19 09:28:00','submitted','dok_prestasi_09.pdf','PM0009',86.00,83.50,'IPA','Ruang 9'),
(10,'Prestasi siswa 10',10,1.70,'2023-06-19 09:38:00','submitted','dok_prestasi_10.pdf','PM0010',79.75,77.50,'IPS','Ruang 10'),
(11,'Prestasi siswa 11',11,2.30,'2023-06-19 09:48:00','submitted','dok_prestasi_11.pdf','PM0011',93.00,90.50,'IPA','Ruang 1'),
(12,'Prestasi siswa 12',12,4.50,'2023-06-19 09:58:00','submitted','dok_prestasi_12.pdf','PM0012',84.50,82.00,'IPS','Ruang 2'),
(13,'Prestasi siswa 13',13,3.80,'2023-06-19 10:08:00','submitted','dok_prestasi_13.pdf','PM0013',88.25,85.75,'IPA','Ruang 3'),
(14,'Prestasi siswa 14',14,1.40,'2023-06-19 10:18:00','submitted','dok_prestasi_14.pdf','PM0014',76.50,74.25,'IPS','Ruang 4'),
(15,'Prestasi siswa 15',15,2.70,'2023-06-19 10:28:00','submitted','dok_prestasi_15.pdf','PM0015',89.75,87.25,'IPA','Ruang 5'),
(16,'Prestasi siswa 16', 1,1.10,'2023-06-19 10:38:00','submitted','dok_prestasi_16.pdf','PM0016',83.00,80.50,'IPS','Ruang 6'),
(17,'Prestasi siswa 17', 2,2.20,'2023-06-19 10:48:00','submitted','dok_prestasi_17.pdf','PM0017',92.25,89.75,'IPA','Ruang 7'),
(18,'Prestasi siswa 18', 3,3.30,'2023-06-19 10:58:00','submitted','dok_prestasi_18.pdf','PM0018',77.50,75.25,'IPS','Ruang 8'),
(19,'Prestasi siswa 19', 4,1.60,'2023-06-19 11:08:00','submitted','dok_prestasi_19.pdf','PM0019',87.75,85.25,'IPA','Ruang 9'),
(20,'Prestasi siswa 20', 5,2.40,'2023-06-19 11:18:00','submitted','dok_prestasi_20.pdf','PM0020',81.25,78.75,'IPS','Ruang 10'),
(21,'Prestasi siswa 21', 6,3.70,'2023-06-19 11:28:00','submitted','dok_prestasi_21.pdf','PM0021',90.50,88.00,'IPA','Ruang 1'),
(22,'Prestasi siswa 22', 7,1.30,'2023-06-19 11:38:00','submitted','dok_prestasi_22.pdf','PM0022',85.75,83.25,'IPS','Ruang 2'),
(23,'Prestasi siswa 23', 8,4.10,'2023-06-19 11:48:00','submitted','dok_prestasi_23.pdf','PM0023',78.00,75.50,'IPA','Ruang 3'),
(24,'Prestasi siswa 24', 9,2.80,'2023-06-19 11:58:00','submitted','dok_prestasi_24.pdf','PM0024',94.25,91.75,'IPS','Ruang 4'),
(25,'Prestasi siswa 25',10,1.90,'2023-06-19 12:08:00','submitted','dok_prestasi_25.pdf','PM0025',82.50,80.00,'IPA','Ruang 5'),
(26,'Prestasi siswa 26',11,3.20,'2023-06-19 12:18:00','submitted','dok_prestasi_26.pdf','PM0026',88.00,85.50,'IPS','Ruang 6'),
(27,'Prestasi siswa 27',12,2.50,'2023-06-19 12:28:00','submitted','dok_prestasi_27.pdf','PM0027',76.25,73.75,'IPA','Ruang 7'),
(28,'Prestasi siswa 28',13,4.30,'2023-06-19 12:38:00','submitted','dok_prestasi_28.pdf','PM0028',91.75,89.25,'IPS','Ruang 8'),
(29,'Prestasi siswa 29',14,1.80,'2023-06-19 12:48:00','submitted','dok_prestasi_29.pdf','PM0029',84.00,81.50,'IPA','Ruang 9'),
(30,'Prestasi siswa 30',15,3.60,'2023-06-19 12:58:00','submitted','dok_prestasi_30.pdf','PM0030',79.50,77.00,'IPS','Ruang 10'),
(31,'Prestasi siswa 31', 1,1.50,'2023-06-19 13:08:00','submitted','dok_prestasi_31.pdf','PM0031',86.75,84.25,'IPA','Ruang 1'),
(32,'Prestasi siswa 32', 2,2.70,'2023-06-19 13:18:00','submitted','dok_prestasi_32.pdf','PM0032',92.00,89.50,'IPS','Ruang 2'),
(33,'Prestasi siswa 33', 3,1.20,'2023-06-19 13:28:00','submitted','dok_prestasi_33.pdf','PM0033',80.25,77.75,'IPA','Ruang 3'),
(34,'Prestasi siswa 34', 4,3.10,'2023-06-19 13:38:00','submitted','dok_prestasi_34.pdf','PM0034',87.50,85.00,'IPS','Ruang 4'),
(35,'Prestasi siswa 35', 5,2.30,'2023-06-19 13:48:00','submitted','dok_prestasi_35.pdf','PM0035',83.75,81.25,'IPA','Ruang 5'),
(36,'Prestasi siswa 36', 6,4.40,'2023-06-19 13:58:00','submitted','dok_prestasi_36.pdf','PM0036',89.00,86.50,'IPS','Ruang 6'),
(37,'Prestasi siswa 37', 7,1.70,'2023-06-19 14:08:00','submitted','dok_prestasi_37.pdf','PM0037',75.50,73.00,'IPA','Ruang 7'),
(38,'Prestasi siswa 38', 8,3.90,'2023-06-19 14:18:00','submitted','dok_prestasi_38.pdf','PM0038',93.25,90.75,'IPS','Ruang 8'),
(39,'Prestasi siswa 39', 9,2.10,'2023-06-19 14:28:00','submitted','dok_prestasi_39.pdf','PM0039',81.50,79.00,'IPA','Ruang 9'),
(40,'Prestasi siswa 40',10,1.40,'2023-06-19 14:38:00','submitted','dok_prestasi_40.pdf','PM0040',88.75,86.25,'IPS','Ruang 10'),
(41,'Prestasi siswa 41',11,3.60,'2023-06-19 14:48:00','submitted','dok_prestasi_41.pdf','PM0041',77.25,74.75,'IPA','Ruang 1'),
(42,'Prestasi siswa 42',12,2.80,'2023-06-19 14:58:00','submitted','dok_prestasi_42.pdf','PM0042',90.00,87.50,'IPS','Ruang 2'),
(43,'Prestasi siswa 43',13,4.20,'2023-06-19 15:08:00','submitted','dok_prestasi_43.pdf','PM0043',84.25,81.75,'IPA','Ruang 3'),
(44,'Prestasi siswa 44',14,1.60,'2023-06-19 15:18:00','submitted','dok_prestasi_44.pdf','PM0044',91.00,88.50,'IPS','Ruang 4'),
(45,'Prestasi siswa 45',15,3.30,'2023-06-19 15:28:00','submitted','dok_prestasi_45.pdf','PM0045',78.75,76.25,'IPA','Ruang 5'),
(46,'Prestasi siswa 46', 1,2.10,'2023-06-19 15:38:00','submitted','dok_prestasi_46.pdf','PM0046',85.00,82.50,'IPS','Ruang 6'),
(47,'Prestasi siswa 47', 2,1.80,'2023-06-19 15:48:00','submitted','dok_prestasi_47.pdf','PM0047',87.25,84.75,'IPA','Ruang 7'),
(48,'Prestasi siswa 48', 3,3.20,'2023-06-19 15:58:00','submitted','dok_prestasi_48.pdf','PM0048',82.75,80.25,'IPS','Ruang 8'),
(49,'Prestasi siswa 49', 4,2.50,'2023-06-19 16:08:00','submitted','dok_prestasi_49.pdf','PM0049',89.50,87.00,'IPA','Ruang 9'),
(50,'Prestasi siswa 50', 5,1.40,'2023-06-19 16:18:00','submitted','dok_prestasi_50.pdf','PM0050',76.75,74.25,'IPS','Ruang 10'),
(51,'Prestasi siswa 51', 6,4.10,'2023-06-19 16:28:00','submitted','dok_prestasi_51.pdf','PM0051',93.50,91.00,'IPA','Ruang 1'),
(52,'Prestasi siswa 52', 7,2.90,'2023-06-19 16:38:00','submitted','dok_prestasi_52.pdf','PM0052',80.50,78.00,'IPS','Ruang 2'),
(53,'Prestasi siswa 53', 8,3.70,'2023-06-19 16:48:00','submitted','dok_prestasi_53.pdf','PM0053',88.00,85.50,'IPA','Ruang 3'),
(54,'Prestasi siswa 54', 9,1.20,'2023-06-19 16:58:00','submitted','dok_prestasi_54.pdf','PM0054',83.25,80.75,'IPS','Ruang 4'),
(55,'Prestasi siswa 55',10,2.60,'2023-06-19 17:08:00','submitted','dok_prestasi_55.pdf','PM0055',90.75,88.25,'IPA','Ruang 5'),
(56,'Prestasi siswa 56',11,3.40,'2023-06-19 17:18:00','submitted','dok_prestasi_56.pdf','PM0056',77.00,74.50,'IPS','Ruang 6'),
(57,'Prestasi siswa 57',12,1.90,'2023-06-19 17:28:00','submitted','dok_prestasi_57.pdf','PM0057',91.25,88.75,'IPA','Ruang 7'),
(58,'Prestasi siswa 58',13,4.30,'2023-06-19 17:38:00','submitted','dok_prestasi_58.pdf','PM0058',85.50,83.00,'IPS','Ruang 8'),
(59,'Prestasi siswa 59',14,2.10,'2023-06-19 17:48:00','submitted','dok_prestasi_59.pdf','PM0059',79.25,76.75,'IPA','Ruang 9'),
(60,'Prestasi siswa 60',15,3.50,'2023-06-19 17:58:00','submitted','dok_prestasi_60.pdf','PM0060',86.50,84.00,'IPS','Ruang 10');

-- ============================================================
-- REGISTRATIONS_ZONASI (60 data)
-- ============================================================
INSERT INTO registrations_zonasi (
    user_id, school_destination_id, distance,
    verification_schedule, status, documents, kk_date
) VALUES
(1,  1, 1.20,'2023-06-19 08:12:00','submitted','dok_zonasi_01.pdf','2020-01-10'),
(2,  2, 2.10,'2023-06-19 08:22:00','submitted','dok_zonasi_02.pdf','2020-02-15'),
(3,  3, 1.80,'2023-06-19 08:32:00','submitted','dok_zonasi_03.pdf','2020-03-20'),
(4,  4, 3.40,'2023-06-19 08:42:00','submitted','dok_zonasi_04.pdf','2020-04-25'),
(5,  5, 2.60,'2023-06-19 08:52:00','submitted','dok_zonasi_05.pdf','2020-05-30'),
(6,  6, 1.50,'2023-06-19 09:02:00','submitted','dok_zonasi_06.pdf','2020-06-05'),
(7,  7, 4.20,'2023-06-19 09:12:00','submitted','dok_zonasi_07.pdf','2020-07-10'),
(8,  8, 3.10,'2023-06-19 09:22:00','submitted','dok_zonasi_08.pdf','2020-08-15'),
(9,  9, 2.90,'2023-06-19 09:32:00','submitted','dok_zonasi_09.pdf','2020-09-20'),
(10,10, 1.70,'2023-06-19 09:42:00','submitted','dok_zonasi_10.pdf','2020-10-25'),
(11,11, 2.30,'2023-06-19 09:52:00','submitted','dok_zonasi_11.pdf','2020-11-30'),
(12,12, 4.50,'2023-06-19 10:02:00','submitted','dok_zonasi_12.pdf','2020-12-05'),
(13,13, 3.80,'2023-06-19 10:12:00','submitted','dok_zonasi_13.pdf','2021-01-10'),
(14,14, 1.40,'2023-06-19 10:22:00','submitted','dok_zonasi_14.pdf','2021-02-15'),
(15,15, 2.70,'2023-06-19 10:32:00','submitted','dok_zonasi_15.pdf','2021-03-20'),
(16, 1, 1.10,'2023-06-19 10:42:00','submitted','dok_zonasi_16.pdf','2021-04-25'),
(17, 2, 2.20,'2023-06-19 10:52:00','submitted','dok_zonasi_17.pdf','2021-05-30'),
(18, 3, 3.30,'2023-06-19 11:02:00','submitted','dok_zonasi_18.pdf','2021-06-05'),
(19, 4, 1.60,'2023-06-19 11:12:00','submitted','dok_zonasi_19.pdf','2021-07-10'),
(20, 5, 2.40,'2023-06-19 11:22:00','submitted','dok_zonasi_20.pdf','2021-08-15'),
(21, 6, 3.70,'2023-06-19 11:32:00','submitted','dok_zonasi_21.pdf','2021-09-20'),
(22, 7, 1.30,'2023-06-19 11:42:00','submitted','dok_zonasi_22.pdf','2021-10-25'),
(23, 8, 4.10,'2023-06-19 11:52:00','submitted','dok_zonasi_23.pdf','2021-11-30'),
(24, 9, 2.80,'2023-06-19 12:02:00','submitted','dok_zonasi_24.pdf','2021-12-05'),
(25,10, 1.90,'2023-06-19 12:12:00','submitted','dok_zonasi_25.pdf','2022-01-10'),
(26,11, 3.20,'2023-06-19 12:22:00','submitted','dok_zonasi_26.pdf','2022-02-15'),
(27,12, 2.50,'2023-06-19 12:32:00','submitted','dok_zonasi_27.pdf','2022-03-20'),
(28,13, 4.30,'2023-06-19 12:42:00','submitted','dok_zonasi_28.pdf','2022-04-25'),
(29,14, 1.80,'2023-06-19 12:52:00','submitted','dok_zonasi_29.pdf','2022-05-30'),
(30,15, 3.60,'2023-06-19 13:02:00','submitted','dok_zonasi_30.pdf','2022-06-05'),
(31, 1, 1.50,'2023-06-19 13:12:00','submitted','dok_zonasi_31.pdf','2022-07-10'),
(32, 2, 2.70,'2023-06-19 13:22:00','submitted','dok_zonasi_32.pdf','2022-08-15'),
(33, 3, 1.20,'2023-06-19 13:32:00','submitted','dok_zonasi_33.pdf','2022-09-20'),
(34, 4, 3.10,'2023-06-19 13:42:00','submitted','dok_zonasi_34.pdf','2022-10-25'),
(35, 5, 2.30,'2023-06-19 13:52:00','submitted','dok_zonasi_35.pdf','2022-11-30'),
(36, 6, 4.40,'2023-06-19 14:02:00','submitted','dok_zonasi_36.pdf','2022-12-05'),
(37, 7, 1.70,'2023-06-19 14:12:00','submitted','dok_zonasi_37.pdf','2023-01-10'),
(38, 8, 3.90,'2023-06-19 14:22:00','submitted','dok_zonasi_38.pdf','2023-02-15'),
(39, 9, 2.10,'2023-06-19 14:32:00','submitted','dok_zonasi_39.pdf','2023-03-20'),
(40,10, 1.40,'2023-06-19 14:42:00','submitted','dok_zonasi_40.pdf','2023-04-25'),
(41,11, 3.60,'2023-06-19 14:52:00','submitted','dok_zonasi_41.pdf','2023-05-01'),
(42,12, 2.80,'2023-06-19 15:02:00','submitted','dok_zonasi_42.pdf','2023-05-05'),
(43,13, 4.20,'2023-06-19 15:12:00','submitted','dok_zonasi_43.pdf','2023-05-10'),
(44,14, 1.60,'2023-06-19 15:22:00','submitted','dok_zonasi_44.pdf','2023-05-15'),
(45,15, 3.30,'2023-06-19 15:32:00','submitted','dok_zonasi_45.pdf','2023-05-20'),
(46, 1, 2.10,'2023-06-19 15:42:00','submitted','dok_zonasi_46.pdf','2023-05-25'),
(47, 1, 1.80,'2023-06-19 15:52:00','submitted','dok_zonasi_47.pdf','2023-05-28'),
(48, 1, 3.20,'2023-06-19 16:02:00','submitted','dok_zonasi_48.pdf','2023-06-01'),
(49, 2, 2.50,'2023-06-19 16:12:00','submitted','dok_zonasi_49.pdf','2023-06-03'),
(50, 2, 1.40,'2023-06-19 16:22:00','submitted','dok_zonasi_50.pdf','2023-06-05'),
(51, 1, 4.10,'2023-06-19 16:32:00','submitted','dok_zonasi_51.pdf','2023-06-07'),
(52, 1, 2.90,'2023-06-19 16:42:00','submitted','dok_zonasi_52.pdf','2023-06-09'),
(53, 2, 3.70,'2023-06-19 16:52:00','submitted','dok_zonasi_53.pdf','2023-06-11'),
(54, 2, 1.20,'2023-06-19 17:02:00','submitted','dok_zonasi_54.pdf','2023-06-13'),
(55, 1, 2.60,'2023-06-19 17:12:00','submitted','dok_zonasi_55.pdf','2023-06-15'),
(56, 1, 3.40,'2023-06-19 17:22:00','submitted','dok_zonasi_56.pdf','2023-06-16'),
(57, 2, 1.90,'2023-06-19 17:32:00','submitted','dok_zonasi_57.pdf','2023-06-17'),
(58, 2, 4.30,'2023-06-19 17:42:00','submitted','dok_zonasi_58.pdf','2023-06-18'),
(59, 1, 2.10,'2023-06-19 17:52:00','submitted','dok_zonasi_59.pdf','2023-06-19'),
(60, 2, 3.50,'2023-06-19 18:02:00','submitted','dok_zonasi_60.pdf','2023-06-19');
```

---

## STEP 5: Insert VERIFIKASI — Campuran Approved dan Rejected

```sql
-- ============================================================
-- VERIFICATION_AFIRMASI (60 data)
-- Kerusakan: 
--   id 1-40  : 'approved'  → akan ada penerimaan
--   id 41-50 : 'approved'  → TIDAK akan ada penerimaan (rusak!)
--   id 51-60 : 'rejected'  → TAPI akan tetap dibuat penerimaan (rusak!)
-- ============================================================
INSERT INTO verification_afirmasi (
    registration_id, operator_id, action, alasan_batal
) VALUES
-- NORMAL: approved (id 1-40)
(1, 1,'approved',NULL),(2, 2,'approved',NULL),(3, 3,'approved',NULL),
(4, 4,'approved',NULL),(5, 5,'approved',NULL),(6, 6,'approved',NULL),
(7, 7,'approved',NULL),(8, 8,'approved',NULL),(9, 9,'approved',NULL),
(10,10,'approved',NULL),(11,11,'approved',NULL),(12,12,'approved',NULL),
(13,13,'approved',NULL),(14,14,'approved',NULL),(15,15,'approved',NULL),
(16,16,'approved',NULL),(17,17,'approved',NULL),(18,18,'approved',NULL),
(19,19,'approved',NULL),(20,20,'approved',NULL),(21,21,'approved',NULL),
(22,22,'approved',NULL),(23,23,'approved',NULL),(24,24,'approved',NULL),
(25,25,'approved',NULL),(26,26,'approved',NULL),(27,27,'approved',NULL),
(28,28,'approved',NULL),(29,29,'approved',NULL),(30,30,'approved',NULL),
(31,31,'approved',NULL),(32,32,'approved',NULL),(33,33,'approved',NULL),
(34,34,'approved',NULL),(35,35,'approved',NULL),(36,36,'approved',NULL),
(37,37,'approved',NULL),(38,38,'approved',NULL),(39,39,'approved',NULL),
(40,40,'approved',NULL),
-- RUSAK: approved tapi TIDAK akan ada penerimaan (id 41-50)
(41,41,'approved',NULL),(42,42,'approved',NULL),(43,43,'approved',NULL),
(44,44,'approved',NULL),(45,45,'approved',NULL),(46,46,'approved',NULL),
(47,47,'approved',NULL),(48,48,'approved',NULL),(49,49,'approved',NULL),
(50,50,'approved',NULL),
-- RUSAK: rejected tapi AKAN ada penerimaan (id 51-60)
(51,51,'rejected','Dokumen KIP tidak valid'),
(52,52,'rejected','Dokumen tidak lengkap'),
(53,53,'rejected','NISN tidak ditemukan'),
(54,54,'rejected','Foto tidak jelas'),
(55,55,'rejected','Tanda tangan tidak sah'),
(56,56,'rejected','Dokumen KIP kadaluarsa'),
(57,57,'rejected','Alamat tidak sesuai KK'),
(58,58,'rejected','Dokumen tidak asli'),
(59,59,'rejected','Data tidak konsisten'),
(60,60,'rejected','Berkas tidak lengkap');

-- ============================================================
-- VERIFICATION_MUTASI (60 data)
-- Kerusakan serupa dengan afirmasi
-- ============================================================
INSERT INTO verification_mutasi (
    registration_id, operator_id, action, alasan_batal
) VALUES
(1, 1,'approved',NULL),(2, 2,'approved',NULL),(3, 3,'approved',NULL),
(4, 4,'approved',NULL),(5, 5,'approved',NULL),(6, 6,'approved',NULL),
(7, 7,'approved',NULL),(8, 8,'approved',NULL),(9, 9,'approved',NULL),
(10,10,'approved',NULL),(11,11,'approved',NULL),(12,12,'approved',NULL),
(13,13,'approved',NULL),(14,14,'approved',NULL),(15,15,'approved',NULL),
(16,16,'approved',NULL),(17,17,'approved',NULL),(18,18,'approved',NULL),
(19,19,'approved',NULL),(20,20,'approved',NULL),(21,21,'approved',NULL),
(22,22,'approved',NULL),(23,23,'approved',NULL),(24,24,'approved',NULL),
(25,25,'approved',NULL),(26,26,'approved',NULL),(27,27,'approved',NULL),
(28,28,'approved',NULL),(29,29,'approved',NULL),(30,30,'approved',NULL),
(31,31,'approved',NULL),(32,32,'approved',NULL),(33,33,'approved',NULL),
(34,34,'approved',NULL),(35,35,'approved',NULL),(36,36,'approved',NULL),
(37,37,'approved',NULL),(38,38,'approved',NULL),(39,39,'approved',NULL),
(40,40,'approved',NULL),
-- RUSAK: approved tapi tidak ada penerimaan
(41,41,'approved',NULL),(42,42,'approved',NULL),(43,43,'approved',NULL),
(44,44,'approved',NULL),(45,45,'approved',NULL),(46,46,'approved',NULL),
(47,47,'approved',NULL),(48,48,'approved',NULL),(49,49,'approved',NULL),
(50,50,'approved',NULL),
-- RUSAK: rejected tapi akan ada penerimaan
(51,51,'rejected','Surat tugas tidak valid'),
(52,52,'rejected','Instansi tidak dikenal'),
(53,53,'rejected','Periode tugas tidak sesuai'),
(54,54,'rejected','Tanda tangan pejabat palsu'),
(55,55,'rejected','Dokumen expired'),
(56,56,'rejected','Tidak ada legalisir'),
(57,57,'rejected','Format surat salah'),
(58,58,'rejected','Cap instansi tidak ada'),
(59,59,'rejected','Nama tidak sesuai KTP'),
(60,60,'rejected','Dokumen tidak bisa dibuka');

-- ============================================================
-- VERIFICATION_PRESTASI_MANDIRI (60 data)
-- ============================================================
INSERT INTO verification_prestasi_mandiri (
    registration_id, operator_id, action, alasan_batal
) VALUES
(1, 1,'approved',NULL),(2, 2,'approved',NULL),(3, 3,'approved',NULL),
(4, 4,'approved',NULL),(5, 5,'approved',NULL),(6, 6,'approved',NULL),
(7, 7,'approved',NULL),(8, 8,'approved',NULL),(9, 9,'approved',NULL),
(10,10,'approved',NULL),(11,11,'approved',NULL),(12,12,'approved',NULL),
(13,13,'approved',NULL),(14,14,'approved',NULL),(15,15,'approved',NULL),
(16,16,'approved',NULL),(17,17,'approved',NULL),(18,18,'approved',NULL),
(19,19,'approved',NULL),(20,20,'approved',NULL),(21,21,'approved',NULL),
(22,22,'approved',NULL),(23,23,'approved',NULL),(24,24,'approved',NULL),
(25,25,'approved',NULL),(26,26,'approved',NULL),(27,27,'approved',NULL),
(28,28,'approved',NULL),(29,29,'approved',NULL),(30,30,'approved',NULL),
(31,31,'approved',NULL),(32,32,'approved',NULL),(33,33,'approved',NULL),
(34,34,'approved',NULL),(35,35,'approved',NULL),(36,36,'approved',NULL),
(37,37,'approved',NULL),(38,38,'approved',NULL),(39,39,'approved',NULL),
(40,40,'approved',NULL),
(41,41,'approved',NULL),(42,42,'approved',NULL),(43,43,'approved',NULL),
(44,44,'approved',NULL),(45,45,'approved',NULL),(46,46,'approved',NULL),
(47,47,'approved',NULL),(48,48,'approved',NULL),(49,49,'approved',NULL),
(50,50,'approved',NULL),
(51,51,'rejected','Nilai tidak memenuhi minimum'),
(52,52,'rejected','Piagam tidak terverifikasi'),
(53,53,'rejected','Tingkat kejuaraan tidak sesuai'),
(54,54,'rejected','Tahun kejuaraan terlalu lama'),
(55,55,'rejected','Nama kejuaraan tidak valid'),
(56,56,'rejected','Scan piagam tidak jelas'),
(57,57,'rejected','Cabang lomba tidak sesuai jurusan'),
(58,58,'rejected','Lembaga penyelenggara tidak diakui'),
(59,59,'rejected','Piagam tidak ditandatangani'),
(60,60,'rejected','Data piagam tidak konsisten');

-- ============================================================
-- VERIFICATION_ZONASI (60 data)
-- ============================================================
INSERT INTO verification_zonasi (
    registration_id, operator_id, action, alasan_batal
) VALUES
(1, 1,'approved',NULL),(2, 2,'approved',NULL),(3, 3,'approved',NULL),
(4, 4,'approved',NULL),(5, 5,'approved',NULL),(6, 6,'approved',NULL),
(7, 7,'approved',NULL),(8, 8,'approved',NULL),(9, 9,'approved',NULL),
(10,10,'approved',NULL),(11,11,'approved',NULL),(12,12,'approved',NULL),
(13,13,'approved',NULL),(14,14,'approved',NULL),(15,15,'approved',NULL),
(16,16,'approved',NULL),(17,17,'approved',NULL),(18,18,'approved',NULL),
(19,19,'approved',NULL),(20,20,'approved',NULL),(21,21,'approved',NULL),
(22,22,'approved',NULL),(23,23,'approved',NULL),(24,24,'approved',NULL),
(25,25,'approved',NULL),(26,26,'approved',NULL),(27,27,'approved',NULL),
(28,28,'approved',NULL),(29,29,'approved',NULL),(30,30,'approved',NULL),
(31,31,'approved',NULL),(32,32,'approved',NULL),(33,33,'approved',NULL),
(34,34,'approved',NULL),(35,35,'approved',NULL),(36,36,'approved',NULL),
(37,37,'approved',NULL),(38,38,'approved',NULL),(39,39,'approved',NULL),
(40,40,'approved',NULL),
(41,41,'approved',NULL),(42,42,'approved',NULL),(43,43,'approved',NULL),
(44,44,'approved',NULL),(45,45,'approved',NULL),(46,46,'approved',NULL),
(47,47,'approved',NULL),(48,48,'approved',NULL),(49,49,'approved',NULL),
(50,50,'approved',NULL),
(51,51,'rejected','Jarak melebihi radius zonasi'),
(52,52,'rejected','KK baru dibuat < 1 tahun'),
(53,53,'rejected','Alamat KK tidak sesuai'),
(54,54,'rejected','KK tidak bisa diverifikasi'),
(55,55,'rejected','Daerah bukan zonasi sekolah ini'),
(56,56,'rejected','KK expired'),
(57,57,'rejected','Nama di KK berbeda dengan ijazah'),
(58,58,'rejected','KK tidak ditandatangani lurah'),
(59,59,'rejected','Foto KK tidak jelas'),
(60,60,'rejected','Data koordinat tidak valid');
```

---

## STEP 6: Insert PENERIMAAN — Data Rusak

```sql
-- ============================================================
-- PENERIMAAN_AFIRMASI
-- KERUSAKAN TIPE 1: id 41-50 (approved) TIDAK dimasukkan
-- KERUSAKAN TIPE 2: id 51-60 (rejected) DIMASUKKAN (salah!)
-- Hanya id 1-40 yang benar + id 51-60 yang harusnya tidak ada
-- ============================================================
INSERT INTO penerimaan_afirmasi (
    verification_id, kepsek_id, code, seen_at
) VALUES
-- BENAR: id 1-40 (approved → ada penerimaan) ✓
(1, 6,'AFR-2023-0001','2023-06-23 08:00:00'),
(2, 6,'AFR-2023-0002','2023-06-23 08:05:00'),
(3, 6,'AFR-2023-0003','2023-06-23 08:10:00'),
(4, 6,'AFR-2023-0004','2023-06-23 08:15:00'),
(5, 6,'AFR-2023-0005','2023-06-23 08:20:00'),
(6, 6,'AFR-2023-0006','2023-06-23 08:25:00'),
(7, 6,'AFR-2023-0007','2023-06-23 08:30:00'),
(8, 6,'AFR-2023-0008','2023-06-23 08:35:00'),
(9, 6,'AFR-2023-0009','2023-06-23 08:40:00'),
(10,6,'AFR-2023-0010','2023-06-23 08:45:00'),
(11,6,'AFR-2023-0011','2023-06-23 08:50:00'),
(12,6,'AFR-2023-0012','2023-06-23 08:55:00'),
(13,6,'AFR-2023-0013','2023-06-23 09:00:00'),
(14,6,'AFR-2023-0014','2023-06-23 09:05:00'),
(15,6,'AFR-2023-0015','2023-06-23 09:10:00'),
(16,6,'AFR-2023-0016','2023-06-23 09:15:00'),
(17,6,'AFR-2023-0017','2023-06-23 09:20:00'),
(18,6,'AFR-2023-0018','2023-06-23 09:25:00'),
(19,6,'AFR-2023-0019','2023-06-23 09:30:00'),
(20,6,'AFR-2023-0020','2023-06-23 09:35:00'),
(21,6,'AFR-2023-0021','2023-06-23 09:40:00'),
(22,6,'AFR-2023-0022','2023-06-23 09:45:00'),
(23,6,'AFR-2023-0023','2023-06-23 09:50:00'),
(24,6,'AFR-2023-0024','2023-06-23 09:55:00'),
(25,6,'AFR-2023-0025','2023-06-23 10:00:00'),
(26,6,'AFR-2023-0026','2023-06-23 10:05:00'),
(27,6,'AFR-2023-0027','2023-06-23 10:10:00'),
(28,6,'AFR-2023-0028','2023-06-23 10:15:00'),
(29,6,'AFR-2023-0029','2023-06-23 10:20:00'),
(30,6,'AFR-2023-0030','2023-06-23 10:25:00'),
(31,6,'AFR-2023-0031','2023-06-23 10:30:00'),
(32,6,'AFR-2023-0032','2023-06-23 10:35:00'),
(33,6,'AFR-2023-0033','2023-06-23 10:40:00'),
(34,6,'AFR-2023-0034','2023-06-23 10:45:00'),
(35,6,'AFR-2023-0035','2023-06-23 10:50:00'),
(36,6,'AFR-2023-0036','2023-06-23 10:55:00'),
(37,6,'AFR-2023-0037','2023-06-23 11:00:00'),
(38,6,'AFR-2023-0038','2023-06-23 11:05:00'),
(39,6,'AFR-2023-0039','2023-06-23 11:10:00'),
(40,6,'AFR-2023-0040','2023-06-23 11:15:00'),
-- id 41-50 SENGAJA TIDAK DIINSERT (approved tapi tidak ada penerimaan) ← RUSAK
-- RUSAK: id 51-60 (rejected) tapi ADA penerimaan ← SEHARUSNYA TIDAK ADA!
(51,6,'AFR-2023-0051','2023-06-23 14:00:00'),
(52,6,'AFR-2023-0052','2023-06-23 14:05:00'),
(53,6,'AFR-2023-0053','2023-06-23 14:10:00'),
(54,6,'AFR-2023-0054','2023-06-23 14:15:00'),
(55,6,'AFR-2023-0055','2023-06-23 14:20:00'),
(56,6,'AFR-2023-0056','2023-06-23 14:25:00'),
(57,6,'AFR-2023-0057','2023-06-23 14:30:00'),
(58,6,'AFR-2023-0058','2023-06-23 14:35:00'),
(59,6,'AFR-2023-0059','2023-06-23 14:40:00'),
(60,6,'AFR-2023-0060','2023-06-23 14:45:00');

-- ============================================================
-- PENERIMAAN_MUTASI (sama polanya dengan afirmasi)
-- ============================================================
INSERT INTO penerimaan_mutasi (verification_id, code, seen_at) VALUES
(1,'MTS-2023-0001','2023-06-23 08:02:00'),(2,'MTS-2023-0002','2023-06-23 08:07:00'),
(3,'MTS-2023-0003','2023-06-23 08:12:00'),(4,'MTS-2023-0004','2023-06-23 08:17:00'),
(5,'MTS-2023-0005','2023-06-23 08:22:00'),(6,'MTS-2023-0006','2023-06-23 08:27:00'),
(7,'MTS-2023-0007','2023-06-23 08:32:00'),(8,'MTS-2023-0008','2023-06-23 08:37:00'),
(9,'MTS-2023-0009','2023-06-23 08:42:00'),(10,'MTS-2023-0010','2023-06-23 08:47:00'),
(11,'MTS-2023-0011','2023-06-23 08:52:00'),(12,'MTS-2023-0012','2023-06-23 08:57:00'),
(13,'MTS-2023-0013','2023-06-23 09:02:00'),(14,'MTS-2023-0014','2023-06-23 09:07:00'),
(15,'MTS-2023-0015','2023-06-23 09:12:00'),(16,'MTS-2023-0016','2023-06-23 09:17:00'),
(17,'MTS-2023-0017','2023-06-23 09:22:00'),(18,'MTS-2023-0018','2023-06-23 09:27:00'),
(19,'MTS-2023-0019','2023-06-23 09:32:00'),(20,'MTS-2023-0020','2023-06-23 09:37:00'),
(21,'MTS-2023-0021','2023-06-23 09:42:00'),(22,'MTS-2023-0022','2023-06-23 09:47:00'),
(23,'MTS-2023-0023','2023-06-23 09:52:00'),(24,'MTS-2023-0024','2023-06-23 09:57:00'),
(25,'MTS-2023-0025','2023-06-23 10:02:00'),(26,'MTS-2023-0026','2023-06-23 10:07:00'),
(27,'MTS-2023-0027','2023-06-23 10:12:00'),(28,'MTS-2023-0028','2023-06-23 10:17:00'),
(29,'MTS-2023-0029','2023-06-23 10:22:00'),(30,'MTS-2023-0030','2023-06-23 10:27:00'),
(31,'MTS-2023-0031','2023-06-23 10:32:00'),(32,'MTS-2023-0032','2023-06-23 10:37:00'),
(33,'MTS-2023-0033','2023-06-23 10:42:00'),(34,'MTS-2023-0034','2023-06-23 10:47:00'),
(35,'MTS-2023-0035','2023-06-23 10:52:00'),(36,'MTS-2023-0036','2023-06-23 10:57:00'),
(37,'MTS-2023-0037','2023-06-23 11:02:00'),(38,'MTS-2023-0038','2023-06-23 11:07:00'),
(39,'MTS-2023-0039','2023-06-23 11:12:00'),(40,'MTS-2023-0040','2023-06-23 11:17:00'),
-- id 41-50 SENGAJA TIDAK DIINSERT ← RUSAK
-- id 51-60 (rejected) tapi ada penerimaan ← RUSAK
(51,'MTS-2023-0051','2023-06-23 14:02:00'),(52,'MTS-2023-0052','2023-06-23 14:07:00'),
(53,'MTS-2023-0053','2023-06-23 14:12:00'),(54,'MTS-2023-0054','2023-06-23 14:17:00'),
(55,'MTS-2023-0055','2023-06-23 14:22:00'),(56,'MTS-2023-0056','2023-06-23 14:27:00'),
(57,'MTS-2023-0057','2023-06-23 14:32:00'),(58,'MTS-2023-0058','2023-06-23 14:37:00'),
(59,'MTS-2023-0059','2023-06-23 14:42:00'),(60,'MTS-2023-0060','2023-06-23 14:47:00');

-- ============================================================
-- PENERIMAAN_PRESTASI_MANDIRI
-- ============================================================
INSERT INTO penerimaan_prestasi_mandiri (npsn, verification_id, code, seen_at)
SELECT
    s.npsn,
    v.id,
    'PRS-2023-' || LPAD(v.id::text, 4, '0'),
    CASE
        WHEN v.id <= 40 THEN
            TIMESTAMP '2023-06-23 08:04:00'
            + ((v.id - 1) * INTERVAL '5 minutes')
        ELSE
            TIMESTAMP '2023-06-23 14:04:00'
            + ((v.id - 51) * INTERVAL '5 minutes')
    END
FROM verification_prestasi_mandiri v
JOIN registrations_prestasi_mandiri r
    ON r.id = v.registration_id
JOIN schools s
    ON s.id = r.school_destination_id
WHERE v.id <= 40      -- yang benar (approved)
   OR v.id BETWEEN 51 AND 60;  -- yang rusak (rejected tapi ada penerimaan)
-- id 41-50 SENGAJA DILEWAT ← RUSAK

-- ============================================================
-- PENERIMAAN_ZONASI
-- Tambahan kerusakan: siswa 31-45 juga masuk di zonasi
-- padahal sudah masuk di afirmasi (multi jalur = RUSAK)
-- ============================================================
INSERT INTO penerimaan_zonasi (verification_id, code, seen_at)
SELECT
    vz.id,
    'ZNS-2023-' || LPAD(vz.id::text, 4, '0'),
    CASE
        WHEN vz.id <= 40 THEN
            TIMESTAMP '2023-06-23 08:06:00'
            + ((vz.id - 1) * INTERVAL '5 minutes')
        ELSE
            TIMESTAMP '2023-06-23 14:06:00'
            + ((vz.id - 51) * INTERVAL '5 minutes')
    END
FROM verification_zonasi vz
WHERE vz.id <= 40         -- yang benar
   OR vz.id BETWEEN 51 AND 60;  -- rusak: rejected tapi ada penerimaan
-- id 41-50 SENGAJA DILEWAT ← RUSAK
```

---

## STEP 7: Verifikasi Kerusakan Data

```sql
-- ============================================================
-- CEK REKAP JUMLAH DATA
-- ============================================================
SELECT tabel, jumlah FROM (
    SELECT 'roles'                          AS tabel, COUNT(*) AS jumlah FROM roles
    UNION ALL SELECT 'office_users',          COUNT(*) FROM office_users
    UNION ALL SELECT 'schools',               COUNT(*) FROM schools
    UNION ALL SELECT 'school_users',          COUNT(*) FROM school_users
    UNION ALL SELECT 'junior_schools',        COUNT(*) FROM junior_schools
    UNION ALL SELECT 'users',                 COUNT(*) FROM users
    UNION ALL SELECT 'reg_afirmasi',          COUNT(*) FROM registrations_afirmasi
    UNION ALL SELECT 'reg_mutasi',            COUNT(*) FROM registrations_mutasi
    UNION ALL SELECT 'reg_prestasi_mandiri',  COUNT(*) FROM registrations_prestasi_mandiri
    UNION ALL SELECT 'reg_zonasi',            COUNT(*) FROM registrations_zonasi
    UNION ALL SELECT 'verif_afirmasi',        COUNT(*) FROM verification_afirmasi
    UNION ALL SELECT 'verif_mutasi',          COUNT(*) FROM verification_mutasi
    UNION ALL SELECT 'verif_prestasi',        COUNT(*) FROM verification_prestasi_mandiri
    UNION ALL SELECT 'verif_zonasi',          COUNT(*) FROM verification_zonasi
    UNION ALL SELECT 'penerimaan_afirmasi',   COUNT(*) FROM penerimaan_afirmasi
    UNION ALL SELECT 'penerimaan_mutasi',     COUNT(*) FROM penerimaan_mutasi
    UNION ALL SELECT 'penerimaan_prestasi',   COUNT(*) FROM penerimaan_prestasi_mandiri
    UNION ALL SELECT 'penerimaan_zonasi',     COUNT(*) FROM penerimaan_zonasi
) x ORDER BY tabel;

-- ============================================================
-- KONFIRMASI KERUSAKAN 1:
-- Approved tapi tidak ada penerimaan (id 41-50 di tiap jalur)
-- Harusnya: 10 baris per jalur = 40 total
-- ============================================================
SELECT 'Afirmasi' AS jalur, COUNT(*) AS jumlah_bermasalah
FROM verification_afirmasi va
LEFT JOIN penerimaan_afirmasi pa ON pa.verification_id = va.id
WHERE va.action = 'approved' AND pa.id IS NULL
UNION ALL
SELECT 'Mutasi', COUNT(*)
FROM verification_mutasi vm
LEFT JOIN penerimaan_mutasi pm ON pm.verification_id = vm.id
WHERE vm.action = 'approved' AND pm.id IS NULL
UNION ALL
SELECT 'Prestasi', COUNT(*)
FROM verification_prestasi_mandiri vp
LEFT JOIN penerimaan_prestasi_mandiri pp ON pp.verification_id = vp.id
WHERE vp.action = 'approved' AND pp.id IS NULL
UNION ALL
SELECT 'Zonasi', COUNT(*)
FROM verification_zonasi vz
LEFT JOIN penerimaan_zonasi pz ON pz.verification_id = vz.id
WHERE vz.action = 'approved' AND pz.id IS NULL;

-- Hasil yang diharapkan:
-- jalur    │ jumlah_bermasalah
-- ─────────┼───────────────────
-- Afirmasi │        10
-- Mutasi   │        10
-- Prestasi │        10
-- Zonasi   │        10

-- ============================================================
-- KONFIRMASI KERUSAKAN 2:
-- Rejected tapi ADA penerimaan (id 51-60 di tiap jalur)
-- Harusnya: 10 baris per jalur = 40 total
-- ============================================================
SELECT 'Afirmasi' AS jalur, COUNT(*) AS jumlah_bermasalah
FROM verification_afirmasi va
JOIN penerimaan_afirmasi pa ON pa.verification_id = va.id
WHERE va.action = 'rejected'
UNION ALL
SELECT 'Mutasi', COUNT(*)
FROM verification_mutasi vm
JOIN penerimaan_mutasi pm ON pm.verification_id = vm.id
WHERE vm.action = 'rejected'
UNION ALL
SELECT 'Prestasi', COUNT(*)
FROM verification_prestasi_mandiri vp
JOIN penerimaan_prestasi_mandiri pp ON pp.verification_id = vp.id
WHERE vp.action = 'rejected'
UNION ALL
SELECT 'Zonasi', COUNT(*)
FROM verification_zonasi vz
JOIN penerimaan_zonasi pz ON pz.verification_id = vz.id
WHERE vz.action = 'rejected';

-- Hasil yang diharapkan:
-- jalur    │ jumlah_bermasalah
-- ─────────┼───────────────────
-- Afirmasi │        10
-- Mutasi   │        10
-- Prestasi │        10
-- Zonasi   │        10

-- ============================================================
-- KONFIRMASI KERUSAKAN 3:
-- Siswa diterima di lebih dari 1 jalur (user 31-45)
-- ============================================================
WITH gabung AS (
    SELECT u.id AS uid, u.name, 'Afirmasi' AS jalur
    FROM penerimaan_afirmasi pa
    JOIN verification_afirmasi va ON va.id = pa.verification_id
    JOIN registrations_afirmasi ra ON ra.id = va.registration_id
    JOIN users u ON u.id = ra.user_id
    UNION ALL
    SELECT u.id, u.name, 'Mutasi'
    FROM penerimaan_mutasi pm
    JOIN verification_mutasi vm ON vm.id = pm.verification_id
    JOIN registrations_mutasi rm ON rm.id = vm.registration_id
    JOIN users u ON u.id = rm.user_id
    UNION ALL
    SELECT u.id, u.name, 'Prestasi'
    FROM penerimaan_prestasi_mandiri pp
    JOIN verification_prestasi_mandiri vp ON vp.id = pp.verification_id
    JOIN registrations_prestasi_mandiri rp ON rp.id = vp.registration_id
    JOIN users u ON u.id = rp.user_id
    UNION ALL
    SELECT u.id, u.name, 'Zonasi'
    FROM penerimaan_zonasi pz
    JOIN verification_zonasi vz ON vz.id = pz.verification_id
    JOIN registrations_zonasi rz ON rz.id = vz.registration_id
    JOIN users u ON u.id = rz.user_id
)
SELECT uid, name,
    COUNT(DISTINCT jalur) AS jumlah_jalur,
    STRING_AGG(jalur, ', ' ORDER BY jalur) AS jalur_diterima
FROM gabung
GROUP BY uid, name
HAVING COUNT(DISTINCT jalur) > 1
ORDER BY jumlah_jalur DESC, uid;

-- ============================================================
-- KONFIRMASI KERUSAKAN 4:
-- Sekolah 1 dan 2 overflow pagu zonasi
-- pagu_zonasi = 5, tapi user 46-59 semua diarahkan ke sana
-- ============================================================
SELECT
    s.name AS sekolah,
    s.pagu_afirmasi,
    s.pagu_zonasi,
    COUNT(DISTINCT pa.id) AS realisasi_afirmasi,
    COUNT(DISTINCT pz.id) AS realisasi_zonasi,
    COUNT(DISTINCT pa.id) - s.pagu_afirmasi AS selisih_afirmasi,
    COUNT(DISTINCT pz.id) - s.pagu_zonasi   AS selisih_zonasi
FROM schools s
LEFT JOIN registrations_afirmasi ra ON ra.school_destination_id = s.id
LEFT JOIN verification_afirmasi va  ON va.registration_id = ra.id
    AND va.action = 'approved'
LEFT JOIN penerimaan_afirmasi pa    ON pa.verification_id = va.id
LEFT JOIN registrations_zonasi rz   ON rz.school_destination_id = s.id
LEFT JOIN verification_zonasi vz    ON vz.registration_id = rz.id
    AND vz.action = 'approved'
LEFT JOIN penerimaan_zonasi pz      ON pz.verification_id = vz.id
WHERE s.id IN (1, 2)
GROUP BY s.id, s.name, s.pagu_afirmasi, s.pagu_zonasi;
```

---

## Ringkasan Kerusakan yang Tertanam

```
╔═════╦══════════════════════════════════╦══════════════════════╦══════════╗
║ No  ║ Jenis Kerusakan                  ║ Data yang Terdampak  ║ Jumlah   ║
╠═════╬══════════════════════════════════╬══════════════════════╬══════════╣
║  1  ║ Approved tapi TIDAK ada          ║ verif id 41-50       ║ 40 baris ║
║     ║ penerimaan (4 jalur)             ║ di 4 tabel verif     ║ rusak    ║
╠═════╬══════════════════════════════════╬══════════════════════╬══════════╣
║  2  ║ Rejected tapi ADA penerimaan     ║ verif id 51-60       ║ 40 baris ║
║     ║ (4 jalur)                        ║ di 4 tabel penerimaan║ rusak    ║
╠═════╬══════════════════════════════════╬══════════════════════╬══════════╣
║  3  ║ Siswa diterima di lebih dari     ║ user id 31-45        ║ 15 siswa ║
║     ║ 1 jalur sekaligus                ║ masuk di 2 jalur     ║ rusak    ║
╠═════╬══════════════════════════════════╬══════════════════════╬══════════╣
║  4  ║ Penerimaan melebihi pagu         ║ school id 1 dan 2    ║ 2 sekolah║
║     ║ (overflow kuota)                 ║ pagu=5, realisasi>5  ║ overflow ║
╠═════╬══════════════════════════════════╬══════════════════════╬══════════╣
║  5  ║ Status registrasi tidak atomik   ║ SEMUA users (1-60)   ║ 60 baris ║
║     ║ (tetap 'submitted' meski terima) ║ status = 'submitted' ║ rusak    ║
╚═════╩══════════════════════════════════╩══════════════════════╩══════════╝
```
