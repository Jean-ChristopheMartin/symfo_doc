# API 

## Commande

- Créer un controller API :

``` php bin/console make:controller --no-template ```

## Status code

- `200 OK` - Response to a successful GET, PUT, PATCH or DELETE. Can also be used for a POST that doesn't result in a creation.
- `201 Created` - Response to a POST that results in a creation. Should be combined with a Location header pointing to the location of the new resource
- `204 No Content` - Response to a successful request that won't be returning a body (like a DELETE request)
- `304 Not Modified` - Used when HTTP caching headers are in play
- `400 Bad Request` - The request is malformed, such as if the body does not parse
- `401 Unauthorized` - When no or invalid authentication details are provided. Also useful to trigger an auth popup if the API is used from a browser
- `403 Forbidden` - When authentication succeeded but authenticated user doesn't have access to the resource
- `404 Not Found` - When a non-existent resource is requested
- `405 Method Not Allowed` - When an HTTP method is being requested that isn't allowed for the authenticated user
- `410 Gone` - Indicates that the resource at this end point is no longer available. Useful as a blanket response for old API versions
- `415 Unsupported Media Type` - If incorrect content type was provided as part of the request
- `422 Unprocessable Entity` - Used for validation errors
- `429 Too Many Requests` - When a request is rejected due to rate limiting

## Les grands principes d'une Api rest :

1. Client-serveur: L'architecture est basée sur un modèle client-serveur, où le client envoie une requête au serveur et le serveur renvoie une réponse.

2. Étatlessness: Le serveur ne doit pas mémoriser l'état de la session entre les requêtes du client. Toutes les informations nécessaires pour traiter une requête doivent être incluses dans la requête elle-même.

3. Cacheability: Les réponses à une requête peuvent être mises en cache pour améliorer les performances.

4. Design pour l'interopérabilité: REST est un style d'architecture plutôt qu'un protocole standardisé, ce qui signifie qu'il est conçu pour être interopérable entre différents systèmes.

5. Utilisation des verbes HTTP: Les verbes HTTP tels que GET, POST, PUT, PATCH, DELETE, etc. sont utilisés pour décrire les actions à effectuer sur les ressources.

6. Utilisation des codes de statut HTTP: Les codes de statut HTTP sont utilisés pour informer le client de l'état de la réponse, tels que 200 OK, 404 Not Found, 500 Internal Server Error, etc.

7. Utilisation des hyperliens: Les réponses à une requête peuvent inclure des hyperliens vers d'autres ressources associées pour une navigation plus facile.

8. Format de données standardisé: Les données sont généralement échangées en utilisant un format standard, tel que JSON ou XML.

En respectant ces principes, une API REST peut être facilement utilisée et comprise par différents systèmes, ce qui la rend scalable et flexible pour les développeurs.

évluation d'une Api reste par le modèle de maturité de Richardson.

## Api crud

- List (GET) : 

```php
/**
 * Methode api pour lister les voitures
 * @Route("/api/cars", name="app_api_car_list", methods={"GET"})
 * 
 * @param CarRepository $carRepository
 * @return JsonResponse
 */
public function list(CarRepository $carRepository): JsonResponse
{
    return $this->json( $carRepository->findAll(), Response::HTTP_OK, [], ["groups" => "cars"]);
}
```

- Read (GET) : 

```php
/**
 * Methode api pour afficher une voiture
 * @Route("/api/cars/{id}", name="app_api_car_read", methods={"GET"})
 *
 * @param Car|null $car
 * @return JsonResponse
 */
public function read(?Car $car): JsonResponse
{
    if (!$car){
        return $this->json([
            "status" => "404",
            "error" => "Voiture non trouvée"
        ], Response::HTTP_NOT_FOUND );
    }

    return $this->json( $car, Response::HTTP_OK, [], ["groups" => "cars"]);
}
```

- Create (POST) : 

