# Entidades do Dominio

Descricao completa das entidades do sistema TepConfina.

## BaseEntity

Classe base herdada por todas as entidades do sistema.

| Campo       | Tipo           | Descricao                                    |
|-------------|----------------|----------------------------------------------|
| Id          | `Guid`         | Identificador unico da entidade              |
| CreatedAt   | `DateTime`     | Data de criacao do registro                  |
| UpdatedAt   | `DateTime?`    | Data da ultima atualizacao                   |
| IsDeleted   | `bool`         | Flag de soft delete (exclusao logica)        |
| CreatedBy   | `string?`      | Usuario que criou o registro                 |
| UpdatedBy   | `string?`      | Usuario que fez a ultima atualizacao         |

!!! info "Soft Delete"
    O sistema utiliza exclusao logica via `IsDeleted`. Registros nunca sao removidos fisicamente do banco. Um global query filter no EF Core garante que registros deletados nao aparecem em consultas.

## Tenant

Representa uma organizacao/fazenda no sistema multi-tenant.

| Campo | Tipo     | Descricao                                |
|-------|----------|------------------------------------------|
| Nome  | `string` | Nome da organizacao                      |
| Slug  | `string` | Identificador unico amigavel (URL-safe)  |

## Lote

Entidade central do sistema. Representa um lote de animais em confinamento.

| Campo              | Tipo       | Descricao                                    |
|--------------------|------------|----------------------------------------------|
| Nome               | `string`   | Nome identificador do lote                   |
| ProdutorId         | `Guid`     | Referencia ao produtor                       |
| Propriedade        | `string`   | Nome da propriedade rural                    |
| Cidade             | `string`   | Cidade da propriedade                        |
| Frigorifico        | `string`   | Frigorifico de destino                       |
| Raca               | `string`   | Raca predominante dos animais                |
| DataEntrada        | `DateTime` | Data de entrada dos animais                  |
| DataPrevistaSaida  | `DateTime` | Data prevista para saida/abate               |
| QuantidadeAnimais  | `int`      | Quantidade total de animais                  |
| PesoBrutoEntrada   | `decimal`  | Peso bruto total na entrada (kg)             |
| PesoLiquidoEntrada | `decimal`  | Peso liquido total na entrada (kg)           |
| PesoMedioEntrada   | `decimal`  | Peso medio por animal na entrada (kg)        |
| ValorCabeca        | `decimal`  | Valor pago por cabeca (R$)                   |
| ValorArroba        | `decimal`  | Valor de referencia da arroba (R$)           |
| Status             | `enum`     | Ativo ou Fechado                             |
| PesoSaida          | `decimal?` | Peso total na saida (kg) - ao fechar lote    |
| RendimentoCarcaca  | `decimal?` | Percentual de rendimento de carcaca          |
| GanhoArroba        | `decimal?` | Ganho em arrobas - calculado ao fechar       |
| DataSaida          | `DateTime?`| Data efetiva de saida/abate                  |

## Animal

Representa um animal individual dentro de um lote.

| Campo          | Tipo       | Descricao                                   |
|----------------|------------|---------------------------------------------|
| Identificacao  | `string`   | Numero de identificacao (brinco/chip)       |
| LoteId         | `Guid`     | Referencia ao lote                          |
| Raca           | `string`   | Raca do animal                              |
| Sexo           | `string`   | Sexo do animal (Macho/Femea)                |
| DataNascimento | `DateTime?`| Data de nascimento estimada                 |
| PesoEntrada    | `decimal`  | Peso na entrada do confinamento (kg)        |
| DataEntrada    | `DateTime` | Data de entrada no confinamento             |
| Status         | `enum`     | Ativo, Morto ou Vendido                     |
| DataMorte      | `DateTime?`| Data da morte (se aplicavel)                |
| CausaMorte     | `string?`  | Causa da morte                              |
| DataVenda      | `DateTime?`| Data da venda/abate                         |
| PesoVenda      | `decimal?` | Peso na venda (kg)                          |
| ValorVenda     | `decimal?` | Valor recebido na venda (R$)                |

## Pesagem

Registro de pesagem coletiva de um lote.

| Campo             | Tipo       | Descricao                               |
|-------------------|------------|-----------------------------------------|
| LoteId            | `Guid`     | Referencia ao lote                      |
| DataPesagem       | `DateTime` | Data da pesagem                         |
| PesoTotal         | `decimal`  | Peso total dos animais pesados (kg)     |
| QuantidadeAnimais | `int`      | Quantidade de animais pesados           |
| PesoMedio         | `decimal`  | Peso medio por animal (kg)              |
| GMD               | `decimal`  | Ganho medio diario calculado (kg/dia)   |
| Observacoes       | `string?`  | Observacoes sobre a pesagem             |

