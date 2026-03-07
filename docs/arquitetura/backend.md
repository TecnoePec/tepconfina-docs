# Arquitetura do Backend

O backend do TepConfina e construido com **.NET 10** seguindo **Clean Architecture**. A solucao e composta por quatro projetos organizados em camadas com dependencias unidirecionais.

---

## Estrutura da Solution

```
TepConfina/
├── src/
│   ├── TepConfina.Domain/
│   │   ├── Entities/          # 16 entidades de dominio
│   │   ├── Enums/             # Enumeracoes de negocio
│   │   └── Interfaces/        # Contratos de repositorio
│   │
│   ├── TepConfina.Application/
│   │   ├── Services/          # 18+ servicos de negocio
│   │   ├── DTOs/              # Objetos de transferencia
│   │   ├── Interfaces/        # Contratos de servico
│   │   └── Validators/        # Regras de validacao
│   │
│   ├── TepConfina.Infrastructure/
│   │   ├── Data/
│   │   │   ├── AppDbContext.cs       # DbContext principal
│   │   │   └── Configurations/      # Fluent API configs
│   │   ├── Repositories/            # Implementacoes
│   │   └── Cache/                   # Redis cache service
│   │
│   └── TepConfina.API/
│       ├── Controllers/       # 17 controllers REST
│       ├── Middleware/         # Pipeline customizado
│       ├── Extensions/        # DependencyInjection.cs
│       └── Program.cs         # Entry point e pipeline
│
└── tests/
    ├── TepConfina.UnitTests/
    └── TepConfina.IntegrationTests/
```

---

## Camada Domain

O nucleo do sistema contem as **16 entidades** de dominio, sem dependencias externas.

| Entidade          | Descricao                                    |
|:------------------|:---------------------------------------------|
| `Lote`            | Lote de confinamento com datas e status       |
| `Animal`          | Animal individual com identificacao e raca    |
| `Pesagem`         | Registro de pesagem com peso e data           |
| `Racao`           | Formulacao de racao com ingredientes          |
| `Trato`           | Fornecimento diario de racao ao lote          |
| `PrecoMercado`    | Cotacao de arroba do boi gordo                |
| `Produtor`        | Proprietario dos animais                      |
| `Compra`          | Transacao de compra de animais                |
| `Venda`           | Transacao de venda de animais                 |
| `CustoOperacional`| Custos fixos e variaveis do confinamento      |
| `Alerta`          | Alerta automatico baseado em regras           |
| `Notificacao`     | Notificacao para o usuario                    |
| `Usuario`         | Usuario do sistema com perfil e permissoes    |
| `Tenant`          | Empresa/fazenda (multi-tenancy)               |
| `RefreshToken`    | Token de renovacao JWT                        |
| `AuditLog`        | Registro de auditoria de acoes                |

!!! note "Base Entity"
    Todas as entidades herdam de `BaseEntity`, que fornece `Id`, `CreatedAt`, `UpdatedAt`, `IsDeleted` e `TenantId`. O soft delete e o multi-tenancy sao aplicados via **global query filters** no EF Core.

---

## Camada Application

Contem a logica de negocio encapsulada em **18+ servicos**, cada um com sua interface correspondente.

```mermaid
graph LR
    CTRL[Controller] --> ISVC[IService]
    ISVC -.->|DI| SVC[Service]
    SVC --> IREPO[IRepository]
    IREPO -.->|DI| REPO[Repository]
    SVC --> DTO[DTO]
```

Os servicos realizam:

- Validacao de regras de negocio
- Mapeamento manual entre entidades e DTOs
- Calculo de KPIs (GMD, custo/arroba, resultado financeiro)
- Orquestracao de operacoes entre repositorios

!!! tip "Mapeamento Manual"
    O projeto utiliza mapeamento manual em vez de AutoMapper. Isso permite calcular campos derivados (KPIs) diretamente no mapeamento, mantendo a logica explicita e depuravel.

---

## Camada Infrastructure

Implementa os contratos definidos nas camadas internas.

### DbContext

O `AppDbContext` configura:

- **Global query filters** para `IsDeleted` e `TenantId`
- **Fluent API** para mapeamento de entidades
- **Interceptors** para auditoria automatica (`CreatedAt`, `UpdatedAt`)

### Repositorios

