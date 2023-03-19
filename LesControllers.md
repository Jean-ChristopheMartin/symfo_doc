# Les Controllers :

## Créer un Controller :

- ```php bin/console make:controller```

## Méthode AbstractController : 

### Json :

```php
use SymfonyBundleFrameworkBundleControllerAbstractController;
use SymfonyComponentHttpFoundationJsonResponse;
use SymfonyComponentRoutingAnnotationRoute;

class HomeController extends AbstractController
{
    /**
     * @Route("/")
     *
     * @return JsonReponse
     */
    public function index(): JsonResponse
    {
	$data = [...];

    //attend en second argument optionnel le code de la réponse HTTP, le code 200 étant celui par défaut
	return $this->json($data);
    }
}
```

### createNotFoundException() :

```php
    // Dans le cas où nous n'avons pas trouvé la promotion
    // dont l'id correspond à $id
    if ($promo === null)
    {
        // `throw` est un mot clé de php qui permet de générer une exception
        // L'exception aura un code réponse 404
        throw $this->createNotFoundException("La promo dont l'id est " . $id . " n'est pas définie");
    };
```

### createAccessDeniedException() :

```php
//Cette méthode fonctionne exactement de la même façon que la précédente, sauf que la réponse aura un 
//code HTTP 403. (accés interdit)
throw $this->createAccessDeniedException("Nope, vous n'avez pas le droit d'être là");

```

### redirectToRoute() :

```php
//Cette méthode permet de rediriger l’utilisateur vers une autre page de l’application. Si besoin, 
//nous lui transmettons les paramètres de route sous forme de tableau associatif.
return $this->redirectToRoute($url, $params);

// Ou return $this->redirectToRoute('promo_show', ['id' => 3]);

```

### Flash messages :

```php
$this->addFlash(
    'danger',
    "Le film que vous voulez ajouter n'existe pas"
);
```

### Les sessions :

- Ajouter :
```php
/**
 * Méthode qui ajoute un film en favori
 * 
 * @Route("/favoris/ajouter/{id}", name="app_favorite_add")
 *
 * @IsGranted("ROLE_USER")
 * @param RequestStack $requestStack
 * @param Movie $movie
 * @return Response
 */
public function add(RequestStack $requestStack, Movie $movie): Response
{

    $session = $requestStack->getSession();

    $favorites = $session->get('favorites', []);

    $favorites[$movie->getId()] = $movie;

    $session->set('favorites', $favorites);

    return $this->redirectToRoute('app_favorite_list');
}
```

- Remove :
```php
/**
 * Methode pour remove un film en favori
 * 
 * @Route("/favoris/supprimer/{id}", name="app_favorite_remove")
 *
 * @param RequestStack $requestStack
 * @param integer $id
 * @return Response
 */
public function remove(RequestStack $requestStack, int $id): Response
{
    $session = $requestStack->getSession();

    $favorites = $session->get('favorites');

    if (array_key_exists($id, $favorites)) {
        unset($favorites[$id]);
    }

    $session->set("favorites", $favorites);

    return $this->redirectToRoute('app_favorite_list');
}
```

- Clean :
```php
/**
 * Methode pour supprimer touts les favoris
 * 
 * @Route("/favoris/remove", name="app_favorite_empty")
 *
 * @param RequestStack $requestStack
 * @return Response
 */
public function empty(RequestStack $requestStack): Response
{
    $session = $requestStack->getSession();

    $session->remove('favorites');

    return $this->redirectToRoute('app_favorite_list'); 
}
```