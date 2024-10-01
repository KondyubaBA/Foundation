---
layout: default
title: Главная
---
<link rel="stylesheet" href="style.css">

<div class="container">
  <nav>
    <ul>
      {% for page in site.pages %}
        {% if page.url != '/' %}
          <li><a href="{{ page.url }}">{{ page.title }}</a></li>
        {% endif %}
      {% endfor %}
    </ul>
  </nav>
  <main>
    {% for page in site.pages %}
      {% if page.url == page.url %}
        <h1>{{ page.title }}</h1>
        {{ content }}
      {% endif %}
    {% endfor %}
  </main>
</div>
