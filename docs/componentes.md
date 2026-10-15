# Componentes do Veridit

> Etapas 5 e 6 da documentação arquitetural. Base: `docs/requisitos/veridit-req.md`, ADR-001 (top 4), ADR-002 (captura em navegador da plataforma) e ADR-003 (estilo baseado em serviços). Inferências estão marcadas como **[inferência]**. Diagramas em `docs/diagramas/`.

## 1. Visão geral

O sistema tem **14 componentes**, distribuídos em uma aplicação web, um API Gateway e cinco serviços de domínio (ADR-003). Os componentes dependem de três elementos de infraestrutura e de cinco sistemas externos.

| Agrupamento | Componentes |
|---|---|
| Interface | C01 Aplicação Web |
| Entrada | C02 API Gateway |
| Serviço Contas e Acesso | C03 Gestão de Usuários, **C04 Autenticação e Sessão** |
| Serviço Créditos e Pagamentos | C05 Catálogo e Carteira de Créditos, C06 Pagamentos |
| Serviço Captura | **C07 Orquestrador de Captura**, **C08 Ambiente de Navegação Controlada**, **C09 Selagem de Evidências** |
| Serviço Registros e Evidências | **C10 Repositório de Evidências**, C11 Consulta de Registros, C12 Gerador de Relatório, C13 Gerador de Pacote Zip |
| Serviço Notificações | C14 Notificações por E-mail |

Em **negrito**, os componentes críticos (seção 4).

**Infraestrutura:** banco relacional compartilhado com um esquema por serviço (ADR-004); armazenamento de objetos imutável para as evidências (ADR-004); cofre de chaves para a chave de assinatura da plataforma (ADR-005).

**Sistemas externos:** Mercado Pago (REQ 06); provedor Pix (REQ 06, "plataforma do Veridit" gerando QR Code ou número Pix); provedor de e-mail (REQ 07, 10, 11 duplicado); sites da internet capturados (REQ 08); Autoridade de Carimbo do Tempo **[inferência, ver ADR-005]**.

## 2. Componentes candidatos e responsabilidades

| Id | Componente | Responsabilidade | Requisitos | Características do top 4 afetadas |
|---|---|---|---|---|
| C01 | Aplicação Web | Interface única do usuário: formulários, listagens e o visualizador do navegador remoto durante a captura | REQ 01–15 | Segurança (não guarda credenciais além do token) |
| C02 | API Gateway | Ponto único de entrada: TLS, validação do token de acesso, roteamento para os serviços e limite de requisições | Segurança (seção 3) | Segurança, Modularidade |
| C03 | Gestão de Usuários | Cadastro de advogados (com OAB) e usuários comuns, validação dos campos do Apêndice A, dados de perfil | REQ 01, D1 | Segurança, Modularidade (D1: empresas no futuro) |
| C04 | Autenticação e Sessão | Login, logout, recuperação de senha, emissão e revogação de tokens, guarda das senhas em hash | REQ 02, 03, 04, Segurança | Segurança |
| C05 | Catálogo e Carteira de Créditos | Pacotes básico, médio e premium; saldo; reserva, débito e estorno de créditos | REQ 05, D2 | Confiabilidade (crédito só é consumido por captura concluída), Modularidade |
| C06 | Pagamentos | Dados de faturamento, criação da cobrança, adaptadores Mercado Pago e Pix, recebimento da confirmação | REQ 05, 06 (dispara o REQ 07) | Modularidade (novos meios de pagamento) |
| C07 | Orquestrador de Captura | Cria o registro, reserva o crédito, aloca o navegador, controla o status do registro do início ao fim, trata tempo limite e interrupção | REQ 08, 09, 11 | Confiabilidade, Integridade |
| C08 | Ambiente de Navegação Controlada | Executa navegadores isolados e descartáveis, transmite a tela ao usuário, repassa comandos, grava vídeo e tira capturas de tela | REQ 08, 09, 11 | Integridade, Segurança, Confiabilidade |
| C09 | Selagem de Evidências | Calcula o hash de cada arquivo no momento em que é gerado, monta e assina o manifesto do registro, obtém carimbo de tempo | REQ 14, 15 (base do relatório e do zip) | Integridade |
| C10 | Repositório de Evidências | Guarda arquivos e manifestos sem permitir alteração ou exclusão, verifica a integridade na leitura e só entrega ao dono | REQ 13, 14, 15 | Integridade, Segurança |
| C11 | Consulta de Registros | Lista os registros do usuário e mostra os detalhes, com os campos do Apêndice A | REQ 12, 13 | Segurança (só o dono vê) |
| C12 | Gerador de Relatório | Gera o relatório do registro com os campos do Apêndice A | REQ 14 | Integridade (relatório baseado nos dados selados) |
| C13 | Gerador de Pacote Zip | Monta o zip com os arquivos capturados (e o manifesto) para download | REQ 15 | Integridade |
| C14 | Notificações por E-mail | Recebe os pedidos de e-mail dos outros serviços (vindos das caixas de saída deles) e envia pelo provedor de e-mail, com nova tentativa em caso de falha | REQ 07, 10, REQ 11 duplicado | Confiabilidade (o e-mail não bloqueia a captura) |

