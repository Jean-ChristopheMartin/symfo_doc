# Installation de Symfony

## Installation WebApp :

- ```composer create-project symfony/skeleton:"^5.4" mon_projet```
- ```cd mon_projet```
- ```composer require webapp```

## Installation Skeleton :

- ```composer create-project symfony/skeleton:"^5.4" mon_projet```

### Installation annotations des routes :

- ```composer require annotations```

### Installation du module pour les views "twig" :

- ```composer require twig```
- Emmet html : ctrl+maj+p settings.json -> open workspace -> ```"emmet.includeLanguages": {"twig": "html"}```
- Asset insert : ```composer require symfony/asset```

### Installation console maker :

- ```composer require --dev symfony/maker-bundle``` 
- Liste des make : ```php bin/console list make```

### Installation de Doctrine :

- ```composer require symfony/orm-pack``` 

### Installation des fixtures :

- ```composer require --dev doctrine/doctrine-fixtures-bundle``` 
- ```composer require fakerphp/faker```
  
### Installation débug :

- ```composer require --dev symfony/debug-bundle```
- ToolBar : ```composer require --dev symfony/profiler-pack```

### Installation formulaire : 

- ```composer require symfony/form```
- ```composer require symfony/validator```

### Installation séécurité :

- ```composer require symfony/security-bundle```