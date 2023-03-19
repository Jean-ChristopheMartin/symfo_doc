# Pagination :

## Dans le Controller : 

```php
/**
 * Méthode qui affiche la liste des voitures
 * @Route("/", name="app_car_list")
 * 
 * @param CarRepository $CarRepository
 * @return Response
 */
public function list(CarRepository $carRepository, Request $request): Response
{
    // On définit le nombre d'éléments par page
    $limit = 5;
    
    // On récupére le numéro de la page
    $page = (int)$request->get("page", 1);
    
    // On récupére les voitures par pages
    $cars = $carRepository->getPaginatedCars($page, $limit);

    // On récupére le nombre total de voiture
    $totalCars = $carRepository->getTotalCars();
    
    //$cars = $carRepository->findAllCarsWithBrands();

    return $this->render('car/list.html.twig', [
        'cars' => $cars,
        'limit' => $limit,
        'page' => $page,
        'totalCars' => $totalCars,
    ]);
}
```

### Version API : (On passe limit en get)

```php
<?php 
// src\Controller\BookController.php
// ...

#[Route('/api/books', name: 'books', methods: ['GET'])]
public function getAllBooks(BookRepository $bookRepository, SerializerInterface $serializer, Request $request): JsonResponse
{
    $page = $request->get('page', 1);
    $limit = $request->get('limit', 3);
    $bookList = $bookRepository->findAllWithPagination($page, $limit);

    $jsonBookList = $serializer->serialize($bookList, 'json', ['groups' => 'getBooks']);
        
    return new JsonResponse($jsonBookList, Response::HTTP_OK, [], true);
}
```

L'URL sera donc :

```https://127.0.0.1:8000/api/books?page=3&limit=2```


## Querry builder :

```php
/**
 * Methode qui renvois le nombre de voiture par page
 *
 * @param int $page
 * @param int $limit
 * @return array
 */
public function getPaginatedCars($page, $limit): array
{
    return $this->createQueryBuilder('c')
        ->orderBy('c.id')
        ->setFirstResult(($page * $limit) - $limit) // la premiére voiture commence a 0 ex page2 : 2*5 - 5 = 5 la premiere voiture de la page 2 seras bien la 5éme version plus simple: (($page - 1) * $limit)
        ->setMaxResults($limit)
        ->getQuery()
        ->getResult()
    ;
}

/**
 * Methode qui donne le nombre total de voitures en bdd
 *
 * @return int
 */
public function getTotalCars(): int
{
    return $this->createQueryBuilder('c')
        ->select('count(c)')
        ->getQuery()
        ->getSingleScalarResult() // getSingleScalarResult permet de récupérer direcetement le resultat sans passer par un array
    ;
}
```

## Twig template :

```php
<nav aria-label="...">
    {% set totalPage = (totalCars / limit) | round(0, 'ceil') %}
    <ul class="pagination justify-content-center">

        <li class="page-item {{(page == 1) ? 'disabled'}}">
        <a class="page-link" href="?page={{page - 1}}" tabindex="-1" aria-disabled="true">Previous</a>
        </li>
        
        {% for item in 1..totalPage %}                 
            <li class="page-item {{(item == page) ? 'active'}}"><a class="page-link" href="?page={{item}}">{{item}}</a></li>
        {% endfor %}

        <li class="page-item {{(page == totalPage) ? 'disabled'}}">
        <a class="page-link" href="?page={{page + 1}}">Next</a>
        </li>

    </ul>
</nav>
```