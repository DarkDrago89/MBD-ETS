# MBD-ETS
# Laporan Manajemen Basis Data
## Pemicu 1: Polemik Masalah PPDB Online
### Implementasi PostgreSQL

---

# BAGIAN 1: DESAIN DATABASE PPDB

## 1.1 Entity Relationship Diagram (Konseptual)

```
╔══════════════════════════════════════════════════════════════════╗
║              ENTITY RELATIONSHIP DIAGRAM - PPDB ONLINE          ║
╚══════════════════════════════════════════════════════════════════╝

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 KELOMPOK A: MANAJEMEN PENGGUNA & SEKOLAH
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

[1] Relasi Role dan Petugas
    ROLES ||──────────o{ OFFICE_USERS : "dimiliki oleh"
    (1 role dimiliki oleh 0 atau banyak petugas)

[2] Relasi Petugas dan Sekolah (Many-to-Many via SCHOOL_USERS)
    OFFICE_USERS ||──────────o{ SCHOOL_USERS : "ditugaskan ke"
    SCHOOLS      ||──────────o{ SCHOOL_USERS : "memiliki petugas"
    (1 petugas bisa di banyak sekolah, 1 sekolah punya banyak petugas)
    Constraint UNIQUE (office_user_username, school_npsn)
    mencegah duplikasi penugasan

[3] Relasi Sekolah Asal Siswa
    JUNIOR_SCHOOLS ||──────────o{ USERS : "asal sekolah dari"
    (1 SMP menjadi asal dari 0 atau banyak calon siswa)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 KELOMPOK B: JALUR PENDAFTARAN
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

[4] Calon Siswa ke Registrasi
    USERS ||──────────o{ REGISTRATIONS_AFIRMASI         : "mendaftar via"
    USERS ||──────────o{ REGISTRATIONS_MUTASI           : "mendaftar via"
    USERS ||──────────o{ REGISTRATIONS_PRESTASI_MANDIRI : "mendaftar via"
    USERS ||──────────o{ REGISTRATIONS_ZONASI           : "mendaftar via"

[5] Sekolah Tujuan ke Registrasi
    SCHOOLS ||──────────o{ REGISTRATIONS_AFIRMASI         : "menjadi tujuan"
    SCHOOLS ||──────────o{ REGISTRATIONS_MUTASI           : "menjadi tujuan"
    SCHOOLS ||──────────o{ REGISTRATIONS_PRESTASI_MANDIRI : "menjadi tujuan"
    SCHOOLS ||──────────o{ REGISTRATIONS_ZONASI           : "menjadi tujuan"

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 KELOMPOK C: PROSES VERIFIKASI
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

[6] Registrasi ke Verifikasi
    REGISTRATIONS_AFIRMASI         ||──────────|| VERIFICATION_AFIRMASI         : "diverifikasi"
    REGISTRATIONS_MUTASI           ||──────────|| VERIFICATION_MUTASI           : "diverifikasi"
    REGISTRATIONS_PRESTASI_MANDIRI ||──────────|| VERIFICATION_PRESTASI_MANDIRI : "diverifikasi"
    REGISTRATIONS_ZONASI           ||──────────|| VERIFICATION_ZONASI           : "diverifikasi"
    (1 registrasi tepat 1 verifikasi, karena registration_id UNIQUE NOT NULL)

[7] Petugas ke Verifikasi
    OFFICE_USERS ||──────────o{ VERIFICATION_AFIRMASI         : "memverifikasi"
    OFFICE_USERS ||──────────o{ VERIFICATION_MUTASI           : "memverifikasi"
    OFFICE_USERS ||──────────o{ VERIFICATION_PRESTASI_MANDIRI : "memverifikasi"
    OFFICE_USERS ||──────────o{ VERIFICATION_ZONASI           : "memverifikasi"
    (operator_id NOT NULL, petugas wajib ada saat verifikasi)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 KELOMPOK D: PROSES PENERIMAAN
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

[8] Verifikasi ke Penerimaan
    VERIFICATION_AFIRMASI         ||──────────|| PENERIMAAN_AFIRMASI         : "menghasilkan"
    VERIFICATION_MUTASI           ||──────────|| PENERIMAAN_MUTASI           : "menghasilkan"
    VERIFICATION_PRESTASI_MANDIRI ||──────────|| PENERIMAAN_PRESTASI_MANDIRI : "menghasilkan"
    VERIFICATION_ZONASI           ||──────────|| PENERIMAAN_ZONASI           : "menghasilkan"
    (verification_id UNIQUE NOT NULL, relasi 1-ke-1 penuh)

[9] Kepala Sekolah ke Penerimaan Afirmasi (KHUSUS jalur afirmasi)
    OFFICE_USERS ||──────────o{ PENERIMAAN_AFIRMASI : "ditetapkan oleh kepala sekolah"
    (kepsek_id nullable / ON DELETE SET NULL,
     hanya ada di jalur afirmasi)

[10] Sekolah ke Penerimaan Prestasi Mandiri (via NPSN)
    SCHOOLS ||──────────o{ PENERIMAAN_PRESTASI_MANDIRI : "tempat diterima"
    (npsn NOT NULL, ON DELETE RESTRICT,
     hanya ada di jalur prestasi mandiri)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 ALUR PROSES LENGKAP
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  JUNIOR_SCHOOLS ──(asal sekolah)──► USERS ◄──(sekolah tujuan)── SCHOOLS
                                       │                               │
                         ┌─────────────┼───────────────────────────────┘
                         │             │ (memilih jalur)
               ┌─────────▼──────────┐  │
               │    Memilih Jalur   │  │
               └──┬──────┬──────┬───┘  │
                  │      │      │      │
          ┌───────┘  ┌───┘  ┌───┘  ┌───┘
          ▼          ▼      ▼      ▼
    REG_         REG_    REG_    REG_
    AFIRMASI     MUTASI  PRESTASI ZONASI
          │          │      │      │
          └──────────┴──────┴──────┘
                         │
              ┌──────────▼──────────┐
              │  VERIFICATION       │◄── OFFICE_USERS
              │  (registration_id   │    (operator, NOT NULL)
              │   UNIQUE NOT NULL)  │
              └──────────┬──────────┘
                         │ (1 verifikasi → 1 penerimaan)
              ┌──────────▼──────────┐
              │  PENERIMAAN         │
              │  (verification_id   │◄── OFFICE_USERS (kepsek,
              │   UNIQUE NOT NULL)  │    khusus afirmasi, nullable)
              └─────────────────────┘
                         │
                    (khusus prestasi)
                         │
                      SCHOOLS (via npsn)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 KETERANGAN NOTASI
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  ||   = Tepat satu / exactly one (1)
  o{   = Nol atau banyak / zero or many (0..*)
  o|   = Nol atau satu / zero or one (0..1)
  ──   = Garis relasi / relationship line
```

