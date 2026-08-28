### 1. Diagrama de Componentes (Component Diagram)

Este diagrama ilustra a divisão física e lógica do ecossistema. No frontend, adotamos um padrão de **Microfrontends baseado em Next.js (Multi-Zone/Module Federation)** com um **BFF (Backend for Frontend)** integrado. No backend, mapeamos as fronteiras do **Hexágono** do Spring Boot.

```mermaid
graph TB
    subgraph Client_Browser ["Navegador do Cliente (Estilos via Tailwind)"]
        direction TB
        subgraph MFE_Architecture ["Arquitetura de Microfrontends (Next.js Client)"]
            Shell["MFE Host / Shell <br> (Roteamento, Auth.js, Layouts)"]
            MFE_CRUD["MFE CRUD Remote <br> (Listagem, Cadastro, Edição)"]
            Shared_UI["MFE Shared Components <br> (Design System, Botões, inputs)"]
            
            Shell -->|Lazy Loading / Zones| MFE_CRUD
            MFE_CRUD -->|Importa UI| Shared_UI
        end
    end

    subgraph Nextjs_BFF_Server ["Servidor Next.js (BFF Node.js Runtime)"]
        direction TB
        Route_Handlers["Route Handlers (/api/*) <br> (Proxy Seguro)"]
        Next_Middleware["Next.js Middleware <br> (Verificação de Rotas e Sessão)"]
        
        Shell -->|Requests locais /api/crud| Route_Handlers
        Route_Handlers -->|Injeta JWT via httpOnly Cookie| Next_Middleware
    end

    subgraph Backend_Hexagonal ["Backend Spring Boot (JVM Container)"]
        direction TB
        subgraph Adapters_In ["Driving Adapters (Entrada)"]
            REST_Controller["REST Controller <br> (@RestController)"]
        end
        
        subgraph Domain_Hexagon ["Núcleo do Domínio (Hexágono Secundário)"]
            In_Ports["Portas de Entrada (Interfaces) <br> (Use Cases)"]
            Domain_Services["Serviços de Domínio <br> (Lógica de Validação CRUD)"]
            Out_Ports["Portas de Saída (Interfaces) <br> (SPIs / Repositories)"]
            Domain_Entities["Entidades de Domínio <br> (Modelos Puros)"]
            
            In_Ports --> Domain_Services
            Domain_Services --> Domain_Entities
            Domain_Services --> Out_Ports
        end

        subgraph Adapters_Out ["Driven Adapters (Saída)"]
            JPA_Adapter["JPA Repository Adapter <br> (Spring Data JPA)"]
            Redis_Adapter["Redis Cache Adapter <br> (Spring Data Redis)"]
        end
        
        REST_Controller -->|Invoca| In_Ports
        Out_Ports -->|Implementado por| JPA_Adapter
        Out_Ports -->|Implementado por| Redis_Adapter
    end

    subgraph Data_Storage ["Persistência de Dados"]
        Postgres_DB[("PostgreSQL Database <br> (Esquema via Flyway)")]
        Redis_DB[("Redis Cache Store")]
    end

    subgraph Observability_Stack ["Stack de Observabilidade & Monitoramento"]
        Prometheus["Prometheus <br> (Coleta Métricas)"]
        Grafana["Grafana <br> (Dashboards)"]
        Loki["Grafana Loki / Promtail <br> (Coleta Logs)"]
    end

    %% Roteamento do BFF para o Backend com resiliência
    Route_Handlers -->|HTTPS + Session Cookie JWT| REST_Controller
    
    %% Conexão dos Adaptadores com os Bancos
    JPA_Adapter -->|Conexão JDBC| Postgres_DB
    Redis_Adapter -->|Lettuce Driver| Redis_DB

    %% Observabilidade / Resiliência
    REST_Controller -.->|Métricas via Micrometer| Prometheus
    Route_Handlers -.->|Logs de Acesso| Loki
    JPA_Adapter -.->|Tracing e Conexões de Pool HikariCP| Prometheus
```

---

### 2. Diagrama de Classes (Class Diagram)

Este diagrama representa a implementação clássica das classes de uma entidade de negócio (ex: `Product`) no backend usando a **Arquitetura Hexagonal**, conectada à tipagem do Frontend para manter o **Type Safety**.

