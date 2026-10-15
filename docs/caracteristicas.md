# Características Arquiteturais Candidatas — Veridit

> Etapa 2 da documentação arquitetural. Base: `docs/requisitos/veridit-req.md` (REQ 01–REQ 15, atributo de Segurança na seção 3, decisões D1/D2, Apêndice A) e `docs/analise-requisitos.md` (lacunas e inferências citadas). Ainda NÃO seleciona o top 4 — essa escolha será justificada por ADRs posteriormente. Inferências estão marcadas como **[inferência]**, conforme as regras do AGENTS.md.

## Convenção de classificação

- **Explícita:** o requisito ou atributo de qualidade está no documento de requisitos.
- **Implícita:** deduzida do domínio do sistema (registro de conteúdo da internet como evidência para uso legal) ou das lacunas documentadas na análise. Quando implícita, indicamos por que a dedução é razoável.

---

## Características candidatas

### 1. Segurança (autenticação e controle de acesso)

- **Definição:** garantir que apenas usuários autenticados (cadastrados via REQ 01) acessem o sistema e seus registros.
- **Classificação:** **explícita** — único atributo de qualidade listado na seção 3 ("acesso apenas por usuários cadastrados"); operacionalizado por REQ 01 (cadastro), REQ 03 (login) e REQ 04 (logout).
- **Justificativa:** é o único atributo de qualidade nomeado no documento, e os requisitos REQ 01, REQ 03 e REQ 04 formam o ciclo completo de autenticação (cadastro, log-in, log-out). O campo "senha" aparece nos perfis Advogado e Usuário comum (Apêndice A), reforçando a natureza de credencial de acesso. A recuperação de senha (REQ 02) fecha o ciclo da gestão de credenciais.
- **Risco se não atendida:** usuários não autorizados poderiam ver registros capturados de terceiros — que podem ser materiais de prova de casos reais de advogados (D1, Apêndice A: o relatório inclui CPF do usuário). Vazamento de evidências comprometeria a confiança no produto e poderia gerar responsabilidade legal para a organização.

### 2. Integridade da evidência (autenticidade e não-repúdio)

- **Definição:** garantir que o conteúdo capturado (tela/vídeo) e o relatório não sejam alterados sem detecção desde a captura até o download.
- **Classificação:** **implícita** — **[inferência]** o documento não menciona hash, cadeia de custódia ou carimbo de tempo. A dedução vem do domínio: o relatório do registro (Apêndice A) inclui CPF do usuário, datas/hora de início e fim, url do site navegado e nome/tamanho do arquivo gerado — estrutura típica de um documento para uso probatório, produzido por um sistema com usuários advogados (D1).
- **Justificativa:** o produto final do Veridit é o relatório (REQ 14) e o arquivo zip (REQ 15) com o conteúdo capturado (REQ 08–REQ 11). Se o propósito é uso como evidência (domínio inferido de D1 — advogados — e dos campos do relatório no Apêndice A), o valor do produto depende de que os arquivos e seus metadados (data/hora, url) sejam confiáveis e não-repudiáveis. A lacuna foi registrada na análise de requisitos (item de valor probatório, seção 5).
- **Risco se não atendida:** a evidência poderia ser questionada em juízo ("como sei que a página não foi alterada?"), invalidando o caso do advogado e destruindo a proposta de valor central do produto — o pior cenário de negócio possível.

### 3. Confiabilidade da captura (fidedignidade e completude da operação)

