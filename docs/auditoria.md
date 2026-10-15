# Auditoria Final — Documentação Arquitetural do Veridit

> Etapa 11. Conferência do repositório contra os entregáveis do `AGENTS.md` e do enunciado do Trabalho II, feita em 15/10/2026.

## 1. Entregáveis

| # | Entregável | Onde | Situação |
|---|---|---|---|
| 1 | Características arquiteturais com justificativa | `docs/caracteristicas.md` | ✅ 12 candidatas + 5 descartadas |
| 2 | Top 4 justificado por ADR | `docs/adr/ADR-001-top4-caracteristicas.md` | ✅ (status Proposta, ver seção 4) |
| 3 | Componentes candidatos e responsabilidades | `docs/componentes.md`, seções 1–3 | ✅ 14 componentes, com rastreabilidade |
| 4 | Diagrama de componentes com ≥ 30% dos críticos especificados internamente | `docs/diagramas/componentes.*` e `componentes-criticos.*`; `docs/componentes.md`, seções 4–5 | ✅ 5 de 14 = 35,7%, com critério de escolha |
| 5 | Estilo arquitetural justificado por ADR | `docs/adr/ADR-003-estilo-baseado-em-servicos.md` | ✅ Baseado em serviços |
| 6 | Diagrama de módulos (pacotes UML) | `docs/diagramas/modulos.*` | ✅ |
| 7 | Táticas por característica, com objetivo | `docs/taticas.md` | ✅ 28 táticas para o top 4 + 5 para as não priorizadas |
| 8 | Conjunto de ADRs | `docs/adr/` | ✅ ADR-001 a ADR-009 |
| 9 | Telas da aplicação | `docs/telas.md` | ✅ lista de 16 telas; ⏳ protótipo no Figma |
| — | PDF e PowerPoint para apresentação | — | ⏳ pendente |
| — | Histórico no Git mostrando o que mudou e por quê | histórico de commits | ✅ um commit por etapa, com motivo |

## 2. Cobertura dos requisitos

| Requisito | Componentes | Telas | ADRs |
|---|---|---|---|
| REQ 01 | C03 | T02 | ADR-006 |
| REQ 02 | C04, C14 | T03, T04 | ADR-006, ADR-008 |
| REQ 03 | C04 | T01 | ADR-006 |
| REQ 04 | C04 | menu (Sair) | ADR-006 |
| REQ 05 | C05, C06 | T06, T07 | ADR-007, ADR-009 |
| REQ 06 | C06 | T08, T09 | ADR-007 |
| REQ 07 | C06, C14 | T10 | ADR-007, ADR-008 |
| REQ 08 | C07, C08 | T11, T12 | ADR-002, ADR-009 |
| REQ 09 | C07, C08, C09, C10 | T12, T13 | ADR-005, ADR-009 |
| REQ 10 | C07, C14 | T13 | ADR-008 |
| REQ 11 | C07, C08, C09, C10 | T12, T13 | ADR-005, ADR-009 |
| REQ 11 (duplicado) | C07, C14 | T13 | ADR-008 |
| REQ 12 | C11 | T14 | ADR-009 (status) |
| REQ 13 | C10, C11 | T15 | ADR-004 |
| REQ 14 | C10, C12 | T16 | ADR-005 |
| REQ 15 | C10, C13 | T15 | ADR-004, ADR-005 |
| Segurança (seção 3) | C02, C03, C04, C10, C11 | todas após T01 | ADR-006 |
| D1 | C03 | T02 | ADR-003, ADR-006 |
| D2 | C05 | T06 | ADR-007 |

Nenhum requisito ficou sem componente, tela ou decisão.

## 3. Consistência entre documentos

- **Nomes de componentes:** os mesmos 14 nomes e ids (C01–C14) aparecem em `componentes.md`, nos diagramas, nas ADRs e em `taticas.md`.
- **Top 4:** todas as táticas e ADRs se referem às mesmas quatro características da ADR-001.
- **Corrigido nesta auditoria:** `caracteristicas.md` dizia que os dados de faturamento têm 12 campos; o Apêndice A lista 11 (nome completo, email, telefone, cpf, cep, endereço, número, complemento, bairro, cidade, estado).
- **Corrigido na Etapa 4:** `comparacao-estilos.md` avaliava a arquitetura baseada em serviços como se tivesse banco por serviço e sagas.
- **Registrado:** a resposta da equipe à pergunta 1 da análise de requisitos (onde roda a captura) foi anotada em `analise-requisitos.md`.

## 4. Pendências para confirmar com o engenheiro de requisitos (professor)

Todas as ADRs estão com status **Proposta**, seguindo a regra da própria ADR-001: enquanto o valor probatório do relatório não for confirmado, as ADRs que dependem da Integridade da evidência não passam a Aceita.

| # | Pergunta | Afeta |
|---|---|---|
| 1 | O relatório precisa ter valor de prova (hash, carimbo de tempo)? | ADR-001, ADR-005 |
| 2 | Podemos incluir no relatório os hashes dos arquivos, que não estão no Apêndice A? | ADR-005, T16 |
| 3 | Quando o crédito é consumido? (proposta: reserva ao iniciar, débito na conclusão, estorno se falhar) | ADR-009 |
| 4 | Quais são os valores de "status"? (proposta: INICIANDO, EM_ANDAMENTO, FINALIZANDO, CONCLUIDO, INTERROMPIDO, FALHOU) | ADR-009, T14 |
| 5 | Tela e vídeo são tipos alternativos de registro, escolhidos ao iniciar? | `componentes.md`, T11 |
| 6 | O e-mail do REQ 07 sai na solicitação ou na confirmação do pagamento? (proposta: na confirmação) | ADR-007 |
| 7 | Por quanto tempo as evidências ficam guardadas? | ADR-004 |
| 8 | Um registro interrompido pode gerar relatório e zip? | ADR-009 |
| 9 | Qual dos dois REQ 11 deveria ser REQ 16? | documentação inteira |

Se o professor confirmar o valor probatório (pergunta 1), basta mudar o status das ADRs para **Aceita** num commit dizendo "confirmação do engenheiro de requisitos em DD/MM".

## 5. Afirmações marcadas como inferência

As decisões que não têm base direta no documento estão marcadas como **[inferência]** no texto: mecanismo de recuperação de senha, regra de créditos, valores de status, tipos de registro, provedor Pix, tecnologia de transmissão da tela, carimbo de tempo, hashes no relatório, formato PDF do relatório, saldo na tela inicial e tempos limite da captura.
