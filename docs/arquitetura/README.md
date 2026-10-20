# Arquitetura do Veridit

- [documento-arquitetural.md](documento-arquitetural.md) — documento completo (características,
  top 4, componentes, diagramas, táticas, rastreabilidade).
- [adr/](adr/README.md) — Registros de Decisão Arquitetural (ADRs).
- [diagramas/](diagramas/) — fontes PlantUML (`.puml`) e imagens geradas (`.png`, `.svg`).

## Gerar os diagramas

```bash
java -jar plantuml.jar -charset UTF-8 -tpng diagramas/*.puml
java -jar plantuml.jar -charset UTF-8 -tsvg diagramas/*.puml
```

| Arquivo | Conteúdo |
|---|---|
| `d01-componentes.puml` | Diagrama de componentes geral |
| `d02-pacotes.puml` | Diagrama de pacotes UML (módulos) |
| `c04`, `c08`, `c09`, `c11`, `c12`, `c13` | Estrutura interna dos componentes críticos |
| `c11-estados.puml` | Estados do registro (saga) |
| `s01-registro.puml`, `s02-compra.puml` | Fluxos principais |
