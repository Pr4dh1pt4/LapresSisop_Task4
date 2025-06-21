# Task4
## A.) Implementasikan fungsi `printString`, `readString`, dan `clearScreen` di `kernel.c` yang akan menampilkan dan membaca string di layar.
  - `printString`: Menampilkan string yang diakhiri null menggunakan `int 10h` dengan `AH=0x0E`.
  - `readString`: Membaca karakter dari keyboard menggunakan `int 16h` dengan `AH=0x00` sampai Enter ditekan. Termasuk penanganan Backspace dasar.
  - `clearScreen`: Membersihkan layar dan mengatur kursor ke pojok kiri atas `(0, 0)` menggunakan `int 10h` dengan `AH=0x06` dan `AH=0x02`. Buffer video untuk warna karakter akan diubah menjadi putih.

### Kernel

#### [`kernel.asm`](./src/kernel.asm)

```asm
global _putInMemory
global _interrupt

; void putInMemory(int segment, int address, char character)
_putInMemory:
	push bp
	mov bp,sp
	push ds
	mov ax,[bp+4]
	mov si,[bp+6]
	mov cl,[bp+8]
	mov ds,ax
	mov [si],cl
	pop ds
	pop bp
	ret

; int interrupt(int number, int AX, int BX, int CX, int DX)
_interrupt:
	push bp
	mov bp,sp
	mov ax,[bp+4]
	push ds
	mov bx,cs
	mov ds,bx
	mov si,intr
	mov [si+1],al
	pop ds
	mov ax,[bp+6]
	mov bx,[bp+8]
	mov cx,[bp+10]
	mov dx,[bp+12]

intr:	int 0x00

	mov ah,0
	pop bp
	ret
```