### Observações sobre os requisitos

- **Modo do registro:** o REQ 08 diz que o conteúdo "pode ser uma tela de um site ou vídeo de navegação". Os componentes assumem que o usuário escolhe o tipo ao iniciar: em um registro do tipo **captura de tela**, ele navega e tira uma ou mais imagens e depois conclui (REQ 11); em um registro do tipo **vídeo de navegação**, a gravação vai do início até a conclusão (REQ 09). **[inferência: resolve a ambiguidade do item 7 da análise de requisitos]**
- **Recuperação de senha por e-mail:** o REQ 02 não diz como a senha é recuperada. Assume-se um link de uso único enviado por e-mail (C04 usando C14) **[inferência]**.
- **Consumo de créditos:** o documento não define quando o crédito é debitado (análise, seção 4, item 14). Assume-se reserva ao iniciar o registro e débito apenas na conclusão, com estorno se a captura falhar (ADR-009) **[inferência]**.

## 3. Rastreabilidade: requisito → componentes

| Requisito | Componentes |
|---|---|
| REQ 01 Cadastrar Usuário | C01, C02, C03 |
| REQ 02 Recuperar Senha | C01, C02, C04, C14 |
| REQ 03 Logar no Sistema | C01, C02, C04 |
| REQ 04 Sair do Sistema | C01, C02, C04 |
| REQ 05 Comprar Créditos | C01, C02, C05, C06 |
| REQ 06 Efetuar Pagamento | C01, C06 (Mercado Pago, provedor Pix) |
| REQ 07 Confirmar pagamento por e-mail | C06, C14 |
| REQ 08 Iniciar Registro de conteúdo | C01, C02, C05, C07, C08 |
| REQ 09 Concluir gravação | C07, C08, C09, C10 |
| REQ 10 Confirmar gravação por e-mail | C07, C14 |
| REQ 11 Concluir captura de tela | C07, C08, C09, C10 |
| REQ 11 (duplicado) Confirmar captura de tela por e-mail | C07, C14 |
| REQ 12 Listar Registros | C01, C11 |
| REQ 13 Visualizar Detalhes | C01, C10, C11 |
| REQ 14 Gerar Relatório | C01, C10, C12 |
| REQ 15 Gerar Arquivo zip | C01, C10, C13 |
| Atributo Segurança (seção 3) | C02, C03, C04, C10, C11 |
| D1 (advogados e usuários comuns; empresas no futuro) | C03 |
| D2 (pacotes básico, médio e premium) | C05 |

Todos os requisitos estão cobertos. Nenhum componente existe sem requisito ou decisão de origem.

## 4. Componentes críticos

### Critério

Um componente é crítico quando uma falha ou um defeito nele compromete diretamente uma característica do top 4 (ADR-001) de forma **difícil ou impossível de reverter**. Componentes cuja falha só causa indisponibilidade temporária, ou cujo resultado pode ser gerado de novo, não foram considerados críticos.