- **Definição:** assegurar que a captura iniciada (REQ 08) seja concluída sem perda de dados, e que o status final do registro reflita fielmente o que ocorreu.
- **Classificação:** **implícita** (com apoio parcial explícito) — **[inferência]** derivada do ciclo REQ 08 (iniciar) → REQ 09/REQ 11 (concluir) e do campo "status" em "Registros realizados" (Apêndice A). O campo "status" é explícito, mas os valores e a semântica de falha não estão definidos (análise, seção 4).
- **Justificativa:** uma gravação de vídeo de navegação (REQ 08, REQ 09) é uma operação de longa duração, vulnerável a falhas de rede, fechamento acidental de aba ou timeout. O usuário pagou créditos para realizar o registro (REQ 05) e recebe confirmação por email da conclusão (REQ 10 e REQ 11 duplicado). Se a captura falha parcialmente sem que o usuário saiba, o relatório (REQ 14) conterá evidência incompleta ou enganosa. É razoável **[inferência]** que o produto precise distinguir "concluído", "interrompido" e "falhou" via o campo "status" já previsto no Apêndice A.
- **Risco se não atendida:** o usuário perde créditos por uma captura falha que parece completa; o advogado leva ao caso uma evidência incompleta — dano financeiro (créditos) e dano reputacional/legal simultâneos.

### 4. Modularidade (separação entre captura, pagamentos e evidências)

- **Definição:** organizar o sistema em módulos coesos com acoplamento controlado entre domínios funcionais.
- **Classificação:** **implícita** — **[inferência]** derivada da existência de domínios funcionais claramente distintos no documento: gestão de usuários (REQ 01–REQ 04), pagamentos/créditos (REQ 05–REQ 07, D2), captura/registro (REQ 08–REQ 11) e consulta/exportação (REQ 12–REQ 15), cada um com entidades próprias no Apêndice A.
- **Justificativa:** o Apêndice A modela entidades com campos muito diferentes por domínio (dados de faturamento vs. detalhes de registros vs. relatório), e as decisões D1 e D2 delimitam variações por versão (empresas fora da v1; pacotes fixos). Isso indica que partes do sistema vão evoluir em ritmos diferentes — por exemplo, novos meios de pagamento (REQ 06 já prevê dois: Mercado Pago e Pix nativo) e novos tipos de usuário (D1). Modularidade reduz o custo dessas evoluções.
- **Risco se não atendida:** acoplamento entre pagamentos e captura dificultaria adicionar meios de pagamento ou tipos de usuário (D1, D2), aumentando custo e defeitos nas evoluções previstas.

### 5. Disponibilidade (do sistema e dos downloads)

- **Definição:** manter o sistema operacional quando o usuário precisa capturar conteúdo ou resgatar um registro/relatório já produzido.
- **Classificação:** **implícita** — **[inferência]** o documento não define metas de disponibilidade (lacuna registrada na análise, seção 4, itens sobre atributos de qualidade). A dedução vem do fluxo de uso: a captura (REQ 08) depende da janela em que o usuário está registrando o site, e o download (REQ 15) normalmente é necessário em um prazo ligado ao processo legal.
- **Justificativa:** a compra de créditos (REQ 05) é transacional e irreversível no momento do uso; se o sistema está indisponível durante a janela de captura, o valor do produto naquele momento é perdido. Para o advogado (D1), a indisponibilidade no momento da navegação a ser provada significa perder a evidência definitivamente — a navegação de um site não pode ser repetida no mesmo estado.
- **Risco se não atendida:** perda de janelas de captura irrepetíveis (páginas podem ser alteradas ou removidas) e frustração de pagamentos já realizados (REQ 05–REQ 07), gerando reembolsos e desconfiança.

### 6. Desempenho / capacidade (captura de vídeo e geração de zip/relatório)

- **Definição:** responder e processar as operações de captura e exportação dentro de tempos aceitáveis para o usuário, inclusive com gravações simultâneas.
- **Classificação:** **implícita** — **[inferência]** não há metas numéricas no documento (lacuna registrada na análise). Deduzida do tipo de dado: gravação de vídeo de navegação (REQ 08/REQ 09) e empacotamento zip de arquivos capturados (REQ 15) são operações intensivas em E/S, memória e rede.
- **Justificativa:** capturar tela e vídeo "de um site da internet" (REQ 08) em tempo real exige processamento contínuo; gerar o zip (REQ 15) e o relatório (REQ 14) envolve ler e empacotar os mídia arquivos do registro. Vários usuários gravando simultaneamente multiplicam a carga **[inferência — não há estimativa de volume no documento]**. O tamanho do arquivo é até um campo do relatório (Apêndice A), sinalizando que a dimensão dos artefatos importa para o produto.
- **Risco se não atendida:** gravações truncadas (molda-se ao risco de confiabilidade), downloads demorados demais para serem úteis em prazos de processo, e degradação geral percebida pelos clientes pagantes.

