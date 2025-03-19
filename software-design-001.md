# ASP.NET Core MVC: Controller como Mediator e uso de Ambassador

Este exemplo demonstra como estruturar um *Controller* em ASP.NET Core MVC atuando como um *mediator* e chamando uma classe *Ambassador*. Além disso, inclui uma implementação de uma classe *Mediator* separada, que centraliza a lógica empresarial.

## Estrutura de Código

### Controller (Mediator)
```csharp
// C#
using Microsoft.AspNetCore.Mvc;

public class ProductController : Controller
{
    private readonly IMediator _mediator;
    private readonly IProductAmbassador _ambassador;

    public ProductController(IMediator mediator, IProductAmbassador ambassador)
    {
        _mediator = mediator;
        _ambassador = ambassador;
    }

    [HttpGet("products")]
    public IActionResult GetProducts()
    {
        // Atuando como um mediador para delegar lógica empresarial
        var products = _mediator.FetchAllProducts();
        return Ok(products);
    }

    [HttpPost("products/external")]
    public IActionResult SendProductsToExternalService([FromBody] List<Product> products)
    {
        // Usando o ambassador para lidar com integração externa
        var response = _ambassador.SendProducts(products);
        return Ok(response);
    }
}
```

### Mediator
```csharp
// C#
public interface IMediator
{
    List<Product> FetchAllProducts();
}

public class ProductMediator : IMediator
{
    private readonly IProductRepository _repository;

    public ProductMediator(IProductRepository repository)
    {
        _repository = repository;
    }

    public List<Product> FetchAllProducts()
    {
        // Centralizando a lógica empresarial
        var products = _repository.GetAllProducts();
        return products.Where(p => p.InStock).ToList(); // Apenas produtos em estoque
    }
}
```

### Ambassador
```csharp
// C#
public interface IProductAmbassador
{
    string SendProducts(List<Product> products);
}

public class ProductAmbassador : IProductAmbassador
{
    public string SendProducts(List<Product> products)
    {
        // Simulação de uma chamada a API de um serviço externo
        return $"Sent {products.Count} products to the external service!";
    }
}
```

### Repository
```csharp
// C#
public interface IProductRepository
{
    List<Product> GetAllProducts();
}

public class ProductRepository : IProductRepository
{
    public List<Product> GetAllProducts()
    {
        // Dados simulados
        return new List<Product>
        {
            new Product { Id = 1, Name = "Laptop", InStock = true },
            new Product { Id = 2, Name = "Phone", InStock = false },
            new Product { Id = 3, Name = "Monitor", InStock = true },
        };
    }
}
```

### Model
```csharp
// C#
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public bool InStock { get; set; }
}
```

### Diferenças e Quando Aplicar

#### Controller como Mediator

Vantagens:
 - Simplicidade: Lógica simples pode ser tratada diretamente no Controller.
 - Menor sobrecarga: Evita a criação de camadas adicionais.
 - Direto e coerente.

Desvantagens:
 - Acoplamento elevado: Difícil de testar e manter à medida que cresce.
 - Baixa reutilização: Lógica empresarial encapsulada não é compartilhável.
 - Risco de crescimento descontrolado: Pode levar ao "God Controller".
 - Quando usar: Aplicações pequenas ou baixa complexidade.

#### Mediator (Classe Separada)

Vantagens:
 - Centralização: Lógica empresarial concentrada em um só lugar.
 - Facilidade de teste: Separação clara entre lógica e comunicação HTTP.
 - Reutilização: A lógica empresarial pode ser usada em vários Controllers.

Desvantagens:
 - Complexidade extra: Pode parecer excessivo para projetos menores.
 - Sobrecarga inicial: Requer mais configuração e estrutura.
 - Quando usar: Aplicações médias ou grandes com lógica complexa.

#### Ambassador
Vantagens:
 - Isolamento: Simplifica e encapsula a lógica de serviços externos.
 - Facilidade de manutenção: Mudanças em integrações não afetam outras partes do sistema.
 - Funcionalidades adicionais: Implementação de caching, logging ou retry.

