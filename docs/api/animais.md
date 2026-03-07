# Animais

Endpoints para gerenciamento individual de animais no confinamento.

!!! info "Autenticacao Obrigatoria"
    Todos os endpoints desta secao requerem header `Authorization: Bearer {token}`.

## Resumo dos Endpoints

| Metodo | Rota | Roles | Descricao |
|--------|------|-------|-----------|
| `GET` | `/api/animais` | Todos | Listar animais com filtros |
| `GET` | `/api/animais/{id}` | Todos | Detalhe do animal |
| `PUT` | `/api/animais/{id}` | Admin, Gerente | Atualizar animal |
| `DELETE` | `/api/animais/{id}` | Admin | Remover animal |
| `PUT` | `/api/animais/{id}/mortalidade` | Admin, Gerente | Registrar mortalidade |
| `PUT` | `/api/animais/{id}/venda` | Admin, Gerente | Registrar venda |
| `GET` | `/api/animais/{animalId}/pesagens` | Todos | Pesagens do animal |
| `POST` | `/api/animais/{animalId}/pesagens` | Admin, Gerente | Criar pesagem individual |
| `DELETE` | `/api/animais/{animalId}/pesagens/{pesagemId}` | Admin | Remover pesagem |
| `GET` | `/api/lotes/{loteId}/animais` | Todos | Listar animais do lote |
| `POST` | `/api/lotes/{loteId}/animais` | Admin, Gerente | Criar animal no lote |
| `POST` | `/api/lotes/{loteId}/animais/lote` | Admin, Gerente | Criacao em lote (batch) |

---

## GET /api/animais

Lista animais com paginacao e filtros.

**Query Parameters:**

| Parametro | Tipo | Padrao | Descricao |
|-----------|------|--------|-----------|
| `pageNumber` | int | 1 | Numero da pagina |
| `pageSize` | int | 10 | Itens por pagina |
| `status` | string | - | Filtro: Ativo, Vendido, Morto |
| `loteId` | guid | - | Filtro por lote |

**Resposta 200:**

```json
{
  "items": [
    {
      "id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
      "identificacao": "AN-001",
      "brinco": "BR-12345",
      "sexo": "Macho",
      "raca": "Nelore",
      "pesoEntrada": 380.0,
      "pesoAtual": 445.0,
      "status": "Ativo",
      "loteId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
      "loteCodigo": "LT-2026-001"
    }
  ],
  "pageNumber": 1,
  "pageSize": 10,
  "totalCount": 120,
  "totalPages": 12
}
```

---

## GET /api/animais/{id}

Retorna os detalhes completos de um animal.

**Resposta 200:**

```json
{
  "id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "identificacao": "AN-001",
  "brinco": "BR-12345",
  "sexo": "Macho",
  "raca": "Nelore",
  "pesoEntrada": 380.0,
  "pesoAtual": 445.0,
  "gmd": 1.44,
  "status": "Ativo",
  "dataEntrada": "2026-01-15",
  "loteId": "3fa85f64-5717-4562-b3fc-2c963f66afa6"
}
```

| Codigo | Descricao |
|--------|-----------|
| `404` | Animal nao encontrado |

---

## PUT /api/animais/{id}

!!! warning "Permissao"
    Requer role **Admin** ou **Gerente**.

**Request Body:**

```json
{
  "identificacao": "AN-001",
  "brinco": "BR-12345",
  "sexo": "Macho",
  "raca": "Nelore",
  "pesoEntrada": 385.0
}
```

**Resposta 200:** Animal atualizado.

| Codigo | Descricao |
|--------|-----------|
| `400` | Dados invalidos |
| `403` | Sem permissao |
| `404` | Animal nao encontrado |

---

## DELETE /api/animais/{id}

!!! danger "Permissao Restrita"
    Requer role **Admin**.

**Resposta 204:** Removido com sucesso.

| Codigo | Descricao |
|--------|-----------|
| `403` | Sem permissao |
| `404` | Animal nao encontrado |

---

## PUT /api/animais/{id}/mortalidade

!!! warning "Permissao"
    Requer role **Admin** ou **Gerente**.

Registra a mortalidade de um animal, alterando seu status para `Morto`.

**Request Body:**

```json
{
  "dataMorte": "2026-02-20",
  "causa": "Doenca respiratoria"
}
```

**Resposta 200:**

```json
{
  "id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "status": "Morto",
  "dataMorte": "2026-02-20",
  "causa": "Doenca respiratoria"
}
```

---

## PUT /api/animais/{id}/venda

!!! warning "Permissao"
    Requer role **Admin** ou **Gerente**.

Registra a venda de um animal.

**Request Body:**

```json
{
  "dataVenda": "2026-03-01",
  "pesoVenda": 520.0,
  "valorArroba": 310.00
}
```

**Resposta 200:**

```json
{
  "id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "status": "Vendido",
  "pesoVenda": 520.0,
  "valorTotal": 10746.67
}
```

---

## Pesagens Individuais

### GET /api/animais/{animalId}/pesagens

Lista todas as pesagens de um animal especifico.

**Resposta 200:**

```json
[
  {
    "id": "c3d4e5f6-a7b8-9012-cdef-234567890123",
    "data": "2026-02-01",
    "peso": 410.5,
    "observacao": "Pesagem mensal"
  }
]
```

### POST /api/animais/{animalId}/pesagens

!!! warning "Permissao"
    Requer role **Admin** ou **Gerente**.

```json
{
  "data": "2026-03-01",
  "peso": 445.0,
  "observacao": "Pesagem mensal"
}
```

**Resposta 201:** Pesagem criada.

### DELETE /api/animais/{animalId}/pesagens/{pesagemId}

!!! danger "Permissao Restrita"
    Requer role **Admin**.

**Resposta 204:** Pesagem removida.

---

## Animais por Lote

### GET /api/lotes/{loteId}/animais

Lista todos os animais vinculados a um lote.

**Resposta 200:** Mesmo formato da listagem de animais.

### POST /api/lotes/{loteId}/animais

!!! warning "Permissao"
    Requer role **Admin** ou **Gerente**.

Cria um animal individual vinculado ao lote.

```json
{
  "identificacao": "AN-121",
  "brinco": "BR-99001",
  "sexo": "Macho",
  "raca": "Nelore",
  "pesoEntrada": 375.0,
  "dataEntrada": "2026-01-15"
}
```

**Resposta 201:** Animal criado.

### POST /api/lotes/{loteId}/animais/lote

!!! warning "Permissao"
    Requer role **Admin** ou **Gerente**.

Criacao em lote (batch) de multiplos animais.

```json
{
  "animais": [
    {
      "identificacao": "AN-122",
      "brinco": "BR-99002",
      "sexo": "Femea",
      "raca": "Nelore",
      "pesoEntrada": 365.0
    },
    {
      "identificacao": "AN-123",
      "brinco": "BR-99003",
      "sexo": "Macho",
      "raca": "Nelore",
      "pesoEntrada": 390.0
    }
  ],
  "dataEntrada": "2026-01-15"
}
```

**Resposta 201:**

```json
{
  "criados": 2,
  "erros": []
}
```

| Codigo | Descricao |
|--------|-----------|
| `400` | Dados invalidos ou identificacao duplicada |
| `403` | Sem permissao |
| `404` | Lote nao encontrado |
