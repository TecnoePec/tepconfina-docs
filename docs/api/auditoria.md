# Auditoria

Endpoint para consulta do log de auditoria de ações realizadas no sistema.

!!! info "Autenticacao Obrigatoria"
    Requer role `Admin` ou `SuperAdmin`.

## Resumo dos Endpoints

| Metodo | Rota | Roles | Descricao |
|--------|------|-------|-----------|
| `GET` | `/api/audit` | Admin, SuperAdmin | Listar logs de auditoria paginados |

---

## GET /api/audit

Retorna os registros de auditoria com suporte a filtros e paginação.

**Query Parameters:**

| Parametro | Tipo | Descrição |
|-----------|------|-----------|
| `method` | string | Filtrar por método HTTP (`POST`, `PUT`, `PATCH`, `DELETE`) |
| `userEmail` | string | Filtrar por e-mail do usuário que executou a ação |
| `de` | datetime | Data/hora de início do período |
| `ate` | datetime | Data/hora de fim do período |
| `page` | int | Número da página (padrão: 1) |

!!! note "Isolamento por tenant"
    Admins visualizam apenas logs do próprio tenant. SuperAdmins visualizam logs de todos os tenants.

**Resposta `200`:**

```json
{
  "items": [
    {
      "id": "uuid",
      "method": "POST",
      "path": "/api/lotes",
      "statusCode": 201,
      "userEmail": "gerente@fazenda.com",
      "ipAddress": "192.168.1.1",
      "durationMs": 45,
      "tenantId": "uuid",
      "createdAt": "2025-06-01T14:32:00Z"
    }
  ],
  "totalCount": 1250,
  "totalPages": 25,
  "page": 1
}
```

**Campos do log:**

| Campo | Descrição |
|-------|-----------|
| `method` | Verbo HTTP da requisição |
| `path` | Rota acessada |
| `statusCode` | Código HTTP de resposta |
| `userEmail` | E-mail do usuário autenticado |
| `ipAddress` | IP de origem da requisição |
| `durationMs` | Tempo de resposta em milissegundos |
| `createdAt` | Timestamp UTC da ação |
