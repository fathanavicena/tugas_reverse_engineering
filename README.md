# tugas_reverse_engineering
## Description
Tantangan ini adalah sebuah program *Crackme* berbasis Linux (ELF). Program ini meminta input data kredensial dari pengguna, lalu melakukan dua tingkatan validasi internal menggunakan arsitektur assembly untuk menentukan apakah kombinasi data tersebut valid atau tidak. Jika salah satu tingkatan validasi gagal, program akan menolak akses pengguna.

## Analysis
Proses analisis biner dilakukan menggunakan alat disassembler **Cutter (Rizin)** untuk memetakan struktur logika percabangan fungsi. Dari penelusuran aliran grafik, program ini memiliki dua tingkat keamanan utama:

1. **Tingkat Pertama (Validasi Panjang Input):**
   Pada fungsi pembuka, program melakukan pemeriksaan panjang karakter string input pada alamat `0x0000252e` menggunakan instruksi `cmp rax, 0x18680`. Jika ukuran string input tidak memenuhi batas konstanta tersebut, alur program akan langsung dialihkan secara bersyarat ke blok fungsi pencetakan kesalahan (*error handling*).

2. **Tingkat Kedua (Algoritma Kalkulasi Karakter):**
   Setelah lolos dari pengecekan awal, program masuk ke tingkat kedua yang berupa fungsi kalkulasi raksasa berukuran 2868 byte (`fcn.00002880`). Fungsi ini memproses string input karakter demi karakter menggunakan mekanisme *looping* internal:
   * Mengambil nilai ASCII dari karakter input menggunakan instruksi `movsx eax, byte [rax]`.
   * Mengalikan setiap nilai ASCII tersebut dengan konstanta bilangan hexadecimal `0x539` (atau `1337` dalam desimal) lewat instruksi `imul eax, eax, 0x539`.
   * Mengakumulasikan hasil perkalian ke dalam register penampung (`add ecx, eax`) sebelum dibandingkan dengan kunci jawaban di akhir fungsi.

## Solution
Berdasarkan hasil analisis tingkat pertama dan tingkat kedua di atas, kita dapat membuat sebuah skrip otomatisasi menggunakan Python (*Solver Script*) untuk mensimulasikan perhitungan nilai serial yang valid:
