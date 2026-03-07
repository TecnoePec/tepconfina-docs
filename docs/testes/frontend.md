# Testes do Frontend

Estrategia e estrutura de testes do frontend React do TepConfina.

## Framework e Ferramentas

| Ferramenta        | Funcao                                    |
|-------------------|-------------------------------------------|
| Vitest            | Framework de testes unitarios             |
| Testing Library   | Testes de componentes React               |
| MSW               | Mock Service Worker para interceptar APIs |
| Playwright        | Testes end-to-end                         |

## Estrutura do Projeto

```
src/
  __tests__/
    utils/
      formatters.test.ts
      validators.test.ts
    services/
      authService.test.ts
      loteService.test.ts
    stores/
      authStore.test.ts
    components/
      Sidebar.test.tsx
      Header.test.tsx
    pages/
      LoginPage.test.tsx
      Dashboard.test.tsx
      LotesPage.test.tsx
e2e/
  global.setup.ts
  auth.spec.ts
  lotes.spec.ts
  dashboard.spec.ts
```

## Testes Unitarios (100 testes)

### Testes de Utilitarios

```typescript
// formatters.test.ts
import { formatCurrency, formatWeight, formatDate } from '@/utils/formatters';

describe('formatCurrency', () => {
  it('deve formatar valor em reais', () => {
    expect(formatCurrency(1500.50)).toBe('R$ 1.500,50');
  });

  it('deve formatar zero', () => {
    expect(formatCurrency(0)).toBe('R$ 0,00');
  });

  it('deve formatar valores negativos', () => {
    expect(formatCurrency(-100)).toBe('-R$ 100,00');
  });
});
```

### Testes de Validacao

```typescript
// validators.test.ts
import { loteSchema } from '@/utils/validators';

describe('loteSchema', () => {
  it('deve validar lote com dados corretos', () => {
    const result = loteSchema.safeParse({
      nome: 'Lote 001',
      quantidadeAnimais: 30,
      pesoLiquidoEntrada: 15000,
      dataEntrada: '2026-01-15',
    });
    expect(result.success).toBe(true);
  });

  it('deve rejeitar quantidade de animais negativa', () => {
    const result = loteSchema.safeParse({
      nome: 'Lote 001',
      quantidadeAnimais: -5,
    });
    expect(result.success).toBe(false);
  });
});
```

### Testes de Services com MSW

```typescript
// loteService.test.ts
import { server } from '@/mocks/server';
import { http, HttpResponse } from 'msw';
import { loteService } from '@/services/loteService';

describe('loteService', () => {
  it('deve buscar lista de lotes', async () => {
    server.use(
      http.get('/api/lotes', () => {
        return HttpResponse.json([
          { id: '1', nome: 'Lote 001', status: 'Ativo' },
          { id: '2', nome: 'Lote 002', status: 'Fechado' },
        ]);
      })
    );

    const lotes = await loteService.listar();
    expect(lotes).toHaveLength(2);
    expect(lotes[0].nome).toBe('Lote 001');
  });
});
```

### Testes de Store (Zustand)

```typescript
// authStore.test.ts
import { useAuthStore } from '@/stores/authStore';

describe('authStore', () => {
  beforeEach(() => {
    useAuthStore.setState({ token: null, user: null });
  });

  it('deve armazenar token apos login', () => {
    useAuthStore.getState().setAuth({
      token: 'jwt-token',
      user: { id: '1', nome: 'Admin', role: 'Admin' },
    });

    expect(useAuthStore.getState().token).toBe('jwt-token');
    expect(useAuthStore.getState().isAuthenticated).toBe(true);
  });

  it('deve limpar estado apos logout', () => {
    useAuthStore.getState().setAuth({ token: 'jwt-token', user: {} });
    useAuthStore.getState().logout();

    expect(useAuthStore.getState().token).toBeNull();
    expect(useAuthStore.getState().isAuthenticated).toBe(false);
  });
});
```

### Distribuicao dos Testes Unitarios

| Categoria     | Qtd  | Descricao                              |
|---------------|------|----------------------------------------|
| Formatters    | 20   | Formatacao de moeda, peso, data, porcentagem |
| Validation    | 15   | Schemas zod para formularios           |
| Services      | 25   | Chamadas API com MSW                   |
| Stores        | 10   | Estado global Zustand                  |
| Components    | 20   | Renderizacao e interacao               |
| Hooks         | 10   | Hooks customizados                     |

## Testes E2E (22 testes)

### Configuracao Global

```typescript
// global.setup.ts
import { chromium } from '@playwright/test';

async function globalSetup() {
  const browser = await chromium.launch();
  const page = await browser.newPage();

  await page.goto('http://localhost:5173/login');
  await page.fill('[name="email"]', 'admin@tepconfina.com');
  await page.fill('[name="password"]', 'Admin@123');
  await page.click('button[type="submit"]');
  await page.waitForURL('/dashboard');

  await page.context().storageState({ path: 'e2e/.auth/state.json' });
  await browser.close();
}

export default globalSetup;
```

### Exemplo de Teste E2E

```typescript
// lotes.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Lotes', () => {
  test('deve listar lotes na pagina', async ({ page }) => {
    await page.goto('/lotes');
    await expect(page.getByRole('heading', { name: 'Lotes' })).toBeVisible();
    await expect(page.locator('table tbody tr')).toHaveCount.greaterThan(0);
  });

  test('deve criar novo lote', async ({ page }) => {
    await page.goto('/lotes/novo');
    await page.fill('[name="nome"]', 'Lote E2E Test');
    await page.fill('[name="quantidadeAnimais"]', '25');
    await page.click('button[type="submit"]');
    await expect(page.getByText('Lote criado com sucesso')).toBeVisible();
  });
});
```

## Helpers de Teste

### Render com Providers

```typescript
import { render } from '@testing-library/react';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { BrowserRouter } from 'react-router-dom';

export function renderWithProviders(ui: React.ReactElement) {
  const queryClient = new QueryClient({
    defaultOptions: { queries: { retry: false } },
  });

  return render(
    <QueryClientProvider client={queryClient}>
      <BrowserRouter>{ui}</BrowserRouter>
    </QueryClientProvider>
  );
}
```

## Executando os Testes

| Comando                    | Descricao                          |
|----------------------------|------------------------------------|
| `npx vitest run`           | Executar todos os testes unitarios |
| `npx vitest --watch`       | Modo watch (desenvolvimento)       |
| `npx vitest --coverage`    | Gerar relatorio de cobertura      |
| `npx playwright test`      | Executar testes E2E                |
| `npx playwright test --ui` | Executar E2E com interface visual  |

!!! warning "Pre-requisito E2E"
    Os testes E2E requerem que o backend e o frontend estejam rodando localmente antes da execucao.