### Análise por componente

| Id | Componente | Se falhar ou tiver defeito... | Crítico? |
|---|---|---|---|
| C01 | Aplicação Web | O usuário fica sem interface, mas nenhum dado é perdido ou alterado | Não |
| C02 | API Gateway | O sistema fica inacessível (afeta Disponibilidade, fora do top 4). A validação apenas aplica a verificação definida em C04 e recusa por padrão | Não |
| C03 | Gestão de Usuários | Cadastros incorretos podem ser corrigidos | Não |
| **C04** | **Autenticação e Sessão** | **Um defeito permite que alguém acesse registros e evidências de outra pessoa. O vazamento não pode ser desfeito (Segurança)** | **Sim** |
| C05 | Catálogo e Carteira | Saldo incorreto é corrigível por conciliação com os pagamentos e as capturas | Não |
| C06 | Pagamentos | Compras ficam pendentes; a confirmação pode ser reprocessada | Não |
| **C07** | **Orquestrador de Captura** | **O status do registro deixa de refletir o que aconteceu: captura incompleta marcada como concluída, ou crédito consumido sem captura (Confiabilidade)** | **Sim** |
| **C08** | **Ambiente de Navegação Controlada** | **A navegação não pode ser repetida no mesmo estado: o que não foi gravado está perdido. Falta de isolamento entre sessões vaza dados de um usuário para outro (Confiabilidade, Integridade, Segurança)** | **Sim** |
| **C09** | **Selagem de Evidências** | **Arquivos guardados sem hash ou com manifesto incorreto não podem mais ser comprovados como originais (Integridade)** | **Sim** |
| **C10** | **Repositório de Evidências** | **Uma evidência alterada ou apagada não pode ser recuperada (Integridade, Segurança)** | **Sim** |
| C11 | Consulta de Registros | Apenas leitura; pode ser corrigido sem perda | Não |
| C12 | Gerador de Relatório | O relatório pode ser gerado de novo a partir dos dados selados | Não |
| C13 | Gerador de Pacote Zip | O zip pode ser gerado de novo a partir dos arquivos selados | Não |
| C14 | Notificações por E-mail | O e-mail pode ser reenviado; a captura não depende dele | Não |

**Resultado:** 5 críticos de 14 componentes = **35,7%** (o mínimo exigido é 30%).

## 5. Especificação interna dos componentes críticos

### C04 Autenticação e Sessão

**Objetivo:** garantir que somente usuários cadastrados acessem o sistema (seção 3) e que cada um acesse apenas o que é seu.

| Subcomponente | Responsabilidade |
|---|---|
| Verificador de Credenciais | Compara a senha informada com o hash guardado (algoritmo lento de hash de senha, ex.: Argon2 ou bcrypt); conta tentativas e bloqueia temporariamente após repetidas falhas |
| Emissor de Tokens | Emite um token de acesso de curta duração, assinado, com o id e o perfil do usuário (D1), e um token de renovação |
| Registro de Sessões | Guarda os tokens de renovação ativos; o logout (REQ 04) os revoga |
| Recuperação de Senha | Gera link de uso único com validade curta, pede o envio do e-mail a C14 e redefine a senha (REQ 02) **[inferência sobre o mecanismo]** |
| Publicador de Chaves | Disponibiliza a chave pública para que C02 valide os tokens sem consultar C04 a cada requisição |

**Interfaces oferecidas:** `POST /sessoes` (login, REQ 03); `DELETE /sessoes/atual` (logout, REQ 04); `POST /senhas/recuperacao` e `POST /senhas/redefinicao` (REQ 02); `GET /chaves` (chave pública para C02).
**Interfaces requeridas:** C03 (dados do usuário, no mesmo serviço); C14 (e-mail de recuperação); esquema `contas` do banco.
**Dados mantidos:** hash das senhas, tokens de renovação, tokens de recuperação, contagem de tentativas de login.

### C07 Orquestrador de Captura

