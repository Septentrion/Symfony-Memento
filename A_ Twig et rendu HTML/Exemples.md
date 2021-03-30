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
    {% foreach p in person  }
    <tr>
      <td class="prenom">{{ p.prenom }}</td>
      <td class="nom">{{ p.nom }}</td>
    </tr>
    {% endforeach %}
    </tbody>
</table>
```

Filtres
```twig
<div>
  {# Affichage du nom en majuscules #}
  <p><span>{{ prenom }}</span> <span>{{ nom | upper }}</span></p>
  {# Affichage de la date de naissance au format français #}
  <p>né le {{ cateNaissance | date('d/m/Y')}}
</div>
```

Extension
```twig
{# gabarit parent (layout.html.twig ) #}
<body>
{% block header %}{% endblock %}
{% block main %}{% endblock %}
{% block footer %}{% endblock %}
```

```twig
{# gabarit enfant #}
{% extends 'layout.html.twig' %}
{% block header %}
  <nav></nav>
{% endblock %}
{% block main %}
  
{% endblock %}
{% block footer %}{% endblock %}
```

Requête secondaire
```twig
<aside>
  <h1>Denières informations</h1>
  {{ render_hinclude(controller('App\\Controller\\NewsController::latestNews'),  {
      default: 'default/news.loading.html.twig',
      limit: 10
  }) }}
</aside>
```


Requête secondaire asynchrone
```twig
<aside>
  <h1>Denières informations</h1>
  {{ render_hinclude(controller('App\\Controller\\NewsController::latestNews'),  {
      default: 'default/news.loading.html.twig',
      limit: 10
  }) }}
</aside>
```
