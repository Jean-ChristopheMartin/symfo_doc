### Les Formulaires :

## Commande de base 

- créer un form : 

```php bin/console make:form```

- Bootstraping des forms (config/pakages/twig) :

```form_themes: ['bootstrap_5_layout.html.twig']```

- Changer la langue (config/packages/translation/fr) :

```composer require symfony/translation```


## Modification et contrainte :

- Modifications des champs

```php
    public function buildForm(FormBuilderInterface $builder, array $options): void
    {
        $builder
            ->add('title', TextType::class, [
                "label" => "Titre du film",
                "attr" => [
                    "placeholder" => "Titre du film"
                ],
            ])
            ->add('duration', IntegerType::class, [
                "label" => "Durée du film",
                "attr" => [
                    "placeholder" => "Durée du film"                
                ],
                "help" => "Durée en minutes"
            ])
            ->add('release_date', DateType::class, [
                "label" => "Date de sortie du film",
                'widget' => 'single_text',                
            ])
            ->add('synopsis',TextareaType::class,[
                "label" => "Synopsis",
                "attr" => [
                    "placeholder" => "Synopsis"
                ]
            ])
            ->add('summary',TextareaType::class,[
                "label" => "Sommaire",
                "attr" => [
                    "placeholder" => "Sommaire",
                    "help" => "* Maximum 500 mots"
                ]
            ])
            ->add('poster',UrlType::class,[
                "label" => "Votre image *",
                "attr" => [
                    "placeholder" => "Votre image"
                ],
                "help"=> "* L'url d'une image"
            ])
            ->add('type',ChoiceType::class,[
                "choices" => [
                    "Film" => "Film",
                    "Série" => "Série"
                ],
                "label" => "Film ou série",
            ])
                ->add('genres', EntityType::class,[
                    "class" => Genre::class,
                    "label" => "Genres *",
                    "multiple" => true,
                    "expanded" => true,
                    "help" => "* Vous pouvez choisir plusieurs genres"
                ])
        ;
    }
```

- Contraintes ; 

```php
use Symfony\Component\Validator\Constraints as Assert;
    /**
     * @ORM\Column(type="string", length=255)
     * @Assert\Length(min = 1, max = 255)
     * @Assert\NotBlank
     */
    private $title;

    /**
     * @ORM\Column(type="integer", nullable=true)
     * @Assert\Range(min = 1, max = 400)
     * @Assert\NotBlank
     */
    private $duration;

    /**
     * @ORM\Column(type="date", nullable=true)
     * @Assert\NotBlank
     */
    private $release_date;

    /**
     * @ORM\Column(type="text")
     * @Assert\Length(min = 50, max = 2000)
     * @Assert\NotBlank
     */
    private $synopsis;

    /**
     * @ORM\Column(type="string", length=500)
     * @Assert\Length(min = 50, max = 500)
     * @Assert\NotBlank
     */
    private $summary;

    /**
     * @ORM\Column(type="string", length=255)
     * @Assert\Url
     * @Assert\Length(min = 10, max = 255)
     * @Assert\NotBlank
     */
    private $poster;
```

## Affichage et traitement du form : 

- Ajouter :

```php
/**
 * @Route("/review/ajouter/{id}", name="app_review_add")
 * 
 * @IsGranted("ROLE_USER")
 * @param Movie $movie
 * @param ReviewRepository $reviewRepository
 * @param Request $request
 * @return Response
 */
public function add(Movie $movie, ReviewRepository $reviewRepository, Request $request): Response
{
    
    $review = new Review();

    $form = $this->createForm(ReviewType::class, $review);

    $form->handleRequest($request);

    if ($form->isSubmitted() && $form->isValid()) {
        $form->getData();

        $review->setMovie($movie);

        $reviewRepository->add($review, true);

        $this->addFlash(
            "success",
            "Commentaire bien postée"
        );

        return $this->redirectToRoute('app_movie_show', ["id" => $movie->getId()]);
    }


    return $this->renderform('front/review/form.html.twig', [
        'controller_name' => 'ReviewController',
        'form'            => $form,
        'movie'           => $movie,
    ]);
}
```

- Modifier :

```php
/**
 * @Route("/article/modifier/{id}", name="app_post_update")
 * @param Post $post correspond à notre objet post récupéré en bdd
 * @param EntityManagerInterface Notre entityManager de l'application
 * @param Request $request correspond à l'instance de notre requete http et pleins d'informations qui sont accessible dans l'objet $request comme les données du formulaire
 */
public function update(Post $post,EntityManagerInterface $entityManager, Request $request): Response
{


    // Je créer le formulaire avec le bon type et le bon objet instancié (ici un post vide)
    $form = $this->createForm(PostType::class, $post);

    $form->handleRequest($request);

        // Cette condition ne sera valide que lorsque le formulaire sera envoyé et valide
        if ($form->isSubmitted() && $form->isValid()) {
        
        // getData va récupérer les données, et les setters automatiquement sur l'objet qui est lié au formulaire, ici mon $post
        $form->getData();

        $post->setUpdatedAt(new DateTimeImmutable());

        // Sauvegarde l'objet en bdd
        $entityManager->flush();

        // On traite un formulaire donc on redirige une fois que c'est fait
        return $this->redirectToRoute("app_post_index");

    }


    return $this->renderForm('post/form.html.twig',[
        "form" => $form
    ]);

}
```