```mermaid
classDiagram
    %% --- FRONTEND TYPING (TypeScript) ---
    class ProductTS {
        <<interface>>
        +id: string
        +name: string
        +price: number
        +quantity: number
        +createdAt: string
    }

    class ProductCreateInputTS {
        <<interface>>
        +name: string
        +price: number
        +quantity: number
    }

    %% --- BACKEND DOMAIN CORE ---
    class Product {
        -UUID id
        -String name
        -BigDecimal price
        -Integer quantity
        -Instant createdAt
        +Product(UUID id, String name, BigDecimal price, Integer quantity)
        +getId() UUID
        +getName() String
        +getPrice() BigDecimal
        +getQuantity() Integer
        +getCreatedAt() Instant
        +validate() void
    }

    %% --- INBOUND PORTS (Interfaces de Caso de Uso) ---
    class ManageProductUseCase {
        <<interface>>
        +createProduct(ProductCreateCommand command) Product
        +getProductById(UUID id) Product
        +updateProduct(UUID id, ProductUpdateCommand command) Product
        +deleteProduct(UUID id) void
    }

    class ProductCreateCommand {
        -String name
        -BigDecimal price
        -Integer quantity
        +ProductCreateCommand(String name, BigDecimal price, Integer quantity)
        +getName() String
        +getPrice() BigDecimal
        +getQuantity() Integer
    }

    %% --- DOMAIN SERVICES (Implementações do Caso de Uso) ---
    class ProductDomainService {
        -ProductDatabasePort databasePort
        +ProductDomainService(ProductDatabasePort databasePort)
        +createProduct(ProductCreateCommand command) Product
        +getProductById(UUID id) Product
        +updateProduct(UUID id, ProductUpdateCommand command) Product
        +deleteProduct(UUID id) void
    }

    %% --- OUTBOUND PORTS (Interfaces de Persistência / SPI) ---
    class ProductDatabasePort {
        <<interface>>
        +save(Product product) Product
        +findById(UUID id) Optional~Product~
        +delete(UUID id) void
        +existsById(UUID id) boolean
    }

    %% --- INBOUND ADAPTERS (REST Controllers & DTOs) ---
    class ProductRestController {
        -ManageProductUseCase useCase
        +ProductRestController(ManageProductUseCase useCase)
        +create(ProductRequestDTO dto) ResponseEntity~ProductResponseDTO~
        +getById(UUID id) ResponseEntity~ProductResponseDTO~
    }

    class ProductRequestDTO {
        -String name
        -BigDecimal price
        -Integer quantity
        +ProductRequestDTO()
        +getName() String
        +getPrice() BigDecimal
        +getQuantity() Integer
    }

    class ProductResponseDTO {
        -UUID id
        -String name
        -BigDecimal price
        -Integer quantity
        -Instant createdAt
        +ProductResponseDTO(UUID id, String name, BigDecimal price, Integer quantity, Instant createdAt)
        +getId() UUID
        +getName() String
        +getPrice() BigDecimal
        +getQuantity() Integer
        +getCreatedAt() Instant
    }

    %% --- OUTBOUND ADAPTERS (JPA Adapters & Entities) ---
    class ProductJpaAdapter {
        -SpringDataProductRepository repository
        +ProductJpaAdapter(SpringDataProductRepository repository)
        +save(Product product) Product
        +findById(UUID id) Optional~Product~
        +delete(UUID id) void
        +existsById(UUID id) boolean
    }

    class ProductJpaEntity {
        -UUID id
        -String name
        -BigDecimal price
        -Integer quantity
        -Instant createdAt
        +ProductJpaEntity()
        +ProductJpaEntity(UUID id, String name, BigDecimal price, Integer quantity)
        +getId() UUID
        +setId(UUID id) void
        +getName() String
        +setName(String name) void
    }

    class SpringDataProductRepository {
        <<interface>>
    }

    %% --- RELACIONAMENTOS ---
    ProductCreateInputTS ..> ProductTS : "Define base"
    ProductRestController ..> ProductRequestDTO : "Recebe payload"
    ProductRestController ..> ProductResponseDTO : "Retorna payload"
    ProductRequestDTO ..> ProductCreateCommand : "Converte para"
    
    ProductRestController --> ManageProductUseCase : "Usa"
    ProductDomainService ..|> ManageProductUseCase : "Implementa"
    ProductDomainService --> Product : "Manipula"
    ProductDomainService --> ProductDatabasePort : "Usa"
    
    ProductJpaAdapter ..|> ProductDatabasePort : "Implementa"
    ProductJpaAdapter --> SpringDataProductRepository : "Delega para"
    ProductJpaAdapter ..> ProductJpaEntity : "Converte de/para"
    ProductJpaEntity <.. SpringDataProductRepository : "Gerencia"
```

---

### 3. Diagrama de Implantação (Deployment Diagram)

Este diagrama detalha como a aplicação será hospedada, orquestrada, escalada de forma resiliente e conteinerizada através do **Docker/Docker Compose**. Ele inclui as barreiras de proteção de borda (**Caddy/Nginx**), a camada de aplicação isolada e a monitoração ativa.

