---
title: "Ejercicios del Documento de inteligencia de Azure\_AI"
permalink: index.html
layout: home
---

# Ejercicios del Documento de inteligencia de Azure AI

Los siguientes ejercicios están diseñados como apoyo para los módulos de Microsoft Learn.


% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/Exercises'" %} {% for activity in labs  %}
- [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) {% endfor %}
