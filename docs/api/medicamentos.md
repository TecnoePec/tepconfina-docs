# Medicamentos

Endpoints para gerenciamento de medicamentos aplicados aos animais de um lote.

!!! info "Autenticacao Obrigatoria"
    Todos os endpoints desta secao requerem header `Authorization: Bearer {token}`.

## Resumo dos Endpoints

| Metodo | Rota | Roles | Descricao |
|--------|------|-------|-----------|
| `GET` | `/api/lotes/{loteId}/medicamentos` | Todos | Listar medicamentos do lote |
| `GET` | `/api/lotes/{loteId}/medicamentos/{id}` | Todos | Detalhe do medicamento |
| `POST` | `/api/lotes/{loteId}/medicamentos` | Admin, Gerente | Registrar medicamento |
| `PUT` | `/api/lotes/{loteId}/medicamentos/{id}` | Admin, Gerente | Atualizar medicamento |
| `DELETE` | `/api/lotes/{loteId}/medicamentos/{id}` | Admin | Remover medicamento |

---

## GET /api/lotes/{loteId}/medicamentos

Lista todos os registros de medicamentos aplicados em um lote.

**Path Parameters:**

| Parametro | Tipo | Descricao |
|-----------|------|-----------|
| `loteId` | guid | ID do lote |

**Resposta 200:**

```json
[
  {
    "id": "f2a3b4c5-d6e7-8901-f234-567890123456",
    "nome": "Ivermectina 1%",
    "tipo": "Antiparasitario",
    "dataAplicacao": "2026-01-20",
    "dosagem": "1ml/50kg",
    "quantidadeAnimais": 120,
    "custoUnitario": 8.50,
    "custoTotal": 1020.00,
    "loteId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "observacao": "Aplicacao preventiva na entrada"
  },
  {
    "id": "a3b4c5d6-e7f8-9012-a345-678901234567",
    "nome": "Vacina Aftosa",
    "tipo": "Vacina",
    "dataAplicacao": "2026-02-10",
    "dosagem": "5ml",
    "quantidadeAnimais": 120,
    "custoUnitario": 3.20,
    "custoTotal": 384.00,
    "loteId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "observacao": "Campanha vacinacao"
  }
]
```

| Codigo | Descricao |
|--------|-----------|
| `404` | Lote nao encontrado |

---

## GET /api/lotes/{loteId}/medicamentos/{id}

Retorna os detalhes de um registro de medicamento especifico.

**Path Parameters:**

| Parametro | Tipo | Descricao |
|-----------|------|-----------|
| `loteId` | guid | ID do lote |
| `id` | guid | ID do medicamento |

**Resposta 200:**

```json
{
  "id": "f2a3b4c5-d6e7-8901-f234-567890123456",
  "nome": "Ivermectina 1%",
  "tipo": "Antiparasitario",
  "dataAplicacao": "2026-01-20",
  "dosagem": "1ml/50kg",
  "quantidadeAnimais": 120,
  "custoUnitario": 8.50,
  "custoTotal": 1020.00,
  "loteId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "loteCodigo": "LT-2026-001",
  "observacao": "Aplicacao preventiva na entrada",
  "fornecedor": "Vetpharma Ltda"
}
```

| Codigo | Descricao |
|--------|-----------|
| `404` | Lote ou medicamento nao encontrado |

---

## POST /api/lotes/{loteId}/medicamentos

!!! warning "Permissao"
    Requer role **Admin** ou **Gerente**.

Registra a aplicacao de um medicamento no lote.

**Request Body:**

```json
{
  "nome": "Oxitetraciclina LA",
  "tipo": "Antibiotico",
  "dataAplicacao": "2026-03-01",
  "dosagem": "1ml/10kg",
  "quantidadeAnimais": 15,
  "custoUnitario": 12.00,
  "observacao": "Tratamento de animais com sintomas respiratorios",
  "fornecedor": "Vetpharma Ltda"
}
```

**Resposta 201:**

```json
{
  "id": "b4c5d6e7-f8a9-0123-b456-789012345678",
  "nome": "Oxitetraciclina LA",
  "tipo": "Antibiotico",
  "dataAplicacao": "2026-03-01",
  "custoTotal": 180.00,
  "loteId": "3fa85f64-5717-4562-b3fc-2c963f66afa6"
}
```

| Codigo | Descricao |
|--------|-----------|
| `400` | Dados invalidos |
| `403` | Sem permissao |
| `404` | Lote nao encontrado |

---

## PUT /api/lotes/{loteId}/medicamentos/{id}

!!! warning "Permissao"
    Requer role **Admin** ou **Gerente**.

Atualiza um registro de medicamento existente.

**Request Body:**

```json
{
  "nome": "Oxitetraciclina LA",
  "tipo": "Antibiotico",
  "dataAplicacao": "2026-03-01",
  "dosagem": "1ml/10kg",
  "quantidadeAnimais": 18,
  "custoUnitario": 12.00,
  "observacao": "Tratamento ampliado - mais 3 animais identificados",
  "fornecedor": "Vetpharma Ltda"
}
```

**Resposta 200:** Medicamento atualizado.

| Codigo | Descricao |
|--------|-----------|
| `400` | Dados invalidos |
| `403` | Sem permissao |
| `404` | Lote ou medicamento nao encontrado |

---

## DELETE /api/lotes/{loteId}/medicamentos/{id}

!!! danger "Permissao Restrita"
    Requer role **Admin**.

Remove um registro de medicamento.

**Resposta 204:** Removido com sucesso.

| Codigo | Descricao |
|--------|-----------|
| `403` | Sem permissao |
| `404` | Lote ou medicamento nao encontrado |
