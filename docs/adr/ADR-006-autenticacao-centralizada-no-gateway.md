# ADR-006: Autenticação centralizada com tokens validados no API Gateway

- **Título:** ADR-006: O serviço Contas e Acesso emite tokens de acesso assinados e de curta duração; o API Gateway valida o token em toda requisição e os serviços verificam se o usuário é dono do recurso
- **Status:** Proposta
- **Data:** 2026-10-15
- **Requisitos e restrições relacionados:**
  - Atributo de qualidade Segurança (seção 3): "acesso ao sistema apenas por usuários cadastrados".
  - REQ 01 (cadastro), REQ 02 (recuperar senha), REQ 03 (login), REQ 04 (logout); campo "senha" do Apêndice A; D1 (perfis advogado e usuário comum).
  - Apêndice A, "Detalhes dos registros": "usuário responsável pelo registro".
  - ADR-001 (Segurança); ADR-003 (gateway e cinco serviços).

## Contexto

Com cinco serviços (ADR-003), cada requisição precisa ter a identidade do usuário verificada sem que cada serviço implemente a autenticação de um jeito. Além de "estar logado", o usuário só pode ver os próprios registros e evidências, que podem ser material de processos reais (ADR-001).

## Decisão

1. O **C04** (serviço Contas e Acesso) é o único que conhece senhas. Ele guarda as senhas com um algoritmo lento de hash de senha (Argon2 ou bcrypt) e, no login (REQ 03), emite:
   - um **token de acesso** assinado, válido por poucos minutos, com o id e o perfil do usuário (D1);
   - um **token de renovação**, guardado no servidor, que permite obter novos tokens de acesso.
2. O **API Gateway** valida a assinatura e a validade do token de acesso em **todas** as requisições, usando a chave pública publicada por C04. Requisições sem token válido são recusadas antes de chegar a qualquer serviço (exceto cadastro, login e recuperação de senha).
3. Os **serviços** verificam a **posse do recurso**: um registro, arquivo, relatório ou compra só é entregue se o id do usuário do token for o do dono (Apêndice A: "usuário responsável pelo registro").
4. O **logout** (REQ 04) revoga o token de renovação. Como o token de acesso dura poucos minutos, a sessão termina logo em seguida.
5. A **recuperação de senha** (REQ 02) usa um link de uso único, com validade curta, enviado por e-mail **[inferência sobre o mecanismo]**.
6. Os serviços ficam em rede privada; só o gateway recebe tráfego da internet, sempre por HTTPS.

## Alternativas consideradas

**A. Sessão guardada no servidor, consultada a cada requisição**
- Prós: revogação imediata no logout.
- Contras: toda requisição de todos os serviços consulta o serviço de contas, que vira gargalo e ponto único de falha.

**B. Cada serviço valida o login por conta própria**
- Prós: nenhum componente central.
- Contras: lógica de segurança duplicada em cinco lugares; um serviço que esqueça a validação abre uma brecha.

**C. Provedor de identidade externo (login social, ex.: Google)**
- Prós: menos código de autenticação.
- Contras: o Apêndice A exige cadastro com senha, CPF e OAB; dependência externa sem requisito que a justifique.

## Justificativa

- **Segurança:** uma única porta de entrada (gateway) e um único emissor de identidade (C04) reduzem as chances de um serviço ficar sem proteção. A verificação de posse em cada serviço impede que um usuário logado acesse o registro de outro.
- **Desempenho e disponibilidade:** o gateway valida o token com a chave pública, sem consultar C04 a cada requisição.
- **Modularidade:** quando as empresas entrarem (D1), novos perfis entram no token sem mudar os outros serviços.

## Consequências

- **Positivas:**
  - Regras de autenticação num só lugar; os serviços recebem a identidade já validada.
  - Fácil de testar: toda rota que não for pública precisa recusar requisições sem token.
- **Negativas:**
  - Depois do logout, um token de acesso já emitido continua válido até expirar (poucos minutos). É uma janela pequena e aceita.
  - A chave de assinatura de tokens precisa ser protegida e trocada periodicamente.
  - O gateway passa a ser um ponto único de falha e precisa de mais de uma instância.
