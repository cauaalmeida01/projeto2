# ADR-003: Estilo arquitetural baseado em serviços

- **Título:** ADR-003: Adotar a arquitetura baseada em serviços, com cinco serviços de domínio, interface web única, API Gateway e banco de dados compartilhado
- **Status:** Proposta
- **Data:** 2026-10-15
- **Requisitos e restrições relacionados:**
  - Domínios funcionais: REQ 01–04 (contas e acesso), REQ 05–07 (créditos e pagamentos), REQ 08–11 (captura), REQ 12–15 (registros e exportação); e-mails dos REQ 07, REQ 10 e REQ 11 duplicado.
  - Atributo de qualidade Segurança (seção 3); D1 (tipos de usuário); D2 (pacotes de créditos).
  - ADR-001 (top 4: Segurança, Integridade da evidência, Confiabilidade da captura, Modularidade); ADR-002 (captura em navegador da plataforma).
  - Comparação completa em `docs/comparacao-estilos.md` (inclusive a revisão de 15/10).

## Contexto

O estilo precisa atender as quatro características da ADR-001. A ADR-002 decidiu que a captura roda em navegadores na infraestrutura do Veridit. Com isso, o sistema passou a ter duas partes com características muito diferentes:

- **a captura** (REQ 08–11): pesada (navegador + gravação de vídeo por sessão), longa (minutos) e sujeita a falhas;
- **o restante** (REQ 01–07, REQ 12–15): operações curtas de cadastro, login, compra e consulta.

Na terminologia de Richards & Ford, são **quanta arquiteturais diferentes**: se ficarem na mesma unidade de implantação, uma falha da captura derruba também o login e a compra. Por outro lado, nenhum requisito define metas de escala ou volume (análise de requisitos, seção 4, item 17), e a equipe tem 5 pessoas, o que torna estilos muito distribuídos desproporcionais para a v1 (D1, D2) **[inferência sobre o contexto do trabalho]**.

## Decisão

Adotar a **arquitetura baseada em serviços** com a seguinte estrutura:

| Serviço | Responsabilidade | Requisitos |
|---|---|---|
| **Contas e Acesso** | Cadastro de advogados e usuários comuns, login, logout, recuperação de senha, emissão de tokens | REQ 01–04, Segurança, D1 |
| **Créditos e Pagamentos** | Pacotes, faturamento, pagamento por Mercado Pago ou Pix, saldo e débito de créditos | REQ 05–07, D2 |
| **Captura** | Sessões de navegador remoto, gravação de vídeo e de telas, selagem dos arquivos | REQ 08, 09, 11 |
| **Registros e Evidências** | Guarda das evidências, listagem, detalhes, relatório e zip | REQ 12–15 |
| **Notificações** | Envio dos e-mails de confirmação | REQ 07, 10, REQ 11 duplicado |

Elementos comuns:
- **Uma aplicação web única** para o usuário.
- **API Gateway** como ponto único de entrada: valida o token de acesso e encaminha a requisição ao serviço (ADR-006).
- **Um banco relacional compartilhado**, dividido em esquemas lógicos por serviço; cada serviço só escreve no próprio esquema (ADR-004).
- **Armazenamento de arquivos separado e imutável** para as evidências (ADR-004, ADR-005).
- Comunicação síncrona (HTTP/REST) entre gateway e serviços e, quando necessário, entre serviços; os e-mails são enviados de forma assíncrona (ADR-008).

## Alternativas consideradas

**Monólito modular**
- Prós: uma única implantação; o mais simples de construir, testar e operar; transações locais em tudo.
- Contras: os navegadores da captura (ADR-002) rodariam na mesma unidade que login e pagamento. Um travamento ou um pico de capturas afetaria todo o sistema, o que vai contra a Confiabilidade da captura. Para evitar isso seria preciso tirar a captura do monólito, chegando a uma arquitetura baseada em serviços.

**Orientado a eventos (com broker)**
- Prós: trata naturalmente operações assíncronas longas (captura, confirmação de Pix); log de eventos ajuda o rastreamento.
- Contras: exige um broker externo, idempotência em todos os consumidores e consistência eventual. O rastro de um registro (Apêndice A: usuário, datas, arquivos) ficaria espalhado em eventos, o que dificulta garantir a Integridade. O custo é maior do que o que a v1 justifica.

**Microsserviços**
- Prós: máximo isolamento e escala independente por serviço.
- Contras: banco por serviço e consistência eventual pioram a Integridade da evidência; operação muito mais complexa (muitos serviços, observabilidade distribuída), sem nenhuma meta de escala no documento que justifique.

## Justificativa

- **Confiabilidade da captura:** a captura é um serviço próprio, implantado e escalado separadamente. Se ele falhar, login, compra, listagem e download continuam funcionando, e o usuário consegue ver o status do registro (Apêndice A).
- **Integridade da evidência:** como o banco é compartilhado, as operações que precisam ser atômicas (ex.: concluir a captura, registrar os arquivos e confirmar o débito do crédito) ficam em transações locais, sem consistência eventual. A captura e a selagem ficam num serviço isolado, que nenhum outro serviço consegue alterar.
- **Segurança:** o gateway é a única porta de entrada e valida a identidade em todas as requisições (seção 3: "acesso apenas por usuários cadastrados"). Os serviços ficam em rede privada, sem acesso direto pela internet.
- **Modularidade:** o sistema é dividido por domínio, seguindo os grupos de requisitos e de entidades do Apêndice A. As evoluções previstas ficam contidas num único serviço: empresas (D1) em Contas e Acesso; novos pacotes ou meios de pagamento (D2, REQ 06) em Créditos e Pagamentos.
- **Custo:** cinco serviços de grão grosso, um banco e um gateway são viáveis para a equipe. É um meio-termo entre o monólito (que não isola a captura) e microsserviços (que trazem complexidade sem necessidade).

## Consequências

- **Positivas:**
  - A captura, que é a parte pesada e arriscada do sistema, fica isolada e pode ganhar mais máquinas sem mexer no resto.
  - Fronteiras claras entre os domínios facilitam dividir o trabalho entre os membros da equipe.
  - É possível migrar um serviço para banco próprio no futuro, se algum crescer muito.
- **Negativas:**
  - **Acoplamento pelo banco compartilhado:** mudar uma tabela pode afetar mais de um serviço. Mitigação: esquemas separados e regra de que cada serviço só escreve no próprio esquema (ADR-004).
  - **Chamadas pela rede entre serviços** (ex.: a Captura pede reserva de crédito a Créditos e Pagamentos). Elas podem falhar e precisam de tempo limite e tratamento de erro (táticas de Confiabilidade).
  - Há mais unidades para implantar e monitorar do que em um monólito.
  - O gateway passa a ser um ponto único de falha e precisa de pelo menos duas instâncias em produção **[inferência]**.
