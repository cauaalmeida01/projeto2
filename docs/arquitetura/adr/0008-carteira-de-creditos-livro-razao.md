# ADR-0008: Modelar a carteira de créditos como livro-razão com reserva, consumo e estorno

- **Status:** Aceita
- **Data:** 2026-10-15
- **Decisores:** <nomes da equipe>
- **Características arquiteturais relacionadas:** Confiabilidade, Integridade (dos dados financeiros), Auditabilidade
- **Requisitos e restrições relacionados:** REQ05 (comprar créditos), REQ06 (pagamento), REQ08 (iniciar registro consome créditos); D2 (pacotes)

## Contexto

Créditos comprados (REQ05) são consumidos ao registrar conteúdo (REQ08). Uma coluna de
"saldo" atualizada diretamente tem três problemas: condições de corrida (duas capturas
simultâneas com um único crédito), créditos perdidos quando a captura falha e falta de
histórico para resolver contestações do usuário.

Premissa: a quantidade de créditos consumida por tipo de registro (tela ou vídeo, e por
duração do vídeo) não está definida nos requisitos; será configurável no catálogo e deve ser
validada com o cliente.

## Decisão

- A carteira será um **livro-razão somente de inserção**, com lançamentos do tipo
  `CREDITO_COMPRA`, `RESERVA`, `CONSUMO`, `ESTORNO_RESERVA` e `AJUSTE`.
- O saldo disponível é materializado com **controle de concorrência** (versão otimista ou
  bloqueio por carteira) e sempre igual à soma dos lançamentos.
- Ao iniciar um registro, o serviço de Registros **reserva** os créditos (chamada síncrona);
  sem saldo, a operação é recusada e o usuário é levado à compra.
- Quando o registro é selado, a reserva vira **consumo**; em falha, cancelamento ou timeout, a
  reserva é **estornada**. Reservas têm prazo de validade para não ficarem presas.
- Toda operação tem **chave de idempotência** (id do registro ou do pagamento).

## Alternativas consideradas

| Alternativa | Prós | Contras |
|---|---|---|
| Coluna de saldo simples | Muito simples | Corridas, sem histórico, créditos perdidos em falhas |
| Debitar só no fim da captura | Nunca cobra captura que falhou | Usuário sem saldo iniciaria capturas caras; risco de saldo negativo |
| Debitar no início, sem reserva | Simples | Exige fluxo de reembolso em toda falha; menos explícito que a reserva |

## Justificativa

O livro-razão torna cada movimentação rastreável e permite recalcular o saldo a qualquer
momento. A reserva com consumo ou estorno garante que o usuário nunca perde crédito por
falha da plataforma e que não há gasto duplo, atendendo à confiabilidade (ADR-0001).

## Consequências

**Positivas**

- Histórico completo para suporte e contestação.
- Sem gasto duplo e sem créditos perdidos.

**Negativas / trade-offs**

- Modelo e consultas mais complexos que uma coluna de saldo.
- Necessidade de job de expiração de reservas e de verificação de invariantes.

## Conformidade

- Job de verificação: saldo materializado = soma dos lançamentos, para todas as carteiras.
- Teste de concorrência: N reservas simultâneas com saldo para apenas uma resultam em exatamente
  um sucesso.

## Notas

- ADRs relacionadas: 0006, 0007.
- Elaborada em sala com apoio de ferramenta de IA; registro da interação em <local>.