```mermaid
flowchart TB
    %% --- CLIENT ---
    subgraph Client_Machine["Computador do Cliente"]
        Browser_Runtime["Navegador Web / Client Chrome-Safari"]
    end

    %% --- PRODUCTION CLUSTER ---
    subgraph Production_Cluster["Servidor de Produção / Docker Swarm or K8s Node"]
        
        %% DMZ / EDGE
        subgraph Edge_Network["Rede Externa / DMZ"]
            subgraph Web_Proxy_Pod["Container: Caddy/Nginx"]
                Reverse_Proxy_Config[["Configurações do Reverse Proxy<br/>SSL via Let's Encrypt"]]
            end
        end

        %% INTERNAL NETWORK
        subgraph Internal_Private_Network["Rede Interna Docker / Overlay Private Network"]
            
            subgraph Frontend_App_Pod["Container Multi-Stage: Next.js Node-Runtime"]
                NextJS_Build[["Production Build .next<br/>Static & Dynamic Server"]]
            end

            subgraph Backend_API_Pod["Container: Spring Boot JVM (Alpine-JDK-Slim)"]
                JAR_Artifact[["packaged-app.jar<br/>Embedded Tomcat"]]
            end

            subgraph Cache_Cluster["Container: Redis Store"]
                Memory_Cache[("Sessões e Cache<br/>de Consultas CRUD")]
            end

            subgraph Database_Pod["Container: PostgreSQL DB Engine"]
                Postgres_Storage[("Postgres Data Vol<br/>Flyway Migrations")]
            end

            subgraph Observability_Pod["Container: Promtail / Loki / Prometheus"]
                Monitor_Metrics[["Métricas de Aplicação Actuator<br/>e Logs"]]
            end
        end
    end

    %% --- CONNECTIONS & TRAFFIC FLOWS ---
    Browser_Runtime -->|"HTTPS (Porta 443)<br/>Acesso de Usuário"| Web_Proxy_Pod
    
    Web_Proxy_Pod -->|"HTTP (Porta 3000) Interno<br/>Roteia MFE Shell e APIs do BFF"| Frontend_App_Pod
    Web_Proxy_Pod -->|"HTTP (Porta 8080) Interno<br/>Opcional: Acesso direto de APIs públicas"| Backend_API_Pod
    
    Frontend_App_Pod -->|"HTTP (Porta 8080) Interno + BFF Token Proxy<br/>Chamada do BFF (Route Handlers) p/ Spring Boot"| Backend_API_Pod
    
    Backend_API_Pod -->|"TCP (Porta 5432)<br/>Persistência JPA (Pool HikariCP)"| Database_Pod
    Backend_API_Pod -->|"TCP (Porta 6379)<br/>Validação de Cache"| Cache_Cluster
    
    Backend_API_Pod -.-|"Scrape /actuator/prometheus (Porta 8080)<br/>Monitoramento Ativo Actuator"| Observability_Pod
    Frontend_App_Pod -.-|"HTTP Log Stream<br/>Logs do Servidor BFF Node"| Observability_Pod
```

---

### Detalhes Técnicos de Resiliência, Observabilidade e CRUD

1. **Resiliência (Backend & Frontend):**
   * **Spring Boot (Circuit Breaker):** Configuração de resiliência usando o *Resilience4j* encapsulando as consultas do banco de dados PostgreSQL. Se o banco apresentar picos de lentidão, o Circuit Breaker abre e respostas amigáveis ou dados em cache (oriundos do Redis) são retornados para a aplicação.
   * **BFF HTTP Retry & Timeout:** O Next.js Route Handler define tempos limite rigorosos (*timeouts*) em todas as requisições repassadas ao Spring Boot. Caso ocorra um erro de rede transitório `503`, o frontend executa tentativas automáticas de requisição (*retries* assíncronos) de forma invisível ao usuário.

2. **Observabilidade Completa:**
   * **Spring Boot Actuator + Micrometer:** Expõe métricas de saúde da aplicação (`/actuator/health`) e dados de desempenho do JVM para o Prometheus.
   * **Logs Estruturados (SLF4J + Logback):** Saída de logs em formato JSON padronizado no console, facilitando a captura automática por coletores como o *Promtail* ou *Fluentd* e indexação imediata no *Grafana Loki*.
   * **Tratamento Global de Exceções:** Implementado com `@RestControllerAdvice` para capturar exceções SQL, de validação e de segurança, evitando a exposição de dados sensíveis na resposta HTTP e registrando adequadamente o erro nos logs de observabilidade.

3. **Arquitetura de Microfrontends (Frontend):**
   * O **Next.js Host (Shell)** atua como orquestrador principal, sendo responsável por prover a folha de estilos global (Tailwind), gerenciar o estado da sessão de login via *Auth.js* e realizar o roteamento.
   * O **MFE CRUD (Remote)** é injetado sob demanda em páginas específicas (ex: `/produtos`), carregando em tempo de execução somente o JavaScript necessário para as listagens e mutações.
