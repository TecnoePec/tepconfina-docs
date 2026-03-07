# Setup do Backend

Guia completo para configurar e executar o backend do TepConfina localmente.

## Pre-requisitos

| Ferramenta    | Versao Minima | Observacao                          |
|---------------|---------------|-------------------------------------|
| .NET SDK      | 10.0          | [Download](https://dotnet.microsoft.com) |
| PostgreSQL    | 15            | Via Docker ou instalacao local      |
| Redis         | 7             | Via Docker ou instalacao local      |
| Git           | 2.40+         | Controle de versao                  |

!!! warning "Versao do .NET"
    O projeto utiliza .NET 10. Certifique-se de nao usar versoes anteriores, pois existem APIs especificas desta versao em uso.

## Clonando o Repositorio

```bash
git clone git@github.com:TecnoePec/tepconfina-api.git
cd tepconfina-api
```

## Configuracao de Ambiente

Copie o arquivo de exemplo de variaveis de ambiente:

```bash
cp .env.example .env
```

Edite o `.env` com suas configuracoes locais:

```env
ASPNETCORE_ENVIRONMENT=Development
DATABASE_HOST=localhost
DATABASE_PORT=5432
DATABASE_NAME=tepconfina
DATABASE_USER=tep_sales
DATABASE_PASSWORD=tep_sales_dev
REDIS_CONNECTION=localhost:6379
JWT_SECRET=sua-chave-secreta-com-pelo-menos-32-caracteres
JWT_ISSUER=TepConfina
JWT_AUDIENCE=TepConfinaClient
```

## Restaurando Dependencias

```bash
dotnet restore
dotnet build
```

!!! tip "Verificar build"
    Execute `dotnet build` antes de rodar. Se houver erros de compilacao, verifique se o SDK .NET 10 esta corretamente instalado.

## Banco de Dados

### Aplicando Migrations

```bash
./scripts/migrate.sh apply
```

O script aceita os seguintes comandos:

| Comando    | Descricao                                     |
|------------|-----------------------------------------------|
| `status`   | Mostra migrations pendentes                   |
| `apply`    | Aplica todas as migrations pendentes          |
| `rollback` | Reverte a ultima migration                    |
| `dry-run`  | Mostra o SQL que sera executado sem aplicar    |

### Seed de Dados

Ao iniciar em modo `Development`, o sistema cria automaticamente o usuario administrador:

| Campo | Valor                      |
|-------|----------------------------|
| Email | `admin@tepconfina.com`     |
| Senha | `Admin@123`                |

## Executando o Projeto

```bash
dotnet run --project src/TepConfina.API
```

A API estara disponivel em `http://localhost:5000`.

!!! success "Verificacao"
    Acesse `http://localhost:5000/health` para confirmar que a API esta respondendo corretamente.

## Estrutura do Projeto

```
src/
  TepConfina.API/            # Controllers, Middlewares, DI
  TepConfina.Application/    # Services, DTOs, Interfaces
  TepConfina.Domain/         # Entidades, Enums, Value Objects
  TepConfina.Infrastructure/ # EF Core, Repositories, Cache
```

## Solucao de Problemas

!!! danger "Erro de conexao com PostgreSQL"
    Verifique se o container Docker do PostgreSQL esta rodando: `docker ps | grep postgres`. Caso necessario, suba com `docker-compose up -d postgres`.

!!! danger "Erro de conexao com Redis"
    Verifique se o Redis esta acessivel: `redis-cli ping`. A resposta esperada e `PONG`.
