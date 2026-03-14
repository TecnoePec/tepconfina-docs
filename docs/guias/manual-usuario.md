# Manual do Usuário — TepConfina

> Sistema de Gestão de Confinamento Bovino
> Versão 1.1 · Tecnoepec

---

## Sumário

1. [Acesso ao Sistema](#1-acesso-ao-sistema)
2. [Visão Geral da Interface](#2-visão-geral-da-interface)
3. [Dashboard](#3-dashboard)
4. [Lotes](#4-lotes)
5. [Animais](#5-animais)
6. [Pesagens](#6-pesagens)
7. [Rações](#7-rações)
8. [Estoque de Insumos](#8-estoque-de-insumos)
9. [Produtores](#9-produtores)
10. [Mercado e Preços](#10-mercado-e-preços)
11. [Simulador de Hedge](#11-simulador-de-hedge)
12. [Financeiro](#12-financeiro)
13. [Relatórios](#13-relatórios)
14. [Agente IA](#14-agente-ia)
15. [Alertas de Preço](#15-alertas-de-preço)
16. [Notificações](#16-notificações)
17. [Usuários](#17-usuários)
18. [Perfis e Permissões](#18-perfis-e-permissões)
19. [Protocolo Sanitário](#19-protocolo-sanitário)
20. [Importação via Excel](#20-importação-via-excel)
21. [Auditoria](#21-auditoria)
22. [Aplicativo Mobile](#22-aplicativo-mobile)
23. [Perguntas Frequentes](#23-perguntas-frequentes)
17. [Usuários](#17-usuários)
18. [Perfis e Permissões](#18-perfis-e-permissões)
19. [Aplicativo Mobile](#19-aplicativo-mobile)
20. [Perguntas Frequentes](#20-perguntas-frequentes)

---

## 1. Acesso ao Sistema

### Web

Acesse **https://tepconfina.tecnoepec.com.br** em qualquer navegador moderno (Chrome, Firefox, Edge, Safari).

| Campo | Valor padrão (ambiente inicial) |
|-------|--------------------------------|
| E-mail | `admin@tepconfina.com` |
| Senha | `Admin@123` |

> **Recomendação:** Altere a senha padrão no primeiro acesso em **Usuários → Editar → Alterar Senha**.

### Login

1. Informe o e-mail e a senha cadastrados.
2. Clique em **Entrar**.
3. Caso esqueça a senha, entre em contato com o administrador do sistema para redefinição.

---

## 2. Visão Geral da Interface

```
┌──────────────────┬──────────────────────────────────────────┐
│                  │  Header: usuário, notificações, busca    │
│   Sidebar        ├──────────────────────────────────────────┤
│   (menu lateral) │                                          │
│                  │   Área de conteúdo principal             │
│   • Dashboard    │                                          │
│   • Lotes        │                                          │
│   • Animais      │                                          │
│   • ...          │                                          │
└──────────────────┴──────────────────────────────────────────┘
```

- **Sidebar** — Navegação principal. Pode ser recolhida clicando na seta `‹` (desktop).
- **Header** — Exibe o usuário logado, sino de notificações e busca rápida (`Ctrl+K`).
- **Breadcrumb** — Caminho atual exibido no topo de cada página.
- **Paginação** — Todas as listas paginam com 20 registros por página.
- **Busca** — Campo de pesquisa presente em todas as listas principais.

---

## 3. Dashboard

O Dashboard exibe um resumo executivo da operação:

| Card | Descrição |
|------|-----------|
| **Lotes Ativos** | Quantidade de lotes em confinamento |
| **Animais** | Total de animais nos lotes ativos |
| **Peso Médio** | Peso médio atual dos animais |
| **GMD** | Ganho Médio Diário do rebanho |
| **Receita Estimada** | Projeção de receita ao abate |
| **Custo Total** | Soma de todos os custos lançados |

### Gráficos

- **Evolução de Peso** — Linha com pesagens dos últimos meses.
- **Custo por Categoria** — Pizza com distribuição: ração, medicamentos, outros.
- **Preço Mercado** — Cotação do boi gordo (CEPEA) em tempo real.

> Os dados do dashboard são atualizados automaticamente a cada 30 segundos via conexão em tempo real (SignalR).

---

## 4. Lotes

O lote é a unidade central do sistema. Representa um grupo de animais em confinamento.

### Listar Lotes

Acesse **Lotes** no menu lateral. A lista exibe:

- Nome do lote
- Data de entrada
- Número de animais
- Peso médio
- GMD
- Status (Ativo / Finalizado)

Use o campo de **busca** para filtrar por nome. Use os filtros para filtrar por **status** ou **raça**.

### Criar Lote

1. Clique em **Novo Lote**.
2. Preencha os campos:

| Campo | Obrigatório | Descrição |
|-------|-------------|-----------|
| Nome | Sim | Identificação do lote |
| Data Entrada | Sim | Data de início do confinamento |
| Raça | Não | Ex: Nelore, Angus, Brangus |
| Peso Médio Entrada | Sim | Peso médio ao entrar (kg) |
| Qtd. Animais | Sim | Número de cabeças |
| Valor Compra / Arroba | Não | Custo de aquisição |
| Produtor | Não | Vínculo com produtor cadastrado |
| Previsão de Abate | Não | Data estimada de saída |

3. Clique em **Salvar**.

### Detalhe do Lote

Ao clicar em um lote, você acessa suas abas:

| Aba | Conteúdo |
|-----|----------|
| **Resumo** | Indicadores, custos, GMD, projeção de resultado |
| **Animais** | Lista de animais individuais do lote |
| **Pesagens** | Histórico e lançamento de pesagens |
| **Ração** | Consumo diário de ração |
| **Medicamentos** | Aplicações e tratamentos |
| **Suplementos** | Consumo de suplementos minerais |
| **Arrendamentos** | Custo de arrendamento da área |
| **Outros Gastos** | Despesas diversas |
| **Gráficos** | Evolução de peso e custos |

### Encerrar Lote

Na aba **Resumo**, clique em **Finalizar Lote** para registrar o abate. Informe:
- Data de abate
- Peso de abate (arroba)
- Preço de venda (R$/arroba)

O sistema calculará automaticamente o resultado financeiro.

### Exportar Relatório PDF

Em qualquer detalhe de lote, clique no botão **Exportar PDF** para gerar o relatório completo do lote em PDF, incluindo todos os custos, pesagens e resultado.

---

## 5. Animais

Cadastro individual de animais dentro de um lote.

### Acessar

- Via menu **Animais** (visão geral de todos os animais) ou
- Via aba **Animais** dentro de um lote específico.

### Campos do Animal

| Campo | Descrição |
|-------|-----------|
| Identificação | Brinco / número do animal |
| Sexo | M / F |
| Raça | Raça do animal |
| Data Nascimento | Para cálculo da idade |
| Peso Entrada | Peso ao entrar no lote (kg) |
| Lote | Lote vinculado |

---

## 6. Pesagens

Registro de pesagens periódicas para acompanhamento do GMD.

### Lançar Pesagem

1. No detalhe do lote, acesse a aba **Pesagens**.
2. Clique em **Lançar Pesagem**.
3. Informe:
   - Data da pesagem
   - Peso médio atual (kg)
   - Observações (opcional)
4. Clique em **Salvar**.

O sistema calcula automaticamente:
- **GMD** = (Peso atual − Peso anterior) ÷ Dias
- **Arrobas atuais** = Peso médio ÷ 15

### Histórico

A aba exibe todas as pesagens com data, peso, GMD e variação percentual.

---

## 7. Rações

Gerenciamento do catálogo de rações e lançamento de consumo.

### Cadastrar Ração

1. Acesse **Rações → Nova Ração**.
2. Informe nome, fabricante, tipo e custo por kg.
3. Salve.

### Lançar Consumo

Na aba **Ração** do lote:

1. Clique em **Registrar Consumo**.
2. Selecione a ração, informe a data e a quantidade consumida (kg).
3. O sistema calcula o custo automaticamente.

### Consumo por Animal

O sistema exibe o consumo médio por animal por dia (kg/cab/dia) e o custo correspondente.

---

## 8. Estoque de Insumos

Controle de estoque de ração e outros insumos.

### Funcionalidades

- **Entrada de estoque** — Registra compra de insumos com quantidade e valor.
- **Saída** — Vincula ao consumo do lote.
- **Saldo atual** — Exibe o estoque disponível por produto.
- **Alertas de estoque mínimo** — Avisa quando o saldo fica abaixo do mínimo configurado.

---

## 9. Produtores

Cadastro dos produtores proprietários dos animais.

### Criar Produtor

1. Acesse **Produtores → Novo Produtor**.
2. Informe nome, CPF/CNPJ, telefone, e-mail e endereço.
3. Salve.

Os produtores podem ser vinculados a lotes para rastreabilidade de propriedade.

---

## 10. Mercado e Preços

Acompanhamento das cotações em tempo real.

### Cotações Disponíveis

| Indicador | Fonte |
|-----------|-------|
| Boi Gordo (CEPEA) | Scraper CEPEA — atualizado diariamente |
| Futuro Boi Gordo (B3) | Scraper B3 — cotações do contrato atual |
| Bezerro | CEPEA |

### Funcionalidades da Página

- **Gráfico Candlestick** — Visualização histórica do preço futuro B3.
- **Card de Cotação Atual** — Preço spot e variação do dia.
- **Tabela de Contratos Futuros** — Vencimentos disponíveis na B3.
- **Atualização em tempo real** — Os preços são atualizados via WebSocket (SignalR) sem recarregar a página.

---

## 11. Simulador de Hedge

Ferramenta para simulação e análise de operações de hedge no mercado futuro da B3.

### Como Usar

1. Acesse **Simulador** no menu.
2. Informe:
   - Peso estimado de abate (arrobas)
   - Preço de custo (R$/arroba)
   - Margem desejada (%)
   - Cotação futuro atual (preenchido automaticamente)
3. O simulador calcula:
   - **Preço mínimo de venda** para cobrir custos
   - **Resultado com hedge** (travar preço no futuro)
   - **Resultado sem hedge** (venda a mercado)
   - **Recomendação** — O sistema sugere fazer ou não o hedge com base na análise.

### Hedge Decision Card

O card de decisão exibe:
- **Fazer Hedge** (verde) — Quando o preço futuro cobre custos + margem.
- **Aguardar** (amarelo) — Quando a margem está abaixo do mínimo.
- **Não Hedgear** (vermelho) — Quando o mercado está desfavorável.

---

## 12. Financeiro

Visão financeira consolidada da operação.

### Resumo Financeiro

| Indicador | Descrição |
|-----------|-----------|
| Receita Projetada | Valor estimado ao abate |
| Custo Total | Soma de todos os custos lançados |
| Resultado Bruto | Receita − Custo |
| Margem % | Resultado / Receita × 100 |

### Categorias de Custo

- **Compra de animais** — Valor pago pelos bezerros/bois
- **Ração** — Consumo total de ração
- **Medicamentos** — Vacinas e tratamentos
- **Suplementos** — Minerais e vitaminas
- **Arrendamento** — Custo da pastagem/confinamento
- **Outros** — Despesas diversas (frete, energia, etc.)

### Exportação

Clique em **Exportar Excel** ou **Exportar PDF** para baixar o relatório financeiro.

---

## 13. Relatórios

Central de relatórios gerenciais.

### Relatórios Disponíveis

| Relatório | Formato |
|-----------|---------|
| Resumo de Lotes | PDF / Excel |
| Financeiro por Período | PDF / Excel |
| Evolução de Peso | PDF |
| Consumo de Ração | Excel |
| Estoque de Insumos | Excel |
| Desempenho por Raça | PDF |

### Como Gerar

1. Acesse **Relatórios**.
2. Selecione o tipo de relatório.
3. Defina o período ou filtros.
4. Clique em **Gerar PDF** ou **Exportar Excel**.

---

## 14. Agente IA

Assistente inteligente integrado ao ChatGPT/Claude para análise da operação.

### Como Usar

1. Acesse **Agente IA** no menu.
2. Digite sua pergunta em linguagem natural. Exemplos:
   - *"Qual lote tem o melhor GMD este mês?"*
   - *"Qual o custo de ração por arroba produzida no Lote A?"*
   - *"Calcule o resultado financeiro se vender a R$ 280/arroba"*
   - *"Faça um resumo da operação atual"*
3. O agente consulta os dados do seu confinamento e responde com análise contextualizada.

> O agente tem acesso somente aos dados do seu tenant (empresa). Dados de outros clientes nunca são compartilhados.

---

## 15. Alertas de Preço

Configure alertas automáticos para ser notificado quando o preço do boi gordo atingir determinados valores.

### Criar Alerta

1. Acesse **Alertas** no menu.
2. Clique em **Novo Alerta**.
3. Configure:
   - **Tipo**: Preço acima de / Preço abaixo de
   - **Valor**: Preço em R$/arroba
   - **Indicador**: Boi Gordo CEPEA ou Futuro B3
4. Salve.

### Recebimento

Quando o preço atingir o valor configurado:
- Notificação **push** no app mobile
- Notificação **in-app** (sino no header)

---

## 16. Notificações

Central de todas as notificações do sistema.

Acesse clicando no **sino** no header ou pelo menu **Notificações**.

| Tipo | Descrição |
|------|-----------|
| Alerta de Preço | Cotação atingiu valor configurado |
| Estoque Mínimo | Insumo abaixo do mínimo |
| Pesagem Pendente | Lote sem pesagem há mais de X dias |
| Sistema | Comunicados da plataforma |

Clique em uma notificação para ir diretamente ao item relacionado. Use **Marcar todas como lidas** para limpar o contador.

---

## 17. Usuários

Gerenciamento dos usuários da sua empresa. *(Disponível para perfil Admin)*

### Criar Usuário

1. Acesse **Usuários → Novo Usuário**.
2. Informe nome, e-mail, perfil e senha inicial.
3. Salve.

O usuário receberá acesso imediato com as permissões do perfil selecionado.

### Editar Usuário

- Clique no ícone de lápis na lista de usuários.
- Você pode alterar nome, perfil, status (Ativo/Inativo) e senha.

### Desativar Usuário

Para remover o acesso sem excluir o histórico, edite o usuário e mude o status para **Inativo**.

---

## 18. Perfis e Permissões

| Perfil | Acesso |
|--------|--------|
| **SuperAdmin** | Acesso total à plataforma, incluindo gestão de tenants (clientes) |
| **Admin** | Acesso completo à empresa: lotes, financeiro, usuários |
| **Gerente** | Lotes, pesagens, ração, financeiro (sem gestão de usuários) |
| **Funcionário** | Lançamentos operacionais: pesagens, ração, medicamentos |

### Restrições por Perfil

| Funcionalidade | SuperAdmin | Admin | Gerente | Funcionário |
|----------------|:---:|:---:|:---:|:---:|
| Dashboard | ✅ | ✅ | ✅ | ✅ |
| Lotes (CRUD) | ✅ | ✅ | ✅ | ✅ |
| Financeiro | ✅ | ✅ | ✅ | ❌ |
| Relatórios | ✅ | ✅ | ✅ | ❌ |
| Usuários | ✅ | ✅ | ❌ | ❌ |
| Agente IA | ✅ | ✅ | ✅ | ❌ |
| Gestão de Tenants | ✅ | ❌ | ❌ | ❌ |

---

## 19. Aplicativo Mobile

O TepConfina possui aplicativo nativo para Android e iOS.

### Download

- **Android**: Disponível na Google Play Store (em breve)
- **iOS**: Disponível na App Store (em breve)

### Funcionalidades Mobile

- Dashboard com indicadores em tempo real
- Lançamento de pesagens em campo
- Consulta de lotes e animais
- Alertas de preço via push notification
- Consumo de ração
- Funcionamento **offline** — dados sincronizados ao retornar a cobertura de rede

### Login Mobile

Use o mesmo e-mail e senha do sistema web. A sessão é mantida de forma segura no dispositivo.

---

## 20. Perguntas Frequentes

**Como altero minha senha?**
Acesse **Usuários**, encontre seu usuário, clique em **Editar** e use o campo **Nova Senha**.

**O sistema funciona sem internet?**
O app mobile suporta modo offline para consulta e lançamentos. O sistema web requer conexão ativa.

**Como exporto os dados para o Excel?**
Em qualquer lista ou na página de Relatórios, clique em **Exportar Excel**.

**Os dados de outras empresas aparecem para mim?**
Não. O sistema é multi-tenant: cada empresa (tenant) visualiza exclusivamente seus próprios dados.

**O que é GMD?**
Ganho Médio Diário — indica quantos kg o animal ganhou por dia no período. Calculado como:
`GMD = (Peso Final − Peso Inicial) ÷ Número de Dias`

**O que é uma arroba?**
Unidade de medida de peso para bovinos: **1 arroba = 15 kg** de peso vivo.

**Como funciona o hedge?**
O hedge é uma operação no mercado futuro da B3 para travar o preço de venda do boi gordo. O simulador calcula se vale a pena travar o preço com base nos seus custos e margem desejada.

**Esqueci minha senha, como recupero?**
Entre em contato com o administrador da sua empresa (perfil Admin) para redefinição. Se o Admin esquecer a senha, contate o suporte Tecnoepec.

---

## Suporte

| Canal | Contato |
|-------|---------|
| E-mail | suporte@tecnoepec.com.br |
| Site | tecnoepec.com.br |

---

---

## 19. Protocolo Sanitário

O calendário sanitário permite agendar e rastrear vacinas, vermífugos, vitaminas e outros procedimentos veterinários por lote.

**Acessar:** dentro do detalhe do lote, clique na aba **Sanitário**.

### Criar protocolo

1. Clique em **Novo Protocolo**
2. Informe:
   - **Produto** — nome do medicamento (ex: Aftosa, Ivermectina)
   - **Tipo** — Vacina, Vermifugo, Vitamina, Antibiotico ou Outro
   - **Data Prevista** — quando o procedimento deve ser realizado
   - **Dose (mL)** e **Custo por animal** — opcionais, usados para controle financeiro
3. Clique em **Salvar**

### Marcar como realizado

1. Clique em **Realizar** no protocolo desejado
2. Informe a data de realização e confirme

Protocolos com data vencida e não realizados são destacados em vermelho como alerta.

---

## 20. Importação via Excel

Permite importar pesagens em massa sem digitar linha a linha.

**Acesse:** menu **Importação** na barra lateral.

### Passo a passo

1. **Baixar template** — clique em "Baixar template de pesagens" para obter o modelo `.xlsx`
2. **Preencher** — abra o arquivo e preencha as colunas:
   - `LoteNome` — nome exato do lote ativo
   - `Data` — no formato `YYYY-MM-DD` (ex: `2025-06-01`)
   - `PesoMedio_kg` — peso médio em kg (ex: `420.5`)
   - `QuantidadeAnimais` — opcional; se omitido, usa a quantidade cadastrada no lote
   - `Observacoes` — opcional
3. **Selecionar arquivo** — clique na área de upload e escolha o arquivo `.xlsx` preenchido
4. **Importar** — clique em "Iniciar importação"

O sistema exibe quantas pesagens foram importadas e lista as linhas com erro (se houver), sem interromper as demais.

!!! warning "Atenção"
    Verifique os dados antes de importar. O sistema não detecta duplicatas automaticamente.

---

## 21. Auditoria

O log de auditoria registra todas as ações de escrita realizadas no sistema (criação, edição e exclusão).

**Acesse:** menu **Auditoria** (visível apenas para Admin e SuperAdmin).

**Filtros disponíveis:**

- **Método** — POST, PUT, PATCH ou DELETE
- **Usuário** — filtrar por e-mail do usuário
- **Período** — intervalo de datas

Cada entrada exibe: método, rota acessada, código de resposta, usuário, IP de origem, duração e horário.

!!! note "Isolamento"
    Admins veem apenas os logs da sua empresa. SuperAdmins veem os logs de todos os tenants.

---

*TepConfina — Gestão Inteligente de Confinamento Bovino*
*© 2025 Tecnoepec. Todos os direitos reservados.*
