# Ransomware

> Catatan: Proyek ini murni akademis, gunakan dengan risiko Anda sendiri. Saya tidak menganjurkan penggunaan perangkat lunak ini secara ilegal atau untuk menyerang target tanpa izin mereka sebelumnya
**Tujuan di sini adalah untuk menyebarluaskan dan mengajarkan lebih banyak tentang keamanan di dunia nyata. Ingat, keamanan selalu menjadi pedang bermata dua**

### Apa itu Ransomware?
Ransomware adalah jenis malware yang mencegah atau membatasi pengguna mengakses sistem mereka, baik dengan mengunci layar sistem maupun dengan mengunci berkas pengguna kecuali tebusan dibayarkan. Keluarga ransomware yang lebih modern, yang secara kolektif dikategorikan sebagai kripto-ransomware, mengenkripsi jenis berkas tertentu pada sistem yang terinfeksi dan memaksa pengguna membayar tebusan melalui metode pembayaran daring tertentu untuk mendapatkan kunci dekripsi.

### Ringkasan Proyek
Proyek ini bertujuan untuk membangun kripto-ransomware yang hampir berfungsi untuk tujuan pendidikan, ditulis dalam bahasa Go. Pada dasarnya, program ini akan mengenkripsi berkas Anda di latar belakang menggunakan AES-256-CFB, sebuah algoritma enkripsi yang kuat, menggunakan RSA-2048 untuk mengamankan pertukaran kunci dengan server. Ya, malware seperti Cryptolocker.

Program ini terdiri dari dua bagian utama, server dan malware itu sendiri.

Server bertanggung jawab untuk menyimpan ID dan kunci enkripsi masing-masing, yang diterima dari biner malware selama eksekusi.

Malware mengenkripsi dengan kunci publik RSA-2048 Anda muatan yang berisi id/kunci yang dibuat saat runtime, kemudian mengirimkannya ke server, tempat muatan tersebut didekripsi dengan benar dengan kunci pribadi RSA yang sesuai, lalu disimpan untuk penggunaan di masa mendatang.

### Installation

You need Go at least 1.7

```
git clone https://github.com/mauri870/ransomware.git
go get -v github.com/akavel/rsrc
cd ransomware
```
> Proyek ini menggunakan Glide untuk manajemen vendor
### Building the binaries

> JANGAN JALANKAN ransomware.exe DI MESIN PRIBADI ANDA, JALANKAN HANYA DI LINGKUNGAN PENGUJIAN!

#### Build

Membangun proyek ini membutuhkan banyak langkah, seperti pembuatan kunci RSA, membangun tiga biner, dan menyematkan berkas manifes. Jadi, mari kita serahkan `make` untuk mengerjakannya.
```
make
```
Jika Anda ingin membangun server untuk Windows dari mesin Unix, jalankan `env GOOS=windows make`

Malware akan berjalan di latar belakang. Anda dapat melihat apa yang terjadi dengan menghapus `-ldflags="-H windowsgui"` dari bagian ransomware di Makefile sebelum membangun.
#### Manually
Anda dapat menjalankan perintah yang ditentukan pada `Makefile` secara manual.
Beberapa hal yang perlu diketahui:
- Kunci RSA yang dilindungi kata sandi tidak didukung
- Berkas .syso yang dihasilkan oleh rsrc harus berada di direktori yang sama dengan `ransomware.go` selama `go build`
- Kunci harus diisi dengan benar di dalam konversi `[]byte`, bersifat privat di server dan publik di `ransomware.go`

After build, a binary called `ransomware.exe`, `server`/`server.exe` and `unlocker.exe` will be generated on the bin folder. The execution of `ransomware.exe` and `unlocker.exe` (even if it is compiled for linux/darwin) is locked to windows machines only.

By default, the server will listen on `localhost:8080`

> JANGAN JALANKAN ransomware.exe DI MESIN PRIBADI ANDA, JALANKAN HANYA DI LINGKUNGAN PENGUJIAN!

## Usage and How it Works
Silakan edit parameter di seluruh berkas untuk pengujian.
Letakkan biner di lingkungan pengujian Windows yang tepat, jalankan server dengan klik dua kali atau jalankan di terminal.
Server akan menunggu kontak malware dan menyimpan ID/kunci enkripsi.

Ketika biner `ransomware.exe` diklik dua kali, ia akan berjalan di latar belakang, menelusuri direktori yang menarik dan mengenkripsi semua berkas yang sesuai dengan ekstensi berkas tersebut menggunakan AES-256-CFB, kemudian membuatnya kembali dengan konten terenkripsi dan ekstensi khusus (.encrypted secara default), dan membuat berkas READ_TO_DECRYPT.html di desktop.

Secara teori, untuk mendekripsi berkas Anda, Anda perlu mengirimkan sejumlah BTC ke dompet penyerang, diikuti oleh kontak yang mengirimkan ID Anda (terletak pada berkas yang dibuat di desktop). Jika pembayaran Anda terkonfirmasi, penyerang kemungkinan akan mengembalikan kunci enkripsi dan `unlocker.exe` Anda, dan Anda dapat menggunakannya untuk memulihkan berkas Anda. Pertukaran ini dapat dilakukan dengan beberapa cara.

Misalkan Anda mendapatkan kembali kunci enkripsi Anda, Anda dapat mengambilnya dengan menunjuk ke url berikut:

```
curl http://localhost:8080/api/keys/:id
```
Where `:id` is your identification stored on the file on desktop. After, run on a terminal:

```
unlocker.exe decrypt yourencryptionkeyhere
```
And that's it, got your files back :smile:

## Server endpoints

Server hanya memiliki dua titik akhir.

`POST api/keys/add` - Digunakan oleh malware untuk menyimpan kunci baru. Beberapa verifikasi dilakukan, seperti verifikasi keaslian RSA. Mengembalikan 204 (konten kosong) jika berhasil atau terjadi kesalahan JSON.

`GET api/keys/:id` - ID adalah parameter 32 karakter, yang mewakili ID yang sudah disimpan. Mengembalikan JSON yang berisi kunci enkripsi atau kesalahan JSON.

## The end

Seperti yang Anda lihat, membangun ransomware yang fungsional, dengan beberapa algoritma terbaik yang ada bukanlah hal yang sulit, siapa pun dengan keterampilan pemrograman dan keamanan dapat membangunnya.
