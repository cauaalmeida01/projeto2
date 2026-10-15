# ADR-008: E-mails e pedidos entre serviços enviados por caixa de saída transacional

- **Título:** ADR-008: Gravar os pedidos para outros serviços (e-mails, débito ou estorno de créditos) numa caixa de saída, na mesma transação da mudança de estado, e entregá-los com nova tentativa
- **Status:** Proposta
- **Data:** 2026-10-15
- **Requisitos e restrições relacionados:**
  - REQ 07 (e-mail da compra), REQ 10 (e-mail da conclusão da gravação), REQ 11 duplicado (e-mail da conclusão da captura de tela); REQ 02 (recuperação de senha, se feita por e-mail).
  - REQ 05 e REQ 08–11 (consumo de créditos pela captura).
  - ADR-001 (Confiabilidade da captura); ADR-003 (sem broker externo); ADR-004 (banco compartilhado com esquemas por serviço); ADR-009.

## Contexto

Vários momentos do sistema precisam avisar outro serviço depois de mudar de estado: concluir a captura precisa confirmar o débito do crédito e enviar o e-mail (REQ 10, REQ 11 duplicado); confirmar o pagamento precisa enviar o e-mail (REQ 07). Se o aviso for uma chamada direta logo depois da transação, uma queda entre as duas operações deixa o sistema inconsistente: captura concluída sem e-mail, ou crédito reservado para sempre. Um broker de mensagens resolveria, mas a ADR-003 o descartou por custo.

## Decisão

1. Cada serviço tem uma tabela **`caixa_saida`** no próprio esquema (ADR-004).
2. Quando um serviço muda de estado e precisa avisar outro, ele grava o pedido na `caixa_saida` **na mesma transação** que a mudança de estado. Ou as duas coisas acontecem, ou nenhuma.
3. Um **despachante** em cada serviço lê os pedidos pendentes e os entrega ao serviço de destino por HTTP, repetindo com intervalos crescentes até receber confirmação.
4. Cada pedido tem um identificador único. O destino ignora pedidos repetidos (**idempotência**), já que a nova tentativa pode entregar o mesmo pedido mais de uma vez.
5. Pedidos que falham repetidamente vão para uma lista de pendências visível à equipe de operação.
6. O C14 aplica a mesma lógica com o provedor de e-mail: o e-mail é marcado como enviado só depois que o provedor aceita.

## Alternativas consideradas

**A. Chamada direta logo após a transação**
- Prós: simples e imediato.
- Contras: se o processo cair entre a transação e a chamada, o aviso se perde sem ninguém saber.

**B. Broker de mensagens (ex.: RabbitMQ)**
- Prós: entrega confiável e desacoplamento total.
- Contras: mais um componente de infraestrutura para operar; continua precisando de caixa de saída para garantir que a mensagem só seja publicada se a transação confirmar. É desproporcional para a v1 (ADR-003).

**C. Enviar o e-mail dentro da própria transação de conclusão**
- Prós: nenhuma estrutura extra.
- Contras: se o provedor de e-mail estiver lento ou fora do ar, a conclusão da captura também falha. A captura (top 4) passaria a depender de um serviço externo secundário.

## Justificativa

- **Confiabilidade da captura:** a conclusão do registro nunca falha por causa do e-mail ou de uma indisponibilidade momentânea de outro serviço, e nenhum aviso se perde.
- **Consistência:** o crédito reservado sempre termina confirmado ou estornado, porque o pedido foi gravado junto com o novo status.
- **Custo:** usa o banco que já existe (ADR-004), sem broker.

## Consequências

- **Positivas:**
  - Os REQ 07, 10 e 11 duplicado ficam garantidos mesmo com falhas temporárias do provedor de e-mail.
  - Os pedidos pendentes ficam visíveis no banco, o que facilita diagnosticar problemas.
- **Negativas:**
  - O e-mail pode chegar com alguns segundos ou minutos de atraso.
  - Cada serviço de destino precisa tratar pedidos repetidos.
  - O despachante consulta o banco periodicamente, o que gera alguma carga extra.
