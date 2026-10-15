# ADR-001: Top 4 características arquiteturais prioritárias do Veridit

- **Título:** ADR-001: Priorizar Segurança, Integridade da evidência, Confiabilidade da captura e Modularidade
- **Status:** Proposta
- **Data:** 2026-10-13
- **Requisitos e restrições relacionados:**
  - **Segurança (autenticação e controle de acesso):** atributo de Segurança (seção 3 do documento de requisitos); REQ 01 (Cadastrar Usuário), REQ 03 (Logar no Sistema), REQ 04 (Sair do Sistema); campos "senha", "CPF" e "OAB" do Apêndice A.
  - **Integridade da evidência:** REQ 08 (Iniciar Registro de conteúdo), REQ 09 (Concluir gravação), REQ 11 (Concluir captura de tela), REQ 14 (Gerar Relatório), REQ 15 (Gerar Arquivo zip); campos "Registros realizados" e "Relatório do registro de conteúdo" do Apêndice A; D1 (perfis de usuário, incluindo advogados).
  - **Confiabilidade da captura:** REQ 08, REQ 09, REQ 10 (Confirmar gravação por email), REQ 11 e o REQ 11 duplicado (Confirmar captura de tela por email); campo "status" em "Registros realizados" e campos de data/hora de início e fim do Apêndice A; REQ 05 (Comprar Créditos, que os vincula "para realizar o registro de ocorrências").
  - **Modularidade:** D1 (tipos de usuário e escopo da v1); D2 (pacotes de créditos da v1); REQ 05–REQ 07 (domínio de pagamentos/créditos); REQ 08–REQ 11 (domínio de captura); REQ 12–REQ 15 (domínio de consulta/exportação); grupos de campos do Apêndice A.
  - **Restrição de escopo geral:** primeira versão delimitada por D1 (apenas advogados e usuários comuns) e D2 (pacotes básico, médio e premium).
  - Documentos de apoio: `docs/analise-requisitos.md`, `docs/caracteristicas.md`, `docs/priorizacao-caracteristicas.md` (Combinação A).

## Contexto

A etapa 2 identificou 12 características arquiteturais candidatas (`docs/caracteristicas.md`) e a etapa 3 avaliou todas com quatro critérios e apresentou três combinações possíveis de top 4 (`docs/priorizacao-caracteristicas.md`). É necessário escolher as características dirigentes que orientarão o estilo arquitetural, os componentes e as táticas. Nenhum prazo, orçamento ou meta de escala consta no documento de requisitos — avaliações de custo são inferências do contexto acadêmico do trabalho.

## Decisão

Adotar como características arquiteturais prioritárias do Veridit: **Segurança (autenticação e controle de acesso), Integridade da evidência, Confiabilidade da captura e Modularidade** (Combinação A — "Corrente da evidência").

As características não priorizadas continuam documentadas em `docs/caracteristicas.md` (ver seção "Características não priorizadas"); em especial, Privacidade/LGPD e Interoperabilidade serão tratadas como decisões de design derivadas (privacidade reusa a estrutura de Segurança; interoperabilidade vira isolamento de integrações em componentes), sem ADR dirigente própria nesta etapa.

## Características não priorizadas

