# ADR-0001: Priorizar segurança, integridade das evidências, confiabilidade e escalabilidade

- **Status:** Aceita
- **Data:** 2026-10-15
- **Decisores:** Cauã Marinho, João Ferraz, Guilherme Loiola e Davi Araújo
- **Características arquiteturais relacionadas:** Segurança, Integridade/Autenticidade das evidências, Confiabilidade, Escalabilidade (elasticidade da captura)
- **Requisitos e restrições relacionados:** Atributo de qualidade "Segurança"; REQ01–REQ04 (acesso); REQ05–REQ07 (créditos e pagamento); REQ08–REQ11 (captura); REQ14–REQ15 (relatório e .zip); D1 (advogados como público)

## Contexto

O Veridit registra conteúdo da web (captura de tela ou vídeo de navegação) e entrega um
relatório e um pacote .zip com os arquivos capturados. Um dos públicos previstos são
advogados (D1), o que indica que o produto final tende a ser usado como prova. O documento
de requisitos explicita apenas uma característica (segurança); as demais são implícitas no
domínio.

Levantamos 12 características candidatas (seção 2 do documento de arquitetura). Seguindo
Richards & Ford, cada característica suportada aumenta a complexidade do sistema, então a
arquitetura deve priorizar poucas: as que mais influenciam a estrutura e cuja falha
inviabiliza o produto. As demais continuam atendidas, mas por táticas locais e não como
direcionadoras do estilo.

## Decisão

As quatro características prioritárias do Veridit, em ordem, são:

1. **Segurança** (explícita). Apenas usuários cadastrados acessam o sistema, cada usuário
   só acessa os próprios registros, e dados pessoais e de pagamento ficam protegidos.
   *Cenário mensurável:* 100% dos endpoints, exceto cadastro, login e recuperação de senha,
   rejeitam requisições sem token válido; nenhuma senha é armazenada em formato reversível.
2. **Integridade e autenticidade das evidências** (implícita). Deve ser possível demonstrar
   que os arquivos não foram alterados desde a captura e quando ela ocorreu.
   *Cenário:* a alteração de qualquer byte de um arquivo do pacote é detectada pela
   verificação do manifesto; 100% dos registros concluídos têm hash e carimbo do tempo.
3. **Confiabilidade** (implícita). Nenhum pagamento confirmado deixa de virar crédito,
   nenhum crédito é consumido sem registro concluído, e nenhuma captura concluída se perde.
   *Cenário:* com webhooks duplicados ou queda de um serviço no meio do fluxo, o efeito
   final é aplicado exatamente uma vez (0 créditos duplicados ou perdidos).
4. **Escalabilidade, com elasticidade no serviço de captura** (implícita). A captura é a
   operação mais cara (navegador dedicado e codificação de vídeo por minutos) e a demanda é
   irregular. *Cenário:* um aumento de 10x nas sessões simultâneas é absorvido apenas
   adicionando instâncias de captura, sem mudança de código, com tempo de espera p95 por uma
   sessão de até 30 s.

Os valores dos cenários são metas iniciais da equipe e devem ser validados com o cliente.

## Alternativas consideradas

| Alternativa | Prós | Contras |
|---|---|---|
| Disponibilidade no lugar de escalabilidade | Importante para qualquer SaaS | Não há SLA nos requisitos; uma indisponibilidade curta é tolerável se nada se perde (coberto por confiabilidade), e a redundância já vem do escalonamento horizontal |
| Desempenho no top 4 | Melhora a experiência | O tempo da captura depende do site de destino; não diferencia o produto nem direciona o estilo |
| Auditabilidade como característica separada | Visível para o cliente | Já é coberta pela cadeia de custódia (integridade) e pelos logs de acesso (segurança) |
| Extensibilidade/configurabilidade no top 4 (D1, D2) | Antecipa empresas e novos pacotes | Atendida por parametrização e polimorfismo nos componentes, sem exigir um estilo específico |
| Privacidade (LGPD) como característica separada | Destaca dados pessoais (CPF, endereço) | Tratada dentro de segurança, com táticas específicas de proteção de dados |

## Justificativa

Escolhemos as características que, se falharem, tornam o produto inútil ou perigoso, e que
mudam a forma da arquitetura. Segurança é a única explícita e envolve dados pessoais e de
pagamento. Uma evidência que pode ser contestada por falta de integridade não tem valor para
um advogado, então integridade é o núcleo do valor do produto. O usuário paga antes de usar
(créditos), o que torna inaceitável perder pagamento ou crédito: daí a confiabilidade. Por
fim, a captura tem perfil de carga diferente de todo o resto (pesada, longa e irregular), o
que exige escalar só essa parte. Essas quatro forças, juntas, levam a separar a captura do
núcleo transacional e a usar comunicação assíncrona (ADR-0002).

## Consequências

**Positivas**

- Critério claro para escolher o estilo (ADR-0002) e para resolver trade-offs futuros.
- Cada característica prioritária tem um cenário mensurável e pelo menos uma função de aptidão.

**Negativas / trade-offs**

- Custo maior que o de uma aplicação simples: carimbo do tempo, armazenamento imutável,
  broker de mensagens e orquestração de contêineres.
- Desempenho e disponibilidade recebem menos investimento inicial.

**Riscos**

- Se o cliente definir um SLA de disponibilidade rígido, esta ADR deve ser revista.

## Conformidade

- Revisar a lista sempre que uma nova versão do documento de requisitos for publicada.
- Cada característica prioritária tem funções de aptidão listadas na seção 9 do documento de
  arquitetura (táticas) e nas ADRs 0003 a 0010.

## Notas

- ADRs relacionadas: 0002 (estilo), 0003 e 0004 (integridade), 0005 e 0006 (segurança),
  0006, 0007 e 0008 (confiabilidade), 0009 (escalabilidade).
- Elaborada em sala com apoio de ferramenta de IA; registro da interação em <local>.
  Revisada e aceita pela equipe.
