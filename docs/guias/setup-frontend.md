# Setup do Frontend

Guia completo para configurar e executar o frontend do TepConfina localmente.

## Pre-requisitos

| Ferramenta | Versao Minima | Observacao                           |
|------------|---------------|--------------------------------------|
| Node.js    | 20.0          | LTS recomendado                      |
| npm        | 10.0          | Incluso no Node.js                   |
| Git        | 2.40+         | Controle de versao                   |

!!! info "Backend obrigatorio"
    O frontend depende da API backend rodando em `localhost:5000`. Configure o backend antes de prosseguir.

## Clonando o Repositorio

```bash
git clone git@github.com:TecnoePec/tepconfina-web.git
cd tepconfina-web
```

## Instalando Dependencias

```bash
npm install
```

### Principais Dependencias

| Pacote           | Funcao                          |
|------------------|---------------------------------|
| React 18         | Framework UI                    |
| Vite 5           | Build tool e dev server         |
| TypeScript       | Tipagem estatica                |
| Tailwind CSS 3   | Framework de estilos            |
| TanStack Query   | Gerenciamento de estado servidor|
| Zustand          | Gerenciamento de estado local   |
| react-hook-form  | Gerenciamento de formularios    |
| zod              | Validacao de schemas            |

## Configuracao de Ambiente

Copie o arquivo de exemplo:

```bash
cp .env.example .env
```

Conteudo do `.env`:

```env
VITE_API_URL=http://localhost:5000
VITE_APP_NAME=TepConfina
VITE_APP_VERSION=1.0.0
```

## Proxy da API

O Vite esta configurado para redirecionar chamadas `/api/*` para o backend:

```typescript
// vite.config.ts
export default defineConfig({
  server: {
    port: 5173,
    proxy: {
      '/api': {
        target: 'http://localhost:5000',
        changeOrigin: true,
      },
    },
  },
});
```

!!! tip "Sem problemas de CORS"
    O proxy do Vite elimina problemas de CORS em desenvolvimento, pois as requisicoes sao feitas pelo mesmo dominio.

## Executando o Projeto

```bash
npm run dev
```

O frontend estara disponivel em `http://localhost:5173`.

### Scripts Disponiveis

| Comando            | Descricao                           |
|--------------------|-------------------------------------|
| `npm run dev`      | Inicia o servidor de desenvolvimento|
| `npm run build`    | Gera o build de producao           |
| `npm run preview`  | Visualiza o build de producao      |
| `npm run lint`     | Executa o ESLint                   |
| `npm run test`     | Executa os testes com Vitest       |

## Login

Utilize as credenciais do usuario seed:

| Campo | Valor                      |
|-------|----------------------------|
| Email | `admin@tepconfina.com`     |
| Senha | `Admin@123`                |

## Estrutura do Projeto

```
src/
  components/     # Componentes reutilizaveis (Sidebar, Header)
  pages/          # Paginas da aplicacao (Dashboard, Lotes)
  services/       # Comunicacao com a API (api.ts, loteService)
  stores/         # Estado global com Zustand (authStore)
  hooks/          # Hooks customizados
  types/          # Tipagens TypeScript
  utils/          # Funcoes utilitarias (formatters)
```

## Cores do Tema

| Nome    | Hex       | Uso                        |
|---------|-----------|----------------------------|
| Primary | `#1B3A5C` | Sidebar, botoes principais |
| Accent  | `#2E7D32` | Indicadores positivos      |

!!! success "Verificacao"
    Apos o login, voce deve ser redirecionado para o Dashboard com os indicadores de lotes ativos.
