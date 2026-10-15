# ADR-002: Captura executada em navegador controlado pela plataforma

- **Título:** ADR-002: Executar a captura de conteúdo em um navegador isolado, na infraestrutura do Veridit, controlado remotamente pelo usuário
- **Status:** Proposta
- **Data:** 2026-10-15
- **Requisitos e restrições relacionados:**
  - REQ 08 (Iniciar Registro de conteúdo: "captura de conteúdo através de um site da internet"; "tela de um site ou vídeo de navegação"), REQ 09 (Concluir gravação), REQ 11 (Concluir captura de tela), REQ 10 e REQ 11 duplicado (confirmações por e-mail), REQ 14 (Relatório), REQ 15 (Arquivo zip).
  - Apêndice A: "Registro de conteúdo" (endereço do site, título); "Relatório do registro de conteúdo" (url do site navegado, data/hora início e fim, duração, imagens capturadas, nome e tamanho do arquivo gerado pelo sistema).
  - ADR-001: Segurança, Integridade da evidência e Confiabilidade da captura.
  - Lacuna registrada em `docs/analise-requisitos.md` (seção 4, item 11; seção 5, pergunta 1).

## Contexto

O REQ 08 exige que o usuário capture o conteúdo de um site como imagem da tela ou como vídeo de navegação, mas o documento de requisitos não diz **onde** a captura acontece. A análise de requisitos apontou essa lacuna como a de maior impacto arquitetural: dependendo da resposta, a captura é uma operação leve (o servidor só recebe arquivos) ou pesada (o servidor executa navegadores e grava vídeo).

Ao mesmo tempo, a ADR-001 colocou Integridade da evidência e Segurança no topo das prioridades. O produto final do Veridit é um relatório (REQ 14) e um pacote de arquivos (REQ 15) que o usuário, em especial o advogado (D1), vai apresentar como registro do que estava publicado no site. Se o arquivo puder ser modificado entre a captura e o armazenamento, o registro perde valor **[inferência: o documento não exige valor probatório, ver ADR-001]**.

## Decisão

A captura será executada em um **navegador que roda na infraestrutura do Veridit**, criado para cada sessão de captura e descartado ao final:

1. Ao iniciar um registro (REQ 08), o usuário informa o endereço do site e o título (Apêndice A). O sistema abre um navegador novo e limpo (sem cookies, histórico ou extensões), isolado das demais sessões.
2. O usuário **vê e controla esse navegador pela aplicação web**: a imagem da tela é transmitida para ele e os cliques, a rolagem e o teclado são repassados ao navegador remoto.
3. A gravação do vídeo e as capturas de tela são feitas **no próprio servidor**. Os arquivos não passam pelo computador do usuário.
4. Ao concluir (REQ 09/REQ 11), os arquivos são selados e guardados (ADR-005), e o navegador é destruído junto com todos os dados da sessão.

## Alternativas consideradas

**A. Extensão no navegador do usuário**
- Prós: barata para a plataforma (o processamento fica no computador do usuário); usa a sessão que o usuário já tem aberta nos sites; interação sem atraso.
- Contras: o ambiente é controlado pelo usuário, que pode alterar a página (ex.: editar o HTML pelas ferramentas de desenvolvedor) ou os arquivos antes do envio, o que vai contra a Integridade da evidência; depende do navegador e do sistema operacional de cada usuário (Confiabilidade); exige instalação.

**B. Upload de imagens e vídeos feitos pelo próprio usuário**
- Prós: implementação trivial.
- Contras: nenhuma garantia sobre a origem do arquivo. Equivale ao "print" comum que o produto pretende substituir.

**C. Captura automática no servidor, sem interação do usuário** (o servidor abre a URL e tira a imagem)
- Prós: ambiente totalmente controlado; mais simples que a decisão escolhida.
- Contras: não atende o "vídeo de navegação" do REQ 08, que pressupõe o usuário navegando pelo site; não alcança conteúdo que exige interação (rolar, abrir comentários, fazer login no site).

## Justificativa

- **Integridade da evidência:** o conteúdo é obtido diretamente do site pelo servidor da plataforma, e os arquivos são gerados e selados num ambiente que o usuário não consegue modificar. É a única alternativa em que a plataforma pode afirmar no relatório (Apêndice A) que aquele arquivo, com aquele nome e tamanho, foi gerado pelo sistema a partir daquela url, naquele intervalo de data e hora.
- **Segurança:** o navegador roda isolado e é descartado ao final, então nada de uma sessão vaza para outra. O computador do usuário não executa o conteúdo do site capturado, e nenhuma extensão com acesso amplo ao navegador do usuário precisa ser instalada.
- **Confiabilidade da captura:** a gravação não depende da máquina do usuário. Se a conexão dele cair, o que já foi gravado continua no servidor, e o status do registro (Apêndice A) pode refletir a interrupção (ADR-009).
- A alternativa C foi descartada por não atender o REQ 08, e as alternativas A e B por comprometerem a característica mais importante do top 4.

## Consequências

- **Positivas:**
  - O sistema controla todo o caminho do arquivo, da captura ao armazenamento, o que permite selar as evidências na origem (ADR-005).
  - A experiência é a mesma para qualquer usuário, sem instalação, independentemente do navegador ou do sistema operacional.
- **Negativas:**
  - **Custo de infraestrutura:** cada captura em andamento ocupa um navegador e processamento de vídeo no servidor. Será preciso limitar o número de sessões simultâneas (táticas de Confiabilidade em `docs/taticas.md`).
  - **Atraso na interação:** o usuário navega por uma imagem transmitida, com alguma latência (afeta a Usabilidade, que não está no top 4).
  - **Sites que bloqueiam acessos de servidores** ou pedem verificação contra robôs podem dificultar a captura **[inferência]**.
  - **Login em sites de terceiros:** para capturar conteúdo que exige login (ex.: redes sociais), o usuário digita a senha desse site no navegador remoto. Por isso a sessão precisa ser descartada por completo ao final e o sistema não pode guardar nada além dos arquivos da evidência **[inferência]**.
  - A captura passa a ter características muito diferentes do resto do sistema (pesada, longa, sujeita a falhas). Isso influencia diretamente a escolha do estilo arquitetural (ADR-003).
