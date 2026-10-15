# Contexto do projeto
Sistema: Veridit. Documento de requisitos em docs/requisitos/veridit-req.md.
Objetivo do trabalho: produzir a documentação arquitetural do sistema.

# Entregáveis
1. Características arquiteturais com justificativa (1-2 parágrafos cada).
2. Top 4 características, justificadas por ADR.
3. Componentes candidatos e suas responsabilidades.
4. Diagrama de componentes, especificando internamente pelo menos 30% dos componentes mais críticos, com justificativa da escolha.
5. Estilo arquitetural escolhido, justificado por ADR.
6. Diagrama de módulos (pacotes UML).
7. Táticas para cada característica, com o objetivo de cada tática.
8. Conjunto de ADRs do projeto, incluindo as do top 4.
9. Lista de telas da aplicação.

# Regras para o agente
- Toda afirmação sobre o sistema deve citar o requisito (REQ xx), o atributo de qualidade, a decisão (D1, D2) ou o campo do Apêndice A que a sustenta. Se algo não estiver no documento, diga explicitamente que é uma inferência e explique por quê.
- Nunca invente requisitos que não existem no documento.
- Respeite o escopo da primeira versão definido em D1 e D2.
- Apresente alternativas com prós e contras; a decisão final é da equipe.
- Escreva em português do Brasil.
- Não faça commits sem a equipe pedir.

# Modelo de ADR (obrigatório)
Arquivo: docs/adr/ADR-XXX-titulo-curto.md
- Título: ADR-XXX: [decisão]
- Status: Proposta | Aceita | Substituída por ADR-YYY
- Data:
- Requisitos e restrições relacionados: [REQ, atributos, D1/D2]
- Contexto
- Decisão
- Alternativas consideradas (cada uma com prós e contras)
- Justificativa
- Consequências (positivas e negativas)