- Supprimer :

```php
/**
 * @Route("/article/supprimer/{id}", name="app_post_delete")
 */
public function delete(Post $post, PostRepository $postRepository): Response
{

    $postRepository->remove($post,true);

    // L'autowiring scann l'url et la class, si un paramètre concorde il va automatiquement instancier le bon objet s'il n'existe pas 404
    return $this->redirectToRoute("app_post_index");
}
```

## Form de création de compte : 

### Controller :

```php
    /**
     * @Route("/user/register", name="user_register")
     */
    public function register(Request $request, UserPasswordEncoderInterface $encoder, RoleRepository $roleRepository)
    {
        $user = new User();

        $form = $this->createForm(RegisterType::class, $user);

        $form->handleRequest($request);

        if ($form->isSubmitted() && $form->isValid()) {
            // Encodage du mot de passe
            $user->setPassword($encoder->encodePassword($user, $user->getPassword()));
            // Assignartion du rôle par défaut VIA le nom du rôle et non l'ID
            $role = $roleRepository->findOneByRoleString('ROLE_USER');
            $user->setRole($role);

            $entityManager = $this->getDoctrine()->getManager();
            $entityManager->persist($user);
            $entityManager->flush();

            $this->addFlash('success', 'Vous êtes enregistré. Vous pouvez maintenant vous connecter.');

            return $this->redirectToRoute('login');
        }

        return $this->render('user/register.html.twig', [
            'form' => $form->createView(),
        ]);
    }
```

### Formulaire : 

- Avec confirmation Mdp et options :

```php
class UserType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options): void
    {
        $builder
            ->add('username', TextType::class, [
                "label" => "Votre pseudo",
                "attr" => [
                    "placeholder" => "Votre pseudo"
                ]
            ])
            ->add('roles', ChoiceType::class, [
                "choices"=>[
                    "Manager" => "ROLE_MANAGER",
                    "Admin" => "ROLE_ADMIN"
                ],
                "expanded" => true,
                "multiple" => true
            ])
            ->add('email', EmailType::class, [
                "label" => "L'email",
                "attr" => [
                    "placeholder" => "L'email"
                ]
            ]);
        // ici si l'option = false on afficher le champ password avec sa double validation
        if (!$options["edit"]) {
            $builder
                ->add('password', RepeatedType::class, [
                    'type' => PasswordType::class,
                    'invalid_message' => 'Les deux mots de passes doivent être identiques.',
                    'first_options'  => [
                        'label' => 'Mot de passe',
                        "attr" => [
                            "placeholder" => "Le mot de passe"
                        ]
                    ],
                    'second_options' => [
                        'label' => 'Répéter le mot de passe',
                        "attr" => [
                            "placeholder" => "Le mot de passe"
                        ]
                    ],
            ]);
        }
    }

    public function configureOptions(OptionsResolver $resolver): void
    {
        $resolver->setDefaults([
            'data_class' => User::class,
            // Options définis
            "edit" => false
        ]);
    }
}
```

- utilisation de l'option :

```php
public function edit(Request $request, User $user, UserRepository $userRepository): Response
{
    // gestion de l'option dans le controller
    $form = $this->createForm(UserType::class, $user, ["edit" => true]);
    $form->handleRequest($request);
```

- Gestion du hash du Mdp :

```php
/**
 * @Route("/ajouter", name="app_user_new", methods={"GET", "POST"})
 */
public function new(Request $request, UserRepository $userRepository, UserPasswordHasherInterface $passwordHasher): Response
{
    $user = new User();
    $form = $this->createForm(UserType::class, $user);
    $form->handleRequest($request);
    
    if ($form->isSubmitted() && $form->isValid()) {

        // Hash du Mdp
        $hashedPassword = $passwordHasher->hashPassword(
            $user,
            $user->getPassword()
        );

        $user->setPassword($hashedPassword);

        $userRepository->add($user, true);

        return $this->redirectToRoute('app_user_index', [], Response::HTTP_SEE_OTHER);
    }

    return $this->renderForm('user/new.html.twig', [
        'user' => $user,
        'form' => $form,
    ]);
}
```

- Bouton delete en post :

```php
<form class="d-inline-flex " method="post" action="{{ path('app_user_delete', {'id': user.id}) }}" onsubmit="return confirm('Etes vous sur de vouloir supprimer l'utilisateur?');">
    <input type="hidden" name="_token" value="{{ csrf_token('delete' ~ user.id) }}">
    <button class="btn btn-danger" onclick="return confirm('Voulez vous supprimer ?')"><i class="bi bi-trash-fill"></i></button>
</form>
```