# La sécurité :

**lien de la doc** : https://symfony.com/doc/current/security.html

## Créer un User : 

```php bin/console make:user```

## Créer un login : 

```php bin/console make:controller Login```

## Controller de login/logout

```php
class SecurityController extends AbstractController
{
    /**
     * Méthode qui affiche le formulaire de log
     * @Route("/login", name="app_security_login")
     * 
     * @param AuthenticationUtils $authenticationUtils
     * @return Response
     */
    public function login(AuthenticationUtils $authenticationUtils): Response
    {
        // get the login error if there is one
        $error = $authenticationUtils->getLastAuthenticationError();

        // last username entered by the user
        $lastUsername = $authenticationUtils->getLastUsername();

        return $this->render('security/index.html.twig', [
        'last_username' => $lastUsername,
        'error'         => $error,
        ]);
    }

    /**
     * Méthode de déconexion
     * @Route("/logout", name="app_security_logout", methods={"GET"})
     *
     * @return void
     */
    public function logout(): void
    {
        // controller can be blank: it will never be called!
        throw new \Exception('Don\'t forget to activate logout in security.yaml');
    }
}
```

## Sécurity .yaml - login/logout : 

```php
        main:
            lazy: true
            provider: app_user_provider
            form_login:
                # "app_login" is the name of the route created previously
                login_path: app_security_login
                check_path: app_security_login
                enable_csrf: true
            logout:
                path: app_security_logout
```

## bouton et affichage user : 

```php
{% if is_granted("IS_AUTHENTICATED_FULLY") %}
    <ul class="navbar-nav ms-auto mb-3 me-3 mb-lg-0">
        <li class="nav-item">
            <span class="text-light">
            {{app.user.username}}|
            {% if "ROLE_ADMIN" in app.user.roles %}
                Administrateur
            {% elseif "ROLE_MODERATOR" in app.user.roles %}
                Modérateur
            {% else %}
                Utilisateur
            {% endif %}
            </span>
        </li>
    </ul>
    <a href="{{path("app_security_logout")}}" class="btn btn-danger me-2">Déconnexion</a>
{% else %}
    <a href="{{path("app_security_login")}}" class="btn btn-danger me-2">Connexion</a>
{% endif %}
```

## Sécurity accés-control : 

### Hierarchy

```php
role_hierarchy:
    ROLE_USER :       
    ROLE_MANAGER :  ROLE_USER
    ROLE_ADMIN:     ROLE_MANAGER 
```

### Access control

- ex 1 :

```php
access_control:
    # - { path: ^/favoris, roles: ROLE_USER }
    # - { path: ^/movie, roles: ROLE_USER }
    - { path: ^/review|favoris, roles: ROLE_USER }
    - { path: ^/back-office.*(ajouter|modifier|supprimer), roles: ROLE_ADMIN }
    - { path: ^/back-office, roles: ROLE_MANAGER }
    # - { path: ^/back-office, roles: ROLE_ADMIN }
    # - { path: ^/profile, roles: ROLE_USER }
```

- ex 2 : 

```php
# Easy way to control access for large sections of your site
# Note: Only the *first* access control that matches will be used
access_control:
    - { path: ^/admin/(question|answer)/toggle, roles: ROLE_MODERATOR }
    - { path: ^/admin/(tag|user), roles: ROLE_MODERATOR }
    - { path: ^/answer/validate, roles: ROLE_USER }
    - { path: ^/question/add, roles: ROLE_USER }
    - { path: ^/question/edit/\d+, roles: ROLE_USER }
    - { path: ^/question/\d+, roles: ROLE_USER, methods: ["POST"] }
    - { path: ^/user/(profile|edit), roles: ROLE_USER }
```

- Annotation sécurité :

```php
use Sensio\Bundle\FrameworkExtraBundle\Configuration\IsGranted;

@IsGranted("ROLE_USER")
```

- Twig gestion : 

```php
{% if is_granted("ROLE_MODERATOR") %}
    <li class="nav-item dropdown">
        <a class="nav-link dropdown-toggle" data-bs-toggle="dropdown" href="#" role="button" aria-haspopup="true" aria-expanded="false"><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">Marques</font></font></a>
        <div class="dropdown-menu">
            {% for car in cars %}
            <a class="dropdown-item" href="{{path("app_brand_show", {id:car.brand.id})}}"><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">{{car.brand}}</font></font></a>
            {% endfor %}
        </div>
    </li>
{% endif %}
```