## PesagemAnimal

Registro de pesagem individual de um animal.

| Campo       | Tipo       | Descricao                               |
|-------------|------------|-----------------------------------------|
| AnimalId    | `Guid`     | Referencia ao animal                    |
| DataPesagem | `DateTime` | Data da pesagem                         |
| Peso        | `decimal`  | Peso do animal (kg)                     |
| GMD         | `decimal`  | Ganho medio diario individual (kg/dia)  |
| Observacoes | `string?`  | Observacoes sobre a pesagem             |

## Racao

Registro de racao comprada e estoque.

| Campo         | Tipo       | Descricao                               |
|---------------|------------|-----------------------------------------|
| Nome          | `string`   | Nome/tipo da racao                      |
| Fornecedor    | `string`   | Nome do fornecedor                      |
| DataCompra    | `DateTime` | Data da compra                          |
| QuantidadeKg  | `decimal`  | Quantidade comprada (kg)                |
| PrecoUnitario | `decimal`  | Preco por kg (R$)                       |
| ValorTotal    | `decimal`  | Valor total da compra (R$)              |
| EstoqueAtualKg| `decimal`  | Estoque disponivel atual (kg)           |

## ConsumoRacao

Registro de consumo diario de racao por lote.

| Campo           | Tipo       | Descricao                              |
|-----------------|------------|----------------------------------------|
| LoteId          | `Guid`     | Referencia ao lote                     |
| RacaoId         | `Guid`     | Referencia a racao utilizada           |
| DataConsumo     | `DateTime` | Data do consumo                        |
| QuantidadeKg    | `decimal`  | Quantidade consumida (kg)              |
| QuantidadeAnimais| `int`     | Quantidade de animais que consumiram   |
| ConsumoPerCapita| `decimal`  | Consumo medio por animal (kg)          |

## AplicacaoMedicamento

Registro de aplicacao de medicamentos/tratamentos.

| Campo             | Tipo       | Descricao                              |
|-------------------|------------|----------------------------------------|
| LoteId            | `Guid`     | Referencia ao lote                     |
| Nome              | `string`   | Nome do medicamento                    |
| Fornecedor        | `string`   | Fornecedor do medicamento              |
| QuantidadeAnimais | `int`      | Animais que receberam o medicamento    |
| ValorUnitario     | `decimal`  | Valor por dose (R$)                    |
| ValorTotal        | `decimal`  | Valor total gasto (R$)                 |
| DataAplicacao     | `DateTime` | Data da aplicacao                      |

## Produtor

Cadastro de produtores rurais.

| Campo     | Tipo     | Descricao                                |
|-----------|----------|------------------------------------------|
| Nome      | `string` | Nome completo do produtor                |
| CPF_CNPJ  | `string` | CPF ou CNPJ do produtor                  |
| Telefone  | `string` | Telefone de contato                      |
| Email     | `string` | Email de contato                         |
| Endereco  | `string` | Endereco completo                        |
| Cidade    | `string` | Cidade                                   |
| Estado    | `string` | UF do estado                             |

## CompraLote

Múltiplas compras vinculadas a um mesmo lote, com rateio proporcional do custo por animal. Permite que um lote seja formado por várias entradas em datas/preços diferentes.

| Campo            | Tipo       | Descricao                                 |
|------------------|------------|-------------------------------------------|
| LoteId           | `Guid`     | Referência ao lote                        |
| DataCompra       | `DateTime` | Data da compra                            |
| QuantidadeAnimais| `int`      | Animais comprados nesta operação         |
| PesoTotal        | `decimal`  | Peso total dos animais comprados (kg)     |
| ValorTotal       | `decimal`  | Valor total pago (R$)                     |
| ValorCabeca      | `decimal`  | Valor por cabeça (R$)                     |
| ValorArroba      | `decimal`  | Valor por arroba (R$)                     |
| Vendedor         | `string?`  | Nome do vendedor                          |
| Frete            | `decimal?` | Valor do frete (R$)                       |
| Observacoes      | `string?`  | Observações                               |

## CustoAnimal

Custo individual rateado por animal a partir das CompraLotes do lote (proporcional ao peso de entrada). Permite calcular margem real de cada animal vendido.