### 7. Usabilidade (fluxo de captura sem atrito)

- **Definição:** permitir que o usuário realize o ciclo cadastro → compra → captura → exportação com facilidade e sem erros.
- **Classificação:** **implícita** — **[inferência]** usabilidade não aparece como atributo no documento; a dedução vem da própria estrutura do fluxo em sequência de passos descrita em REQ 05 ("selecionar o pacote... informar os dados de faturamento e realizar o pagamento") e REQ 08–REQ 11 (iniciar/concluir captura), além da grande quantidade de campos de formulário no Apêndice A (dados de faturamento com 12 campos).
- **Justificativa:** o modelo de negócio depende de o usuário comprar créditos e usá-los para registrar conteúdo (REQ 05). Formulários longos de faturamento (Apêndice A) e operações de captura sensíveis a erro criam atrito direto sobre a receita e sobre a qualidade da evidência produzida. Embora usabilidade seja mais um atributo de UX do que arquitetural, ela influencia decisões como execução da captura no navegador do usuário vs. servido pela plataforma (ver análise, seção 5, pergunta 1).
- **Risco se não atendida:** abandono no meio do funil de compra (REQ 05) e registros mal feitos, reduzindo receita e qualidade da evidência.

### 8. Escalabilidade (de carga e de armazenamento)

- **Definição:** suportar crescimento no número de usuários, gravações simultâneas e volume de mídia armazenada sem degradar as características anteriores.
- **Classificação:** **implícita** — **[inferência]** nenhuma meta de escala está no documento (lacuna registrada). Deduzida da natureza dos dados: os registros acumulam imagens e vídeos ("número de imagens e vídeos" — Apêndice A) por usuário (D1: dois tipos de usuário já na v1), e o zip (REQ 15) precisa conter todos os arquivos da captura.
- **Justificativa:** o armazenamento cresce monotonicamente com o uso (cada registro gera mídia permanente até a eventual exclusão — que não está prevista em requisito algum, ver análise, seção 5, pergunta de retenção). A captura de vídeo em tempo real é a operação mais cara em recursos; ela deve escalar sem competir com as operações de consulta (REQ 12, REQ 13) e exportação (REQ 14, REQ 15).
- **Risco se não atendida:** custo de infraestrutura cresce mais rápido que a receita (créditos — REQ 05) ou o sistema se degrada conforme a base de registros cresce, afetando downloads antigos.

### 9. Auditabilidade (rastreabilidade das operações)

- **Definição:** registrar de forma recuperável quem fez o quê, quando, em cada registro de conteúdo e transação.
- **Classificação:** **implícita** — **[inferência]** o documento prevê rastros parciais explícitos: "usuário responsável pelo registro" (Apêndice A, Detalhes dos registros), Data/hora de início e fim (Apêndice A, em "Registros realizados" e no relatório), e o id do registro aparece em mais de um grupo de campos. A inferência é de que o domínio legal (advogados, D1) exige que esses rastros sejam completos e verificáveis.
- **Justificativa:** o relatório do registro (REQ 14, Apêndice A) precisa conectar CPF do usuário, id do registro, datas e o arquivo gerado; isso só é possível se o sistema mantiver metadados confiáveis de cada operação. Compras de créditos (REQ 05–REQ 07) igualmente geram obrigações de rastreio financeiro **[inferência — nenhum requisito fiscal está no documento]**.
- **Risco se não atendida:** disputas entre usuário e plataforma sobre pagamentos (REQ 05) ou sobre o conteúdo de uma captura não podem ser esclarecidas — dano legal e reputacional.

### 10. Privacidade / proteção de dados (LGPD)

