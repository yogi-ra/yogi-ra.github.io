---
layout: page
title: "Cara Setup Environmnent Linux Fresh Install., ZSH, NVim, dan tool useful lainnya"
categories: blog misc
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

Pada post ini, saya akan gunakan untuk mencatat apa-apa saja yang sangat rekomended untuk di-<em>install</em> pada linux fresh install (debian based)

# Contents
- [oh-my-zsh](#omz)
- [pyenv](#pyenv)
- [NVim (NeoVim)](#nvim)
- [Penutup](#penutup)

# oh-my-zsh (zsh)
{: #omz}

Sebelum instalasi dilakukan, pastikan shell yang digunakan adalah zsh, jika tidak ada, install terlebih dahulu.
{: style="color: red;"}

Merupakan sebuah framework open-source untuk manajemen zsh (interactive shell yang berisi banyak gabungan fungsi shell dari bash, ksh, tcsh) yang akan membuatmu merasa seperti 10x developer wkwkwkk

Instalasinya sangat mudah yak, `sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"` atau bisa dilihat langsung [disini](https://github.com/ohmyzsh/ohmyzsh){: target="_blank"}. Tapi yang sangat menarik adalah, kita bisa kustomisasi tampilan shell sesuai dengan keinginan kita~

Berikut merupakan perubahan yang saya terapkan pada omz (oh-my-zsh) yang saya gunakan untuk menulis blog ini

![ohmyzsh modified files](/assets/img/Screenshot 2025-05-26 222630.png){: style="display:block; margin-left: auto; margin-right: auto;"}

## Perubahan pada oh-my-zsh.sh
<pre style="color: #10630d">
+
+if [ -z "$FANCY_MOTD" ]; then
+    ~/fancy-motd/motd.sh
+    export FANCY_MOTD=1
+fi
</pre>

Nah, jadi perubahan hanya terdapat pada MOTD, biar keren aja hehee, kalau penasaran dengan ini bisa kunjungi aja [disini](https://github.com/bcyran/fancy-motd){: target="_blank"}

## Perubahan pada plugins/git/git.plugin.zsh
Hmm ini ga perlu ditunjukin ya sepertinya, isinya cuman comment line gf (untuk git fetch) variable, karena saya pakai command [gf](https://github.com/1ndianl33t/Gf-Patterns){: target="blank"} untuk nyari pattern vulnerability (seperti lfi, xss, dll)

---

Mong-ngomong untuk theme pada zsh bisa didefinisikan juga pada ~/.zshrc dengan cara mengubah ZSH_THEME variable menjadi nama theme kita, contoh saya menggunakan theme yogi.zsh-theme, maka ditulis `ZSH_THEME="yogi"`

Ini merupakan theme yang saya gunakan, sedikit perubahan dari robbyrussell theme~
```bash
» cat custom/themes/yogi.zsh-theme
PROMPT="%(?:%{$fg_bold[green]%}➜ :%{$fg_bold[red]%}➜ ) %{$fg[cyan]%}%~%{$reset_color%}"
PROMPT+=' $(git_prompt_info)'
PROMPT+=$'\n'
PROMPT+='%(?:%{$fg_bold[green]%}%B» %b:%{$fg_bold[red]%}%B» %b)'

ZSH_THEME_GIT_PROMPT_PREFIX="%{$fg_bold[blue]%}git:(%{$fg[red]%}"
ZSH_THEME_GIT_PROMPT_SUFFIX="%{$reset_color%}"
ZSH_THEME_GIT_PROMPT_DIRTY="%{$fg[blue]%}) %{$fg[yellow]%}"
ZSH_THEME_GIT_PROMPT_CLEAN="%{$fg[blue]%})"
```

# pyenv
{: #pyenv}
Sebuah tool untuk python version management, ini merupakan satu tool yang masuk dalam list "I Wish I Knew this Earlier" buat mereka yang sering ngerjain projek atau task pakai python, tetapi pengen lompat-lompat versi menyesuaikan tool yang digunakan. Contoh use case: public archive tool python 2 di github

Oke, langsung aja ke cara penggunaannya, disini saya cuman pakai 1 versi, supaya ga boros penyimpananー wajar, device budget wkwkwk

Untuk instalasi bisa dilakukan secara otomatis:

```bash
curl -fsSL https://pyenv.run | bash
```

Lalu untuk mengaktifkan pyenv setiap kali login dengan meletakkan skrip dibawah pada ~/.zshrc atau ~/.bashrc tergantung shell mana yang pada user

```bash
enable-pyenv() {
    export PYENV_ROOT="$HOME/.pyenv"
    [[ -d $PYENV_ROOT/bin ]] && export PATH="$PYENV_ROOT/bin:$PATH"
    eval "$(pyenv init -)"
    eval "$(pyenv virtualenv-init -)"
}

enable-pyenv
```

Selanjutnya aktifkan secara pyenv dengan cara relog atau `source ~/.zshrc`

# NVim (NeoVim)
{: #nvim}
Nah, ini masih jadi perdebatan sih sebenarnya, karena mayoritas text editor yang digunakan adalah nano, tapi saya bisa tunjukin bagian yang paling asik jika kamu bisa ngetik menggunakan semua jari-jarimu, maka NVim adalah tool yang tepat buat kamu! (kebetulan saya belajar ngetik cepat sepuluh jari, makanya makai ini~)

![Workstation atmin dengan tmux + nvim](/assets/img/Screenshot 2025-05-27 054713.png)

Gimana workspace tmux + nvim-nya? agak intimidatif bukan? wkwkwk berasa heker~ padahal mah lagi makai text-editor aja, belum tentu jago😅

Oke, jadi apa saja fitur yang sering saya gunakan pada nvim ini dan plugin apa saja yang di-<em>install</em>? Lanjut...

Jadi saya pakai Lazy.NVim sebagai plugin manager, kita bisa melakukan instalasi banyak hal seperti linter, formatter, LSP, dan lainnya menggunakan Mason, dll

Instalasinya terbilang cukup gampang, tinggal ke [sini](https://lazy.folke.io/installation){: target="_blank"} lalu simpan file sesuai kenginan, structured setup, atau single file setup.

Setelah lazy.nvim terinstall, ini apa saja yang bisa kita lakukan, pertama buka NVim, lalu jalankan perintah `:Mason`

![Gambar tampilan mason](/assets/img/Screenshot 2025-05-27 062543.png)

Penggunaannya sangat straight-forward yah, kita bisa menjalankan `g?` untuk help, dan disana bakal terdapat list untuk instalasi dan sebagainya.. lalu jalankan `g?` kembali untuk balik dari mode help, lalu `1` `2` `3` `4` `5` untuk berpindah antara kategori plugin mana yang ingin di-<em>install</em>

Selain mason juga ada yang useful nih, bisa dicoba yaitu nvim-tree yang udah ter<em>install</em> langsung sebagai bawaan dari Lazy.NVim, untuk settingan yang saya gunakan yaitu dengan menambahkan toggle pada `<leader>n` karena leader saya menggunakan space, jadi `<space>n` untuk membuka tutup file tree.

Berikut settingan pada `~/.config/nvim/lua/config/nvim-tree.lua`

```lua
local nvim_tree = require("nvim-tree").setup({
  actions = {
    open_file = {
      quit_on_open = false,
      window_picker = {
        enable = true,
        chars = "ABCDEFGHIJKLMNOPQRSTUVWXYZ1234567890",
        exclude = {
          filetype = { "notify", "packer", "qf", "diff", "fugitive", "fugitiveblame" },
          buftype = { "nofile", "terminal", "help" },
        },
      },
    },
  },
})

vim.keymap.set("n", "<leader>n", ":NvimTreeToggle<CR>", { desc = "Toggle NvimTree" })
```
Jangan lupa juga tambahkan config pada `~/.config/nvim/init.lua` yaitu `require("config.nvim-tree")`. Berikut isi dari init.lua yang saya gunakan
```
➜  ~/.config/nvim
» cat init.lua
require("config.lazy")
require("core.options")
require("core.keymaps")
require("config.nvim-tree")
```

Lalu, berikut merupakan list command yang sering saya gunakan dan sangat berguna untuk menggunakan nvim sebagai workspace

```
gcc: comment a line
<space>fg: grep all file on current directory and subdirectories
<space>ff: find file on current directory and subdirectories
K: show help | KK: show help, and enter the manual
zz: center the cursor
<ctrl>w: change between windows
<ctrl>wl: change to right window | <ctrl>w2l: change window 2 step to the right
<ctrl>wh: change to left window | <ctrl>w2h: change window 2 step to the left
<ctrl>wq: close window 
<ctrl>wv: new vertical window (tapi kalau dipraktekkan malah jadi horizontal, bisa dicoba sendiri)
<ctrl>ws: new horizontal window (tapi kalau dipraktekkan malah jadi vertical, bisa dicoba sendiri)
```

---

Jika terdapat masalah selama instalasi, google dan AI adalah teman kita～

# Penutup
{: #penutup}
Baiklah sekian saja yang dapat saya share terkait tool-tool yang berguna untuk fresh install linuxーkedepannya list akan terus bertambah. Jika ada masukan terkait tool yang perlu ditambahkan, silahkan hubungi saya!

Terima kasih sudah membaca!

<script>
    function onHashChange() {
        const currentHash = window.location.hash;
        const target = document.getElementById(currentHash);
        target.scrollIntoView({ behavior: smooth });
    }

    window.addEventListener('hashchange', onHashChange);
</script>

