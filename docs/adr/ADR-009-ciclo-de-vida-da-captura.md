# ADR-009: Ciclo de vida da captura como máquina de estados persistida, com reserva de créditos

- **Título:** ADR-009: Controlar cada registro por uma máquina de estados gravada no banco, reservar o crédito ao iniciar e só debitá-lo na conclusão, preservando o que foi gravado em caso de interrupção
- **Status:** Proposta
- **Data:** 2026-10-15
- **Requisitos e restrições relacionados:**
  - REQ 05 (créditos "para realizar o registro de ocorrências"), REQ 08 (iniciar), REQ 09 (concluir gravação), REQ 11 (concluir captura de tela), REQ 10 e REQ 11 duplicado (e-mails), REQ 12 e REQ 13 (status nos registros).
  - Apêndice A: campo "status" em "Registros realizados"; data/hora de início e fim; duração.
  - ADR-001 (Confiabilidade da captura); ADR-002; ADR-005; ADR-008.
  - Lacunas da análise de requisitos: valores de status (seção 4, item 8) e momento do débito de créditos (seção 4, item 14).

## Contexto

A captura é uma operação longa e sujeita a falhas (ADR-002): o navegador pode travar, o usuário pode perder a conexão, o site pode não responder. O Apêndice A prevê o campo "status" em "Registros realizados", mas o documento não define os valores possíveis nem quando o crédito é consumido. Sem uma regra clara, o usuário pode perder créditos por uma captura que falhou ou ver como "concluído" um registro incompleto, o que é o risco principal da Confiabilidade da captura (ADR-001).

## Decisão

1. Cada registro segue esta máquina de estados **[inferência: os valores não estão no documento]**:

   | Status | Significado | Próximos status |
   |---|---|---|
   | `INICIANDO` | Registro criado, crédito sendo reservado, navegador sendo alocado | `EM_ANDAMENTO`, `FALHOU` |
   | `EM_ANDAMENTO` | Usuário navegando; arquivos sendo gerados e selados | `FINALIZANDO`, `INTERROMPIDO` |
   | `FINALIZANDO` | Usuário concluiu (REQ 09/REQ 11); manifesto sendo fechado | `CONCLUIDO`, `INTERROMPIDO` |
   | `CONCLUIDO` | Manifesto assinado e guardado; crédito debitado; e-mail enviado | final |
   | `INTERROMPIDO` | A sessão caiu ou passou do tempo limite; os arquivos já selados são mantidos | final |
   | `FALHOU` | Não foi possível começar (sem crédito, sem navegador disponível, site inacessível) | final |

2. **Toda transição é gravada no banco antes de ser executada**, com data/hora. Se o serviço reiniciar, ele retoma ou encerra cada registro a partir do último status gravado.
3. **Créditos:** ao iniciar, o C07 pede ao C05 a **reserva** do crédito (o saldo disponível diminui, mas o crédito ainda não foi gasto). Ao chegar em `CONCLUIDO`, o débito é confirmado; em `INTERROMPIDO` ou `FALHOU`, a reserva é estornada. Confirmação e estorno são pedidos pela caixa de saída (ADR-008) **[inferência: o documento não define a regra de consumo]**.
4. **Supervisão:** o Supervisor de Sessões do C07 recebe um sinal de vida de cada navegador. Se o sinal parar, ou se a sessão passar do tempo limite de inatividade ou da duração máxima, o registro vai para `INTERROMPIDO`.
5. **Preservação:** o vídeo é gravado em segmentos curtos, selados um a um (ADR-005). Uma interrupção perde no máximo o segmento em andamento, e os segmentos anteriores continuam disponíveis no registro `INTERROMPIDO`, que aparece com esse status na listagem (REQ 12) e nos detalhes (REQ 13).

## Alternativas consideradas

**A. Débito na hora de iniciar, sem estorno**
- Prós: o mais simples.
- Contras: o usuário paga por capturas que falharam por culpa do sistema.

**B. Status só em memória durante a captura, gravado no fim**
- Prós: menos escritas no banco.
- Contras: se o serviço cair, não sobra registro de que a captura existia; os créditos e os arquivos ficam órfãos.

**C. Descartar tudo o que foi gravado quando a captura é interrompida**
- Prós: só existem registros completos.
- Contras: a navegação não pode ser repetida no mesmo estado (o conteúdo pode ter sido apagado). Perder o que já foi gravado vai contra o propósito do produto.

## Justificativa

- **Confiabilidade da captura:** o status sempre corresponde ao que aconteceu, inclusive depois de uma queda do serviço, e nada do que foi gravado se perde sem aviso.
- **Integridade:** um registro só fica `CONCLUIDO` depois que o manifesto foi assinado e guardado. Um registro `INTERROMPIDO` mostra claramente que está incompleto, em vez de parecer íntegro.
- **Justiça com o usuário:** créditos só são consumidos por capturas concluídas.

## Consequências

- **Positivas:**
  - Resolve as lacunas de status e de débito de créditos com regras explícitas, que podem ser validadas com o engenheiro de requisitos.
  - Facilita o suporte: o histórico de transições mostra o que aconteceu em cada registro.
- **Negativas:**
  - Mais escritas no banco durante a captura.
  - Os tempos limite (inatividade e duração máxima) não estão no documento e precisam ser definidos **[inferência]**.
  - Registros `INTERROMPIDO` aparecem na listagem e podem precisar de explicação na interface.
  - Fica em aberto se um registro `INTERROMPIDO` pode gerar relatório e zip (REQ 14, REQ 15). Proposta: sim, com o status indicado de forma destacada no relatório.
