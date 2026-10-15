# Priorização das Características Arquiteturais — Veridit

> Etapa 3 da documentação arquitetural. Base: `docs/caracteristicas.md` (12 candidatas) e `docs/analise-requisitos.md`. **Esta etapa NÃO fecha o top 4**: apresenta critérios, avaliação completa e combinações possíveis com trade-offs. A decisão final cabe à equipe, e será registrada em ADRs (entregável 2 do AGENTS.md). Inferências seguem marcadas **[inferência]**.

Nota: a equipe revisou `docs/caracteristicas.md` e não relatou ajustes nesta etapa; o arquivo permaneceu como documentado na etapa 2.

---

## 1. Critérios de priorização

Propomos quatro critérios, com pesos sugeridos. A equipe pode ajustar os pesos antes de decidir.

### C1 — Impacto no negócio e no risco legal (peso alto)

O quão diretamente a característica protege a proposta de valor do Veridit (evidência confiável para advogados e usuários — D1; receita por créditos — REQ 05; escopo da v1 — D1/D2) e o quanto sua ausência gera dano irreversível (evidência questionada em juízo, vazamento de dados, perda de janela de captura). Características cuja falha destrói o produto vencem características cuja falha apenas degrada a experiência.

### C2 — Influência na estrutura do sistema (peso alto)

O quanto a característica, se priorizada, muda decisões estruturais de arquitetura (estilo, decomposição em componentes, separação de módulos, escolha de mecanismos como criptografia, filas, armazenamento de mídia). Características que apenas exigem "fazer bem" operações pontuais influenciam menos a estrutura.

### C3 — Sustentação documental (peso médio)

O quanto a característica está apoiada no documento de requisitos (explícita ou com apoio parcial) versus puramente inferida. Características explícitas tendem a ser priorizadas porque sua ausência contradiz diretamente o documento; inferências fortes por domínio também pontuam alto, mas exigem confirmação do engenheiro de requisitos (seção 5 da análise).

### C4 — Custo de não-atender versus custo de atender (peso médio)

Equilíbrio entre o risco de deixar a característica de fora do top 4 (ela pode ser esquecida nas decisões de projeto) e o custo de atendê-la na v1 (D1/D2 delimitam escopo enxuto). Características "desejáveis mas caras" podem ficar fora do top 4 e ser tratadas como requisitos de acompanhamento, sem descartá-las.

---

## 2. Avaliação de todas as características

Escala: **Alta (A) / Média (M) / Baixa (B)** por critério, com uma síntese de pontuação (A=3, M=2, B=1, ponderada pelos pesos sugeridos: C1 ×1,5; C2 ×1,5; C3 ×1; C4 ×1 — pontuação máxima possível 12). A pontuação é apoio à discussão, não substitui o julgamento da equipe.