---

## 1.2 Skema Relasional

```
╔══════════════════════════════════════════════════════════════════╗
║                        SKEMA RELASIONAL                         ║
╚══════════════════════════════════════════════════════════════════╝

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 KELOMPOK A: MANAJEMEN PENGGUNA & SEKOLAH
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

┌─────────────────────────────────────────────────────────────────┐
│ ROLES                                                           │
├──────────────┬──────────────────┬──────────────────────────────┤
│ Kolom        │ Tipe Data        │ Constraint / Keterangan      │
├──────────────┼──────────────────┼──────────────────────────────┤
│ id           │ SERIAL           │ PRIMARY KEY                  │
│ name         │ VARCHAR(100)     │ NOT NULL                     │
│ created_at   │ TIMESTAMP        │ DEFAULT CURRENT_TIMESTAMP    │
│ updated_at   │ TIMESTAMP        │ DEFAULT CURRENT_TIMESTAMP    │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│ OFFICE_USERS                                                    │
├──────────────┬──────────────────┬──────────────────────────────┤
│ Kolom        │ Tipe Data        │ Constraint / Keterangan      │
├──────────────┼──────────────────┼──────────────────────────────┤
│ id           │ SERIAL           │ PRIMARY KEY                  │
│ name         │ VARCHAR(255)     │ NOT NULL                     │
│ username     │ VARCHAR(100)     │ NOT NULL, UNIQUE             │
│ password     │ VARCHAR(255)     │ NOT NULL                     │
│ role_id      │ INT              │ NOT NULL                     │
│              │                  │ FK → ROLES(id)               │
│              │                  │ ON UPDATE CASCADE            │
│              │                  │ ON DELETE RESTRICT           │
│ created_at   │ TIMESTAMP        │ DEFAULT CURRENT_TIMESTAMP    │
│ updated_at   │ TIMESTAMP        │ DEFAULT CURRENT_TIMESTAMP    │
└─────────────────────────────────────────────────────────────────┘
  Index: idx_office_users_role_id ON (role_id)

┌─────────────────────────────────────────────────────────────────┐
│ SCHOOLS                                                         │
├──────────────────────────────┬──────────────┬───────────────────┤
│ Kolom                        │ Tipe Data    │ Constraint        │
├──────────────────────────────┼──────────────┼───────────────────┤
│ id                           │ SERIAL       │ PRIMARY KEY       │
│ name                         │ VARCHAR(255) │ NOT NULL          │
│ kode                         │ VARCHAR(50)  │                   │
│ npsn                         │ VARCHAR(20)  │ NOT NULL, UNIQUE  │
│ minimum_average              │ DECIMAL(5,2) │                   │
│ city_id                      │ INT          │                   │
│ created_at                   │ TIMESTAMP    │ DEFAULT NOW()     │
│ updated_at                   │ TIMESTAMP    │ DEFAULT NOW()     │
│ latitude                     │ DECIMAL(10,7)│                   │
│ longitude                    │ DECIMAL(10,7)│                   │
│ pagu_afirmasi                │ INT          │ DEFAULT 0         │
│ pagu_mutasi                  │ INT          │ DEFAULT 0         │
│ pagu_prestasi_undangan       │ INT          │ DEFAULT 0         │
│ pagu_zonasi                  │ INT          │ DEFAULT 0         │
│ pagu_prestasi_tesmandiri     │ INT          │ DEFAULT 0         │
│ pagu_tidak_naik_kelas        │ INT          │ DEFAULT 0         │
│ school_code                  │ VARCHAR(20)  │                   │
│ kebijakan_sisa_pagu_afirmasi │ TEXT         │                   │
│ kebijakan_sisa_pagu_mutasi   │ TEXT         │                   │
│ kebijakan_sisa_pagu_undangan │ TEXT         │                   │
│ kebijakan_sisa_pagu_zonasi   │ TEXT         │                   │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│ SCHOOL_USERS              [Junction Table: Many-to-Many]        │
├──────────────────────┬──────────────┬─────────────────────────── ┤
│ Kolom                │ Tipe Data    │ Constraint / Keterangan    │
├──────────────────────┼──────────────┼────────────────────────────┤
│ id                   │ SERIAL       │ PRIMARY KEY                │
│ office_user_username │ VARCHAR(100) │ NOT NULL                   │
│                      │              │ FK → OFFICE_USERS(username)│
│                      │              │ ON UPDATE CASCADE          │
│                      │              │ ON DELETE CASCADE          │
│ school_npsn          │ VARCHAR(20)  │ NOT NULL                   │
│                      │              │ FK → SCHOOLS(npsn)         │
│                      │              │ ON UPDATE CASCADE          │
│                      │              │ ON DELETE CASCADE          │
│ ── UNIQUE ──         │              │ (office_user_username,     │
│                      │              │  school_npsn)              │
└─────────────────────────────────────────────────────────────────┘
  Index: idx_school_users_office_user_username ON (office_user_username)
         idx_school_users_school_npsn          ON (school_npsn)

┌─────────────────────────────────────────────────────────────────┐
│ JUNIOR_SCHOOLS                                                  │
├──────────────┬──────────────────┬──────────────────────────────┤
│ Kolom        │ Tipe Data        │ Constraint / Keterangan      │
├──────────────┼──────────────────┼──────────────────────────────┤
│ id           │ SERIAL           │ PRIMARY KEY                  │
│ npsn         │ VARCHAR(20)      │ NOT NULL, UNIQUE             │
│ name         │ VARCHAR(255)     │ NOT NULL                     │
│ created_at   │ TIMESTAMP        │ DEFAULT CURRENT_TIMESTAMP    │
│ updated_at   │ TIMESTAMP        │ DEFAULT CURRENT_TIMESTAMP    │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│ USERS                              [Calon Peserta Didik]        │
├──────────────────────┬──────────────┬──────────────────────────┤
│ Kolom                │ Tipe Data    │ Constraint / Keterangan  │
├──────────────────────┼──────────────┼──────────────────────────┤
│ id                   │ SERIAL       │ PRIMARY KEY              │
│ npsn                 │ VARCHAR(20)  │ NOT NULL                 │
│                      │              │ FK → JUNIOR_SCHOOLS(npsn)│
│                      │              │ ON UPDATE CASCADE        │
│                      │              │ ON DELETE RESTRICT       │
│ nisn                 │ VARCHAR(20)  │ NOT NULL, UNIQUE         │
│ birth_date           │ DATE         │                          │
│ name                 │ VARCHAR(255) │ NOT NULL                 │
│ gender               │ VARCHAR(10)  │ CHECK (NULL,'L','P')     │
│ address              │ TEXT         │                          │
│ phone                │ VARCHAR(20)  │                          │
│ school_name          │ VARCHAR(255) │                          │
│ cluster              │ VARCHAR(100) │                          │
│ created_at           │ TIMESTAMP    │ DEFAULT NOW()            │
│ updated_at           │ TIMESTAMP    │ DEFAULT NOW()            │
│ school_destination_id│ INT          │ FK → SCHOOLS(id)         │
│                      │              │ ON UPDATE CASCADE        │
│                      │              │ ON DELETE SET NULL       │
│ latitude             │ DECIMAL(10,7)│                          │
│ longitude            │ DECIMAL(10,7)│                          │
│ registration_1_type  │ VARCHAR(50)  │ Jenis jalur ke-1         │
│ registration_2_type  │ VARCHAR(50)  │ Jenis jalur ke-2         │
│ registration_3_type  │ VARCHAR(50)  │ Jenis jalur ke-3         │
│ registration_1_id    │ INT          │ ID registrasi ke-1       │
│ registration_2_id    │ INT          │ ID registrasi ke-2       │
│ registration_3_id    │ INT          │ ID registrasi ke-3       │
│ acceptance_type      │ VARCHAR(50)  │ Jalur yang diterima      │
│ acceptance_id        │ INT          │ ID penerimaan            │
│ deleted_at           │ TIMESTAMP    │ Soft delete              │
│ jalur_prestasi       │ VARCHAR(100) │ Jenis prestasi           │
└─────────────────────────────────────────────────────────────────┘
  Index: idx_users_npsn                 ON (npsn)
         idx_users_school_destination_id ON (school_destination_id)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 KELOMPOK B: TABEL REGISTRASI (4 JALUR)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

┌─────────────────────────────────────────────────────────────────┐
│ REGISTRATIONS_AFIRMASI                                          │
├──────────────────────┬──────────────┬──────────────────────────┤
│ Kolom                │ Tipe Data    │ Constraint / Keterangan  │
├──────────────────────┼──────────────┼──────────────────────────┤
│ id                   │ SERIAL       │ PRIMARY KEY              │
│ user_id              │ INT          │ NOT NULL                 │
│                      │              │ FK → USERS(id)           │
│                      │              │ ON UPDATE CASCADE        │
│                      │              │ ON DELETE CASCADE        │
│ jenis                │ VARCHAR(100) │ Jenis afirmasi           │
│ school_destination_id│ INT          │ NOT NULL                 │
│                      │              │ FK → SCHOOLS(id)         │
│                      │              │ ON UPDATE CASCADE        │
│                      │              │ ON DELETE RESTRICT       │
│ distance             │ DECIMAL(10,2)│ Jarak ke sekolah (km)   │
│ verification_schedule│ TIMESTAMP    │ Jadwal verifikasi        │
│ status               │ VARCHAR(50)  │ pending/verified/        │
│                      │              │ rejected/accepted        │
│ created_at           │ TIMESTAMP    │ DEFAULT NOW()            │
│ updated_at           │ TIMESTAMP    │ DEFAULT NOW()            │
│ documents            │ TEXT         │ Daftar dokumen           │
└─────────────────────────────────────────────────────────────────┘
  Index: idx_reg_afirmasi_user_id              ON (user_id)
         idx_reg_afirmasi_school_destination_id ON (school_destination_id)

┌─────────────────────────────────────────────────────────────────┐
│ REGISTRATIONS_MUTASI                                            │
├──────────────────────┬──────────────┬──────────────────────────┤
│ Kolom                │ Tipe Data    │ Constraint / Keterangan  │
├──────────────────────┼──────────────┼──────────────────────────┤
│ id                   │ SERIAL       │ PRIMARY KEY              │
│ user_id              │ INT          │ NOT NULL                 │
│                      │              │ FK → USERS(id)           │
│                      │              │ ON UPDATE CASCADE        │
│                      │              │ ON DELETE CASCADE        │
│ jenis                │ VARCHAR(100) │ Jenis mutasi             │
│ school_destination_id│ INT          │ NOT NULL                 │
│                      │              │ FK → SCHOOLS(id)         │
│                      │              │ ON UPDATE CASCADE        │
│                      │              │ ON DELETE RESTRICT       │
│ distance             │ DECIMAL(10,2)│ Jarak ke sekolah (km)   │
│ verification_schedule│ TIMESTAMP    │ Jadwal verifikasi        │
│ status               │ VARCHAR(50)  │ pending/verified/        │
│                      │              │ rejected/accepted        │
│ created_at           │ TIMESTAMP    │ DEFAULT NOW()            │
│ updated_at           │ TIMESTAMP    │ DEFAULT NOW()            │
│ documents            │ TEXT         │ Daftar dokumen           │
└─────────────────────────────────────────────────────────────────┘
  Index: idx_reg_mutasi_user_id              ON (user_id)
         idx_reg_mutasi_school_destination_id ON (school_destination_id)

┌─────────────────────────────────────────────────────────────────┐
│ REGISTRATIONS_PRESTASI_MANDIRI                                  │
├──────────────────────┬──────────────┬──────────────────────────┤
│ Kolom                │ Tipe Data    │ Constraint / Keterangan  │
├──────────────────────┼──────────────┼──────────────────────────┤
│ id                   │ SERIAL       │ PRIMARY KEY              │
│ user_id              │ INT          │ NOT NULL                 │
│                      │              │ FK → USERS(id)           │
│                      │              │ ON UPDATE CASCADE        │
│                      │              │ ON DELETE CASCADE        │
│ data                 │ TEXT         │ Data prestasi            │
│ school_destination_id│ INT          │ NOT NULL                 │
│                      │              │ FK → SCHOOLS(id)         │
│                      │              │ ON UPDATE CASCADE        │
│                      │              │ ON DELETE RESTRICT       │
│ distance             │ DECIMAL(10,2)│ Jarak ke sekolah (km)   │
│ verification_schedule│ TIMESTAMP    │ Jadwal verifikasi        │
│ status               │ VARCHAR(50)  │ pending/verified/        │
│                      │              │ rejected/accepted        │
│ created_at           │ TIMESTAMP    │ DEFAULT NOW()            │
│ updated_at           │ TIMESTAMP    │ DEFAULT NOW()            │
│ documents            │ TEXT         │ Daftar dokumen           │
│ test_number          │ VARCHAR(50)  │ Nomor ujian              │
│ final_score          │ DECIMAL(5,2) │ Nilai akhir              │
│ test_score           │ DECIMAL(5,2) │ Nilai ujian              │
│ jurusan              │ VARCHAR(100) │ IPA / IPS                │
│ ruang_tes            │ VARCHAR(50)  │ Ruang ujian              │
└─────────────────────────────────────────────────────────────────┘
  Index: idx_reg_prestasi_user_id              ON (user_id)
         idx_reg_prestasi_school_destination_id ON (school_destination_id)

┌─────────────────────────────────────────────────────────────────┐
│ REGISTRATIONS_ZONASI                                            │
├──────────────────────┬──────────────┬──────────────────────────┤
│ Kolom                │ Tipe Data    │ Constraint / Keterangan  │
├──────────────────────┼──────────────┼──────────────────────────┤
│ id                   │ SERIAL       │ PRIMARY KEY              │
│ user_id              │ INT          │ NOT NULL                 │
│                      │              │ FK → USERS(id)           │
│                      │              │ ON UPDATE CASCADE        │
│                      │              │ ON DELETE CASCADE        │
│ school_destination_id│ INT          │ NOT NULL                 │
│                      │              │ FK → SCHOOLS(id)         │
│                      │              │ ON UPDATE CASCADE        │
│                      │              │ ON DELETE RESTRICT       │
│ distance             │ DECIMAL(10,2)│ Jarak ke sekolah (km)   │
│ verification_schedule│ TIMESTAMP    │ Jadwal verifikasi        │
│ status               │ VARCHAR(50)  │ pending/verified/        │
│                      │              │ rejected/accepted        │
│ created_at           │ TIMESTAMP    │ DEFAULT NOW()            │
│ updated_at           │ TIMESTAMP    │ DEFAULT NOW()            │
│ documents            │ TEXT         │ Daftar dokumen           │
│ kk_date              │ DATE         │ Tanggal terbit KK        │
└─────────────────────────────────────────────────────────────────┘
  Index: idx_reg_zonasi_user_id              ON (user_id)
         idx_reg_zonasi_school_destination_id ON (school_destination_id)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 KELOMPOK C: TABEL VERIFIKASI (4 JALUR)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

┌─────────────────────────────────────────────────────────────────┐
│ VERIFICATION_AFIRMASI                                           │
├──────────────────┬──────────────────┬───────────────────────────┤
│ Kolom            │ Tipe Data        │ Constraint / Keterangan   │
├──────────────────┼──────────────────┼───────────────────────────┤
│ id               │ SERIAL           │ PRIMARY KEY               │
│ registration_id  │ INT              │ NOT NULL, UNIQUE          │
│                  │                  │ FK →                      │
│                  │                  │ REGISTRATIONS_AFIRMASI(id)│
│                  │                  │ ON UPDATE CASCADE         │
│                  │                  │ ON DELETE CASCADE         │
│ operator_id      │ INT              │ NOT NULL                  │
│                  │                  │ FK → OFFICE_USERS(id)     │
│                  │                  │ ON UPDATE CASCADE         │
│                  │                  │ ON DELETE RESTRICT        │
│ action           │ VARCHAR(50)      │ approve/reject/pending    │
│ created_at       │ TIMESTAMP        │ DEFAULT NOW()             │
│ updated_at       │ TIMESTAMP        │ DEFAULT NOW()             │
│ alasan_batal     │ TEXT             │ Alasan jika ditolak       │
└─────────────────────────────────────────────────────────────────┘
  Index: idx_ver_afirmasi_operator_id ON (operator_id)
  Catatan: registration_id UNIQUE → relasi 1-ke-1 dengan registrasi

┌─────────────────────────────────────────────────────────────────┐
│ VERIFICATION_MUTASI                                             │
├──────────────────┬──────────────────┬───────────────────────────┤
│ Kolom            │ Tipe Data        │ Constraint / Keterangan   │
├──────────────────┼──────────────────┼───────────────────────────┤
│ id               │ SERIAL           │ PRIMARY KEY               │
│ registration_id  │ INT              │ NOT NULL, UNIQUE          │
│                  │                  │ FK →                      │
│                  │                  │ REGISTRATIONS_MUTASI(id)  │
│                  │                  │ ON UPDATE CASCADE         │
│                  │                  │ ON DELETE CASCADE         │
│ operator_id      │ INT              │ NOT NULL                  │
│                  │                  │ FK → OFFICE_USERS(id)     │
│                  │                  │ ON UPDATE CASCADE         │
│                  │                  │ ON DELETE RESTRICT        │
│ action           │ VARCHAR(50)      │ approve/reject/pending    │
│ created_at       │ TIMESTAMP        │ DEFAULT NOW()             │
│ updated_at       │ TIMESTAMP        │ DEFAULT NOW()             │
│ alasan_batal     │ TEXT             │ Alasan jika ditolak       │
└─────────────────────────────────────────────────────────────────┘
  Index: idx_ver_mutasi_operator_id ON (operator_id)

┌─────────────────────────────────────────────────────────────────┐
│ VERIFICATION_PRESTASI_MANDIRI                                   │
├──────────────────┬──────────────────┬───────────────────────────┤
│ Kolom            │ Tipe Data        │ Constraint / Keterangan   │
├──────────────────┼──────────────────┼───────────────────────────┤
│ id               │ SERIAL           │ PRIMARY KEY               │
│ registration_id  │ INT              │ NOT NULL, UNIQUE          │
│                  │                  │ FK →                      │
│                  │                  │ REGISTRATIONS_PRESTASI_   │
│                  │                  │ MANDIRI(id)               │
│                  │                  │ ON UPDATE CASCADE         │
│                  │                  │ ON DELETE CASCADE         │
│ operator_id      │ INT              │ NOT NULL                  │
│                  │                  │ FK → OFFICE_USERS(id)     │
│                  │                  │ ON UPDATE CASCADE         │
│                  │                  │ ON DELETE RESTRICT        │
│ action           │ VARCHAR(50)      │ approve/reject/pending    │
│ created_at       │ TIMESTAMP        │ DEFAULT NOW()             │
│ updated_at       │ TIMESTAMP        │ DEFAULT NOW()             │
│ alasan_batal     │ TEXT             │ Alasan jika ditolak       │
└─────────────────────────────────────────────────────────────────┘
  Index: idx_ver_prestasi_operator_id ON (operator_id)

┌─────────────────────────────────────────────────────────────────┐
│ VERIFICATION_ZONASI                                             │
├──────────────────┬──────────────────┬───────────────────────────┤
│ Kolom            │ Tipe Data        │ Constraint / Keterangan   │
├──────────────────┼──────────────────┼───────────────────────────┤
│ id               │ SERIAL           │ PRIMARY KEY               │
│ registration_id  │ INT              │ NOT NULL, UNIQUE          │
│                  │                  │ FK →                      │
│                  │                  │ REGISTRATIONS_ZONASI(id)  │
│                  │                  │ ON UPDATE CASCADE         │
│                  │                  │ ON DELETE CASCADE         │
│ operator_id      │ INT              │ NOT NULL                  │
│                  │                  │ FK → OFFICE_USERS(id)     │
│                  │                  │ ON UPDATE CASCADE         │
│                  │                  │ ON DELETE RESTRICT        │
│ action           │ VARCHAR(50)      │ approve/reject/pending    │
│ created_at       │ TIMESTAMP        │ DEFAULT NOW()             │
│ updated_at       │ TIMESTAMP        │ DEFAULT NOW()             │
│ alasan_batal     │ TEXT             │ Alasan jika ditolak       │
└─────────────────────────────────────────────────────────────────┘
  Index: idx_ver_zonasi_operator_id ON (operator_id)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 KELOMPOK D: TABEL PENERIMAAN (4 JALUR)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

┌─────────────────────────────────────────────────────────────────┐
│ PENERIMAAN_AFIRMASI                                             │
├──────────────────┬──────────────────┬───────────────────────────┤
│ Kolom            │ Tipe Data        │ Constraint / Keterangan   │
├──────────────────┼──────────────────┼───────────────────────────┤
│ id               │ SERIAL           │ PRIMARY KEY               │
│ verification_id  │ INT              │ NOT NULL, UNIQUE          │
│                  │                  │ FK →                      │
│                  │                  │ VERIFICATION_AFIRMASI(id) │
│                  │                  │ ON UPDATE CASCADE         │
│                  │                  │ ON DELETE CASCADE         │
│ kepsek_id        │ INT              │ nullable                  │
│                  │                  │ FK → OFFICE_USERS(id)     │
│                  │                  │ ON UPDATE CASCADE         │
│                  │                  │ ON DELETE SET NULL        │
│                  │                  │ [Khusus jalur afirmasi]   │
│ created_at       │ TIMESTAMP        │ DEFAULT NOW()             │
│ updated_at       │ TIMESTAMP        │ DEFAULT NOW()             │
│ code             │ VARCHAR(100)     │ UNIQUE, Kode penerimaan   │
│ seen_at          │ TIMESTAMP        │ Waktu dilihat siswa       │
└─────────────────────────────────────────────────────────────────┘
  Index: idx_penerimaan_afirmasi_kepsek_id ON (kepsek_id)
  Catatan: verification_id UNIQUE → relasi 1-ke-1 dengan verifikasi
           kepsek_id nullable karena ON DELETE SET NULL

┌─────────────────────────────────────────────────────────────────┐
│ PENERIMAAN_MUTASI                                               │
├──────────────────┬──────────────────┬───────────────────────────┤
│ Kolom            │ Tipe Data        │ Constraint / Keterangan   │
├──────────────────┼──────────────────┼───────────────────────────┤
│ id               │ SERIAL           │ PRIMARY KEY               │
│ created_at       │ TIMESTAMP        │ DEFAULT NOW()             │
│ updated_at       │ TIMESTAMP        │ DEFAULT NOW()             │
│ verification_id  │ INT              │ NOT NULL, UNIQUE          │
│                  │                  │ FK →                      │
│                  │                  │ VERIFICATION_MUTASI(id)   │
│                  │                  │ ON UPDATE CASCADE         │
│                  │                  │ ON DELETE CASCADE         │
│ code             │ VARCHAR(100)     │ UNIQUE, Kode penerimaan   │
│ seen_at          │ TIMESTAMP        │ Waktu dilihat siswa       │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│ PENERIMAAN_PRESTASI_MANDIRI                                     │
├──────────────────┬──────────────────┬───────────────────────────┤
│ Kolom            │ Tipe Data        │ Constraint / Keterangan   │
├──────────────────┼──────────────────┼───────────────────────────┤
│ id               │ SERIAL           │ PRIMARY KEY               │
│ npsn             │ VARCHAR(20)      │ NOT NULL                  │
│                  │                  │ FK → SCHOOLS(npsn)        │
│                  │                  │ ON UPDATE CASCADE         │
│                  │                  │ ON DELETE RESTRICT        │
│                  │                  │ [Sekolah penerima]        │
│ verification_id  │ INT              │ NOT NULL, UNIQUE          │
│                  │                  │ FK →                      │
│                  │                  │ VERIFICATION_PRESTASI_    │
│                  │                  │ MANDIRI(id)               │
│                  │                  │ ON UPDATE CASCADE         │
│                  │                  │ ON DELETE CASCADE         │
│ code             │ VARCHAR(100)     │ UNIQUE, Kode penerimaan   │
│ seen_at          │ TIMESTAMP        │ Waktu dilihat siswa       │
└─────────────────────────────────────────────────────────────────┘
  Index: idx_penerimaan_prestasi_npsn ON (npsn)
  Catatan: TIDAK memiliki created_at dan updated_at
           Memiliki kolom npsn FK → SCHOOLS(npsn) yang unik
           di antara tabel penerimaan lainnya

┌─────────────────────────────────────────────────────────────────┐
│ PENERIMAAN_ZONASI                                               │
├──────────────────┬──────────────────┬───────────────────────────┤
│ Kolom            │ Tipe Data        │ Constraint / Keterangan   │
├──────────────────┼──────────────────┼───────────────────────────┤
│ id               │ SERIAL           │ PRIMARY KEY               │
│ created_at       │ TIMESTAMP        │ DEFAULT NOW()             │
│ updated_at       │ TIMESTAMP        │ DEFAULT NOW()             │
│ verification_id  │ INT              │ NOT NULL, UNIQUE          │
│                  │                  │ FK →                      │
│                  │                  │ VERIFICATION_ZONASI(id)   │
│                  │                  │ ON UPDATE CASCADE         │
│                  │                  │ ON DELETE CASCADE         │
│ code             │ VARCHAR(100)     │ UNIQUE, Kode penerimaan   │
│ seen_at          │ TIMESTAMP        │ Waktu dilihat siswa       │
└─────────────────────────────────────────────────────────────────┘

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 PERBANDINGAN STRUKTUR ANTAR TABEL PENERIMAAN
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  Kolom              AFIRMASI   MUTASI   PRESTASI_MANDIRI   ZONASI
  ────────────────────────────────────────────────────────────────
  id                 ✅         ✅       ✅                 ✅
  verification_id    ✅ UNIQUE   ✅UNIQUE ✅ UNIQUE          ✅ UNIQUE
  code               ✅ UNIQUE   ✅UNIQUE ✅ UNIQUE          ✅ UNIQUE
  seen_at            ✅         ✅       ✅                 ✅
  created_at         ✅         ✅       ❌ Tidak ada        ✅
  updated_at         ✅         ✅       ❌ Tidak ada        ✅
  kepsek_id          ✅ Khusus  ❌       ❌                 ❌
  npsn (FK Schools)  ❌         ❌       ✅ Khusus          ❌

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 RINGKASAN FOREIGN KEY DAN REFERENTIAL ACTION
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  Tabel Asal                    Kolom FK               Tujuan                  ON DELETE
  ──────────────────────────────────────────────────────────────────────────────────────
  OFFICE_USERS                  role_id                ROLES(id)               RESTRICT
  SCHOOL_USERS                  office_user_username   OFFICE_USERS(username)  CASCADE
  SCHOOL_USERS                  school_npsn            SCHOOLS(npsn)           CASCADE
  USERS                         npsn                   JUNIOR_SCHOOLS(npsn)    RESTRICT
  USERS                         school_destination_id  SCHOOLS(id)             SET NULL
  REGISTRATIONS_AFIRMASI        user_id                USERS(id)               CASCADE
  REGISTRATIONS_AFIRMASI        school_destination_id  SCHOOLS(id)             RESTRICT
  REGISTRATIONS_MUTASI          user_id                USERS(id)               CASCADE
  REGISTRATIONS_MUTASI          school_destination_id  SCHOOLS(id)             RESTRICT
  REGISTRATIONS_PRESTASI_MANDIRI user_id               USERS(id)               CASCADE
  REGISTRATIONS_PRESTASI_MANDIRI school_destination_id SCHOOLS(id)             RESTRICT
  REGISTRATIONS_ZONASI          user_id                USERS(id)               CASCADE
  REGISTRATIONS_ZONASI          school_destination_id  SCHOOLS(id)             RESTRICT
  VERIFICATION_AFIRMASI         registration_id        REG_AFIRMASI(id)        CASCADE
  VERIFICATION_AFIRMASI         operator_id            OFFICE_USERS(id)        RESTRICT
  VERIFICATION_MUTASI           registration_id        REG_MUTASI(id)          CASCADE
  VERIFICATION_MUTASI           operator_id            OFFICE_USERS(id)        RESTRICT
  VERIFICATION_PRESTASI_MANDIRI registration_id        REG_PRESTASI(id)        CASCADE
  VERIFICATION_PRESTASI_MANDIRI operator_id            OFFICE_USERS(id)        RESTRICT
  VERIFICATION_ZONASI           registration_id        REG_ZONASI(id)          CASCADE
  VERIFICATION_ZONASI           operator_id            OFFICE_USERS(id)        RESTRICT
  PENERIMAAN_AFIRMASI           verification_id        VER_AFIRMASI(id)        CASCADE
  PENERIMAAN_AFIRMASI           kepsek_id              OFFICE_USERS(id)        SET NULL
  PENERIMAAN_MUTASI             verification_id        VER_MUTASI(id)          CASCADE
  PENERIMAAN_PRESTASI_MANDIRI   npsn                   SCHOOLS(npsn)           RESTRICT
  PENERIMAAN_PRESTASI_MANDIRI   verification_id        VER_PRESTASI(id)        CASCADE
  PENERIMAAN_ZONASI             verification_id        VER_ZONASI(id)          CASCADE
```

