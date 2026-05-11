---
title: "Hexagonal Architecture with Symfony 7 — A Practical Retrospective"
description: "How to structure a Symfony application by separating business domain from infrastructure: ports, adapters, commands, and feature-based organisation."
pubDate: 2026-04-15
tags: ["Symfony", "Architecture", "PHP"]
lang: en
slug: architecture-symfony-hexagonale
---

## Introduction: Why Hexagonal Architecture

When starting out with Symfony, the temptation is to put everything in controllers: business logic, database calls, email sending. It works for a prototype, but quickly becomes unmanageable as the application grows. Tests become hard to write, and infrastructure changes (replacing Doctrine with an external API, for example) require rewriting large chunks of code.

Hexagonal architecture — also known as *Ports and Adapters* — provides a clear answer to this problem: **the business domain must know nothing about the infrastructure**. The domain sits at the centre, and everything else (database, HTTP, email, queue) adapts to it, not the other way around.

After several Symfony projects using hexagonal architecture, here is what I have learned.

## The Core Concepts

### The Domain

The domain contains pure business rules: entities, value objects, domain services. It has no dependency on Symfony, Doctrine, or anything external. A `User`, an `Invoice`, a `Money` — these are plain PHP objects.

### Ports

Ports are **interfaces** defined by the domain. They express what the domain needs without specifying how it is implemented. For example:

```php
// src/Domain/Repository/UserRepositoryInterface.php
interface UserRepositoryInterface
{
    public function findByEmail(Email $email): ?User;
    public function save(User $user): void;
}
```

### Adapters

Adapters **implement** ports. The Doctrine adapter implements `UserRepositoryInterface` using the ORM. The in-memory adapter implements it with a simple array — perfect for tests.

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

## In Practice with Symfony 7

### Folder Structure

I organise projects by **feature** rather than by technical type:

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

### Dependency Injection

Symfony makes wiring straightforward through autowiring. Simply bind the interface to its implementation in `services.yaml`:

```yaml
services:
    App\Domain\User\Repository\UserRepositoryInterface:
        class: App\Infrastructure\Persistence\Doctrine\DoctrineUserRepository
```

For tests, simply swap in the in-memory adapter.

## Commands and Query Handlers

I separate **commands** (which modify state) from **queries** (which read state). This is lightweight CQRS.

A command is an immutable object describing an intention:

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

The handler contains the application logic:

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

The controller becomes simple *glue code*: it validates the HTTP request, builds the command, and delegates.

## Tests: The Payoff of a Decoupled Architecture

This is where the initial investment pays off. Testing a handler requires no database:

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

The test is fast (no database, no network), isolated, and readable. You are testing business logic, not infrastructure.

## Conclusion

Hexagonal architecture requires more upfront effort than a classic MVC approach. But on any project that is expected to evolve — which is most real-world projects — it makes the code more maintainable, more testable, and more resilient to infrastructure changes.

Symfony 7, with its service container and powerful autowiring, is an excellent framework for implementing this architecture. The investment pays off within the first few weeks, as soon as the first scope changes arrive.