- **Definição:** proteger os dados pessoais coletados (nome, email, CPF, OAB, dados de faturamento) e os conteúdos capturados.
- **Classificação:** **implícita** — **[inferência]** o Apêndice A prevê coleta explícita de CPF em três entidades (Advogado, Usuário comum, Dados da compra) e o relatório inclui CPF do usuário; o conteúdo capturado é de sites da internet e pode conter dados de terceiros. Nenhum requisito menciona LGPD ou privacidade — a característica é deduzida da legislação aplicável a sistemas brasileiros que tratam CPF.
- **Justificativa:** coletar e armazenar CPF, dados de endereço completos (12 campos de faturamento — Apêndice A) e mídia de navegação coloca o Veridit como controlador de dados pessoais sensíveis. Isso implica decisões arquiteturais de criptografia, controle de acesso fino (coerente com o atributo de Segurança explícito na seção 3) e eventual retenção/exclusão (pergunta de retenção na análise, seção 5).
- **Risco se não atendida:** sanções legais, bloqueio do produto e perda de confiança — especialmente grave porque o público-alvo inclui advogados (D1), que têm sensibilidade elevada a conformidade.

### 11. Interoperabilidade (integrações externas)

- **Definição:** integrar-se de forma robusta a sistemas externos: gateway de pagamento (Mercado Pago), Pix (QR Code) e serviço de email transacional.
- **Classificação:** **explícita quanto à necessidade, implícita quanto à forma** — REQ 06 exige pagamento "através do mercado pago ou pela plataforma do Veridit" com geração de QR Code/número Pix; REQ 02, REQ 07, REQ 10 e o REQ 11 duplicado exigem envio de emails (o serviço externo de email é inferência registrada na análise, seção 3).
- **Justificativa:** dois dos fluxos críticos dependem de terceiros: a receita (compra de créditos — REQ 05/REQ 06) e a comunicação com o usuário (emails — REQ 07, REQ 10, REQ 11 duplicado). Falhas, latências e mudanças de API desses terceiros afetam diretamente o sistema, o que exige isolar essas integrações atrás de interfaces próprias.
- **Risco se não atendida:** queda do serviço de pagamento bloqueia toda a receita (REQ 05); falhas de email deixam o usuário sem confirmações obrigatórias (REQ 07, REQ 10, REQ 11 duplicado), gerando chamados de suporte e desconfiança.

### 12. Observabilidade

- **Definição:** permitir que a equipe monitore a saúde das operações (capturas em curso, pagamentos, emails, jobs de exportação) e diagnostique falhas.
- **Classificação:** **implícita** — **[inferência]** nenhuma exigência no documento; deduzida de operações assíncronas distribuídas: pagamento por Pix com confirmação em momento posterior ao da solicitação (REQ 06, análise seção 4 item 6), gravações de longa duração (REQ 08/REQ 09) e emails disparados (REQ 07, REQ 10, REQ 11 duplicado).
- **Justificativa:** quando o usuário reclama que "paguei e não recebi créditos" ou "a gravação sumiu", a equipe precisa reconstruir o que aconteceu entre solicitação (REQ 06), confirmação (REQ 07) e captura (REQ 08–REQ 11). Sem observabilidade, a operação do produto fica cega exatamente nos pontos onde terceiros e assincronia dominam.
- **Risco se não atendida:** incidentes de pagamento/captura ficam indetectáveis e sem causa raiz, transformando falhas técnicas em disputas com clientes.

---

## Características consideradas e descartadas

### Descartada: Portabilidade

- **Definição considerada:** executar o sistema em múltiplas plataformas/ambientes.
- **Motivo do descarte:** o documento não menciona plataformas-alvo, dispositivos ou ambientes de implantação (REQ 08 fala apenas em "um site da internet", sem indicar cliente). Não há requisito nem decisão (D1/D2) que a sustente; incluí-la seria inventar requisito, o que as regras do AGENTS.md proíbem. Pode ser revisitada se a resposta sobre o mecanismo de captura (navegador do usuário vs. servido — pergunta 1 da análise) implicar cliente nativo.
- **Status:** descartada nesta etapa; pode retornar como decisão de tecnologia, não como característica dirigida.