| Campo               | Tipo      | Descricao                              |
|---------------------|-----------|----------------------------------------|
| AnimalId            | `Guid`    | Referência ao animal                   |
| LoteId              | `Guid`    | Referência ao lote                     |
| CustoCompra         | `decimal` | Custo de aquisição rateado (R$)        |
| CustoOperacionalAcc | `decimal` | Acumulado de ração+med+suplementos+etc |

## PrecoMercado

Tabela de cotações **legacy** que alimenta os cards de preço do dashboard (Preço atual, Média 7d, Média 30d). 1 registro por dia/fonte.

| Campo               | Tipo       | Descricao                                  |
|---------------------|------------|--------------------------------------------|
| Fonte               | `string`   | `TRADINGVIEW`, `BRAPI`, `MANUAL` (legacy: `CEPEA`, `B3`) |
| Indice              | `string`   | Símbolo do contrato (`BGI1!`, `BBOI11`, etc) |
| PrecoArroba         | `decimal`  | Preço (R$) — semântica varia por fonte    |
| Data                | `DateTime` | Data da cotação                            |
| Variacao            | `decimal?` | Variação absoluta vs. anterior             |
| VariacaoPercentual  | `decimal?` | Variação percentual                        |

!!! warning "Semântica do PrecoArroba"
    Quando `Fonte=TRADINGVIEW`/`Indice=BGI1!`, o valor é o preço **real da arroba** (~R$ 360). Quando `Fonte=BRAPI`/`Indice=BBOI11`, é a cota do **ETF** (~R$ 11), não a arroba. O frontend trata isso e exibe label apropriado.

## MarketContract

Contratos disponíveis no provider ativo. Sincronizado a cada ciclo do `MarketIngestionHostedService`.

| Campo          | Tipo        | Descricao                                |
|----------------|-------------|------------------------------------------|
| Symbol         | `string`    | Símbolo genérico (`BOI`)                 |
| ContractCode   | `string`    | Código do contrato (`BGI1!`, `BGIK26`)   |
| Description    | `string?`   | Descrição amigável                       |
| ExpirationDate | `DateTime?` | Vencimento (null para contratos contínuos) |
| IsActive       | `bool`      | Se está sendo coletado                   |

## MarketQuote

Cotação atual por contrato. Provider-agnóstico (campo `Source` indica origem).

| Campo            | Tipo       | Descricao                                   |
|------------------|------------|---------------------------------------------|
| Symbol           | `string`   | Símbolo (`BOI`)                             |
| ContractCode     | `string?`  | Código do contrato                          |
| Price            | `decimal`  | Último preço                                |
| OpenPrice        | `decimal?` | Abertura do dia                             |
| HighPrice        | `decimal?` | Máxima do dia                               |
| LowPrice         | `decimal?` | Mínima do dia                               |
| VariationPercent | `decimal?` | Variação % vs. fechamento anterior          |
| Volume           | `decimal?` | Volume                                      |
| Source           | `string`   | `tradingview`, `brapi`, `stonex`, `mock`    |
| QuoteTimestamp   | `DateTime` | Timestamp da coleta                         |

## MarketCandle

Candle OHLC para gráfico. Chave única: `Symbol + ContractCode + Timeframe + CandleTime`.

| Campo        | Tipo       | Descricao                                  |
|--------------|------------|--------------------------------------------|
| Symbol       | `string`   | Símbolo (`BOI`)                            |
| ContractCode | `string?`  | Código do contrato                         |
| Timeframe    | `string`   | `1m`, `5m`, `15m`, `1h`, `4h`, `1d`, `1w` |
| Open         | `decimal`  | Abertura                                   |
| High         | `decimal`  | Máxima                                     |
| Low          | `decimal`  | Mínima                                     |
| Close        | `decimal`  | Fechamento                                 |
| Volume       | `decimal?` | Volume                                     |
| Source       | `string`   | Provider que coletou                       |
| CandleTime   | `DateTime` | Início do candle                           |

## User

Usuarios do sistema.

| Campo         | Tipo     | Descricao                                |
|---------------|----------|------------------------------------------|
| Nome          | `string` | Nome completo do usuario                 |
| Email         | `string` | Email (usado para login)                 |
| PasswordHash  | `string` | Hash BCrypt da senha                     |
| Role          | `string` | Papel do usuario (Admin, User)           |
| IsActive      | `bool`   | Se o usuario esta ativo                  |
| TenantId      | `Guid`   | Referencia ao tenant                     |
| RefreshToken  | `string?`| Token de refresh JWT atual               |

