# Lab8Web



## Struktur Proyek Saya

Saya menempatkan semua file di dalam folder **`lab8_php_database`** di dalam `htdocs` XAMPP saya.

```
lab8_php_database/
├── index.php         (Halaman Utama: Menampilkan Data)
├── tambah.php        (Formulir: Menambah Data Baru)
├── ubah.php          (Formulir: Mengubah Data Lama)
├── hapus.php         (Logika: Menghapus Data)
├── koneksi.php       (Konfigurasi: Menghubungkan ke Database)
├── style.css         (Tampilan: Menata Halaman)
└── gambar/           (Folder Fisik: Tempat Menyimpan File Gambar)
```

-----

## 1. Database dan Koneksi

Sebelum *coding* PHP, saya harus pastikan database dan koneksi sudah siap.

### 1.1 Menyiapkan Database (`latihan1`)

Saya menggunakan **phpMyAdmin** untuk membuat database.

Saya membuat *database* bernama **`latihan1`**. Di dalamnya, saya membuat satu tabel yaitu **`data_barang`**. Yang paling penting, saya atur kolom **`id_barang`** sebagai **Primary Key** dan **Auto Increment**. Kolom **`gambar`** saya buat sebagai `VARCHAR` (teks) karena dia hanya menyimpan nama file, bukan data gambarnya.

**Query SQL yang saya gunakan:**

```sql
CREATE DATABASE latihan1;

USE latihan11;

CREATE TABLE data_barang (
    id_barang int(10) auto_increment Primary Key,
    kategori varchar(30),
    nama varchar(50),
    gambar varchar(100),
    harga_beli decimal(10,0),
    harga_jual decimal(10,0),
    stok int(4)
);
```

### 1.2 Membuat Koneksi (`koneksi.php`)

File ini adalah *backbone* koneksi saya. Saya definisikan empat variabel untuk kredensial XAMPP saya (`localhost`, `root`, `""`, dan nama database `latihan1`). Saya menggunakan fungsi **`mysqli_connect()`** untuk mencoba menghubungkan PHP ke MySQL. Jika berhasil, variabel `$conn` akan saya gunakan di *script* PHP lain. Jika gagal, *script* akan berhenti (`die()`).

**Kodingan `koneksi.php`:**

```php
<?php
$host = "localhost";
$user = "root";
$pass = "";
$db = "latihan1";

$conn = mysqli_connect($host, $user, $pass, $db);

if (!$conn)
{
    echo "Koneksi ke server gagal: " . mysqli_connect_error();
    die();
}
?>
```

-----

## 2. Implementasi CRUD: Read, Create, Update, Delete

### 2.1 Read (Membaca Data): `index.php`

Ini adalah *homepage* aplikasi saya. Setelah meng-*include* `koneksi.php`, saya menjalankan query **`SELECT * FROM data_barang`** untuk mengambil semua data. Kemudian, saya menggunakan struktur HTML tabel. Di bagian tengah tabel, saya memasukkan *loop* PHP (`while`) dengan **`mysqli_fetch_array()`**. *Loop* ini berulang untuk mencetak baris data (`<tr>` dan `<td>`) secara dinamis. Kolom **Aksi** berfungsi sebagai navigasi ke fitur Update dan Delete.

**Kodingan `index.php`**

```php
<?php
// Bagian PHP: Logika koneksi dan pengambilan data
include("koneksi.php");

// Query untuk menampilkan semua data
$sql  = 'SELECT * FROM data_barang ORDER BY id_barang DESC'; // Tambahkan ORDER BY agar data terbaru di atas
$result  = mysqli_query($conn, $sql);
?>
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <link href="style.css" rel="stylesheet" type="text/css" />
    <title>Data Barang</title>
</head>
<body>
    <div class="container">
        <h1>Data Barang</h1>
        <a href="tambah.php" class="btn-tambah">Tambah Barang</a> 
        <div class="main">
            <table>
                <tr>
                    <th>Gambar</th>
                    <th>Nama Barang</th>
                    <th>Kategori</th>
                    <th>Harga Jual</th>
                    <th>Harga Beli</th>
                    <th>Stok</th>
                    <th>Aksi</th>
                </tr>
                <?php if($result && mysqli_num_rows($result) > 0): ?>
                <?php while($row = mysqli_fetch_array($result)): ?>
                <tr>
                    <td>
                        <?php if ($row['gambar']): ?>
                            <img src="gambar/<?= htmlspecialchars($row['gambar']);?>" alt="<?= htmlspecialchars($row['nama']);?>" style="max-width: 100px; max-height: 100px;">
                        <?php else: ?>
                            Tidak Ada Gambar
                        <?php endif; ?>
                    </td> 
                    
                    <td><?= htmlspecialchars($row['nama']);?></td>
                    <td><?= htmlspecialchars($row['kategori']);?></td>
                    <td><?= htmlspecialchars($row['harga_jual']);?></td>
                    <td><?= htmlspecialchars($row['harga_beli']);?></td>
                    <td><?= htmlspecialchars($row['stok']);?></td>
                    <td>
                        <a href="ubah.php?id=<?= $row['id_barang'];?>">Ubah</a> 
                        <a href="hapus.php?id=<?= $row['id_barang'];?>" onclick="return confirm('Yakin akan menghapus data ini?')">Hapus</a>
                    </td>
                </tr>
                <?php endwhile; else: ?>
                <tr>
                    <td colspan="7">Belum ada data di database.</td>
                </tr>
                <?php endif; ?>
            </table>
        </div>
    </div>
</body>
</html>
```