| # | Característica | C1 Negócio/risco | C2 Estrutura | C3 Sustentação | C4 Custo × risco | Pontuação ponderada | Observação-chave |
|---|---|---|---|---|---|---|---|
| 1 | Segurança (autenticação) | **A** — vazamento de evidências = dano legal (seção 3; D1; Apêndice A/CPF) | **A** — define camada de autenticação/autorização atravessando todo o sistema | **A** — única explícita (seção 3; REQ 01–04) | **A** — custo baixo, risco altíssimo | **12,0** | Candidata virtualmente indiscutível |
| 2 | Integridade da evidência | **A** — sem ela o produto não vale nada em juízo (REQ 14/15; Apêndice A; domínio legal) | **A** — exige mecanismos de hash/carimbo/metadados imutáveis na cadeia captura→relatório | **M** — implícita forte por domínio, sem requisito textual (análise, seção 5) | **M** — custo moderado (hash, armazenamento imutável), risco máximo | **11,0** | Coração da proposta de valor |
| 3 | Confiabilidade da captura | **A** — captura falha = créditos perdidos + evidência incompleta (REQ 08–11) | **A** — exige máquina de estados do registro, tratamento de falha de longa duração | **M** — "status" explícito no Apêndice A, semântica indefinida | **M** — custo moderado, risco alto | **11,0** | Par natural da integridade |
| 4 | Modularidade | **M** — protege evoluções D1/D2, mas não falha visível ao usuário no curto prazo | **A** — condiciona estilo e decomposição em componentes (entregáveis 3–6) | **M** — inferida dos domínios REQ 01–15 e Apêndice A | **A** — custo baixo se decidida cedo, alto se remendada depois | **11,0** | Escolha estrutural de base |
| 5 | Disponibilidade | **A** — janela de captura irrepetível; pagamentos em andamento (REQ 05/06; REQ 08) | **M** — influencia redundância e implantação, menos o desenho de componentes | **B** — sem metas no documento; pura inferência | **M** — custo de infraestrutura de redundância não trivial na v1 | **9,5** | Forte em risco, fraca em sustentação |
| 6 | Desempenho/capacidade | **M** — degradação percebida; gravações truncadas | **A** — define arquitetura de captura de vídeo (streaming, workers, armazenamento) | **B** — nenhuma meta numérica (lacuna registrada) | **M** — custo alto de provisionar sem meta definida | **9,5** | Depende de responder perguntas da análise |
| 7 | Usabilidade | **M** — abandono do funil de compra (REQ 05; Apêndice A) | **B** — quase não muda a estrutura; é problema de UX/telas | **B** — implícita apenas pelos fluxos em passos | **A** — custo baixo, risco moderado | **8,0** | Mais tática do que arquitetural |
| 8 | Escalabilidade | **M** — custo cresce com uso; degradação futura | **A** — define armazenamento de mídia e separação de cargas | **B** — nenhuma meta (inferência de crescimento de mídia) | **M** — custo alto se superdimensionada na v1 | **9,5** | Sobreposição parcial com #6 |
| 9 | Auditabilidade | **M** — disputas sem prova interna (REQ 05; Apêndice A) | **M** — exige log estruturado e trilhas, mas não redefine componentes | **M** — rastros parciais explícitos (usuário responsável, datas, ids) | **A** — custo baixo se prevista desde o início | **10,0** | Boa candidata "de base", barata se planejada |
| 10 | Privacidade/LGPD | **A** — sanções legais; CPF em 3 entidades (Apêndice A) | **M** — influencia criptografia e controles, reusa estrutura de segurança | **M** — inferida da legislação, apoiada nos campos coletados | **M** — custo moderado; parte herda de #1 | **10,5** | Forte candidata, parcialmente sobreposta com #1 |
| 11 | Interoperabilidade | **A** — receita depende de gateway/Pix; emails obrigatórios (REQ 05–07, 10, 11) | **M** — isola integrações em adaptadores; não reestrutura todo o sistema | **A** — necessidade explícita (REQ 06), forma inferida | **A** — custo baixo (bibliotecas padrão), risco só se mal isolada | **10,5** | Tratável como decisão de design de componentes |
| 12 | Observabilidade | **M** — incidentes cegos em pagamentos/capturas | **M** — exige instrumentação transversal, sem redefinir componentes | **B** — pura inferência da assincronia REQ 06–11 | **A** — custo baixo (logging/tracing padrão) | **9,0** | Companheira de #3 e #11, não dirigente |

### Leitura da tabela

- **Grupo "indiscutível"** (≥ 11): #1 Segurança, #2 Integridade da evidência, #3 Confiabilidade da captura, #4 Modularidade.
- **Grupo "quase lá"** (9,5–10,5): #10 Privacidade/LGPD, #11 Interoperabilidade, #9 Auditabilidade, #5 Disponibilidade, #6 Desempenho, #8 Escalabilidade.
- **Grupo "complementar"** (≤ 9): #12 Observabilidade, #7 Usabilidade.

Sobreposições relevantes que a equipe deve considerar: #10 herda boa parte da estrutura de #1; #6 e #8 se sobrepõem (capacidade vs. crescimento); #5 é consequência parcial de decisões de #3/#6.

---

## 3. Combinações possíveis de top 4 (com trade-offs)

Nenhuma combinação está fechada. Cada uma tem prós e contras; a escolha é da equipe e será registrada em ADRs.

### Combinação A — "Corrente da evidência" (recomendada para discussão)