## AlertaPreco

Alertas de preco configurados pelo usuario.

| Campo      | Tipo      | Descricao                                |
|------------|-----------|------------------------------------------|
| UserId     | `Guid`    | Referencia ao usuario                    |
| Produto    | `string`  | Produto monitorado (Boi Gordo, etc)      |
| PrecoAlvo  | `decimal` | Preco que dispara o alerta (R$/@)        |
| Tipo       | `enum`    | Acima ou Abaixo do preco alvo            |
| Ativo      | `bool`    | Se o alerta esta ativo                   |

## Notificacao

Notificacoes enviadas aos usuarios.

| Campo     | Tipo       | Descricao                                |
|-----------|------------|------------------------------------------|
| UserId    | `Guid`     | Referencia ao usuario                    |
| Titulo    | `string`   | Titulo da notificacao                    |
| Mensagem  | `string`   | Conteudo da notificacao                  |
| Tipo      | `string`   | Tipo (Alerta, Info, Aviso)               |
| Lida      | `bool`     | Se a notificacao foi lida                |
| DataEnvio | `DateTime` | Data e hora do envio                     |

## AuditLog

Registro de auditoria de todas as operacoes.

| Campo      | Tipo       | Descricao                                |
|------------|------------|------------------------------------------|
| UserId     | `Guid`     | Usuario que realizou a acao              |
| EntityName | `string`   | Nome da entidade afetada                 |
| EntityId   | `Guid`     | ID da entidade afetada                   |
| Action     | `string`   | Acao realizada (Create, Update, Delete)  |
| OldValues  | `string?`  | Valores anteriores em JSON               |
| NewValues  | `string?`  | Novos valores em JSON                    |
| Timestamp  | `DateTime` | Data e hora da operacao                  |

## DeviceToken

Tokens de dispositivos moveis para push notifications.

| Campo    | Tipo     | Descricao                                |
|----------|----------|------------------------------------------|
| UserId   | `Guid`   | Referencia ao usuario                    |
| Token    | `string` | Token FCM do dispositivo                 |
| Platform | `string` | Plataforma (Android, iOS)                |

---

## Entidades de operação ampliada

### Suplemento e ConsumoSuplemento

Suplementos minerais/vitamínicos cadastrados como insumos separados da ração principal.

**Suplemento**

| Campo       | Tipo      | Descricao                                |
|-------------|-----------|------------------------------------------|
| Nome        | `string`  | Nome do suplemento                       |
| Tipo        | `string`  | Mineral, Vitamina, Aditivo               |
| Fornecedor  | `string?` | Fornecedor                               |
| PrecoUnitarioKg | `decimal` | Custo por kg (R$)                    |

**ConsumoSuplemento**

| Campo        | Tipo       | Descricao                              |
|--------------|------------|----------------------------------------|
| LoteId       | `Guid`     | Lote                                   |
| SuplementoId | `Guid`     | Suplemento                             |
| DataConsumo  | `DateTime` | Data                                   |
| QuantidadeKg | `decimal`  | Quantidade consumida                   |

### Arrendamento

Custo de pastagem ou área de confinamento arrendada.

| Campo           | Tipo       | Descricao                              |
|-----------------|------------|----------------------------------------|
| LoteId          | `Guid`     | Lote                                   |
| Proprietario    | `string`   | Nome do proprietário                   |
| DataInicio      | `DateTime` | Início do contrato                     |
| DataFim         | `DateTime` | Fim do contrato                        |
| ValorMensal     | `decimal`  | Valor mensal pago (R$)                 |
| ValorTotal      | `decimal`  | Valor total acumulado (R$)             |
| Observacoes     | `string?`  | Detalhes do contrato                   |

### EstoqueInsumo

Controle de estoque para ração, suplementos e medicamentos.

| Campo        | Tipo       | Descricao                              |
|--------------|------------|----------------------------------------|
| Nome         | `string`   | Nome do insumo                         |
| Categoria    | `string`   | `Racao`, `Suplemento`, `Medicamento`   |
| EstoqueAtual | `decimal`  | Quantidade atual em estoque            |
| Unidade      | `string`   | `kg`, `g`, `un`, `L`                   |
| EstoqueMinimo| `decimal?` | Disparo de alerta quando abaixo        |
| PrecoUltimaCompra | `decimal?` | Custo de referência               |

### OutrosGastos

Despesas operacionais diversas (frete, energia, mão de obra, etc).

