---
title: "Ejercicios de Documento de inteligencia de Azure\_AI"
permalink: index.html
layout: home
---

# Ejercicios de Documento de inteligencia de Azure AI

Los siguientes ejercicios están diseñados como apoyo para los módulos de Microsoft Learn.


{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/Exercises'" %} {% for activity in labs  %} {% if activity.url contains 'ai-foundry' %} {% continue %} {% endif %}
- [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) {% endfor %}
