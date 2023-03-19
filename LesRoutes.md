# Les Route :

## Déclarer une route :

```php
/**
 * Correspond à /blog/
 *
 * @Route("/blog", name="app_blog_list")
 */
public function list()
{
    // La valeur de l'attribut `name` permettra de générer les url correspondant à la route
    // pratique si le client décide de changer le nom des url !
    // ...

        return $this->render('blog/list.html.twig', [
        'controller_name' => 'BlogController',
    ]);
}
```

- Chemin vers cette route :
  - ```twig
    {{ path('app_blog_list') }}
    ```

## Déclarer une route avec un slug:

```php
/**
 * Correspond à /blog/*
 *
 * @Route("/blog/{slug}", name="app_blog_readBySlug")
 */
public function readBySlug($slug)
{
    // $slug sera égal à la partie dynamique de l'URL
    // si l'URL est `/blog/super-routing`, alors `$slug='super-routing'`
    // ...

        return $this->render('blog/readBySlug.html.twig', [
        'controller_name' => 'BlogController',
    ]);
}
```

## Déclarer une route avec des contraintes :

```php
/**
 *
 * @Route("/blog/article/{id}", name="app_blog_readById", requirements={"id"="\d+"})
 */
public function readById(int $id)
{
    // si l'id fournit n'est pas de type numérique, le router n'exécutera pas cette méthode
    // ...

        return $this->render('blog/readById.html.twig', [
        'controller_name' => 'BlogController',
    ]);
}
```

- Chemin vers cette route :
  - ```twig
    {{ path('app_blog_list', {id:article.id}) }}
    ```

## Déclarer une route avec des contraintes :

```php
/**
 * @Route("blog/edit/{id}", name="app_blog_edit", methods={"GET","POST"})
 */
public function edit(int $id)
{
     // cette méthode de controlleur ne sera accessible que si la requête http est effectuée avec le verbe POST ou GET

        return $this->render('blog/edit.html.twig', [
        'controller_name' => 'BlogController',
    ]);
}
```