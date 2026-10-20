# ADR-0004: Selar as evidências com hash, manifesto assinado, carimbo do tempo e armazenamento imutável

- **Status:** Aceita
- **Data:** 2026-10-15
- **Decisores:** Cauã Marinho, João Ferraz, Guilherme Loiola e Davi Araújo
- **Características arquiteturais relacionadas:** Integridade e autenticidade das evidências, Auditabilidade, Segurança
- **Requisitos e restrições relacionados:** REQ09, REQ11 (conclusão da captura), REQ13 (detalhes), REQ14 (relatório com nome e tamanho dos arquivos), REQ15 (.zip); Apêndice A (campos do relatório)

## Contexto

Para que o registro tenha valor como prova, é preciso demonstrar a terceiros (juiz, perito,
parte contrária) que os arquivos não foram alterados desde a captura e em que momento a
captura ocorreu. Guardar os arquivos e um hash no banco da própria plataforma não basta: quem
administra o banco poderia alterar os dois.

## Decisão

Ao fim de cada captura, o serviço de Evidências vai:

1. Calcular o **SHA-256** de cada arquivo capturado.
2. Montar um **manifesto** (JSON) com id do registro, usuário, URL, data/hora de início e
   fim, metadados técnicos e a lista de arquivos com nome, tamanho e hash.
3. **Assinar** o manifesto com a chave privada da plataforma, guardada em serviço de
   gerenciamento de chaves (KMS/HSM) e nunca exportada.
4. Obter um **carimbo do tempo** (RFC 3161) de uma Autoridade de Carimbo do Tempo credenciada
   na ICP-Brasil sobre o hash do manifesto.
5. Gravar arquivos, manifesto, assinatura e carimbo em **armazenamento de objetos com
   bloqueio de escrita (WORM / Object Lock)** pelo prazo de retenção.
6. Registrar cada passo na **cadeia de custódia** (trilha de auditoria com encadeamento de
   hashes).

O relatório (REQ14) inclui os hashes e instruções de verificação. O .zip (REQ15) contém os
arquivos, o manifesto, a assinatura, o carimbo e o relatório, permitindo verificação offline.

O registro só muda para "Concluído" depois de selado.

## Alternativas consideradas

| Alternativa | Prós | Contras |
|---|---|---|
| Apenas hash guardado no banco | Simples e sem custo | Quem controla o banco altera arquivo e hash juntos; sem prova de data |
| Registro do hash em blockchain pública | Verificação independente e pública | Taxas e latência; menor familiaridade de juízes e peritos; mais complexidade; pode ser extensão futura |
| Assinatura digital sem carimbo do tempo | Prova a origem | Não prova quando a captura ocorreu |
| Ata notarial em cartório | Fé pública reconhecida | Lenta e cara; é justamente o processo que o produto busca simplificar |

## Justificativa

A combinação de hash, assinatura e carimbo do tempo de terceiro confiável permite que
qualquer pessoa verifique, sem depender do Veridit, que o pacote não mudou e que existia na
data indicada. O armazenamento imutável impede alteração ou remoção, inclusive por
administradores. A trilha de custódia documenta o caminho do arquivo desde o worker até o
pacote. Juntas, essas táticas atendem diretamente à característica de integridade
(ADR-0001).

## Consequências

**Positivas**

- Pacote verificável por perito, offline, com ferramentas comuns (sha256sum, OpenSSL).
- Qualquer adulteração é detectável.

**Negativas / trade-offs**

- Custo por carimbo do tempo e por armazenamento retido.
- Dependência da disponibilidade da Autoridade de Carimbo do Tempo. Mitigação: a selagem é
  assíncrona com novas tentativas; o registro fica em "Selando" até o carimbo ser obtido.
- Arquivos sob retenção WORM não podem ser apagados antes do prazo, mesmo a pedido do
  usuário. Base legal a validar: exercício regular de direitos em processo (LGPD, art. 7º, VI).

**Riscos**

- Comprometimento da chave da plataforma. Mitigação: chave em KMS/HSM, rotação e carimbo do
  tempo independente.

## Conformidade

- Job diário que, por amostragem, recalcula hashes e valida assinatura e carimbo de registros
  concluídos; qualquer divergência gera alerta crítico.
- Teste automatizado: tentar sobrescrever ou apagar um objeto do bucket de evidências deve falhar.

## Notas

- ADRs relacionadas: 0001, 0003, 0010.
- Elaborada em sala com apoio de ferramenta de IA; registro da interação em <local>.