Desvantagens:
 - Latência externa: Problemas no serviço remoto podem impactar o sistema.
 - Complexidade adicional: Uma camada extra para gerenciar.
 - Quando usar: Sistemas com integrações externas frequentes ou complexas.

### Resumo

| Abordagem           | Quando usar                                                 | Complexidade    |
|---------------------|------------------------------------------------------------|-----------------|
| Controller Mediator | Apps pequenas ou lógica simples                             | Baixa           |
| Mediator            | Apps médias/grandes com lógica empresarial complexa         | Moderada        |
| Ambassador          | Integrações externas frequentes e necessidade de isolamento | Moderada/Alta   |


## Pipes & Filter

```csharp
// C#
// salvar em um arquivo PipesFilters.csx
// para executar abra o terminal e rode o arquivo com o comando "csi PipesFilters.csx"

using System;

public static class Pipeline
{
    public static T ApplyPipeline<T>(T input, params Func<T, T>[] filters)
    {
        return filters.Aggregate(input, (current, filter) => filter(current));
    }
}

public static class Filters
{
    public static double Receiver(double eventValue)
    {
        Console.WriteLine($"received {eventValue}");
        return eventValue * 100;
    }

    public static double Transformer(double eventValue)
    {
        Console.WriteLine($"transformed {eventValue}");
        return eventValue * 10;
    }

    public static double Adder(double eventValue, double constant)
    {
        Console.WriteLine($"added {eventValue}");
        return eventValue + constant;
    }

    public static double Send(double eventValue)
    {
        Console.WriteLine($"sent {eventValue}");
        return eventValue / 1000;
    }
}

public class PipesFilters
{
    public static void Teste()
    {
        var input = 10.0;

        var result = Pipeline.ApplyPipeline(input,
            Filters.Receiver,
            Filters.Transformer,
            e => Filters.Adder(e, 20),
            Filters.Send);

        Console.WriteLine(result);
    }
}

PipesFilters.Teste();
```

```java
// Java
// salvar em um arquivo PipesFilters.java
// para rodar execute o comando "jshell PipesFilters.java", quando o shell abrir
// execute "PipesFilters.main(null);"

import java.util.function.Function;

class PipesFilters {
    public static void main(String[] args) {
        var input = 10.0;

        var result = Pipeline.applyPipeline(input,
                Filters::receiver,
                Filters::transformer,
                e -> Filters.adder(e, 20),
                Filters::send);

        System.out.println(result);
    }

    public class Pipeline {

        @SafeVarargs
        private static <T> T applyPipeline(T input, Function<T, T>... filters) {
            for (Function<T, T> filter : filters) {
                input = filter.apply(input);
            }
            return input;
        }
    }

    public class Filters {
        public static double receiver(double event) {
            System.out.printf("received %s\n", event);
            return event * 100;
        }

        public static double transformer(double event) {
            System.out.printf("transformed %s\n", event);
            return event * 10;
        }

        public static double adder(double event, double constant) {
            System.out.printf("added %s\n", event);
            return event + constant;
        }

        public static double send(double event) {
            System.out.printf("sent %s\n", event);
            return event / 1000;
        }
    }
}
```

Explicação do Pattern
 - Pipeline Centralizado: Ambos os exemplos implementam pipelines, onde cada filtro processa e encaminha os dados para o próximo.
 - Filtros Reutilizáveis: Transformações e persistência são encapsuladas como filtros.
 - Flexibilidade: Novos filtros podem ser facilmente adicionados ao pipeline.

## Saga, command e state machine

**Saga Pattern (muito usado em arquiteturas de microsserviços):**
 - Esse padrão é excelente para gerenciar transações distribuídas e processos longos. Cada etapa do fluxo (captura do cartão, criação da assinatura, ativação do serviço, etc.) seria uma transação ou tarefa separada, e o Saga orquestraria ou coreografaria essas etapas.
 - Ele permite executar compensações caso uma das etapas falhe, como reverter a criação da assinatura caso a ativação do serviço não seja bem-sucedida.

