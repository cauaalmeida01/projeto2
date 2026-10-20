# ADR-0005: Autenticar com token de acesso curto e refresh token revogável, com senhas em Argon2id

- **Status:** Aceita
- **Data:** 2026-10-15
- **Decisores:** Cauã Marinho, João Ferraz, Guilherme Loiola e Davi Araújo
- **Características arquiteturais relacionadas:** Segurança, Escalabilidade
- **Requisitos e restrições relacionados:** Atributo de qualidade "Segurança" (acesso só por usuários cadastrados); REQ01 (cadastro), REQ02 (recuperar senha), REQ03 (login), REQ04 (logout); D1 (advogado, usuário comum e, futuramente, empresa)

## Contexto

Vários serviços precisam saber quem é o usuário em cada requisição. Consultar o serviço de
Identidade a cada chamada cria acoplamento e um ponto único de falha. Ao mesmo tempo, o
logout (REQ04) precisa ter efeito real e a recuperação de senha (REQ02) é um vetor clássico
de ataque.

## Decisão

- O serviço de Identidade e Acesso emite um **token de acesso JWT** assinado com chave
  assimétrica, válido por **15 minutos**, com id do usuário e perfil (advogado, comum).
- Emite também um **refresh token** opaco e **rotativo**, guardado com hash no banco,
  válido por 7 dias e enviado em cookie `HttpOnly`, `Secure`, `SameSite=Strict`. Reuso de um
  refresh token já trocado revoga toda a família de tokens (indício de roubo).
- O **API Gateway** valida a assinatura do JWT com as chaves públicas (JWKS) e repassa a
  identidade; cada serviço autoriza pelo dono do recurso e pelo perfil.
- **Logout (REQ04):** revoga o refresh token e coloca o `jti` do token de acesso em uma
  lista de revogação no cache do gateway até ele expirar.
- **Senhas:** hash com **Argon2id**; política mínima de senha; verificação de e-mail no cadastro.
- **Recuperação de senha (REQ02):** token aleatório de uso único (256 bits), guardado com
  hash, válido por 30 min, enviado por e-mail; a resposta não revela se o e-mail existe; após
  a troca, todas as sessões são revogadas.
- **Proteção contra força bruta:** limite de tentativas por IP e por conta, com bloqueio
  progressivo.

## Alternativas consideradas

| Alternativa | Prós | Contras |
|---|---|---|
| Sessão no servidor (cookie + armazenamento de sessão) | Revogação imediata; simples | Gateway ou serviços precisam consultar o armazenamento de sessão a cada requisição |
| JWT de longa duração sem refresh | Muito simples | Logout não tem efeito real; token roubado vale por muito tempo |
| Provedor de identidade externo (Keycloak, Auth0, Cognito) | Maduro; MFA e SSO prontos | Custo e operação adicionais; considerado para quando houver contas de empresa (D1) |
| bcrypt em vez de Argon2id | Amplamente suportado | Argon2id é a recomendação atual da OWASP, por resistir melhor a ataques com GPU |

## Justificativa

Tokens curtos permitem que o gateway valide a identidade localmente, sem chamada extra, o
que escala com o número de serviços. O refresh token rotativo e a lista de revogação dão ao
logout efeito imediato. Argon2id, tokens de recuperação de uso único e limitação de
tentativas cobrem os ataques mais comuns contra login e recuperação de senha.

## Consequências

**Positivas**

- Validação de identidade sem estado nos serviços.
- Logout efetivo e detecção de roubo de refresh token.

**Negativas / trade-offs**

- Gestão e rotação de chaves de assinatura.
- A lista de revogação no gateway adiciona um pequeno estado (cache).

**Riscos**

- Vazamento da chave privada de assinatura. Mitigação: chave em KMS e rotação periódica.

## Conformidade

- Função de aptidão no CI: varre a especificação OpenAPI e verifica que todo endpoint não
  público responde 401 sem token.
- Teste de autorização: acessar registro de outro usuário retorna 404.
- Análise estática (SAST) e de dependências (SCA) no pipeline.

## Notas

- ADRs relacionadas: 0001, 0002.
- Elaborada em sala com apoio de ferramenta de IA; registro da interação em <local>.
