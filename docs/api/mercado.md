# Precos de Mercado e Alertas

Endpoints para consulta de precos de mercado do boi gordo e gerenciamento de alertas de preco.

!!! info "Autenticacao Obrigatoria"
    Todos os endpoints desta secao requerem header `Authorization: Bearer {token}`.

## Resumo dos Endpoints

| Metodo | Rota | Roles | Descricao |
|--------|------|-------|-----------|
| `GET` | `/api/precos-mercado/resumo` | Todos | Resumo de precos |
| `GET` | `/api/precos-mercado/historico` | Todos | Historico de precos |
| `POST` | `/api/precos-mercado` | Admin, Gerente | Criar registro de preco |
| `GET` | `/api/alertas-preco` | Todos | Listar alertas |
| `POST` | `/api/alertas-preco` | Todos | Criar alerta |
| `PUT` | `/api/alertas-preco/{id}` | Todos | Atualizar alerta |
| `DELETE` | `/api/alertas-preco/{id}` | Todos | Remover alerta |

---

## Precos de Mercado

### GET /api/precos-mercado/resumo

Retorna o resumo dos precos de mercado mais recentes.

**Query Parameters:**

| Parametro | Tipo | Padrao | Descricao |
|-----------|------|--------|-----------|
| `fonte` | string | `CEPEA` | Fonte dos precos (ex: CEPEA) |

**Exemplo de requisicao:**

```
GET /api/precos-mercado/resumo?fonte=CEPEA
```

**Resposta 200:**

```json
{
  "fonte": "CEPEA",
  "dataUltimaAtualizacao": "2026-03-06",
  "precoArroba": 310.50,
  "variacaoDiaria": 1.25,
  "variacaoSemanal": -0.80,
  "variacaoMensal": 3.45,
  "precoMinimo30d": 298.00,
  "precoMaximo30d": 315.00
}
```

---

### GET /api/precos-mercado/historico

Retorna o historico de precos para construcao de graficos e analises.

**Query Parameters:**

| Parametro | Tipo | Padrao | Descricao |
|-----------|------|--------|-----------|
| `fonte` | string | `CEPEA` | Fonte dos precos |
| `dias` | int | `90` | Quantidade de dias do historico |

**Exemplo de requisicao:**

```
GET /api/precos-mercado/historico?fonte=CEPEA&dias=90
```

**Resposta 200:**

```json
{
  "fonte": "CEPEA",
  "periodo": {
    "inicio": "2025-12-07",
    "fim": "2026-03-07"
  },
  "precos": [
    {
      "data": "2026-03-06",
      "valor": 310.50
    },
    {
      "data": "2026-03-05",
      "valor": 309.25
    }
  ]
}
```

---

### POST /api/precos-mercado

!!! warning "Permissao"
    Requer role **Admin** ou **Gerente**.

Cria um registro manual de preco de mercado.

**Request Body:**

```json
{
  "data": "2026-03-07",
  "valor": 311.00,
  "fonte": "CEPEA",
  "tipo": "BoiGordo"
}
```

**Resposta 201:**

```json
{
  "id": "a7b8c9d0-e1f2-3456-a789-012345678901",
  "data": "2026-03-07",
  "valor": 311.00,
  "fonte": "CEPEA",
  "tipo": "BoiGordo"
}
```

| Codigo | Descricao |
|--------|-----------|
| `400` | Dados invalidos ou registro duplicado para data/fonte |
| `403` | Sem permissao |

---

## Alertas de Preco

### GET /api/alertas-preco

Lista os alertas de preco configurados pelo usuario autenticado.

**Resposta 200:**

```json
[
  {
    "id": "b8c9d0e1-f2a3-4567-b890-123456789012",
    "tipo": "PrecoAcima",
    "valorAlvo": 320.00,
    "fonte": "CEPEA",
    "ativo": true,
    "criadoEm": "2026-02-01T10:00:00Z"
  }
]
```

---

### POST /api/alertas-preco

Cria um novo alerta de preco.

**Request Body:**

```json
{
  "tipo": "PrecoAbaixo",
  "valorAlvo": 290.00,
  "fonte": "CEPEA"
}
```

**Resposta 201:**

```json
{
  "id": "c9d0e1f2-a3b4-5678-c901-234567890123",
  "tipo": "PrecoAbaixo",
  "valorAlvo": 290.00,
  "fonte": "CEPEA",
  "ativo": true,
  "criadoEm": "2026-03-07T14:30:00Z"
}
```

| Codigo | Descricao |
|--------|-----------|
| `400` | Dados invalidos |

---

### PUT /api/alertas-preco/{id}

Atualiza um alerta existente.

**Request Body:**

```json
{
  "tipo": "PrecoAbaixo",
  "valorAlvo": 285.00,
  "fonte": "CEPEA",
  "ativo": false
}
```

**Resposta 200:** Alerta atualizado.

| Codigo | Descricao |
|--------|-----------|
| `400` | Dados invalidos |
| `404` | Alerta nao encontrado |

---

### DELETE /api/alertas-preco/{id}

Remove um alerta de preco.

**Resposta 204:** Removido com sucesso.

| Codigo | Descricao |
|--------|-----------|
| `404` | Alerta nao encontrado |
