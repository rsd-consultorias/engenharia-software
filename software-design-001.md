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
using Confluent.Kafka;
using Microsoft.AspNetCore.Mvc;
using Npgsql;
using System.Text.Json;

namespace PipesAndFilters.Controllers
{
    public class Pipeline
    {
        public static TOutput Execute<TInput, TOutput>(TInput input, params Func<TInput, TOutput>[] filters)
        {
            foreach (var filter in filters)
            {
                input = (TInput)(object)filter((TInput)input);
            }
            return (TOutput)(object)input;
        }
    }

    public class KafkaController : Controller
    {
        private const string KafkaTopic = "meu_topico";
        private const string KafkaServer = "localhost:9092";
        private const string ConnectionString = "Host=localhost;Username=meuusuario;Password=minhasenha;Database=meubanco";

        public void ProcessEvents()
        {
            var config = new ConsumerConfig
            {
                BootstrapServers = KafkaServer,
                GroupId = "consumer-group",
                AutoOffsetReset = AutoOffsetReset.Earliest
            };

            using var consumer = new ConsumerBuilder<Ignore, string>(config).Build();
            consumer.Subscribe(KafkaTopic);

            using var connection = new NpgsqlConnection(ConnectionString);
            connection.Open();

            while (true)
            {
                var result = consumer.Consume();
                var rawEvent = JsonSerializer.Deserialize<Event>(result.Message.Value);

                // Aplicando o padrão Pipes and Filters
                Pipeline.Execute(rawEvent,
                    TransformEvent, // Filtro 1: Transformar
                    e => SaveToDatabase(e, connection) // Filtro 2: Persistir no banco
                );
            }
        }

        private Event TransformEvent(Event rawEvent)
        {
            return new Event
            {
                Id = rawEvent.Id,
                Name = rawEvent.Name.ToUpper(),
                Timestamp = rawEvent.Timestamp
            };
        }

        private Event SaveToDatabase(Event transformedEvent, NpgsqlConnection connection)
        {
            using var cmd = new NpgsqlCommand("INSERT INTO events (id, name, timestamp) VALUES (@id, @name, @timestamp)", connection);
            cmd.Parameters.AddWithValue("id", transformedEvent.Id);
            cmd.Parameters.AddWithValue("name", transformedEvent.Name);
            cmd.Parameters.AddWithValue("timestamp", transformedEvent.Timestamp);
            cmd.ExecuteNonQuery();

            return transformedEvent; // Opcional, para fins de pipeline
        }
    }

    public class Event
    {
        public int Id { get; set; }
        public string Name { get; set; }
        public DateTime Timestamp { get; set; }
    }
}
```

```java
import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.clients.consumer.ConsumerRecords;
import org.apache.kafka.clients.consumer.KafkaConsumer;
import org.apache.kafka.clients.consumer.ConsumerRecord;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;
import java.util.function.Function;

public class KafkaPipeline {

    private static final String TOPIC = "meu_topico";
    private static final String BOOTSTRAP_SERVERS = "localhost:9092";
    private static final String DB_URL = "jdbc:postgresql://localhost/meubanco";
    private static final String DB_USER = "meuusuario";
    private static final String DB_PASSWORD = "minhasenha";

    public static void main(String[] args) {
        Properties props = new Properties();
        props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, BOOTSTRAP_SERVERS);
        props.put(ConsumerConfig.GROUP_ID_CONFIG, "consumer-group");
        props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringDeserializer");
        props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringDeserializer");
        props.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");

        try (KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
             Connection connection = DriverManager.getConnection(DB_URL, DB_USER, DB_PASSWORD)) {

            consumer.subscribe(Collections.singletonList(TOPIC));

            while (true) {
                ConsumerRecords<String, String> records = consumer.poll(1000);

                for (ConsumerRecord<String, String> record : records) {
                    Event rawEvent = parseEvent(record.value());

                    // Aplicando o padrão Pipes and Filters
                    Event result = applyPipeline(
                        rawEvent,
                        KafkaPipeline::transformEvent, // Filtro 1: Transformar
                        event -> saveToDatabase(event, connection) // Filtro 2: Persistir no banco
                    );
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    private static Event parseEvent(String json) {
        // Simulação simples de parsing
        return new Event(json); // Ajuste para sua lógica de parsing
    }

    private static Event transformEvent(Event event) {
        event.setName(event.getName().toUpperCase());
        return event;
    }

    private static Event saveToDatabase(Event event, Connection connection) throws Exception {
        String query = "INSERT INTO events (id, name, timestamp) VALUES (?, ?, ?)";
        try (PreparedStatement stmt = connection.prepareStatement(query)) {
            stmt.setInt(1, event.getId());
            stmt.setString(2, event.getName());
            stmt.setTimestamp(3, new java.sql.Timestamp(event.getTimestamp().getTime()));
            stmt.executeUpdate();
        }
        return event; // Opcional, para fins de pipeline
    }

    @SafeVarargs
    private static <T> T applyPipeline(T input, Function<T, T>... filters) {
        for (Function<T, T> filter : filters) {
            input = filter.apply(input);
        }
        return input;
    }

    static class Event {
        private int id;
        private String name;
        private java.util.Date timestamp;

        // Getters e setters
        public int getId() { return id; }
        public void setId(int id) { this.id = id; }

        public String getName() { return name; }
        public void setName(String name) { this.name = name; }

        public java.util.Date getTimestamp() { return timestamp; }
        public void setTimestamp(java.util.Date timestamp) { this.timestamp = timestamp; }
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


