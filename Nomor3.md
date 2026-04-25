# BAGIAN 3: INVESTIGASI PENYEBAB MASALAH

## 3.1 Analisis Penyebab Masalah PPDB

```
╔══════════════════════════════════════════════════════════════════╗
║              INVESTIGASI PENYEBAB MASALAH PPDB                  ║
╚══════════════════════════════════════════════════════════════════╝

Masalah yang terjadi:
  1. Nama siswa yang sudah lolos seleksi di website
     tiba-tiba hilang saat daftar ulang di sekolah
  2. Jumlah siswa yang lolos berkurang dari 130 → 100
     saat proses daftar ulang

Dugaan penyebab:
  A. Race Condition  → Dua proses menulis data secara
                       bersamaan tanpa locking
  B. Inkonsistensi   → Tidak ada UNIQUE constraint pada
     Transaksi         verification_id di tabel penerimaan
  C. Pagu Terlampaui → Tidak ada pengecekan kapasitas
                       sebelum insert penerimaan
  D. Data Tumpang    → Satu verifikasi menghasilkan lebih
     Tindih            dari satu record penerimaan
```

### 3.2 Query Investigasi 1: Deteksi Duplikasi Penerimaan

```sql
-- ============================================================
-- INVESTIGASI 1:
-- Deteksi verification_id yang muncul lebih dari 1 kali
-- di tabel penerimaan (bukti race condition / bug insert)
-- ============================================================

-- Duplikasi di jalur Afirmasi
SELECT
    va.id                       AS verification_id,
    ra.id                       AS registration_id,
    u.name                      AS nama_siswa,
    u.nisn,
    s.name                      AS sekolah_tujuan,
    COUNT(pa.id)                AS jumlah_penerimaan,
    STRING_AGG(pa.code, ', ')   AS kode_penerimaan,
    'Afirmasi'                  AS jalur
FROM verification_afirmasi va
JOIN registrations_afirmasi ra ON ra.id = va.registration_id
JOIN users u                   ON u.id  = ra.user_id
JOIN schools s                 ON s.id  = ra.school_destination_id
JOIN penerimaan_afirmasi pa    ON pa.verification_id = va.id
GROUP BY
    va.id, ra.id, u.name, u.nisn, s.name
HAVING COUNT(pa.id) > 1

UNION ALL

-- Duplikasi di jalur Mutasi
SELECT
    vm.id,
    rm.id,
    u.name,
    u.nisn,
    s.name,
    COUNT(pm.id),
    STRING_AGG(pm.code, ', '),
    'Mutasi'
FROM verification_mutasi vm
JOIN registrations_mutasi rm ON rm.id = vm.registration_id
JOIN users u                 ON u.id  = rm.user_id
JOIN schools s               ON s.id  = rm.school_destination_id
JOIN penerimaan_mutasi pm    ON pm.verification_id = vm.id
GROUP BY
    vm.id, rm.id, u.name, u.nisn, s.name
HAVING COUNT(pm.id) > 1

UNION ALL

-- Duplikasi di jalur Prestasi Mandiri
SELECT
    vp.id,
    rp.id,
    u.name,
    u.nisn,
    s.name,
    COUNT(pp.id),
    STRING_AGG(pp.code, ', '),
    'Prestasi Mandiri'
FROM verification_prestasi_mandiri vp
JOIN registrations_prestasi_mandiri rp ON rp.id = vp.registration_id
JOIN users u                           ON u.id  = rp.user_id
JOIN schools s                         ON s.id  = rp.school_destination_id
JOIN penerimaan_prestasi_mandiri pp    ON pp.verification_id = vp.id
GROUP BY
    vp.id, rp.id, u.name, u.nisn, s.name
HAVING COUNT(pp.id) > 1

UNION ALL

-- Duplikasi di jalur Zonasi
SELECT
    vz.id,
    rz.id,
    u.name,
    u.nisn,
    s.name,
    COUNT(pz.id),
    STRING_AGG(pz.code, ', '),
    'Zonasi'
FROM verification_zonasi vz
JOIN registrations_zonasi rz ON rz.id = vz.registration_id
JOIN users u                 ON u.id  = rz.user_id
JOIN schools s               ON s.id  = rz.school_destination_id
JOIN penerimaan_zonasi pz    ON pz.verification_id = vz.id
GROUP BY
    vz.id, rz.id, u.name, u.nisn, s.name
HAVING COUNT(pz.id) > 1

ORDER BY jalur, jumlah_penerimaan DESC;
```

### 3.3 Query Investigasi 2: Deteksi Data Hilang

```sql
-- ============================================================
-- INVESTIGASI 2:
-- Deteksi siswa yang status-nya 'accepted' di registrasi
-- namun TIDAK ADA record penerimaan yang sesuai
-- (nama hilang saat daftar ulang)
-- ============================================================

SELECT
    u.id            AS user_id,
    u.name          AS nama_siswa,
    u.nisn,
    u.phone,
    s.name          AS sekolah_tujuan,
    ra.status       AS status_registrasi,
    va.action       AS aksi_verifikasi,
    pa.id           AS penerimaan_id,
    'Afirmasi'      AS jalur,
    CASE
        WHEN va.id IS NULL
            THEN 'Belum diverifikasi'
        WHEN va.action != 'approve'
            THEN 'Verifikasi tidak approve'
        WHEN pa.id IS NULL
            THEN 'MASALAH: Lolos tapi tidak ada penerimaan'
        ELSE 'Normal'
    END             AS keterangan
FROM registrations_afirmasi ra
JOIN users u    ON u.id  = ra.user_id
JOIN schools s  ON s.id  = ra.school_destination_id
LEFT JOIN verification_afirmasi va  ON va.registration_id = ra.id
LEFT JOIN penerimaan_afirmasi pa    ON pa.verification_id = va.id
WHERE ra.status = 'accepted'
  AND (va.id IS NULL OR pa.id IS NULL)

UNION ALL

SELECT
    u.id, u.name, u.nisn, u.phone,
    s.name, rm.status, vm.action, pm.id,
    'Mutasi',
    CASE
        WHEN vm.id IS NULL  THEN 'Belum diverifikasi'
        WHEN vm.action != 'approve' THEN 'Verifikasi tidak approve'
        WHEN pm.id IS NULL  THEN 'MASALAH: Lolos tapi tidak ada penerimaan'
        ELSE 'Normal'
    END
FROM registrations_mutasi rm
JOIN users u    ON u.id  = rm.user_id
JOIN schools s  ON s.id  = rm.school_destination_id
LEFT JOIN verification_mutasi vm ON vm.registration_id = rm.id
LEFT JOIN penerimaan_mutasi pm   ON pm.verification_id = vm.id
WHERE rm.status = 'accepted'
  AND (vm.id IS NULL OR pm.id IS NULL)

UNION ALL

SELECT
    u.id, u.name, u.nisn, u.phone,
    s.name, rp.status, vp.action, pp.id,
    'Prestasi Mandiri',
    CASE
        WHEN vp.id IS NULL  THEN 'Belum diverifikasi'
        WHEN vp.action != 'approve' THEN 'Verifikasi tidak approve'
        WHEN pp.id IS NULL  THEN 'MASALAH: Lolos tapi tidak ada penerimaan'
        ELSE 'Normal'
    END
FROM registrations_prestasi_mandiri rp
JOIN users u    ON u.id  = rp.user_id
JOIN schools s  ON s.id  = rp.school_destination_id
LEFT JOIN verification_prestasi_mandiri vp ON vp.registration_id = rp.id
LEFT JOIN penerimaan_prestasi_mandiri pp   ON pp.verification_id = vp.id
WHERE rp.status = 'accepted'
  AND (vp.id IS NULL OR pp.id IS NULL)

UNION ALL

SELECT
    u.id, u.name, u.nisn, u.phone,
    s.name, rz.status, vz.action, pz.id,
    'Zonasi',
    CASE
        WHEN vz.id IS NULL  THEN 'Belum diverifikasi'
        WHEN vz.action != 'approve' THEN 'Verifikasi tidak approve'
        WHEN pz.id IS NULL  THEN 'MASALAH: Lolos tapi tidak ada penerimaan'
        ELSE 'Normal'
    END
FROM registrations_zonasi rz
JOIN users u    ON u.id  = rz.user_id
JOIN schools s  ON s.id  = rz.school_destination_id
LEFT JOIN verification_zonasi vz ON vz.registration_id = rz.id
LEFT JOIN penerimaan_zonasi pz   ON pz.verification_id = vz.id
WHERE rz.status = 'accepted'
  AND (vz.id IS NULL OR pz.id IS NULL)

ORDER BY jalur, nama_siswa;
```