```php
/**
 * Methode api pour ajouter une voiture
 * @Route("/api/cars", name="app_api_car_create", methods={"POST"})
 * @isGranted("ROLE_ADMIN", message="Vous devez être un administrateur")
 * 
 * @param Request $request
 * @param SerializerInterface $serializer
 * @param ValidatorInterface $validator
 * @param CarRepository $carRepository
 * @param BrandRepository $brandRepository
 * @return JsonResponse
 */
public function create(Request $request, SerializerInterface $serializer, ValidatorInterface $validator, CarRepository $carRepository, BrandRepository $brandRepository): JsonResponse
{
    $json = $request->getContent();

    // On transforme la request en json en objet car
    $car = $serializer->deserialize($json, Car::class, 'json');

    // Erreur géré mtn avec le exception subscriber
    // On gére si je json envoyé est correcte
    try{

        $car = $serializer->deserialize($json, Car::class, 'json');

    }catch(NotEncodableValueException $e){

        return $this->json([
            "status"  => 400,
            "message" => "Json non valide"
        ], Response::HTTP_BAD_REQUEST);
    }
    
    $errors = $validator->validate($car);

    // Version courte
    // if (count($errors) > 0) {
    //     return $this->json($errors, 422);
    // }

    // Version plus détaillée
    // Validation des données voire assert entity
    if(count($errors) > 0){
        // Je créer un tableau avec mes erreurs
        $errorsArray = [];
        foreach($errors as $error){
            // A l'index qui correspond au champs mal remplis, j'y injecte le/les messages d'erreurs
            $errorsArray[$error->getPropertyPath()][] = $error->getMessage();
        }
        return $this->json($errorsArray,Response::HTTP_UNPROCESSABLE_ENTITY);
    }

    $car->setCreatedAt(new \DateTimeImmutable);
    
    // Récupération de l'ensemble des données envoyées sous forme de tableau
    $content = $request->toArray();
    
    // Récupération de l'idBrand 
    $idBrand = $content['brand'];

    // Je set la marque avec le find et l'id de la marque
    $car->setBrand($brandRepository->find($idBrand));

    $carRepository->add($car, true);
    
    // Je gére le cas ou l'id de l'auteur n'est pas valide
    // Erreur géré par la suite dans le subscriber exception
    try{

        $carRepository->add($car, true);

    }catch(NotNullConstraintViolationException $e){

        return $this->json([
            "status"  => 422,
            "message" => "Id de la marque non valide"
        ], Response::HTTP_UNPROCESSABLE_ENTITY);

    }

    return $this->json($car,
        Response::HTTP_CREATED,
    [
        "Location" => $this->generateUrl("app_api_car_read", ["id" => $car->getId()])
    ], 
    [
        "groups" => "cars"
    ]);
}
```

- Edit (PUT) : 

```php
/**
 * Methode api pour modifier une voiture
 * @Route("/api/cars/{id}", name="app_api_car_edit", methods={"PUT"})
 *
 * @param Car|null $car
 * @param Request $request
 * @param EntityManagerInterface $em
 * @param SerializerInterface $serializer
 * @param ValidatorInterface $validator
 * @return JsonResponse
 */
public function edit(?Car $car, Request $request, EntityManagerInterface $em, SerializerInterface $serializer, BrandRepository $brandRepository, ValidatorInterface $validator): JsonResponse
{
    if (!$car){
        return $this->json([
            "status" => "404",
            "error" => "Voiture non trouvée"
        ], Response::HTTP_NOT_FOUND );
    }
    
    $json = $request->getContent();

    try{

        $editCar = $serializer->deserialize($json, Car::class, 'json', [AbstractNormalizer::OBJECT_TO_POPULATE => $car]);

    }catch(NotEncodableValueException $e){

        return $this->json([
            "status"  => 400,
            "message" => "Json non valide"
        ], Response::HTTP_BAD_REQUEST);
    }

    $errors = $validator->validate($car);

    if(count($errors) > 0){
        // Je créer un tableau avec mes erreurs
        $errorsArray = [];
        foreach($errors as $error){
            // A l'index qui correspond au champs mal remplis, j'y injecte le/les messages d'erreurs
            $errorsArray[$error->getPropertyPath()][] = $error->getMessage();
        }
        return $this->json($errorsArray,Response::HTTP_UNPROCESSABLE_ENTITY);
    }


    $car->setUpdatedAt(new \DateTimeImmutable);

    $content = $request->toArray();
    
    if (array_key_exists('brand', $content)) {

        $idBrand = $content['brand'];

        $car->setBrand($brandRepository->find($idBrand));
    }

    $em->persist($editCar);
    
    try{

        $em->flush();

    }catch(NotNullConstraintViolationException $e){

        return $this->json([
            "status"  => 422,
            "message" => "Id de la marque non valide"
        ], Response::HTTP_UNPROCESSABLE_ENTITY);

    }

    $em->persist($editCar);

    $em->flush();

    return $this->json(null,
    Response::HTTP_NO_CONTENT,
    [
        "Location" => $this->generateUrl("app_api_car_read", ["id" => $car->getId()])
    ], 
    [
        "groups" => "cars"
    ]);
}
```

- Update (PATCH) : 

