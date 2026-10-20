# ADR-0007: Comunicar de forma assíncrona com outbox transacional, filas duráveis e DLQ

- **Status:** Aceita
- **Data:** 2026-10-15
- **Decisores:** <nomes da equipe>
- **Características arquiteturais relacionadas:** Confiabilidade, Escalabilidade, Disponibilidade
- **Requisitos e restrições relacionados:** REQ05–REQ07 (compra → crédito → e-mail), REQ08–REQ11 (registro → captura → selagem → e-mail), REQ10, REQ11-B

## Contexto

Os fluxos principais atravessam vários serviços: pagamento → créditos → e-mail, e registro →
captura → selagem → conclusão → e-mail. Se um serviço grava no banco e cai antes de publicar
o evento (problema da escrita dupla), o fluxo para no meio: crédito pago e não creditado, ou
captura feita e registro nunca concluído. Chamadas síncronas em cadeia, por sua vez, tornam
o fluxo frágil e não combinam com capturas que duram minutos.

## Decisão

- Usar um **broker de mensagens** (RabbitMQ ou equivalente gerenciado) com filas duráveis e
  confirmação manual (*ack*).
- Produtores usam **outbox transacional**: o evento é gravado na mesma transação que a mudança
  de estado, e um *relay* publica e marca como enviado.
- Consumidores são **idempotentes** (tabela de mensagens já processadas por id do evento).
- Falhas têm **retentativas com backoff exponencial**; após N falhas, a mensagem vai para uma
  **fila de mensagens mortas (DLQ)** com alerta.
- Eventos levam `correlationId` e esquema versionado, definido no pacote
  `compartilhado/contratos`.
- O fluxo do registro é uma **saga orquestrada** pelo serviço de Registros: reservar
  créditos → solicitar captura → aguardar selagem → consumir créditos; em falha, estornar a
  reserva (compensação).

## Alternativas consideradas

| Alternativa | Prós | Contras |
|---|---|---|
| Só chamadas REST síncronas | Simples de entender e depurar | Acopla a disponibilidade dos serviços; timeouts em capturas longas; sem buffer para picos |
| Publicar direto no broker após o commit (sem outbox) | Menos código | Perde eventos se o serviço cair entre o commit e a publicação |
| Kafka | Alto volume e replay de eventos | Operação pesada; desproporcional ao volume do Veridit |
| Transação distribuída (2PC) | Consistência forte | Não suportada pela maioria dos brokers; bloqueios e baixa disponibilidade |

## Justificativa

O outbox garante que nenhum evento se perca; filas duráveis com ack garantem a entrega ao
menos uma vez; consumidores idempotentes transformam isso em efeito exatamente uma vez.
A saga com compensação garante que nenhum crédito fique preso ou seja perdido. A fila também
serve de buffer para picos de captura (ADR-0009).

## Consequências

**Positivas**

- Nenhum pagamento ou captura se perde por falha parcial.
- Serviços podem ficar fora do ar por algum tempo sem perder trabalho.

**Negativas / trade-offs**

- Consistência eventual: o usuário vê estados intermediários ("Processando", "Selando").
- Mais infraestrutura (broker) e complexidade de depuração.

**Riscos**

- Mensagens acumulando na DLQ sem tratamento. Mitigação: alerta quando a DLQ não estiver vazia
  e procedimento de reprocessamento documentado.

## Conformidade

- Teste de caos: derrubar o consumidor no meio do processamento; a mensagem é reentregue e o
  efeito é aplicado uma única vez.
- Métricas com alerta: tamanho da DLQ > 0; atraso do outbox p95 > 5 s.

## Notas

- ADRs relacionadas: 0002, 0006, 0008, 0009.
- Elaborada em sala com apoio de ferramenta de IA; registro da interação em <local>.