### 3.4 Query Investigasi 3: Deteksi Kapasitas Terlampaui

```sql
-- ============================================================
-- INVESTIGASI 3:
-- Deteksi sekolah yang jumlah penerimaan melebihi pagu
-- ============================================================

WITH rekapitulasi_penerimaan AS (
    -- Hitung penerimaan per sekolah per jalur
    SELECT
        s.id                AS school_id,
        s.name              AS nama_sekolah,
        s.npsn,
        s.pagu_afirmasi,
        s.pagu_mutasi,
        s.pagu_prestasi_tesmandiri,
        s.pagu_zonasi,
        (s.pagu_afirmasi + s.pagu_mutasi +
         s.pagu_prestasi_tesmandiri + s.pagu_zonasi)
                            AS total_pagu,

        -- Hitung per jalur
        COUNT(DISTINCT pa.id) AS diterima_afirmasi,
        COUNT(DISTINCT pm.id) AS diterima_mutasi,
        COUNT(DISTINCT pp.id) AS diterima_prestasi,
        COUNT(DISTINCT pz.id) AS diterima_zonasi

    FROM schools s

    -- Afirmasi
    LEFT JOIN registrations_afirmasi ra
           ON ra.school_destination_id = s.id
    LEFT JOIN verification_afirmasi va
           ON va.registration_id = ra.id
          AND va.action = 'approve'
    LEFT JOIN penerimaan_afirmasi pa
           ON pa.verification_id = va.id

    -- Mutasi
    LEFT JOIN registrations_mutasi rm
           ON rm.school_destination_id = s.id
    LEFT JOIN verification_mutasi vm
           ON vm.registration_id = rm.id
          AND vm.action = 'approve'
    LEFT JOIN penerimaan_mutasi pm
           ON pm.verification_id = vm.id

    -- Prestasi Mandiri
    LEFT JOIN penerimaan_prestasi_mandiri pp
           ON pp.npsn = s.npsn
    LEFT JOIN verification_prestasi_mandiri vp
           ON vp.id = pp.verification_id
          AND vp.action = 'approve'

    -- Zonasi
    LEFT JOIN registrations_zonasi rz
           ON rz.school_destination_id = s.id
    LEFT JOIN verification_zonasi vz
           ON vz.registration_id = rz.id
          AND vz.action = 'approve'
    LEFT JOIN penerimaan_zonasi pz
           ON pz.verification_id = vz.id

    GROUP BY
        s.id, s.name, s.npsn,
        s.pagu_afirmasi, s.pagu_mutasi,
        s.pagu_prestasi_tesmandiri, s.pagu_zonasi
)
SELECT
    school_id,
    nama_sekolah,
    npsn,
    pagu_afirmasi,
    pagu_mutasi,
    pagu_prestasi_tesmandiri,
    pagu_zonasi,
    total_pagu,
    diterima_afirmasi,
    diterima_mutasi,
    diterima_prestasi,
    diterima_zonasi,
    (diterima_afirmasi + diterima_mutasi +
     diterima_prestasi + diterima_zonasi)   AS total_diterima,
    -- Selisih per jalur
    (diterima_afirmasi - pagu_afirmasi)     AS selisih_afirmasi,
    (diterima_mutasi   - pagu_mutasi)       AS selisih_mutasi,
    (diterima_prestasi - pagu_prestasi_tesmandiri) AS selisih_prestasi,
    (diterima_zonasi   - pagu_zonasi)       AS selisih_zonasi,
    CASE
        WHEN (diterima_afirmasi + diterima_mutasi +
              diterima_prestasi + diterima_zonasi) > total_pagu
        THEN '⚠ MELEBIHI PAGU!'
        WHEN (diterima_afirmasi + diterima_mutasi +
              diterima_prestasi + diterima_zonasi) = total_pagu
        THEN '✓ Tepat pagu'
        ELSE '✓ Dalam batas pagu'
    END                                     AS status_pagu
FROM rekapitulasi_penerimaan
WHERE (diterima_afirmasi + diterima_mutasi +
       diterima_prestasi + diterima_zonasi) > 0
ORDER BY
    status_pagu DESC,
    total_diterima DESC;
```

### 3.5 Query Investigasi 4: Deteksi Siswa Diterima di Lebih dari Satu Jalur

