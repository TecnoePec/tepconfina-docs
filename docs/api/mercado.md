# Mercado, Preços e Alertas

Endpoints para consulta de cotações do boi gordo (Boi Gordo Futuro B3 — BGI), candles OHLC, histórico de preços e gerenciamento de alertas de preço.

!!! info "Autenticação Obrigatória"
    Todos os endpoints desta seção requerem header `Authorization: Bearer {token}`.

## Provider de Market Data

A API usa uma abstração `IMarketDataProvider` que pode ser trocada via configuração `MarketData:Provider`:

| Provider | Status | Símbolo principal | Observação |
|----------|--------|-------------------|------------|
| `tradingview` | **Ativo** | `BGI1!` (Boi Gordo Futuro continuous, BMFBOVESPA) | Usa sidecar Node.js `tepconfina-tv-scraper` (porta 3001 na task ECS). Preço da arroba real (~R$ 360). |
| `brapi` | Disponível | `BBOI11` (ETF Boi Gordo) | Cota do ETF, **não** é preço da arroba (~R$ 11). Usado quando o sidecar TV não responde. |
| `stonex` | Stub | — | Implementação preparada para integração paga futura. |
| `mock` | Dev | — | Dados sintéticos para testes locais. |

A migração entre providers é transparente para o frontend e o schema do banco — só muda a configuração e o registro de DI.

---

## Resumo dos Endpoints

| Método | Rota | Roles | Descrição |
|--------|------|-------|-----------|
| `GET` | `/api/market/contracts` | Todos | Lista contratos disponíveis no provider ativo |
| `GET` | `/api/market/quote` | Todos | Cotação atual de um contrato |
| `GET` | `/api/market/quotes/history` | Todos | Histórico de cotações |
| `GET` | `/api/market/candles` | Todos | Candles OHLC para gráfico |
| `GET` | `/api/market/provider/health` | Todos | Status do provider ativo |
| `GET` | `/api/precos-mercado/resumo` | Todos | Resumo legacy (alimenta dashboard) |
| `GET` | `/api/precos-mercado/historico` | Todos | Histórico legacy |
| `POST` | `/api/precos-mercado` | Admin, Gerente | Criar registro manual de preço |
| `GET` | `/api/alertas-preco` | Todos | Listar alertas |
| `POST` | `/api/alertas-preco` | Todos | Criar alerta |
| `PUT` | `/api/alertas-preco/{id}` | Todos | Atualizar alerta |
| `DELETE` | `/api/alertas-preco/{id}` | Todos | Remover alerta |

---

## Market Data (novo — pipeline TradingView)

### GET /api/market/contracts

Lista os contratos disponíveis no provider ativo.

**Resposta 200:**

```json
[
  {
    "id": "f7a8b9c0-1234-5678-9abc-def012345678",
    "symbol": "BOI",
    "contractCode": "BGI1!",
    "description": "Boi Gordo Futuro — Continuous Front Month (B3)",
    "expirationDate": null,
    "isActive": true
  },
  {
    "id": "...",
    "symbol": "BOI",
    "contractCode": "BGIK26",
    "description": "Boi Gordo Maio 2026",
    "expirationDate": "2026-05-31T00:00:00Z",
    "isActive": true
  }
]
```

---

### GET /api/market/quote

Cotação atual de um contrato.

**Query Parameters:**

| Parâmetro | Tipo | Obrigatório | Descrição |
|-----------|------|-------------|-----------|
| `symbol` | string | Sim | Ex: `BOI` |
| `contractCode` | string | Não | Ex: `BGI1!`. Se omitido, usa o símbolo principal do provider. |

**Exemplo:**

```
GET /api/market/quote?symbol=BOI&contractCode=BGI1!
```

**Resposta 200:**

```json
{
  "symbol": "BOI",
  "contractCode": "BGI1!",
  "price": 360.00,
  "openPrice": 359.90,
  "highPrice": 360.00,
  "lowPrice": 359.90,
  "variationPercent": 0.0,
  "volume": 11,
  "source": "tradingview",
  "quoteTimestamp": "2026-04-19T14:01:20Z"
}
```

| Código | Descrição |
|--------|-----------|
| `404` | Contrato não encontrado |
| `502` | Provider indisponível (sidecar TV offline, BRAPI 5xx) |

---

### GET /api/market/candles

Retorna candles OHLC para construção de gráfico.

**Query Parameters:**

| Parâmetro | Tipo | Padrão | Descrição |
|-----------|------|--------|-----------|
| `symbol` | string | — | Ex: `BOI` |
| `contractCode` | string | — | Ex: `BGI1!` |
| `timeframe` | string | `1d` | `1m`, `5m`, `15m`, `30m`, `1h`, `4h`, `1d`, `1w`, `1M` |
| `limit` | int | `90` | Máximo 500 |

**Exemplo:**

```
GET /api/market/candles?symbol=BOI&contractCode=BGI1!&timeframe=1d&limit=30
```

**Resposta 200:**

```json
[
  {
    "symbol": "BOI",
    "contractCode": "BGI1!",
    "timeframe": "1d",
    "open": 362.50,
    "high": 362.60,
    "low": 358.50,
    "close": 360.00,
    "volume": 1144,
    "source": "tradingview",
    "candleTime": "2026-04-16T20:05:00Z"
  }
]
```

