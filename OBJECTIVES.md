# Objetivos

## Sumário
- [Objetivos](#objetivos)
  - [Sumário](#sumário)
    - [1. Objetivos](#1-objetivos)
    - [2. Autenticação e autorização](#2-autenticação-e-autorização)
    - [3. Ambiente de execução e falhas](#3-ambiente-de-execução-e-falhas)
    - [4. Stack](#4-stack)
    - [5. Garantias obrigatórias](#5-garantias-obrigatórias)
    - [6. Modelo de domínio](#6-modelo-de-domínio)
    - [6.1 Modelo de Money](#61-modelo-de-money)
    - [6.2 Modelo de Wallet](#62-modelo-de-wallet)
    - [6.3 WagerTransaction](#63-wagertransaction)
    - [6.4. WalletLedgerEntry](#64-walletledgerentry)
    - [6.5. Inbox e outbox](#65-inbox-e-outbox)
    - [7. Operações e refêrencias](#7-operações-e-refêrencias)
    - [8. Concorrência](#8-concorrência)
    - [9. Contratos HTTP](#9-contratos-http)
      - [9.1. Abertura de carteira](#91-abertura-de-carteira)
      - [9.2. Leitura](#92-leitura)
      - [9.3. Envio de operação](#93-envio-de-operação)
      - [9.4. Reconciliação](#94-reconciliação)
      - [9.5. Health checks](#95-health-checks)
    - [10. Consumidor SQS](#10-consumidor-sqs)
      - [10.1. Filas e mensagens](#101-filas-e-mensagens)
      - [10.2. Processamento e recuperação](#102-processamento-e-recuperação)
    - [11. Publicação com transactional outbox](#11-publicação-com-transactional-outbox)
      - [11.1. Atomicidade e worker](#111-atomicidade-e-worker)
      - [11.2. Eventos](#112-eventos)
    - [12. Observabilidade](#12-observabilidade)
      - [12.1. Logs e métricas](#121-logs-e-métricas)
    - [13. Verificação obrigatória](#13-verificação-obrigatória)
      - [13.1. Testes unitários](#131-testes-unitários)
      - [13.2. Testes de integração](#132-testes-de-integração)
      - [13.3. Autenticação e autorização](#133-autenticação-e-autorização)
      - [13.4. Concorrência e recuperação](#134-concorrência-e-recuperação)
    - [14. Critérios de avaliação](#14-critérios-de-avaliação)
    - [15. Entrega](#15-entrega)
      - [15.1. Artefatos e execução](#151-artefatos-e-execução)
      - [15.2. Arquitetura](#152-arquitetura)
      - [15.3. Comandos e testes](#153-comandos-e-testes)

A seguir, seguem os objetivos descritos no desafio e sua complitude até o momento, ordenados pelo momento em que aparecem no texto base:

### 1. Objetivos

- [ ] API HTTP externa.
- [ ] Um consumidor de mensagens que movimentem carteiras de jogadores com garantias equivalentes.
- [ ] Demonstrar que o resultado financeiro continua correto no caso onde existam várias instâncias em execução e falhas entre as etapas do processamento.

> <br/>
> <i>"A avaliação considera precisão monetária, integridade do ledger, idempotência persistente, concorrência, recuperação de falhas e decisões de arquitetura."</i>
> <div style="text-align: end; font-style: italic;">— Desafio Backend</div>

<br/>

<div style="text-align: end;"><a href="#sumário">Voltar para o sumário</a></div>

### 2. Autenticação e autorização

- [ ] Autenticação e autorização, com integração a um IdP externo OAuth 2.0/OIDC.
- [ ] Justificar a escolha do IdP, a validação de credenciais e o modelo de permissões em ARCHITECTURE.md. 
- [ ] A identidade autenticada deve determinar o providerId autorizado. 
- [ ] Provedores devem acessar apenas suas próprias transações, inclusive em replays.
- [ ] Operações de carteira restritas ao serviço interno.
- [ ] O acesso à mensageria deve ser controlado por credenciais e políticas do broker.
- [ ] Preservar as validações de domínio no consumidor.

> <br/>
> <i>"Cadastro de senhas e emissão própria de tokens estão fora do escopo."</i>
> <div style="text-align: end; font-style: italic;">— Desafio Backend</div>

<br/>

<div style="text-align: end;"><a href="#sumário">Voltar para o sumário</a></div>

### 3. Ambiente de execução e falhas

- [ ] Deve conter os tipos externos: `BET`, `WIN`, `LOSS`, `REFUND` e `ROLLBACK`.
- [ ] Entregas **at-least-once** com soluções para:
    - [ ] Recebimento repetido de uma mesma operação, inclusive por HTTP e SQS.
    - [ ] Chegada de uma reversão antes da transação que ela referencia.
    - [ ] Processamento simultâneo de operações da mesma carteira.
    - [ ] Encerramento abrupto antes ou depois de um commit.
    - [ ] Publicação repetida de um evento de integração.
    - [ ] Indisponibilidade temporária do PostgreSQL ou do SQS.

> <br/>
> <i>"Nenhuma dessas situações pode gerar movimentação duplicada, saldo negativo ou perda de um evento cujo registro foi confirmado no banco."</i>
> <div style="text-align: end; font-style: italic;">— Desafio Backend</div>

<br/>

<div style="text-align: end;"><a href="#sumário">Voltar para o sumário</a></div>

### 4. Stack

- [ ] Usar linguagem de programação Golang, declarando a versão no `go.mod` e no _Dockerfile_.
- [ ] Utilizar os modulos da linguagem Golang, com `go.mod` e `go.sum` versionados.
- [ ] Usar o framework _Uber Fx_.
  - [ ] Para composição de configuração.
  - [ ] Para conexões.
  - [ ] Para repositórios.
  - [ ] Para casos de uso.
  - [ ] Para handlers e workers.
  - [ ] Para injeção por construtores.
  - [ ] Com organização por `fx.Module`, `fx.Provide` e `fx.Invoke`.
  - [ ] Utilizando `fx.Lifecycle`
    - [ ] Na inicialização com validação de configuração e dependências.
    - [ ] Nos cancelamentos.
    - [ ] Nos prazos de execução e término observável dos workers.
    - [ ] No shutdown com interrupção de novas entradas e conclusão ou liberação do trabalho em andamento.
    - [ ] No fechamento das dependências após a finalização dos componentes que as utilizam.
- [ ] Utilizar um roteador de escolha livre na linguagem Golang ou o pacote padrão http.
  - [ ] Documentar em `ARCHITECTURE.md` a biblioteca escolhida.
- [ ] Documentar em `ARCHITECTURE.md` a mapeamento de `Money`.
- [ ] Documentar em `ARCHITECTURE.md` a delimitação da transação SQL entre os repositórios.
- [ ] Utilizar autenticação OAuth 2.0/OIDC.
- [ ] Utilizar PostgreSQL.
- [ ] Utilizar `pgx` preferencialmente para conexão com banco de dados.
- [ ] Utilizar AWS SQS localmente com LocalStack ou MiniStack.
- [ ] Utilizar Docker Compose.
- [ ] Utilizar Migrações Versionadas, com aplicação e reversão documentadas.
- [ ] Utilizar o pacote de teste padrão da linguagem Golang com flag -race, implicitamente, preparar testes.

> <br/>
> <i>"O domínio deve permanecer independente de Fx, HTTP, SQS e bibliotecas de persistência. A organização dos pacotes fica a critério do candidato."</i>
> <div style="text-align: end; font-style: italic;">— Desafio Backend</div>
<br/>

### 5. Garantias obrigatórias

- [ ] Dinheiro não pode passar por float32 ou float64, nem durante parsing, cálculo, serialização ou persistência.
- [ ] Idempotência deve ser persistente e sobreviver ao reinício de todos os processos.
- [ ] As invariantes financeiras devem ser garantidas no banco, independentemente de locks locais e da deduplicação do SQS FIFO.
- [ ] Eventos externos publicados apenas após a confirmação da transação que os originou.
- [ ] Ledger append-only: correções financeiras exigem novos lançamentos.
- [ ] Carteiras independentes avançam em paralelo.
- [ ] Não utilizar locks globais.
- [ ] Atualizações de saldo não impedem lost updates.
- [ ] Unicidade, não negatividade e imutabilidade do ledger impostas pelo schema, pelas constraints e pelos mecanismos de proteção do banco.

### 6. Modelo de domínio

- [ ] Entidades com estado encapsulado.
- [ ] Construtores com validação.
- [ ] Métodos explícitos de transição.
- [ ] Invariantes preservadas em todas as operações públicas.
- [ ] Criação separada de reidratação.
- [ ] Reidratação nã reaplica movimentações, transições ou emissão de eventos.
- [ ] Rejeitar valores de domínio não inicializados ou inválidos.
- [ ] Erros de domínio classificáveis por tipo ou `errors.Is`/`errors.As`.
- [ ] `panic` não representa rejeições de negócio.
- [ ] Operações de I/O respeitam cancelamento e timeout, fazendo questão de utilizar `context.Context`.

### 6.1 Modelo de Money

- [ ] Utilizar `int64` em unidades mínimas ou uma biblioteca decimal de precisão exata. 
  - [ ] Documentar a representação e seus limites.    
  - [ ] O contrato externo recebe e devolve valores como {"amount":"25.00","currency":"BRL"}.
  - [ ] Use escala fixa de duas casas e código de moeda ISO 4217.
  - [ ] Rejeite valores vazios, NaN, Infinity, notação científica, escala excedente e valores negativos nas entradas financeiras externas.
  - [ ] Não arredonde silenciosamente uma entrada inválida. Caso aceite formas equivalentes, documente a normalização anterior ao hash de idempotência.
  - [ ] Aritmética e comparação de valores monetários exigem moedas compatíveis.
  - [ ] Se utilizar int64, trate overflow no parsing, na soma, na subtração e na negação.
  - [ ] Valores negativos são permitidos em diferenças e cálculos internos, mas não no saldo da carteira.
  - [ ] A persistência deve preservar exatamente valor e moeda, por exemplo com unidades mínimas em BIGINT ou decimal em NUMERIC.

> <br/>
> <i>"É permitido operar apenas em BRL nos cenários principais, desde que o tipo carregue a moeda e existam testes de incompatibilidade entre moedas."</i>
> <div style="text-align: end; font-style: italic;">— Desafio Backend</div>
<br/>

### 6.2 Modelo de Wallet

- [ ] Deve carregar `identidade`, `jogador`, `moeda`, `saldo`, `versão` e instantes de criação e atualização (na forma de `createdAt`, `updatedAt` e `deletedAt`).
- [ ] Expor criação, reidratação e operações de débito/crédito, mantendo a alteração do saldo sob controle do agregado e da transação SQL. 
  - Subentende-se que a instância da entidade possa ser criada, reidratada e sofrer manipulações nos dados de saldo, sendo a persistência dessas mudanças nos dados de saldo controladas por transações SQL. 
- [ ] O par (playerId, currency) identifica uma única carteira.
- [ ] Débitos preservam saldo maior ou igual a zero.
- [ ] A moeda de cada movimentação coincide com a da carteira.
- [ ] Cada mudança financeira exige o lançamento correspondente no ledger, confirmado junto com o saldo.
- [ ] A versão inicial é 1; 
  - [ ] Após da criação, é feita o incremento apenas quando há mudança de saldo.
- [ ] Disputas entre escritores não descartam uma atualização confirmada.
- [ ] A estratégia de controle de concorrência está documentada.

### 6.3 WagerTransaction

- [ ] Contem os tipos `OPENING`, `BET`, `WIN`, `LOSS`, `REFUND` e `ROLLBACK`.
- [ ] Registra as seguintes informações para transações externas:
  - [ ] Timestamps (na forma de `createdAt`, `updatedAt` e `deletedAt`).
  - [ ] Estado (na forma de `status`).
  - [ ] Referência externa (opcional).
  - [ ] Identificadores internos.
  - [ ] Identificadores externos.
  - [ ] Chave de idempotência.
  - [ ] Hash do payload.
  - [ ] Carteira.
  - [ ] Provedor.
  - [ ] Jogador. 
  - [ ] Rodada. 
  - [ ] Money. 
  - [ ] Jogo. 
  - [ ] Tipo. 
- [ ] Persistir (quando aplicável) as seguintes informações:
  - [ ] Resultado financeiro retornado ao provedor.
  - [ ] Referência interna resolvida.
  - [ ] O código de falha.
- [ ] Rejeição e falha permanente validadas pelo domínio.
- [ ] Transações terminais não sofrem novas transições.
  - [ ] Replay consulta seu resultado persistido sem reaplicar a operação.
- [ ] Máquina de estados e distinção entre falhas transitórias e falhas permanentes documentada.
- [ ] `OPENING` reservado à abertura interna de carteira. 
  - [ ] Rejeitar esse tipo quando enviado por HTTP ou SQS.
- [ ] `OPENING` exige identidade interna estável, carteira, jogador, moeda, valor, estado e timestamps. 
  - [ ] Provedor, ID externo, chave e hash externos, rodada, jogo e referência não se aplicam a essa origem.
  - [ ] O _schema_ deve distinguir operações internas e externas e impedir crédito inicial duplicado.

### 6.4. WalletLedgerEntry

- [ ] Cada lançamento registra os seguintes dados: id, walletId, transactionId, direção (DEBIT ou CREDIT), valor, saldo anterior, saldo posterior e instante de criação.
- [ ] O lançamento é imutável.
- [ ] Sua construção deve validar `balanceAfter = balanceBefore ± money`, conforme a direção.
- [ ] O banco de dados contém a unicidade de (walletId, transactionId) 
  - [ ] Contém proteção contra edição ou exclusão. 
  - [ ] LOSS e operações rejeitadas não produzem lançamentos. 

### 6.5. Inbox e outbox

- [ ] Na entrada por SQS, o registro da inbox e a conclusão durável do tratamento compartilham a transação SQL das alterações de domínio, do ledger e dos eventos correspondentes.
- [ ] Uma referência pendente pode ter sua mensagem de entrada concluída após a pendência estar persistida
  - [ ] O worker de referências assume a continuidade.

### 7. Operações e refêrencias

- [ ] Referências não recebem duas reversões bem-sucedidas do mesmo tipo.
- [ ] Tratamento de combinações de REFUND e ROLLBACK sobre a mesma aposta documentado.
  - [ ] Preserva a coerência financeira ao impedir a devolução duplicada do mesmo débito.
  - [ ] Quando processado, produz `WagerTransactionProcessed`, sem `WalletBalanceChanged`.
- [ ] BET, WIN, REFUND e ROLLBACK exigem valor maior que zero.
- [ ] LOSS exige a moeda da carteira.
- [ ] Reversões que precisam debitar mais que o saldo disponível são rejeitadas e auditáveis. 
  - [ ] O código de falha é diferente do que é usado para uma aposta sem saldo.
- [ ] A operação é persistida como `PENDING_REFERENCE` quando a referência ainda não tiver chegado. 
  - [ ] Um worker tenta novamente com backoff exponencial, inclusive após reinicialização da aplicação.
  - [ ] Número máximo de tentativas ou TTL definado. 
    - [ ] Quando esgotado, finalizado como REJECTED.
      - [ ] Informa um código de referência não encontrada.
      - [ ] Produz um evento de rejeição. 
      - [ ] Comportamento quando a referência existe, mas ainda está pendente ou terminou sem sucesso documentado e explicado.
      - [ ] Toda rejeição fornece um `failureCode` estável e documentado, distinguindo entradas corrigíveis de resultados definitivos.

### 8. Concorrência

- [ ] Coordenação por carteira. 
- [ ] Utilizar locking pessimista, controle otimista com retry limitado, atualização atômica condicionada ou uma combinação justificável.

<div style="text-align: end;"><a href="#sumário">Voltar para o sumário</a></div>

Claro. A melhor forma é agrupar por **comportamento verificável**, usando subchecklists para detalhes relacionados. Assim o checklist fica bem menor sem perder cobertura.

### 9. Contratos HTTP

<div style="text-align: end;"><a href="#sumário">Voltar para o sumário</a></div>

#### 9.1. Abertura de carteira

- [ ] `POST /wallets` aceita `playerId` e `initialBalance { amount, currency }` e retorna `id`, `playerId`, `balance` e `version`.
- [ ] Abertura com saldo positivo cria, no mesmo commit:
  - [ ] `OPENING` em `PROCESSED`.
  - [ ] Lançamento de crédito no ledger.
  - [ ] Outbox de `WagerTransactionProcessed` e `WalletBalanceChanged`.
  - [ ] Versão da carteira igual a `1`.
  - [ ] Eventos internos sem metadados externos inaplicáveis.
- [ ] Abertura com saldo zero não cria `OPENING`, ledger ou eventos financeiros.
- [ ] Nova carteira para o mesmo jogador e moeda resulta em conflito.

<div style="text-align: end;"><a href="#sumário">Voltar para o sumário</a></div>

#### 9.2. Leitura

- [ ] Implementa `GET /wallets/:walletId`, `GET /wallets/:walletId/ledger?cursor=...&limit=50`, `GET /wagering/transactions/:transactionId` e `GET /providers/:providerId/wagering/transactions/:externalTransactionId`.
  - [ ] Ledger usa cursor opaco e ordenação estável.
  - [ ] Consultas de transação permitem acompanhar pendências e consultar códigos de rejeição/falha.

<div style="text-align: end;"><a href="#sumário">Voltar para o sumário</a></div>

#### 9.3. Envio de operação

- [ ] `POST /wagering/transactions` aceita `providerId`, `externalTransactionId`, `playerId`, `walletId`, `roundId`, `gameId`, `kind` e `money { amount, currency }`.
  - [ ] `Idempotency-Key` é obrigatório.
  - [ ] O cliente pode usar `{providerId}:{externalTransactionId}`, sem o servidor substituir silenciosamente a chave recebida.
  - [ ] Reversões aceitam `referenceExternalTransactionId`.
- [ ] A idempotência utiliza hash determinístico do conteúdo de negócio.
  - [ ] Hash usa JSON canônico com chaves ordenadas.
  - [ ] Exclui `Idempotency-Key` e metadados de transporte.
  - [ ] Algoritmo, campos e normalizações são documentados.
  - [ ] O cálculo é equivalente entre HTTP e SQS.
- [ ] Chave e conteúdo equivalentes retornam o resultado persistido com `idempotentReplay: true`.
- [ ] Chave reutilizada com conteúdo diferente resulta em conflito.
- [ ] `(providerId, externalTransactionId)` não pode ser reaplicado com outra chave.
- [ ] Replay de operação concluída retorna o saldo observado no processamento original, independentemente de movimentações posteriores.
- [ ] O contrato documenta códigos HTTP e corpos distintos para entrada inválida, conflito, rejeição de negócio, processamento pendente e indisponibilidade transitória.

<div style="text-align: end;"><a href="#sumário">Voltar para o sumário</a></div>

#### 9.4. Reconciliação

- [ ] `POST /wallets/:walletId/reconciliation` reconstrói o saldo a partir do ledger, incluindo a abertura, e compara com o saldo armazenado em visão consistente.
  - [ ] `difference = saldo armazenado - saldo reconstruído`.
  - [ ] Resposta, logs e métricas reportam divergências.
  - [ ] Reconciliação não altera o saldo.

<div style="text-align: end;"><a href="#sumário">Voltar para o sumário</a></div>

#### 9.5. Health checks

- [ ] `GET /health/live` verifica liveness do processo e `GET /health/ready` verifica readiness de PostgreSQL e SQS.

### 10. Consumidor SQS

#### 10.1. Filas e mensagens

- [ ] Provisiona `wager-transactions.fifo` e `wager-transactions-dlq.fifo`, incluindo redrive.
- [ ] `WagerTransactionRequested` possui envelope com `messageId`, `type`, `occurredAt` e `data`; `data` contém os dados financeiros da operação e `idempotencyKey`.
  - [ ] SQS usa `data.idempotencyKey` para idempotência financeira e inbox para deduplicação adicional.
  - [ ] `messageId` é a identidade durável da mensagem e seu hash é validado em reentregas.
  - [ ] `MessageGroupId` e `MessageDeduplicationId` são documentados.
- [ ] HTTP e SQS compartilham o mesmo caso de uso e garantias de idempotência, com concorrência entre ambos validada.

<div style="text-align: end;"><a href="#sumário">Voltar para o sumário</a></div>

#### 10.2. Processamento e recuperação

- [ ] Mensagem só é removida após o commit do tratamento durável.
- [ ] Rejeições de negócio confirmadas são terminais e permitem remoção.
- [ ] Falhas transitórias usam retry com backoff; erros permanentes ou tentativas esgotadas chegam à DLQ.
- [ ] São documentados limite de tentativas, `visibility timeout` e tratamento de mensagens inválidas.
- [ ] `SIGTERM` interrompe busca de trabalho e conclui o processamento em andamento dentro do prazo ou libera a visibilidade para reentrega segura.

### 11. Publicação com transactional outbox

#### 11.1. Atomicidade e worker

- [ ] Estado da operação, saldo, ledger, inbox e eventos são confirmados atomicamente conforme aplicável.
- [ ] Worker separado publica outbox pendente e suporta múltiplos publishers, disputa, backoff e recuperação de trabalho abandonado.
  - [ ] Recupera interrupção entre commit e publicação.
  - [ ] Recupera interrupção entre publicação e confirmação na outbox.
  - [ ] Outra instância assume eventos pendentes.
  - [ ] Republicações preservam `eventId`.

<div style="text-align: end;"><a href="#sumário">Voltar para o sumário</a></div>

#### 11.2. Eventos

- [ ] Destino dos eventos é provisionado e contratos de roteamento/consumo são documentados.
- [ ] Eventos possuem gatilhos e payloads concretos:
  - [ ] `WagerTransactionProcessed`: conclusão bem-sucedida, incluindo `LOSS`.
  - [ ] `WagerTransactionRejected`: rejeição definitiva por regra de negócio.
  - [ ] `WalletBalanceChanged`: alteração efetiva do saldo.
  - [ ] `WagerTransactionPendingReference`: registro de espera pela referência.
- [ ] Todo envelope contém `eventId`, `eventType`, `aggregateId`, `correlationId`, `causationId` opcional, `occurredAt`, `version` e `data` tipado.
  - [ ] Tipo e versão são definidos pelo construtor.
  - [ ] Timestamps usam UTC/RFC 3339 e valores monetários usam strings decimais.
  - [ ] Payload da outbox é snapshot imutável.
- [ ] `WalletBalanceChanged` contém `walletId`, `transactionId`, `direction`, `money`, `balanceBefore`, `balanceAfter` e `walletVersion`.

<div style="text-align: end;"><a href="#sumário">Voltar para o sumário</a></div>

### 12. Observabilidade

#### 12.1. Logs e métricas

- [ ] Logs são JSON e incluem, quando disponíveis, `correlationId`, `messageId`, `transactionId`, `walletId` e `providerId`.
  - [ ] Não registram credenciais, dados sensíveis ou payloads financeiros completos.
- [ ] Métricas cobrem resultados por status, duplicatas, retries, DLQ, conflitos de concorrência, atraso da outbox, latência de processamento e divergências de reconciliação.
- [ ] Health checks fazem parte da observabilidade.
- [ ] OpenTelemetry e dashboards são diferenciais opcionais.

<div style="text-align: end;"><a href="#sumário">Voltar para o sumário</a></div>

### 13. Verificação obrigatória

#### 13.1. Testes unitários

- [ ] Cobrem `Money`: parsing, operações, escala, limites numéricos, entradas inválidas e moedas incompatíveis.
- [ ] Cobrem invariantes da carteira, transições de estado, regras dos cinco tipos externos, valores zero e conflito de payload para mesma chave.
- [ ] Cobrem abertura interna, incluindo metadados e eventos.

#### 13.2. Testes de integração

- [ ] Executam PostgreSQL, IdP e LocalStack/MiniStack em containers reais.
  - [ ] Verificam migrations, constraints, imutabilidade do ledger, atomicidade financeira, inbox, reentrega, outbox concorrente, retry, DLQ e recuperação após reinicialização.
  - [ ] Verificam composição Fx, início/encerramento e liberação dos workers.
  - [ ] Não substituem toda a infraestrutura por mocks.

<div style="text-align: end;"><a href="#sumário">Voltar para o sumário</a></div>

#### 13.3. Autenticação e autorização

- [ ] Integração real com IdP rejeita credenciais ausentes, inválidas e expiradas.
- [ ] Existe isolamento entre provedores em consultas e replays e restrição das operações internas.
- [ ] Acessos não autorizados não produzem efeitos financeiros nem expõem dados.

#### 13.4. Concorrência e recuperação

- [ ] 50 apostas iguais em paralelo produzem um único débito.
- [ ] Duas apostas de `80.00` sobre saldo `100.00` são disputadas sem violar a integridade.
- [ ] Carteiras distintas são processadas simultaneamente.
- [ ] Cenários relevantes são executados com pelo menos três instâncias independentes.
- [ ] Interrupção após commit e antes da remoção da mensagem comprova reentrega.
- [ ] Dois publishers disputando a mesma outbox comprovam recuperação.
- [ ] `REFUND`/`ROLLBACK` antes da referência comprova resolução posterior ou rejeição por expiração.
- [ ] Reinicialização preserva idempotência, pendências e consistência financeira.
- [ ] Fluxo `PENDING` interrompido antes da execução é retomado por outra instância.
- [ ] Saldo armazenado é comparado com créditos menos débitos do ledger.
- [ ] Existem cenários cruzando HTTP e SQS para a mesma operação.
- [ ] Duplicidades exercitam e comprovam deduplicação da aplicação.
- [ ] `go test -race` é executado nos testes aplicáveis.

<div style="text-align: end;"><a href="#sumário">Voltar para o sumário</a></div>

### 14. Critérios de avaliação

- [ ] **Integridade financeira — 20 pts:** precisão, invariantes, reversões e reconciliação confiáveis.
- [ ] **Concorrência — 20 pts:** coordenação entre processos e ausência de atualizações perdidas.
- [ ] **Idempotência — 15 pts:** persistência, detecção de conflito e reprodução do resultado original.
- [ ] **Mensageria e recuperação — 15 pts:** inbox, outbox, retries, DLQ e encerramento seguro.
- [ ] **Modelagem e arquitetura — 10 pts:** encapsulamento Go, Fx, autenticação e autorização.
- [ ] **Testes — 10 pts:** integração real, isolamento entre provedores, paralelismo e interrupção.
- [ ] **Observabilidade — 5 pts:** logs, métricas e health checks.
- [ ] **Documentação — 5 pts:** execução reproduzível e decisões técnicas explicadas.
- [ ] Critérios eliminatórios são atendidos:
  - [ ] Autenticação efetiva nos endpoints de negócio.
  - [ ] Nenhum acesso não autorizado a operações/transações.
  - [ ] Nenhum cálculo monetário em ponto flutuante.
  - [ ] Nenhum saldo negativo causado por concorrência.
  - [ ] Nenhuma movimentação duplicada.
  - [ ] Idempotência persistida, não apenas em memória.
  - [ ] Funcionamento correto com múltiplas instâncias.
  - [ ] Nenhuma publicação anterior ao commit.
  - [ ] Ledger auditável.
  - [ ] PostgreSQL, SQS e IdP não são integralmente substituídos por mocks.
- [ ] Diferenciais opcionais: partidas dobradas, tracing e testes de carga.
  - [ ] Testes de carga, se existentes, documentam comando, ambiente, metodologia, throughput, p50/p95/p99, erros, conflitos e atraso da outbox.
  - [ ] Não há meta mínima de RPS.

<div style="text-align: end;"><a href="#sumário">Voltar para o sumário</a></div>

### 15. Entrega

#### 15.1. Artefatos e execução

- [ ] Entrega código, migrations, Docker Compose e instruções para reprodução a partir de checkout limpo.
- [ ] Código está formatado com `gofmt` e possui dependências reproduzíveis.
- [ ] `README.md` documenta pré-requisitos, variáveis de ambiente, inicialização das filas/aplicação, aplicação e reversão de migrations, exemplos de chamadas e comandos de teste.
- [ ] `.env.example` contém valores locais de exemplo e nenhum segredo real.
- [ ] IdP é provisionado automaticamente, com identidades de teste e instruções para fluxos autenticados.

<div style="text-align: end;"><a href="#sumário">Voltar para o sumário</a></div>

#### 15.2. Arquitetura

- [ ] `ARCHITECTURE.md` documenta dinheiro, transações, idempotência, locks, referências pendentes, reversões, inbox/outbox, autenticação/autorização, Fx e shutdown.
  - [ ] Também explicita limitações, interpretações adotadas e trabalho não concluído.

#### 15.3. Comandos e testes

- [ ] Disponibiliza `docker compose up --build`, `go test ./...`, `go test -race ./...` e `go vet ./...`, ou equivalentes documentados.
- [ ] Documenta separadamente preparação das dependências, integração, múltiplas instâncias e simulações de falha.
- [ ] Documenta comandos correspondentes quando utiliza build tags.
