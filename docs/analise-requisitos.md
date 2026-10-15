# Análise do Documento de Requisitos — Veridit

> Análise preparada pela equipe de arquitetura com base em `docs/requisitos/veridit-req.md` (versão 01.00, 20/4/2026, RiSE Labs). Ainda NÃO propõe arquitetura. Toda afirmação cita o requisito (REQ xx), a decisão (D1/D2) ou o campo do Apêndice A que a sustenta. Inferências estão marcadas explicitamente como **[inferência]**.

## 1. Resumo do sistema (≤ 5 linhas)

O Veridit é um sistema web para **registro de conteúdo de páginas da internet como prova/evidência** (Apêndice A, "Registro de conteúdo" e "Relatório do registro"). Usuários — advogados e usuários comuns na primeira versão (D1) — se cadastrarão (REQ 01), compram pacotes de créditos (REQ 05, REQ 06, D2) e gastam créditos para capturar telas e vídeos de navegação de um site (REQ 08–REQ 11). Os registros podem ser listados (REQ 12), visualizados em detalhes (REQ 13), exportados como relatório (REQ 14) e como arquivo zip (REQ 15). Notificações por email acompanham compras e gravações (REQ 07, REQ 10, REQ 11 — ver duplicidade na seção 4).

## 2. Tabela de requisitos

| Nº | Nome | Prioridade | Implicação técnica (sustentada no documento) |
|---|---|---|---|
| REQ 01 | Cadastrar Usuário | Alta | Formulário de cadastro com os campos do Apêndice A por perfil: Advogado (nome completo, email, senha, CPF, OAB) e Usuário comum (nome completo, email, senha, CPF); Empresas citadas em D1 mas **fora do escopo da v1**. Implica modelagem de dois tipos de conta e validação de campos (CPF, OAB). |
| REQ 02 | Recuperar Senha | Média | Fluxo de redefinição de senha. Não especifica mecanismo (email? token?). **[inferência]** Exige comunicação com o usuário e token de redefinição seguro — coerente com o atributo de Segurança (seção 3). |
| REQ 03 | Logar no Sistema | Alta | Autenticação de usuários já cadastrados (REQ 01). Suportada pelo atributo de Segurança: "acesso apenas por usuários cadastrados" (seção 3). Implica sessão/token de autenticação. |
| REQ 04 | Sair do Sistema | Alta | Encerramento de sessão (log-out), complementar ao REQ 03. |
| REQ 05 | Comprar Créditos | Alta | Fluxo de compra: selecionar pacote → informar dados de faturamento → pagar. Pacotes definidos no Apêndice A ("Pacote de créditos": nome, quantidade de créditos, valor por crédito, benefícios) e restritos a básico/médio/premium na v1 (D2). Implica catálogo de pacotes, formulário de faturamento (campos do Apêndice A) e integração de pagamento (REQ 06). |
| REQ 06 | Efetuar Pagamento de Créditos | Alta | Dois meios de pagamento: Mercado Pago (integração externa) ou plataforma do Veridit com geração de QR Code / número Pix. Implica integração com gateway externo, geração de Pix **[inferência: código EMV/Pix Copia e Cola]** e recebimento/conciliação do pagamento **[inferência: conciliação]**. O documento não diz como o sistema saberá que o pagamento do QR/Pix foi concluído (webhook? polling?). |
| REQ 07 | Confirmar pagamento por email | Alta | Envio de email transacional após compra de créditos. Implica serviço de envio de email e **[inferência: gatilho associado à confirmação do pagamento, ligado a REQ 05/REQ 06]**. |
| REQ 08 | Iniciar Registro de conteúdo | Alta | Captura de conteúdo de um site "com base nos campos definidos no apêndice A" (Registro de conteúdo: endereço do site, título do registro). Conteúdo pode ser **tela de um site** (screenshot) ou **vídeo de navegação**. Implica captura de tela e/ou gravação de navegação no navegador, consumo de créditos **[inferência: REQ 05 diz que os créditos servem "para realizar o registro de ocorrências", mas não define quando/consumo]** e armazenamento de arquivos. |
| REQ 09 | Concluir gravação de conteúdo | Alta | Encerramento da gravação de vídeo iniciada no REQ 08. Implica gerenciamento do ciclo de vida da captura (início/fim, datas do Apêndice A). |
| REQ 10 | Confirmar gravação por email | Alta | Email transacional após conclusão da gravação (gatilho do REQ 09). |
| REQ 11 | Concluir captura de Tela | Alta | Encerramento da captura de tela (screenshot) — ver duplicidade de numeração na seção 4. Implica ciclo de vida separado da captura de tela em relação ao vídeo. |
| REQ 11 (duplicado) | Confirmar captura de tela por email | Alta | Email transacional após a captura de tela (gatilho do REQ 11). |
| REQ 12 | Listar Registros realizados | Alta | Listagem dos registros do usuário com os campos "Registros realizados" do Apêndice A (título, data/hora início/fim, detalhes, status). Implica consulta paginável **[inferência — paginação não é citada]** e associação registro↔usuário (Apêndice A: "usuário responsável pelo registro"). |
| REQ 13 | Visualizar Detalhes dos Registros | Alta | Tela de detalhes com os campos "Detalhes dos registros" do Apêndice A (id do sistema, título, datas, tipos de dados, nº de imagens e vídeos, usuário responsável). |
| REQ 14 | Gerar Relatório do Registro | Alta | Geração de relatório com os campos do Apêndice A ("Relatório do registro de conteúdo": título, ids, usuário, CPF, datas, duração, imagens capturadas, site da imagem, url do site navegado, nome e tamanho do arquivo). Implica geração de documento (formato não especificado — PDF presumível, **[inferência]**) e consulta aos dados da captura. |
| REQ 15 | Gerar Arquivo zip do Registro | Alta | Geração de arquivo zip para download com os arquivos capturados. Implica empacotamento dos mídia arquivos e download pelo usuário. |

