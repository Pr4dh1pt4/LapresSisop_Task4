# Task4
## A.) Implementasikan fungsi `printString`, `readString`, dan `clearScreen` di `kernel.c` yang akan menampilkan dan membaca string di layar.
  - `printString`: Menampilkan string yang diakhiri null menggunakan `int 10h` dengan `AH=0x0E`.
  - `readString`: Membaca karakter dari keyboard menggunakan `int 16h` dengan `AH=0x00` sampai Enter ditekan. Termasuk penanganan Backspace dasar.
  - `clearScreen`: Membersihkan layar dan mengatur kursor ke pojok kiri atas `(0, 0)` menggunakan `int 10h` dengan `AH=0x06` dan `AH=0x02`. Buffer video untuk warna karakter akan diubah menjadi putih.

#### [`printString`](./src/printString)

```
void printString(const char* str) {
    while (*str != '\0') {
        __asm__ __volatile__ (
            "int $0x10"
            : 
            : "a" (0x0E00 | *str), "b" (0x0007)
        );
        str++;
    }
}
```

#### [`readString`](./src/readString)

```
void readString(char* buf) {
    int idx = 0;
    char key;

    while (true) {
        __asm__ __volatile__ (
            "int $0x16"
            : "=a" (key)
            : "a" (0x0000)
        );

        switch (key) {
            case 0x08:
                if (idx > 0) {
                    idx--;
                    printString("\b \b");
                }
                break;

            case 0x0D:
                buf[idx] = '\0';
                return;

            default:
                if (idx < 127 && key >= 32 && key <= 126) {
                    buf[idx++] = key;
                    printString(&key);
                }
                break;
        }
    }
}
```

#### [`clearScreen`](./src/clearScreen)

```
void clearScreen() {
     __asm__ __volatile__ (
        "int $0x10"
        : 
        : "a" (0x0600), "b" (0x07), "c" (0x0000), "d" (0x184F)
    );

    __asm__ __volatile__ (
        "int $0x10"
        : 
        : "a" (0x0200), "b" (0x00), "d" (0x0000)
    );
}
```


## B.) Lengkapi implementasi fungsi-fungsi di [`std_lib.h`](./include/std_lib.h) dalam [`std_lib.c`](./src/std_lib.c).
  - `int div` : berfungsi untuk melakukan pembagian integer tanpa menggunakan operator `/`.
  - `int mod` : berfungsi untuk menghitung sisa pembagian (modulus) tanpa operator %
  - `void memcpy` : Fungsi ini menyalin `size` byte data dari `src` ke `dst`
  - `unsigned int strlen` : Berfungsi untuk menghitung panjang string sampai karakter null `('\0')`.
  - `bool strcmp` : Berfungsi untuk membandingkan dua string, apakah isinya sama atau tidak.
  - `void strcpy` : Berfungsi untuk menyalin isi string `src` ke `dst`.
  - `void clear` : Berfungsi untuk mengisi buffer dengan nol (0) sebanyak `size` byte.
#### [`std_lib.c`](./src/std_lib.c)

```
#include "std_lib.h"

int div(int a, int b) {
    int result = 0;
    while (a >= b) {
        a -= b;
        result++;
    }
    return result;
}


int mod(int a, int b) {
    while (a >= b) {
        a -= b;
    }
    return a;
}

void memcpy(byte* src, byte* dst, unsigned int size) {
    for (unsigned int i = 0; i < size; i++) {
        dst[i] = src[i];
    }
}

unsigned int strlen(char* str) {
    unsigned int len = 0;
    while (str[len] != '\0') {
        len++;
    }
    return len;
}

bool strcmp(char* str1, char* str2) {
    while (*str1 && *str2) {
        if (*str1 != *str2) {
            return false;
        }
        str1++;
        str2++;
    }
    return (*str1 == '\0' && *str2 == '\0');
}

void strcpy(char* src, char* dst) {
    while (*src) {
        *dst++ = *src++;
    }
    *dst = '\0';
}

void clear(byte* buf, unsigned int size) {
    for (unsigned int i = 0; i < size; i++) {
        buf[i] = 0;
    }
}
```
