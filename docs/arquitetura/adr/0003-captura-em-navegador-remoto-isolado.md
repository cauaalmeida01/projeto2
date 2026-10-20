# ADR-0003: Realizar a captura em navegador remoto isolado, executado no servidor

- **Status:** Aceita
- **Data:** 2026-10-15
- **Decisores:** Cauã Marinho, João Ferraz, Guilherme Loiola e Davi Araújo
- **Características arquiteturais relacionadas:** Integridade e autenticidade das evidências, Segurança, Escalabilidade
- **Requisitos e restrições relacionados:** REQ08 (iniciar registro: tela ou vídeo de navegação), REQ09 (concluir gravação), REQ11 (concluir captura de tela), REQ14 (relatório com URL e site)

## Contexto

O usuário informa o endereço do site e o título do registro e pode capturar uma tela ou
gravar um vídeo da navegação (REQ08). Se a captura acontecer no dispositivo do usuário
(extensão de navegador, print local ou aplicativo), o próprio usuário pode alterar a página
antes de capturar (DevTools, edição do HTML, arquivo hosts, imagem editada). Nesse caso a
plataforma não consegue afirmar que o conteúdo estava de fato publicado no site.

Além disso, abrir sites arbitrários é arriscado: a página pode conter código malicioso.

## Decisão

A captura será feita **no servidor**, por workers do serviço de Captura:

- Cada sessão roda em um **contêiner efêmero** com navegador Chromium controlado por
  automação (ex.: Playwright).
- A tela do navegador remoto é transmitida para a aplicação web, e os cliques e teclas do
  usuário são repassados ao navegador por WebSocket. O usuário navega, mas o conteúdo é
  renderizado e capturado no ambiente da plataforma.
- Capturas de tela e gravação de vídeo são produzidas pelo worker, junto com **metadados
  técnicos**: URL final, IP do servidor de destino, resolução DNS, cabeçalhos HTTP,
  certificado TLS do site e horário sincronizado por NTP.
- O contêiner é destruído ao final, sem cookies ou cache persistentes, e roda em rede isolada,
  sem acesso à rede interna da plataforma.
- Limites por sessão: duração máxima (ex.: 30 min) e número de sessões simultâneas por usuário.

## Alternativas consideradas

| Alternativa | Prós | Contras |
|---|---|---|
| Extensão de navegador no computador do usuário | Barata; sem custo de servidor; navegação fluida | Ambiente não confiável: o usuário controla o DOM e a rede; evidência facilmente contestável |
| Aplicativo desktop de captura | Mais controle que a extensão | Continua sendo ambiente do usuário; custo de distribuição e atualização |
| Captura automática só pela URL, sem interação | Simples; fácil de escalar | Não atende "vídeo de navegação"; não alcança conteúdo que exige rolagem, cliques ou login |
| API de screenshot de terceiros | Rápida de integrar | Cadeia de custódia fora do controle da plataforma; dependência externa; não grava navegação |

## Justificativa

Capturar em ambiente controlado pela plataforma é o que permite afirmar como, quando e de
onde o conteúdo foi obtido, que é a base da integridade e da autenticidade (ADR-0001). O
isolamento em contêiner efêmero atende à segurança, porque o conteúdo externo nunca é
executado perto dos dados sensíveis. O modelo de workers sem estado é o que permite escalar
a captura de forma independente (ADR-0009).

## Consequências

**Positivas**

- Evidência com maior força probatória; ambiente padronizado e reproduzível.
- Metadados técnicos coletados automaticamente para o relatório (REQ14).

**Negativas / trade-offs**

- Custo de infraestrutura por sessão (CPU, memória e banda).
- Latência na interação perceptível para o usuário.
- Complexidade do streaming da tela.

**Riscos**

- Sites que bloqueiam IPs de datacenter. Mitigação: pool de saída de rede com IPs
  identificáveis e documentados no relatório.
- O usuário pode digitar credenciais pessoais (ex.: rede social) dentro da sessão remota.
  Mitigação: aviso na interface; campos de senha não aparecem no vídeo (são mascarados pelo
  próprio navegador); a sessão é destruída ao final.

## Conformidade

- Teste automatizado de política de rede: o contêiner de captura não alcança endereços da
  rede interna.
- O serviço de Evidências só aceita artefatos publicados por workers autenticados (identidade
  do worker via mTLS).

## Notas

- ADRs relacionadas: 0001, 0004, 0009.
- Elaborada em sala com apoio de ferramenta de IA; registro da interação em <local>.