| Repositorio              | Tipo          | Descricao                              |
|:-------------------------|:--------------|:---------------------------------------|
| `Repository<T>`          | Generico      | CRUD basico para todas as entidades    |
| `LoteRepository`         | Especializado | Queries com includes e KPIs            |
| `PrecoMercadoRepository` | Especializado | Cotacoes com filtros por data/praca    |
| `AnimalRepository`       | Especializado | Queries com historico de pesagens      |

### Redis Cache

O cache Redis e utilizado para:

- Cotacoes de mercado (TTL: 15 minutos)
- Dados de dashboard (TTL: 5 minutos)
- Sessoes de refresh token

---

## Camada API

### Controllers

Os **17 controllers** seguem o padrao RESTful com versionamento via URL (`/api/v1/`).

| Controller              | Endpoints | Descricao                         |
|:------------------------|:---------:|:----------------------------------|
| `AuthController`        | 4         | Login, registro, refresh, logout  |
| `LotesController`       | 6         | CRUD + fechar lote + KPIs        |
| `AnimaisController`     | 5         | CRUD + historico de pesagens     |
| `PesagensController`    | 4         | CRUD + pesagem em lote           |
| `RacoesController`      | 4         | CRUD de formulacoes              |
| `TratosController`      | 4         | Registro de tratos diarios       |
| `ProdutoresController`  | 4         | CRUD de produtores               |
| `ComprasController`     | 4         | Registro de compras              |
| `VendasController`      | 4         | Registro de vendas               |
| `CustosController`      | 4         | Gestao de custos operacionais    |
| `PrecoMercadoController`| 3         | Cotacoes de mercado              |
| `AlertasController`     | 4         | Gestao de alertas                |
| `NotificacoesController`| 3         | Listagem e marcacao como lida    |
| `DashboardController`   | 3         | Metricas consolidadas            |
| `UsuariosController`    | 4         | Gestao de usuarios               |
| `TenantsController`     | 3         | Gestao de tenants (admin)        |
| `HealthController`      | 2         | Liveness e readiness probes      |

### Pipeline de Middleware

A ordem dos middlewares no `Program.cs` e critica para o funcionamento correto:

```mermaid
graph TD
    REQ[Request] --> S1[1. Serilog Request Logging]
    S1 --> S2[2. Security Headers]
    S2 --> S3[3. CORS]
    S3 --> S4[4. Authentication]
    S4 --> S5[5. Authorization]
    S5 --> S6[6. Rate Limiting]
    S6 --> S7[7. Business Metrics Middleware]
    S7 --> S8[8. Controllers]
    S8 --> RES[Response]

    style REQ fill:#0277BD,color:#fff
    style RES fill:#2E7D32,color:#fff
```

!!! warning "Ordem dos Middlewares"
    A autenticacao **deve** vir antes da autorizacao, e ambas antes do rate limiting. O middleware de metricas de negocio deve ser o ultimo antes dos controllers para capturar tempos de resposta precisos.

---

## Registro de Dependencias

Toda a configuracao de DI esta centralizada no metodo de extensao `AddTepConfinaServices()` em `DependencyInjection.cs`:

```csharp
public static class DependencyInjection
{
    public static IServiceCollection AddTepConfinaServices(
        this IServiceCollection services,
        IConfiguration configuration)
    {
        // Infrastructure
        services.AddDbContext<AppDbContext>(...);
        services.AddStackExchangeRedisCache(...);

        // Repositories
        services.AddScoped(typeof(IRepository<>), typeof(Repository<>));
        services.AddScoped<ILoteRepository, LoteRepository>();

        // Services
        services.AddScoped<IAuthService, AuthService>();
        services.AddScoped<ILoteService, LoteService>();
        // ... demais servicos

        return services;
    }
}
```

---

## Health Checks

O sistema expoe dois endpoints de health check:

| Endpoint           | Tipo       | Verifica                              |
|:-------------------|:-----------|:--------------------------------------|
| `/health/live`     | Liveness   | Aplicacao esta respondendo            |
| `/health/ready`    | Readiness  | PostgreSQL + Redis conectados         |

---

## Observabilidade

O backend integra **OpenTelemetry** para traces e metricas:

- **Traces**: Requisicoes HTTP, queries EF Core, chamadas Redis
- **Metricas**: Contadores de requisicao, latencia, erros por endpoint
- **Logs**: Serilog estruturado com enrichers de correlacao

!!! tip "Exporters"
    Em desenvolvimento, os dados sao exportados para o console. Em producao, sao enviados para o AWS X-Ray via OTLP exporter.

---

*Paginas relacionadas: [Visao Geral](visao-geral.md) | [Frontend](frontend.md) | [Mobile](mobile.md)*