```sql
-- ============================================================
-- INVESTIGASI 4:
-- Deteksi siswa yang diterima di lebih dari 1 jalur
-- sekaligus (seharusnya tidak boleh terjadi)
-- ============================================================

WITH semua_penerimaan AS (
    -- Kumpulkan semua siswa yang punya record penerimaan
    SELECT
        u.id            AS user_id,
        u.name          AS nama_siswa,
        u.nisn,
        s.name          AS sekolah_tujuan,
        'Afirmasi'      AS jalur,
        pa.code         AS kode_penerimaan,
        pa.created_at   AS waktu_penerimaan
    FROM penerimaan_afirmasi pa
    JOIN verification_afirmasi va  ON va.id  = pa.verification_id
    JOIN registrations_afirmasi ra ON ra.id  = va.registration_id
    JOIN users u                   ON u.id   = ra.user_id
    JOIN schools s                 ON s.id   = ra.school_destination_id

    UNION ALL

    SELECT
        u.id, u.name, u.nisn, s.name,
        'Mutasi', pm.code, pm.created_at
    FROM penerimaan_mutasi pm
    JOIN verification_mutasi vm  ON vm.id  = pm.verification_id
    JOIN registrations_mutasi rm ON rm.id  = vm.registration_id
    JOIN users u                 ON u.id   = rm.user_id
    JOIN schools s               ON s.id   = rm.school_destination_id

    UNION ALL

    SELECT
        u.id, u.name, u.nisn, s.name,
        'Prestasi Mandiri', pp.code, NULL
    FROM penerimaan_prestasi_mandiri pp
    JOIN verification_prestasi_mandiri vp ON vp.id = pp.verification_id
    JOIN registrations_prestasi_mandiri rp ON rp.id = vp.registration_id
    JOIN users u ON u.id = rp.user_id
    JOIN schools s ON s.npsn = pp.npsn

    UNION ALL

    SELECT
        u.id, u.name, u.nisn, s.name,
        'Zonasi', pz.code, pz.created_at
    FROM penerimaan_zonasi pz
    JOIN verification_zonasi vz  ON vz.id  = pz.verification_id
    JOIN registrations_zonasi rz ON rz.id  = vz.registration_id
    JOIN users u                 ON u.id   = rz.user_id
    JOIN schools s               ON s.id   = rz.school_destination_id
)
SELECT
    user_id,
    nama_siswa,
    nisn,
    COUNT(DISTINCT jalur)           AS jumlah_jalur_diterima,
    STRING_AGG(
        jalur || ' (' || sekolah_tujuan || ')',
        ' | ' ORDER BY jalur
    )                               AS detail_penerimaan,
    '⚠ DITERIMA DI BANYAK JALUR'   AS status
FROM semua_penerimaan
GROUP BY user_id, nama_siswa, nisn
HAVING COUNT(DISTINCT jalur) > 1
ORDER BY jumlah_jalur_diterima DESC, nama_siswa;
```

---

# BAGIAN 4: ALTERNATIF QUERY SOLUSI

## 4.1 Solusi Query 1: Menggunakan LEFT JOIN

```sql
-- ============================================================
-- SOLUSI 1: LEFT JOIN
-- Menampilkan siswa yang lolos verifikasi (approve)
-- namun tidak memiliki record penerimaan
-- ============================================================

SELECT
    u.id                AS user_id,
    u.name              AS nama_siswa,
    u.nisn,
    u.phone,
    s.name              AS sekolah_tujuan,
    ra.status           AS status_registrasi,
    va.action           AS aksi_verifikasi,
    'Afirmasi'          AS jalur,
    pa.code             AS kode_penerimaan,
    pa.seen_at          AS waktu_lihat
FROM registrations_afirmasi ra
JOIN users u                    ON u.id  = ra.user_id
JOIN schools s                  ON s.id  = ra.school_destination_id
JOIN verification_afirmasi va   ON va.registration_id = ra.id
                                AND va.action = 'approve'
LEFT JOIN penerimaan_afirmasi pa ON pa.verification_id = va.id
WHERE pa.id IS NULL

UNION ALL

SELECT
    u.id, u.name, u.nisn, u.phone,
    s.name, rm.status, vm.action,
    'Mutasi', pm.code, pm.seen_at
FROM registrations_mutasi rm
JOIN users u                  ON u.id  = rm.user_id
JOIN schools s                ON s.id  = rm.school_destination_id
JOIN verification_mutasi vm   ON vm.registration_id = rm.id
                              AND vm.action = 'approve'
LEFT JOIN penerimaan_mutasi pm ON pm.verification_id = vm.id
WHERE pm.id IS NULL

UNION ALL

SELECT
    u.id, u.name, u.nisn, u.phone,
    s.name, rp.status, vp.action,
    'Prestasi Mandiri', pp.code, pp.seen_at
FROM registrations_prestasi_mandiri rp
JOIN users u                           ON u.id  = rp.user_id
JOIN schools s                         ON s.id  = rp.school_destination_id
JOIN verification_prestasi_mandiri vp  ON vp.registration_id = rp.id
                                       AND vp.action = 'approve'
LEFT JOIN penerimaan_prestasi_mandiri pp ON pp.verification_id = vp.id
WHERE pp.id IS NULL

UNION ALL

SELECT
    u.id, u.name, u.nisn, u.phone,
    s.name, rz.status, vz.action,
    'Zonasi', pz.code, pz.seen_at
FROM registrations_zonasi rz
JOIN users u                  ON u.id  = rz.user_id
JOIN schools s                ON s.id  = rz.school_destination_id
JOIN verification_zonasi vz   ON vz.registration_id = rz.id
                              AND vz.action = 'approve'
LEFT JOIN penerimaan_zonasi pz ON pz.verification_id = vz.id
WHERE pz.id IS NULL

ORDER BY jalur, nama_siswa;
```

## 4.2 Solusi Query 2: Menggunakan NOT EXISTS

