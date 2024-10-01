---
layout: default
title: Главная
---

<div class="container">
  <nav>
    <ul id="pages">
      {% for page in site.pages %}
        {% if page.url != '/' %}
          <li><a href="{{ page.url }}">{{ page.title }}</a></li>
        {% endif %}
      {% endfor %}
    </ul>
  </nav>
  <main>
    <div id="content"></div>
  </main>
</div>

<script>
document.getElementById('pages').addEventListener('click', function(e) {
  e.preventDefault();
  var link = e.target.closest('a');
  if (link) {
    var url = link.href;
    fetch(url)
      .then(response => response.text())
      .then(data => {
        var content = document.createElement('div');
        content.innerHTML = data;
        document.getElementById('content').innerHTML = content.innerHTML;
      });
  }
});
</script>
