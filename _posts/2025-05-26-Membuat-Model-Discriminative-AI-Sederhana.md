---
layout: page
title: "Membuat Model Discriminative AI Sederhana dengan XGBoost: Memperkirakan Harga Properti"
categories: AI
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

Pada post ini, saya akan membagikan apa saja yang saya dapatkan dari mempelajari tentang supervised machine learning. Bagaimana cara kerja discriminative AI? pertama, lihat pada gambar di bawah ini.

![Discriminative vs Generative AI tabel perbandingan](/assets/img/0*cO8mdESnWdg20U4g.jpg)

Yap, dari sini kita dapat menyimpulkan, semua AI itu menggunakan probabilitas untuk menentukan prediksi, discriminative model khusus untuk mengklasifikasi, generative model khusus untuk sesuai namanya, membuat text, gambar, video, audio berdasarkan pola yang ada, dan lain-lain. 

Karena kita hanya ingin membuat model untuk menentukan kemungkinan harga berdasarkan feature (input/masukan) tertentu, maka kita tidak perlu menggunakan generative, karena itu akan sangat overkill, karena untuk membuat discriminative model, dataset yang dibutuhkan sangat sedikit dibandingkan dengan generative yang membutuhkan sangat banyak dataset.

Oke, langsung saja ke pembahasan bagaimana cara membuat discriminative model untuk menentukan harga properti di spesifik lokasi tertentu.

