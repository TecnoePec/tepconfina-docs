# Pesagens por Lote

Endpoints para gerenciamento de pesagens coletivas vinculadas a um lote de confinamento.

!!! info "Autenticacao Obrigatoria"
    Todos os endpoints desta secao requerem header `Authorization: Bearer {token}`.

## Resumo dos Endpoints

| Metodo | Rota | Roles | Descricao |
|--------|------|-------|-----------|
| `GET` | `/api/lotes/{loteId}/pesagens` | Todos | Listar pesagens do lote |
| `GET` | `/api/lotes/{loteId}/pesagens/{id}` | Todos | Detalhe da pesagem |
| `POST` | `/api/lotes/{loteId}/pesagens` | Admin, Gerente | Criar pesagem |
| `PUT` | `/api/lotes/{loteId}/pesagens/{id}` | Admin, Gerente | Atualizar pesagem |
| `DELETE` | `/api/lotes/{loteId}/pesagens/{id}` | Admin | Remover pesagem |
| `GET` | `/api/pesagens` | Todos | Listar todas as pesagens (global) |

---

## GET /api/lotes/{loteId}/pesagens

Lista todas as pesagens registradas para um lote.

**Path Parameters:**

| Parametro | Tipo | Descricao |
|-----------|------|-----------|
| `loteId` | guid | ID do lote |

**Resposta 200:**

```json
[
  {
    "id": "d4e5f6a7-b8c9-0123-def4-567890123456",
    "data": "2026-02-01",
    "pesoMedio": 410.5,
    "quantidadeAnimais": 120,
    "observacao": "Pesagem mensal - fevereiro",
    "loteId": "3fa85f64-5717-4562-b3fc-2c963f66afa6"
  },
  {
    "id": "e5f6a7b8-c9d0-1234-ef56-789012345678",
    "data": "2026-01-15",
    "pesoMedio": 380.0,
    "quantidadeAnimais": 120,
    "observacao": "Pesagem de entrada",
    "loteId": "3fa85f64-5717-4562-b3fc-2c963f66afa6"
  }
]
```

| Codigo | Descricao |
|--------|-----------|
| `404` | Lote nao encontrado |

---

## GET /api/lotes/{loteId}/pesagens/{id}

Retorna os detalhes de uma pesagem especifica.

**Path Parameters:**

| Parametro | Tipo | Descricao |
|-----------|------|-----------|
| `loteId` | guid | ID do lote |
| `id` | guid | ID da pesagem |

**Resposta 200:**

```json
{
  "id": "d4e5f6a7-b8c9-0123-def4-567890123456",
  "data": "2026-02-01",
  "pesoMedio": 410.5,
  "quantidadeAnimais": 120,
  "observacao": "Pesagem mensal - fevereiro",
  "loteId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "loteCodigo": "LT-2026-001"
}
```

| Codigo | Descricao |
|--------|-----------|
| `404` | Lote ou pesagem nao encontrado |

---

## POST /api/lotes/{loteId}/pesagens

!!! warning "Permissao"
    Requer role **Admin** ou **Gerente**.

Cria uma nova pesagem coletiva para o lote.

**Request Body:**

```json
{
  "data": "2026-03-01",
  "pesoMedio": 445.0,
  "quantidadeAnimais": 118,
  "observacao": "Pesagem mensal - marco"
}
```

**Resposta 201:**

```json
{
  "id": "f6a7b8c9-d0e1-2345-f678-901234567890",
  "data": "2026-03-01",
  "pesoMedio": 445.0,
  "quantidadeAnimais": 118,
  "loteId": "3fa85f64-5717-4562-b3fc-2c963f66afa6"
}
```

| Codigo | Descricao |
|--------|-----------|
| `400` | Dados invalidos |
| `403` | Sem permissao |
| `404` | Lote nao encontrado |

---

## PUT /api/lotes/{loteId}/pesagens/{id}

!!! warning "Permissao"
    Requer role **Admin** ou **Gerente**.

Atualiza uma pesagem existente.

**Request Body:**

```json
{
  "data": "2026-03-01",
  "pesoMedio": 447.5,
  "quantidadeAnimais": 118,
  "observacao": "Pesagem mensal - marco (corrigida)"
}
```

**Resposta 200:** Pesagem atualizada.

| Codigo | Descricao |
|--------|-----------|
| `400` | Dados invalidos |
| `403` | Sem permissao |
| `404` | Lote ou pesagem nao encontrado |

---

## DELETE /api/lotes/{loteId}/pesagens/{id}

!!! danger "Permissao Restrita"
    Requer role **Admin**.

Remove uma pesagem do lote.

**Resposta 204:** Removida com sucesso.

| Codigo | Descricao |
|--------|-----------|
| `403` | Sem permissao |
| `404` | Lote ou pesagem nao encontrado |

---

## GET /api/pesagens (Global)

Endpoint global que lista pesagens de todos os lotes com paginacao.

**Query Parameters:**

| Parametro | Tipo | Padrao | Descricao |
|-----------|------|--------|-----------|
| `pageNumber` | int | 1 | Numero da pagina |
| `pageSize` | int | 10 | Itens por pagina |

**Resposta 200:**

```json
{
  "items": [
    {
      "id": "d4e5f6a7-b8c9-0123-def4-567890123456",
      "data": "2026-03-01",
      "pesoMedio": 445.0,
      "quantidadeAnimais": 118,
      "loteId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
      "loteCodigo": "LT-2026-001"
    }
  ],
  "pageNumber": 1,
  "pageSize": 10,
  "totalCount": 25,
  "totalPages": 3
}
```
