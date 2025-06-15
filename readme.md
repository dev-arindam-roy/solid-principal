# S.O.L.I.D
The SOLID principles are a set of five design principles in object-oriented software development that help developers create more maintainable, scalable, and robust code. These principles were introduced by Robert C. Martin (Uncle Bob) and are considered a cornerstone of good software architecture.

Here’s what SOLID stands for:

### S – Single Responsibility Principle (SRP)
### O – Open/Closed Principle (OCP)
### L – Liskov Substitution Principle (LSP)
### I – Interface Segregation Principle (ISP)
### D – Dependency Inversion Principle (DIP)

## Single Responsibility Principle (SRP)
❌ Bad Example – Doing too much in a controller:
```php
class UserController extends Controller
{
    public function register(Request $request)
    {
        $user = User::create($request->all());
        Mail::to($user->email)->send(new WelcomeMail($user));
    }
}

```
✅ Good Example – Separate responsibilities into services:
```php
class UserController extends Controller
{
    protected UserService $userService;

    public function __construct(UserService $userService)
    {
        $this->userService = $userService;
    }

    public function register(Request $request)
    {
        $this->userService->register($request->all());
    }
}
class UserService
{
    public function register(array $data)
    {
        $user = User::create($data);
        Mail::to($user->email)->send(new WelcomeMail($user));
    }
}

```

## Open/Closed Principle (OCP)
❌ Bad Example – Modifying a method every time a new payment method is added:
```php
class PaymentService
{
    public function pay($type, $amount)
    {
        if ($type == 'paypal') {
            // PayPal logic
        } elseif ($type == 'stripe') {
            // Stripe logic
        }
    }
}
```
✅ Good Example – Use interfaces to extend without modifying:
```php
interface PaymentMethodInterface
{
    public function pay($amount);
}

class PaypalPayment implements PaymentMethodInterface
{
    public function pay($amount) { /* PayPal logic */ }
}

class StripePayment implements PaymentMethodInterface
{
    public function pay($amount) { /* Stripe logic */ }
}

class PaymentService
{
    public function __construct(private PaymentMethodInterface $method) {}

    public function pay($amount)
    {
        $this->method->pay($amount);
    }
}
```

## Liskov Substitution Principle (LSP)
❌ Bad Example – A subclass breaks parent expectations:
```php
class Bird
{
    public function fly() {}
}

class Ostrich extends Bird
{
    public function fly() {
        throw new Exception("I can't fly");
    }
}
```
✅ Good Example – Use interfaces properly:
```php
interface BirdInterface
{
    public function makeSound();
}

interface FlyingBirdInterface extends BirdInterface
{
    public function fly();
}

class Sparrow implements FlyingBirdInterface
{
    public function fly() {}
    public function makeSound() {}
}

class Ostrich implements BirdInterface
{
    public function makeSound() {}
}
```

## Interface Segregation Principle (ISP)
❌ Bad Example – Forcing implementation of unused methods:
```php
interface WorkerInterface
{
    public function work();
    public function eat();
}

class RobotWorker implements WorkerInterface
{
    public function work() {}
    public function eat() {
        // Not applicable
    }
}
```
✅ Good Example – Break into smaller interfaces:
```php
interface Workable
{
    public function work();
}

interface Eatable
{
    public function eat();
}

class HumanWorker implements Workable, Eatable
{
    public function work() {}
    public function eat() {}
}

class RobotWorker implements Workable
{
    public function work() {}
}
```

## Dependency Inversion Principle (DIP)
❌ Bad Example – High-level module depends on low-level module:
```php
class ReportService
{
    public function generate()
    {
        $pdf = new Dompdf();
        $pdf->loadHtml('report content');
        $pdf->render();
    }
}
```
✅ Good Example – Depend on abstraction:
```php
interface ReportGeneratorInterface
{
    public function generate(string $content);
}

class PDFReportGenerator implements ReportGeneratorInterface
{
    public function generate(string $content)
    {
        $pdf = new Dompdf();
        $pdf->loadHtml($content);
        $pdf->render();
    }
}

class ReportService
{
    public function __construct(private ReportGeneratorInterface $generator) {}

    public function generateReport()
    {
        $this->generator->generate('report content');
    }
}
```
Register the dependency in a Laravel Service Provider or in AppServiceProvider:
```php
$this->app->bind(ReportGeneratorInterface::class, PDFReportGenerator::class);
```

# EXAMPLE

### 1. UserInterface

```php
namespace App\Contracts;

use App\Models\User;

interface UserInterface
{
    public function createUser(array $userData): User;
    public function getUserById(int $id): ?User;
    public function getAllUsers();
    public function updateUser(int $id, array $userData): bool;
    public function deleteUser(int $id): bool;
}
```

### 2. UserRepository

```php

namespace App\Repositories;

use App\Models\User;
use App\Contracts\UserInterface;
use Illuminate\Support\Facades\Hash;

class UserRepository implements UserInterface
{
    public function createUser(array $userData): User
    {
        $userData['password'] = Hash::make($userData['password']);
        return User::create($userData);
    }

    public function getUserById(int $id): ?User
    {
        return User::find($id);
    }

    public function getAllUsers()
    {
        return User::all();
    }

    public function updateUser(int $id, array $userData): bool
    {
        $user = User::find($id);
        return $user ? $user->update($userData) : false;
    }

    public function deleteUser(int $id): bool
    {
        $user = User::find($id);
        return $user ? $user->delete() : false;
    }
}
```

### 3. UserService

```php
namespace App\Services;

use App\Repositories\UserRepository;
use App\Models\User;

class UserService
{
    protected $userRepository;

    public function __construct(UserRepository $userRepository)
    {
        $this->userRepository = $userRepository;
    }

    public function createUser(array $userData): User
    {
        return $this->userRepository->createUser($userData);
    }

    public function getUserById(int $id): ?User
    {
        return $this->userRepository->getUserById($id);
    }

    public function getAllUsers()
    {
        return $this->userRepository->getAllUsers();
    }

    public function updateUser(int $id, array $userData): bool
    {
        return $this->userRepository->updateUser($id, $userData);
    }

    public function deleteUser(int $id): bool
    {
        return $this->userRepository->deleteUser($id);
    }
}
```

### 4. UserController

```php
namespace App\Http\Controllers;

use Illuminate\Http\Request;
use App\Services\UserService;

class UserController extends Controller
{
    protected $userService;

    public function __construct(UserService $userService)
    {
        $this->userService = $userService;
    }

    public function getUserById()
    {
        dd($this->userService->getUserById(1));
        return $this->userService->getUserById(1);
    }
}
```