## 3. Atores e principais fluxos de uso

### Atores

| Ator | Sustentação |
|---|---|
| **Usuário comum** | REQ 01 + D1 ("apenas advogados e usuários comuns serão cadastrados" na v1); campos no Apêndice A. |
| **Advogado** | REQ 01 + D1; campos no Apêndice A (CPF + OAB). |
| **Empresa** | Citada em D1, mas explicitamente **excluída da primeira versão**. |
| **Serviço de pagamento (Mercado Pago)** | REQ 06 ("através do mercado pago"). **[inferência]** é um sistema externo, não um usuário do Veridit. |
| **Serviço de email** | REQ 07, REQ 10, REQ 11 enviam emails (REQ 02 não menciona email). **[inferência]** provavelmente um serviço externo de email transacional. |

O documento não define perfis/permissões distintos entre advogado e usuário comum além dos campos de cadastro — qualquer diferença funcional é **[inferência]** e não está no documento.

### Fluxo principal: do cadastro até o download do relatório

Derivado dos REQ 01–REQ 15, na ordem em que o documento os apresenta:

1. **Cadastro** (REQ 01): usuário informa os campos do Apêndice A conforme seu perfil (advogado com OAB; usuário comum sem OAB).
2. **Login** (REQ 03): acesso somente para usuários cadastrados (seção 3, Segurança). Opcionalmente, recuperação de senha (REQ 02).
3. **Compra de créditos** (REQ 05): escolhe um dos pacotes básico/médio/premium (D2, Apêndice A), preenche os dados de faturamento (Apêndice A) e paga via Mercado Pago ou via QR Code/Pix gerado pela plataforma do Veridit (REQ 06).
4. **Confirmação por email** (REQ 07): usuário recebe email da compra.
5. **Início do registro** (REQ 08): usuário informa endereço do site e título do registro (Apêndice A) e inicia a captura — tela do site e/ou vídeo de navegação.
6. **Conclusão da captura** (REQ 09 para vídeo; REQ 11 para tela) com confirmações por email (REQ 10 e o REQ 11 duplicado).
7. **Consulta** (REQ 12, REQ 13): usuário lista seus registros e visualiza detalhes (id, datas, tipos de dados, quantidade de imagens/vídeos — Apêndice A).
8. **Exportação** (REQ 14, REQ 15): geração do relatório do registro e geração/Download do arquivo zip com os arquivos capturados.

