# ADR-004: Banco relacional compartilhado com esquemas por serviço e armazenamento imutável para evidências

- **Título:** ADR-004: Usar um banco relacional compartilhado, dividido em esquemas por serviço, e guardar os arquivos de evidência num armazenamento de objetos com bloqueio de escrita
- **Status:** Proposta
- **Data:** 2026-10-15
- **Requisitos e restrições relacionados:**
  - Entidades do Apêndice A: Advogado, Usuário comum, Pacote de créditos, Dados da compra, Registro de conteúdo, Registros realizados, Detalhes dos registros, Relatório do registro.
  - REQ 08–REQ 11 (geração de imagens e vídeos), REQ 13 ("número de imagens e vídeos"), REQ 15 (zip com os arquivos capturados).
  - ADR-001 (Integridade da evidência, Modularidade); ADR-003 (estilo baseado em serviços).

## Contexto

O sistema tem dois tipos de dados muito diferentes:
- **dados estruturados**, pequenos e relacionados entre si: usuários, pacotes, compras, registros e seus status (Apêndice A);
- **arquivos grandes**: as imagens e os vídeos capturados (REQ 08–11), que são a evidência em si e não podem ser alterados depois de gerados (ADR-001).

A ADR-003 escolheu a arquitetura baseada em serviços, que no livro de referência usa normalmente um banco compartilhado. É preciso decidir como os serviços dividem esse banco e onde ficam os arquivos.

## Decisão

1. **Um único banco relacional** (ex.: PostgreSQL) com **um esquema por serviço**: `contas`, `creditos`, `captura`, `registros` e `notificacoes`.
2. **Cada serviço só escreve no próprio esquema.** Quando precisa de dado de outro serviço, chama a API dele. Leituras diretas entre esquemas só são permitidas por meio de visões publicadas pelo dono dos dados.
3. **Arquivos de evidência** (imagens, vídeos e manifestos) ficam num **armazenamento de objetos com bloqueio de escrita** (modo "uma escrita, várias leituras"): o armazenamento recusa sobrescrita e exclusão durante o período de retenção, mesmo que a aplicação peça. No banco ficam apenas os metadados (nome, tamanho, hash, tipo, registro).
4. Somente o serviço Captura tem permissão de gravar no armazenamento de evidências; os demais só leem por meio do C10.

## Alternativas consideradas

**A. Banco próprio para cada serviço**
- Prós: isolamento total; cada serviço evolui o esquema livremente.
- Contras: operações que atravessam serviços (reserva de crédito e conclusão da captura) passam a depender de consistência eventual; mais bancos para operar. É o modelo de microsserviços, descartado na ADR-003.

**B. Banco compartilhado sem divisão por esquema**
- Prós: o mais simples de começar.
- Contras: qualquer serviço pode alterar qualquer tabela; perde-se a fronteira entre domínios (contra a Modularidade) e um defeito em um serviço pode corromper o registro de outro (contra a Integridade).

**C. Guardar os arquivos dentro do banco relacional**
- Prós: um só lugar para tudo; transação única.
- Contras: vídeos deixam o banco grande e lento; o banco não oferece bloqueio de escrita por objeto, então a imutabilidade dependeria só da aplicação.

**D. Sistema de arquivos comum no servidor**
- Prós: simples.
- Contras: qualquer processo com acesso ao disco pode alterar ou apagar a evidência; difícil de replicar.

## Justificativa

- **Integridade da evidência:** o bloqueio de escrita do armazenamento garante que nem um defeito da aplicação nem um operador apaguem ou modifiquem um arquivo selado. A integridade deixa de depender só do código.
- **Modularidade:** esquemas separados e a regra de escrita só no próprio esquema mantêm as fronteiras de domínio da ADR-003 dentro do banco.
- **Consistência:** com um banco único, as operações de cada serviço continuam em transações locais, sem consistência eventual.
- **Custo:** um banco e um armazenamento de objetos são suficientes para a v1.

## Consequências

- **Positivas:**
  - A evidência fica protegida em duas camadas: hash e assinatura (ADR-005) para detectar alteração, e bloqueio de escrita para impedi-la.
  - Se algum serviço crescer muito no futuro, o esquema dele já está separado e pode ser migrado para um banco próprio.
- **Negativas:**
  - O banco continua sendo compartilhado: uma indisponibilidade dele afeta todos os serviços.
  - A regra "cada serviço só escreve no próprio esquema" depende de disciplina e de permissões de banco por usuário de serviço, que precisam ser configuradas.
  - Arquivos com bloqueio de escrita não podem ser apagados antes do fim da retenção. O documento não define retenção nem exclusão (análise, seção 5, pergunta 12), então o prazo precisa ser confirmado com o engenheiro de requisitos **[inferência]**.
