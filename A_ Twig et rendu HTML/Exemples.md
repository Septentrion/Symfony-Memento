# Exemples Twig

1. Affichage d'une variable
```twig
<div class="fiche">
  <p class="prenom">{{ prenom }}</p>
  <p class="nom">{{ nom }}</p>
</div>
```

2. Affichage d'un tableau
```twig
<table class="fiche">
  <thead>
    <tr>
      <td>Prenom</td>
      <td>Nom</td>
    </tr>
  </thead>
  <tbody>
    {% for p in person  }
    <tr>
      <td class="prenom">{{ p.prenom }}</td>
      <td class="nom">{{ p.nom }}</td>
    </tr>
    {% endfor %}
    </tbody>
</table>
```

3. Filtres
```twig
<div>
  {# Affichage du nom en majuscules #}
  <p><span>{{ prenom }}</span> <span>{{ nom | upper }}</span></p>
  {# Affichage de la date de naissance au format français #}
  <p>né le {{ cateNaissance | date('d/m/Y')}}
</div>
```

4. Inclusion
```twig
{# injection d'un gabarit auquel est transmis la variable `limit` — instruction #}
{% include 'footer.html.twig' with {'limit': 3} %}
{# injection d'un gabarit auquel est transmis la variable `limit` et un thème `dark` — fonction #}
{% include('footer.html.twig', {'limit': 3}) | dark %}
```

5. Extension
```twig
{# gabarit parent (layout.html.twig ) #}
<body>
{# Les blocs sont vide a priori #}
{% block header %}{% endblock %}
{% block main %}{% endblock %}
{# Bloc contenant de l'information #}
{% block footer %}<div>...</div>{% endblock %}
```

```twig
{# gabarit enfant #}
{% extends 'layout.html.twig' %}
{% block header %}
  <nav></nav>
{% endblock %}
{% block main %}
<section> ... </section>
{% endblock %}
{% block footer %}
{# Conservation du contenu du bloc parent #}
{{ parent() }}
{% endblock %}
```

6. Requête secondaire
```twig
<aside>
  <h1>Denières informations</h1>
  {{ render(controller('App\\Controller\\NewsController::latestNews'),  {
      limit: 10
  }) }}
</aside>
```


7. Requête secondaire asynchrone
```twig
<aside>
  <h1>Denières informations</h1>
  {{ render_hinclude(controller('App\\Controller\\NewsController::latestNews'),  {
      default: 'default/news.loading.html.twig',
      limit: 10
  }) }}
</aside>
```
