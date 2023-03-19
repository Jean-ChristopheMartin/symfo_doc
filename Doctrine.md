# Doctrine :

## Les commande de base :

- connexion bdd .env :

```DATABASE_URL="mysql://UTILISATEUR:MDP@127.0.0.1:3306/BDD_NAME?serverVersion=mariadb-10.3.258&charset=utf8mb4"```

- Créer une base de donnée : 

```php bin/console doctrine:database:create```

- Créer une Table/Entity : 

```make:entity```

- Régénéré repo/set/get d'une entity: 

```php bin/console make:entity --regenerate```

- Créer la migration : 

```php bin/console make:migration```

- Migrer la migration en bdd : 

```php bin/console doctrine:migrations:migrate```

## Relation Many to many avec attribut :

- Il faut décomposer la relation en deux many to one, on crée les deux entités, on crée la table pivot comme une entité à part entière, elle aura comme champs de type relation les deux entités auquelle elle sera lié, possibilité de rajouter des attributs en plus vu que c'est une entité.
attention à la modification de l'Id de la table pivot.

## Requéte personalisé (Repository) :

- Barre de recherche : 
```php
/**
 * 
 * @return Movie[] Returns an array of Movie objects ordered by title
 */
public function findAllOrderByTitleSearch($needle = null){
    // Quand on créer requpete personnalisé avec le builder , on utilise la ligne ci-dessous
    return $this->createQueryBuilder('m')
        ->orderBy("m.title")
        ->where("m.title LIKE :needle")
        ->setParameter("needle","%$needle%")
        ->getQuery()
        ->getResult();
}
```

- Avec jointure :
```php
/**
 * Méthode pour récupérer les voitures et leurs marques en une requéte
 *
 * @return array
 */
public function findAllCarsWithBrands(): array
{
    return $this->createQueryBuilder('c')
        ->innerJoin("c.brand","b")
        ->addSelect("b")
        ->orderBy("c.id")
        ->getQuery()
        ->getResult()
    ;
}
```

- Avec jointure table pivot :

```php
/**
* 
* @return Movie[] Returns an array of Movie objects ordered by title
*/
public function findAllMovieByGenre($id){
    // Quand on créer requpete personnalisé avec le builder , on utilise la ligne ci-dessous
    return $this->createQueryBuilder('m')
        ->innerJoin('m.genres', 'g')
        ->where('g.id = :id')
        ->setParameter('id', $id)
        ->getQuery()
        ->getResult();
}
```

- SQL basique :

```php
public function findRandom(){

    $conn = $this->getEntityManager()->getConnection();
    $sql = '
        SELECT * FROM movie
        ORDER BY RAND()
        LIMIT 1
    ';

    $stmt = $conn->prepare($sql);
    $resultSet = $stmt->executeQuery();

        // returns the result
        return $resultSet->fetchAssociative();
}
```