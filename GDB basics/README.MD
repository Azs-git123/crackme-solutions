# GDB Basics - Crackmes.one Write-up

## Informasi Challenge

- **Nama:** GDB Basics
- **Author:** Varsovie
- **Platform:** Linux x86_64
- **Bahasa:** C/C++
- **Difficulty:** 1.5

---

## Deskripsi

Challenge ini bertujuan untuk memperkenalkan penggunaan dasar **GNU Debugger (GDB)** untuk menganalisis sebuah program sederhana dan menemukan nilai yang dibutuhkan agar program menampilkan pesan keberhasilan.

---

## Analisis Awal

Pertama, dilakukan identifikasi file menggunakan perintah:

```bash
file a.out
```

Output:

```bash
a.out: ELF 64-bit LSB pie executable, x86-64, dynamically linked, not stripped
```

### Kesimpulan:

- Binary bertipe **ELF 64-bit**.
- Arsitektur **x86-64**.
- Binary **tidak di-strip**, sehingga simbol seperti `main()` masih tersedia.
- Menggunakan mekanisme **PIE (Position Independent Executable)**.

---

## Pemeriksaan Metadata

Menggunakan:

```bash
readelf -n a.out
```

Output menunjukkan informasi Build ID dan ABI Linux.

Karena binary tidak di-strip, analisis menggunakan GDB akan menjadi lebih mudah.

---

## Menjalankan Program

Program dijalankan secara langsung:

```bash
./a.out
```

Output:

```text
Enter a number: 1
I cannot provide you a flag !
```

Program meminta sebuah angka sebagai input.

---

## String Enumeration

Untuk mencari string menarik digunakan:

```bash
strings a.out
```

Ditemukan beberapa string penting:

```text
Enter a number:
There is the flag : I_LOVE_YOU
I cannot provide you a flag !
```

Dari hasil tersebut diketahui bahwa program sebenarnya dapat menampilkan flag:

```text
I_LOVE_YOU
```

Namun diperlukan kondisi tertentu agar string tersebut ditampilkan.

---

## Analisis Menggunakan GDB

### Membuka Binary

```bash
gdb a.out
```

Breakpoint dipasang pada fungsi `main`.

```gdb
(gdb) break main
(gdb) run
```

---

### Melihat Register

```gdb
(gdb) info registers
```

Perintah ini digunakan untuk melihat kondisi register saat program berhenti.

---

### Disassembly Fungsi `main`

```gdb
(gdb) disassemble main
```

Potongan kode penting:

```asm
movl   $0x2,-0x4(%rbp)
movl   $0x2,-0x8(%rbp)

cmpl   $0x13,-0x8(%rbp)
jg     <main+44>

mov    -0x4(%rbp),%eax
imul   -0x8(%rbp),%eax
mov    %eax,-0x4(%rbp)

addl   $0x1,-0x8(%rbp)
jmp    <main+22>
```

Bagian di atas merupakan sebuah loop:

```c
int result = 2;
int i = 2;

while (i <= 19) {
    result = result * i;
    i++;
}
```

Loop tersebut menghitung:

```text
2 × 2 × 3 × 4 × ... × 19
```

atau setara dengan:

```text
19!
```

---

## Pemeriksaan Kondisi Validasi

Bagian berikut digunakan untuk memeriksa input pengguna:

```asm
mov    -0xc(%rbp),%eax
cmp    %eax,-0x4(%rbp)
jne    <main+107>
```

Artinya:

```c
if (input == result)
    puts("There is the flag : I_LOVE_YOU");
else
    puts("I cannot provide you a flag !");
```

Sehingga kita harus mengetahui nilai dari:

```text
result = 2 × 2 × 3 × ... × 19
```

---

## Menggunakan GDB untuk Mengetahui Nilai

Breakpoint dipasang tepat sebelum proses perbandingan:

```gdb
(gdb) break *main+88
(gdb) run
```

Program meminta input sembarang:

```text
Enter a number: 123
```

Setelah breakpoint tercapai:

```gdb
(gdb) print /d *(int *)($rbp - 4)
```

Output:

```gdb
$3 = 219283456
```

Nilai inilah yang dibandingkan dengan input pengguna.

---

## Mendapatkan Flag

Jalankan kembali program:

```bash
./a.out
```

Masukkan:

```text
219283456
```

Output:

```text
There is the flag : I_LOVE_YOU
```

---

## Flag

```text
I_LOVE_YOU
```

---

## Kesimpulan

Challenge ini memperkenalkan beberapa konsep dasar reverse engineering menggunakan GDB, seperti:

- Melihat metadata binary menggunakan `file` dan `readelf`.
- Enumerasi string menggunakan `strings`.
- Menggunakan breakpoint pada GDB.
- Melihat register dengan `info registers`.
- Melakukan disassembly fungsi `main`.
- Memeriksa variabel di stack menggunakan GDB.
- Menentukan kondisi validasi untuk memperoleh flag.

Challenge ini sangat cocok untuk pemula yang baru mempelajari penggunaan dasar GDB dalam reverse engineering.