**Objetivo:** fazer com que cada registro termine em um estado que corresponda ao que realmente aconteceu e que o crédito só seja consumido por uma captura concluída.

| Subcomponente | Responsabilidade |
|---|---|
| API de Registros | Recebe os comandos do usuário: iniciar (REQ 08), tirar captura de tela, concluir gravação (REQ 09), concluir captura de tela (REQ 11) |
| Máquina de Estados do Registro | Controla o campo "status" (Apêndice A): `INICIANDO → EM_ANDAMENTO → FINALIZANDO → CONCLUIDO`, ou `INTERROMPIDO` / `FALHOU`. Toda transição é gravada no banco antes de ser executada (ADR-009) |
| Gestor de Créditos da Captura | Reserva o crédito em C05 ao iniciar (chamada síncrona: sem reserva, a captura não começa); pede a confirmação do débito ou o estorno ao final, pela caixa de saída |
| Supervisor de Sessões | Acompanha o sinal de vida de cada navegador; aplica tempo limite de inatividade e duração máxima; marca o registro como `INTERROMPIDO` e preserva o que já foi gravado |
| Caixa de Saída | Tabela no esquema `captura` onde, na mesma transação que muda o status, são gravados os pedidos para outros serviços (débito ou estorno para C05; e-mail para C14, REQ 10 e REQ 11 duplicado). Um despachante os entrega com nova tentativa (ADR-008) |

**Interfaces oferecidas:** `POST /registros` (REQ 08); `POST /registros/{id}/capturas-de-tela`; `POST /registros/{id}/conclusao-gravacao` (REQ 09); `POST /registros/{id}/conclusao-captura` (REQ 11); `GET /registros/{id}/status`.
**Interfaces requeridas:** C05 (reservar, confirmar e estornar crédito); C08 (alocar, comandar e liberar navegador); C09 (selar arquivos e fechar manifesto); C10 (guardar evidências); C14 (pedidos de e-mail).
**Dados mantidos:** registro (id gerado pelo sistema, usuário, endereço do site, título, tipo, status, data/hora de início e fim, conforme o Apêndice A), histórico das transições de status e caixa de saída.

### C08 Ambiente de Navegação Controlada

**Objetivo:** executar a navegação num ambiente que o usuário não consegue adulterar, sem misturar sessões, e gravar tudo no servidor (ADR-002).

| Subcomponente | Responsabilidade |
|---|---|
| Pool de Navegadores Isolados | Mantém navegadores prontos, cada um num contêiner descartável, sem cookies nem extensões; impõe o limite de sessões simultâneas |
| Transmissor de Tela | Envia a imagem do navegador ao C01 em tempo real (ex.: WebRTC ou WebSocket) **[inferência sobre a tecnologia]** |
| Repassador de Comandos | Repassa cliques, rolagem e teclado ao navegador; bloqueia downloads, uploads e acesso à rede interna da plataforma |
| Gravador de Vídeo | Grava a tela do navegador em segmentos curtos, para que uma falha perca no máximo o segmento em andamento |
| Capturador de Tela | Gera as imagens (REQ 11) junto com a url e a data/hora do servidor no momento da captura ("site da imagem", "url do site navegado", Apêndice A) |

**Interfaces oferecidas (internas ao serviço Captura):** `alocar(registroId)`; `canalDeVisualizacao(sessao)`; `capturarTela()`; `iniciarGravacao()`; `encerrarGravacao()`; `liberar()`.
**Interfaces requeridas:** acesso de saída à internet pública (sites capturados); C09 (cada arquivo gerado é entregue para selagem antes de sair do ambiente).
**Dados mantidos:** nenhum dado persistente. Os arquivos ficam numa área temporária do contêiner até serem selados e guardados; o contêiner é destruído ao final.

### C09 Selagem de Evidências

**Objetivo:** permitir que qualquer alteração em um arquivo, depois de capturado, seja detectada (ADR-005).