**Nota:** o documento não especifica **quando os créditos são debitados** (na compra? ao iniciar cada registro?). Que um registro "consome" créditos é inferido de REQ 05 ("comprar créditos **para realizar o registro de ocorrências**"), mas a regra de consumo (1 crédito = 1 registro? por minuto? por tipo de captura?) não está no documento — ver perguntas na seção 5.

## 4. Ambiguidades, inconsistências e lacunas

### Inconsistências

1. **Nome do sistema divergente no título:** o documento se intitula "Veridt" (linha 1) enquanto o repositório e o contexto do trabalho usam "Veridit" (AGENTS.md). Divergência presente no documento original.
2. **REQ 11 numerado duas vezes** (linhas 86 e 92): "Concluir captura de Tela" e "Confirmar captura de tela por email" compartilham o mesmo número. Um deles deveria ser REQ 16 (ou a sequência REQ 08–REQ 16 deveria ser renumerada). Impacto: toda citação "REQ 11" é ambígua na documentação arquitetural.
3. **Relatório cita apenas imagens, mas REQ 08 permite vídeo:** os campos do "Relatório do registro de conteúdo" (Apêndice A) mencionam "imagens capturadas" e "site da imagem", sem qualquer menção a vídeos — embora REQ 08 permita "vídeo de navegação" e o campo "Detalhes dos registros" (Apêndice A) conte "número de imagens e vídeos". O relatório deve ou não contemplar vídeos?
4. **Duplicidade de "id" no relatório:** os campos do "Relatório do registro de conteúdo" (Apêndice A) listam "id gerado pelo sistema" e "id do registro gerado pelo sistema" sem diferenciar os dois — qual é o id de qual entidade?
5. **Campo "Detalhes" indefinido:** o grupo "Registros realizados" (Apêndice A) inclui o campo "Detalhes", sem definir seu conteúdo; os "Detalhes dos registros" (Apêndice A) são um grupo de campos separado — os dois se referem à mesma coisa?
6. **REQ 07: "compra" vs "confirmação de pagamento":** o requisito chama-se "Confirmar pagamento por email", mas o texto fala em "informar o usuário por email da **compra** dos créditos realizada". Com Pix (REQ 06), a solicitação de pagamento e a confirmação do pagamento ocorrem em momentos distintos — o email deve ser enviado na solicitação ou na confirmação efetiva?
7. **Vocabulário inconsistente sobre o que se captura:** REQ 08 fala em "captura de conteúdo" (tela **ou** vídeo de navegação), REQ 09 em "gravação de conteúdo", REQ 11 em "captura de tela". Não fica claro se são dois modos independentes de um mesmo fluxo ou dois fluxos paralelos, e se o REQ 11 se aplica ao modo tela e o REQ 09 ao modo vídeo.
8. **Título do REQ 12 diz "Listar Registros realizados"** com base nos campos "Registros realizados" do Apêndice A, que inclui "status" — mas nenhum requisito define os valores possíveis de status (ex.: "concluído", "em andamento", "falhou"?). **[inferência sobre a falta]**.

### Requisitos vagos

9. **REQ 02 (Recuperar Senha):** não especifica mecanismo (email, token, perguntas de segurança) nem prazo de validade do token. Vago demais para desenhar o componente de segurança.
10. **REQ 06:** não define **como o sistema detecta a conclusão** do pagamento por Pix (webhook, polling, confirmação manual), nem o comportamento em caso de expiração do QR Code.
11. **REQ 08:** não define limites de duração da gravação, formato dos arquivos (PNG? MP4?), resolução, nem se a captura é executada no navegador do usuário ou em um navegador servido pela plataforma (headless). Essa é a lacuna com maior impacto arquitetural.
12. **REQ 14:** não especifica o **formato do relatório** (PDF? HTML?) nem se é gerado sob demanda ou assíncronamente.
13. **REQ 15:** não diz se o zip é gerado sob demanda ou se há limite de tamanho, nem se fica disponível para re-download.
14. **Nenhum requisito define quando/quantos créditos são debitados** por registro, embora créditos existam "para realizar o registro de ocorrências" (REQ 05). Também não há requisito de **consultar saldo de créditos** ou histórico de compras.
15. **Nenhum requisito de administração** (moderação, cadastro de pacotes, gestão de usuários) — tudo sugere que os pacotes são fixos (D2), mas não está dito explicitamente quem os mantém.
16. **Nenhum requisito de cancelamento/reembolso** de compra ou de tratamento de falha de pagamento.

