---
layout: default
title: Главная
---

<div id="menu">
  <ul>
    <li><a href="https://kondyubaba.github.io/Design-Patterns/%D0%A1%D1%82%D1%80%D1%83%D0%BA%D1%82%D1%83%D1%80%D0%BD%D1%8B%D0%B5/%D0%90%D0%B4%D0%B0%D0%BF%D1%82%D0%B5%D1%80.md">Page 1</a></li>
    <li><a href="#page2">Page 2</a></li>
    <!-- Add more pages here -->
  </ul>
</div>
<div id="content">
</div>

<script>
  document.querySelectorAll('#menu a').forEach(a => {
  a.addEventListener('click', function (e) {
    e.preventDefault();
    fetch(e.target.getAttribute('href') + '.html') // assuming your pages are .html files
      .then(response => response.text())
      .then(html => {
        document.getElementById('content').innerHTML = html;
      });
  });
});
</script>
