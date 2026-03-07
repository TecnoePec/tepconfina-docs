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

## PrecoMercado

Cotacoes de mercado de commodities pecuarias.

| Campo    | Tipo       | Descricao                                |
|----------|------------|------------------------------------------|
| Fonte    | `string`   | Fonte da cotacao (CEPEA, B3)             |
| Produto  | `string`   | Tipo de produto (Boi Gordo, Bezerro)     |
| Preco    | `decimal`  | Preco da arroba (R$)                     |
| Data     | `DateTime` | Data da cotacao                          |
| Variacao | `decimal`  | Variacao percentual em relacao ao anterior|

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