---

### GET /api/market/quotes/history

Histórico de cotações (último N registros).

**Query Parameters:**

| Parâmetro | Tipo | Padrão | Descrição |
|-----------|------|--------|-----------|
| `symbol` | string | — | Ex: `BOI` |
| `contractCode` | string | — | Ex: `BGI1!` |
| `limit` | int | `100` | Quantidade de registros |

**Resposta 200:** Array de `MarketQuote` (mesma estrutura de `/quote`).

---

### GET /api/market/provider/health

Status do provider ativo. Útil para alarmes CloudWatch.

**Resposta 200:**

```json
{
  "provider": "tradingview",
  "isHealthy": true,
  "checkedAt": "2026-04-19T14:39:01Z"
}
```

---

## Preços de Mercado (legacy — alimenta dashboard)

Os endpoints abaixo continuam servindo o dashboard com dados agregados (preço atual, médias móveis 7d/30d). Internamente, o `MarketIngestionHostedService` grava o **primeiro contrato configurado** em `ActiveContracts` na tabela `PrecosMercado` (1 registro por dia/fonte) para alimentá-los.

### GET /api/precos-mercado/resumo

**Query Parameters:**

| Parâmetro | Tipo | Padrão | Descrição |
|-----------|------|--------|-----------|
| `fonte` | string | `TRADINGVIEW` | `TRADINGVIEW`, `BRAPI`, `CEPEA` (legacy), `MANUAL` |

**Resposta 200:**

```json
{
  "ultimoPreco": {
    "id": "...",
    "data": "2026-04-19",
    "precoArroba": 360.00,
    "variacao": 0.50,
    "variacaoPercentual": 0.14
  },
  "mediaMovel7d": 361.20,
  "mediaMovel30d": 358.45,
  "historico": [
    { "id": "...", "data": "2026-04-19", "precoArroba": 360.00, "variacao": 0.50, "variacaoPercentual": 0.14 }
  ]
}
```

---

### GET /api/precos-mercado/historico

Histórico para gráficos legacy do dashboard.

**Query Parameters:**

| Parâmetro | Tipo | Padrão |
|-----------|------|--------|
| `fonte` | string | `TRADINGVIEW` |
| `dias` | int | `90` |

---

### POST /api/precos-mercado

!!! warning "Permissão"
    Requer role **Admin** ou **Gerente**.

Cria registro manual (ex: anotação de preço regional). Salvo com `Fonte=MANUAL`.

**Request Body:**

```json
{
  "data": "2026-04-19",
  "precoArroba": 358.00,
  "fonte": "MANUAL",
  "indice": "REGIONAL_MT"
}
```

| Código | Descrição |
|--------|-----------|
| `400` | Duplicado para data/fonte |
| `403` | Sem permissão |

---

## Alertas de Preço

### GET /api/alertas-preco

Lista os alertas do usuário autenticado.

**Resposta 200:**

```json
[
  {
    "id": "b8c9d0e1-f2a3-4567-b890-123456789012",
    "tipo": "PrecoAcima",
    "valorAlvo": 365.00,
    "fonte": "TRADINGVIEW",
    "ativo": true,
    "criadoEm": "2026-04-01T10:00:00Z"
  }
]
```

---

### POST /api/alertas-preco

```json
{
  "tipo": "PrecoAbaixo",
  "valorAlvo": 350.00,
  "fonte": "TRADINGVIEW"
}
```

**Resposta 201:** alerta criado.

---

### PUT /api/alertas-preco/{id}

```json
{
  "tipo": "PrecoAbaixo",
  "valorAlvo": 348.00,
  "fonte": "TRADINGVIEW",
  "ativo": false
}
```

---

### DELETE /api/alertas-preco/{id}

Remove o alerta. Resposta 204.

---

## Pipeline de Ingestão

```mermaid
graph LR
    TV[TradingView WebSocket] -->|@mathieuc/tradingview| SIDECAR[tv-scraper Node :3001]
    SIDECAR -->|HTTP /candles /quote| API[.NET API]
    API --> PROVIDER[TradingViewMarketDataProvider]
    PROVIDER --> INGEST[MarketIngestionHostedService cada 5 min]
    INGEST --> DB[(MarketCandles + MarketQuotes + PrecosMercado)]
    DB --> CTRL[MarketController + PrecosMercadoController]
    CTRL --> WEB[CandleChart.tsx no frontend]
```

- Intervalo de ingestão: `MarketData:SyncIntervalSeconds` (default 300s).
- Contratos puxados: `MarketData:ActiveContracts` (default `["BGI1!"]`).
- Falha silenciosa por contrato: se um contrato falhar, os outros continuam.
- Deduplicação: `SaveCandlesAsync` e `SaveQuoteAsync` checam por chave única (`Symbol+ContractCode+Timeframe+CandleTime`).

---

## Quando trocar para Cedro Technologies

Plano de migração futura quando houver receita (~R$ 440/mês):

1. Criar `CedroMarketDataProvider.cs` implementando `IMarketDataProvider`.
2. Registrar em `DependencyInjection.cs` com `MarketData:Provider=cedro`.
3. Setar env var na task ECS, force-new-deployment.
4. Desligar `tv-scraper` (essential=false na task def, basta remover do containerDefinitions).

Nada muda no schema, nas rotas ou no frontend.