### Descartada: Desempenho de resposta interativa em todas as telas

- **Definição considerada:** metas de latência para todas as operações de UI.
- **Motivo do descarte parcial:** o documento não define metas de desempenho (lacuna registrada na análise); tratamos desempenho apenas onde há sustentação técnica real (captura de vídeo e geração de zip/relatório — característica 6). Atribuir metas globais de latência sem base documental seria inferência sem sustentação prática.
- **Status:** subsumida pela característica 6 (Desempenho/capacidade), com escopo restrito ao que o domínio sustenta.

### Descartada: Modificabilidade/evolução como característica de primeira ordem

- **Definição considerada:** facilidade de modificar o sistema como objetivo próprio.
- **Motivo do descarte:** é um meio, não um fim de negócio no contexto do documento: as evoluções previstas (empresas futuras — D1; novos pacotes/meios de pagamento — D2, REQ 06) estão contempladas pela característica 4 (Modularidade), que já captura a necessidade com base nos requisitos. Manter as duas separadas dividiria a mesma justificativa.
- **Status:** subsumida pela característica 4.

### Descartada: Testabilidade como característica dirigente

- **Definição considerada:** facilidade de verificar o sistema automaticamente.
- **Motivo do descarte:** nenhum requisito do documento exige testabilidade (não há requisitos de homologação, conformidade regulatória de processo ou automação). É uma característica que a equipe deve buscar como boa prática de engenharia, mas não se sustenta como dirigente de arquitetura neste caso, pois não há risco de negócio documentado ou inferido forte o suficiente.
- **Status:** descartada como dirigente; tratada como prática de engenharia esperada.

### Descartada (por enquanto): Escalonabilidade internacional / i18n

- **Definição considerada:** suporte a múltiplos idiomas e moedas.
- **Motivo do descarte:** o documento é todo localizado no contexto brasileiro — Pix (REQ 06), CPF e OAB (Apêndice A), organização e universidade brasileiras. Nada sugere escopo internacional na v1 (D1, D2). i18n adicionaria custo a tudo sem sustentação documental.
- **Status:** descartada; reavaliar apenas se o público-alvo mudar.

---

## Resumo

| # | Característica | Classificação | Sustentação principal |
|---|---|---|---|
| 1 | Segurança (autenticação) | Explícita | Seção 3 (Segurança), REQ 01, REQ 03, REQ 04 |
| 2 | Integridade da evidência | Implícita | **[inferência]** Domínio legal; REQ 14/REQ 15; Apêndice A (relatório) |
| 3 | Confiabilidade da captura | Implícita (apoio parcial) | REQ 08–REQ 11; Apêndice A ("status") |
| 4 | Modularidade | Implícita | Domínios de REQ 01–REQ 15; D1, D2; Apêndice A |
| 5 | Disponibilidade | Implícita | **[inferência]** Janela de captura; REQ 08; REQ 05 |
| 6 | Desempenho/capacidade | Implícita | **[inferência]** REQ 08/REQ 09 (vídeo), REQ 14/REQ 15 (zip/relatório) |
| 7 | Usabilidade | Implícita | REQ 05 (fluxo em passos), Apêndice A (formulários) |
| 8 | Escalabilidade | Implícita | **[inferência]** Apêndice A (mídia), REQ 12–REQ 15 |
| 9 | Auditabilidade | Implícita (apoio parcial) | Apêndice A (usuário responsável, datas, ids) |
| 10 | Privacidade/LGPD | Implícita | **[inferência]** CPF e faturamento no Apêndice A; Seção 3 (Segurança) |
| 11 | Interoperabilidade | Explícita (necessidade) | REQ 06 (Mercado Pago/Pix); REQ 07, REQ 10, REQ 11 (emails) |
| 12 | Observabilidade | Implícita | **[inferência]** REQ 06/REQ 07 (assincronia), REQ 08–REQ 11 |

A seleção do **top 4** e suas justificativas por ADR serão feitas na etapa seguinte, conforme o plano de entregas do AGENTS.md.
