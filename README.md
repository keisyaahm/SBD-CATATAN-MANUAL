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