### 2.2 Create (Menambah Data): `tambah.php`

File ini memproses pembuatan data baru. **Logika PHP harus di atas HTML\!** Jika form di-*submit* (`if (isset($_POST['submit']))`), saya mengambil data dari **`$_POST`**. Untuk gambar, saya menggunakan **`$_FILES`** dan fungsi **`move_uploaded_file()`** untuk menyimpan file fisik ke folder `gambar/`. Setelah itu, saya menjalankan query **`INSERT INTO`**. Form HTML saya wajib memiliki atribut **`enctype="multipart/form-data"`** karena ada *input* file.

**Kodingan `tambah.php` (Create):**

```php
// ... Kodingan PHP di atas HTML ...
if (isset($_POST['submit']))
{
    // ... Ambil data dan proses upload gambar ...
    if ($file_gambar ['error'] == 0) 
    {
        // Pindahkan file ke folder 'gambar'
        if(move_uploaded_file($file_gambar ['tmp_name'], $destination)) 
        {
            $gambar = $filename;
        }
    }
    
    // Query INSERT
    $sql = "INSERT INTO data_barang (nama, kategori, ..., gambar) 
            VALUES ('{$nama}', '{$kategori}', ..., '{$gambar}')";
    // ... eksekusi query dan redirect ...
}
?>
<form method="post" action="tambah.php" enctype="multipart/form-data">
    <div class="input">
        <label>File Gambar</label>
        <input type="file" name="file_gambar" />
    </div>
    <input type="submit" name="submit" value="Simpan" />
</form>
```

### 2.3 Update (Mengubah Data): `ubah.php`

Ini adalah file terumit. **Pertama**, saya buat logika `if (isset($_POST['submit']))` di atas untuk menangani query **`UPDATE ... WHERE id_barang = ID`** jika ada perubahan yang disimpan. **Kedua**, saya harus mengambil data lama menggunakan `$_GET['id']` dan query `SELECT` sebelum HTML, agar form terisi. Di bagian HTML, saya menggunakan PHP di dalam atribut `value` setiap *input* (`value="<?php echo $data['nama'];?>"`) untuk menampilkan data lama. Saya juga menambahkan `input type="hidden"` untuk membawa ID barang.

**Kodingan `ubah.php` (Update):**

