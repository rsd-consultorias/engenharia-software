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
