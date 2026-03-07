# Notificacoes, Dashboard e Health

Endpoints para gerenciamento de notificacoes do usuario, KPIs do dashboard e verificacao de saude da API.

## Resumo dos Endpoints

| Metodo | Rota | Autenticacao | Descricao |
|--------|------|:------------:|-----------|
| `GET` | `/api/notificacoes` | Sim | Listar notificacoes |
| `GET` | `/api/notificacoes/nao-lidas/count` | Sim | Contagem de nao lidas |
| `PUT` | `/api/notificacoes/{id}/lida` | Sim | Marcar como lida |
| `PUT` | `/api/notificacoes/todas/lidas` | Sim | Marcar todas como lidas |
| `GET` | `/api/dashboard/summary` | Sim | KPIs gerais do dashboard |
| `GET` | `/api/health` | Nao | Health check |

---

## Notificacoes

!!! info "Autenticacao Obrigatoria"
    Todos os endpoints de notificacoes requerem header `Authorization: Bearer {token}`.

### GET /api/notificacoes

Lista as notificacoes do usuario autenticado.

**Resposta 200:**

```json
[
  {
    "id": "c5d6e7f8-a9b0-1234-c567-890123456789",
    "titulo": "Alerta de Preco",
    "mensagem": "O preco da arroba atingiu R$ 320,00 (acima do alerta configurado de R$ 315,00)",
    "tipo": "AlertaPreco",
    "lida": false,
    "criadaEm": "2026-03-06T14:30:00Z",
    "dados": {
      "precoAtual": 320.00,
      "alertaId": "b8c9d0e1-f2a3-4567-b890-123456789012"
    }
  },
  {
    "id": "d6e7f8a9-b0c1-2345-d678-901234567890",
    "titulo": "Pesagem Pendente",
    "mensagem": "O lote LT-2026-001 esta ha 35 dias sem pesagem",
    "tipo": "PesagemPendente",
    "lida": true,
    "criadaEm": "2026-03-05T08:00:00Z",
    "dados": {
      "loteId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
      "loteCodigo": "LT-2026-001"
    }
  }
]
```

---

### GET /api/notificacoes/nao-lidas/count

Retorna a contagem de notificacoes nao lidas do usuario.

**Resposta 200:**

```json
{
  "count": 5
}
```

---

### PUT /api/notificacoes/{id}/lida

Marca uma notificacao especifica como lida.

**Path Parameters:**

| Parametro | Tipo | Descricao |
|-----------|------|-----------|
| `id` | guid | ID da notificacao |

**Resposta 200:**

```json
{
  "id": "c5d6e7f8-a9b0-1234-c567-890123456789",
  "lida": true
}
```

| Codigo | Descricao |
|--------|-----------|
| `404` | Notificacao nao encontrada |

---

### PUT /api/notificacoes/todas/lidas

Marca todas as notificacoes do usuario como lidas.

**Resposta 200:**

```json
{
  "atualizadas": 5,
  "message": "Todas as notificacoes foram marcadas como lidas"
}
```

---

## Dashboard

!!! info "Autenticacao Obrigatoria"
    Requer header `Authorization: Bearer {token}`.

### GET /api/dashboard/summary

Retorna os KPIs gerais do confinamento para exibicao no dashboard principal.

**Resposta 200:**

```json
{
  "totalLotesAtivos": 5,
  "totalLotesFechados": 12,
  "totalAnimaisAtivos": 580,
  "totalAnimaisVendidos": 1200,
  "totalAnimaisMortos": 15,
  "pesoMedioAtual": 445.2,
  "gmdMedioGeral": 1.42,
  "custoTotalRacao": 650000.00,
  "custoTotalMedicamentos": 85000.00,
  "receitaTotalVendas": 1250000.00,
  "precoArrobaAtual": 310.50,
  "lotesProximosFechamento": [
    {
      "loteId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
      "codigo": "LT-2026-001",
      "diasConfinamento": 85,
      "pesoMedioAtual": 510.0
    }
  ],
  "alertasNaoLidos": 3
}
```

---

## Health Check

### GET /api/health

Endpoint publico para verificacao de saude da API. Nao requer autenticacao.

**Resposta 200:**

```json
{
  "status": "Healthy",
  "timestamp": "2026-03-07T10:00:00Z"
}
```

!!! note "Uso"
    Este endpoint e utilizado por balanceadores de carga e ferramentas de monitoramento para verificar se a API esta operacional. Retorna `200` quando o servico esta saudavel e `503` em caso de falha.

| Codigo | Descricao |
|--------|-----------|
| `200` | API operacional |
| `503` | Servico indisponivel |