```sql
-- ============================================================
-- SOLUSI 2: NOT EXISTS (Subquery)
-- Lebih efisien karena berhenti saat baris pertama ditemukan
-- ============================================================

SELECT
    u.id            AS user_id,
    u.name          AS nama_siswa,
    u.nisn,
    u.phone,
    s.name          AS sekolah_tujuan,
    ra.status       AS status_registrasi,
    'Afirmasi'      AS jalur
FROM registrations_afirmasi ra
JOIN users u    ON u.id  = ra.user_id
JOIN schools s  ON s.id  = ra.school_destination_id
WHERE EXISTS (
    SELECT 1
    FROM verification_afirmasi va
    WHERE va.registration_id = ra.id
      AND va.action = 'approve'
)
AND NOT EXISTS (
    SELECT 1
    FROM verification_afirmasi va
    JOIN penerimaan_afirmasi pa
      ON pa.verification_id = va.id
    WHERE va.registration_id = ra.id
      AND va.action = 'approve'
)

UNION ALL

SELECT
    u.id, u.name, u.nisn, u.phone,
    s.name, rm.status, 'Mutasi'
FROM registrations_mutasi rm
JOIN users u    ON u.id  = rm.user_id
JOIN schools s  ON s.id  = rm.school_destination_id
WHERE EXISTS (
    SELECT 1
    FROM verification_mutasi vm
    WHERE vm.registration_id = rm.id
      AND vm.action = 'approve'
)
AND NOT EXISTS (
    SELECT 1
    FROM verification_mutasi vm
    JOIN penerimaan_mutasi pm
      ON pm.verification_id = vm.id
    WHERE vm.registration_id = rm.id
      AND vm.action = 'approve'
)

UNION ALL

SELECT
    u.id, u.name, u.nisn, u.phone,
    s.name, rp.status, 'Prestasi Mandiri'
FROM registrations_prestasi_mandiri rp
JOIN users u    ON u.id  = rp.user_id
JOIN schools s  ON s.id  = rp.school_destination_id
WHERE EXISTS (
    SELECT 1
    FROM verification_prestasi_mandiri vp
    WHERE vp.registration_id = rp.id
      AND vp.action = 'approve'
)
AND NOT EXISTS (
    SELECT 1
    FROM verification_prestasi_mandiri vp
    JOIN penerimaan_prestasi_mandiri pp
      ON pp.verification_id = vp.id
    WHERE vp.registration_id = rp.id
      AND vp.action = 'approve'
)

UNION ALL

SELECT
    u.id, u.name, u.nisn, u.phone,
    s.name, rz.status, 'Zonasi'
FROM registrations_zonasi rz
JOIN users u    ON u.id  = rz.user_id
JOIN schools s  ON s.id  = rz.school_destination_id
WHERE EXISTS (
    SELECT 1
    FROM verification_zonasi vz
    WHERE vz.registration_id = rz.id
      AND vz.action = 'approve'
)
AND NOT EXISTS (
    SELECT 1
    FROM verification_zonasi vz
    JOIN penerimaan_zonasi pz
      ON pz.verification_id = vz.id
    WHERE vz.registration_id = rz.id
      AND vz.action = 'approve'
)

ORDER BY jalur, nama_siswa;
```

## 4.3 Solusi Query 3: Menggunakan CTE

```sql
-- ============================================================
-- SOLUSI 3: CTE (Common Table Expression)
-- Modular dan mudah dibaca / dimaintain
-- ============================================================

WITH
-- CTE 1: Kumpulkan semua yang sudah terverifikasi approve
terverifikasi AS (
    SELECT
        ra.id           AS reg_id,
        ra.user_id,
        ra.school_destination_id,
        va.id           AS verif_id,
        'Afirmasi'      AS jalur
    FROM registrations_afirmasi ra
    JOIN verification_afirmasi va
      ON va.registration_id = ra.id
     AND va.action = 'approve'

    UNION ALL

    SELECT
        rm.id, rm.user_id, rm.school_destination_id,
        vm.id, 'Mutasi'
    FROM registrations_mutasi rm
    JOIN verification_mutasi vm
      ON vm.registration_id = rm.id
     AND vm.action = 'approve'

    UNION ALL

    SELECT
        rp.id, rp.user_id, rp.school_destination_id,
        vp.id, 'Prestasi Mandiri'
    FROM registrations_prestasi_mandiri rp
    JOIN verification_prestasi_mandiri vp
      ON vp.registration_id = rp.id
     AND vp.action = 'approve'

    UNION ALL

    SELECT
        rz.id, rz.user_id, rz.school_destination_id,
        vz.id, 'Zonasi'
    FROM registrations_zonasi rz
    JOIN verification_zonasi vz
      ON vz.registration_id = rz.id
     AND vz.action = 'approve'
),

-- CTE 2: Kumpulkan semua yang sudah punya penerimaan
sudah_diterima AS (
    SELECT verification_id, 'Afirmasi' AS jalur
    FROM penerimaan_afirmasi

    UNION ALL

    SELECT verification_id, 'Mutasi'
    FROM penerimaan_mutasi

    UNION ALL

    SELECT verification_id, 'Prestasi Mandiri'
    FROM penerimaan_prestasi_mandiri

    UNION ALL

    SELECT verification_id, 'Zonasi'
    FROM penerimaan_zonasi
),

-- CTE 3: Siswa yang terverifikasi tapi BELUM ada penerimaannya
bermasalah AS (
    SELECT t.*
    FROM terverifikasi t
    LEFT JOIN sudah_diterima sd
           ON sd.verification_id = t.verif_id
          AND sd.jalur = t.jalur
    WHERE sd.verification_id IS NULL
)

-- Tampilkan hasil akhir dengan detail siswa dan sekolah
SELECT
    b.reg_id,
    b.verif_id,
    u.id            AS user_id,
    u.name          AS nama_siswa,
    u.nisn,
    u.phone,
    s.name          AS sekolah_tujuan,
    b.jalur,
    'MASALAH: Terverifikasi approve tapi tidak ada penerimaan'
                    AS keterangan
FROM bermasalah b
JOIN users u    ON u.id = b.user_id
JOIN schools s  ON s.id = b.school_destination_id
ORDER BY b.jalur, u.name;
```

---

# BAGIAN 5: OPERASI SET

## 5.1 UNION - Gabungkan Semua Siswa Bermasalah

```sql
-- ============================================================
-- UNION: Semua siswa bermasalah dari semua jalur
-- (Menghilangkan duplikat antar jalur berdasarkan user_id)
-- ============================================================

SELECT
    u.id    AS user_id,
    u.name  AS nama_siswa,
    u.nisn,
    s.name  AS sekolah_tujuan,
    'Afirmasi' AS jalur
FROM registrations_afirmasi ra
JOIN users u    ON u.id = ra.user_id
JOIN schools s  ON s.id = ra.school_destination_id
JOIN verification_afirmasi va
  ON va.registration_id = ra.id AND va.action = 'approve'
LEFT JOIN penerimaan_afirmasi pa ON pa.verification_id = va.id
WHERE pa.id IS NULL

UNION

SELECT
    u.id, u.name, u.nisn, s.name, 'Mutasi'
FROM registrations_mutasi rm
JOIN users u    ON u.id = rm.user_id
JOIN schools s  ON s.id = rm.school_destination_id
JOIN verification_mutasi vm
  ON vm.registration_id = rm.id AND vm.action = 'approve'
LEFT JOIN penerimaan_mutasi pm ON pm.verification_id = vm.id
WHERE pm.id IS NULL

UNION

SELECT
    u.id, u.name, u.nisn, s.name, 'Prestasi Mandiri'
FROM registrations_prestasi_mandiri rp
JOIN users u    ON u.id = rp.user_id
JOIN schools s  ON s.id = rp.school_destination_id
JOIN verification_prestasi_mandiri vp
  ON vp.registration_id = rp.id AND vp.action = 'approve'
LEFT JOIN penerimaan_prestasi_mandiri pp ON pp.verification_id = vp.id
WHERE pp.id IS NULL

UNION

SELECT
    u.id, u.name, u.nisn, s.name, 'Zonasi'
FROM registrations_zonasi rz
JOIN users u    ON u.id = rz.user_id
JOIN schools s  ON s.id = rz.school_destination_id
JOIN verification_zonasi vz
  ON vz.registration_id = rz.id AND vz.action = 'approve'
LEFT JOIN penerimaan_zonasi pz ON pz.verification_id = vz.id
WHERE pz.id IS NULL

ORDER BY jalur, nama_siswa;
```