| Característica candidata | Motivo de não priorizar |
|---|---|
| Privacidade / proteção de dados (LGPD) | Forte em risco legal, mas **[inferência]**: nenhum requisito a menciona; parte significativa de sua estrutura (controle de acesso, proteção de credenciais) coincide com a Segurança priorizada. Reavaliar nas táticas. |
| Interoperabilidade (integrações externas) | Necessidade explícita (REQ 06: Mercado Pago/Pix; REQ 07, REQ 10 e o REQ 11 duplicado: emails), mas é tratável como decisão de design de componentes (adaptadores), sem dirigir o estilo arquitetural. |
| Auditabilidade | Rastros parciais explícitos no Apêndice A (usuário responsável, datas, ids), mas o documento não exige trilha completa; custo baixo se prevista nas táticas de integridade/confiabilidade. |
| Desempenho / capacidade da captura | Nenhuma meta numérica no documento (lacuna registrada na análise); depende de responder ao engenheiro como a captura será executada (navegador do usuário vs. plataforma). Acompanhar via perguntas da análise. |
| Disponibilidade | Forte em risco (janela de captura irrepetível), mas pura inferência — sem metas no documento; custo de redundância não trivial na v1 (D1/D2). |
| Escalabilidade (da captura e de armazenamento) | Sem qualquer meta de volume ou usuários simultâneos no documento; dedicar um posto dirigente seria sobre-engenharia para o escopo da v1 (D1/D2). Acompanhar como consequência das táticas de confiabilidade e desempenho. |
| Usabilidade | Implícita apenas pelos fluxos em passos (REQ 05; Apêndice A); mais atributo de UX do que arquitetural; tratada na definição de telas (entregável 9). |
| Observabilidade | Pura inferência da assincronia (REQ 06–11); custo baixo, mas sem risco de negócio sustentado no documento; tratada como prática de engenharia. |

## Alternativas consideradas

**Combinação B — "Operação confiável" (Segurança, Integridade da evidência, Confiabilidade da captura, Interoperabilidade):**
- Prós: núcleo da evidência + receita; melhor sustentação textual (duas características explícitas: Segurança — seção 3; Interoperabilidade — REQ 06).
- Contras: decomposição e estilo ficam sem justificativa dirigente própria (sem Modularidade); evoluções de D1/D2 perdem cobertura.

**Combinação C — "Conformidade primeiro" (Segurança, Privacidade/LGPD, Integridade da evidência, Confiabilidade da captura):**
- Prós: máxima cobertura de risco legal (conta, dados pessoais, evidência).
- Contras: deixa a estrutura arquitetural sem dirigente (Modularidade fora); sobreposição entre Segurança e Privacidade tende a gerar ADRs redundantes.

## Justificativa

### Critérios de priorização e pesos

A seleção aplicou quatro critérios, com pesos definidos pela equipe (cada característica foi avaliada de Baixa=1 a Alta=3 em cada critério; pontuação máxima ponderada = 12):

1. **Impacto no negócio e no risco legal** (peso ×1,5): o quanto a característica protege a proposta de valor — evidência confiável para advogados e usuários (D1), receita por créditos (REQ 05) — e o quanto sua falha é irreversível.
2. **Influência na estrutura do sistema** (peso ×1,5): o quanto priorizá-la muda decisões estruturais (estilo, decomposição, mecanismos).
3. **Sustentação documental** (peso ×1): o quanto está apoiada no documento de requisitos — explícita pontua mais que inferida.
4. **Custo de não-atender versus custo de atender** (peso ×1): equilíbrio entre o risco de esquecê-la nas decisões e o custo de atendê-la dentro do escopo da v1 (D1/D2).

As quatro características escolhidas obtiveram as maiores pontuações ponderadas dessa avaliação (detalhes por característica em `docs/priorizacao-caracteristicas.md`) e, qualitativamente, cobrem o ciclo de valor completo: acesso controlado, captura que não falha silenciosamente, evidência que resiste a questionamento e estrutura preparada para as evoluções previstas. Justificativas por característica a seguir.

### Segurança (autenticação e controle de acesso)

É o único atributo de qualidade nomeado no documento de requisitos: a seção 3 exige "acesso ao sistema apenas por usuários cadastrados". Os requisitos REQ 01 (cadastro), REQ 03 (log-in) e REQ 04 (log-out) formam o ciclo completo de autenticação, e REQ 02 (recuperar senha) completa a gestão de credenciais. O campo "senha" aparece em ambos os perfis da v1 — Advogado e Usuário comum (Apêndice A, com D1) — confirmando o caráter de credencial.

A priorização segue diretamente do peso maior do critério de impacto: uma falha de acesso expõe registros capturados de terceiros, que podem ser provas de casos reais — o Apêndice A inclui CPF no relatório do registro e nos perfis de usuário, dados que não podem vazar. Não há inferência relevante aqui: tudo está no texto do documento.

### Integridade da evidência

