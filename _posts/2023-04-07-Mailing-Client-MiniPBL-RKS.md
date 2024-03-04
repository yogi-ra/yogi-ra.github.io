---
layout: page
title: "Mailing Client - MiniPBL Rekayasa Keamanan Siber"
categories: Python Mail
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

![logo](/assets/img/Screenshot 2023-05-15 192521.png)
Pada artikel ini, Saya akan membahas bagaimana Team MiniPBL kami membuat Mailing Client Project dengan Python.

Saat ini, dengan semakin pesatnya perkembangan teknologi, komunikasi melalui email telah menjadi salah satu cara paling umum untuk berhubungan secara elektronik. Namun, dengan meningkatnya ancaman keamanan siber, penting bagi kita untuk menjaga privasi dan kerahasiaan pesan-pesan yang kita kirimkan melalui email. Untuk itu, kami membuat Project Mailing Client untuk memenuhi tugas Mini Project Based Learning (Mini PBL), sebuah aplikasi web sederhana yang dikembangkan dengan fokus pada keamanan siber.

<div style="display: flex; justify-content: center;">
  <img src="/assets/img/symmetric_cipher.svg" alt="Symmetric Cycle">
</div>

Project Mailing Client yang kami buat memanfaatkan framework Flask dan beberapa library Python lain, seperti email, smtplib, imaplib, dan pycryptodome, untuk menyediakan layanan pengiriman dan penerimaan email yang aman. Aplikasi ini dilengkapi dengan fitur enkripsi pesan menggunakan algoritma Blowfish dan dekripsi pesan dengan menggunakan kunci yang valid.

Fitur Utama Project Mailing Client ini sebagai berikut:

- Login dan Logout
- Mengirim Email (SMTPS)
- Menerima dan Membaca Email (IMAPS)
- Enkripsi dan Dekripsi pesan dengan algoritma Blowfish

Thanks to:
- Project Manager 