# ADR-005: Selagem das evidências no momento da captura

- **Título:** ADR-005: Calcular o hash de cada arquivo assim que é gerado, encadear os hashes e fechar o registro com um manifesto assinado pela plataforma e com carimbo de tempo
- **Status:** Proposta
- **Data:** 2026-10-15
- **Requisitos e restrições relacionados:**
  - REQ 09 e REQ 11 (conclusão da captura), REQ 14 (relatório), REQ 15 (zip).
  - Apêndice A, "Relatório do registro de conteúdo": usuário, CPF, id do registro, data/hora início e fim, duração, url do site navegado, nome e tamanho do arquivo gerado pelo sistema.
  - ADR-001 (Integridade da evidência, característica com base em **[inferência]**); ADR-002 (captura no servidor); ADR-004 (armazenamento imutável).

## Contexto

O relatório (REQ 14) e o zip (REQ 15) são o produto final do Veridit. Quem receber esses arquivos (um juiz, a parte contrária) precisa conseguir verificar que eles não foram alterados depois da captura. O armazenamento imutável (ADR-004) impede alterações dentro da plataforma, mas não ajuda depois que o usuário baixa o zip. Também é preciso provar **quando** a captura aconteceu.

O documento de requisitos não exige hash, assinatura ou carimbo de tempo. Esta ADR depende da Integridade da evidência, que é uma característica inferida (ADR-001).

## Decisão

1. **Hash na origem:** o C09 calcula o SHA-256 de cada imagem e de cada segmento de vídeo **dentro do serviço Captura, imediatamente após o arquivo ser gerado** e antes de ele ser guardado.
2. **Encadeamento:** cada arquivo registra também o hash do arquivo anterior do mesmo registro. Remover, inserir ou reordenar um arquivo quebra a cadeia.
3. **Manifesto:** na conclusão (REQ 09/REQ 11) é gerado um manifesto com o id do registro, o usuário, a url, a data/hora de início e fim e, para cada arquivo, nome, tamanho e hash (campos do Apêndice A).
4. **Assinatura:** o manifesto é assinado digitalmente com a chave privada da plataforma, guardada num cofre de chaves que só o serviço Captura acessa.
5. **Carimbo de tempo:** o hash do manifesto é enviado a uma Autoridade de Carimbo do Tempo, que devolve um carimbo assinado comprovando que o manifesto existia naquele momento **[inferência: o valor jurídico depende de confirmação do engenheiro de requisitos]**.
6. O manifesto assinado e o carimbo entram no zip (REQ 15), e o relatório (REQ 14) informa o hash de cada arquivo e do manifesto, para que qualquer pessoa possa conferir.

## Alternativas consideradas

**A. Apenas armazenamento imutável, sem hash**
- Prós: nada a implementar além da ADR-004.
- Contras: depois do download, não há como provar que o arquivo é o mesmo que foi capturado.

**B. Hash calculado só no fim da captura**
- Prós: mais simples (um cálculo por arquivo ao final).
- Contras: entre a geração e o fim, os arquivos ficam sem proteção; numa captura interrompida, os segmentos já gravados não teriam comprovação.

**C. Registro em blockchain pública**
- Prós: prova pública e descentralizada de existência.
- Contras: custo por transação, dependência de terceiros e complexidade desproporcionais para a v1; o carimbo de tempo atende ao mesmo objetivo de forma mais simples.

## Justificativa

- **Integridade da evidência:** o hash calculado na origem, num ambiente que o usuário não controla (ADR-002), cobre todo o caminho do arquivo. O encadeamento detecta remoção ou inserção de arquivos, e a assinatura impede que alguém gere um manifesto falso.
- **Verificação independente:** qualquer pessoa com o zip consegue recalcular os hashes e conferir com o manifesto e com o relatório, sem depender da plataforma.
- **Data e hora confiáveis:** o carimbo de tempo de uma autoridade externa reforça a data/hora de início e fim que o Apêndice A exige no relatório.

## Consequências

- **Positivas:**
  - O relatório e o zip podem ser verificados fora da plataforma.
  - Uma captura interrompida (ADR-009) mantém comprovação dos segmentos que já tinham sido selados.
- **Negativas:**
  - Dependência de uma Autoridade de Carimbo do Tempo externa (custo e disponibilidade). Se ela estiver fora do ar, o manifesto é assinado e o carimbo é pedido de novo depois.
  - **O relatório passa a incluir hashes, que não estão entre os campos do Apêndice A.** Isso precisa ser confirmado com o engenheiro de requisitos **[inferência]**.
  - A chave privada da plataforma vira um ativo crítico: se vazar, manifestos falsos podem ser assinados. Ela precisa de cofre, rotação e acesso restrito.
