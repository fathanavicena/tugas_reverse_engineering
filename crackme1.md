## Description
Pada folder **`crackme 1`** ini, tantangan berfokus pada analisis tingkat pertama (validasi awal) yang berada di dalam fungsi pembuka program (`main`). Program meminta input dari pengguna dan langsung melakukan pengujian struktural mendasar pada panjang karakter sebelum mengizinkan alur kode masuk ke logika kalkulasi yang lebih dalam.

## Analysis
Analisis pada tingkat pertama ini dilakukan menggunakan alat disassembler **Cutter** untuk menginspeksi fungsi `main`. Di sini ditemukan mekanisme pertahanan awal berupa pembatasan panjang string input:

* **Pengecekan Konstanta:** Pada alamat memori `0x0000252e`, terdapat instruksi komparasi `cmp rax, 0x18680` yang memeriksa panjang karakter dari string input.
* **Conditional Jump (Percabangan Bersyarat):** Tepat setelah proses komparasi tersebut, terdapat instruksi lompatan bersyarat. Jika kondisi panjang string input tidak terpenuhi, alur eksekusi program dipaksa melompat masuk ke blok penanganan kesalahan (*error handling*) pada alamat `0x00002530` ke bawah.
* **Output Kesalahan:** Blok *error handling* tersebut mengeksekusi pemanggilan string untuk mencetak teks kesalahan `--ERROR--: Wrong password length!` ke layar terminal pengguna.

## Solution
Untuk menyelesaikan rintangan pada `crackme 1` ini, kita menggunakan teknik **Binary Patching** langsung pada file biner mentah melalui terminal Linux untuk memanipulasi arah aliran eksekusi (*Control Flow Modification*):

1. **Menghilangkan Jebakan Bersyarat:** Instruksi percabangan bersyarat asli di alamat `0x0000252e` dimodifikasi.
2. **Lompatan Paksa (Unconditional Jump):** Struktur kode biner ditimpa menggunakan op-code `\xeb\x23` yang merepresentasikan instruksi `jmp 0x2553`. Perubahan ini memaksa program untuk selalu melompati blok pencetakan teks error tersebut dan langsung mendarat dengan aman ke fungsi kelanjutan di alamat `0x00002553`.

Perintah penambalan (*patching*) biner yang dieksekusi melalui terminal Linux:
```bash
printf '\xeb\x23' | dd of=relax_keygen bs=1 seek=$((0x252e)) conv=notrunc