```java
// Java
public class SagaExample {
    public static void main(String[] args) {
        try {
            new Saga()
                .step(() -> System.out.println("Capturing Credit Card..."))
                .step(() -> System.out.println("Creating Subscription..."))
                .step(() -> System.out.println("Activating Service..."))
                .execute();
        } catch (Exception e) {
            System.out.println("Transaction failed. Compensating...");
        }
    }
}

class Saga {
    private final List<Runnable> steps = new ArrayList<>();

    public Saga step(Runnable step) {
        steps.add(step);
        return this;
    }

    public void execute() {
        for (Runnable step : steps) {
            step.run();
        }
    }
}
```

```csharp
// C#
using System;
using System.Collections.Generic;

class SagaExample
{
    static void Main(string[] args)
    {
        try
        {
            new Saga()
                .Step(() => Console.WriteLine("Capturing Credit Card..."))
                .Step(() => Console.WriteLine("Creating Subscription..."))
                .Step(() => Console.WriteLine("Activating Service..."))
                .Execute();
        }
        catch (Exception)
        {
            Console.WriteLine("Transaction failed. Compensating...");
        }
    }
}

class Saga
{
    private readonly List<Action> steps = new List<Action>();

    public Saga Step(Action step)
    {
        steps.Add(step);
        return this;
    }

    public void Execute()
    {
        foreach (var step in steps)
        {
            step();
        }
    }
}
```

**Command Pattern:**
 - Ideal se você precisa encapsular cada etapa do processo como um comando discreto. Por exemplo, você pode ter comandos como CaptureCreditCard, CreateSubscription e ActivateService.
 - Isso é útil para manter responsabilidades bem separadas, facilitando o teste e a manutenção do código.

```csharp
// C#
interface ICommand
{
    void Execute();
}

class CaptureCreditCard : ICommand
{
    public void Execute()
    {
        Console.WriteLine("Capturing Credit Card...");
    }
}

class CreateSubscription : ICommand
{
    public void Execute()
    {
        Console.WriteLine("Creating Subscription...");
    }
}

class ActivateService : ICommand
{
    public void Execute()
    {
        Console.WriteLine("Activating Service...");
    }
}

class CommandExample
{
    static void Main(string[] args)
    {
        ICommand[] commands = {
            new CaptureCreditCard(),
            new CreateSubscription(),
            new ActivateService()
        };

        foreach (var command in commands)
        {
            command.Execute();
        }
    }
}
```

```java
// Java
interface Command {
    void execute();
}

class CaptureCreditCard implements Command {
    public void execute() {
        System.out.println("Capturing Credit Card...");
    }
}

class CreateSubscription implements Command {
    public void execute() {
        System.out.println("Creating Subscription...");
    }
}

class ActivateService implements Command {
    public void execute() {
        System.out.println("Activating Service...");
    }
}

public class CommandExample {
    public static void main(String[] args) {
        Command[] commands = {
            new CaptureCreditCard(),
            new CreateSubscription(),
            new ActivateService()
        };

        for (Command command : commands) {
            command.execute();
        }
    }
}
```

**State Machine Pattern:**
 - Se o processo de captura de cartão e ativação de serviços envolver múltiplos estados (como "captura pendente", "assinatura criada", "serviço ativado"), uma máquina de estados pode ajudar a organizar e controlar essas transições de maneira clara.

```csharp
// C#
enum State
{
    Captured,
    Subscribed,
    Activated
}

class StateMachineExample
{
    static void Main(string[] args)
    {
        State state = State.Captured;

        switch (state)
        {
            case State.Captured:
                Console.WriteLine("Credit Card Captured.");
                state = State.Subscribed;
                break;
            case State.Subscribed:
                Console.WriteLine("Subscription Created.");
                state = State.Activated;
                break;
            case State.Activated:
                Console.WriteLine("Service Activated.");
                break;
        }
    }
}
```