---

## 1.3 DDL Lengkap (PostgreSQL)

```sql
-- ============================================================
-- Pemicu 1: Polemik Masalah PPDB Online
-- Skema Database PPDB - PostgreSQL
-- ============================================================

-- Drop tables in dependency order
DROP TABLE IF EXISTS penerimaan_zonasi CASCADE;
DROP TABLE IF EXISTS penerimaan_prestasi_mandiri CASCADE;
DROP TABLE IF EXISTS penerimaan_mutasi CASCADE;
DROP TABLE IF EXISTS penerimaan_afirmasi CASCADE;
DROP TABLE IF EXISTS verification_zonasi CASCADE;
DROP TABLE IF EXISTS verification_prestasi_mandiri CASCADE;
DROP TABLE IF EXISTS verification_mutasi CASCADE;
DROP TABLE IF EXISTS verification_afirmasi CASCADE;
DROP TABLE IF EXISTS registrations_zonasi CASCADE;
DROP TABLE IF EXISTS registrations_prestasi_mandiri CASCADE;
DROP TABLE IF EXISTS registrations_mutasi CASCADE;
DROP TABLE IF EXISTS registrations_afirmasi CASCADE;
DROP TABLE IF EXISTS users CASCADE;
DROP TABLE IF EXISTS school_users CASCADE;
DROP TABLE IF EXISTS junior_schools CASCADE;
DROP TABLE IF EXISTS office_users CASCADE;
DROP TABLE IF EXISTS schools CASCADE;
DROP TABLE IF EXISTS roles CASCADE;

-- ============================================================
-- KELOMPOK A: MANAJEMEN PENGGUNA DAN SEKOLAH
-- ============================================================

CREATE TABLE roles (
    id         SERIAL PRIMARY KEY,
    name       VARCHAR(100) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE office_users (
    id         SERIAL PRIMARY KEY,
    name       VARCHAR(255) NOT NULL,
    username   VARCHAR(100) NOT NULL UNIQUE,
    password   VARCHAR(255) NOT NULL,
    role_id    INT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    CONSTRAINT fk_office_users_roles
        FOREIGN KEY (role_id)
        REFERENCES roles(id)
        ON UPDATE CASCADE
        ON DELETE RESTRICT
);

CREATE TABLE schools (
    id                            SERIAL PRIMARY KEY,
    name                          VARCHAR(255) NOT NULL,
    kode                          VARCHAR(50),
    npsn                          VARCHAR(20) NOT NULL UNIQUE,
    minimum_average               DECIMAL(5,2),
    city_id                       INT,
    created_at                    TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at                    TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    latitude                      DECIMAL(10,7),
    longitude                     DECIMAL(10,7),
    pagu_afirmasi                 INT DEFAULT 0,
    pagu_mutasi                   INT DEFAULT 0,
    pagu_prestasi_undangan        INT DEFAULT 0,
    pagu_zonasi                   INT DEFAULT 0,
    pagu_prestasi_tesmandiri      INT DEFAULT 0,
    pagu_tidak_naik_kelas         INT DEFAULT 0,
    school_code                   VARCHAR(20),
    kebijakan_sisa_pagu_afirmasi  TEXT,
    kebijakan_sisa_pagu_mutasi    TEXT,
    kebijakan_sisa_pagu_undangan  TEXT,
    kebijakan_sisa_pagu_zonasi    TEXT
);

CREATE TABLE school_users (
    id                   SERIAL PRIMARY KEY,
    office_user_username VARCHAR(100) NOT NULL,
    school_npsn          VARCHAR(20)  NOT NULL,

    CONSTRAINT fk_school_users_office_users
        FOREIGN KEY (office_user_username)
        REFERENCES office_users(username)
        ON UPDATE CASCADE
        ON DELETE CASCADE,

    CONSTRAINT fk_school_users_schools
        FOREIGN KEY (school_npsn)
        REFERENCES schools(npsn)
        ON UPDATE CASCADE
        ON DELETE CASCADE,

    CONSTRAINT uq_school_users_assignment
        UNIQUE (office_user_username, school_npsn)
);

CREATE TABLE junior_schools (
    id         SERIAL PRIMARY KEY,
    npsn       VARCHAR(20)  NOT NULL UNIQUE,
    name       VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE users (
    id                    SERIAL PRIMARY KEY,
    npsn                  VARCHAR(20)  NOT NULL,
    nisn                  VARCHAR(20)  NOT NULL UNIQUE,
    birth_date            DATE,
    name                  VARCHAR(255) NOT NULL,
    gender                VARCHAR(10),
    address               TEXT,
    phone                 VARCHAR(20),
    school_name           VARCHAR(255),
    cluster               VARCHAR(100),
    created_at            TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at            TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    school_destination_id INT,
    latitude              DECIMAL(10,7),
    longitude             DECIMAL(10,7),
    registration_1_type   VARCHAR(50),
    registration_2_type   VARCHAR(50),
    registration_3_type   VARCHAR(50),
    registration_1_id     INT,
    registration_2_id     INT,
    registration_3_id     INT,
    acceptance_type       VARCHAR(50),
    acceptance_id         INT,
    deleted_at            TIMESTAMP,
    jalur_prestasi        VARCHAR(100),

    CONSTRAINT chk_users_gender
        CHECK (gender IS NULL OR gender IN ('L', 'P')),

    CONSTRAINT fk_users_junior_schools
        FOREIGN KEY (npsn)
        REFERENCES junior_schools(npsn)
        ON UPDATE CASCADE
        ON DELETE RESTRICT,

    CONSTRAINT fk_users_schools
        FOREIGN KEY (school_destination_id)
        REFERENCES schools(id)
        ON UPDATE CASCADE
        ON DELETE SET NULL
);

-- ============================================================
-- KELOMPOK B: TABEL REGISTRASI
-- ============================================================

CREATE TABLE registrations_afirmasi (
    id                    SERIAL PRIMARY KEY,
    user_id               INT NOT NULL,
    jenis                 VARCHAR(100),
    school_destination_id INT NOT NULL,
    distance              DECIMAL(10,2),
    verification_schedule TIMESTAMP,
    status                VARCHAR(50),
    created_at            TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at            TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    documents             TEXT,

    CONSTRAINT fk_reg_afirmasi_users
        FOREIGN KEY (user_id)
        REFERENCES users(id)
        ON UPDATE CASCADE
        ON DELETE CASCADE,

    CONSTRAINT fk_reg_afirmasi_schools
        FOREIGN KEY (school_destination_id)
        REFERENCES schools(id)
        ON UPDATE CASCADE
        ON DELETE RESTRICT
);

CREATE TABLE registrations_mutasi (
    id                    SERIAL PRIMARY KEY,
    user_id               INT NOT NULL,
    jenis                 VARCHAR(100),
    school_destination_id INT NOT NULL,
    distance              DECIMAL(10,2),
    verification_schedule TIMESTAMP,
    status                VARCHAR(50),
    created_at            TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at            TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    documents             TEXT,

    CONSTRAINT fk_reg_mutasi_users
        FOREIGN KEY (user_id)
        REFERENCES users(id)
        ON UPDATE CASCADE
        ON DELETE CASCADE,

    CONSTRAINT fk_reg_mutasi_schools
        FOREIGN KEY (school_destination_id)
        REFERENCES schools(id)
        ON UPDATE CASCADE
        ON DELETE RESTRICT
);

CREATE TABLE registrations_prestasi_mandiri (
    id                    SERIAL PRIMARY KEY,
    user_id               INT NOT NULL,
    data                  TEXT,
    school_destination_id INT NOT NULL,
    distance              DECIMAL(10,2),
    verification_schedule TIMESTAMP,
    status                VARCHAR(50),
    created_at            TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at            TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    documents             TEXT,
    test_number           VARCHAR(50),
    final_score           DECIMAL(5,2),
    test_score            DECIMAL(5,2),
    jurusan               VARCHAR(100),
    ruang_tes             VARCHAR(50),

    CONSTRAINT fk_reg_prestasi_users
        FOREIGN KEY (user_id)
        REFERENCES users(id)
        ON UPDATE CASCADE
        ON DELETE CASCADE,

    CONSTRAINT fk_reg_prestasi_schools
        FOREIGN KEY (school_destination_id)
        REFERENCES schools(id)
        ON UPDATE CASCADE
        ON DELETE RESTRICT
);

CREATE TABLE registrations_zonasi (
    id                    SERIAL PRIMARY KEY,
    user_id               INT NOT NULL,
    school_destination_id INT NOT NULL,
    distance              DECIMAL(10,2),
    verification_schedule TIMESTAMP,
    status                VARCHAR(50),
    created_at            TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at            TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    documents             TEXT,
    kk_date               DATE,

    CONSTRAINT fk_reg_zonasi_users
        FOREIGN KEY (user_id)
        REFERENCES users(id)
        ON UPDATE CASCADE
        ON DELETE CASCADE,

    CONSTRAINT fk_reg_zonasi_schools
        FOREIGN KEY (school_destination_id)
        REFERENCES schools(id)
        ON UPDATE CASCADE
        ON DELETE RESTRICT
);

-- ============================================================
-- KELOMPOK C: TABEL VERIFIKASI
-- ============================================================

CREATE TABLE verification_afirmasi (
    id              SERIAL PRIMARY KEY,
    registration_id INT NOT NULL UNIQUE,
    operator_id     INT NOT NULL,
    action          VARCHAR(50),
    created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    alasan_batal    TEXT,

    CONSTRAINT fk_ver_afirmasi_reg
        FOREIGN KEY (registration_id)
        REFERENCES registrations_afirmasi(id)
        ON UPDATE CASCADE
        ON DELETE CASCADE,

    CONSTRAINT fk_ver_afirmasi_operator
        FOREIGN KEY (operator_id)
        REFERENCES office_users(id)
        ON UPDATE CASCADE
        ON DELETE RESTRICT
);

CREATE TABLE verification_mutasi (
    id              SERIAL PRIMARY KEY,
    registration_id INT NOT NULL UNIQUE,
    operator_id     INT NOT NULL,
    action          VARCHAR(50),
    created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    alasan_batal    TEXT,

    CONSTRAINT fk_ver_mutasi_reg
        FOREIGN KEY (registration_id)
        REFERENCES registrations_mutasi(id)
        ON UPDATE CASCADE
        ON DELETE CASCADE,

    CONSTRAINT fk_ver_mutasi_operator
        FOREIGN KEY (operator_id)
        REFERENCES office_users(id)
        ON UPDATE CASCADE
        ON DELETE RESTRICT
);

CREATE TABLE verification_prestasi_mandiri (
    id              SERIAL PRIMARY KEY,
    registration_id INT NOT NULL UNIQUE,
    operator_id     INT NOT NULL,
    action          VARCHAR(50),
    created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    alasan_batal    TEXT,

    CONSTRAINT fk_ver_prestasi_reg
        FOREIGN KEY (registration_id)
        REFERENCES registrations_prestasi_mandiri(id)
        ON UPDATE CASCADE
        ON DELETE CASCADE,

    CONSTRAINT fk_ver_prestasi_operator
        FOREIGN KEY (operator_id)
        REFERENCES office_users(id)
        ON UPDATE CASCADE
        ON DELETE RESTRICT
);

CREATE TABLE verification_zonasi (
    id              SERIAL PRIMARY KEY,
    registration_id INT NOT NULL UNIQUE,
    operator_id     INT NOT NULL,
    action          VARCHAR(50),
    created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    alasan_batal    TEXT,

    CONSTRAINT fk_ver_zonasi_reg
        FOREIGN KEY (registration_id)
        REFERENCES registrations_zonasi(id)
        ON UPDATE CASCADE
        ON DELETE CASCADE,

    CONSTRAINT fk_ver_zonasi_operator
        FOREIGN KEY (operator_id)
        REFERENCES office_users(id)
        ON UPDATE CASCADE
        ON DELETE RESTRICT
);

-- ============================================================
-- KELOMPOK D: TABEL PENERIMAAN
-- ============================================================

CREATE TABLE penerimaan_afirmasi (
    id              SERIAL PRIMARY KEY,
    verification_id INT NOT NULL UNIQUE,
    kepsek_id       INT,
    created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    code            VARCHAR(100) UNIQUE,
    seen_at         TIMESTAMP,

    CONSTRAINT fk_penerimaan_afirmasi_ver
        FOREIGN KEY (verification_id)
        REFERENCES verification_afirmasi(id)
        ON UPDATE CASCADE
        ON DELETE CASCADE,

    CONSTRAINT fk_penerimaan_afirmasi_kepsek
        FOREIGN KEY (kepsek_id)
        REFERENCES office_users(id)
        ON UPDATE CASCADE
        ON DELETE SET NULL
);

CREATE TABLE penerimaan_mutasi (
    id              SERIAL PRIMARY KEY,
    created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    verification_id INT NOT NULL UNIQUE,
    code            VARCHAR(100) UNIQUE,
    seen_at         TIMESTAMP,

    CONSTRAINT fk_penerimaan_mutasi_ver
        FOREIGN KEY (verification_id)
        REFERENCES verification_mutasi(id)
        ON UPDATE CASCADE
        ON DELETE CASCADE
);

CREATE TABLE penerimaan_prestasi_mandiri (
    id              SERIAL PRIMARY KEY,
    npsn            VARCHAR(20) NOT NULL,
    verification_id INT NOT NULL UNIQUE,
    code            VARCHAR(100) UNIQUE,
    seen_at         TIMESTAMP,

    CONSTRAINT fk_penerimaan_prestasi_school
        FOREIGN KEY (npsn)
        REFERENCES schools(npsn)
        ON UPDATE CASCADE
        ON DELETE RESTRICT,

    CONSTRAINT fk_penerimaan_prestasi_ver
        FOREIGN KEY (verification_id)
        REFERENCES verification_prestasi_mandiri(id)
        ON UPDATE CASCADE
        ON DELETE CASCADE
);

CREATE TABLE penerimaan_zonasi (
    id              SERIAL PRIMARY KEY,
    created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    verification_id INT NOT NULL UNIQUE,
    code            VARCHAR(100) UNIQUE,
    seen_at         TIMESTAMP,

    CONSTRAINT fk_penerimaan_zonasi_ver
        FOREIGN KEY (verification_id)
        REFERENCES verification_zonasi(id)
        ON UPDATE CASCADE
        ON DELETE CASCADE
);

-- ============================================================
-- INDEX UNTUK OPTIMASI QUERY
-- ============================================================

-- Kelompok A
CREATE INDEX idx_office_users_role_id
    ON office_users(role_id);
CREATE INDEX idx_school_users_office_user_username
    ON school_users(office_user_username);
CREATE INDEX idx_school_users_school_npsn
    ON school_users(school_npsn);
CREATE INDEX idx_users_npsn
    ON users(npsn);
CREATE INDEX idx_users_school_destination_id
    ON users(school_destination_id);

-- Kelompok B
CREATE INDEX idx_reg_afirmasi_user_id
    ON registrations_afirmasi(user_id);
CREATE INDEX idx_reg_afirmasi_school_destination_id
    ON registrations_afirmasi(school_destination_id);
CREATE INDEX idx_reg_mutasi_user_id
    ON registrations_mutasi(user_id);
CREATE INDEX idx_reg_mutasi_school_destination_id
    ON registrations_mutasi(school_destination_id);
CREATE INDEX idx_reg_prestasi_user_id
    ON registrations_prestasi_mandiri(user_id);
CREATE INDEX idx_reg_prestasi_school_destination_id
    ON registrations_prestasi_mandiri(school_destination_id);
CREATE INDEX idx_reg_zonasi_user_id
    ON registrations_zonasi(user_id);
CREATE INDEX idx_reg_zonasi_school_destination_id
    ON registrations_zonasi(school_destination_id);

-- Kelompok C
CREATE INDEX idx_ver_afirmasi_operator_id
    ON verification_afirmasi(operator_id);
CREATE INDEX idx_ver_mutasi_operator_id
    ON verification_mutasi(operator_id);
CREATE INDEX idx_ver_prestasi_operator_id
    ON verification_prestasi_mandiri(operator_id);
CREATE INDEX idx_ver_zonasi_operator_id
    ON verification_zonasi(operator_id);

-- Kelompok D
CREATE INDEX idx_penerimaan_afirmasi_kepsek_id
    ON penerimaan_afirmasi(kepsek_id);
CREATE INDEX idx_penerimaan_prestasi_npsn
    ON penerimaan_prestasi_mandiri(npsn);
```
