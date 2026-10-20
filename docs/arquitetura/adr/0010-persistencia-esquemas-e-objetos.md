# ADR-0010: Usar PostgreSQL compartilhado com esquemas isolados por serviço e armazenamento de objetos para arquivos

- **Status:** Aceita
- **Data:** 2026-10-15
- **Decisores:** Cauã Marinho, João Ferraz, Guilherme Loiola e Davi Araújo
- **Características arquiteturais relacionadas:** Segurança (privacidade), Manutenibilidade, Recuperabilidade, Integridade
- **Requisitos e restrições relacionados:** Apêndice A (dados de usuários, compras e registros); REQ12–REQ15 (consultas, relatório e .zip)

## Contexto

No estilo baseado em serviços é comum compartilhar o banco de dados, o que simplifica a
operação. Porém tabelas compartilhadas criam acoplamento: uma mudança de esquema pode quebrar
vários serviços, e qualquer serviço consegue ler dados que não deveria (ex.: faturamento). O
sistema também lida com arquivos grandes (imagens, vídeos, relatórios, .zip), que não devem
ficar dentro do banco relacional.

## Decisão

- Uma instância **PostgreSQL** (com réplica e backups com recuperação ponto a ponto), com um
  **esquema por serviço**: `identidade`, `creditos`, `registros`, `evidencias`, `auditoria`.
- Cada serviço conecta com **usuário de banco próprio**, com permissão apenas no próprio esquema.
  Não há *joins* entre esquemas; dados de outro serviço vêm por API ou evento.
- Arquivos em **armazenamento de objetos compatível com S3**, em buckets separados:
  `captura-temporaria`, `evidencias` (com Object Lock, ADR-0004) e `relatorios`.
- Criptografia em repouso em ambos; CPF, telefone e endereço também **criptografados em nível
  de campo** pela aplicação.

## Alternativas consideradas

| Alternativa | Prós | Contras |
|---|---|---|
| Banco por serviço (estilo microsserviços) | Independência total | Mais custo operacional; consultas e consistência entre serviços ficam mais difíceis |
| Um esquema único compartilhado | Simples | Alto acoplamento; qualquer serviço lê dados de pagamento e pessoais |
| Arquivos dentro do banco (BLOB) | Uma única fonte | Banco grande, backups pesados, sem bloqueio WORM |

## Justificativa

O isolamento por esquema e usuário mantém a simplicidade operacional do banco compartilhado e
limita o acoplamento e a exposição de dados, como recomendam Richards & Ford para o particionamento
lógico do banco no estilo baseado em serviços. Separar os arquivos em armazenamento de objetos
permite o bloqueio WORM exigido pela integridade e mantém o banco leve.

## Consequências

**Positivas**

- Operação simples, com transações locais em cada serviço.
- Caminho de evolução para banco por serviço: os esquemas já estão separados.

**Negativas / trade-offs**

- O banco é ponto único de falha e capacidade compartilhada (mitigado por réplica e failover).
- Os serviços que usam o banco formam um único quantum arquitetural; aceitável porque só a
  Captura precisa de elasticidade independente.

## Conformidade

- Verificação no CI: as migrações do serviço X só alteram o esquema X.
- Auditoria periódica das permissões (*grants*) do banco.
- Teste de restauração de backup a cada mês.

## Notas

- ADRs relacionadas: 0002, 0004.
- Elaborada em sala com apoio de ferramenta de IA; registro da interação em <local>.
