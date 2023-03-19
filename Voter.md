# Les voters

**Lien de la doc :** https://symfony.com/doc/5.4/security/voters.html#creating-the-custom-voter

## Introduction

Les voters sont utilisé dans symfony, pour sortir la logique de contrôle d'accès des controlleurs, ils permettent de limité l'accès dans une méthode ou de bloqué l'affichage dans twig selon une logique données écrite dans le voter.


## Créer un voter custom 

Pour créer un voter générique la console : `php bin/console make:voter`

Dans ce voter générique nous allons retrouver la méthode supports, qui va définir les actions du voter ainsi que le sujet :

- Les actions sont a définir sous forme de constante : 
```php
//mes permissions je les déclare pour la maintenance
    const QUESTION_EDIT = 'question_edit';
    const ANSWER_VALIDATE = 'answer_validate';
```
- Elles sont ensuite réutilisé dans la méthode supports :
```php
protected function supports(string $attribute, $question): bool
{
    // replace with your own logic
    // https://symfony.com/doc/current/security/voters.html
    return in_array($attribute, [self::QUESTION_EDIT, self::ANSWER_VALIDATE])
        && $question instanceof \App\Entity\Question;
}
```
- Si l'action n'est pas présente le voter renvera false, libre a vous de mettre autant d'actions que vous le souhaitez par voter

- Il traitera aussi d'un sujet, en général une classe :
```php
$subject instanceof \App\Entity\Question;
```

on peux aussi gérer l'accés au roles :

```php
// On vérifie si l'utilisateur et manager
if ($this->security->isGranted('ROLE_MODERATOR')) {
    return true;
}
```

Dans le cas expliqué ci-dessus, le sujet est la classe Question et les actions possible sont QUESTION_EDIT et ANSWER_VALIDATE


- Dans le voter il y a la méthode voteOnAttributes, c'est dans celle-ci que vous devrais retourner un false si vous voulez bloquer l'accès et un true si vous autorisez l'accès.
- En général un switch case est fait sur les différentes actions, et la logique de gestion d'accès viendra définir si selon l'action le voter renvois true ou false, exemple : 

```php
// ... (check conditions and return true to grant permission) ...
switch ($attribute) {
    case self::QUESTION_EDIT:
        // logic to determine if the user can EDIT
        // return true or false
        // On vérifie que l'on peux modifier la question
        return $this->canEdit($question, $user);
        break;
    case self::ANSWER_VALIDATE:
        // On Vérifie que l'on peux valider la reponse à la question
        return $this->canValidate($question, $user);
            break;
}
```

```php
private function canEdit(Question $question, User $user)
{
    // Le user de la question peux la modifier
    return $user === $question->getUser();
}

private function canValidate(Question $question, User $user)
{ 
    // Le user de la question peux valider
    return $user === $question->getUser();
    
}
```

Ci-dessus le voter appel une méthode privée hasRight, qui contient la logique pour renvoyer true ou false si l'utilisateur a bien crée la question ou non.


## Controler les accès

Sur le controlleur : 

```php
 // Appel au voter qui va bloqué l'accès ou non pour l'action question_edit et le sujet $question qui est une instance de l'entité Question
$this->denyAccessUnlessGranted('question_edit', $question);
// Protéger le validate dans l'Url
$this->denyAccessUnlessGranted('answer_validate', $answer->getQuestion());
```

Sur twig : 

```html
{% if is_granted("question_edit", question) %}
    <p class="mt-3"><a class="btn btn-success" href="{{ path('question_edit', {id: question.id}) }}">Modifier</a></p>

{% endif %}
```