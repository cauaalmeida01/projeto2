# ADR-0006: Delegar pagamentos ao Mercado Pago (checkout e Pix), sem armazenar dados de pagamento

- **Status:** Aceita
- **Data:** 2026-10-15
- **Decisores:** Cauã Marinho, João Ferraz, Guilherme Loiola e Davi Araújo
- **Características arquiteturais relacionadas:** Segurança, Confiabilidade, Interoperabilidade
- **Requisitos e restrições relacionados:** REQ05 (comprar créditos), REQ06 (pagamento pelo Mercado Pago ou pela plataforma, com QR Code/código Pix), REQ07 (e-mail de confirmação); D2 (pacotes); Apêndice A (dados de faturamento)

## Contexto

O REQ06 exige duas formas de pagamento: pelo Mercado Pago ou pela própria plataforma do
Veridit, gerando QR Code ou código Pix. Processar cartões diretamente coloca o sistema no
escopo completo do PCI DSS. Além disso, a confirmação de pagamento é assíncrona: um Pix pode
ser pago minutos depois de gerado, e o aviso chega por webhook, que pode atrasar, se repetir
ou se perder.

## Decisão

O componente de Processamento de Pagamentos vai integrar com o Mercado Pago por meio de um
**adaptador** (porta `GatewayPagamento`):

- **"Pagar com Mercado Pago":** cria uma preferência no Checkout do Mercado Pago e redireciona
  o usuário.
- **"Pagar pela plataforma Veridit":** cria um pagamento Pix pela API do provedor e exibe, na
  interface do Veridit, o QR Code e o código copia-e-cola, com prazo de expiração (ex.: 30 min).
- **Confirmação:** o webhook tem a assinatura validada; o sistema consulta o status na API do
  provedor (não confia no corpo do webhook) e processa de forma **idempotente** pelo id do
  pagamento. Um job de **reconciliação** periódica busca pagamentos pendentes para cobrir
  webhooks perdidos.
- Ao confirmar: o pedido muda para "Pago", a carteira recebe os créditos (ADR-0008) e o evento
  `PagamentoConfirmado` dispara o e-mail (REQ07).
- O Veridit guarda apenas os dados de faturamento do Apêndice A e os ids do provedor; **nunca
  dados de cartão**.

## Alternativas consideradas

| Alternativa | Prós | Contras |
|---|---|---|
| Processar cartões direto com uma adquirente | Taxas potencialmente menores | Escopo completo do PCI DSS; risco e custo de conformidade altos |
| Pix direto com banco (API Pix) | Sem intermediário para o Pix | Exige conta PJ, certificados mTLS e conciliação própria; viável em versão futura |
| Outro provedor (Stripe, PagSeguro, Asaas) | Alternativas maduras | O requisito cita o Mercado Pago; o adaptador permite trocar depois |

## Justificativa

Delegar a captura dos dados de pagamento ao provedor reduz drasticamente o escopo de
segurança (o sistema nunca vê dados de cartão). Usar um único provedor para as duas opções do
REQ06 simplifica a integração e a conciliação. Validação de webhook, consulta de status,
idempotência e reconciliação garantem que todo pagamento confirmado vire crédito exatamente
uma vez, atendendo à confiabilidade (ADR-0001).

## Consequências

**Positivas**

- Menor escopo de PCI DSS; nenhum dado de cartão armazenado.
- Troca de provedor isolada no adaptador.

**Negativas / trade-offs**

- Dependência de um fornecedor e de suas taxas.
- Indisponibilidade do provedor impede compras (mas não afeta capturas de quem já tem créditos).

**Riscos**

- Interpretação do REQ06: assumimos que "pela plataforma do Veridit" significa pagar via Pix
  sem sair da interface do Veridit, ainda que o Pix seja gerado pelo provedor. **Validar com o cliente.**

## Conformidade

- Teste de idempotência: o mesmo webhook processado duas vezes gera um único lançamento de crédito.
- Verificação no CI de que nenhuma migração cria colunas de número de cartão ou CVV.
- Alerta para pedidos "Pendentes" com Pix expirado sem conciliação.

## Notas

- ADRs relacionadas: 0001, 0007, 0008.
- Elaborada em sala com apoio de ferramenta de IA; registro da interação em <local>.