```java
// Java
enum State {
    CAPTURED, SUBSCRIBED, ACTIVATED
}

public class StateMachineExample {
    public static void main(String[] args) {
        State state = State.CAPTURED;

        switch (state) {
            case CAPTURED:
                System.out.println("Credit Card Captured.");
                state = State.SUBSCRIBED;
            case SUBSCRIBED:
                System.out.println("Subscription Created.");
                state = State.ACTIVATED;
            case ACTIVATED:
                System.out.println("Service Activated.");
        }
    }
}
```


----

# Design Patterns - Características Principais

| **Padrão de Design**      | **Descrição**                                                                 | **Características Identificáveis**                                                                                     |
|----------------------------|-------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------|
| **Singleton**             | Garante uma única instância global de uma classe.                            | Método estático `GetInstance`, construtor privado, variável estática para instância.                                   |
| **Factory Method**        | Permite a criação de objetos sem especificar a classe concreta.              | Classe com métodos abstratos/fábrica, subclasse decide qual objeto instanciar.                                        |
| **Abstract Factory**      | Fornece uma interface para criar famílias de objetos relacionados.           | Métodos para criar objetos de diferentes famílias, implementação consistente entre famílias.                           |
| **Builder**               | Separa a construção de um objeto complexo da sua representação final.        | Classe separada para construir objetos passo a passo, uso de métodos encadeados (`fluent interface`).                  |
| **Prototype**             | Cria novos objetos clonando uma instância existente.                         | Implementação de um método `Clone`, reutilização de estados de objetos existentes.                                     |
| **Adapter**               | Permite que interfaces incompatíveis trabalhem juntas.                       | Classe intermediária que converte uma interface para outra (ex.: `ConvertToXYZ()`).                                    |
| **Decorator**             | Permite adicionar funcionalidades a objetos dinamicamente.                   | Classe que "envolve" outra classe, implementação da interface original com funcionalidade extra.                       |
| **Observer**              | Define uma relação "um-para-muitos" entre objetos.                           | Classe `Subject` com lista de observadores, métodos como `Attach`, `Detach` e `Notify`.                                |
| **Mediator**              | Centraliza a comunicação entre objetos para reduzir interdependências.       | Classe mediadora que gerencia a interação entre componentes, objetos não comunicam diretamente entre si.               |
| **Strategy**              | Define uma família de algoritmos e os torna intercambiáveis.                 | Interface ou classe base para algoritmos, uso de composição para alternar implementações em tempo de execução.         |
| **Command**               | Encapsula uma solicitação como um objeto, permitindo desfazer ou enfileirar. | Implementação de uma interface `Command`, métodos como `Execute` e `Undo`.                                            |
| **Chain of Responsibility** | Permite que múltiplos objetos tenham a chance de processar uma solicitação. | Implementação de métodos `HandleRequest` em cadeia, passando requisições de um objeto para outro até serem resolvidas. |
| **Composite**             | Trata objetos individuais e composições de objetos de maneira uniforme.      | Classe base comum, estrutura hierárquica com métodos para acessar e manipular componentes (ex.: `Add` e `Remove`).     |
| **Proxy**                 | Fornece um substituto ou intermediário para controlar o acesso a um objeto.   | Classe com a mesma interface do objeto real, implementa lógica de controle antes de delegar chamadas ao objeto real.   |
| **Ambassador**            | Atua como um intermediário especializado para lidar com comunicação externa. | Proxies que implementam autenticação, monitoramento e roteamento para facilitar integrações.                           |
| **Pipes and Filters**     | Permite o processamento sequencial de dados através de uma cadeia de etapas. | Estrutura modular onde cada componente (filtro) realiza uma transformação nos dados antes de passá-los adiante.        |


