## Description
Pada folder **`crackme 2`** ini, analisis berfokus pada mekanisme pertahanan tingkat kedua yang terletak di dalam fungsi kalkulasi terpisah. Setelah program berhasil melewati validasi panjang karakter awal, program akan mengirimkan data input *username* ke dalam algoritma pemrosesan matematika berbasis iterasi untuk membentuk nilai kunci.

## Analysis
Analisis mendalam pada fungsi raksasa `fcn.00002880` (berukuran 2868 byte) menggunakan fitur *Graph View* di **Cutter** mengungkap algoritma pembuatan serial karakter demi karakter (*looping extraction*):

* **Pengambilan Karakter Tunggal:** Program mengambil nilai ASCII dari tiap huruf pada string input menggunakan instruksi `movsx eax, byte [rax]`.
* **Operasi Perkalian Khas (Obfuscation):** Setiap nilai ASCII huruf tersebut dikalikan secara konstan dengan bilangan hexadecimal `0x539` (atau bernilai `1337` dalam format desimal) melalui perintah `imul eax, eax, 0x539`.
* **Akumulasi Nilai:** Hasil perkalian konstan tersebut terus ditambahkan ke dalam sebuah register akumulator (`add ecx, eax`) hingga seluruh huruf selesai diproses di dalam lingkaran *looping*.

## Solution
Untuk mensimulasikan dan memecahkan algoritma matematika tingkat kedua yang diterapkan oleh program, kita dapat membuat sebuah skrip otomatisasi (*Keygen Solver*) menggunakan bahasa Python:

```python
def solve_crackme_2(username):
    accumulator = 0
    # Simulasi logika assembly: Ekstraksi dan perkalian karakter
    for char in username:
        ascii_val = ord(char)
        # imul eax, eax, 0x539 (1337 desimal)
        calculated = ascii_val * 1337
        # add ecx, eax
        accumulator += calculated
    return accumulator

# Contoh pengujian pembuatan serial
username_input = "fathana"
key_result = solve_crackme_2(username_input)

print(f"[*] Input Username: {username_input}")
print(f"[*] Hasil Keygen  : {key_result}")
