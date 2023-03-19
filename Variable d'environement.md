# Variable d'environement :

Les variables sont placé dans le fichier .env :

```php
###> symfony/mailer ###
MAILER_DSN=mailjet+smtp://559820d83b5d30bd047377ff20d8d6f3:a5aaeeeb8aa009dd6ce74a8d80b8e5f4@default
###< symfony/mailer ###

# AdminMail
ADMIN_MAIL="mjeanchristophe88@gmail.com"

# Slugger
TO_LOWER=true

# OmdbApi
API_KEY="5d7f5566"

# Maintenace
MAINTENANCE=true
```

Elles sont passé dans le fichier services.yaml :

```php
parameters:
    app.adminMail: '%env(string:ADMIN_MAIL)%'
    app.toLower: '%env(bool:TO_LOWER)%'
    MAINTENANCE: '%env(bool:MAINTENANCE)%'
```
On peux récupérer diréctement les paramétres grace a : 

```php
use Symfony\Component\DependencyInjection\ParameterBag\ParameterBagInterface;
ParameterBagInterface $parameterBag

$this->parameterBag->get('MAINTENANCE')
```

On peux aussi les autowire grace a :

```php
services:
    
    _defaults:

    App\:
        resource: '../src/'
        exclude:
            - '../src/DependencyInjection/'
            - '../src/Entity/'
            - '../src/Kernel.php'

    App\Service\SendMailService:
        arguments:
            $adminMail: '%app.adminMail%'

    App\Service\MySlugger:
        arguments:
            $toLower: '%app.toLower%'

    App\Service\OmdbApi:
        arguments:
            $apiKey: '%env(string:API_KEY)%'
```

```php
    private $mailer;
    private $adminMail;

    public function __construct(MailerInterface $mailer, string $adminMail)
    {
        $this->mailer = $mailer;
        $this->adminMail = $adminMail;
    }
```