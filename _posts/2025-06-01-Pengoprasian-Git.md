---
layout: page
title: "Pengoprasian Git"
categories: blog tools
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

Pada blog post ini, saya akan gunakan untuk membagikan dan mencatat bagaimana cara mengoperasikan git

# Contents
- [Apa itu Git?](#apaitugit)
- [Terms yang berkaitan dengan Git](#gitterms)
- [Instalasi Git](#instalasi)
- [Perintah Git](#commands)
- [Penutup](#penutup)

# Apa itu Git?
{: #apaitugit}

Git merupakan alat untuk melakukan kontrol versi dari suatu _source code_.. Emang apa fungsinya kontrol versi dari _source code_??

Kita semua pasti pernah melakukan perubahan pada suatu file, tapi pas lagi di tengah melakukan perubahan kepikiran, `eh, kayaknya perubahan sebelumnya lebih bagus`, tanpa disadari ternyata udah ga bisa _undo_ dan akhirnya hanya bisa meratapi layar selama beberapa waktu😅 
Dari pada kesusahan begini, mending dari awal udah siapin pencegahan, nah.. oleh karena itu pada blog ini saya akan memberitahu bagaimana pengoperasian git...

Jadi git itu sendiri fungsi utamanya ada 2:
 - Melihat perbedaan dari _source code_ sebelum dirubah dan setelah dirubah
 - Mengembalikan _source code_ ke sebelum perubahan diterapkan

Diantara teman saya, sering ketukar-tukar nih antara `Git` sama `Github`.. jadi keduanya itu berbeda yak.. `Git` merupakan alat untuk mengelola versi _source code_, sedangkan `GitHub` merupakan online platform untuk menyimpan, berbagi, dan kolaborasi _source code_ yang dikelola versinya menggunakan `Git` dan terdiri atas banyak folder project (project ini biasa disebut repository).. Jadi jangan sampai kebalik lagi~

# Terms yang berkaitan dengan Git
{: #gitterms}

- [Repository](#repositori)
- [Commit](#commit)
- [Branch](#branch)
- [Merge](#merge)

## Repository
{: #repositori}

Semua kode yang ingin dikelola versinya dalam satu folder (biasa disebut project) adalah definisi dari repositori. berikut contoh suatu repositori dari oh-my-zsh

<pre>
➜  ~/.oh-my-zsh git:(master)
» tree -a -L 1 /home/yogi/.oh-my-zsh --gitignore
/home/yogi/.oh-my-zsh
├── CODE_OF_CONDUCT.md
├── CONTRIBUTING.md
├── .devcontainer
├── .editorconfig
├── .git
--<em>omitted</em>--

9 directories, 11 files
</pre>

## Commit
{: #commit}

Pencatatan yang harus dilakukan dari setiap penambahan, perubahan, dan penghapusan yang terjadi pada suatu repositoriー semisal kita menambahkan file A kedalam repositori dan commit, lalu melakukan perubahan pada file A di repositori itu lalu stage dan commitー maka sudah terdapat 2 versi dari repositori tersebut yang tercatat dalam riwayat commit (committed).

Misal terdapat suatu repositori:
```
$ git status
On branch master
Your branch is up-to-date with 'origin/master'.
nothing to commit, working tree clean
```

File dan folder yang terdapat di dalam suatu repositori bisa terdiri dari:

- **Staged file**:
File yang berada pada repositori dan sudah dimasukkan ke dalam staging area (staged).

```
$ git status
On branch master
Your branch is up-to-date with 'origin/master'.
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)

    new file:   README
```

- **Modified file**:
File yang berada pada repositori dan sudah tersimpan pada riwayat commit, namun terdapat perubahan, ada 2 kemungkinan:
    - Perubahan dilakukan, file ditambahkan ke staging area untuk commit
    - Perubahan dilakukan, tetapi file belum ditambahkan ke staging area dan tidak ditambahkan ke dalam riwayat commit

```
$ git status
On branch master
Your branch is up-to-date with 'origin/master'.
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

    modified:   README

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

    modified:   CONTRIBUTING.md
```

- **Untracked file**:
File yang berada pada repositori, namun tidak dikelola oleh git yang artinya file tersebut tidak terdapat pada staging area maupun riwayat commit.

```
$ echo 'My Project' > README
$ git status
On branch master
Your branch is up-to-date with 'origin/master'.
Untracked files:
  (use "git add <file>..." to include in what will be committed)

    README

nothing added to commit but untracked files present (use "git add" to track)
```

Kita juga bisa menggunakan `git status -s` atau `git status --short` untuk menentukan status dari file yang ada di dalam sebuah repositori bisa dibedakan berdasarkan simbolnya `[kode] [nama file]`
```
$ git status -s
 M README               # File ini telah dimodifikasi, namun belum staged
MM Rakefile             # File ini telah dimodifikasi, dan staged, tetapi dimodifikasi kembali
A  lib/git.rb           # File yang baru ditambahkan ke dalam repositori
M  lib/simplegit.rb     # File yang telah dimodifikasi dan staged
?? LICENSE.txt          # Untracked file
```

---

Setiap kita melakukan commit pada repositori, maka kita akan membuat:
- commit object yang bersisi (commit, referensi tree object, author, committer)
- tree object yang berisi list konten yang mendefinisikan setiap nama file disimpan pada blob yang mana satu, dan
- blob objects dari setiap file yang ada pada suatu repositori

![commit objects](/assets/img/commit-and-tree.png)

Lebih detail lagi bisa dibaca di [sini](https://git-scm.com/book/en/v2/Git-Basics-Recording-Changes-to-the-Repository){: target="_blank"} dan di [sini](https://git-scm.com/book/en/v2/Git-Branching-Branches-in-a-Nutshell){: target="_blank"}



## Branch
{: #branch}

Tujuannya agar kita dapat mengerjakan pengembangan/perbaikan fungsi spesifik suatu projek tanpa mengganggu main/base branch yang ada (kalau dulu `main`, sekarang `master` branch). Ini merupakan "killer feature" dari git, karena desainnya yang ringan (menggunakan snapshot untuk setiap commit yang dilakukan), sehingga membuat branch dan pergantian antar branch sangat cepat, dan membuat fitur ini yang paling diandalkan dibanding VCS lain, ini bukan saya yang bilangー tapi statement dari git officialnya langsung~ 

Lebih detail lagi bisa dibaca di [sini](https://git-scm.com/book/en/v2/Git-Branching-Branches-in-a-Nutshell){: target="_blank"}

## Merge
{: #merge}

Tujuannya untuk mengimpor perubahan yang terjadi pada suatu branch yang telah dilakukan perubahan terhadap suatu branchー uhh.. mungkin bahasanya terlalu ribet...

Semisal ada suatu base branch (main), untuk menambahkan suatu fitur baru tanpa mengganggu branch main, dibuatlah branch (feature/a), setelah fitur berhasil dibuat pada (feature/a)ー maka hasil perubahan ini dapat diimpor ke base branch dalam hal ini adalah main... 

Tapi kenapa nggak jadiin aja branch (feature/a) sebagai branch utama? kalau bekerja sebagai tim, mengelola kodenya akan pusing..

Jika kita menggunakan merge, semisal ada 2 orang (A dan B), si A membuat branch baru dari (main) branch untuk membuat fitur A, dan B membuat branch baru juga untuk membuat fitur B, setetlah si A selesai, dia dapat merge ke branch (main), dan B dapat melakukan merge setelahnya ketika dia sudah selesai juga pada branch (main)ー begini kan enak...

![git merge](/assets/img/Screenshot 2025-05-31 184838.png){: style="display:block; margin-left: auto; margin-right: auto;"}

Kalau keduanya memutuskan untuk membuat branch masing2 jadi branch utama malah jadi susah yang ada.. projek gak selesai, ribut yang ada😅

Oke, sekarang udah terselesaikan gimana dua orang mengerjakan fitur yang berbeda... tapi gimana kalau misalnya masing-masing si A dan B merubah file yang sama? maka akan terjadi konflik...

![git merge conflict](/assets/img/Screenshot 2025-06-01 054739.png)

Gimana cara untuk memperbaiki konflik tersebut? kita bahas pada perintah git di bawah..

# Instalasi Git
{: #instalasi}

Instalasi sangat mudah sekali, karena hanya perlu mengunjungi [Git - Downloads](https://git-scm.com/downloads){: target="_blank"} atau `brew install git` dan `apt install git` untuk macOS dan Linux (Debian based)

# Perintah Git
{: #commands}

Saya akan menggunakan direktori berikut untuk menerapkan perintah git
```
./operasi
├── matematika.py
└── statistika.py
```

`git --version` untuk melihat versi dari git dan memastikan git telah terinstall

Setup git untuk penggunaan pribadi dengan menambahkan name dan email pada global config dengan cara:
- `git config --global user.name "Isi Nama Disini"`
- `git config --global user.email "isi_email_disini@mail.com"`


Untuk inisiasi repositori pada direktori projek yang sedang dikerjakanー lakukan `git init <path>` atau kalau projek yang ingin dijadikan repositori ada di working directory saat ini, maka bisa `git init .`

Akan muncul hasilnya seperti berikut

```
➜  ~/random/python_thing/operasi
» git init .
hint: Using 'master' as the name for the initial branch. This default branch name
hint: is subject to change. To configure the initial branch name to use in all
hint: of your new repositories, which will suppress this warning, call:
hint:
hint:   git config --global init.defaultBranch <name>
hint:
hint: Names commonly chosen instead of 'master' are 'main', 'trunk' and
hint: 'development'. The just-created branch can be renamed via this command:
hint:
hint:   git branch -m <name>
Initialized empty Git repository in /home/yogi/random/python_thing/operasi/.git/
```

Lalu lakukan command `git status` atau `git status -s` untuk melihat bagaimana status repositori sekarang, maka akan muncul seperti berikut

```
➜  ~/random/python_thing/operasi git:(master)
» git status -s
?? matematika.py
?? statistika.py
```

Terlihat terdapat untracked file ditandai dengan simbol `??`, untuk memasukkan file tersebut pada staging area agar versi file tersebut dikelola oleh git (tracked) adalah dengan cara `git add <nama_file>`, `git stage <nama_file>` atau `<nama_file>` bisa menggunakan `.` untuk memasukkan semua file yang ada pada projek ke dalam repositori

```
➜  ~/random/python_thing/operasi git:(master)
» git status -s
A  matematika.py
A  statistika.py
```

Maka akan muncul simbol `A ` menandakan file baru tersebut telah ditambahkan ke staging area, dan dapat dilakukan commit

Lalu saya akan mencoba mengedit file `statistika.py` dalam keadaan telah ditambahkan ke staging area, maka

```
» git status -s
A  matematika.py
AM statistika.py
```

Terlihat bahwa status file statistika adalah `AM` atau telah ditambahkan lalu dimodifikasi... kita juga dapat melihat apa saja perubahan yang diterapkan pada file `statistika.py` dengan perintah `git diff`

```
diff --git a/statistika.py b/statistika.py
index 104af12..a4ef10f 100644
--- a/statistika.py
+++ b/statistika.py
@@ -1,7 +1,5 @@
 #!/usr/bin/env python3
+import scipy.special as spsp

-def tambah(x, y):
-    return x + y
-
-def kali(x, y):
-    return x * y
+def binomial(n, k, p):
+    return spsp.comb(n, k=k) * (p ** k) * ((1-p) ** (n-k))
```

Maka akan terlihat line mana saja yang telah dikurangi pada old file, dan line mana saja yang ditambah pada new file, masing-masing ditandai dengan simbol `-` untuk yang dihapus dan `+` untuk yang ditambah... untuk memasukkan file `statistika.py` kembali pada staging area, bisa gunakan `git add <nama_file>` atau `git stage <nama_file>` kembali.

Selanjutnya adalah commit changes terhadap file yang telah ditambahkan pada staging area dengan cara `git commit -m "Pesan Commit"`, untuk awal repositori, biasanya orang akan memasukkan "initial commit", atau "first commit".

```
➜  ~/random/python_thing/operasi git:(master)
» git commit -m "first commit"
[master (root-commit) 82317ea] first commit
 2 files changed, 12 insertions(+)
 create mode 100644 matematika.py
 create mode 100644 statistika.py
```

Terlihat pada hasil commit tersebut terdapat hexadecimal `82317ea`, ini merupakan nilai hash dari checksum sha-1 commit object yang telah dibuat, dan ini biasa disebut `commit id`

Selanjutnya saya sudah mencoba menjalankan file `statistika.py` dengan mengimpornya.. sehingga python membuat file direktori/folder `__pycache__/` untuk dilakukan percobaan `.gitignore`

```
➜  ~/random/python_thing/operasi git:(master)
» ls
matematika.py  __pycache__  statistika.py
➜  ~/random/python_thing/operasi git:(master)
» git status -s
?? __pycache__/
```

Terlihat direktori `__pycache__/` dibuat, dan dideteksi sebagai untracked file

```
➜  ~/random/python_thing/operasi git:(master)
» cat .gitignore
__pycache__/
➜  ~/random/python_thing/operasi git:(master)
» git status -s
?? .gitignore
```

Setelah `__pycache__/` ditambahkan pada file `.gitignore` maka tidak akan tampak lagi pada repositori untuk masa yang akan datang juga😅 Jangan lupa untuk menambahkan file `.gitignore` ke dalam staging area dan commit dengan message `Add .gitignore`, pastikan semua pesan commit sesuai dengan penambahan, perubahan, atau penghapusan apa yang telah dilakukan...

---

Bagaimana dengan branching dan merge???

## Merge dengan menambahkan file baru (tidak memodifikasi base branch)

Pertama yang harus dilakukan yaitu seperti mengklon master branch dengan cara `git checkout -b feature/fisika`

```
➜  ~/random/python_thing/operasi git:(master)
» git checkout -b feature/fisika
Switched to a new branch 'feature/fisika'
➜  ~/random/python_thing/operasi git:(feature/fisika)
» ls
matematika.py  __pycache__  statistika.py
```

Selanjutnya saya akan menambahkan file `fisika.py` dan melakukan merge 
```
➜  ~/random/python_thing/operasi git:(feature/fisika)
» git status -s
?? fisika.py

➜  ~/random/python_thing/operasi git:(feature/fisika)
» git add fisika.py

➜  ~/random/python_thing/operasi git:(feature/fisika)
» git commit -m "add fisika.py"
[feature/fisika 6ae4e2d] add fisika.py
 1 file changed, 3 insertions(+)
 create mode 100644 fisika.py

➜  ~/random/python_thing/operasi git:(feature/fisika)
» git checkout master
Switched to branch 'master'

➜  ~/random/python_thing/operasi git:(master)
» git merge feature/fisika
Updating ead4ec9..6ae4e2d
Fast-forward
 fisika.py | 3 +++
 1 file changed, 3 insertions(+)
 create mode 100644 fisika.py
```

Jika teradapat anggota tim yang mengerjakan branch berbeda dan menggunakan snapshot dari base branch (master) yang sama dengan `feature/fisika`, maka tidak akan mengganggu proses merge dari anggota tim tersebut nantinyaー karena branch `feature/fisika` hanya menambahkan file, dan tidak memodifikasi file dari base branchー suatu konflik juga bisa terjadi jika penamaan suatu file dari branch berbeda yang ingin di-merge ke base branch adalah sama.

## Merge dengan memodifikasi base branch (ketika terjadi konflik)

Saya akan mencoba untuk merubah suatu fungsi dalam file yang sama dari dua branch yang berbeda...

Berikut merupakan perubahan yang saya terapkan pada branch `bugfix/matematika-perkalian`, tidak terlalu banyak perubahan, karena hanya untuk mengetes merge
- Pertama saya akan membuat 2 branch dari base branch (master)
- Membandingkan perubahan yang telah diterapkan terhadap file base branch (master) dengan `git diff`
- Melakukan merge base branch (master) terhadap kedua branch


<table style="display:block; margin-left: auto; margin-right: auto;">
    <tr>
        <th>
            Nama Branch
        </th>
        <th>
            Aksi
        </th>
        <th>
            Detail
        </th>
    </tr>
    <tr>
        <td>bugfix/matematika-perkalian</td>
        <td>Membuat branch</td>
        <td>
<pre>
➜  ~/random/python_thing/operasi git:(master)
» git checkout -b bugfix/matematika-perkalian
Switched to a new branch 'bugfix/matematika-perkalian'
</pre>
        </td>
    </tr>
    <tr>
        <td>feature/tambah-argumen-kali</td>
        <td>Membuat branch</td>
        <td>
<pre>
➜  ~/random/python_thing/operasi git:(master)
» git checkout -b feature/tambah-argumen-kali
Switched to a new branch 'feature/tambah-argumen-kali'
</pre>
        </td>
    </tr>
    <tr>
        <td>bugfix/matematika-perkalian</td>
        <td>Perubahan file</td>
        <td>
<pre>
diff --git a/matematika.py b/matematika.py
index 104af12..d3d84d9 100644
--- a/matematika.py
+++ b/matematika.py
@@ -3,5 +3,5 @@
 def tambah(x, y):
     return x + y

-def kali(x, y):
+def kali(x, y) -> int:
     return x * y

</pre>
        </td>
    </tr>
    <tr>
        <td>feature/tambah-argumen-kali</td>
        <td>Perubahan file</td>
        <td>
<pre>
diff --git a/matematika.py b/matematika.py
index 104af12..03f9f3a 100644
--- a/matematika.py
+++ b/matematika.py
@@ -3,5 +3,8 @@
 def tambah(x, y):
     return x + y

-def kali(x, y):
-    return x * y
+def kali(*args):
+    result = 0
+    for i in range(len(args)-1):
+        result += args[i] * args[i+1]
+    return result

</pre>
        </td>
    </tr>
    <tr>
        <td>master</td>
        <td>Merge bugfix/matematika-perkalian</td>
        <td>
<pre>
➜  ~/random/python_thing/operasi git:(bugfix/matematika-perkalian)
» git add matematika.py
➜  ~/random/python_thing/operasi git:(bugfix/matematika-perkalian)
» git commit -m "bug fix matematika"
[bugfix/matematika-perkalian 653f084] bug fix matematika
 1 file changed, 1 insertion(+), 1 deletion(-)
➜  ~/random/python_thing/operasi git:(bugfix/matematika-perkalian)
» git checkout master
Switched to branch 'master'
➜  ~/random/python_thing/operasi git:(master)
» git merge bugfix/matematika-perkalian
Updating 6ae4e2d..653f084
Fast-forward
 matematika.py | 2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)
</pre>
        </td>
    </tr>
    <tr>
        <td>master</td>
        <td>Merge feature/tambah-argumen-kali</td>
        <td>
<pre>
➜  ~/random/python_thing/operasi git:(master)
» git merge bugfix/matematika-perkalian
Updating 6ae4e2d..653f084
Fast-forward
 matematika.py | 2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)
➜  ~/random/python_thing/operasi git:(master)
» git merge feature/tambah-argumen-kali
Auto-merging matematika.py
CONFLICT (content): Merge conflict in matematika.py
Automatic merge failed; fix conflicts and then commit the result.
</pre>
        </td>
    </tr>
</table>

Terlihat pada kolom terakhir tabel di atas, konflik terjadi dikarenakan file base branch (maseter) dimodifikasi pada branch `bugfix/matematika-perkalian`, untuk memperbaikinya kita dapat melihat terlebih dahulu file apa saja yang terdapat konflik dengan `git status -s` atau `git status`

```
➜  ~/random/python_thing/operasi git:(master)
» git status -s
UU matematika.py

➜  ~/random/python_thing/operasi git:(master)
» git status
On branch master
You have unmerged paths.
  (fix conflicts and run "git commit")
  (use "git merge --abort" to abort the merge)

Unmerged paths:
  (use "git add <file>..." to mark resolution)
        both modified:   matematika.py

no changes added to commit (use "git add" and/or "git commit -a")
```

Ternyata file yang terjadi konflik adalah `matematika.py`, langsung saja kita lihat bagaimana isi file `matematika.py`

```
➜  ~/random/python_thing/operasi git:(master)
» cat matematika.py
#!/usr/bin/env python3

def tambah(x, y):
    return x + y

<<<<<<< HEAD
def kali(x, y) -> int:
    return x * y
=======
def kali(*args):
    result = 0
    for i in range(len(args)-1):
        result += args[i] * args[i+1]
    return result
>>>>>>> feature/tambah-argumen-kali
```

Terlihat terdapat penanda
- `<<<<<<< HEAD`: Memberitahu perubahan terakhir file `matematika.py` pada branch master
- `=======`: Pembatas antara old changes dan new changes;
- dan `>>>>>>> feature/tambah-argumen-kali`: Pembatas bawah perubahan baru/new changes

Selanjutnya saya akan memperbaiki file tersebut, lalu melakukan commit ulang

```
➜  ~/random/python_thing/operasi git:(master)
» cat matematika.py
#!/usr/bin/env python3

def tambah(x, y):
    return x + y

def kali(*args):
    result = 0
    for i in range(len(args)-1):
        result += args[i] * args[i+1]
    return result

➜  ~/random/python_thing/operasi git:(master)
» git add matematika.py

➜  ~/random/python_thing/operasi git:(master)
» git status -s
M  matematika.py

➜  ~/random/python_thing/operasi git:(master)
» git commit -m "Merge branch feature/tambah-argumen-kali"
[master f3e1f85] Merge branch feature/tambah-argumen-kali
```

```
Graph commit yang telah dilakukan

*   2025-06-01 f3e1f85 (HEAD -> master) Merge branch feature/tambah-argumen-kali (bofuryuu)
|\  
| * 2025-06-01 f1e4d2a (feature/tambah-argumen-kali) membuat fungsi kali bisa mengalikan beberapa angka (bofuryuu)
* | 2025-05-31 653f084 (bugfix/matematika-perkalian) bug fix matematika (bofuryuu)
|/  
* 2025-05-31 6ae4e2d (feature/fisika) add fisika.py (bofuryuu)
* 2025-05-31 ead4ec9 Add .gitignore (bofuryuu)
* 2025-05-31 82317ea first commit (bofuryuu)
```

Baiklah, mungkin segitu saja untuk pengoperasian git... 

Oh iya, untuk hard reset dan commit revert

| Perintah | Efek | Aman untuk remote repositori |
|:---|:---|:---|
|`git reset --hard <commit_id1>`| Jika dijalankan, menghapus commit setelah `<commit_id>`| ❌ Tidak aman|
|`git revert <commit_id>`|Jika dijalankan, menambah commit baru yang membalik commit tertentu|✅ Aman|

Kalau butuh "undo" tapi tetap mau menjaga riwayat commit, pakai `git revert`. Kalau mau benar-benar "hapus jejak", dan yakin sedang bekerja sendiri atau tahu risikonya, baru `git reset --hard`.

Mungkin bisa dicoba praktekkan~

---

# Penutup
{: #penutup}
Baiklah sekian saja yang dapat saya share terkait pengoperasian git, jika ada kesalahan dalam penyampaian mohon dimaafkan😅

Terima kasih sudah membaca!

*[VCS]: Version Control System