## 5.2 INTERSECT - Siswa Bermasalah di Lebih dari Satu Jalur

```sql
-- ============================================================
-- INTERSECT: Siswa yang bermasalah DI SEMUA jalur sekaligus
-- (Terdaftar di afirmasi DAN zonasi, keduanya tidak ada penerimaan)
-- ============================================================

SELECT u.id, u.name, u.nisn
FROM registrations_afirmasi ra
JOIN users u    ON u.id = ra.user_id
JOIN verification_afirmasi va
  ON va.registration_id = ra.id AND va.action = 'approve'
LEFT JOIN penerimaan_afirmasi pa ON pa.verification_id = va.id
WHERE pa.id IS NULL

INTERSECT

SELECT u.id, u.name, u.nisn
FROM registrations_mutasi rm
JOIN users u    ON u.id = rm.user_id
JOIN verification_mutasi vm
  ON vm.registration_id = rm.id AND vm.action = 'approve'
LEFT JOIN penerimaan_mutasi pm ON pm.verification_id = vm.id
WHERE pm.id IS NULL

INTERSECT

SELECT u.id, u.name, u.nisn
FROM registrations_zonasi rz
JOIN users u    ON u.id = rz.user_id
JOIN verification_zonasi vz
  ON vz.registration_id = rz.id AND vz.action = 'approve'
LEFT JOIN penerimaan_zonasi pz ON pz.verification_id = vz.id
WHERE pz.id IS NULL

ORDER BY name;
```

## 5.3 EXCEPT - Siswa Bermasalah di Afirmasi Tapi Tidak di Zonasi

```sql
-- ============================================================
-- EXCEPT: Siswa bermasalah di Afirmasi
-- yang TIDAK bermasalah di Zonasi
-- (Masalah hanya spesifik di satu jalur)
-- ============================================================

-- Bermasalah di Afirmasi
SELECT u.id, u.name, u.nisn
FROM registrations_afirmasi ra
JOIN users u    ON u.id = ra.user_id
JOIN verification_afirmasi va
  ON va.registration_id = ra.id AND va.action = 'approve'
LEFT JOIN penerimaan_afirmasi pa ON pa.verification_id = va.id
WHERE pa.id IS NULL

EXCEPT

-- Bermasalah di Zonasi (dikecualikan)
SELECT u.id, u.name, u.nisn
FROM registrations_zonasi rz
JOIN users u    ON u.id = rz.user_id
JOIN verification_zonasi vz
  ON vz.registration_id = rz.id AND vz.action = 'approve'
LEFT JOIN penerimaan_zonasi pz ON pz.verification_id = vz.id
WHERE pz.id IS NULL

ORDER BY name;
```

---

# BAGIAN 6: OPTIMASI QUERY

## 6.1 Tambahan Index untuk Optimasi

```sql
-- ============================================================
-- INDEX TAMBAHAN UNTUK OPTIMASI QUERY INVESTIGASI
-- ============================================================

-- Index pada kolom status (sering difilter)
CREATE INDEX IF NOT EXISTS idx_reg_afirmasi_status
    ON registrations_afirmasi(status);
CREATE INDEX IF NOT EXISTS idx_reg_mutasi_status
    ON registrations_mutasi(status);
CREATE INDEX IF NOT EXISTS idx_reg_prestasi_status
    ON registrations_prestasi_mandiri(status);
CREATE INDEX IF NOT EXISTS idx_reg_zonasi_status
    ON registrations_zonasi(status);

-- Index pada kolom action di verifikasi (sering difilter)
CREATE INDEX IF NOT EXISTS idx_ver_afirmasi_action
    ON verification_afirmasi(action);
CREATE INDEX IF NOT EXISTS idx_ver_mutasi_action
    ON verification_mutasi(action);
CREATE INDEX IF NOT EXISTS idx_ver_prestasi_action
    ON verification_prestasi_mandiri(action);
CREATE INDEX IF NOT EXISTS idx_ver_zonasi_action
    ON verification_zonasi(action);

-- Index composite (registration_id + action) untuk query join+filter
CREATE INDEX IF NOT EXISTS idx_ver_afirmasi_reg_action
    ON verification_afirmasi(registration_id, action);
CREATE INDEX IF NOT EXISTS idx_ver_mutasi_reg_action
    ON verification_mutasi(registration_id, action);
CREATE INDEX IF NOT EXISTS idx_ver_prestasi_reg_action
    ON verification_prestasi_mandiri(registration_id, action);
CREATE INDEX IF NOT EXISTS idx_ver_zonasi_reg_action
    ON verification_zonasi(registration_id, action);

-- Index pada verification_id di tabel penerimaan
CREATE INDEX IF NOT EXISTS idx_pen_afirmasi_verif_id
    ON penerimaan_afirmasi(verification_id);
CREATE INDEX IF NOT EXISTS idx_pen_mutasi_verif_id
    ON penerimaan_mutasi(verification_id);
CREATE INDEX IF NOT EXISTS idx_pen_prestasi_verif_id
    ON penerimaan_prestasi_mandiri(verification_id);
CREATE INDEX IF NOT EXISTS idx_pen_zonasi_verif_id
    ON penerimaan_zonasi(verification_id);
```

## 6.2 Query Teroptimasi dengan EXPLAIN ANALYZE

```sql
-- ============================================================
-- QUERY TEROPTIMASI: Gabungan CTE + Index
-- Gunakan EXPLAIN ANALYZE untuk melihat query plan
-- ============================================================

EXPLAIN (ANALYZE, BUFFERS, FORMAT TEXT)
WITH
terverifikasi_afirmasi AS (
    -- Filter awal hanya pada action = 'approve'
    -- Memanfaatkan idx_ver_afirmasi_reg_action
    SELECT
        va.registration_id,
        va.id AS verif_id
    FROM verification_afirmasi va
    WHERE va.action = 'approve'
),
tanpa_penerimaan_afirmasi AS (
    -- Anti-join: verifikasi approve yang tidak punya penerimaan
    -- Memanfaatkan idx_pen_afirmasi_verif_id
    SELECT ta.registration_id, ta.verif_id
    FROM terverifikasi_afirmasi ta
    LEFT JOIN penerimaan_afirmasi pa
           ON pa.verification_id = ta.verif_id
    WHERE pa.id IS NULL
)
SELECT
    u.id            AS user_id,
    u.name          AS nama_siswa,
    u.nisn,
    s.name          AS sekolah_tujuan,
    'Afirmasi'      AS jalur
FROM tanpa_penerimaan_afirmasi tpa
-- Memanfaatkan idx_reg_afirmasi_user_id
JOIN registrations_afirmasi ra ON ra.id = tpa.registration_id
JOIN users u                   ON u.id  = ra.user_id
JOIN schools s                 ON s.id  = ra.school_destination_id
ORDER BY s.name, u.name;
```

