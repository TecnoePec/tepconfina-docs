# Testes de Carga

Testes de performance e carga da API do TepConfina com k6.

## Framework

| Ferramenta | Versao | Descricao                                |
|------------|--------|------------------------------------------|
| k6         | 0.50+  | Ferramenta de testes de carga open-source|

## Estrutura dos Scripts

```
tests/load/k6/
  smoke.js        # Teste basico de sanidade
  load.js         # Teste de carga tipica
  stress.js       # Teste de estresse
  helpers/
    auth.js       # Helper de autenticacao
    config.js     # Configuracoes compartilhadas
```

## Cenarios

### Smoke Test

Verifica se a API responde corretamente sob carga minima.

| Configuracao | Valor     |
|--------------|-----------|
| VUs          | 1         |
| Duracao      | 30 segundos |

```javascript
// smoke.js
import http from 'k6/http';
import { check, sleep } from 'k6';
import { authenticate } from './helpers/auth.js';

export const options = {
  vus: 1,
  duration: '30s',
  thresholds: {
    http_req_duration: ['p(95)<500'],
    http_req_failed: ['rate<0.01'],
  },
};

export function setup() {
  return authenticate();
}

export default function (data) {
  const headers = { Authorization: `Bearer ${data.token}` };

  // Listar lotes
  const lotesRes = http.get(`${__ENV.BASE_URL}/api/lotes`, { headers });
  check(lotesRes, {
    'lotes status 200': (r) => r.status === 200,
    'lotes tem dados': (r) => JSON.parse(r.body).length > 0,
  });

  sleep(1);

  // Dashboard
  const dashRes = http.get(`${__ENV.BASE_URL}/api/dashboard`, { headers });
  check(dashRes, {
    'dashboard status 200': (r) => r.status === 200,
  });

  sleep(1);
}
```

### Load Test

Simula carga tipica de uso em producao.

| Configuracao | Valor          |
|--------------|----------------|
| VUs          | 20             |
| Duracao      | 3 minutos      |
| Ramp-up      | 30 segundos    |
| Ramp-down    | 30 segundos    |

```javascript
// load.js
import http from 'k6/http';
import { check, sleep } from 'k6';
import { authenticate } from './helpers/auth.js';

export const options = {
  stages: [
    { duration: '30s', target: 20 },  // ramp-up
    { duration: '2m', target: 20 },   // carga estavel
    { duration: '30s', target: 0 },   // ramp-down
  ],
  thresholds: {
    http_req_duration: ['p(95)<500'],
    http_req_failed: ['rate<0.01'],
  },
};

export function setup() {
  return authenticate();
}

export default function (data) {
  const headers = { Authorization: `Bearer ${data.token}` };

  // Fluxo: listar lotes -> detalhe -> pesagens
  const lotesRes = http.get(`${__ENV.BASE_URL}/api/lotes`, { headers });
  check(lotesRes, { 'lotes ok': (r) => r.status === 200 });

  const lotes = JSON.parse(lotesRes.body);
  if (lotes.length > 0) {
    const loteId = lotes[0].id;

    const detalheRes = http.get(
      `${__ENV.BASE_URL}/api/lotes/${loteId}`, { headers });
    check(detalheRes, { 'detalhe ok': (r) => r.status === 200 });

    const pesagensRes = http.get(
      `${__ENV.BASE_URL}/api/lotes/${loteId}/pesagens`, { headers });
    check(pesagensRes, { 'pesagens ok': (r) => r.status === 200 });
  }

  sleep(1);
}
```

### Stress Test

Testa os limites da API sob carga extrema.

| Configuracao | Valor          |
|--------------|----------------|
| VUs          | 50             |
| Duracao      | 5 minutos      |
| Ramp-up      | 1 minuto       |
| Ramp-down    | 1 minuto       |

```javascript
// stress.js
import http from 'k6/http';
import { check, sleep } from 'k6';
import { authenticate } from './helpers/auth.js';

export const options = {
  stages: [
    { duration: '1m', target: 50 },   // ramp-up agressivo
    { duration: '3m', target: 50 },   // carga maxima
    { duration: '1m', target: 0 },    // ramp-down
  ],
  thresholds: {
    http_req_duration: ['p(95)<1000'],
    http_req_failed: ['rate<0.05'],
  },
};
```

## Cenarios Testados

| Cenario          | Endpoints                                         |
|------------------|----------------------------------------------------|
| Autenticacao     | `POST /api/auth/login`, `POST /api/auth/refresh`  |
| Lotes CRUD       | `GET /api/lotes`, `GET /api/lotes/:id`, `POST /api/lotes` |
| Pesagens         | `GET /api/lotes/:id/pesagens`, `POST /api/pesagens`|
| Animais          | `GET /api/lotes/:id/animais`, `POST /api/animais` |

## Thresholds (Criterios de Aceite)

| Metrica              | Smoke     | Load      | Stress    |
|----------------------|-----------|-----------|-----------|
| p(95) latencia       | < 500ms   | < 500ms   | < 1000ms  |
| Taxa de erro         | < 1%      | < 1%      | < 5%      |
| p(99) latencia       | < 1000ms  | < 1000ms  | < 2000ms  |

## Helper de Autenticacao

```javascript
// helpers/auth.js
import http from 'k6/http';

export function authenticate() {
  const res = http.post(`${__ENV.BASE_URL}/api/auth/login`,
    JSON.stringify({
      email: 'admin@tepconfina.com',
      password: 'Admin@123',
    }),
    { headers: { 'Content-Type': 'application/json' } }
  );

  return { token: JSON.parse(res.body).token };
}
```

## Executando os Testes

```bash
# Smoke test
k6 run --env BASE_URL=http://localhost:5000 tests/load/k6/smoke.js

# Load test
k6 run --env BASE_URL=http://localhost:5000 tests/load/k6/load.js

# Stress test
k6 run --env BASE_URL=http://localhost:5000 tests/load/k6/stress.js
```

!!! warning "Ambiente de teste"
    Execute testes de carga apenas em ambientes de staging. Nunca execute em producao sem autorizacao.

## Interpretando Resultados

```
     ✓ lotes ok
     ✓ detalhe ok

     http_req_duration...: avg=45ms  min=12ms  p(95)=120ms  p(99)=250ms
     http_req_failed.....: 0.00%
     http_reqs...........: 2400     40/s
     vus.................: 20       min=20  max=20
```

| Metrica              | Descricao                              |
|----------------------|----------------------------------------|
| `http_req_duration`  | Distribuicao de latencia               |
| `http_req_failed`    | Percentual de requisicoes com falha    |
| `http_reqs`          | Total de requisicoes e taxa por segundo|
| `vus`                | Usuarios virtuais simultanoes          |

!!! tip "Metricas customizadas"
    Use `Trend`, `Counter` e `Rate` do k6 para criar metricas especificas do negocio, como tempo de resposta por endpoint.
