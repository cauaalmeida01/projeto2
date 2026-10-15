# Táticas Arquiteturais — Veridit

> Etapa 8 da documentação arquitetural. Para cada característica do top 4 (ADR-001), as táticas que a arquitetura adota, com o objetivo de cada uma, como verificar se o objetivo foi atingido, onde ela é aplicada (componentes de `docs/componentes.md`) e a ADR relacionada. Os nomes seguem o catálogo de táticas de Bass, Clements e Kazman (*Software Architecture in Practice*). Inferências estão marcadas como **[inferência]**.

## 1. Segurança (autenticação e controle de acesso)

Base: seção 3 do documento de requisitos ("acesso apenas por usuários cadastrados"); REQ 01–04; ADR-006.

| Tática | Objetivo | Como verificar | Onde | ADR | Custo / trade-off |
|---|---|---|---|---|---|
| **Autenticar atores** | Só usuários cadastrados (REQ 01) acessam o sistema, por login com senha (REQ 03) | Toda rota não pública recusa requisição sem token válido (teste automatizado por rota) | C04, C02 | ADR-006 | Token de acesso curto obriga a renovações frequentes |
| **Autorizar atores** (verificação de posse) | Cada usuário só acessa os próprios registros, arquivos e compras ("usuário responsável pelo registro", Apêndice A) | Teste: usuário A pede um registro de B e recebe recusa | C10, C11, C12, C13, C06 | ADR-006 | Uma verificação extra em cada consulta |
| **Limitar acesso** (ponto único de entrada) | Os serviços não ficam expostos na internet; tudo passa pelo gateway | Varredura de rede: só o gateway responde por fora | C02 | ADR-003, ADR-006 | Gateway vira ponto único de falha (precisa de 2+ instâncias) |
| **Criptografar dados** | Senhas nunca são guardadas em texto; dados trafegam só por HTTPS | Senhas no banco só em hash lento (Argon2/bcrypt); nenhuma rota sem HTTPS | C04, C02 | ADR-006 | Hash lento deixa o login alguns milissegundos mais lento, de propósito |
| **Detectar e bloquear ataques** (limite de tentativas) | Dificultar a descoberta de senhas por tentativa e erro | Após N tentativas erradas, a conta fica bloqueada temporariamente | C04, C02 | ADR-006 | Um atacante pode bloquear temporariamente a conta de outra pessoa |
| **Revogar acesso** | O logout (REQ 04) encerra a sessão; o link de recuperação de senha (REQ 02) só funciona uma vez | Após logout, o token de renovação é recusado; um link já usado é recusado | C04 | ADR-006 | O token de acesso já emitido vale até expirar (poucos minutos) |
| **Separar entidades** (isolamento do navegador) | O site capturado não alcança a rede interna da plataforma nem outras sessões | Teste: tentar abrir um endereço interno no navegador remoto é bloqueado | C08 | ADR-002 | Configuração de rede específica para os contêineres de captura |

## 2. Integridade da evidência

Base: **[inferência]** domínio legal (D1, advogados) e campos do relatório (Apêndice A); REQ 14, REQ 15; ADR-002, ADR-004, ADR-005.

| Tática | Objetivo | Como verificar | Onde | ADR | Custo / trade-off |
|---|---|---|---|---|---|
| **Controlar o ambiente de origem** | O conteúdo é gerado num ambiente que o usuário não controla | Nenhum arquivo de evidência entra no sistema por upload do usuário | C08 | ADR-002 | Custo de infraestrutura por sessão |
| **Verificar integridade da mensagem** (hash na origem) | Qualquer alteração de um arquivo após a captura é detectável | Alterar 1 byte de um arquivo faz a verificação acusar divergência | C09 | ADR-005 | Cálculo de hash por arquivo (custo baixo) |
| **Encadeamento de hashes** | Detectar remoção, inserção ou reordenação de arquivos de um registro | Remover um segmento do zip faz a verificação da cadeia falhar | C09 | ADR-005 | Arquivos precisam ser selados em ordem |
| **Não repúdio** (assinatura digital do manifesto) | A plataforma comprova que gerou o manifesto; ninguém consegue forjar um | A assinatura do manifesto confere com a chave pública da plataforma | C09 | ADR-005 | A chave privada precisa de cofre e rotação |
| **Carimbo de tempo** | Comprovar por terceiro quando a captura aconteceu (data/hora do Apêndice A) | O carimbo da autoridade confere com o hash do manifesto | C09 | ADR-005 | Dependência externa **[inferência: precisa de confirmação]** |
| **Armazenamento imutável** | Impedir que a aplicação ou um operador alterem ou apaguem evidências | Pedido de sobrescrita ou exclusão no armazenamento é recusado | C10 | ADR-004 | Arquivos não podem ser apagados antes do fim da retenção |
| **Verificar na leitura** (auditoria contínua) | Nunca entregar ao usuário uma evidência corrompida sem aviso | Cada download recalcula o hash; varredura periódica não encontra divergências | C10 | ADR-004, ADR-005 | Leitura um pouco mais lenta em arquivos grandes |

## 3. Confiabilidade da captura

Base: REQ 08–11; campo "status" (Apêndice A); REQ 05 (créditos); ADR-002, ADR-003, ADR-008, ADR-009.