## 6.3 View Teroptimasi untuk Query Berulang

```sql
-- ============================================================
-- VIEW: Rekapitulasi siswa bermasalah
-- Dibuat agar tidak perlu menulis query panjang berulang kali
-- ============================================================

CREATE OR REPLACE VIEW v_siswa_bermasalah AS
WITH
terverifikasi AS (
    SELECT va.registration_id, va.id AS verif_id, 'Afirmasi' AS jalur
    FROM verification_afirmasi va WHERE va.action = 'approve'
    UNION ALL
    SELECT vm.registration_id, vm.id, 'Mutasi'
    FROM verification_mutasi vm WHERE vm.action = 'approve'
    UNION ALL
    SELECT vp.registration_id, vp.id, 'Prestasi Mandiri'
    FROM verification_prestasi_mandiri vp WHERE vp.action = 'approve'
    UNION ALL
    SELECT vz.registration_id, vz.id, 'Zonasi'
    FROM verification_zonasi vz WHERE vz.action = 'approve'
),
sudah_diterima AS (
    SELECT verification_id, 'Afirmasi' AS jalur FROM penerimaan_afirmasi
    UNION ALL
    SELECT verification_id, 'Mutasi'   FROM penerimaan_mutasi
    UNION ALL
    SELECT verification_id, 'Prestasi Mandiri' FROM penerimaan_prestasi_mandiri
    UNION ALL
    SELECT verification_id, 'Zonasi'   FROM penerimaan_zonasi
),
bermasalah AS (
    SELECT t.registration_id, t.verif_id, t.jalur
    FROM terverifikasi t
    LEFT JOIN sudah_diterima sd
           ON sd.verification_id = t.verif_id
          AND sd.jalur = t.jalur
    WHERE sd.verification_id IS NULL
)
SELECT
    u.id            AS user_id,
    u.name          AS nama_siswa,
    u.nisn,
    u.phone,
    s.name          AS sekolah_tujuan,
    b.jalur,
    b.verif_id,
    'Terverifikasi approve tapi tidak ada penerimaan'
                    AS keterangan
FROM bermasalah b
JOIN (
    SELECT id, user_id, school_destination_id
    FROM registrations_afirmasi
    UNION ALL
    SELECT id, user_id, school_destination_id
    FROM registrations_mutasi
    UNION ALL
    SELECT id, user_id, school_destination_id
    FROM registrations_prestasi_mandiri
    UNION ALL
    SELECT id, user_id, school_destination_id
    FROM registrations_zonasi
) reg ON reg.id = b.registration_id
JOIN users u    ON u.id = reg.user_id
JOIN schools s  ON s.id = reg.school_destination_id;

-- Cara penggunaan view
SELECT * FROM v_siswa_bermasalah
ORDER BY jalur, nama_siswa;

-- Filter per jalur
SELECT * FROM v_siswa_bermasalah
WHERE jalur = 'Zonasi'
ORDER BY sekolah_tujuan, nama_siswa;

-- Hitung per sekolah
SELECT
    sekolah_tujuan,
    jalur,
    COUNT(*) AS jumlah_bermasalah
FROM v_siswa_bermasalah
GROUP BY sekolah_tujuan, jalur
ORDER BY jumlah_bermasalah DESC;
```

---

# BAGIAN 7: UJI DAN PERBANDINGAN QUERY

## 7.1 Uji Performa Ketiga Solusi

```sql
-- ============================================================
-- UJI PERFORMA: Jalankan satu per satu dan bandingkan
-- Execution Time dan Query Plan
-- ============================================================

-- TEST 1: LEFT JOIN (Solusi 1)
EXPLAIN (ANALYZE, BUFFERS, FORMAT TEXT)
SELECT
    u.id, u.name, u.nisn, s.name, 'Afirmasi' AS jalur
FROM registrations_afirmasi ra
JOIN users u                    ON u.id  = ra.user_id
JOIN schools s                  ON s.id  = ra.school_destination_id
JOIN verification_afirmasi va   ON va.registration_id = ra.id
                                AND va.action = 'approve'
LEFT JOIN penerimaan_afirmasi pa ON pa.verification_id = va.id
WHERE pa.id IS NULL;

-- TEST 2: NOT EXISTS (Solusi 2)
EXPLAIN (ANALYZE, BUFFERS, FORMAT TEXT)
SELECT
    u.id, u.name, u.nisn, s.name, 'Afirmasi' AS jalur
FROM registrations_afirmasi ra
JOIN users u    ON u.id  = ra.user_id
JOIN schools s  ON s.id  = ra.school_destination_id
WHERE EXISTS (
    SELECT 1
    FROM verification_afirmasi va
    WHERE va.registration_id = ra.id
      AND va.action = 'approve'
)
AND NOT EXISTS (
    SELECT 1
    FROM verification_afirmasi va
    JOIN penerimaan_afirmasi pa
      ON pa.verification_id = va.id
    WHERE va.registration_id = ra.id
      AND va.action = 'approve'
);

-- TEST 3: CTE (Solusi 3)
EXPLAIN (ANALYZE, BUFFERS, FORMAT TEXT)
WITH
terverifikasi AS (
    SELECT va.registration_id, va.id AS verif_id
    FROM verification_afirmasi va
    WHERE va.action = 'approve'
),
tanpa_penerimaan AS (
    SELECT t.registration_id
    FROM terverifikasi t
    LEFT JOIN penerimaan_afirmasi pa
           ON pa.verification_id = t.verif_id
    WHERE pa.id IS NULL
)
SELECT u.id, u.name, u.nisn, s.name, 'Afirmasi' AS jalur
FROM tanpa_penerimaan tp
JOIN registrations_afirmasi ra ON ra.id = tp.registration_id
JOIN users u    ON u.id = ra.user_id
JOIN schools s  ON s.id = ra.school_destination_id;
```

## 7.2 Tabel Perbandingan Hasil Uji

