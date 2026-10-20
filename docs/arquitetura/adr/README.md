# Registros de Decisão Arquitetural (ADRs) — Veridit

Modelo padronizado: [0000-modelo-adr.md](0000-modelo-adr.md) (contexto, decisão, alternativas
consideradas, justificativa, consequências, conformidade e requisitos relacionados).

| ADR | Decisão | Características | Status |
|---|---|---|---|
| [0001](0001-top4-caracteristicas.md) | Top 4: segurança, integridade das evidências, confiabilidade e escalabilidade | Top 4 | Aceita |
| [0002](0002-estilo-baseado-em-servicos.md) | Estilo baseado em serviços com mensageria assíncrona | Top 4 | Aceita |
| [0003](0003-captura-em-navegador-remoto-isolado.md) | Captura em navegador remoto isolado no servidor | Integridade, Segurança | Aceita |
| [0004](0004-selagem-das-evidencias.md) | Selagem: SHA-256, manifesto assinado, carimbo do tempo, WORM | Integridade | Aceita |
| [0005](0005-autenticacao-tokens-e-senhas.md) | JWT curto + refresh token revogável; Argon2id | Segurança | Aceita |
| [0006](0006-pagamentos-via-mercado-pago.md) | Pagamentos delegados ao Mercado Pago (checkout e Pix) | Segurança, Confiabilidade | Aceita |
| [0007](0007-mensageria-outbox-e-dlq.md) | Outbox transacional, filas duráveis, DLQ e saga | Confiabilidade | Aceita |
| [0008](0008-carteira-de-creditos-livro-razao.md) | Carteira de créditos como livro-razão com reserva | Confiabilidade | Aceita |
| [0009](0009-escalonamento-dos-workers-de-captura.md) | Workers de captura sem estado com autoescalonamento | Escalabilidade | Aceita |
| [0010](0010-persistencia-esquemas-e-objetos.md) | PostgreSQL com esquema por serviço + armazenamento de objetos | Segurança, Manutenibilidade | Aceita |

Regras da disciplina: ADRs são criadas, editadas e têm status alterado apenas em sala, com
commit durante a sessão. Alterações em ADR aceita: marcar a anterior como "Substituída por
ADR-XXXX" e criar uma nova, preservando o histórico.
