# Panduan Uji Coba (Testing) Trigger SQL - Kelompok 3

Dokumen ini berisi skenario kueri SQL untuk menguji coba seluruh trigger yang telah dibuat (`INSERT`, `UPDATE`, dan `DELETE`). Skenario dibagi menjadi dua bagian: memicu pesan eror/validasi (**BEFORE**) dan melihat hasil sukses pencatatan riwayat (**AFTER**).

---

## 1. Uji Coba Trigger INSERT

### Tes BEFORE INSERT (Validasi Umur)
Coba masukkan data mahasiswa dengan umur di bawah 17 tahun. Database harus otomatis menolak proses ini dan menampilkan pesan eror.
```sql
INSERT INTO Mahasiswa (nama, umur) VALUES ('Rian Kecil', 15);
```
* **Hasil yang Diharapkan:** Muncul pesan eror `Umur minimal 17 tahun`.

### Tes AFTER INSERT (Log Aktivitas)
Masukkan data mahasiswa yang valid (umur >= 17 tahun) untuk melihat apakah data masuk dan log tercatat otomatis.
```sql
INSERT INTO Mahasiswa (nama, umur) VALUES ('Budi Santoso', 20);
```
* **Hasil yang Diharapkan:** Data sukses masuk ke tabel `Mahasiswa`.

### Cek Hasil Log Aktivitas
```sql
SELECT * FROM Log_Aktivitas;
```
* **Hasil di Tabel Log:** Kolom aktivitas harus otomatis memuat teks: `"Mahasiswa Budi Santoso ditambahkan"`.

---

## 2. Uji Coba Trigger UPDATE

### Tes BEFORE UPDATE (Mencegah Umur Negatif)
Coba ubah umur mahasiswa yang sudah terdaftar menjadi angka minus atau negatif.
```sql
UPDATE Mahasiswa SET umur = -5 WHERE nama = 'Budi Santoso';
```
* **Hasil yang Diharapkan:** Muncul pesan eror `Umur tidak boleh negatif`.

###  Tes AFTER UPDATE (Log Perubahan Data)
Coba ubah atau edit nama mahasiswa dari 'Budi Santoso' menjadi 'Budi Perkasa'.
```sql
UPDATE Mahasiswa SET nama = 'Budi Perkasa' WHERE nama = 'Budi Santoso';
```
* **Hasil yang Diharapkan:** Nama sukses diperbarui di tabel `Mahasiswa`.

### Cek Hasil Log Aktivitas
```sql
SELECT * FROM Log_Aktivitas;
```
* **Hasil di Tabel Log:** Kolom aktivitas harus otomatis memuat teks: `"Data Budi Santoso diubah menjadi Budi Perkasa"`.

---

## 3. Uji Coba Trigger DELETE

### Tes BEFORE DELETE (Melindungi Data Admin)
Sebelum melakukan tes ini, masukkan data bernama 'Admin' terlebih dahulu, lalu coba hapus data tersebut.
```sql
-- Jalankan ini dulu untuk menyiapkan data uji coba
INSERT INTO Mahasiswa (nama, umur) VALUES ('Admin', 25);

-- Coba lakukan penghapusan (Pasti Ditolak)
DELETE FROM Mahasiswa WHERE nama = 'Admin';
```
* **Hasil yang Diharapkan:** Muncul pesan eror `Data Admin tidak boleh dihapus`.

###  Tes AFTER DELETE (Log Penghapusan)
Coba hapus data mahasiswa biasa yang tadi sudah diubah namanya ('Budi Perkasa').
```sql
DELETE FROM Mahasiswa WHERE nama = 'Budi Perkasa';
```
* **Hasil yang Diharapkan:** Data sukses terhapus dari tabel `Mahasiswa`.

### Cek Hasil Log Aktivitas
```sql
SELECT * FROM Log_Aktivitas;
```
* **Hasil di Tabel Log:** Kolom aktivitas harus otomatis memuat teks: `"Mahasiswa Budi Perkasa dihapus"`.

---



# Panduan Implementasi Trigger SQL - Kelompok 3