| Subcomponente | Responsabilidade |
|---|---|
| Calculador de Hash | Calcula o SHA-256 de cada imagem e de cada segmento de vídeo assim que ele é fechado |
| Encadeador | Inclui no registro de cada arquivo o hash do arquivo anterior, formando uma cadeia: remover ou reordenar um arquivo quebra a cadeia |
| Montador de Manifesto | Monta o manifesto do registro: id do registro, usuário, url, data/hora de início e fim, e para cada arquivo nome, tamanho e hash (campos do Apêndice A) |
| Assinador | Assina o manifesto com a chave privada da plataforma, guardada no cofre de chaves |
| Carimbador de Tempo | Obtém carimbo de tempo de uma autoridade externa sobre o hash do manifesto **[inferência: depende de confirmação do engenheiro de requisitos, ver ADR-005]** |

**Interfaces oferecidas:** `selarArquivo(registroId, arquivo, metadados) → hash`; `fecharManifesto(registroId) → manifesto assinado`; `verificar(registroId) → íntegro | divergência`.
**Interfaces requeridas:** cofre de chaves; Autoridade de Carimbo do Tempo; C10.
**Dados mantidos:** nenhum além da chave no cofre; manifestos e hashes são guardados em C10.

### C10 Repositório de Evidências

**Objetivo:** guardar as evidências de modo que não possam ser alteradas nem apagadas pela aplicação e entregá-las apenas ao dono.

| Subcomponente | Responsabilidade |
|---|---|
| API Somente-Inclusão | Aceita a gravação de arquivos novos apenas do serviço Captura; não existe operação de alterar ou apagar |
| Armazenamento Imutável | Armazenamento de objetos com bloqueio de escrita (o próprio armazenamento recusa sobrescrita e exclusão durante o período de retenção) |
| Catálogo de Evidências | Metadados no esquema `registros`: arquivo, tipo (imagem ou vídeo), tamanho, hash, registro e usuário responsável |
| Verificador de Integridade | Recalcula o hash a cada leitura e em varredura periódica; marca e alerta qualquer divergência |
| Controle de Acesso por Dono | Só entrega arquivos do registro ao "usuário responsável pelo registro" (Apêndice A) |

**Interfaces oferecidas:** `armazenar(registroId, arquivo, hash)` (apenas para o serviço Captura); `listarArquivos(registroId)`; `obterArquivo(arquivoId)`; `obterManifesto(registroId)`.
**Interfaces requeridas:** armazenamento de objetos; esquema `registros` do banco.
**Dados mantidos:** imagens, vídeos, manifestos assinados e o catálogo de metadados.

## 6. Fluxo principal de uma captura

1. O usuário informa endereço e título na C01 (REQ 08). A requisição passa pelo C02, que valida o token.
2. C07 cria o registro com status `INICIANDO` e pede a reserva de crédito a C05.
3. C07 pede um navegador a C08. C08 abre o site e começa a transmitir a tela para a C01. O status passa a `EM_ANDAMENTO`.
4. O usuário navega. No modo vídeo, C08 grava em segmentos; no modo tela, gera uma imagem a cada pedido. Cada arquivo é entregue a C09, que calcula o hash e o encadeia, e é guardado em C10.
5. O usuário conclui (REQ 09 ou REQ 11). C07 passa o status a `FINALIZANDO`, C09 fecha e assina o manifesto, e C10 o guarda.
6. Numa única transação local (esquema `captura`), C07 marca o registro como `CONCLUIDO` e grava na caixa de saída do serviço dois pedidos: confirmar o débito do crédito reservado (para C05) e enviar o e-mail (para C14, REQ 10 ou REQ 11 duplicado). Os dois pedidos são entregues com nova tentativa até darem certo e processados de forma idempotente (ADR-008, ADR-009). Como o crédito já estava reservado desde o passo 2, ele não pode ser usado em outro registro nesse intervalo.
7. C08 destrói o navegador e a área temporária.
8. Se o navegador ou a conexão falhar no meio, o Supervisor de Sessões marca o registro como `INTERROMPIDO`, os segmentos já selados são preservados e o crédito é estornado (ADR-009).