```php
<?php
// Bagian PHP: Logika pemrosesan dan pengambilan data (HARUS DITARUH PALING ATAS)
error_reporting (E_ALL);
include_once 'koneksi.php';

// Fungsi helper untuk menentukan opsi 'selected' pada dropdown
function is_select($val, $var) {
    if ($var == $val) return 'selected="selected"';
    return '';
}

// 1. Logika Pemrosesan Form UPDATE saat di-submit
if (isset($_POST['submit']))
{
    // Ambil data dari form
    $id = mysqli_real_escape_string($conn, $_POST['id']);
    $nama = mysqli_real_escape_string($conn, $_POST['nama']);
    $kategori = mysqli_real_escape_string($conn, $_POST['kategori']);
    $harga_jual = mysqli_real_escape_string($conn, $_POST['harga_jual']);
    $harga_beli = mysqli_real_escape_string($conn, $_POST['harga_beli']);
    $stok = mysqli_real_escape_string($conn, $_POST['stok']);
    $file_gambar = $_FILES['file_gambar'];
    $gambar  = null;

    // Proses upload gambar baru (jika ada)
    if ($file_gambar ['error'] == 0)
    {
        $filename  = str_replace(' ', '_', $file_gambar['name']);
        $destination = dirname(__FILE__) . '/gambar/' . $filename;

        if (move_uploaded_file($file_gambar['tmp_name'], $destination))
        {
            $gambar = $filename; // Simpan nama file ke database
        }
    }
    
    // Query UPDATE: bagian set data
    $sql = 'UPDATE data_barang SET ';
    $sql.= "nama = '{$nama}', kategori = '{$kategori}', ";
    $sql.= "harga_jual = '{$harga_jual}', harga_beli = '{$harga_beli}', stok = '{$stok}' ";
    
    if (!empty($gambar)) // Jika ada gambar baru, update kolom gambar
    {
        $sql.=", gambar = '{$gambar}' ";
    }
    
    $sql.= "WHERE id_barang = '{$id}'"; // Klausa WHERE untuk menentukan data
    
    $result  = mysqli_query($conn, $sql);
    
    if ($result) {
        header('location: index.php');
        exit;
    } else {
        echo "Gagal memperbarui data: " . mysqli_error($conn);
    }
}

// 2. Logika Pengambilan Data untuk ditampilkan di form
// Pastikan ID ada di URL
if (!isset($_GET['id'])) {
    header('location: index.php');
    exit;
}
$id = mysqli_real_escape_string($conn, $_GET['id']);
$sql = "SELECT * FROM data_barang WHERE id_barang = '{$id}'";
$result  = mysqli_query($conn, $sql);

if (!$result || mysqli_num_rows($result) == 0) {
    die('Error: Data tidak ditemukan.');
}
$data  = mysqli_fetch_array($result);

?>
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <link href="style.css" rel="stylesheet" type="text/css" />
    <title>Ubah Barang</title>
</head>
<body>
    <div class="container">
        <h1>Ubah Barang</h1>
        <div class="main">
            <form method="post" action="ubah.php" enctype="multipart/form-data">
                <div class="input">
                    <label>Nama Barang</label>
                    <input type="text" name="nama" value="<?php echo htmlspecialchars($data['nama']);?>" required/>
                </div>
                <div class="input">
                    <label>Kategori</label>
                    <select name="kategori" required>
                        <option <?php echo is_select($data['kategori'], 'Komputer'); ?> value="Komputer">Komputer</option>
                        <option <?php echo is_select($data['kategori'], 'Elektronik');?> value="Elektronik">Elektronik</option>
                        <option <?php echo is_select($data['kategori'], 'Hand Phone'); ?> value="Hand Phone">Hand Phone</option>
                    </select>
                </div>
                <div class="input">
                    <label>Harga Jual</label>
                    <input type="number" name="harga_jual" value="<?php echo htmlspecialchars($data['harga_jual']);?>" required/>
                </div>
                <div class="input">
                    <label>Harga Beli</label>
                    <input type="number" name="harga_beli" value="<?php echo htmlspecialchars($data['harga_beli']);?>" required/>
                </div>
                <div class="input">
                    <label>Stok</label>
                    <input type="number" name="stok" value="<?php echo htmlspecialchars($data['stok']);?>" required/>
                </div>
                <div class="input">
                    <label>File Gambar (Kosongkan jika tidak diubah)</label>
                    <input type="file" name="file_gambar" />
                    <?php if ($data['gambar']): ?>
                        <p>Gambar Saat Ini: 
                            <img src="gambar/<?php echo htmlspecialchars($data['gambar']);?>" style="max-width: 100px; max-height: 100px; display: block; margin-top: 10px;">
                        </p>
                    <?php endif; ?>
                </div>
                <div class="submit">
                    <input type="hidden" name="id" value="<?php echo htmlspecialchars($data['id_barang']);?>" />
                    <input type="submit" name="submit" value="Simpan Perubahan" />
                </div>
            </form>
        </div>
    </div>
</body>
</html>
```

### 2.4 Delete (Menghapus Data): `hapus.php`

File ini **tidak memiliki HTML**. Tugasnya sangat spesifik: mengambil ID barang dari URL (`$_GET['id']`), menjalankan query **`DELETE FROM data_barang WHERE id_barang = ID`**, dan langsung mengarahkan pengguna kembali ke `index.php` menggunakan **`header('location: index.php')`**.

**Kodingan `hapus.php` (Delete):**

