---
title: Instructions hébergées en ligne
permalink: index.html
layout: home
---

# Exercices Microsoft Fabric

Les exercices suivants sont conçus pour prendre en charge les modules sur [Microsoft Learn](https://aka.ms/learn-fabric).

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/Labs'" %}
| Module | Laboratoire |
| --- | --- | 
{% for activity in labs  %}| {{ activity.lab.module }} | [{{ activity.lab.title }}{% if activity.lab.type %} - {{ activity.lab.type }}{% endif %}]({{ site.github.url }}{{ activity.url }}) |
{% endfor %}

