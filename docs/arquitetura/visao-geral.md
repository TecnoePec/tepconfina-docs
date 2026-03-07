# Visao Geral da Arquitetura

O TepConfina segue uma arquitetura distribuida com tres aplicacoes independentes (backend, frontend e mobile) que se comunicam via API REST. O backend adota **Clean Architecture**, garantindo separacao de responsabilidades e testabilidade.

---

## Diagrama de Contexto (C4 - Nivel 1)

```mermaid
C4Context
    title Diagrama de Contexto - TepConfina

    Person(user, "Usuario", "Gestor de confinamento, veterinario ou administrador")
    Person(producer, "Produtor", "Proprietario dos animais confinados")

    System(tepconfina, "TepConfina", "Plataforma SaaS de gestao de confinamento bovino")

    System_Ext(firebase, "Firebase", "Push notifications")
    System_Ext(aws, "AWS", "Hospedagem e infraestrutura")

    Rel(user, tepconfina, "Gerencia lotes, animais, pesagens, racoes")
    Rel(producer, tepconfina, "Consulta relatorios e KPIs")
    Rel(tepconfina, firebase, "Envia notificacoes push")
    Rel(tepconfina, aws, "Hospedado em ECS Fargate")
```

---

## Arquitetura de Aplicacoes

```mermaid
graph TB
    subgraph Cliente
        WEB[Frontend React 18<br/>Vite 5 + TypeScript]
        MOB[Mobile Flutter 3<br/>Riverpod + Hive]
    end

    subgraph Backend
        API[API .NET 10<br/>Controllers + Middleware]
        APP[Application<br/>Services + DTOs]
        DOM[Domain<br/>Entities + Interfaces]
        INF[Infrastructure<br/>EF Core + Redis]
    end

    subgraph Dados
        PG[(PostgreSQL 15)]
        RD[(Redis 7)]
    end

    WEB -->|HTTPS/REST| API
    MOB -->|HTTPS/REST| API
    API --> APP
    APP --> DOM
    APP --> INF
    INF --> PG
    INF --> RD

    style DOM fill:#1B3A5C,color:#fff
    style APP fill:#2E7D32,color:#fff
    style INF fill:#E65100,color:#fff
    style API fill:#0277BD,color:#fff
```

---

## Clean Architecture

O backend e organizado em quatro camadas concentricas. A **regra de dependencia** determina que camadas externas podem depender das internas, mas nunca o contrario.

```mermaid
graph TB
    subgraph "Camada 4 - API"
        C[Controllers]
        MW[Middleware]
        PROG[Program.cs]
    end

    subgraph "Camada 3 - Infrastructure"
        DB[DbContext / EF Core]
        REPO[Repositories]
        CACHE[Redis Cache]
    end

    subgraph "Camada 2 - Application"
        SVC[Services]
        DTO[DTOs]
        VAL[Validators]
        IFACE_APP[Interfaces]
    end

    subgraph "Camada 1 - Domain"
        ENT[Entities]
        ENUM[Enums]
        IFACE_DOM[Interfaces]
    end

    C --> SVC
    SVC --> ENT
    SVC --> IFACE_APP
    REPO -.->|implementa| IFACE_DOM
    DB --> ENT

    style ENT fill:#1B3A5C,color:#fff
    style ENUM fill:#1B3A5C,color:#fff
    style IFACE_DOM fill:#1B3A5C,color:#fff
```

| Camada             | Projeto                     | Responsabilidade                                    |
|:-------------------|:----------------------------|:----------------------------------------------------|
| **Domain**         | `TepConfina.Domain`         | Entidades, enums, interfaces de repositorio         |
| **Application**    | `TepConfina.Application`    | Servicos de negocio, DTOs, validadores, interfaces  |
| **Infrastructure** | `TepConfina.Infrastructure` | EF Core, repositorios, cache Redis, integracao      |
| **API**            | `TepConfina.API`            | Controllers, middleware, configuracao, DI            |

!!! warning "Regra de Dependencia"
    A camada **Domain** nao referencia nenhuma outra camada. Ela define interfaces que sao implementadas pela **Infrastructure**. A inversao de dependencia (DIP) e aplicada via injecao de dependencia no `Program.cs`.

---

## Estrutura de Repositorios

O projeto esta dividido em **tres repositorios independentes** para permitir CI/CD isolado e escalabilidade de equipe.

| Repositorio               | Conteudo                              | CI/CD                    |
|:--------------------------|:--------------------------------------|:-------------------------|
| `tepconfina-api`          | Backend .NET + Terraform              | GitHub Actions → AWS ECS |
| `tepconfina-web`          | Frontend React + Vite                 | GitHub Actions → S3/CloudFront |
| `tepconfina-mobile`       | App Flutter                           | GitHub Actions → App Store/Play Store |

---

## Padroes de Design

| Padrao                  | Aplicacao no TepConfina                                      |
|:------------------------|:-------------------------------------------------------------|
| **Repository Pattern**  | `Repository<T>` generico + repositorios especializados       |
| **CQRS-lite**           | Separacao de queries e commands nos services                  |
| **Soft Delete**         | `IsDeleted` + global query filter no EF Core                 |
| **Multi-tenancy**       | Coluna `TenantId` em todas as entidades + filtro global      |
| **DTO Mapping**         | Mapeamento manual com calculo de KPIs nos DTOs de resposta   |
| **Refresh Token**       | JWT de curta duracao + refresh token para renovacao silenciosa|
| **Offline Queue**       | Fila de mutacoes no mobile, sincronizada ao reconectar       |

!!! tip "Por que CQRS-lite?"
    Em vez de implementar CQRS completo com event sourcing e filas de mensagens, o TepConfina adota uma versao simplificada: os services possuem metodos separados para leitura (`Get`, `List`) e escrita (`Create`, `Update`, `Delete`), mantendo a simplicidade sem overhead de infraestrutura.

---

*Proximas paginas: [Backend](backend.md) | [Frontend](frontend.md) | [Mobile](mobile.md) | [Decisoes Tecnicas](decisoes-tecnicas.md)*