```php
/**
 * Methode api pour modifier une voiture
 * @Route("/api/cars/{id}", name="app_api_car_update", methods={"PATCH"})
 *
 * @param Car|null $car
 * @param Request $request
 * @param EntityManagerInterface $em
 * @param SerializerInterface $serializer
 * @param ValidatorInterface $validator
 * @return JsonResponse
 */
public function update(?Car $car, Request $request, EntityManagerInterface $em, SerializerInterface $serializer, BrandRepository $brandRepository, ValidatorInterface $validator): JsonResponse
{
    if (!$car){
        return $this->json([
            "status" => "404",
            "error" => "Voiture non trouvée"
        ], Response::HTTP_NOT_FOUND );
    }
    
    $json = $request->getContent();

    try{

        $editCar = $serializer->deserialize($json, Car::class, 'json', [AbstractNormalizer::OBJECT_TO_POPULATE => $car]);

    }catch(NotEncodableValueException $e){

        return $this->json([
            "status"  => 400,
            "message" => "Json non valide"
        ], Response::HTTP_BAD_REQUEST);
    }

    $errors = $validator->validate($car);

    if(count($errors) > 0){
        // Je créer un tableau avec mes erreurs
        $errorsArray = [];
        foreach($errors as $error){
            // A l'index qui correspond au champs mal remplis, j'y injecte le/les messages d'erreurs
            $errorsArray[$error->getPropertyPath()][] = $error->getMessage();
        }
        return $this->json($errorsArray,Response::HTTP_UNPROCESSABLE_ENTITY);
    }


    $car->setUpdatedAt(new \DateTimeImmutable);

    $content = $request->toArray();
    
    if (array_key_exists('brand', $content)) {

        $idBrand = $content['brand'];

        $car->setBrand($brandRepository->find($idBrand));
    }

    $em->persist($editCar);

    try{

        $em->flush();

    }catch(NotNullConstraintViolationException $e){

        return $this->json([
            "status"  => 422,
            "message" => "Id de la marque non valide"
        ], Response::HTTP_UNPROCESSABLE_ENTITY);

    }

    return $this->json(null,
    Response::HTTP_NO_CONTENT,
    [
        "Location" => $this->generateUrl("app_api_car_read", ["id" => $car->getId()])
    ], 
    [
        "groups" => "cars"
    ]);
}
```

- Delete (DELETE) : 

```php
/**
 * Methode pour supprimer une voiture
 * @Route("/api/cars/{id}", name="app_api_car_delete", methods={"DELETE"})
 *
 * @param Car|null $car
 * @param EntityManagerInterface $em
 * @return JsonResponse
 */
public function delete(?Car $car, EntityManagerInterface $em): JsonResponse
{
    if (!$car){
        return $this->json([
            "status" => "404",
            "error" => "Voiture non trouvée"
        ], Response::HTTP_NOT_FOUND );
    }
    
    $em->remove($car);
    $em->flush();

    return $this->json(null, Response::HTTP_NO_CONTENT);
}
```

## API Sécurity JWT :

### Install :

- Prérequis :

  - ```make:user```
  - ```fixture```

- Install Postman :

  - ```sudo apt install snapd```
  - ```sudo snap install postman```

- Install Lexik Jwt :

  - Déchifrage : ```sudo apt-get install openssl```
  - Lexit : ```composer require lexik/jwt-authentication-bundle```
  - Générer les clefs : ```php bin/console lexikgenerate-keypair``` (yes mettre JWT_PASSPHRASE= dans le .env.local)

### Configuration :

- Sécurity.ymal :

```php
    firewalls:
        dev:
            pattern: ^/(_(profiler|wdt)|css|images|js)/
            security: false

        login:
            pattern: ^/api/login
            stateless: true
            json_login:
                check_path: /api/login_check
                success_handler: lexik_jwt_authentication.handler.authentication_success
                failure_handler: lexik_jwt_authentication.handler.authentication_failure
        api:
            pattern: ^/api
            stateless: true
            jwt: ~
```

```php
    access_control:
         - { path: ^/voiture.*(ajouter|modifier|supprimer), roles: ROLE_ADMIN }
         - { path: ^/brand, roles: ROLE_MODERATOR }

        # ! API
         - { path: ^/api/login, roles: PUBLIC_ACCESS }
         - { path: ^/api,       roles: IS_AUTHENTICATED_FULLY }
```

- Routes.ymal :
  
```php
api_login_check:
    path: /api/login_check
```

- Postman Config : 

```php
{
    "username": "user",
    "password": "user"
}
```
Header->Authorization->bearer "token généré".

- Protéger une Méthode en utilisation avec : 

```php
use  Sensio\Bundle\FrameworkExtraBundle\Configuration\IsGranted;
// Annotation
@isGranted("ROLE_ADMIN", message="Vous devez être un administrateur")
```

## Mise en cache :

### mise en place :