```
╔══════════════════════════════════════════════════════════════════════╗
║              PERBANDINGAN PERFORMA QUERY                            ║
╠══════════════════╦══════════════╦════════════════╦══════════════════╣
║ Kriteria         ║ Solusi 1     ║ Solusi 2       ║ Solusi 3         ║
║                  ║ LEFT JOIN    ║ NOT EXISTS     ║ CTE              ║
╠══════════════════╬══════════════╬════════════════╬══════════════════╣
║ Kemudahan Baca   ║ ★★★★☆       ║ ★★★☆☆         ║ ★★★★★           ║
║ Performa Kecil   ║ ★★★★☆       ║ ★★★★★         ║ ★★★★☆           ║
║ Performa Besar   ║ ★★★☆☆       ║ ★★★★★         ║ ★★★★★           ║
║ Kemudahan Debug  ║ ★★★☆☆       ║ ★★★☆☆         ║ ★★★★★           ║
║ Maintainability  ║ ★★★☆☆       ║ ★★★☆☆         ║ ★★★★★           ║
║ Reusability      ║ ★★☆☆☆       ║ ★★☆☆☆         ║ ★★★★★           ║
╠══════════════════╬══════════════╬════════════════╬══════════════════╣
║ REKOMENDASI      ║ Data kecil   ║ Data besar,    ║ Query kompleks,  ║
║                  ║ & sederhana  ║ butuh performa ║ tim development  ║
╚══════════════════╩══════════════╩════════════════╩══════════════════╝

KESIMPULAN:
  • Data kecil  → Solusi 1 (LEFT JOIN) paling mudah dipahami
  • Data besar  → Solusi 2 (NOT EXISTS) paling efisien
  • Tim besar   → Solusi 3 (CTE) paling mudah dimaintain
  • REKOMENDASI → Solusi 3 (CTE) + Index = Optimal untuk PPDB
```

---

# BAGIAN 8: TRIGGER

## 8.1 Trigger Row-Level 1: Validasi Pagu Sekolah

```sql
-- ============================================================
-- TRIGGER 1: BEFORE INSERT ROW-LEVEL
-- Mencegah penerimaan zonasi melebihi pagu sekolah
-- ============================================================

CREATE OR REPLACE FUNCTION fn_validasi_pagu_zonasi()
RETURNS TRIGGER AS $$
DECLARE
    v_school_id     INT;
    v_school_name   VARCHAR;
    v_pagu          INT;
    v_total         INT;
BEGIN
    -- Ambil data sekolah dari chain penerimaan
    SELECT
        rz.school_destination_id,
        s.name,
        s.pagu_zonasi
    INTO v_school_id, v_school_name, v_pagu
    FROM verification_zonasi vz
    JOIN registrations_zonasi rz ON rz.id = vz.registration_id
    JOIN schools s               ON s.id  = rz.school_destination_id
    WHERE vz.id = NEW.verification_id;

    -- Hitung total yang sudah diterima di sekolah tersebut
    SELECT COUNT(*)
    INTO v_total
    FROM penerimaan_zonasi pz
    JOIN verification_zonasi vz ON vz.id  = pz.verification_id
    JOIN registrations_zonasi rz ON rz.id = vz.registration_id
    WHERE rz.school_destination_id = v_school_id;

    -- Tolak jika melebihi pagu
    IF v_total >= v_pagu THEN
        RAISE EXCEPTION
            'PAGU PENUH: Sekolah "%" telah mencapai batas pagu '
            'zonasi (%). Total sudah diterima: %',
            v_school_name, v_pagu, v_total;
    END IF;

    RAISE NOTICE
        'Penerimaan zonasi "%" — slot terisi: %/%',
        v_school_name, v_total + 1, v_pagu;

    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

DROP TRIGGER IF EXISTS trg_validasi_pagu_zonasi
    ON penerimaan_zonasi;

CREATE TRIGGER trg_validasi_pagu_zonasi
    BEFORE INSERT ON penerimaan_zonasi
    FOR EACH ROW
    EXECUTE FUNCTION fn_validasi_pagu_zonasi();
```

## 8.2 Trigger Row-Level 2: Auto-Update Status Registrasi

```sql
-- ============================================================
-- TRIGGER 2: AFTER INSERT ROW-LEVEL
-- Otomatis update status registrasi dan users
-- setelah penerimaan berhasil dibuat
-- ============================================================

CREATE OR REPLACE FUNCTION fn_update_status_setelah_penerimaan_zonasi()
RETURNS TRIGGER AS $$
DECLARE
    v_registration_id   INT;
    v_user_id           INT;
    v_user_name         VARCHAR;
BEGIN
    -- Ambil registration_id dari verifikasi
    SELECT vz.registration_id
    INTO v_registration_id
    FROM verification_zonasi vz
    WHERE vz.id = NEW.verification_id;

    -- Update status registrasi zonasi
    UPDATE registrations_zonasi
    SET
        status     = 'accepted',
        updated_at = CURRENT_TIMESTAMP
    WHERE id = v_registration_id
      AND status != 'accepted';

    -- Ambil user_id
    SELECT user_id INTO v_user_id
    FROM registrations_zonasi
    WHERE id = v_registration_id;

    -- Ambil nama untuk logging
    SELECT name INTO v_user_name
    FROM users WHERE id = v_user_id;

    -- Update acceptance info di tabel users
    UPDATE users
    SET
        acceptance_type = 'zonasi',
        acceptance_id   = NEW.id,
        updated_at      = CURRENT_TIMESTAMP
    WHERE id = v_user_id;

    RAISE NOTICE
        'Status registrasi zonasi user "%" diupdate → accepted. '
        'Kode penerimaan: %',
        v_user_name, NEW.code;

    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

DROP TRIGGER IF EXISTS trg_update_status_penerimaan_zonasi
    ON penerimaan_zonasi;

CREATE TRIGGER trg_update_status_penerimaan_zonasi
    AFTER INSERT ON penerimaan_zonasi
    FOR EACH ROW
    EXECUTE FUNCTION fn_update_status_setelah_penerimaan_zonasi();
```

## 8.3 Trigger Row-Level 3: Audit Log Perubahan Status

