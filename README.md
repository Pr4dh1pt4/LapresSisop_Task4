# Task 4 (LilHabOS)
## A.) Implementasi fungsi `printString`, `readString`, dan `clearScreen` di `kernel.c` yang akan menampilkan dan membaca string di layar.
  - `printString`: Menampilkan string yang diakhiri null menggunakan `int 10h` dengan `AH=0x0E`.
  - `readString`: Membaca karakter dari keyboard menggunakan `int 16h` dengan `AH=0x00` sampai Enter ditekan. Termasuk penanganan Backspace dasar.
  - `clearScreen`: Membersihkan layar dan mengatur kursor ke pojok kiri atas `(0, 0)` menggunakan `int 10h` dengan `AH=0x06` dan `AH=0x02`. Buffer video untuk warna karakter akan diubah menjadi putih.

#### [`printString`](./src/printString)

```
void printString(char *str) {
  int i = 0;
  while (str[i] != '\0') {
    if (str[i] == '\n') {
      cursorRow++;

      if (cursorRow >= SCREEN_HEIGHT) {
        clearScreen();
      }
      cursorCol = 0;
    } else {
      putChar(str[i]);
    }

    i++;
  }

  updateCursorPos();
}
```

#### [`readString`](./src/readString)

```
void readString(char *buf) {
  int i = 0;
  char c = 0;
  clear((byte *)buf, 128);

  while (1) {
    c = interrupt(0x16, 0x0000, 0, 0, 0) & 0xFF;

    if (c == '\r') {
      buf[i] = '\0';
      printString("\n");
      break;
    } else if (c == '\b') { // Backspace
      if (i > 0 && cursorCol > SHELL_OFFSET) {
        i--;
        cursorCol--;
        updateCursorPos();
        putChar(' ');
        cursorCol--;
        updateCursorPos();
      }
    } else {
      buf[i++] = c;
      putChar(c); // Echo character
    }
  }
}
```

#### [`clearScreen`](./src/clearScreen)

```
void clearScreen() {
  int i;
  for (i = 0; i < SCREEN_WIDTH * SCREEN_HEIGHT; i++) {
    putInMemory(0xB800, i * 2, ' ');
    putInMemory(0xB800, i * 2 + 1, 0x0F);
  }

  cursorCol = 0;
  cursorRow = 0;

  interrupt(0x10, 0x06 << 8 | 0x00, 0x00, 0x00, 0x18 << 8 | 0x4F);

  interrupt(0x10, 0x02 << 8 | 0x00, 0, 0, 0);

  updateCursorPos();
}
```


## B.) Implementasi fungsi-fungsi di [`std_lib.h`](./include/std_lib.h) dalam [`std_lib.c`](./src/std_lib.c).
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

## C.) Implementasi perintah `echo` dan `grep`
