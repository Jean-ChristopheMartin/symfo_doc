# Les Fixtures :

## Commande de base :

- Créer une fixture :

```php bin/console make:fixture ```

- Lancer les fixtures :

```php bin/console doctrine:fixtures:load ```

- Faker :

```composer require fakerphp/faker```

```php
use Faker;
$faker = Faker\Factory::create("fr_FR");
```

- Populator :

```php
$populator = new \Faker\ORM\Doctrine\Populator($faker, $manager);
$insertedPKs = $populator->execute();
```

```php
// ! Fixtures de MOVIE

// Movie::class pour faire référence à la classe et 5 pour le nom d'itération en bdd
$populator->addEntity(Movie::class, 10,[
    // En troisième argument de la méthode addEntity, il est possible de mettre un tableau avec les columns à formatter manuellement
    // Pour les formater manuellement il faut en valeur de tableau faire passer une fonction, vu que nos fonctions seront appeller uniquement ici, j'utilise des fonctions anonymes
    "duration" => function() use ($faker){
        return $faker->numberBetween(10,240);
    },
    "poster" => function() use ($faker) {
        return "https://picsum.photos/id/".$faker->numberBetween(1,200)."/300/500";
    },
    "rating" => function() use ($faker){
        return $faker->randomFloat(1,1,5);
    },
    "type" => function() use ($faker){
        // ternaire pour choisir entre film et série
        $type = rand(0,1) ? "Série": "Film";
        // return $type;
        return $faker->randomElement(["Série", "Film"]);
    },
    "title" => function() use ($faker){
        return $faker->unique()->movieTitle();
    }
]);
```

- Provider : 

```php
use App\DataFixtures\Providers\AppProvider;
$faker->addProvider(new AppProvider());
```

- Fixture sans populator :

```php
$movies = [];

for ($i=0; $i < 10; $i++) { 
    
    $movie = new Movie();
    
    $movie->setTitle($faker->word());
    $movie->setDuration($faker->numberBetween(20, 180));
    $movie->setReleaseDate($faker->dateTimeThisCentury());
    $movie->setSynopsis($faker->text());
    $movie->setSummary($faker->paragraph());
    $movie->setPoster("https://picsum.photos/id/".$faker->numberBetween(1,200)."/300/500");
    $movie->setRating($faker->randomFloat(1,1,5));
    $movie->SetType($faker->randomElement(['film', 'série']));
    
    $manager->persist($movie);

    $movies[] = $movie;

}
```

```php
foreach ($movies as $movie) {

    if ($movie->gettype() == 'série') {

        for ($i=1; $i < rand(0, 5) ; $i++) { 

            $season = new Season();

            $season->setNumberEpisodes($faker->numberBetween(5, 25));
            $season->setNumberSeason($i);
            $season->setMovie($movie);

            $manager->persist($season);
        }
    }
}
```

- Gérer un mdp :
    - Avec Populator :
     
    ```php
    // ! User
    $populator->addEntity(User::class, 5, [
        "roles" => function() use ($faker){
            return $faker->randomElements(["ROLE_USER", "ROLE_MANAGER ", "ROLE_ADMIN"]);
        },
        "password" => function(){
            return password_hash("azerty", PASSWORD_BCRYPT);
        }
    ]);
    ```

    - Sans Populator :

```php
use Symfony\Component\PasswordHasher\Hasher\UserPasswordHasherInterface;

private $encoder;

public function __construct(UserPasswordHasherInterface $encoder)
{
    $this->encoder = $encoder;
}

// ! User

$UsersRoles = [
    'ADMIN' => 'admin',
    'MODERATOR' => 'moderator',
    'USER' => 'user',
];

foreach ($UsersRoles as $role => $name) {

    $user = new User;
    $user->setUsername($name);
    $user->setRoles(['ROLE_'.$role]);
    $user->setPassword($this->encoder->hashPassword($user, $name));
    $user->setEmail($name.'@gmail.com');

    $manager->persist($user);
}  
```


## Attention à l'ordre des fixtures.