# Contents
- [Requirements](#requirements)
- [Instalasi dan menjalankan jupyter notebook](#instalasi)
- [Train dataset](#practice)
- [Penutup](#penutup)

# Requirements

- python3-notebook
- python3 packages {pandas (untuk mengelola dataset), scikit-learn (untuk memisahkan training dan validation data), xgboost (software discriminative AI dengan gradient boosting framework yang akan digunakan)}
- dataset download [disini](/assets/data/melb_data.csv)

Jika sudah menginstall dan setup python3-notebook, lompat ke [train dataset](#practice)

# Instalasi dan menjalankan jupyter notebook
{: #instalasi}
Jupyter notebook merupakan sebuah aplikasi untuk membuat catatan yang dapat dibagikan― yang terdiri atas kode komputasi (python), deskripsi bahasa secara umum, visualiasi yang beragam seperti 3D models, charts, graphs, figures, dan interactive control.

Jadi kita bakal memanfaatkan fitur kode komputasi yang interactive dengan graphs, untuk melakukan pembuatan discriminative model ini. Oh iya, jadi fungsi interactive ini sangat berguna jika terjadi kesalahan dalam menjalankan kode, kita dapat mengulang kembali proses kode yang salah tanpa harus ribet keluar masuk file (jika coding pada file langsung). Langsung saja ke pembahasannya...

Lakukan instalasi terlebih dahulu menggunakan python3-pip

`pip install notebook` atau `pip3 install notebook`

Jika sudah terinstall, kita bisa langsung coba pada shell `jupyter notebook`

<pre>
» jupyter notebook
[I 2025-05-26 07:06:56.661 ServerApp] jupyter_lsp | extension was successfully linked.
[I 2025-05-26 07:06:56.666 ServerApp] jupyter_server_terminals | extension was successfully linked.
--<em>omitted</em>--
    Or copy and paste one of these URLs:
        http://localhost:8888/tree?token=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
        http://127.0.0.1:8888/tree?token=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
--<em>omitted</em>--
</pre>

Biasanya, setelah muncul token ini, browser akan terbuka secara otomatis, tapi jika tidak, bisa copy link tersebut. Oke, langsung saja buka pada browser link yang sudah didapatkan sebelumnya.

Jika direktori tidak sesuai dengan yang diinginkan, jalankan ulang jupyter notebook pada spesifik direktori yang diinginkan, atau bisa tambah argument pada command `jupyter notebook --notebook-dir /path/to/directory` 

Untuk mempermudah segala urusan, bisa buat file baru, dan dataset yang telah di-<em>download</em> sebelumnya ke dalam direktori yang sama.

![working directory](/assets/img/Screenshot 2025-05-26 090844.png)

Oke, langsung saja kita ke pembuatan model discriminative menggunakan XGBoost.

info lebih lanjut mengenai cara penggunaan jupyter notebook: [jupyter notebook docs](https://jupyter-notebook.readthedocs.io/en/latest/notebook.html){: target="_blank"}

# Train dataset dengan XGBoost
{: #practice}

Oke, kita langsung aja pakai jupyter yang sudah di-<em>setup</em> untuk melakukan training pada model! Jika ingin mencoba saja tanpa menulis dari awal, download ipynb [disini](/assets/data/xgboost-discriminative-property-price.ipynb)

Pertama `!pip install pandas scikit-learn xgboost` untuk instalasi dependensi.

Lalu, pada cell di bawahnya, import dependensi yang ada.
```python
import pandas as pd
from sklearn.model_selection import train_test_split
```

Selanjutnya, mengecek isi dataset seperti heading, contoh sampel isinya (head), dan lainnya dengan cara mengubah data menjadi DataFrame (df) menggunakan pd.
```python
df = pd.read_csv("melb_data.csv")
df.head()
```
untuk melihat apa saja fungsi yang bisa digunakan pada DataFrame gunakan code `dir(df)`.

Lalu kita akan mengambil feature (input/masukan) apa saja yang akan digunakan untuk memprediksi harga properti.
```python
columns_used = ['Bathroom',
 'BuildingArea',
 'Landsize',
 'Price',
 'Rooms',
 'YearBuilt']
df = df[columns_used]
```
Disini saya hanya akan mengambil 6 inputan dari sekian banyak inputan yang bisa digunakan data ini akan menjadi X atau feature, dan column Price akan jadi y atau output.

```python
cleaned_df = df.dropna(axis=1) # Melakukan drop baris jika salah satu dari 6 feature tidak ada isi (kosong)
X = cleaned_df.drop("Price", axis=1) # Menghapus kolom Price dari data yang sudah bersih dari data yang kosong
y = df.Price # Output yang akan digunakan untuk training, dalam hal ini Price
```

Selanjutnya, training data dengan membagi 30% untuk dilakukan tes pada model, 70% untuk training yang akan digunakan pada model.
```python
X_train, X_valid, y_train, y_valid = train_test_split(X, y, test_size=0.3)
```

Ini merupakan bagian intinya, yaitu melakukan training pada model menggunakan dataset yang sudah diolah menjadi feature dan output
```python
from xgboost import XGBRegressor, XGBClassifier

my_model = XGBRegressor()
my_model.fit(X_train, y_train)
```

Lalu untuk mengecek seberapa jauh rata-rata error, menggunakan MAE (Mean Absolute Error), sederhananya jika prediksi adalah 170000 dan validasinya adalah 190000, maka error yang didapat adalah 20000

```python
from sklearn.metrics import mean_absolute_error

predictions = my_model.predict(X_valid)
print("Mean Absolute Error: " + str(mean_absolute_error(predictions, y_valid)))
```
Dari hasil MAE, didapat rata-ratanya adalah 377000.

Terlihat pada gambar di bawah ini perbedaan harga hasil prediksi dibandingkan dengan hasil sesungguhnya.
![perbandingan prediksi dan hasil valid](/assets/img/Screenshot 2025-05-26 102125.png)

Tidak mengejutkan rata-rata error prediksinya sangat jauh, karena dataset yang digunakan sangat sedikit, apabila terdapat lebih banyak data, tentu akan semakin akurat.


# Penutup
{: #penutup}

Baiklah sekian saja materi yang saya pelajari dari discriminative AI untuk memprediksi harga properti di suatu lokasi. Tolong hubungi saya jika ada masukan, insight baru sangat diterima!

Terima kasih sudah membaca!

<script>
    function onHashChange() {
        const currentHash = window.location.hash;
        const target = document.getElementById(currentHash);
        target.scrollIntoView({ behavior: smooth });
    }

    window.addEventListener('hashchange', onHashChange);
</script>