**#1 Segurança, #2 Integridade da evidência, #3 Confiabilidade da captura, #4 Modularidade**

- **Prós:**
  - Cobre o ciclo de valor completo: acesso controlado (#1) → captura que não falha sem perceber (#3) → evidência que resiste a questionamento (#2) → estrutura que permite evoluir conforme D1/D2 (#4).
  - Melhor pontuação ponderada do conjunto (as quatro têm ≥ 11).
  - As quatro juntas definem diretamente o estilo e os componentes (entregáveis 3–6), pois a captura é o coração técnico do sistema (REQ 08–11).
- **Contras:**
  - Deixa de fora **Privacidade/LGPD (#10)**, apesar do CPF em três entidades (Apêndice A) — mitigável tratando privacidade como subparte da Segurança (#1), mas isso precisa ficar explícito na ADR.
  - Deixa de fora **Interoperabilidade (#11)**, embora pagamentos e emails sejam fluxos críticos (REQ 05–07, 10, 11) — mitigável tratando-a como decisão de design de componentes, não característica dirigente.
  - Três das quatro são implícitas (só #1 é explícita) — exigirá confirmação do engenheiro de requisitos (perguntas da análise).

### Combinação B — "Operação confiável"

**#1 Segurança, #2 Integridade da evidência, #3 Confiabilidade da captura, #11 Interoperabilidade**

- **Prós:**
  - Mantém o núcleo da evidência (#1–#3) e adiciona a característica explícita que sustenta a receita: pagamento via Mercado Pago/Pix e emails obrigatórios (REQ 05–07, 10, 11).
  - Mais defensável perante o engenheiro de requisitos, pois inclui duas características com base textual direta (#1 e #11).
  - A modularidade (#4) não desaparece: ela se materializa como consequência do isolamento das integrações (adaptadores), embora sem ADR própria.
- **Contras:**
  - Sem #4 no top 4, a decomposição em componentes e o estilo arquitetural perdem uma justificativa dirigente forte — a estrutura precisaria ser justificada só pelas outras três.
  - Evoluções futuras (empresas — D1; novos pacotes — D2) ficam sem característica de cobertura, aumentando risco de acoplamento.

### Combinação C — "Conformidade primeiro"

**#1 Segurança, #10 Privacidade/LGPD, #2 Integridade da evidência, #3 Confiabilidade da captura**

- **Prós:**
  - Máxima cobertura de risco legal: protege a conta (seção 3), os dados pessoais (CPF/faturamento — Apêndice A) e a evidência em si.
  - Aducação forte para o público advogado (D1), que valoriza conformidade; facilita responder às perguntas de retenção e dados da análise.
- **Contras:**
  - Deixa de fora **Modularidade (#4)**, enfraquecendo a justificativa de estilo/decomposição (entregáveis 3–6 ficam menos ancorados).
  - Deixa de fora **Interoperabilidade (#11)**, com o mesmo efeito da Combinação A — mitigável como decisão de design.
  - #10 e #1 se sobrepõem em estrutura (criptografia, controle de acesso), o que pode fazer as duas ADRs repetirem argumentos; risco de ADRs redundantes.

### Comparativo rápido

| Combinação | Top 4 | Força | Fragilidade |
|---|---|---|---|
| A | #1, #2, #3, #4 | Cobre o ciclo de valor + estrutura; melhor pontuação | LGPD e interoperabilidade ficam fora do dirigente |
| B | #1, #2, #3, #11 | Núcleo da evidência + receita; melhor sustentação textual | Estrutura arquitetural sem dirigente próprio (sem #4) |
| C | #1, #10, #2, #3 | Máxima cobertura de risco legal | Decomposição sem dirigente; sobreposição #1/#10 |

---

## Próximo passo

A equipe escolhe **uma** combinação (ou propõe ajustes, por exemplo, trocar #3 por #10). Após a escolha, escrevemos **um ADR por característica escolhida** (modelo do AGENTS.md), e as características não escolhidas continuam documentadas em `docs/caracteristicas.md` como acompanhamento — nenhuma é descartada por não estar no top 4.