Repositori ini berisi kumpulan kode SQL untuk implementasi **Trigger** pada database, mencakup operasi `INSERT`, `UPDATE`, dan `DELETE`. Setiap operasi dilengkapi dengan trigger `BEFORE` (untuk validasi data) dan `AFTER` (untuk pencatatan log aktivitas otomatis).

---

## Persyaratan Tabel (Prerequisite)

Sebelum memasang trigger, pastikan Anda sudah membuat tabel `Mahasiswa` dan `Log_Aktivitas` terlebih dahulu di database Anda.

---

## Kumpulan Kode Trigger

### A. Trigger INSERT

*   **BEFORE INSERT:** Validasi umur agar mahasiswa yang didaftarkan minimal berusia 17 tahun.
*   **AFTER INSERT:** Otomatis mencatat riwayat ke tabel log setelah data mahasiswa sukses ditambahkan.

```sql
-- BEFORE INSERT: Validasi Umur
DELIMITER $$ 

CREATE TRIGGER before_insert_mahasiswa 
BEFORE INSERT ON Mahasiswa 
FOR EACH ROW 
BEGIN 
    IF NEW.umur < 17 THEN 
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Umur minimal 17 tahun'; 
    END IF; 
END$$

-- AFTER INSERT: Log Aktivitas
CREATE TRIGGER after_insert_mahasiswa 
AFTER INSERT ON Mahasiswa 
FOR EACH ROW 
BEGIN
    INSERT INTO Log_Aktivitas(aktivitas) 
    VALUES(CONCAT('Mahasiswa ', NEW.nama, ' ditambahkan')); 
END$$ 

DELIMITER ;
```

---

### B. Trigger UPDATE

*   **BEFORE UPDATE:** Memastikan data umur yang diubah tidak bernilai negatif (di bawah 0).
*   **AFTER UPDATE:** Otomatis mencatat riwayat perubahan nama lama menjadi nama baru ke tabel log.

```sql
-- BEFORE UPDATE: Mencegah Umur Negatif
DELIMITER $$ 

CREATE TRIGGER before_update_mahasiswa 
BEFORE UPDATE ON Mahasiswa 
FOR EACH ROW 
BEGIN 
    IF NEW.umur < 0 THEN 
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Umur tidak boleh negatif'; 
    END IF; 
END$$

-- AFTER UPDATE: Log Perubahan Data
CREATE TRIGGER after_update_mahasiswa 
AFTER UPDATE ON Mahasiswa 
FOR EACH ROW 
BEGIN
    INSERT INTO Log_Aktivitas(aktivitas) 
    VALUES(CONCAT('Data ', OLD.nama, ' diubah menjadi ', NEW.nama)); 
END$$ 

DELIMITER ;
```

---

### C. Trigger DELETE

*   **BEFORE DELETE:** Memblokir proses penghapusan jika data mahasiswa yang dihapus bernama 'Admin'.
*   **AFTER DELETE:** Otomatis mencatat nama mahasiswa yang dihapus ke dalam tabel log sebagai arsip riwayat.

```sql
-- BEFORE DELETE: Melindungi Admin
DELIMITER $$ 

CREATE TRIGGER before_delete_mahasiswa 
BEFORE DELETE ON Mahasiswa 
FOR EACH ROW 
BEGIN 
    IF OLD.nama = 'Admin' THEN 
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Data Admin tidak boleh dihapus'; 
    END IF; 
END$$

-- AFTER DELETE: Log Penghapusan
CREATE TRIGGER after_delete_mahasiswa 
AFTER DELETE ON Mahasiswa 
FOR EACH ROW 
BEGIN
    INSERT INTO Log_Aktivitas(aktivitas) 
    VALUES(CONCAT('Mahasiswa ', OLD.nama, ' dihapus')); 
END$$ 

DELIMITER ;
```

---

## 🛠️ Cara Menghapus Trigger (Jika Diperlukan)

Jika ingin memperbarui atau menghapus trigger yang sudah terpasang, gunakan perintah berikut:

