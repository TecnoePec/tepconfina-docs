# Relatórios

Endpoints para geração de relatórios analíticos consolidados.

!!! info "Autenticacao Obrigatoria"
    Todos os endpoints requerem header `Authorization: Bearer {token}`.

## Resumo dos Endpoints

| Metodo | Rota | Roles | Descricao |
|--------|------|-------|-----------|
| `GET` | `/api/relatorios/desempenho-raca` | Todos | Comparativo de desempenho por raça |
| `GET` | `/api/relatorios/financeiro` | Todos | Resumo financeiro por período |

---

## GET /api/relatorios/desempenho-raca

Agrupa todos os lotes por raça e calcula indicadores de desempenho comparativos.

**Resposta `200`:**

```json
[
  {
    "raca": "Nelore",
    "totalLotes": 8,
    "totalAnimais": 1200,
    "lotesAtivos": 5,
    "gmdMedio": 1.320,
    "gmdMin": 0.980,
    "gmdMax": 1.650,
    "taxaMortalidade": 1.25,
    "custoTotalMedio": 48500.00
  }
]
```

**Campos:**

| Campo | Descrição |
|-------|-----------|
| `gmdMedio` | GMD médio entre todos os lotes da raça (kg/dia) |
| `gmdMin` / `gmdMax` | GMD mínimo e máximo observados |
| `taxaMortalidade` | Percentual de animais mortos (`mortos / total × 100`) |
| `custoTotalMedio` | Custo médio por lote (ração + medicamentos + outros) |

!!! note "Lotes sem raça"
    Lotes com o campo `Raca` em branco são excluídos do relatório.

---

## GET /api/relatorios/financeiro

Retorna resumo financeiro dos lotes por período de entrada.

**Query Parameters:**

| Parametro | Tipo | Padrão | Descrição |
|-----------|------|--------|-----------|
| `de` | datetime | 3 meses atrás | Data de início (DataEntrada do lote) |
| `ate` | datetime | Hoje | Data de fim |

**Resposta `200`:**

```json
{
  "periodo": { "de": "2025-03-01", "ate": "2025-06-01" },
  "lotes": [
    {
      "id": "uuid",
      "nome": "Lote A",
      "status": "Ativo",
      "dataEntrada": "2025-03-15",
      "dataSaida": null,
      "quantidadeAnimais": 150,
      "custoRacao": 18000.00,
      "custoMed": 2400.00,
      "custoOutros": 1200.00,
      "custoAnimais": 90000.00,
      "custoTotal": 111600.00
    }
  ],
  "totais": {
    "custoTotal": 111600.00,
    "custoRacao": 18000.00,
    "custoMed": 2400.00,
    "custoOutros": 1200.00,
    "totalAnimais": 150
  }
}
```
