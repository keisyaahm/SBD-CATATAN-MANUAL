# 📚 Belajar Sistem Basis Data - Fokus DDL (Data Definition Language)

Repository ini berisi catatan pribadi, *cheatsheet* kode, dan penyelesaian studi kasus terkait **Data Definition Language (DDL)** menggunakan MySQL/MariaDB dan phpMyAdmin. 

Tujuan repository ini adalah untuk mendokumentasikan pemahaman tentang cara membangun fondasi database sebelum memasukkan data (DML).

## 📑 Daftar Isi
1. [Konsep Dasar DDL](#1-konsep-dasar-ddl)
2. [Aturan Main (Constraints)](#2-aturan-main-constraints)
3. [Cheatsheet Syntax SQL](#3-cheatsheet-syntax-sql)
4. [Studi Kasus & Relasi Tabel](#4-studi-kasus--relasi-tabel)
5. [Troubleshooting & Error Handling](#5-troubleshooting--error-handling)

---

## 1. Konsep Dasar DDL
DDL ibarat "Tukang Bangunan" dalam database. Tugasnya adalah membuat, merenovasi, dan menghancurkan struktur wadah data.

* **`CREATE`**: Membuat database atau tabel dari nol.
* **`ALTER`**: Merenovasi tabel yang sudah ada (menambah, mengubah, menghapus kolom).
* **`DROP`**: Menghancurkan tabel/database sampai ke akar-akarnya.
* **`TRUNCATE`**: Mengosongkan isi data di dalam tabel, tapi kerangka tabelnya tetap dipertahankan.

---

## 2. Aturan Main (Constraints)
*Constraint* adalah satpam penjaga pintu tabel agar data yang masuk tidak sembarangan dan tetap konsisten.

| Constraint | Fungsi |
| :--- | :--- |
| **`PRIMARY KEY`** | Identitas utama baris data (wajib unik & tidak boleh kosong). |
| **`FOREIGN KEY`** | Kunci tamu untuk menyambungkan antar tabel (Relasi). |
| **`NOT NULL`** | Data di kolom ini wajib diisi. |
| **`UNIQUE`** | Data tidak boleh ada yang kembar (contoh: Email, No HP). |
| **`DEFAULT`** | Nilai otomatis jika user tidak mengisi data. |
| **`CHECK`** | Validasi syarat masuk (contoh: `CHECK (umur >= 18)`). |

---

## 3. Cheatsheet Syntax SQL

### 🛠️ Membuat Database & Tabel
```sql
-- Membuat Database yang aman dari error
CREATE DATABASE IF NOT EXISTS nama_database;
USE nama_database;

-- Membuat Tabel Dasar
CREATE TABLE pengguna (
    id_user INT PRIMARY KEY AUTO_INCREMENT,
    nama VARCHAR(100) NOT NULL
);
```

### 🏗️ Merenovasi Tabel (`ALTER TABLE`)
```sql
-- Menambah kolom baru (Wajib pakai tipe data!)
ALTER TABLE pengguna ADD umur INT;

-- Mengubah tipe data kolom
ALTER TABLE pengguna MODIFY COLUMN nama VARCHAR(150);

-- Mengubah nama kolom (Harus tulis nama lama, baru, & tipe data)
ALTER TABLE pengguna CHANGE COLUMN umur usia INT;

-- Menghapus sebuah kolom
ALTER TABLE pengguna DROP COLUMN usia;

-- Menambahkan aturan CHECK belakangan
ALTER TABLE pengguna ADD CHECK (umur >= 18);
```

---

## 4. Studi Kasus & Relasi Tabel
Aturan Emas Relasi: **Tabel Parent (Induk) WAJIB dibuat lebih dulu daripada tabel Child (Anak).**

### Contoh Relasi Many-to-Many (Junction Table)
Kasus: Menghubungkan entitas `Pembeli` dan `Produk` melalui tabel `Transaksi`.

```sql
-- 1. Buat Tabel Parent 1
CREATE TABLE Pembeli (
    ID_Pembeli INT PRIMARY KEY,
    Nama_Pembeli VARCHAR(50),
    email VARCHAR(50)
);

-- 2. Buat Tabel Parent 2
CREATE TABLE Produk (
    ID_Produk INT PRIMARY KEY,
    Nama_Produk VARCHAR(50)
);

-- 3. Buat Junction Table (Tabel Penghubung)
CREATE TABLE Transaksi (
    ID_Transaksi INT PRIMARY KEY,
    ID_Pembeli INT,
    ID_Produk INT,
    Tanggal_Transaksi DATE,
    FOREIGN KEY (ID_Pembeli) REFERENCES Pembeli(ID_Pembeli),
    FOREIGN KEY (ID_Produk) REFERENCES Produk(ID_Produk)
);
```

---

## 5. Troubleshooting & Error Handling
Catatan dari pengalaman saat menggunakan **phpMyAdmin**:

1. **Error Linter vs MySQL Engine:** Terkadang kotak teks di phpMyAdmin memunculkan silang merah (contoh: saat mengetik `ADD CHECK (...)`). Jika secara logika SQL sudah benar, **abaikan error visual UI tersebut dan klik Go!**. Mesin MySQL jauh lebih pintar dari UI-nya.
2. **Komentar Bikin Error:**
   Saat membuat komentar di SQL menggunakan tanda `--`, **wajib menambahkan spasi** setelahnya (contoh: `-- ini komentar`). Jika tanpa spasi (`--ini komentar`), MySQL akan mengiranya sebagai perintah aneh dan menyebabkan Syntax Error `#1064`.
3. **Lupa Tipe Data saat ALTER:**
   Saat menambah kolom (`ALTER TABLE pengguna ADD no_telpon;`), MySQL akan menolak. Wajib sertakan tipe datanya: `ALTER TABLE pengguna ADD no_telpon VARCHAR(15);`.
4. **Error #4025 Constraint Failed:**
   Ini BUKAN error sistem yang rusak. Ini artinya aturan `CHECK` (misal umur >= 18) **berhasil bekerja** menolak data (Negative Testing) yang tidak sesuai syarat. Sistem aman!
```
