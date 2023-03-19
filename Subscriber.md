# Subscriber :

## Créer un Subscriber : 

```php bin/console make:subscriber```

- on place notre subscriber au moment ou l'on veux le déclancher.

subscriber qui renvois les infos a twig a l'écoute des controller :

```php
namespace App\EventSubscriber;

use App\Repository\BrandRepository;
use Symfony\Component\EventDispatcher\EventSubscriberInterface;
use Symfony\Component\HttpKernel\Event\ControllerEvent;
use Twig\Environment;

class BrandMenuSubscriber implements EventSubscriberInterface
{
    private $brandRepository;
    private $twig;

    public function __construct(BrandRepository $brandRepository, Environment $twig)
    {
        $this->brandRepository = $brandRepository;
        $this->twig = $twig;
    }

    public function onKernelController(ControllerEvent $event): void
    {
        // Premiere étape récupérer le controller si y'a bien un controller
        $controller = $event->getController();

        // Je vérifie si c'est un tableau (les erreurs ne renvoi pas de tableau) et je récupère l'index 0
        if(is_array($controller)){
            $controller = $controller[0];
        }

        // Je veux récupérer le fqcn
        $controllerName = get_class($controller);

        // Je veux vérifier si mon controller est bien dans App\Controller
        if(!str_contains($controllerName,"App\Controller")){
            return;
        }

        // Récupérer un film aléatoire
        // Faire apsser ce film à ttes mes vues de mon aplli
        $brands = $this->brandRepository->findAll();
        $this->twig->addGlobal("brands", $brands);
    }

    public static function getSubscribedEvents(): array
    {
        return [
            'kernel.controller' => 'onKernelController',
        ];
    }
}
```

Coté twig on peux du coup utiliser "brands" pour tout nos controllers : 

```php
<li class="nav-item dropdown">
    <a class="nav-link dropdown-toggle" data-bs-toggle="dropdown" href="#" role="button" aria-haspopup="true" aria-expanded="false"><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">Marques</font></font></a>
    <div class="dropdown-menu">
        {% for brand in brands %}
        <a class="dropdown-item" href="{{path("app_brand_show", {id:brand.id})}}"><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">{{brand.name}}</font></font></a>
        {% endfor %}
    </div>
</li>
```

### exemple d'un subscriber qui ecoute la response (modification du html) : 

```php
namespace App\EventSubscriber;

use Symfony\Component\DependencyInjection\ParameterBag\ParameterBagInterface;
use Symfony\Component\EventDispatcher\EventSubscriberInterface;
use Symfony\Component\HttpKernel\Event\ResponseEvent;

class MaintenanceSubscriber implements EventSubscriberInterface
{
    private $parameterBag;

    public function __construct(ParameterBagInterface $parameterBag)
    {
        $this->parameterBag = $parameterBag;
    }

    public function onKernelResponse(ResponseEvent $event): void
    {

        // Je récupére le content html de la page envoyé dans la response

        $response = $event->getResponse();

        $content = $response->getContent();

        // symfony par moment comme sur la gestion des erreurs par exemple, envoi des subRequest, ces subRequest vont faire que votre subscriber va se trigger plusieurs fois, on bloque ça grâce à la condition ci-dessous
        if(!$event->isMainRequest()){
            return;
        }
        // Le but de mon subscrbier est de modifier de l'affichage, dans le cas ou la requete est de type ajax ou fetch, l'affichage est son problème du coup on arrête là aussi (façon 1 de bloqué un appel api sur le listener)
        if($event->getRequest()->headers->get("X-Requested-With") === "fetch" || $event->getRequest()->headers->get("X-Requested-With") === "XMLHttpRequest"){
            return;
        }

        // Façon 2 si notre réponse est un json, on arrête là
        if($response->headers->get("Content-Type") === "application/json"){
            return;
        }

        // Je modifie le html

        $newContent = preg_replace("^\<\/nav\>^",'</nav><div class="alert alert-danger mt-3 alert-dismissible fade show">Maintenance prévue mardi 10 janvier à 17h00 <button type="button" class="btn-close" data-bs-dismiss="alert" aria-label="Close"></button></div>',$content,1);

        //dd($this->parameterBag->get('MAINTENANCE'));

        if ($this->parameterBag->get('MAINTENANCE')) {
            
            // J'insert le html
            $response->setContent($newContent);
        }

    }

    public static function getSubscribedEvents(): array
    {
        return [
            'kernel.response' => 'onKernelResponse',
        ];
    }
}
```

### Subscriber pour api qui ecoute les exception et les renvois sous forme de Json : 

```php 
use Doctrine\DBAL\Exception\NotNullConstraintViolationException;
use Symfony\Component\EventDispatcher\EventSubscriberInterface;
use Symfony\Component\HttpFoundation\JsonResponse;
use Symfony\Component\HttpKernel\Event\ExceptionEvent;
use Symfony\Component\HttpKernel\Exception\HttpException;
public function onKernelException(ExceptionEvent $event): void
{
    // on récupère la requete
    $request = $event->getRequest();

    // Je récupère la route
    $route = $request->get("_route");

    // Si api n'est pas trouvé dans la string $route, on return pour stopper la fonction
    if(!strpos($route,"api")){
        return;
    }

    // Je récupère l'exception levé par l'application
    $exception = $event->getThrowable();

    if ($exception instanceof HttpException) {
        $data = [
            'status' => $exception->getStatusCode(),
            'message' => $exception->getMessage()
        ];

        $event->setResponse(new JsonResponse($data));
        
    }elseif ($exception instanceof NotNullConstraintViolationException) {
        
        $data = [
            'status' => 400,
            'message' => $exception->getMessage()
        ];

        $event->setResponse(new JsonResponse($data));
        
    }else{
        $data =[
            'status' => 500,
            'message' => $exception->getMessage()
        ];

        $event->setResponse(new JsonResponse($data));
    }
}
```
