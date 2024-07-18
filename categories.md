---
layout: page
title: "Categories"
---

<div id="archives">
{% for category in site.categories %}
  <div class="archive-group">
    {% capture category_name %}{{ category | first }}{% endcapture %}
    <div id="#{{ category_name | slugize }}"></div>
    <p></p>

    <h3 class="category-head"><a href="#{{ category_name }}" style="color: #212228">{{ category_name }}</a></h3>
    {% for post in site.categories[category_name] %}
      <li><span>{{ post.date | date_to_string }}</span> &nbsp; <a href="{{ post.url }}">{{ post.title }}</a></li>
    {% endfor %}

  </div>
{% endfor %}
</div>
<script>
window.addEventListener('hashchange', function() {
  var categoryName = window.location.hash.substring(1);
  window.location.reload();
});
// Ambil nama kategori dari hash di URL
var categoryName = window.location.hash.substring(1);
// Jika hash tidak ditemukan, tampilkan semua kategori
if (!categoryName) {
  categoryName = 'all';
}
// Sembunyikan semua kategori kecuali yang dicari
var archiveGroups = document.getElementsByClassName('archive-group');
for (var i = 0; i < archiveGroups.length; i++) {
  var group = archiveGroups[i];
  var header = group.querySelector('.category-head');
  if (header.textContent.toLowerCase() !== categoryName.toLowerCase() && categoryName !== 'all') {
    group.style.display = 'none';
  }
}
</script>