```sql
DROP TRIGGER IF EXISTS before_insert_mahasiswa;
DROP TRIGGER IF EXISTS after_insert_mahasiswa;
DROP TRIGGER IF EXISTS before_update_mahasiswa;
DROP TRIGGER IF EXISTS after_update_mahasiswa;
DROP TRIGGER IF EXISTS before_delete_mahasiswa;
DROP TRIGGER IF EXISTS after_delete_mahasiswa;
```


# Panduan Lengkap DDL (Data Definition Language)

Modul ini berfokus secara eksklusif pada DDL. Dalam sistem basis data, perintah SQL terbagi menjadi beberapa kategori, namun DDL adalah fondasi utamanya.

## Daftar Isi

1. [Konsep Dasar dan Analogi Logika](#1-konsep-dasar-dan-analogi-logika)
2. [Tipe Data Esensial](#2-tipe-data-esensial)
3. [Aturan Main (Constraints)](#3-aturan-main-constraints)
4. [Sintaks Lengkap Perintah DDL](#4-sintaks-lengkap-perintah-ddl)
5. [Panduan Menganalisis Soal Cerita (Studi Kasus)](#5-panduan-menganalisis-soal-cerita-studi-kasus)
-----

## 1\. Konsep Dasar dan Analogi Logika

DDL (Data Definition Language) adalah kumpulan perintah SQL yang bertugas sebagai **Arsitek dan Tukang Bangunan**. Tujuan utama DDL adalah membangun kerangka, mendefinisikan struktur, mengubah struktur, atau menghancurkan wadah penyimpanan data.

**Analogi Logika:**

  * **Database:** Sebuah kompleks perumahan atau lahan kosong.
  * **Table (Tabel):** Rumah-rumah yang dibangun di dalam kompleks tersebut.
  * **Column (Kolom/Atribut):** Cetak biru (blueprint) yang menentukan ruangan apa saja yang ada di dalam rumah (misal: ruang tamu, kamar tidur).
  * **Data (Isi):** Perabotan dan manusia yang mengisi rumah. (Ingat: DDL tidak mengurus isi data, pengisian data adalah tugas DML / Data Manipulation Language).

-----

## 2\. Tipe Data Esensial

Sebelum membuat kolom (ruangan), arsitek harus menentukan jenis barang apa yang boleh ditaruh di ruangan tersebut.

  * **String (Teks):**
      * `CHAR(n)`: Teks dengan panjang tetap.
      * `VARCHAR(n)`: Teks dengan panjang dinamis (menyesuaikan isi). Sangat sering digunakan untuk nama, email, nomor telepon (karena nomor telepon tidak dipakai untuk operasi matematika).
      * `TEXT`: Teks yang sangat panjang seperti deskripsi atau komentar.
  * **Numeric (Angka):**
      * `INT`: Angka bilangan bulat (contoh: umur, jumlah barang, ID).
      * `DECIMAL(m,d)` / `FLOAT`: Angka pecahan atau desimal (contoh: nilai ujian, harga).
  * **Date / Waktu:**
      * `DATE`: Format tanggal (Tahun-Bulan-Hari / YYYY-MM-DD).
  * **Boolean / Pilihan:**
      * `ENUM('opsi1', 'opsi2')`: Tipe data yang hanya menerima pilihan yang sudah ditentukan baku (contoh: 'L' atau 'P' untuk jenis kelamin).

-----

## 3\. Aturan Main (Constraints)

Constraint adalah batasan atau aturan ketat yang dipasang pada kolom untuk menjaga integritas data. Jika data yang akan dimasukkan melanggar aturan ini, database akan memblokir dan mengeluarkan error.

1.  **PRIMARY KEY (Kunci Utama):** Identitas tunggal untuk setiap baris data.
      * *Analogi:* Nomor Induk Kependudukan (NIK).
      * *Sifat:* Tidak boleh kosong (NOT NULL) dan tidak boleh ada yang kembar (UNIQUE).
2.  **FOREIGN KEY (Kunci Tamu):** Kolom yang merujuk pada Primary Key di tabel lain. Bertujuan untuk menciptakan relasi (hubungan) antar tabel.
      * *Analogi:* Fotokopi KTP milik tabel Induk yang dititipkan ke tabel Anak.
3.  **NOT NULL:** Aturan yang memaksa kolom tidak boleh dikosongkan.
      * *Analogi:* Formulir dengan tanda bintang merah, wajib diisi.
4.  **UNIQUE:** Aturan yang memastikan tidak ada data yang kembar di dalam kolom tersebut, tetapi data masih boleh dikosongkan jika tidak ada aturan NOT NULL.
      * *Analogi:* Alamat email atau nomor handphone pengguna.
5.  **CHECK:** Validasi kondisional sebelum data diizinkan masuk.
      * *Analogi:* Satpam di depan klub malam yang mengecek KTP. Jika umur di bawah 18 tahun, pengunjung dilarang masuk.
6.  **DEFAULT:** Memberikan nilai bawaan secara otomatis apabila pengguna tidak mengisi data pada kolom tersebut.
      * *Analogi:* Jika tamu tidak mengisi gelar, otomatis sistem memanggil dengan sebutan "Bapak/Ibu".

-----

## 4\. Sintaks Lengkap Perintah DDL

### A. CREATE (Membangun Struktur Baru)

Digunakan untuk membangun database atau tabel dari nol.

**Membuat Database:**

```sql
CREATE DATABASE IF NOT EXISTS nama_database;
USE nama_database;
```

*Tujuan `IF NOT EXISTS`: Mencegah error fatal jika database dengan nama tersebut sudah pernah dibuat sebelumnya.*

**Membuat Tabel Biasa:**

```sql
CREATE TABLE jurusan (
    id_jurusan INT PRIMARY KEY,
    nama_jurusan VARCHAR(50) NOT NULL UNIQUE
);
```

**Membuat Tabel dengan Foreign Key (Relasi 1:N):**
*Aturan Mutlak:* Tabel Induk (Parent) harus dieksekusi terlebih dahulu sebelum tabel Anak (Child).

```sql
CREATE TABLE mahasiswa (
    nrp VARCHAR(15) PRIMARY KEY,
    nama_mahasiswa VARCHAR(100) NOT NULL,
    tanggal_lahir DATE,
    id_jurusan INT,
    
    -- Mendeklarasikan Foreign Key untuk menyambung ke tabel jurusan
    FOREIGN KEY (id_jurusan) REFERENCES jurusan(id_jurusan)
);
```

**Membuat Junction Table (Relasi Many-to-Many):**
Digunakan ketika dua tabel saling memiliki banyak relasi (misal: Pembeli dan Produk). Harus ada tabel ketiga di tengah sebagai perantara.

```sql
CREATE TABLE transaksi (
    id_transaksi INT PRIMARY KEY,
    id_pembeli INT,
    id_produk INT,
    tanggal_transaksi DATE,
    
    -- Menarik dua Foreign Key sekaligus
    FOREIGN KEY (id_pembeli) REFERENCES pembeli(id_pembeli),
    FOREIGN KEY (id_produk) REFERENCES produk(id_produk)
);
```

### B. ALTER TABLE (Merenovasi Struktur)

Digunakan ketika tabel sudah terbentuk, tetapi ada perubahan kebutuhan sistem (menambah kolom, mengubah tipe data, atau menghapus kolom).

**Menambah Kolom Baru (ADD):**

```sql
ALTER TABLE pengguna ADD no_telpon VARCHAR(15);
```

**Menambah Aturan (ADD CHECK/CONSTRAINT):**

```sql
-- Pastikan menuliskan spasi setelah tanda hubung ganda jika menggunakan komentar
-- Menambah aturan batas umur
ALTER TABLE pengguna ADD CHECK (umur >= 18);
```

**Menghapus Kolom (DROP COLUMN):**

```sql
ALTER TABLE pengguna DROP COLUMN no_telpon;
```

**Mengubah Tipe Data Saja (MODIFY):**
*Tujuan: Kolom namanya tetap, hanya kapasitas atau tipe datanya yang diperlebar.*

```sql
ALTER TABLE pengguna MODIFY COLUMN nama_pengguna VARCHAR(200);
```

**Mengubah Nama Kolom Sekaligus Tipe Data (CHANGE):**
*Sintaks wajib: nama\_lama nama\_baru tipe\_data.*

```sql
ALTER TABLE pengguna CHANGE COLUMN no_telpon nomor_whatsapp VARCHAR(20);
```

### C. RENAME (Mengganti Nama Tabel)

```sql
ALTER TABLE pembeli RENAME TO pengguna;
```

### D. DROP vs TRUNCATE (Menghancurkan vs Membersihkan)

**DROP TABLE:**
Tujuan: Menghancurkan tabel secara total hingga kerangkanya hilang. Data dan strukturnya lenyap.

```sql
DROP TABLE transaksi;
```

**TRUNCATE TABLE:**
Tujuan: Mengosongkan seluruh isi data di dalam tabel secara cepat, namun kerangka tabel (kolom dan aturannya) tetap berdiri utuh dan siap diisi data baru.

```sql
TRUNCATE TABLE transaksi;
```

-----

## 5\. Panduan Menganalisis Soal Cerita (Studi Kasus)

Dosen sering memberikan soal ujian berupa narasi bisnis atau skenario sistem. Tugasmu adalah menerjemahkan cerita bahasa manusia menjadi sintaks DDL. Ikuti 4 langkah analisis ini:

### Langkah 1: Identifikasi Entitas (Kata Benda Utama) = Menjadi Tabel

  * **Contoh Narasi:** "Sebuah rumah sakit membutuhkan sistem untuk mencatat data Pasien dan Dokter."
  * **Analisis DDL:** Kamu harus membuat `CREATE TABLE pasien` dan `CREATE TABLE dokter`.

### Langkah 2: Identifikasi Atribut (Detail Entitas) = Menjadi Kolom & Tipe Data

  * **Contoh Narasi:** "Setiap pasien akan dicatat nomor rekam medisnya, nama lengkap, dan riwayat alerginya jika ada."
  * **Analisis DDL:** \* Nomor rekam medis -\> `VARCHAR` atau `INT`.
      * Nama lengkap -\> `VARCHAR(100)`.
      * Riwayat alergi "jika ada" artinya boleh kosong, jadi tidak perlu NOT NULL.

### Langkah 3: Identifikasi Aturan Ketat = Menjadi Constraints

  * **Contoh Narasi:** "Nomor rekam medis tidak boleh sama antar pasien. Nama wajib diisi. Pasien yang mendaftar harus berstatus warga negara WNI. Jika status tidak diisi, otomatis terdaftar sebagai pasien umum."
  * **Analisis DDL:**
      * Tidak boleh sama -\> Jadikan `PRIMARY KEY` atau `UNIQUE`.
      * Wajib diisi -\> `NOT NULL`.
      * Harus berstatus WNI -\> `CHECK (kewarganegaraan = 'WNI')`.
      * Otomatis terdaftar pasien umum -\> `DEFAULT 'Umum'`.

### Langkah 4: Identifikasi Relasi = Menjadi Foreign Key

  * **Contoh Narasi:** "Seorang dokter dapat menangani banyak pasien, tetapi satu pasien pada satu waktu hanya ditangani oleh satu dokter penanggung jawab."

  * **Analisis DDL:** Ini adalah relasi 1:N (Satu Dokter : Banyak Pasien).

      * **Aturan Eksekusi:** Buat tabel `dokter` terlebih dahulu (Parent).
      * Lalu buat tabel `pasien` (Child), dan tambahkan kolom `id_dokter` di dalam tabel `pasien`.
      * Tutup dengan `FOREIGN KEY (id_dokter) REFERENCES dokter(id_dokter)`.

  * **Contoh Narasi Relasi M:N:** "Seorang mahasiswa bisa mendaftar banyak unit kegiatan mahasiswa (UKM), dan satu UKM bisa menampung banyak mahasiswa."

  * **Analisis DDL:** Kamu wajib merancang 3 tabel. `mahasiswa`, `ukm`, dan satu Junction Table bernama `pendaftaran_ukm` yang berisi FK dari mahasiswa dan FK dari UKM.