```php
<?php
include_once 'koneksi.php';

// Cek apakah ID ada di URL
if (isset($_GET['id'])) {
    $id = mysqli_real_escape_string($conn, $_GET['id']);
    
    $sql = "DELETE FROM data_barang WHERE id_barang = '{$id}'"; // Query DELETE
    $result = mysqli_query($conn, $sql);

    if (!$result) {
        die("Gagal menghapus data: " . mysqli_error($conn));
    }
}

// Redirect kembali ke index.php setelah selesai
header('location: index.php');
exit;
?>
```

-----

## 3\. Styling: `style.css`

Saya menggunakan **`style.css`** untuk mempercantik tampilan agar tidak hanya *raw* HTML.

**Penjelasan Praktik:**
Saya membuat aturan CSS untuk *styling* umum (font, warna latar belakang) dan khusus. Misalnya, saya menargetkan `table` agar terlihat bersih, `a.btn-tambah` untuk tombol biru, dan elemen form (`input`, `label`) agar mudah dibaca dan diisi. File ini saya *link* di bagian `<head>` setiap file PHP yang memiliki *output* HTML.

**Kodingan style.css**
````css
/* ======================================
    CSS GLOBAL SETTINGS
    ======================================
*/
body {
    font-family: Arial, sans-serif;
    background-color: #f4f4f4;
    margin: 0;
    padding: 20px;
    display: flex;
    justify-content: center;
    align-items: flex-start;
}

.container {
    background-color: #ffffff;
    padding: 30px;
    border-radius: 8px;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
    width: 90%;
    max-width: 1000px;
}

h1 {
    color: #333;
    border-bottom: 2px solid #ccc;
    padding-bottom: 10px;
    margin-bottom: 20px;
    text-align: center;
}

/* ======================================
    LINK/TOMBOL TAMBAH DATA
    ======================================
*/
a.btn-tambah {
    display: inline-block;
    background-color: #007bff;
    color: white;
    padding: 10px 15px;
    text-decoration: none;
    border-radius: 5px;
    margin-bottom: 20px;
    font-weight: bold;
}

a.btn-tambah:hover {
    background-color: #0056b3;
}


/* ======================================
    TABLE (INDEX.PHP)
    ======================================
*/
.main table {
    width: 100%;
    border-collapse: collapse;
    margin-top: 20px;
}

.main th, .main td {
    border: 1px solid #ddd;
    padding: 10px 8px;
    text-align: left;
}

.main th {
    background-color: #e9e9e9;
    color: #333;
    text-transform: uppercase;
    font-size: 14px;
}

.main tr:nth-child(even) {
    background-color: #f9f9f9;
}

.main td img {
    /* Style untuk gambar yang ditampilkan di tabel */
    display: block;
    width: 100px; /* Atur ukuran maksimum gambar */
    height: auto;
    border-radius: 3px;
    object-fit: cover;
}

/* Style untuk kolom Aksi */
.main td:last-child a {
    text-decoration: none;
    padding: 5px 8px;
    margin-right: 5px;
    border-radius: 3px;
    color: white;
    font-size: 13px;
}

.main td:last-child a:first-child { /* Ubah */
    background-color: #28a745; 
}

.main td:last-child a:last-child { /* Hapus */
    background-color: #dc3545; 
}

.main td:last-child a:hover {
    opacity: 0.8;
}


/* ======================================
    FORM (TAMBAH.PHP & UBAH.PHP)
    ======================================
*/
.input {
    margin-bottom: 15px;
}

.input label {
    display: block;
    margin-bottom: 5px;
    font-weight: bold;
    color: #555;
}

.input input[type="text"],
.input input[type="number"],
.input input[type="file"],
.input select {
    width: 100%;
    padding: 10px;
    border: 1px solid #ccc;
    border-radius: 4px;
    box-sizing: border-box; /* Agar padding tidak menambah lebar */
    margin-top: 3px;
    font-size: 16px;
}

.input input[type="file"] {
    padding: 5px 10px;
    background-color: #eee;
}

/* Style untuk tombol submit */
.submit {
    margin-top: 25px;
}

.submit input[type="submit"] {
    background-color: #007bff;
    color: white;
    padding: 12px 20px;
    border: none;
    border-radius: 5px;
    cursor: pointer;
    font-size: 16px;
    font-weight: bold;
    transition: background-color 0.3s ease;
}

.submit input[type="submit"]:hover {
    background-color: #0056b3;
}
````
-----
