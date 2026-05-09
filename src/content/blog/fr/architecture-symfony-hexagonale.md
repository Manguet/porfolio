---
title: "Architecture hexagonale avec Symfony 7 — retour d'expérience"
description: "Comment structurer une application Symfony en séparant le domaine métier de l'infrastructure : ports, adapters, commandes, et organisation par feature."
pubDate: 2026-04-15
tags: ["Symfony", "Architecture", "PHP"]
lang: fr
slug: architecture-symfony-hexagonale
---

## Introduction : pourquoi l'architecture hexagonale

Quand on débute avec Symfony, la tentation est grande de tout mettre dans les contrôleurs : la logique métier, les appels à la base de données, les envois d'e-mails. Ça fonctionne pour un prototype, mais ça devient vite ingérable dès que l'application grossit. Les tests sont difficiles à écrire, les changements d'infrastructure (remplacer Doctrine par une API externe, par exemple) nécessitent de réécrire une grande partie du code.

L'architecture hexagonale — aussi appelée *Ports and Adapters* — apporte une réponse claire à ce problème : **le domaine métier ne doit rien savoir de l'infrastructure**. C'est lui qui est au centre, et tout le reste (base de données, HTTP, e-mail, queue) s'adapte à lui, et non l'inverse.

Après plusieurs projets Symfony en architecture hexagonale, voici ce que j'ai appris.

## Les concepts fondamentaux

### Le domaine

Le domaine contient les règles métier pures : entités, value objects, services de domaine. Il n'a aucune dépendance vers Symfony, Doctrine ou quoi que ce soit d'externe. Un `User`, une `Invoice`, une `Money` — ce sont des objets PHP simples.

### Les ports

Les ports sont des **interfaces** définies par le domaine. Elles expriment ce dont le domaine a besoin sans préciser comment c'est implémenté. Par exemple :

```php
// src/Domain/Repository/UserRepositoryInterface.php
interface UserRepositoryInterface
{
    public function findByEmail(Email $email): ?User;
    public function save(User $user): void;
}
```

### Les adapters

Les adapters **implémentent** les ports. L'adapter Doctrine implémente `UserRepositoryInterface` en utilisant l'ORM. L'adapter en mémoire l'implémente avec un simple tableau — parfait pour les tests.

```php
// src/Infrastructure/Repository/DoctrineUserRepository.php
class DoctrineUserRepository implements UserRepositoryInterface
{
    public function __construct(private readonly EntityManagerInterface $em) {}

    public function findByEmail(Email $email): ?User
    {
        return $this->em->getRepository(UserEntity::class)
            ->findOneBy(['email' => (string) $email]);
    }
}
```

## En pratique avec Symfony 7

### Structure des dossiers

J'organise mes projets par **feature** plutôt que par type technique :

```
src/
├── Domain/
│   ├── User/
│   │   ├── Entity/User.php
│   │   ├── Repository/UserRepositoryInterface.php
│   │   └── ValueObject/Email.php
│   └── Invoice/
│       ├── Entity/Invoice.php
│       └── Service/InvoiceCalculator.php
├── Application/
│   ├── User/
│   │   ├── Command/RegisterUser.php
│   │   └── Handler/RegisterUserHandler.php
│   └── Invoice/
│       └── Query/GetUserInvoicesQuery.php
└── Infrastructure/
    ├── Persistence/Doctrine/
    │   └── DoctrineUserRepository.php
    └── Http/Controller/
        └── UserController.php
```

### Injection de dépendances

Symfony facilite le câblage grâce à l'autowiring. Il suffit de lier l'interface à son implémentation dans `services.yaml` :

```yaml
services:
    App\Domain\User\Repository\UserRepositoryInterface:
        class: App\Infrastructure\Persistence\Doctrine\DoctrineUserRepository
```

Pour les tests, on remplace simplement par l'adapter en mémoire.

## Commandes et Query Handlers

Je sépare les **commandes** (qui modifient l'état) des **queries** (qui lisent). C'est le pattern CQRS allégé.

Une commande est un objet immutable qui décrit une intention :

```php
final readonly class RegisterUser
{
    public function __construct(
        public string $email,
        public string $plainPassword,
        public string $firstName,
    ) {}
}
```

Le handler contient la logique applicative :

```php
final class RegisterUserHandler
{
    public function __construct(
        private UserRepositoryInterface $users,
        private PasswordHasherInterface $hasher,
        private EventDispatcherInterface $dispatcher,
    ) {}

    public function __invoke(RegisterUser $command): void
    {
        if ($this->users->findByEmail(new Email($command->email))) {
            throw new UserAlreadyExistsException($command->email);
        }

        $user = User::register(
            email: new Email($command->email),
            hashedPassword: $this->hasher->hash($command->plainPassword),
            firstName: $command->firstName,
        );

        $this->users->save($user);
        $this->dispatcher->dispatch(new UserRegistered($user->id()));
    }
}
```

Le contrôleur devient un simple *glue code* : il valide la requête HTTP, construit la commande, et délègue.

## Tests : l'avantage d'une architecture découplée

C'est là que l'investissement initial se rentabilise. Tester un handler ne nécessite pas de base de données :

```php
class RegisterUserHandlerTest extends TestCase
{
    public function test_registers_new_user(): void
    {
        $users = new InMemoryUserRepository();
        $handler = new RegisterUserHandler(
            users: $users,
            hasher: new FakePasswordHasher(),
            dispatcher: new NullEventDispatcher(),
        );

        ($handler)(new RegisterUser(
            email: 'test@example.com',
            plainPassword: 'secret',
            firstName: 'Alice',
        ));

        $this->assertNotNull($users->findByEmail(new Email('test@example.com')));
    }
}
```

Le test est rapide (pas de base de données, pas de réseau), isolé, et lisible. On teste la logique métier, pas l'infrastructure.

## Conclusion

L'architecture hexagonale demande un effort initial plus important qu'une approche classique MVC. Mais sur tout projet amené à évoluer — et c'est le cas de la plupart des projets réels — elle rend le code plus maintenable, plus testable, et plus résilient aux changements d'infrastructure.

Symfony 7, avec son système de services et son autowiring puissant, est un excellent cadre pour mettre en place cette architecture. L'investissement se rentabilise dès les premières semaines, quand les premiers changements de périmètre arrivent.