```sql
-- ============================================================
-- TABEL AUDIT LOG
-- ============================================================

CREATE TABLE IF NOT EXISTS audit_log_ppdb (
    id              SERIAL PRIMARY KEY,
    tabel_asal      VARCHAR(100)    NOT NULL,
    record_id       INT             NOT NULL,
    user_id         INT,
    kolom_berubah   VARCHAR(100),
    nilai_lama      TEXT,
    nilai_baru      TEXT,
    db_user         VARCHAR(100)    DEFAULT current_user,
    changed_at      TIMESTAMP       DEFAULT CURRENT_TIMESTAMP,
    keterangan      TEXT
);

-- ============================================================
-- TRIGGER 3: AFTER UPDATE ROW-LEVEL
-- Audit log setiap perubahan status di registrations_afirmasi
-- ============================================================

CREATE OR REPLACE FUNCTION fn_audit_status_afirmasi()
RETURNS TRIGGER AS $$
BEGIN
    IF OLD.status IS DISTINCT FROM NEW.status THEN
        INSERT INTO audit_log_ppdb (
            tabel_asal,
            record_id,
            user_id,
            kolom_berubah,
            nilai_lama,
            nilai_baru,
            keterangan
        ) VALUES (
            'registrations_afirmasi',
            NEW.id,
            NEW.user_id,
            'status',
            OLD.status,
            NEW.status,
            FORMAT(
                'Status berubah dari [%s] → [%s] pada %s',
                OLD.status,
                NEW.status,
                TO_CHAR(CURRENT_TIMESTAMP, 'DD-MM-YYYY HH24:MI:SS')
            )
        );

        RAISE NOTICE
            'AUDIT: Registrasi afirmasi ID % — status: % → %',
            NEW.id, OLD.status, NEW.status;
    END IF;

    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

DROP TRIGGER IF EXISTS trg_audit_status_afirmasi
    ON registrations_afirmasi;

CREATE TRIGGER trg_audit_status_afirmasi
    AFTER UPDATE OF status ON registrations_afirmasi
    FOR EACH ROW
    EXECUTE FUNCTION fn_audit_status_afirmasi();

-- Cek hasil audit
-- SELECT * FROM audit_log_ppdb ORDER BY changed_at DESC;
```

---

## 8.4 INSTEAD OF TRIGGER pada View

```sql
-- ============================================================
-- VIEW: Gabungan semua penerimaan dari semua jalur
-- ============================================================

CREATE OR REPLACE VIEW v_semua_penerimaan AS
SELECT
    pa.id               AS penerimaan_id,
    'afirmasi'          AS jalur,
    u.id                AS user_id,
    u.name              AS nama_siswa,
    u.nisn,
    s.name              AS nama_sekolah,
    s.npsn,
    pa.code             AS kode_penerimaan,
    pa.seen_at,
    pa.created_at,
    pa.updated_at
FROM penerimaan_afirmasi pa
JOIN verification_afirmasi va   ON va.id = pa.verification_id
JOIN registrations_afirmasi ra  ON ra.id = va.registration_id
JOIN users u                    ON u.id  = ra.user_id
JOIN schools s                  ON s.id  = ra.school_destination_id

UNION ALL

SELECT
    pm.id, 'mutasi',
    u.id, u.name, u.nisn, s.name, s.npsn,
    pm.code, pm.seen_at, pm.created_at, pm.updated_at
FROM penerimaan_mutasi pm
JOIN verification_mutasi vm   ON vm.id = pm.verification_id
JOIN registrations_mutasi rm  ON rm.id = vm.registration_id
JOIN users u                  ON u.id  = rm.user_id
JOIN schools s                ON s.id  = rm.school_destination_id

UNION ALL

SELECT
    pp.id, 'prestasi_mandiri',
    u.id, u.name, u.nisn, s.name, s.npsn,
    pp.code, pp.seen_at,
    NULL AS created_at,
    NULL AS updated_at
FROM penerimaan_prestasi_mandiri pp
JOIN verification_prestasi_mandiri vp ON vp.id = pp.verification_id
JOIN registrations_prestasi_mandiri rp ON rp.id = vp.registration_id
JOIN users u                           ON u.id  = rp.user_id
JOIN schools s                         ON s.npsn = pp.npsn

UNION ALL

SELECT
    pz.id, 'zonasi',
    u.id, u.name, u.nisn, s.name, s.npsn,
    pz.code, pz.seen_at, pz.created_at, pz.updated_at
FROM penerimaan_zonasi pz
JOIN verification_zonasi vz   ON vz.id = pz.verification_id
JOIN registrations_zonasi rz  ON rz.id = vz.registration_id
JOIN users u                  ON u.id  = rz.user_id
JOIN schools s                ON s.id  = rz.school_destination_id;

-- ============================================================
-- INSTEAD OF TRIGGER: Menangani UPDATE seen_at melalui view
-- Ketika admin menandai bahwa siswa sudah melihat pengumuman
-- ============================================================

CREATE OR REPLACE FUNCTION fn_instead_of_update_seen_at()
RETURNS TRIGGER AS $$
BEGIN
    -- Validasi: hanya boleh update seen_at
    IF NEW.seen_at IS NULL THEN
        RAISE EXCEPTION
            'seen_at tidak boleh diset NULL melalui view ini';
    END IF;

    -- Routing UPDATE ke tabel yang sesuai berdasarkan jalur
    IF OLD.jalur = 'afirmasi' THEN
        UPDATE penerimaan_afirmasi
        SET
            seen_at    = NEW.seen_at,
            updated_at = CURRENT_TIMESTAMP
        WHERE id = OLD.penerimaan_id;

    ELSIF OLD.jalur = 'mutasi' THEN
        UPDATE penerimaan_mutasi
        SET
            seen_at    = NEW.seen_at,
            updated_at = CURRENT_TIMESTAMP
        WHERE id = OLD.penerimaan_id;

    ELSIF OLD.jalur = 'prestasi_mandiri' THEN
        UPDATE penerimaan_prestasi_mandiri
        SET seen_at = NEW.seen_at
        WHERE id = OLD.penerimaan_id;

    ELSIF OLD.jalur = 'zonasi' THEN
        UPDATE penerimaan_zonasi
        SET
            seen_at    = NEW.seen_at,
            updated_at = CURRENT_TIMESTAMP
        WHERE id = OLD.penerimaan_id;

    ELSE
        RAISE EXCEPTION
            'Jalur tidak dikenal: %', OLD.jalur;
    END IF;

    RAISE NOTICE
        'seen_at untuk penerimaan ID % jalur % '
        'diupdate menjadi %',
        OLD.penerimaan_id, OLD.jalur, NEW.seen_at;

    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

DROP TRIGGER IF EXISTS trg_instead_of_update_seen_at
    ON v_semua_penerimaan;

CREATE TRIGGER trg_instead_of_update_seen_at
    INSTEAD OF UPDATE ON v_semua_penerimaan
    FOR EACH ROW
    EXECUTE FUNCTION fn_instead_of_update_seen_at();

-- ============================================================
-- Contoh penggunaan INSTEAD OF TRIGGER
-- ============================================================

-- Update seen_at siswa jalur zonasi dengan penerimaan_id = 1
-- UPDATE v_semua_penerimaan
-- SET seen_at = CURRENT_TIMESTAMP
-- WHERE penerimaan_id = 1
--   AND jalur = 'zonasi';

-- Lihat hasil
-- SELECT penerimaan_id, jalur, nama_siswa, seen_at
-- FROM v_semua_penerimaan
-- WHERE penerimaan_id = 1;
```
