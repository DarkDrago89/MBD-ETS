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

[3] Relasi Sekolah Asal Siswa
    JUNIOR_SCHOOLS ||──────────o{ USERS : "asal sekolah dari"
    (1 SMP menjadi asal dari 0 atau banyak calon siswa)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 KELOMPOK B: JALUR PENDAFTARAN
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

[4] Calon Siswa ke Registrasi (1 siswa bisa daftar di masing-masing jalur)
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

[6] Registrasi ke Verifikasi (1 registrasi menghasilkan 0 atau 1 verifikasi)
    REGISTRATIONS_AFIRMASI         ||──────────o| VERIFICATION_AFIRMASI         : "diverifikasi"
    REGISTRATIONS_MUTASI           ||──────────o| VERIFICATION_MUTASI           : "diverifikasi"
    REGISTRATIONS_PRESTASI_MANDIRI ||──────────o| VERIFICATION_PRESTASI_MANDIRI : "diverifikasi"
    REGISTRATIONS_ZONASI           ||──────────o| VERIFICATION_ZONASI           : "diverifikasi"

[7] Petugas ke Verifikasi (1 petugas memverifikasi 0 atau banyak pendaftaran)
    OFFICE_USERS ||──────────o{ VERIFICATION_AFIRMASI         : "memverifikasi"
    OFFICE_USERS ||──────────o{ VERIFICATION_MUTASI           : "memverifikasi"
    OFFICE_USERS ||──────────o{ VERIFICATION_PRESTASI_MANDIRI : "memverifikasi"
    OFFICE_USERS ||──────────o{ VERIFICATION_ZONASI           : "memverifikasi"

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 KELOMPOK D: PROSES PENERIMAAN
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

[8] Verifikasi ke Penerimaan (1 verifikasi menghasilkan 0 atau 1 penerimaan)
    VERIFICATION_AFIRMASI         ||──────────o| PENERIMAAN_AFIRMASI         : "menghasilkan"
    VERIFICATION_MUTASI           ||──────────o| PENERIMAAN_MUTASI           : "menghasilkan"
    VERIFICATION_PRESTASI_MANDIRI ||──────────o| PENERIMAAN_PRESTASI_MANDIRI : "menghasilkan"
    VERIFICATION_ZONASI           ||──────────o| PENERIMAAN_ZONASI           : "menghasilkan"

[9] Kepala Sekolah ke Penerimaan Afirmasi (khusus afirmasi ada kepsek_id)
    OFFICE_USERS ||──────────o{ PENERIMAAN_AFIRMASI : "ditetapkan oleh kepala sekolah"

[10] Sekolah ke Penerimaan Prestasi Mandiri (via NPSN)
    SCHOOLS ||──────────o{ PENERIMAAN_PRESTASI_MANDIRI : "tempat diterima"

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 ALUR PROSES LENGKAP (Flow Diagram)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  JUNIOR_SCHOOLS ──── (asal sekolah) ────► USERS ◄──── SCHOOLS
                                             │              │
                              ┌──────────────┼──────────────┘
                              │              │
                    ┌─────────▼─────────┐    │ (sekolah tujuan)
                    │   Memilih Jalur   │    │
                    └──────┬────────────┘    │
                           │                 │
          ┌────────────────┼─────────────────┼──────────────┐
          │                │                 │              │
          ▼                ▼                 ▼              ▼
    REG_AFIRMASI   REG_MUTASI        REG_PRESTASI    REG_ZONASI
          │                │                 │              │
          └────────────────┼─────────────────┼──────────────┘
                           │
                    ┌──────▼──────┐
                    │  VERIFIKASI  │ ◄─── OFFICE_USERS (operator)
                    │  (per jalur) │
                    └──────┬──────┘
                           │ (jika approve)
                    ┌──────▼──────┐
                    │  PENERIMAAN  │ ◄─── OFFICE_USERS (kepsek, khusus afirmasi)
                    │  (per jalur) │
                    └─────────────┘

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

┌─────────────────────────────────────────────────────────────┐
│ ROLES                                                       │
├──────────────┬──────────────────┬───────────────────────────┤
│ Kolom        │ Tipe Data        │ Keterangan                │
├──────────────┼──────────────────┼───────────────────────────┤
│ id           │ SERIAL           │ PRIMARY KEY               │
│ name         │ VARCHAR(100)     │ NOT NULL                  │
│ created_at   │ TIMESTAMP        │ DEFAULT CURRENT_TIMESTAMP │
│ updated_at   │ TIMESTAMP        │ DEFAULT CURRENT_TIMESTAMP │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│ OFFICE_USERS                                                │
├──────────────┬──────────────────┬───────────────────────────┤
│ Kolom        │ Tipe Data        │ Keterangan                │
├──────────────┼──────────────────┼───────────────────────────┤
│ id           │ SERIAL           │ PRIMARY KEY               │
│ name         │ VARCHAR(255)     │ NOT NULL                  │
│ username     │ VARCHAR(100)     │ UNIQUE NOT NULL           │
│ password     │ VARCHAR(255)     │ NOT NULL                  │
│ role_id      │ INT              │ FK → ROLES(id)            │
│ created_at   │ TIMESTAMP        │ DEFAULT CURRENT_TIMESTAMP │
│ updated_at   │ TIMESTAMP        │ DEFAULT CURRENT_TIMESTAMP │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│ SCHOOLS                                                     │
├──────────────────────────────┬──────────────┬───────────────┤
│ Kolom                        │ Tipe Data    │ Keterangan    │
├──────────────────────────────┼──────────────┼───────────────┤
│ id                           │ SERIAL       │ PRIMARY KEY   │
│ name                         │ VARCHAR(255) │ NOT NULL      │
│ kode                         │ VARCHAR(50)  │               │
│ npsn                         │ VARCHAR(20)  │ UNIQUE NOT NULL│
│ minimum_average              │ DECIMAL(5,2) │               │
│ city_id                      │ INT          │               │
│ created_at                   │ TIMESTAMP    │ DEFAULT NOW() │
│ updated_at                   │ TIMESTAMP    │ DEFAULT NOW() │
│ latitude                     │ DECIMAL(10,7)│               │
│ longitude                    │ DECIMAL(10,7)│               │
│ pagu_afirmasi                │ INT          │ DEFAULT 0     │
│ pagu_mutasi                  │ INT          │ DEFAULT 0     │
│ pagu_prestasi_undangan       │ INT          │ DEFAULT 0     │
│ pagu_zonasi                  │ INT          │ DEFAULT 0     │
│ pagu_prestasi_tesmandiri     │ INT          │ DEFAULT 0     │
│ pagu_tidak_naik_kelas        │ INT          │ DEFAULT 0     │
│ school_code                  │ VARCHAR(20)  │               │
│ kebijakan_sisa_pagu_afirmasi │ TEXT         │               │
│ kebijakan_sisa_pagu_mutasi   │ TEXT         │               │
│ kebijakan_sisa_pagu_undangan │ TEXT         │               │
│ kebijakan_sisa_pagu_zonasi   │ TEXT         │               │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│ SCHOOL_USERS                   [Tabel Junction Many-to-Many]│
├──────────────────────┬──────────────┬────────────────────────┤
│ Kolom                │ Tipe Data    │ Keterangan             │
├──────────────────────┼──────────────┼────────────────────────┤
│ id                   │ SERIAL       │ PRIMARY KEY            │
│ office_user_username │ VARCHAR(100) │ FK → OFFICE_USERS      │
│                      │              │    (username)          │
│ school_npsn          │ VARCHAR(20)  │ FK → SCHOOLS(npsn)     │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│ JUNIOR_SCHOOLS                                              │
├──────────────┬──────────────────┬───────────────────────────┤
│ Kolom        │ Tipe Data        │ Keterangan                │
├──────────────┼──────────────────┼───────────────────────────┤
│ id           │ SERIAL           │ PRIMARY KEY               │
│ npsn         │ VARCHAR(20)      │ UNIQUE NOT NULL           │
│ name         │ VARCHAR(255)     │ NOT NULL                  │
│ created_at   │ TIMESTAMP        │ DEFAULT CURRENT_TIMESTAMP │
│ updated_at   │ TIMESTAMP        │ DEFAULT CURRENT_TIMESTAMP │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│ USERS                              [Calon Peserta Didik]    │
├──────────────────────┬──────────────┬────────────────────────┤
│ Kolom                │ Tipe Data    │ Keterangan             │
├──────────────────────┼──────────────┼────────────────────────┤
│ id                   │ SERIAL       │ PRIMARY KEY            │
│ npsn                 │ VARCHAR(20)  │ FK → JUNIOR_SCHOOLS    │
│                      │              │    (npsn) [asal SMP]   │
│ nisn                 │ VARCHAR(20)  │ UNIQUE NOT NULL        │
│ birth_date           │ DATE         │                        │
│ name                 │ VARCHAR(255) │ NOT NULL               │
│ gender               │ VARCHAR(10)  │ CHECK ('L','P')        │
│ address              │ TEXT         │                        │
│ phone                │ VARCHAR(20)  │                        │
│ school_name          │ VARCHAR(255) │                        │
│ cluster              │ VARCHAR(100) │                        │
│ created_at           │ TIMESTAMP    │ DEFAULT NOW()          │
│ updated_at           │ TIMESTAMP    │ DEFAULT NOW()          │
│ school_destination_id│ INT          │ FK → SCHOOLS(id)       │
│ latitude             │ DECIMAL(10,7)│                        │
│ longitude            │ DECIMAL(10,7)│                        │
│ registration_1_type  │ VARCHAR(50)  │ Jenis jalur ke-1       │
│ registration_2_type  │ VARCHAR(50)  │ Jenis jalur ke-2       │
│ registration_3_type  │ VARCHAR(50)  │ Jenis jalur ke-3       │
│ registration_1_id    │ INT          │ ID registrasi ke-1     │
│ registration_2_id    │ INT          │ ID registrasi ke-2     │
│ registration_3_id    │ INT          │ ID registrasi ke-3     │
│ acceptance_type      │ VARCHAR(50)  │ Jalur yang diterima    │
│ acceptance_id        │ INT          │ ID penerimaan          │
│ deleted_at           │ TIMESTAMP    │ Soft delete            │
│ jalur_prestasi       │ VARCHAR(100) │ Jenis prestasi         │
└─────────────────────────────────────────────────────────────┘

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 KELOMPOK B: TABEL REGISTRASI (4 JALUR)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

┌─────────────────────────────────────────────────────────────┐
│ REGISTRATIONS_AFIRMASI                                      │
├──────────────────────┬──────────────┬────────────────────────┤
│ Kolom                │ Tipe Data    │ Keterangan             │
├──────────────────────┼──────────────┼────────────────────────┤
│ id                   │ SERIAL       │ PRIMARY KEY            │
│ user_id              │ INT          │ FK → USERS(id)         │
│ jenis                │ VARCHAR(100) │ Jenis afirmasi         │
│ school_destination_id│ INT          │ FK → SCHOOLS(id)       │
│ distance             │ DECIMAL(10,2)│ Jarak ke sekolah (km)  │
│ verification_schedule│ TIMESTAMP    │ Jadwal verifikasi      │
│ status               │ VARCHAR(50)  │ pending/verified/      │
│                      │              │ rejected/accepted      │
│ created_at           │ TIMESTAMP    │ DEFAULT NOW()          │
│ updated_at           │ TIMESTAMP    │ DEFAULT NOW()          │
│ documents            │ TEXT         │ Daftar dokumen         │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│ REGISTRATIONS_MUTASI                                        │
├──────────────────────┬──────────────┬────────────────────────┤
│ Kolom                │ Tipe Data    │ Keterangan             │
├──────────────────────┼──────────────┼────────────────────────┤
│ id                   │ SERIAL       │ PRIMARY KEY            │
│ user_id              │ INT          │ FK → USERS(id)         │
│ jenis                │ VARCHAR(100) │ Jenis mutasi           │
│ school_destination_id│ INT          │ FK → SCHOOLS(id)       │
│ distance             │ DECIMAL(10,2)│ Jarak ke sekolah (km)  │
│ verification_schedule│ TIMESTAMP    │ Jadwal verifikasi      │
│ status               │ VARCHAR(50)  │ pending/verified/      │
│                      │              │ rejected/accepted      │
│ created_at           │ TIMESTAMP    │ DEFAULT NOW()          │
│ updated_at           │ TIMESTAMP    │ DEFAULT NOW()          │
│ documents            │ TEXT         │ Daftar dokumen         │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│ REGISTRATIONS_PRESTASI_MANDIRI                              │
├──────────────────────┬──────────────┬────────────────────────┤
│ Kolom                │ Tipe Data    │ Keterangan             │
├──────────────────────┼──────────────┼────────────────────────┤
│ id                   │ SERIAL       │ PRIMARY KEY            │
│ user_id              │ INT          │ FK → USERS(id)         │
│ data                 │ TEXT         │ Data prestasi          │
│ school_destination_id│ INT          │ FK → SCHOOLS(id)       │
│ distance             │ DECIMAL(10,2)│ Jarak ke sekolah (km)  │
│ verification_schedule│ TIMESTAMP    │ Jadwal verifikasi      │
│ status               │ VARCHAR(50)  │ pending/verified/      │
│                      │              │ rejected/accepted      │
│ created_at           │ TIMESTAMP    │ DEFAULT NOW()          │
│ updated_at           │ TIMESTAMP    │ DEFAULT NOW()          │
│ documents            │ TEXT         │ Daftar dokumen         │
│ test_number          │ VARCHAR(50)  │ Nomor ujian            │
│ final_score          │ DECIMAL(5,2) │ Nilai akhir            │
│ test_score           │ DECIMAL(5,2) │ Nilai ujian            │
│ jurusan              │ VARCHAR(100) │ IPA / IPS              │
│ ruang_tes            │ VARCHAR(50)  │ Ruang ujian            │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│ REGISTRATIONS_ZONASI                                        │
├──────────────────────┬──────────────┬────────────────────────┤
│ Kolom                │ Tipe Data    │ Keterangan             │
├──────────────────────┼──────────────┼────────────────────────┤
│ id                   │ SERIAL       │ PRIMARY KEY            │
│ user_id              │ INT          │ FK → USERS(id)         │
│ school_destination_id│ INT          │ FK → SCHOOLS(id)       │
│ distance             │ DECIMAL(10,2)│ Jarak ke sekolah (km)  │
│ verification_schedule│ TIMESTAMP    │ Jadwal verifikasi      │
│ status               │ VARCHAR(50)  │ pending/verified/      │
│                      │              │ rejected/accepted      │
│ created_at           │ TIMESTAMP    │ DEFAULT NOW()          │
│ updated_at           │ TIMESTAMP    │ DEFAULT NOW()          │
│ documents            │ TEXT         │ Daftar dokumen         │
│ kk_date              │ DATE         │ Tanggal terbit KK      │
└─────────────────────────────────────────────────────────────┘

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 KELOMPOK C: TABEL VERIFIKASI (4 JALUR)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

┌─────────────────────────────────────────────────────────────┐
│ VERIFICATION_AFIRMASI                                       │
├──────────────────┬──────────────────┬───────────────────────┤
│ Kolom            │ Tipe Data        │ Keterangan            │
├──────────────────┼──────────────────┼───────────────────────┤
│ id               │ SERIAL           │ PRIMARY KEY           │
│ registration_id  │ INT              │ FK →                  │
│                  │                  │ REGISTRATIONS_        │
│                  │                  │ AFIRMASI(id)          │
│ operator_id      │ INT              │ FK → OFFICE_USERS(id) │
│ action           │ VARCHAR(50)      │ approve/reject/pending│
│ created_at       │ TIMESTAMP        │ DEFAULT NOW()         │
│ updated_at       │ TIMESTAMP        │ DEFAULT NOW()         │
│ alasan_batal     │ TEXT             │ Alasan jika ditolak   │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│ VERIFICATION_MUTASI                                         │
├──────────────────┬──────────────────┬───────────────────────┤
│ Kolom            │ Tipe Data        │ Keterangan            │
├──────────────────┼──────────────────┼───────────────────────┤
│ id               │ SERIAL           │ PRIMARY KEY           │
│ registration_id  │ INT              │ FK →                  │
│                  │                  │ REGISTRATIONS_        │
│                  │                  │ MUTASI(id)            │
│ operator_id      │ INT              │ FK → OFFICE_USERS(id) │
│ action           │ VARCHAR(50)      │ approve/reject/pending│
│ created_at       │ TIMESTAMP        │ DEFAULT NOW()         │
│ updated_at       │ TIMESTAMP        │ DEFAULT NOW()         │
│ alasan_batal     │ TEXT             │ Alasan jika ditolak   │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│ VERIFICATION_PRESTASI_MANDIRI                               │
├──────────────────┬──────────────────┬───────────────────────┤
│ Kolom            │ Tipe Data        │ Keterangan            │
├──────────────────┼──────────────────┼───────────────────────┤
│ id               │ SERIAL           │ PRIMARY KEY           │
│ registration_id  │ INT              │ FK →                  │
│                  │                  │ REGISTRATIONS_        │
│                  │                  │ PRESTASI_MANDIRI(id)  │
│ operator_id      │ INT              │ FK → OFFICE_USERS(id) │
│ action           │ VARCHAR(50)      │ approve/reject/pending│
│ created_at       │ TIMESTAMP        │ DEFAULT NOW()         │
│ updated_at       │ TIMESTAMP        │ DEFAULT NOW()         │
│ alasan_batal     │ TEXT             │ Alasan jika ditolak   │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│ VERIFICATION_ZONASI                                         │
├──────────────────┬──────────────────┬───────────────────────┤
│ Kolom            │ Tipe Data        │ Keterangan            │
├──────────────────┼──────────────────┼───────────────────────┤
│ id               │ SERIAL           │ PRIMARY KEY           │
│ registration_id  │ INT              │ FK →                  │
│                  │                  │ REGISTRATIONS_        │
│                  │                  │ ZONASI(id)            │
│ operator_id      │ INT              │ FK → OFFICE_USERS(id) │
│ action           │ VARCHAR(50)      │ approve/reject/pending│
│ created_at       │ TIMESTAMP        │ DEFAULT NOW()         │
│ updated_at       │ TIMESTAMP        │ DEFAULT NOW()         │
│ alasan_batal     │ TEXT             │ Alasan jika ditolak   │
└─────────────────────────────────────────────────────────────┘

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 KELOMPOK D: TABEL PENERIMAAN (4 JALUR)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

┌─────────────────────────────────────────────────────────────┐
│ PENERIMAAN_AFIRMASI                                         │
├──────────────────┬──────────────────┬───────────────────────┤
│ Kolom            │ Tipe Data        │ Keterangan            │
├──────────────────┼──────────────────┼───────────────────────┤
│ id               │ SERIAL           │ PRIMARY KEY           │
│ verification_id  │ INT              │ FK →                  │
│                  │                  │ VERIFICATION_         │
│                  │                  │ AFIRMASI(id)          │
│ kepsek_id        │ INT              │ FK → OFFICE_USERS(id) │
│                  │                  │ [Kepala Sekolah]      │
│ created_at       │ TIMESTAMP        │ DEFAULT NOW()         │
│ updated_at       │ TIMESTAMP        │ DEFAULT NOW()         │
│ code             │ VARCHAR(100)     │ UNIQUE, Kode terima   │
│ seen_at          │ TIMESTAMP        │ Waktu dilihat siswa   │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│ PENERIMAAN_MUTASI                                           │
├──────────────────┬──────────────────┬───────────────────────┤
│ Kolom            │ Tipe Data        │ Keterangan            │
├──────────────────┼──────────────────┼───────────────────────┤
│ id               │ SERIAL           │ PRIMARY KEY           │
│ created_at       │ TIMESTAMP        │ DEFAULT NOW()         │
│ updated_at       │ TIMESTAMP        │ DEFAULT NOW()         │
│ verification_id  │ INT              │ FK →                  │
│                  │                  │ VERIFICATION_         │
│                  │                  │ MUTASI(id)            │
│ code             │ VARCHAR(100)     │ UNIQUE, Kode terima   │
│ seen_at          │ TIMESTAMP        │ Waktu dilihat siswa   │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│ PENERIMAAN_PRESTASI_MANDIRI                                 │
├──────────────────┬──────────────────┬───────────────────────┤
│ Kolom            │ Tipe Data        │ Keterangan            │
├──────────────────┼──────────────────┼───────────────────────┤
│ id               │ SERIAL           │ PRIMARY KEY           │
│ npsn             │ VARCHAR(20)      │ FK → SCHOOLS(npsn)    │
│                  │                  │ [Sekolah penerima]    │
│ verification_id  │ INT              │ FK →                  │
│                  │                  │ VERIFICATION_PRESTASI │
│                  │                  │ _MANDIRI(id)          │
│ code             │ VARCHAR(100)     │ UNIQUE, Kode terima   │
│ seen_at          │ TIMESTAMP        │ Waktu dilihat siswa   │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│ PENERIMAAN_ZONASI                                           │
├──────────────────┬──────────────────┬───────────────────────┤
│ Kolom            │ Tipe Data        │ Keterangan            │
├──────────────────┼──────────────────┼───────────────────────┤
│ id               │ SERIAL           │ PRIMARY KEY           │
│ created_at       │ TIMESTAMP        │ DEFAULT NOW()         │
│ updated_at       │ TIMESTAMP        │ DEFAULT NOW()         │
│ verification_id  │ INT              │ FK →                  │
│                  │                  │ VERIFICATION_         │
│                  │                  │ ZONASI(id)            │
│ code             │ VARCHAR(100)     │ UNIQUE, Kode terima   │
│ seen_at          │ TIMESTAMP        │ Waktu dilihat siswa   │
└─────────────────────────────────────────────────────────────┘

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 RINGKASAN FOREIGN KEY SELURUH DATABASE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  Tabel Asal                       Kolom FK               → Tabel Tujuan
  ─────────────────────────────────────────────────────────────────────
  OFFICE_USERS                     role_id                → ROLES(id)
  SCHOOL_USERS                     office_user_username   → OFFICE_USERS(username)
  SCHOOL_USERS                     school_npsn            → SCHOOLS(npsn)
  USERS                            npsn                   → JUNIOR_SCHOOLS(npsn)
  USERS                            school_destination_id  → SCHOOLS(id)
  REGISTRATIONS_AFIRMASI           user_id                → USERS(id)
  REGISTRATIONS_AFIRMASI           school_destination_id  → SCHOOLS(id)
  REGISTRATIONS_MUTASI             user_id                → USERS(id)
  REGISTRATIONS_MUTASI             school_destination_id  → SCHOOLS(id)
  REGISTRATIONS_PRESTASI_MANDIRI   user_id                → USERS(id)
  REGISTRATIONS_PRESTASI_MANDIRI   school_destination_id  → SCHOOLS(id)
  REGISTRATIONS_ZONASI             user_id                → USERS(id)
  REGISTRATIONS_ZONASI             school_destination_id  → SCHOOLS(id)
  VERIFICATION_AFIRMASI            registration_id        → REGISTRATIONS_AFIRMASI(id)
  VERIFICATION_AFIRMASI            operator_id            → OFFICE_USERS(id)
  VERIFICATION_MUTASI              registration_id        → REGISTRATIONS_MUTASI(id)
  VERIFICATION_MUTASI              operator_id            → OFFICE_USERS(id)
  VERIFICATION_PRESTASI_MANDIRI    registration_id        → REGISTRATIONS_PRESTASI_MANDIRI(id)
  VERIFICATION_PRESTASI_MANDIRI    operator_id            → OFFICE_USERS(id)
  VERIFICATION_ZONASI              registration_id        → REGISTRATIONS_ZONASI(id)
  VERIFICATION_ZONASI              operator_id            → OFFICE_USERS(id)
  PENERIMAAN_AFIRMASI              verification_id        → VERIFICATION_AFIRMASI(id)
  PENERIMAAN_AFIRMASI              kepsek_id              → OFFICE_USERS(id)
  PENERIMAAN_MUTASI                verification_id        → VERIFICATION_MUTASI(id)
  PENERIMAAN_PRESTASI_MANDIRI      npsn                   → SCHOOLS(npsn)
  PENERIMAAN_PRESTASI_MANDIRI      verification_id        → VERIFICATION_PRESTASI_MANDIRI(id)
  PENERIMAAN_ZONASI                verification_id        → VERIFICATION_ZONASI(id)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 KOREKSI DARI SKEMA AWAL
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  No  Tabel                        Koreksi
  ─────────────────────────────────────────────────────────────────
  1   USERS.npsn                   FK → JUNIOR_SCHOOLS(npsn)
                                   bukan teks biasa
  2   PENERIMAAN_AFIRMASI          kepsek_id hanya ada di jalur
                                   afirmasi, jalur lain TIDAK ada
  3   PENERIMAAN_PRESTASI_MANDIRI  Memiliki kolom npsn sebagai FK
                                   → SCHOOLS(npsn), berbeda dengan
                                   jalur lain yang tidak punya
  4   PENERIMAAN_PRESTASI_MANDIRI  Tidak memiliki created_at dan
                                   updated_at (berbeda dgn jalur lain)
  5   ERD [9] sebelumnya           Relasi "ditetapkan oleh" hanya
                                   berlaku untuk PENERIMAAN_AFIRMASI
                                   (kepsek_id), bukan semua jalur
```


# Letak kesalahan

# Daftar Koreksi dari Skema Awal

---

## Koreksi pada ERD (1.1)

### ❌ Salah (versi awal)
```
9. Relasi Petugas ke Penerimaan
OFFICE_USER ||--o{ PENERIMAAN_AFIRMASI         : "ditetapkan oleh"
OFFICE_USER ||--o{ PENERIMAAN_MUTASI           : "ditetapkan oleh"
OFFICE_USER ||--o{ PENERIMAAN_PRESTASI_MANDIRI : "ditetapkan oleh"
OFFICE_USER ||--o{ PENERIMAAN_ZONASI           : "ditetapkan oleh"
```

### ✅ Benar (versi koreksi)
```
[9] Kepala Sekolah ke Penerimaan Afirmasi (khusus afirmasi ada kepsek_id)
    OFFICE_USERS ||──────────o{ PENERIMAAN_AFIRMASI : "ditetapkan oleh kepala sekolah"

[10] Sekolah ke Penerimaan Prestasi Mandiri (via NPSN)
    SCHOOLS ||──────────o{ PENERIMAAN_PRESTASI_MANDIRI : "tempat diterima"
```

**Alasan:** Kolom `kepsek_id` hanya ada di tabel `PENERIMAAN_AFIRMASI` saja. Tabel penerimaan lainnya (`mutasi`, `prestasi_mandiri`, `zonasi`) **tidak memiliki** kolom `kepsek_id`. Dan khusus `PENERIMAAN_PRESTASI_MANDIRI` memiliki relasi ke `SCHOOLS` via kolom `npsn`, bukan ke `OFFICE_USERS`.

---

## Koreksi pada Skema Relasional (1.2)

### Koreksi 1 — Tabel `USERS`

#### ❌ Salah (versi awal)
```
USERS (
    npsn   -- tidak dijelaskan sebagai FK
    ...
)
```

#### ✅ Benar (versi koreksi)
```
USERS (
    npsn  VARCHAR(20)  FK → JUNIOR_SCHOOLS(npsn)
    ...
)
```

**Alasan:** Kolom `npsn` di tabel `USERS` merujuk ke sekolah asal siswa (SMP), sehingga harus menjadi **Foreign Key** ke tabel `JUNIOR_SCHOOLS(npsn)`.

---

### Koreksi 2 — Tabel `PENERIMAAN_AFIRMASI`

#### ❌ Salah (versi awal)
```
PENERIMAAN_AFIRMASI (
    kepsek_id  -- tidak dijelaskan konteksnya
)
```

#### ✅ Benar (versi koreksi)
```
PENERIMAAN_AFIRMASI (
    kepsek_id  INT  FK → OFFICE_USERS(id)  [Kepala Sekolah]
)
```

**Alasan:** Perlu ditegaskan bahwa `kepsek_id` adalah FK ke `OFFICE_USERS` dan **hanya ada di jalur afirmasi**, tidak di jalur lain.

---

### Koreksi 3 — Tabel `PENERIMAAN_PRESTASI_MANDIRI`

#### ❌ Salah (versi awal)
```
PENERIMAAN_PRESTASI_MANDIRI (
    id PK,
    npsn,          -- tidak dijelaskan sebagai FK
    verification_id FK -> VERIFICATION_PRESTASI_MANDIRI.id,
    code,
    seen_at
)
```

#### ✅ Benar (versi koreksi)
```
PENERIMAAN_PRESTASI_MANDIRI (
    id               SERIAL        PRIMARY KEY
    npsn             VARCHAR(20)   FK → SCHOOLS(npsn)   ← ditambahkan
    verification_id  INT           FK → VERIFICATION_PRESTASI_MANDIRI(id)
    code             VARCHAR(100)  UNIQUE
    seen_at          TIMESTAMP
    -- TIDAK ADA created_at dan updated_at              ← ditegaskan
)
```

**Alasan:**
- Kolom `npsn` harus dijelaskan sebagai **FK ke `SCHOOLS(npsn)`**
- Tabel ini **tidak memiliki** `created_at` dan `updated_at`, berbeda dengan tabel penerimaan lainnya

---

### Koreksi 4 — Perbedaan Struktur Antar Tabel Penerimaan

#### ❌ Salah (versi awal)
```
-- Tidak dijelaskan perbedaan struktur antar tabel penerimaan
```

#### ✅ Benar (versi koreksi)

| Kolom | PENERIMAAN_AFIRMASI | PENERIMAAN_MUTASI | PENERIMAAN_PRESTASI_MANDIRI | PENERIMAAN_ZONASI |
|---|---|---|---|---|
| `id` | ✅ | ✅ | ✅ | ✅ |
| `verification_id` | ✅ | ✅ | ✅ | ✅ |
| `code` | ✅ | ✅ | ✅ | ✅ |
| `seen_at` | ✅ | ✅ | ✅ | ✅ |
| `created_at` | ✅ | ✅ | ❌ Tidak ada | ✅ |
| `updated_at` | ✅ | ✅ | ❌ Tidak ada | ✅ |
| `kepsek_id` | ✅ Khusus | ❌ | ❌ | ❌ |
| `npsn` | ❌ | ❌ | ✅ Khusus | ❌ |

---

## Ringkasan Semua Koreksi

```
No  Bagian            Yang Dikoreksi
────────────────────────────────────────────────────────────────
1   ERD Relasi [9]    Relasi kepsek_id hanya untuk
                      PENERIMAAN_AFIRMASI, bukan semua jalur

2   ERD Relasi [10]   Ditambahkan relasi baru:
                      SCHOOLS → PENERIMAAN_PRESTASI_MANDIRI
                      via kolom npsn

3   USERS.npsn        Ditegaskan sebagai FK →
                      JUNIOR_SCHOOLS(npsn)

4   PENERIMAAN_       Ditegaskan kepsek_id adalah FK →
    AFIRMASI          OFFICE_USERS(id) dan hanya ada
                      di jalur ini

5   PENERIMAAN_       npsn ditegaskan sebagai FK →
    PRESTASI_MANDIRI  SCHOOLS(npsn)

6   PENERIMAAN_       Ditegaskan TIDAK memiliki
    PRESTASI_MANDIRI  created_at dan updated_at
```
