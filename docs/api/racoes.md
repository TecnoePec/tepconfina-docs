# Racoes e Consumo

Endpoints para gerenciamento de racoes e registro de consumo por lote.

!!! info "Autenticacao Obrigatoria"
    Todos os endpoints desta secao requerem header `Authorization: Bearer {token}`.

## Resumo dos Endpoints

| Metodo | Rota | Roles | Descricao |
|--------|------|-------|-----------|
| `GET` | `/api/racoes` | Todos | Listar racoes |
| `GET` | `/api/racoes/{id}` | Todos | Detalhe da racao |
| `POST` | `/api/racoes` | Admin, Gerente | Criar racao |
| `PUT` | `/api/racoes/{id}` | Admin, Gerente | Atualizar racao |
| `DELETE` | `/api/racoes/{id}` | Admin | Remover racao |
| `GET` | `/api/lotes/{loteId}/consumos-racao` | Todos | Listar consumos do lote |
| `GET` | `/api/lotes/{loteId}/consumos-racao/{id}` | Todos | Detalhe do consumo |
| `POST` | `/api/lotes/{loteId}/consumos-racao` | Admin, Gerente | Registrar consumo |
| `PUT` | `/api/lotes/{loteId}/consumos-racao/{id}` | Admin, Gerente | Atualizar consumo |
| `DELETE` | `/api/lotes/{loteId}/consumos-racao/{id}` | Admin | Remover consumo |

---

## Racoes

### GET /api/racoes

Lista racoes cadastradas com paginacao e filtros.

**Query Parameters:**

| Parametro | Tipo | Padrao | Descricao |
|-----------|------|--------|-----------|
| `pageNumber` | int | 1 | Numero da pagina |
| `pageSize` | int | 10 | Itens por pagina |
| `search` | string | - | Busca por nome ou descricao |
| `dataCompraDe` | date | - | Data de compra inicial |
| `dataCompraAte` | date | - | Data de compra final |

**Exemplo de requisicao:**

```
GET /api/racoes?pageNumber=1&pageSize=10&search=milho
```

**Resposta 200:**

```json
{
  "items": [
    {
      "id": "d0e1f2a3-b4c5-6789-d012-345678901234",
      "nome": "Racao Engorda Premium",
      "descricao": "Racao com alto teor energetico para fase de terminacao",
      "custoKg": 2.85,
      "dataCompra": "2026-01-10",
      "quantidadeKg": 50000,
      "fornecedor": "Nutricao Animal Ltda"
    }
  ],
  "pageNumber": 1,
  "pageSize": 10,
  "totalCount": 8,
  "totalPages": 1
}
```

---

### GET /api/racoes/{id}

Retorna os detalhes de uma racao especifica.

**Resposta 200:**

```json
{
  "id": "d0e1f2a3-b4c5-6789-d012-345678901234",
  "nome": "Racao Engorda Premium",
  "descricao": "Racao com alto teor energetico para fase de terminacao",
  "custoKg": 2.85,
  "dataCompra": "2026-01-10",
  "quantidadeKg": 50000,
  "fornecedor": "Nutricao Animal Ltda",
  "composicao": "Milho, farelo de soja, ureia, minerais"
}
```

| Codigo | Descricao |
|--------|-----------|
| `404` | Racao nao encontrada |

---

### POST /api/racoes

!!! warning "Permissao"
    Requer role **Admin** ou **Gerente**.

**Request Body:**

```json
{
  "nome": "Racao Engorda Premium",
  "descricao": "Racao com alto teor energetico para fase de terminacao",
  "custoKg": 2.85,
  "dataCompra": "2026-01-10",
  "quantidadeKg": 50000,
  "fornecedor": "Nutricao Animal Ltda",
  "composicao": "Milho, farelo de soja, ureia, minerais"
}
```

**Resposta 201:** Racao criada.

| Codigo | Descricao |
|--------|-----------|
| `400` | Dados invalidos |
| `403` | Sem permissao |

---

### PUT /api/racoes/{id}

!!! warning "Permissao"
    Requer role **Admin** ou **Gerente**.

**Request Body:** Mesmo formato do POST.

**Resposta 200:** Racao atualizada.

| Codigo | Descricao |
|--------|-----------|
| `400` | Dados invalidos |
| `403` | Sem permissao |
| `404` | Racao nao encontrada |

---

### DELETE /api/racoes/{id}

!!! danger "Permissao Restrita"
    Requer role **Admin**.

**Resposta 204:** Removida com sucesso.

| Codigo | Descricao |
|--------|-----------|
| `403` | Sem permissao |
| `404` | Racao nao encontrada |

---

## Consumo de Racao por Lote

### GET /api/lotes/{loteId}/consumos-racao

Lista todos os registros de consumo de racao de um lote.

**Resposta 200:**

```json
[
  {
    "id": "e1f2a3b4-c5d6-7890-e123-456789012345",
    "data": "2026-02-15",
    "racaoId": "d0e1f2a3-b4c5-6789-d012-345678901234",
    "racaoNome": "Racao Engorda Premium",
    "quantidadeKg": 1200.0,
    "custoTotal": 3420.00,
    "loteId": "3fa85f64-5717-4562-b3fc-2c963f66afa6"
  }
]
```

| Codigo | Descricao |
|--------|-----------|
| `404` | Lote nao encontrado |

---

### GET /api/lotes/{loteId}/consumos-racao/{id}

Retorna o detalhe de um registro de consumo.

**Resposta 200:**

```json
{
  "id": "e1f2a3b4-c5d6-7890-e123-456789012345",
  "data": "2026-02-15",
  "racaoId": "d0e1f2a3-b4c5-6789-d012-345678901234",
  "racaoNome": "Racao Engorda Premium",
  "quantidadeKg": 1200.0,
  "custoTotal": 3420.00,
  "observacao": "Consumo diario normal"
}
```

| Codigo | Descricao |
|--------|-----------|
| `404` | Lote ou consumo nao encontrado |

---

### POST /api/lotes/{loteId}/consumos-racao

!!! warning "Permissao"
    Requer role **Admin** ou **Gerente**.

Registra um novo consumo de racao para o lote.

**Request Body:**

```json
{
  "data": "2026-03-01",
  "racaoId": "d0e1f2a3-b4c5-6789-d012-345678901234",
  "quantidadeKg": 1150.0,
  "observacao": "Consumo diario ajustado"
}
```

**Resposta 201:** Consumo registrado.

| Codigo | Descricao |
|--------|-----------|
| `400` | Dados invalidos ou racao nao encontrada |
| `403` | Sem permissao |
| `404` | Lote nao encontrado |

---

### PUT /api/lotes/{loteId}/consumos-racao/{id}

!!! warning "Permissao"
    Requer role **Admin** ou **Gerente**.

**Request Body:** Mesmo formato do POST.

**Resposta 200:** Consumo atualizado.

| Codigo | Descricao |
|--------|-----------|
| `400` | Dados invalidos |
| `403` | Sem permissao |
| `404` | Lote ou consumo nao encontrado |

---

### DELETE /api/lotes/{loteId}/consumos-racao/{id}

!!! danger "Permissao Restrita"
    Requer role **Admin**.

**Resposta 204:** Removido com sucesso.

| Codigo | Descricao |
|--------|-----------|
| `403` | Sem permissao |
| `404` | Lote ou consumo nao encontrado |
