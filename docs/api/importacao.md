# Importação via Excel

Endpoints para importação em lote de pesagens a partir de arquivos Excel (.xlsx).

!!! info "Autenticacao Obrigatoria"
    Todos os endpoints requerem header `Authorization: Bearer {token}`.

## Resumo dos Endpoints

| Metodo | Rota | Roles | Descricao |
|--------|------|-------|-----------|
| `GET` | `/api/importacao/template/pesagens` | Todos | Download do template Excel |
| `POST` | `/api/importacao/pesagens` | Todos | Importar pesagens via arquivo .xlsx |

---

## GET /api/importacao/template/pesagens

Retorna um arquivo `.xlsx` com o template de importação de pesagens já formatado e com uma linha de exemplo.

**Colunas do template:**

| Coluna | Tipo | Obrigatório | Descrição |
|--------|------|-------------|-----------|
| `LoteNome` | texto | Sim | Nome exato do lote ativo |
| `Data` | data (YYYY-MM-DD) | Sim | Data da pesagem |
| `PesoMedio_kg` | decimal | Sim | Peso médio em kg (ex: 420.5) |
| `QuantidadeAnimais` | inteiro | Não | Número de animais pesados (usa qtd do lote se omitido) |
| `Observacoes` | texto | Não | Observações adicionais |

---

## POST /api/importacao/pesagens

Processa um arquivo `.xlsx` e importa as pesagens para os lotes correspondentes.

**Content-Type:** `multipart/form-data`

**Campo:** `file` — arquivo `.xlsx`

**Regras de validação:**

- O lote deve existir e estar com status `Ativo`
- A data deve estar no formato `YYYY-MM-DD`
- O peso deve ser um número decimal positivo
- Linhas com erro são ignoradas; as válidas são importadas normalmente

**Resposta `200`:**

```json
{
  "importadas": 42,
  "erros": [
    "Linha 5: Lote 'Lote X' não encontrado entre lotes ativos.",
    "Linha 8: Data inválida '2025/06/01'. Use o formato YYYY-MM-DD."
  ]
}
```

!!! warning "Duplicatas"
    O sistema não verifica duplicatas automaticamente. Se o mesmo arquivo for importado duas vezes, as pesagens serão criadas novamente. Verifique antes de reimportar.
