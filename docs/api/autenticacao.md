# Autenticacao

Endpoints para registro, login, renovacao de token e gerenciamento de dispositivos.

## Resumo dos Endpoints

| Metodo | Rota | Autenticacao | Roles | Descricao |
|--------|------|:------------:|-------|-----------|
| `POST` | `/api/auth/register` | Nao | - | Registrar novo usuario |
| `POST` | `/api/auth/login` | Nao | - | Realizar login |
| `POST` | `/api/auth/refresh` | Nao | - | Renovar JWT com refresh token |
| `PUT` | `/api/auth/change-password` | Sim | Todos | Alterar senha |
| `POST` | `/api/auth/device-token` | Sim | Todos | Registrar FCM token |
| `DELETE` | `/api/auth/device-token` | Sim | Todos | Remover FCM token |

---

## POST /api/auth/register

Registra um novo usuario no sistema.

!!! warning "Rate Limit"
    Este endpoint possui limite de **10 requisicoes por minuto** por IP.

**Request Body:**

```json
{
  "nome": "Joao Silva",
  "email": "joao@exemplo.com",
  "password": "SenhaForte@123",
  "confirmPassword": "SenhaForte@123"
}
```

**Resposta 200 - Sucesso:**

```json
{
  "id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "nome": "Joao Silva",
  "email": "joao@exemplo.com",
  "role": "Operador"
}
```

**Erros:**

| Codigo | Descricao |
|--------|-----------|
| `400` | Dados invalidos ou email ja cadastrado |
| `429` | Rate limit excedido |

---

## POST /api/auth/login

Realiza login e retorna JWT access token + refresh token.

!!! warning "Rate Limit"
    Este endpoint possui limite de **10 requisicoes por minuto** por IP.

**Request Body:**

```json
{
  "email": "admin@tepconfina.com",
  "password": "Admin@123"
}
```

**Resposta 200 - Sucesso:**

```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refreshToken": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "expiresIn": 3600,
  "user": {
    "id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "nome": "Administrador",
    "email": "admin@tepconfina.com",
    "role": "Admin"
  }
}
```

**Erros:**

| Codigo | Descricao |
|--------|-----------|
| `400` | Email ou senha invalidos |
| `429` | Rate limit excedido |

---

## POST /api/auth/refresh

Renova o JWT access token utilizando um refresh token valido.

**Request Body:**

```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refreshToken": "a1b2c3d4-e5f6-7890-abcd-ef1234567890"
}
```

**Resposta 200 - Sucesso:**

```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refreshToken": "f6e5d4c3-b2a1-0987-fedc-ba0987654321",
  "expiresIn": 3600
}
```

**Erros:**

| Codigo | Descricao |
|--------|-----------|
| `400` | Refresh token invalido ou expirado |

---

## PUT /api/auth/change-password

!!! info "Autenticacao Obrigatoria"
    Requer header `Authorization: Bearer {token}`.

Altera a senha do usuario autenticado.

**Request Body:**

```json
{
  "currentPassword": "Admin@123",
  "newPassword": "NovaSenha@456",
  "confirmNewPassword": "NovaSenha@456"
}
```

**Resposta 200 - Sucesso:**

```json
{
  "message": "Senha alterada com sucesso"
}
```

**Erros:**

| Codigo | Descricao |
|--------|-----------|
| `400` | Senha atual incorreta ou nova senha invalida |
| `401` | Token ausente ou invalido |

---

## POST /api/auth/device-token

!!! info "Autenticacao Obrigatoria"
    Requer header `Authorization: Bearer {token}`.

Registra um token FCM (Firebase Cloud Messaging) para notificacoes push.

**Request Body:**

```json
{
  "token": "fMI-qKkVR3e...:APA91bH...",
  "platform": "android"
}
```

**Resposta 200 - Sucesso:**

```json
{
  "message": "Device token registrado com sucesso"
}
```

**Erros:**

| Codigo | Descricao |
|--------|-----------|
| `400` | Token invalido |
| `401` | Nao autenticado |

---

## DELETE /api/auth/device-token

!!! info "Autenticacao Obrigatoria"
    Requer header `Authorization: Bearer {token}`.

Remove um token FCM registrado anteriormente.

**Request Body:**

```json
{
  "token": "fMI-qKkVR3e...:APA91bH..."
}
```

**Resposta 200 - Sucesso:**

```json
{
  "message": "Device token removido com sucesso"
}
```

**Erros:**

| Codigo | Descricao |
|--------|-----------|
| `400` | Token nao encontrado |
| `401` | Nao autenticado |