### Atributos de qualidade — lacuna principal

17. **A seção 3 lista apenas um atributo: Segurança** ("acesso apenas por usuários cadastrados"). Não há requisitos de desempenho, disponibilidade, confiabilidade, privacidade dos dados capturados, integridade da evidência (hash? cadeia de custódia?) — embora o produto pareça gerar **evidências legais** (relatório com CPF, datas e url — Apêndice A), o que normalmente exigiria integridade/tamper-resistance. Tudo isso é **[inferência]** sobre lacuna, não requisito existente.
18. **Não há requisitos não funcionais sobre volume/dados:** tamanho dos arquivos de vídeo, retenção dos registros, backup.

### Lacunas de escopo

19. **Empresas (D1):** decididas no documento como tipo de usuário, mas sem campos próprios no Apêndice A e sem requisitos específicos — coerente com "fora da primeira versão", mas o documento não afirma explicitamente que ficará para versões futuras.

## 5. Perguntas para o engenheiro de requisitos

1. **Captura (REQ 08–REQ 11):** a captura é feita no navegador do usuário (extensão/cliente) ou por navegadores na infraestrutura do Veridit? Quais formatos (PNG/JPEG, MP4/WebM)? Há limite de duração por gravação? Tela e vídeo são modos alternativos ou simultâneos? *(maior impacto arquitetural)*
   > **Decisão da equipe (15/10):** a captura será feita em navegador controlado pela plataforma (ADR-002). Tela e vídeo foram tratados como tipos alternativos escolhidos ao iniciar o registro (`docs/componentes.md`, seção 2) **[inferência, a confirmar]**.
2. **Créditos:** quando exatamente os créditos são debitados — por registro iniciado, por registro concluído, ou por tipo de captura (tela vs vídeo)? Existe saldo visível para o usuário e histórico de compras? O que acontece se o usuário não tem créditos suficientes?
3. **Pagamento:** como o Veridit detecta a confirmação do pagamento (webhook do Mercado Pago / do Pix? polling manual?). QR Code e Pix têm validade? O que ocorre em caso de pagamento parcial, falha ou estorno?
4. **Relatório (REQ 14):** formato desejado (PDF?) e se é gerado sob demanda ou em lote; o relatório precisa ter valor probatório (carimbo de data/hora, hash dos arquivos, cadeia de custódia)? *(conecta-se à inconsistência 3: o relatório cita apenas imagens, mas REQ 08 permite vídeo)*
5. **Numeração:** o REQ 11 aparece duas vezes (linhas 86 e 92). Qual dos dois deveria ser REQ 16, ou há outro requisito que faltou (ex.: "visualizar saldo de créditos")?
6. **Status dos registros** (Apêndice A): quais são os valores possíveis e quem os altera?
7. **Recuperar senha (REQ 02):** qual mecanismo esperado (email com token? validade?)?
8. **Emails (REQ 07, REQ 10, REQ 11):** deve haver template/branding, reenvio em falha, e demais emails (boas-vindas, alteração de senha)?
9. **Empresas (D1):** confirmar que ficam fora da v1 e se já devemos prever extensão no modelo (por exemplo, cadastro multiusuário por empresa)?
10. **Perfis:** advogado e usuário comum têm comportamentos diferentes além dos campos de cadastro (OAB), ou a diferença é apenas cadastral?
11. **Atributos de qualidade:** além de Segurança, existem metas de disponibilidade, desempenho (ex.: tempo de geração do relatório/zip), volume esperado de gravações simultâneas e requisitos de privacidade/LGPD para os dados capturados (CPF no relatório — Apêndice A)?
12. **Retenção:** por quanto tempo os registros, arquivos e relatórios ficam disponíveis? Existe exclusão pelo usuário?
13. **Pacotes (D2):** os três pacotes (básico, médio, premium) diferem apenas em quantidade de créditos, ou também em benefícios funcionais ("descrição dos benefícios" do Apêndice A sugere que sim)?
14. **Download (REQ 15):** o zip pode ser gerado/rebaixado quantas vezes quiser, ou é único por registro?
15. **Nome:** o sistema é "Veridit" ou "Veridt" (linha 1)?