| Campo       | Tipo       | Descricao                              |
|-------------|------------|----------------------------------------|
| LoteId      | `Guid?`    | Lote (null para custo geral do tenant) |
| Categoria   | `string`   | Frete, Energia, MaoDeObra, etc         |
| Descricao   | `string`   | Descrição da despesa                   |
| Valor       | `decimal`  | Valor (R$)                             |
| DataDespesa | `DateTime` | Data                                   |

### ProtocoloSanitario e AplicacaoProtocolo

Calendário sanitário recorrente por lote (ex: vacinação a cada 60 dias).

**ProtocoloSanitario**

| Campo               | Tipo      | Descricao                              |
|---------------------|-----------|----------------------------------------|
| Nome                | `string`  | Nome do protocolo                      |
| Tipo                | `string`  | Vacina, Vermífugo, Mineral, etc        |
| FrequenciaDias      | `int`     | A cada N dias                          |
| DiasAposEntrada     | `int?`    | Primeira aplicação após entrada        |

**AplicacaoProtocolo**

| Campo          | Tipo       | Descricao                              |
|----------------|------------|----------------------------------------|
| LoteId         | `Guid`     | Lote                                   |
| ProtocoloId    | `Guid`     | Protocolo aplicado                     |
| DataAplicacao  | `DateTime` | Data efetiva                           |
| QuantidadeAnimais | `int`   | Animais que receberam                  |
| ValorTotal     | `decimal`  | Custo total (R$)                       |
| Observacoes    | `string?`  | Reações, lote do produto, etc          |

### ConferenciaVisual

Resultado de análise de imagem do lote via Claude Vision (BCS — Body Condition Score) e detector TF-Lite (contagem de cabeças).

| Campo              | Tipo       | Descricao                              |
|--------------------|------------|----------------------------------------|
| LoteId             | `Guid`     | Lote                                   |
| ImagemUrl          | `string`   | URL da imagem no S3                    |
| BcsMedio           | `decimal?` | Body Condition Score 1–9 (Claude)      |
| CabecasDetectadas  | `int?`     | Contagem TF-Lite                       |
| ResumoIA           | `string?`  | Texto descritivo gerado                |
| DataAnalise        | `DateTime` | Quando foi processada                  |

### NegocioReportado (Mercado Colaborativo)

Negociações reportadas pelos próprios produtores (crowdsourced) que alimentam o mapa de preços por UF e a referência de margem real.

| Campo            | Tipo       | Descricao                              |
|------------------|------------|----------------------------------------|
| TenantId         | `Guid`     | Quem reportou                          |
| DataNegocio      | `DateTime` | Data da venda                          |
| UF               | `string`   | Estado (2 letras)                      |
| Cidade           | `string`   | Cidade                                 |
| QuantidadeAnimais| `int`      | Animais vendidos                       |
| PrecoArroba      | `decimal`  | Preço/@ negociado (R$)                 |
| Frigorifico      | `string?`  | Frigorífico/comprador                  |
| Observacoes      | `string?`  | Detalhes (raça, peso médio, etc)       |

### HedgeOperacao

Operações de hedge (vendas de contratos BGI) registradas pelo usuário para acompanhamento de P&L.

| Campo          | Tipo       | Descricao                              |
|----------------|------------|----------------------------------------|
| LoteId         | `Guid?`    | Lote associado (opcional)              |
| DataOperacao   | `DateTime` | Data da venda do contrato              |
| ContractCode   | `string`   | Contrato vendido (`BGIK26` etc)        |
| Contratos      | `int`      | Quantos contratos vendidos             |
| PrecoTrava     | `decimal`  | Preço de venda do contrato (R$/@)      |
| Observacoes    | `string?`  | Estratégia, modo, etc                  |

### ImportacaoExcel

Histórico de importações de planilha (pesagens, animais, lotes históricos).

| Campo       | Tipo       | Descricao                              |
|-------------|------------|----------------------------------------|
| TipoEntidade| `string`   | `Pesagem`, `Animal`, `LoteHistorico`   |
| ArquivoNome | `string`   | Nome do arquivo enviado                |
| Linhas      | `int`      | Linhas totais processadas              |
| Sucesso     | `int`      | Linhas importadas com sucesso          |
| Erros       | `int`      | Linhas que falharam                    |
| LogJson     | `string?`  | Detalhamento dos erros (JSON)          |
| UserId      | `Guid`     | Quem importou                          |
| DataImportacao | `DateTime` | Quando                              |
