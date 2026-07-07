# tugas_reverse_engineering
## Description
Tantangan ini bernama `relax_keygen`, sebuah program *Crackme* berbasis Linux (ELF). Program ini meminta input data kredensial dari pengguna, lalu melakukan validasi internal untuk menentukan apakah kombinasi data tersebut valid atau tidak. Jika validasi gagal, program akan memunculkan pesan kesalahan panjang karakter.

## Analysis
Proses analisis biner dilakukan menggunakan alat disassembler **Cutter (Rizin)**. Dari penelusuran aliran grafik fungsi (*Graph View*), ditemukan struktur utama sebagai berikut:

1. **Jebakan Validasi Awal (`main`):**
   Fungsi `main` pada alamat `0x0000252e` melakukan pemeriksaan panjang input terhadap nilai batas `0x18680`. Jika tidak lolos, alur program langsung diarahkan menuju blok pencetakan pesan kesalahan `--ERROR--: Wrong password length!`.

2. **Algoritma Utama Fungsi Raksasa (`fcn.00002880`):**
   Logika verifikasi yang sesungguhnya disembunyikan di dalam fungsi `fcn.00002880` berukuran 2868 byte. Fungsi ini mengiterasi string karakter demi karakter menggunakan *looping*:
   * Mengambil satu per satu nilai ASCII dari karakter input menggunakan instruksi `movsx eax, byte [rax]`.
   * Mengalikan nilai ASCII tersebut dengan konstanta bilangan hexadecimal `0x539` (atau **1337** dalam desimal) lewat instruksi `imul eax, eax, 0x539`.
   * Menambahkan hasil perkalian ke register akumulator.
   * Nilai akumulasi akhir inilah yang menjadi kunci jawaban (serial) yang valid.

## Solution
Berdasarkan hasil analisis algoritma perkalian karakter di atas, solusi pencarian kunci jawaban asli dapat diotomatisasi menggunakan skrip Python (*Keygen Solver*) berikut:

```python
def solve_keygen(username):
    accumulator = 0
    # Iterasi setiap karakter dari username seperti pada fungsi fcn.00002880
    for char in username:
        ascii_val = ord(char)
        # imul eax, eax, 0x539 (1337 desimal)
        calculated = ascii_val * 1337
        # add ecx, eax
        accumulator += calculated
    return accumulator

# Contoh penggunaan menggunakan input nama kamu
username_input = "fathana"
key_solution = solve_keygen(username_input)

print(f"[*] Username : {username_input}")
print(f"[*] Serial/PW: {key_solution}")
