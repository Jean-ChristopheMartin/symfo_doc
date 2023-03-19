# Twig :

## Nommage :

L’ensemble de nos templates seront stockés dans notre dossier templates mais pas n’importe comment… pour chaque contrôleur, nous allons créer un dossier du même nom.

Si nous avons un contrôleur nommé PromoController, alors nous créerons un dossier promo dans notre dossier templates. C’est à l’intérieur de ce dernier que nous stockerons tous nos templates relatifs aux méthodes de notre contrôleur PromoController.

Il est également recommandé d’utiliser le snake_case pour le nommage de nos templates.

## Syntaxe :  

- ForEach :

```twig
{% for key, item in items %}
// les instructions
{% endblock %}
```

- For :

```twig
{% for i in 0..10 %}
    * {{ i }}
{% endfor %}
```

-If :
```twig
{% if.. %}
// les instructions
{% elseif.. %}
// les instructions
{% else %}
// les instructions
{% endif %}
```

- Date :

```twig
{{post.createdAt|format_datetime(locale='fr')}}
```

```twig
{{review.watchedAt|date("d/m/Y")}}
```

- infos en session :

```twig
{% if app.session.get('favorites')[movie.id] is defined %}
```

-infos routes :

```twig
{% if "back" in app.request.get("_route") %}
{% if "movie" in app.request.get("_route") %} active {% endif %}
```

-flash messages

```twig
{% for label, messages in app.flashes %}
    {% for message in messages %}
        <div class="alert alert-{{label}} alert-dismissible fade show" role="alert">
            <strong>{{message}}</strong>
            <button type="button" class="btn-close" data-bs-dismiss="alert" aria-label="Close"></button>
        </div>
    {% endfor %}
{% endfor %}
```

-Si identifié :

```twig
{% if is_granted('IS_AUTHENTICATED_FULLY') %}
```

-Si role :

```twig
{% if is_granted('ROLE_MODERATOR') %}
```

-Si Voter autaurisation :

```twig
{% if is_granted("question_edit", question) %}
```

-include frag :

```twig
{% include "fragments/_rating.html.twig" %}
```

