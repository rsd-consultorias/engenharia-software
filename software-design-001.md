# ASP.NET Core MVC: Controller como Mediator e uso de Ambassador

Este exemplo demonstra como estruturar um *Controller* em ASP.NET Core MVC atuando como um *mediator* e chamando uma classe *Ambassador*. Além disso, inclui uma implementação de uma classe *Mediator* separada, que centraliza a lógica empresarial.

## Estrutura de Código

### Controller (Mediator)
```csharp
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


