# Financeiro

Endpoints para consulta de dados financeiros, rentabilidade e resultados do confinamento.

!!! info "Autenticacao Obrigatoria"
    Todos os endpoints desta secao requerem header `Authorization: Bearer {token}`.

## Resumo dos Endpoints

| Metodo | Rota | Roles | Descricao |
|--------|------|-------|-----------|
| `GET` | `/api/financeiro/summary` | Todos | Resumo financeiro geral |
| `GET` | `/api/financeiro/rentabilidade` | Todos | Rentabilidade dos lotes ativos |
| `GET` | `/api/financeiro/rentabilidade/{loteId}` | Todos | Rentabilidade por lote |
| `GET` | `/api/financeiro/categorias-gasto` | Todos | Categorias de gasto |
| `GET` | `/api/financeiro/relatorio-vendas` | Todos | Relatorio de vendas |
| `GET` | `/api/financeiro/resultado-animais/{loteId}` | Todos | Resultado por animal |

---

## GET /api/financeiro/summary

Retorna o resumo financeiro geral do confinamento, consolidando receitas, custos e margens.

**Resposta 200:**

```json
{
  "receitaTotal": 1250000.00,
  "custoTotal": 980000.00,
  "lucroLiquido": 270000.00,
  "margemLiquida": 21.6,
  "custoRacao": 650000.00,
  "custoMedicamentos": 85000.00,
  "custoCompraAnimais": 245000.00,
  "receitaVendas": 1250000.00,
  "totalLotesAtivos": 5,
  "totalLotesFechados": 12
}
```

---

## GET /api/financeiro/rentabilidade

Retorna a rentabilidade de todos os lotes ativos.

**Resposta 200:**

```json
[
  {
    "loteId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "loteCodigo": "LT-2026-001",
    "custoTotal": 185000.00,
    "custoRacao": 120000.00,
    "custoMedicamentos": 15000.00,
    "custoCompra": 50000.00,
    "valorEstimadoVenda": 240000.00,
    "margemEstimada": 29.7,
    "custoPorArroba": 245.50,
    "custoPorAnimal": 1541.67,
    "diasConfinamento": 45,
    "quantidadeAnimais": 120
  }
]
```

---

## GET /api/financeiro/rentabilidade/{loteId}

Retorna a rentabilidade detalhada de um lote especifico.

**Path Parameters:**

| Parametro | Tipo | Descricao |
|-----------|------|-----------|
| `loteId` | guid | ID do lote |

**Resposta 200:**

```json
{
  "loteId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "loteCodigo": "LT-2026-001",
  "custoTotal": 185000.00,
  "custoRacao": 120000.00,
  "custoMedicamentos": 15000.00,
  "custoCompra": 50000.00,
  "valorEstimadoVenda": 240000.00,
  "margemEstimada": 29.7,
  "custoPorArroba": 245.50,
  "custoPorAnimal": 1541.67,
  "custoPorKgGanho": 8.75,
  "diasConfinamento": 45,
  "gmdMedio": 1.45,
  "quantidadeAnimais": 120
}
```

| Codigo | Descricao |
|--------|-----------|
| `404` | Lote nao encontrado |

---

## GET /api/financeiro/categorias-gasto

Retorna o detalhamento dos gastos agrupados por categoria.

**Resposta 200:**

```json
[
  {
    "categoria": "Racao",
    "valor": 650000.00,
    "percentual": 66.3
  },
  {
    "categoria": "Medicamentos",
    "valor": 85000.00,
    "percentual": 8.7
  },
  {
    "categoria": "Compra de Animais",
    "valor": 245000.00,
    "percentual": 25.0
  }
]
```

---

## GET /api/financeiro/relatorio-vendas

Retorna o relatorio consolidado de vendas realizadas.

**Resposta 200:**

```json
{
  "totalVendas": 350,
  "receitaTotal": 1250000.00,
  "pesoTotalVendido": 182000.0,
  "arrobasTotais": 12133.33,
  "valorMedioArroba": 103.00,
  "vendas": [
    {
      "animalId": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
      "identificacao": "AN-001",
      "loteCodigo": "LT-2025-010",
      "dataVenda": "2026-02-15",
      "pesoVenda": 520.0,
      "valorArroba": 310.00,
      "valorTotal": 10746.67
    }
  ]
}
```

---

## GET /api/financeiro/resultado-animais/{loteId}

Retorna o resultado financeiro individual de cada animal de um lote.

**Path Parameters:**

| Parametro | Tipo | Descricao |
|-----------|------|-----------|
| `loteId` | guid | ID do lote |

**Resposta 200:**

```json
[
  {
    "animalId": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "identificacao": "AN-001",
    "pesoEntrada": 380.0,
    "pesoAtual": 520.0,
    "ganhoTotal": 140.0,
    "gmd": 1.56,
    "custoRacao": 1050.00,
    "custoMedicamentos": 125.00,
    "custoTotal": 1175.00,
    "valorEstimadoVenda": 1793.33,
    "resultadoEstimado": 618.33,
    "status": "Ativo"
  }
]
```

| Codigo | Descricao |
|--------|-----------|
| `404` | Lote nao encontrado |
