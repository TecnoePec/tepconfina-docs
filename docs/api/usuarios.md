# Usuarios

Endpoints para gerenciamento de usuarios do sistema. A maioria das operacoes e restrita ao perfil Admin.

!!! info "Autenticacao Obrigatoria"
    Todos os endpoints desta secao requerem header `Authorization: Bearer {token}`.

## Roles do Sistema

| Role | Descricao |
|------|-----------|
| `Admin` | Acesso total ao sistema, gerencia usuarios |
| `Gerente` | Cria e edita lotes, animais, racoes e medicamentos |
| `Operador` | Acesso somente leitura aos dados |

## Resumo dos Endpoints

| Metodo | Rota | Roles | Descricao |
|--------|------|-------|-----------|
| `GET` | `/api/users` | Admin | Listar usuarios |
| `GET` | `/api/users/{id}` | Admin | Detalhe do usuario |
| `POST` | `/api/users` | Admin | Criar usuario |
| `PUT` | `/api/users/{id}` | Admin | Atualizar usuario |
| `DELETE` | `/api/users/{id}` | Admin | Remover usuario |
| `PUT` | `/api/users/{id}/toggle-active` | Admin | Ativar/desativar usuario |

---

## GET /api/users

!!! danger "Permissao Restrita"
    Requer role **Admin**.

Lista todos os usuarios cadastrados no sistema.

**Resposta 200:**

```json
[
  {
    "id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "nome": "Administrador",
    "email": "admin@tepconfina.com",
    "role": "Admin",
    "ativo": true,
    "criadoEm": "2026-01-01T00:00:00Z"
  },
  {
    "id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "nome": "Carlos Gerente",
    "email": "carlos@tepconfina.com",
    "role": "Gerente",
    "ativo": true,
    "criadoEm": "2026-01-15T10:30:00Z"
  }
]
```

| Codigo | Descricao |
|--------|-----------|
| `403` | Sem permissao (nao e Admin) |

---

## GET /api/users/{id}

!!! danger "Permissao Restrita"
    Requer role **Admin**.

Retorna os detalhes de um usuario especifico.

**Resposta 200:**

```json
{
  "id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "nome": "Carlos Gerente",
  "email": "carlos@tepconfina.com",
  "role": "Gerente",
  "ativo": true,
  "criadoEm": "2026-01-15T10:30:00Z",
  "ultimoLogin": "2026-03-06T08:15:00Z"
}
```

| Codigo | Descricao |
|--------|-----------|
| `403` | Sem permissao |
| `404` | Usuario nao encontrado |

---

## POST /api/users

!!! danger "Permissao Restrita"
    Requer role **Admin**.

Cria um novo usuario no sistema.

**Request Body:**

```json
{
  "nome": "Maria Operadora",
  "email": "maria@tepconfina.com",
  "password": "Senha@123",
  "confirmPassword": "Senha@123",
  "role": "Operador"
}
```

**Resposta 201:**

```json
{
  "id": "b2c3d4e5-f6a7-8901-bcde-f12345678901",
  "nome": "Maria Operadora",
  "email": "maria@tepconfina.com",
  "role": "Operador",
  "ativo": true
}
```

| Codigo | Descricao |
|--------|-----------|
| `400` | Dados invalidos, email duplicado ou role invalido |
| `403` | Sem permissao |

---

## PUT /api/users/{id}

!!! danger "Permissao Restrita"
    Requer role **Admin**.

Atualiza os dados de um usuario existente.

**Request Body:**

```json
{
  "nome": "Maria Gerente",
  "email": "maria@tepconfina.com",
  "role": "Gerente"
}
```

**Resposta 200:** Usuario atualizado.

| Codigo | Descricao |
|--------|-----------|
| `400` | Dados invalidos ou email duplicado |
| `403` | Sem permissao |
| `404` | Usuario nao encontrado |

---

## DELETE /api/users/{id}

!!! danger "Permissao Restrita"
    Requer role **Admin**.

Remove um usuario do sistema.

**Resposta 204:** Removido com sucesso.

!!! warning "Atencao"
    Nao e possivel remover o proprio usuario logado.

| Codigo | Descricao |
|--------|-----------|
| `400` | Tentativa de remover o proprio usuario |
| `403` | Sem permissao |
| `404` | Usuario nao encontrado |

---

## PUT /api/users/{id}/toggle-active

!!! danger "Permissao Restrita"
    Requer role **Admin**.

Alterna o estado ativo/inativo de um usuario. Usuarios inativos nao conseguem realizar login.

**Resposta 200:**

```json
{
  "id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "nome": "Carlos Gerente",
  "ativo": false,
  "message": "Usuario desativado com sucesso"
}
```

!!! warning "Atencao"
    Nao e possivel desativar o proprio usuario logado.

| Codigo | Descricao |
|--------|-----------|
| `400` | Tentativa de desativar o proprio usuario |
| `403` | Sem permissao |
| `404` | Usuario nao encontrado |
