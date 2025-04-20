# Arquitetura-e-Boas-Praticas

## 🏛 **1. Repository Pattern (Padrão Repositório)**

O **Repository Pattern** ajuda a **separar a lógica de acesso ao banco de dados da lógica da aplicação**, facilitando a manutenção e os testes.

### **✅ Quando usar?**

- Quando você quer evitar consultas diretas ao banco dentro dos Controllers.
- Quando precisa reutilizar consultas em diferentes partes do sistema.

### **🚀 Exemplo no Laravel:**

📌 **Passo 1: Criar o Repositório**

```php
namespace App\Repositories;

use App\Models\User;

class UserRepository
{
    public function getAll()
    {
        return User::all();
    }

    public function findById($id)
    {
        return User::find($id);
    }

    public function create(array $data)
    {
        return User::create($data);
    }
}

```

📌 **Passo 2: Criar um Service Provider para Injeção de Dependência**

```php
namespace App\Providers;

use Illuminate\Support\ServiceProvider;
use App\Repositories\UserRepository;

class RepositoryServiceProvider extends ServiceProvider
{
    public function register()
    {
        $this->app->bind('UserRepository', function ($app) {
            return new UserRepository();
        });
    }
}

```

📌 **Passo 3: Usar no Controller**

```php
namespace App\Http\Controllers;

use App\Repositories\UserRepository;
use Illuminate\Http\Request;

class UserController extends Controller
{
    protected $userRepository;

    public function __construct(UserRepository $userRepository)
    {
        $this->userRepository = $userRepository;
    }

    public function index()
    {
        return response()->json($this->userRepository->getAll());
    }
}

```

✅ **Vantagens:** Organização, reuso de código e desacoplamento.

---

## ⚙️ **2. Factory Pattern (Padrão Fábrica)**

O **Factory Pattern** é usado para criar objetos de forma centralizada, evitando a necessidade de instanciar classes manualmente por toda a aplicação.

### **✅ Quando usar?**

- Quando precisa criar objetos de forma padronizada.
- Quando deseja facilitar a manutenção e modificar a lógica de criação em um só lugar.

### **🚀 Exemplo no Laravel (Factory para criar usuários automaticamente)**

📌 **Passo 1: Criar um Factory**

```php
php artisan make:factory UserFactory --model=User

```

📌 **Passo 2: Definir a estrutura do Factory**

```php
namespace Database\Factories;

use App\Models\User;
use Illuminate\Database\Eloquent\Factories\Factory;
use Illuminate\Support\Str;

class UserFactory extends Factory
{
    protected $model = User::class;

    public function definition()
    {
        return [
            'name' => $this->faker->name(),
            'email' => $this->faker->unique()->safeEmail(),
            'password' => bcrypt('password'),
            'remember_token' => Str::random(10),
        ];
    }
}

```

📌 **Passo 3: Usar a Factory para gerar dados fake**

```php
User::factory()->count(10)->create();

```

✅ **Vantagens:** Evita repetição de código e facilita a geração de dados para testes.

---

## 🔔 **3. Observer Pattern (Padrão Observador)**

O **Observer Pattern** permite que um objeto (Observer) **"escute" eventos** e execute ações automaticamente quando algo acontece.

### **✅ Quando usar?**

- Quando precisa executar ações sempre que um modelo for **criado, atualizado ou deletado**.
- Quando deseja **manter a lógica separada** no código.

### **🚀 Exemplo no Laravel (Enviar e-mail ao criar usuário)**

📌 **Passo 1: Criar um Observer**

```php
php artisan make:observer UserObserver --model=User

```

📌 **Passo 2: Definir o que acontece ao criar um usuário**

```php
namespace App\Observers;

use App\Models\User;
use Illuminate\Support\Facades\Mail;
use App\Mail\WelcomeMail;

class UserObserver
{
    public function created(User $user)
    {
        Mail::to($user->email)->send(new WelcomeMail($user));
    }
}

```

📌 **Passo 3: Registrar o Observer no `AppServiceProvider`**

```php
use App\Models\User;
use App\Observers\UserObserver;

public function boot()
{
    User::observe(UserObserver::class);
}

```

✅ **Vantagens:** Código mais modular e automatizado.

---

## 🎭 **4. Strategy Pattern (Padrão Estratégia)**

O **Strategy Pattern** permite definir um conjunto de **algoritmos intercambiáveis** e escolher qual será usado dinamicamente.

### **✅ Quando usar?**

- Quando tem **várias regras de negócio parecidas**, mas com diferenças pontuais.
- Quando precisa trocar a **lógica de execução em tempo de execução**.

### **🚀 Exemplo no Laravel (Diferentes formas de pagamento)**

📌 **Passo 1: Criar a Interface de Pagamento**

```php
namespace App\Services\Payments;

interface PaymentStrategy
{
    public function pay($amount);
}

```

📌 **Passo 2: Criar as Implementações (Boleto e Cartão)**

```php
namespace App\Services\Payments;

class BoletoPayment implements PaymentStrategy
{
    public function pay($amount)
    {
        return "Pagamento de R$ $amount realizado via Boleto.";
    }
}

class CreditCardPayment implements PaymentStrategy
{
    public function pay($amount)
    {
        return "Pagamento de R$ $amount realizado via Cartão de Crédito.";
    }
}

```

📌 **Passo 3: Criar a Classe Principal para Gerenciar Estratégias**

```php
namespace App\Services\Payments;

class PaymentContext
{
    private $paymentStrategy;

    public function setPaymentStrategy(PaymentStrategy $paymentStrategy)
    {
        $this->paymentStrategy = $paymentStrategy;
    }

    public function pay($amount)
    {
        return $this->paymentStrategy->pay($amount);
    }
}

```

📌 **Passo 4: Usar o Strategy no Controller**

```php
$paymentContext = new PaymentContext();

$paymentContext->setPaymentStrategy(new BoletoPayment());
echo $paymentContext->pay(100); // Pagamento via Boleto

$paymentContext->setPaymentStrategy(new CreditCardPayment());
echo $paymentContext->pay(250); // Pagamento via Cartão

```

✅ **Vantagens:** Código flexível e permite adicionar novas formas de pagamento facilmente.

---

## 🔥 **Conclusão**

Agora você conhece quatro **padrões essenciais** que vão melhorar muito a qualidade do seu código Laravel:

1️⃣ **Repository Pattern** → Separação de lógica e consultas ao banco.

2️⃣ **Factory Pattern** → Facilita a criação de objetos.

3️⃣ **Observer Pattern** → Executa ações automáticas baseadas em eventos.

4️⃣ **Strategy Pattern** → Alterna estratégias de execução dinamicamente.
 
