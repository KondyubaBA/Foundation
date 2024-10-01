---
layout: default
title: Главная
---

<div id="menu">
  <ul>
    <li><a href="#page1">Page 1</a></li>
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
