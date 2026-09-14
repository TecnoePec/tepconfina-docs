# Crédito de Carbono — Redução de Metano Entérico

Módulo para **calcular, rastrear e documentar** a redução de metano entérico proporcionada por aditivos antimetânicos na ração, gerando a evidência auditável que o produtor usa para **comercializar crédito de carbono**.

!!! danger "Crédito ≠ estimativa"
    Vender crédito de carbono exige **MRV** (Monitoramento, Relato, Verificação) sob uma **metodologia/standard** (Verra VCS, Gold Standard ou registro nacional), com **baseline, adicionalidade, permanência e verificação por terceiro**. O TepConfina é a **plataforma de dados/MRV** — calcula e documenta a redução; **não emite** o crédito.

## Fórmula

```
CO₂e evitado (animal) = Fator (gCO₂e / kg MS) × MS ingerida (kg)
MS ingerida           = ração consumida (kg)  × (% Matéria Seca da ração / 100)
```

- **Fator** = abatimento por kg de matéria seca ingerida (ex.: **45,7 gCO₂e/kg MS** — o "−45,7" já é o delta pronto).
- **Redução de metano %** (ex.: 14,9%) = referência relativa, para relatório.
- Agregação: soma dos `ConsumoRacao` do lote × %MS da ração × Fator → **tCO₂e evitado** por lote/período/fazenda.

!!! example "Ordem de grandeza"
    Boi ~10 kg MS/dia × 100 dias = 1.000 kg MS → ~**45,7 kg CO₂e/animal** (0,0457 tCO₂e). Lote de 100 cab → ~**4,6 tCO₂e**. A ~R$50–100/tCO₂e no mercado voluntário → ~R$230–460/lote. Modesto por animal, relevante no rebanho.

## Modelo de dados

Campos adicionados à **`Racao`** (marca/tipo):

| Campo | Tipo | Descrição |
|-------|------|-----------|
| `PercentualMateriaSeca` | decimal % | % de MS da ração (ex.: 88) |
| `PossuiAditivoAntimetanico` | bool | marca a ração como redutora de metano |
| `NomeAditivo` | string? | nome comercial do aditivo |
| `FatorReducaoCO2ePorKgMS` | decimal (g) | abatimento por kg de MS (ex.: 45.7) |
| `ReducaoMetanoPercentual` | decimal % | redução relativa de metano (ex.: 14.9) |
| `FonteFatorCarbono` | string? | referência do laudo/estudo (MRV) |
| `DataValidadeFator` | date? | validade do laudo |

O consumo (`ConsumoRacao.QuantidadeKg`, já existente) alimenta o cálculo automaticamente. Preço da tonelada configurável em `Carbono:PrecoToneladaCarbonoBRL`.

!!! note "Decisão de modelagem (futuro)"
    Hoje um `Racao` representa uma **compra** (tem `DataCompra`/`QuantidadeSacos`). O ideal é separar **Catálogo de Marca** (MS + carbono) × **Compra** (lote). No MVP os campos ficam no `Racao` atual.

## API (Fase 1)

- `GET /api/lotes/{id}/carbono` — tCO₂e evitado + valor estimado do lote
- `GET /api/carbono/resumo` — total tCO₂e evitado + potencial (dashboard)
- `GET /api/lotes/{id}/carbono/relatorio` (Fase 2) — evidência MRV em PDF

## Roadmap

| Fase | Entrega |
|------|---------|
| **1 — MVP** | Campos na ração + `CarbonoService` + endpoint + card no lote/dashboard (estimativa interna, MRV-ready) |
| **2 — Evidência MRV** | Relatório PDF auditável, baseline, fonte do fator, trilha (AuditLog) |
| **3 — Comercialização** | Integração com registro/marketplace de carbono + verificação por terceiro |

## Premissas do MVP (revisar com o negócio)
1. **Standard**: estimativa interna por ora, projetada para ser MRV-ready.
2. **Fator**: fixo por marca/aditivo (campo na `Racao`).
3. **% MS**: cadastrado por marca (simples) — evoluir para análise bromatológica por compra.
4. **Preço do carbono**: configurável (`Carbono:PrecoToneladaCarbonoBRL`).

## Caveats
- O `FatorReducaoCO2ePorKgMS` precisa vir de **laudo/estudo validado** para o aditivo (e idealmente para as condições locais).
- Crédito real exige **baseline + adicionalidade + verificação** — o cálculo do TepConfina é a *evidência*, não o crédito.