O produto final do sistema é o relatório (REQ 14) e o arquivo zip (REQ 15) com o conteúdo capturado (REQ 08–REQ 11). O Apêndice A define o relatório com campos tipicamente probatórios — CPF do usuário, id do registro, data/hora de início e fim, duração, url do site navegado, nome e tamanho do arquivo gerado — e o perfil Advogado (D1, Apêndice A) indica o uso legal dessas evidências.

**[inferência]** O documento nunca afirma explicitamente que o relatório terá valor probatório nem exige hash, carimbo de tempo ou cadeia de custódia; a característica é deduzida do domínio (advogados + campos do relatório). Mesmo como inferência, sua priorização é justificada porque, se a intenção probatória se confirmar (ver Consequências), a integridade precisará estar presente na arquitetura desde a captura — retrofitted é caro e o risco de negócio é o maior de todos (evidência invalidada).

### Confiabilidade da captura

O documento define um ciclo explícito de captura: REQ 08 (iniciar registro), REQ 09 (concluir gravação) e REQ 11 (concluir captura de tela), com confirmações por email (REQ 10 e o REQ 11 duplicado). O Apêndice A prevê o campo "status" em "Registros realizados" e data/hora de início e fim — indícios de que o ciclo tem estados intermediários que precisam ser gerenciados.

**[inferência]** O documento não define os valores do "status" nem o comportamento em falha (rede interrompida, aba fechada, timeout). A dedução é que a gravação de vídeo de navegação (REQ 08/REQ 09) é uma operação de longa duração vulnerável a falhas, e que o usuário paga créditos "para realizar o registro de ocorrências" (REQ 05) — logo, uma captura falha sem sinalização correta significa créditos perdidos e, pior, um relatório (REQ 14) com evidência incompleta que o usuário acredita estar íntegra.

### Modularidade

O documento organiza o sistema em quatro domínios funcionais distintos, cada um com entidades e campos próprios no Apêndice A: gestão de usuários (REQ 01–REQ 04), pagamentos e créditos (REQ 05–REQ 07, com D2 definindo três pacotes fixos na v1), captura/registro (REQ 08–REQ 11) e consulta/exportação (REQ 12–REQ 15). As decisões D1 e D2 indicam evolução futura desacoplada: empresas entrarão em versões posteriores (D1) e os meios de pagamento já variam dentro do próprio documento (REQ 06: Mercado Pago e Pix nativo).

**[inferência]** Em relação à justificativa por domínios e evoluções, a base documental é razoável, mas incompleta: o documento não afirma que os domínios devem evoluir em ritmos diferentes, nem define pontos de extensão. O que se infere é que a coexistência de domínios com entidades bem separadas (Apêndice A) e variações de escopo por versão (D1, D2) torna custoso deixá-los entrelaçados. A modularidade é priorizada como estrutura de defesa contra esse acoplamento, não como objetivo de negócio em si — e esta justificativa deve ser reavaliada com o engenheiro de requisitos caso ele indique que nenhuma evolução além de D1/D2 é esperada.

## Consequências

- **Positivas:** decisões de estilo arquitetural, componentes e táticas passam a ter quatro âncoras claras e complementares; Privacidade/LGPD e Interoperabilidade não são abandonadas, apenas tratadas como derivadas; alinhamento direto com o único atributo explícito do documento (Segurança).
- **Negativas / pendências de confirmação:**
  - **Integridade da evidência e Confiabilidade da captura dependem da confirmação do engenheiro de requisitos sobre o valor probatório do relatório** (Apêndice A: campos CPF, datas, url, arquivo gerado) e sobre a semântica do campo "status". As perguntas correspondentes estão na seção 5 de `docs/analise-requisitos.md`. Enquanto não confirmado, ambas permanecem característica dirigente com base **[inferência]**, e as ADRs subsequentes que as referenciarem devem manter status "Proposta".
  - Três das quatro características são implícitas (apenas Segurança é explícita); a ADR só deve migrar para "Aceita" após essas confirmações.
  - LGPD e interoperabilidade ficam fora do conjunto dirigente e podem ser subestimadas se não forem formalmente revisitadas nas etapas de táticas e ADRs.
