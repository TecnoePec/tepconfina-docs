# Protocolos Sanitários

Endpoints para gestão do calendário de vacinação e protocolos sanitários por lote.

!!! info "Autenticacao Obrigatoria"
    Todos os endpoints requerem header `Authorization: Bearer {token}`.

## Resumo dos Endpoints

| Metodo | Rota | Roles | Descricao |
|--------|------|-------|-----------|
| `GET` | `/api/lotes/{loteId}/protocolos` | Todos | Listar protocolos do lote |
| `GET` | `/api/lotes/{loteId}/protocolos/{id}` | Todos | Detalhe de um protocolo |
| `POST` | `/api/lotes/{loteId}/protocolos` | Admin, Gerente | Criar protocolo |
| `PUT` | `/api/lotes/{loteId}/protocolos/{id}` | Admin, Gerente | Atualizar protocolo |
| `POST` | `/api/lotes/{loteId}/protocolos/{id}/realizar` | Admin, Gerente, Funcionario | Marcar como realizado |
| `DELETE` | `/api/lotes/{loteId}/protocolos/{id}` | Admin | Remover protocolo |

---

## GET /api/lotes/{loteId}/protocolos

Retorna todos os protocolos do lote, ordenados por data prevista.

**Resposta `200`:**

```json
[
  {
    "id": "uuid",
    "loteId": "uuid",
    "produto": "Aftosa",
    "tipo": "Vacina",
    "dataPrevista": "2025-06-01",
    "dataRealizada": null,
    "quantidadeMl": 2.0,
    "custoPorAnimal": 3.50,
    "realizado": false,
    "observacoes": null,
    "createdAt": "2025-05-01T10:00:00Z"
  }
]
```

---

## POST /api/lotes/{loteId}/protocolos

Cria um novo protocolo sanitário para o lote.

**Body:**

```json
{
  "produto": "Aftosa",
  "tipo": "Vacina",
  "dataPrevista": "2025-06-01",
  "quantidadeMl": 2.0,
  "custoPorAnimal": 3.50,
  "observacoes": "Dose anual obrigatória"
}
```

**Tipos sugeridos:** `Vacina`, `Vermifugo`, `Vitamina`, `Antibiotico`, `Outro`

---

## POST /api/lotes/{loteId}/protocolos/{id}/realizar

Marca um protocolo como realizado e registra a data de execução.

**Body:**

```json
{
  "dataRealizada": "2025-06-02",
  "quantidadeMl": 2.0,
  "observacoes": "Aplicado sem intercorrências"
}
```

**Resposta `200`:** Protocolo atualizado com `realizado: true` e `dataRealizada` preenchida.
