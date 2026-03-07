# Testes do Backend

Estrategia e estrutura de testes do backend .NET do TepConfina.

## Framework e Ferramentas

| Ferramenta         | Versao | Funcao                              |
|--------------------|--------|-------------------------------------|
| xUnit              | 2.x    | Framework de testes                 |
| NSubstitute        | 5.x    | Biblioteca de mocking               |
| FluentAssertions   | 8.x    | Assertions expressivas              |
| WebApplicationFactory | -   | Testes de integracao ASP.NET Core   |

## Estrutura do Projeto

```
tests/
  TepConfina.UnitTests/
    Services/
      LoteServiceTests.cs
      AnimalServiceTests.cs
      PesagemServiceTests.cs
      RacaoServiceTests.cs
      MedicamentoServiceTests.cs
      OutrosGastosServiceTests.cs
  TepConfina.IntegrationTests/
    CustomWebApplicationFactory.cs
    AuthHelper.cs
    Controllers/
      AuthControllerTests.cs
      LotesControllerTests.cs
      AnimaisControllerTests.cs
      ...
```

## Testes Unitarios (28 testes)

### Padrao Arrange-Act-Assert

Todos os testes seguem o padrao AAA:

```csharp
public class LoteServiceTests
{
    private readonly ILoteRepository _loteRepository;
    private readonly ICurrentUserProvider _currentUserProvider;
    private readonly LoteService _sut;

    public LoteServiceTests()
    {
        _loteRepository = Substitute.For<ILoteRepository>();
        _currentUserProvider = Substitute.For<ICurrentUserProvider>();
        _currentUserProvider.TenantId.Returns(Guid.NewGuid());
        _sut = new LoteService(_loteRepository, _currentUserProvider);
    }

    [Fact]
    public async Task CriarLote_ComDadosValidos_DeveRetornarLoteDto()
    {
        // Arrange
        var request = new CriarLoteRequest
        {
            Nome = "Lote 001",
            QuantidadeAnimais = 30,
            PesoLiquidoEntrada = 15000
        };

        _loteRepository.AddAsync(Arg.Any<Lote>())
            .Returns(Task.CompletedTask);

        // Act
        var resultado = await _sut.CriarAsync(request);

        // Assert
        resultado.Should().NotBeNull();
        resultado.Nome.Should().Be("Lote 001");
        resultado.PesoMedioEntrada.Should().Be(500);
    }
}
```

### Suites de Testes Unitarios

| Suite                       | Qtd | Cenarios Cobertos                          |
|-----------------------------|-----|--------------------------------------------|
| LoteServiceTests            | 8   | CRUD, fechar lote, calculos KPI            |
| AnimalServiceTests          | 5   | CRUD, mudanca de status, validacoes        |
| PesagemServiceTests         | 5   | Registro, calculo GMD, historico           |
| RacaoServiceTests           | 4   | Compra, consumo, estoque                   |
| MedicamentoServiceTests     | 3   | Aplicacao, custos, validacoes              |
| OutrosGastosServiceTests    | 3   | Registro, totalizacao, validacoes          |

### Mocking com NSubstitute

```csharp
// Configurar retorno do mock
_loteRepository.GetByIdAsync(loteId)
    .Returns(new Lote { Id = loteId, Nome = "Lote 001" });

// Verificar chamada ao mock
await _loteRepository.Received(1).UpdateAsync(Arg.Any<Lote>());

// Verificar argumento especifico
await _loteRepository.Received(1).UpdateAsync(
    Arg.Is<Lote>(l => l.Status == LoteStatus.Fechado));
```

## Testes de Integracao (22 testes)

### CustomWebApplicationFactory

Configura o ambiente de testes com banco de dados em memoria:

```csharp
public class CustomWebApplicationFactory : WebApplicationFactory<Program>
{
    protected override void ConfigureWebHost(IWebHostBuilder builder)
    {
        builder.ConfigureServices(services =>
        {
            // Substituir PostgreSQL por banco em memoria
            var descriptor = services.SingleOrDefault(
                d => d.ServiceType == typeof(DbContextOptions<AppDbContext>));
            services.Remove(descriptor);

            services.AddDbContext<AppDbContext>(options =>
            {
                options.UseInMemoryDatabase("TestDb");
            });

            // Seed de dados de teste
            var sp = services.BuildServiceProvider();
            using var scope = sp.CreateScope();
            var db = scope.ServiceProvider.GetRequiredService<AppDbContext>();
            db.Database.EnsureCreated();
            SeedTestData(db);
        });
    }
}
```

### AuthHelper

Utilitario para autenticar nos testes de integracao:

```csharp
public static class AuthHelper
{
    public static async Task<string> GetTokenAsync(HttpClient client)
    {
        var response = await client.PostAsJsonAsync("/api/auth/login",
            new { Email = "admin@tepconfina.com", Password = "Admin@123" });

        var result = await response.Content
            .ReadFromJsonAsync<LoginResponse>();

        return result.Token;
    }

    public static void AddAuth(HttpClient client, string token)
    {
        client.DefaultRequestHeaders.Authorization =
            new AuthenticationHeaderValue("Bearer", token);
    }
}
```

### Exemplo de Teste de Integracao

```csharp
public class LotesControllerTests : IClassFixture<CustomWebApplicationFactory>
{
    private readonly HttpClient _client;

    public LotesControllerTests(CustomWebApplicationFactory factory)
    {
        _client = factory.CreateClient();
    }

    [Fact]
    public async Task GetLotes_Autenticado_DeveRetornar200()
    {
        // Arrange
        var token = await AuthHelper.GetTokenAsync(_client);
        AuthHelper.AddAuth(_client, token);

        // Act
        var response = await _client.GetAsync("/api/lotes");

        // Assert
        response.StatusCode.Should().Be(HttpStatusCode.OK);
        var lotes = await response.Content
            .ReadFromJsonAsync<List<LoteDto>>();
        lotes.Should().NotBeNull();
    }
}
```

## Executando os Testes

### Todos os testes

```bash
dotnet test TepConfina.sln
```

### Apenas testes unitarios

```bash
dotnet test tests/TepConfina.UnitTests
```

### Apenas testes de integracao

```bash
dotnet test tests/TepConfina.IntegrationTests
```

### Com cobertura

```bash
dotnet test --collect:"XPlat Code Coverage"
```

!!! tip "Relatorio de cobertura"
    Use `reportgenerator` para gerar um relatorio HTML a partir do arquivo de cobertura.

## Boas Praticas

- Cada teste deve ser independente e isolado
- Use nomes descritivos: `Metodo_Cenario_ResultadoEsperado`
- Mock apenas dependencias externas (repositorios, servicos)
- Testes de integracao usam banco em memoria para velocidade
- Mantenha os testes rapidos (< 1s por teste unitario)
