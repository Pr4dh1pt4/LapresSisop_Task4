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
