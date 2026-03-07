# Lotes

Endpoints para gerenciamento de lotes de confinamento bovino.

!!! info "Autenticacao Obrigatoria"
    Todos os endpoints desta secao requerem header `Authorization: Bearer {token}`.

## Resumo dos Endpoints

| Metodo | Rota | Roles | Descricao |
|--------|------|-------|-----------|
| `GET` | `/api/lotes` | Todos | Listar lotes com paginacao e filtros |
| `GET` | `/api/lotes/filtros` | Todos | Valores distintos para filtros |
| `GET` | `/api/lotes/{id}` | Todos | Detalhe do lote |
| `POST` | `/api/lotes` | Admin, Gerente | Criar lote |
| `PUT` | `/api/lotes/{id}` | Admin, Gerente | Atualizar lote |
| `DELETE` | `/api/lotes/{id}` | Admin | Remover lote (soft delete) |
| `POST` | `/api/lotes/{id}/fechar` | Admin, Gerente | Fechar lote |

---

## GET /api/lotes

Lista lotes com paginacao e filtros avancados.

**Query Parameters:**

| Parametro | Tipo | Padrao | Descricao |
|-----------|------|--------|-----------|
| `pageNumber` | int | 1 | Numero da pagina |
| `pageSize` | int | 10 | Itens por pagina |
| `search` | string | - | Busca por codigo ou descricao |
| `sortBy` | string | - | Campo para ordenacao |
| `sortDirection` | string | `asc` | Direcao: `asc` ou `desc` |
| `status` | string | - | Filtro por status (Aberto, Fechado) |
| `produtorId` | guid | - | Filtro por produtor |
| `cidade` | string | - | Filtro por cidade |
| `raca` | string | - | Filtro por raca |
| `dataEntradaDe` | date | - | Data entrada inicial |
| `dataEntradaAte` | date | - | Data entrada final |

**Exemplo de requisicao:**

```
GET /api/lotes?pageNumber=1&pageSize=10&status=Aberto&raca=Nelore
```

**Resposta 200:**

```json
{
  "items": [
    {
      "id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
      "codigo": "LT-2026-001",
      "descricao": "Lote Nelore Janeiro",
      "status": "Aberto",
      "quantidadeAnimais": 120,
      "pesoMedioEntrada": 380.5,
      "dataEntrada": "2026-01-15",
      "raca": "Nelore",
      "cidade": "Uberlandia"
    }
  ],
  "pageNumber": 1,
  "pageSize": 10,
  "totalCount": 1,
  "totalPages": 1
}
```

---

## GET /api/lotes/filtros

Retorna valores distintos disponiveis para uso nos filtros de listagem.

**Resposta 200:**

```json
{
  "cidades": ["Uberlandia", "Uberaba", "Araguari"],
  "racas": ["Nelore", "Angus", "Brangus"],
  "status": ["Aberto", "Fechado"]
}
```

---

## GET /api/lotes/{id}

Retorna os detalhes completos de um lote, incluindo KPIs calculados.

**Resposta 200:**

```json
{
  "id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "codigo": "LT-2026-001",
  "descricao": "Lote Nelore Janeiro",
  "status": "Aberto",
  "quantidadeAnimais": 120,
  "pesoMedioEntrada": 380.5,
  "pesoMedioAtual": 445.2,
  "gmdMedio": 1.45,
  "dataEntrada": "2026-01-15",
  "raca": "Nelore",
  "cidade": "Uberlandia",
  "custoTotalRacao": 85000.00,
  "custoTotalMedicamentos": 12000.00
}
```

**Erros:**

| Codigo | Descricao |
|--------|-----------|
| `404` | Lote nao encontrado |

---

## POST /api/lotes

!!! warning "Permissao"
    Requer role **Admin** ou **Gerente**.

Cria um novo lote de confinamento.

**Request Body:**

```json
{
  "codigo": "LT-2026-002",
  "descricao": "Lote Angus Fevereiro",
  "dataEntrada": "2026-02-01",
  "pesoMedioEntrada": 360.0,
  "quantidadeAnimais": 80,
  "raca": "Angus",
  "cidade": "Uberaba",
  "produtorId": "a1b2c3d4-e5f6-7890-abcd-ef1234567890"
}
```

**Resposta 201 - Criado:**

```json
{
  "id": "b2c3d4e5-f6a7-8901-bcde-f12345678901",
  "codigo": "LT-2026-002",
  "status": "Aberto"
}
```

**Erros:**

| Codigo | Descricao |
|--------|-----------|
| `400` | Dados invalidos ou codigo duplicado |
| `403` | Sem permissao |

---

## PUT /api/lotes/{id}

!!! warning "Permissao"
    Requer role **Admin** ou **Gerente**.

Atualiza os dados de um lote existente.

**Request Body:** Mesmo formato do POST.

**Resposta 200:** Lote atualizado.

**Erros:**

| Codigo | Descricao |
|--------|-----------|
| `400` | Dados invalidos |
| `403` | Sem permissao |
| `404` | Lote nao encontrado |

---

## DELETE /api/lotes/{id}

!!! danger "Permissao Restrita"
    Requer role **Admin**. Realiza soft delete (o registro nao e removido fisicamente).

**Resposta 204:** Removido com sucesso (sem corpo).

**Erros:**

| Codigo | Descricao |
|--------|-----------|
| `403` | Sem permissao |
| `404` | Lote nao encontrado |

---

## POST /api/lotes/{id}/fechar

!!! warning "Permissao"
    Requer role **Admin** ou **Gerente**.

Fecha o lote, registrando peso de saida e rendimento de carcaca. Calcula os KPIs finais.

**Request Body:**

```json
{
  "pesoSaida": 520.0,
  "rendimentoCarcaca": 54.5
}
```

**Resposta 200:**

```json
{
  "id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "status": "Fechado",
  "pesoSaida": 520.0,
  "rendimentoCarcaca": 54.5,
  "gmdFinal": 1.52,
  "diasConfinamento": 90
}
```

**Erros:**

| Codigo | Descricao |
|--------|-----------|
| `400` | Lote ja fechado ou dados invalidos |
| `403` | Sem permissao |
| `404` | Lote nao encontrado |
