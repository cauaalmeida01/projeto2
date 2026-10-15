# ADR-007: Integração de pagamentos por adaptadores, com confirmação por notificação do provedor

- **Título:** ADR-007: Isolar Mercado Pago e Pix atrás de uma interface única de pagamento e liberar créditos somente após a confirmação enviada pelo provedor, processada de forma idempotente
- **Status:** Proposta
- **Data:** 2026-10-15
- **Requisitos e restrições relacionados:**
  - REQ 05 (Comprar Créditos: selecionar pacote, informar faturamento, pagar), REQ 06 (pagamento pelo Mercado Pago ou pela plataforma do Veridit com QR Code ou número Pix), REQ 07 (e-mail da compra).
  - D2 (pacotes básico, médio e premium na v1); Apêndice A ("Pacote de créditos", "Dados da compra do pacote de créditos").
  - ADR-001 (Modularidade; Interoperabilidade tratada como decisão derivada); ADR-003.

## Contexto

O REQ 06 já prevê dois meios de pagamento com funcionamentos diferentes: o Mercado Pago (o usuário paga no ambiente do provedor) e o Pix gerado pela plataforma (o sistema mostra um QR Code e o pagamento acontece depois, no aplicativo do banco do usuário). O documento não diz como o sistema fica sabendo que o pagamento foi feito (análise, seção 4, item 10). A Interoperabilidade não entrou no top 4, mas a ADR-001 determinou que ela fosse tratada como isolamento das integrações em componentes.

## Decisão

1. O **C06 Pagamentos** define uma interface única, `ProvedorDePagamento` (criar cobrança, consultar situação, interpretar notificação), com **um adaptador para cada provedor**: `AdaptadorMercadoPago` e `AdaptadorPix` (este usando um banco ou instituição de pagamento que gere o QR Code e o código Pix) **[inferência sobre o provedor Pix]**.
2. A compra (REQ 05) é registrada como `PENDENTE` com os dados de faturamento do Apêndice A.
3. Os créditos são liberados **somente quando o provedor confirma o pagamento**, por meio de uma notificação (webhook) enviada ao Veridit. Se a notificação não chegar, uma tarefa periódica consulta a situação das compras pendentes.
4. A confirmação é **idempotente**: cada pagamento tem um identificador único, e uma confirmação repetida não credita duas vezes.
5. Na mesma transação que marca a compra como `PAGA` e credita o saldo (C05 e C06 ficam no mesmo serviço), o pedido de e-mail (REQ 07) é gravado na caixa de saída (ADR-008).
6. Os pacotes (D2) ficam cadastrados como dados, não no código.

## Alternativas consideradas

**A. Chamar a API de cada provedor diretamente no fluxo de compra**
- Prós: menos código no início.
- Contras: regras de cada provedor espalhadas pelo serviço; incluir um terceiro meio de pagamento exige mexer no fluxo inteiro.

**B. Liberar os créditos quando o usuário volta da tela do provedor**
- Prós: resposta imediata para o usuário.
- Contras: o retorno pode ser falsificado ou não acontecer (o usuário fecha a aba); no Pix não existe "volta", pois o pagamento é feito em outro aplicativo.

**C. Só consultar o provedor periodicamente (sem webhook)**
- Prós: não depende de o provedor alcançar o Veridit.
- Contras: atraso para liberar os créditos e muitas consultas desnecessárias. Ficou só como plano B.

## Justificativa

- **Modularidade:** um novo meio de pagamento é só um novo adaptador; o fluxo de compra não muda (evolução prevista por REQ 06 e D2).
- **Confiabilidade:** créditos só existem para pagamentos confirmados pelo provedor; a idempotência impede crédito em dobro quando o provedor repete a notificação.
- **Segurança:** a notificação recebida tem a assinatura do provedor verificada antes de qualquer processamento.

## Consequências

- **Positivas:**
  - Fluxo de compra igual para os dois meios de pagamento.
  - Resolve a lacuna da análise (como o sistema sabe que o Pix foi pago) com notificação mais consulta periódica.
- **Negativas:**
  - O endereço que recebe as notificações precisa estar acessível pela internet: é a única rota de entrada que não passa pela validação de token do usuário e por isso exige verificação da assinatura do provedor.
  - O e-mail do REQ 07 é enviado na **confirmação** do pagamento, não na solicitação. Isso interpreta a ambiguidade do REQ 07 (análise, seção 4, item 6) **[inferência]**.
  - Estorno, cancelamento e pagamento parcial não estão nos requisitos (análise, seção 4, item 16) e ficam fora da v1.