```php
/**
 * Methode api pour lister les voitures
 * @Route("/api/cars/pagination", name="app_api_car_listPagination", methods={"GET"})
 * 
 * @param CarRepository $carRepository
 * @return JsonResponse
 */
public function listPagination(CarRepository $carRepository, Request $request, TagAwareCacheInterface $cachePool): JsonResponse
{
    $page = $request->get('page', 1);
    $limit = $request->get('limit', 5);

    // Création d'un identifiant carsPaginated-page-limit
    $carsPaginated = "carsPaginated-".$page."-".$limit;

    // Mise en cache de l'élément function anonyme
    $carsListPaginated = $cachePool->get($carsPaginated, function (ItemInterface $item) use ($carRepository, $page, $limit) {
        echo("Element mis en cache");
        $item->tag("carsPaginatedCache");
        //$item->expiresAfter(60);  ici on peux rajouter un tps en s avant l'éxpiration du cache
        return $carRepository->getPaginatedCars($page, $limit);
    });

    return $this->json( $carsListPaginated, Response::HTTP_OK, [], ["groups" => "cars"]);
}
```
Dans certaints cas attention au lazy loading de doctrine!

### Commande pour vider les cache :

Attention avec la persistance des données bien a bien gérer sur la modification des données pour garder des données exacte!

```php bin/console cache:clear```


```php
/**
 * Methode pour supprimer une voiture
 * @Route("/api/cars/{id}", name="app_api_car_delete", methods={"DELETE"})
 *
 * @param Car|null $car
 * @param EntityManagerInterface $em
 * @return JsonResponse
 */
public function delete(Car $car, EntityManagerInterface $em, TagAwareCacheInterface $cachePool): JsonResponse
{       
    // Je supprime la mise en cache pour ne pas faire persisté les éléments supprimés
    $cachePool->invalidateTags(["carsPaginatedCache"]);

    $em->remove($car);
    $em->flush();

    return $this->json(null, Response::HTTP_NO_CONTENT);
}
```

## API Autodécouvrable :

### Installez HATEOAS et JMSSerializer :

```composer require willdurand/hateoas-bundle```

recipes : yes

Permet 'avoir les fichier pour configurer HATEOAS qui a besoin d'un serializer un peux plus puissant que celui de symfo

### Configuration

- jms_serializer.yaml :

```php
# config\packages\jms_serializer.yaml

jms_serializer:
    visitors:
        xml_serialization:
            format_output: '%kernel.debug%'
    property_naming:
        id: jms_serializer.identical_property_naming_strategy
```

Le début devrait normalement être présent, mais nous allons ajouter la propriété property_naming .

Celle-ci nous permet de continuer à utiliser le camel case dans nos appels ( coverText et pas cover_text  ), comme nous l’a proposé Symfony par défaut jusqu’à présent.

- Remplacer les use du serializer de symfo :

```php
// Supprimez cette ligne :
use Symfony\Component\Serializer\SerializerInterface;

// Ajoutez celle ci :
use JMS\Serializer\Serializer;
```

- Gestion des groups :

```php
use JMS\Serializer\SerializationContext;
$context = SerializationContext::create()->setGroups(['getBooks']);
$jsonBook = $serializer->serialize($book, 'json', $context);
return new JsonResponse($jsonBook, Response::HTTP_OK, [], true);
```
Pour faire simple, JMS crée un objet SerializationContext  et lui passe le groupe : c'est le même principe qu'avant, mais avec une écriture différente.

Dans l'entité :

```php
// Annotation à supprimer
use Symfony\Component\Serializer\Annotation\Groups;

// Annotation à rajouter :
use JMS\Serializer\Annotation\Groups;
```

- Gestion PUT PATCH :

```php
$editCar = $serializer->deserialize($json, Car::class, 'json', [AbstractNormalizer::OBJECT_TO_POPULATE => $car]);
```
ne fonctionne plus on ne peux pas deserializer dans un objet avec JMS enfin si mais voire doc : https://jmsyst.com/libs/serializer/master/cookbook/object_constructor

À la mano : 

```php
$newBook = $serializer->deserialize($request->getContent(), Book::class, 'json');
$currentBook->setTitle($newBook->getTitle());
$currentBook->setCoverText($newBook->getCoverText());
```

## Mettre en place HATEOAS :

- Sur l'entité :

```php
use Hateoas\Configuration\Annotation as Hateoas;
/*
 * @Hateoas\Relation(
 *      "delete",
 *      href = @Hateoas\Route(
 *          "deleteBook",
 *          parameters = { "id" = "expr(object.getId())" },
 *      ),
 *      exclusion = @Hateoas\Exclusion(groups="getBooks", excludeIf = "expr(not is_granted('ROLE_ADMIN'))"),
 * )
 *
 * @Hateoas\Relation(
 *      "update",
 *      href = @Hateoas\Route(
 *          "updateBook",
 *          parameters = { "id" = "expr(object.getId())" },
 *      ),
 *      exclusion = @Hateoas\Exclusion(groups="getBooks", excludeIf = "expr(not is_granted('ROLE_ADMIN'))"),
 * )
 *
 */
#[ORM\Entity(repositoryClass: BookRepository::class)]
#[ApiResource()]
class Book
```