| Tática | Objetivo | Como verificar | Onde | ADR | Custo / trade-off |
|---|---|---|---|---|---|
| **Isolamento de falhas** (serviço próprio para a captura) | Uma falha na captura não derruba login, compra e consultas | Derrubar o serviço Captura: REQ 01–07 e REQ 12–15 continuam funcionando | Serviço Captura | ADR-003 | Mais uma unidade para implantar |
| **Sinal de vida (heartbeat)** | Detectar rapidamente um navegador travado ou desconectado | Matar um navegador: o registro vai para `INTERROMPIDO` em poucos segundos | C07, C08 | ADR-009 | Mensagens periódicas entre C08 e C07 |
| **Tempo limite (timeout)** | Nenhuma sessão fica presa indefinidamente ocupando recursos e crédito | Sessão parada além do limite é encerrada automaticamente | C07 | ADR-009 | Limites precisam ser definidos **[inferência]** |
| **Estado persistido** (máquina de estados no banco) | O status sempre reflete o que aconteceu, mesmo após reinício do serviço | Reiniciar o serviço no meio de uma captura: o registro termina em status coerente | C07 | ADR-009 | Mais escritas no banco |
| **Checkpoint** (gravação em segmentos) | Uma falha perde no máximo o segmento em andamento | Interromper uma gravação: os segmentos anteriores estão selados e disponíveis | C08, C09 | ADR-005, ADR-009 | Vídeo dividido em vários arquivos |
| **Transação com compensação** (reserva e estorno de crédito) | Crédito só é consumido por captura concluída | Captura interrompida: o saldo volta ao valor anterior | C07, C05 | ADR-009 | Lógica de reserva e estorno |
| **Repetição com idempotência** (caixa de saída) | Nenhum e-mail ou pedido de débito/estorno se perde, e nenhum é aplicado duas vezes | Derrubar o provedor de e-mail: o e-mail sai quando ele volta, uma única vez | Despachantes, C14, C05 | ADR-008 | Atraso de segundos a minutos no e-mail |
| **Controle de admissão** (limite de sessões simultâneas) | Não aceitar mais capturas do que a infraestrutura suporta, para não degradar as que estão em andamento | Acima do limite, a nova captura recebe "tente em instantes" e o crédito não é reservado | C08, C07 | ADR-002 | O usuário pode precisar esperar em horários de pico |

## 4. Modularidade

Base: domínios de REQ 01–15 e do Apêndice A; D1 e D2; ADR-003, ADR-004, ADR-007.

| Tática | Objetivo | Como verificar | Onde | ADR | Custo / trade-off |
|---|---|---|---|---|---|
| **Aumentar a coesão** (serviços por domínio) | Cada requisito é implementado dentro de um único serviço, sempre que possível | Na rastreabilidade (`componentes.md`, seção 3), cada REQ toca no máximo 2 serviços além da interface | 5 serviços | ADR-003 | Algumas operações exigem chamada entre serviços |
| **Encapsular** (interfaces explícitas) | Um serviço só é usado pela sua API publicada | Nenhum serviço importa código ou escreve em tabela de outro | Todos os serviços | ADR-003, ADR-004 | Mais contratos de API para manter |
| **Restringir dependências** (esquema por serviço) | Mudanças no banco de um domínio não quebram os outros | Permissões de banco: cada usuário de serviço só escreve no próprio esquema | Banco compartilhado | ADR-004 | Configuração de permissões no banco |
| **Usar um intermediário** (adaptadores de pagamento) | Novos meios de pagamento (REQ 06) entram sem mudar o fluxo de compra | Incluir um provedor fictício de teste sem alterar a classe de fluxo de compra | C06 | ADR-007 | Uma camada de abstração a mais |
| **Adiar a vinculação** (configuração em dados) | Novos pacotes (D2) e novos perfis de usuário (D1, empresas) entram sem mudança de código ou com mudança local | Criar um pacote novo só com inclusão no banco | C05, C03 | ADR-007 | Cadastro e validação dos dados de configuração |
| **Abstrair serviços comuns** (notificações) | E-mails de todos os domínios saem por um único serviço | Nenhum outro serviço fala diretamente com o provedor de e-mail | C14 | ADR-008 | Mais um serviço pequeno para implantar |

## 5. Características não priorizadas tratadas por táticas

A ADR-001 deixou estas características fora do top 4, mas pediu que fossem revisitadas aqui.

| Característica | Tática | Objetivo | Onde |
|---|---|---|---|
| Privacidade / LGPD **[inferência]** | Minimizar dados expostos; criptografar dados sensíveis guardados | CPF e dados de faturamento (Apêndice A) aparecem apenas onde o documento exige (cadastro, compra, relatório) e ficam criptografados no banco | C03, C06, C12 |
| Privacidade / LGPD **[inferência]** | Descartar dados da sessão de captura | Cookies, senhas digitadas em sites de terceiros e histórico do navegador remoto são destruídos com o contêiner (ADR-002) | C08 |
| Interoperabilidade | Isolar integrações externas atrás de adaptadores, com tempo limite e nova tentativa | Falha do Mercado Pago, do Pix ou do e-mail não afeta outras funções | C06, C14 |
| Disponibilidade **[inferência]** | Redundância ativa nos pontos únicos | Duas ou mais instâncias do gateway e dos serviços com estado no banco | C02, serviços |
| Observabilidade **[inferência]** | Registro estruturado e correlação de requisições | Cada requisição recebe um identificador que acompanha as chamadas entre serviços e os pedidos da caixa de saída | Todos |
