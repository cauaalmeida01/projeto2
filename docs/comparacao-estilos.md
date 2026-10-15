# Comparação de Estilos Arquiteturais — Veridit

> Etapa 4 da documentação arquitetural. Base: `docs/adr/ADR-001-top4-caracteristicas.md` (top 4: Segurança, Integridade da evidência, Confiabilidade da captura, Modularidade — Combinação A, status Proposta), `docs/caracteristicas.md` e `docs/analise-requisitos.md`. **Esta etapa NÃO fecha a decisão de estilo** — a equipe escolherá e a decisão será registrada em ADR própria (entregável 5 do AGENTS.md). Inferências estão marcadas **[inferência]**.

Restrição de contexto (explicitada no AGENTS.md e coerente com D1/D2, que delimitam uma v1 enxuta): o sistema será construído por uma **equipe de 5 estudantes**. Nenhum requisito do documento define prazo, orçamento ou infraestrutura operacional — avaliações de custo/complexidade são, portanto, **[inferência]** apoiada nesse contexto.

---

## Estilos avaliados

1. **Camadas** — organização do sistema em camadas horizontais (apresentação, lógica de negócio, persistência), cada uma só acessando a camada inferior.
2. **Monólito modular** — aplicação implantada como uma unidade, mas internamente organizada em módulos coesos com acoplamento controlado e interfaces explícitas.
3. **Baseado em serviços** — serviços de maior grão (ex.: captura, pagamentos, evidências) implantados separadamente, comunicando via API.
4. **Microsserviços** — muitos serviços de grão fino, cada um com banco próprio, comunicação via rede e implantação independente.
5. **Orientado a eventos** — componentes produzem e consomem eventos (ex.: `CapturaIniciada`, `PagamentoConfirmado`), com comunicação assíncrona via broker.
6. **Microkernel** — núcleo mínimo com funcionalidades plugáveis (plugins) adicionadas como módulos independentes.
7. **Space-based** — dados e processamento distribuídos em memória compartilhada (tuplespaces) para escala massiva e alta disponibilidade.

---

## 1. Camadas

| Característica do top 4 | Atendimento | Explicação |
|---|---|---|
| Segurança | **Médio** | A autenticação (REQ 01–04, seção 3) naturalmente vira uma camada de serviços de segurança, mas o estilo não impede vazamentos entre camadas se a disciplina de dependências for frouxa. |
| Integridade da evidência | **Médio** | Hash/carimbos podem ficar isolados na camada de persistência, mas nada no estilo incentiva a imutabilidade como princípio estrutural (Apêndice A: relatório probatório). |
| Confiabilidade da captura | **Fraco** | Captura de longa duração (REQ 08/09) não se encaixa no modelo síncrono camada a camada; gestão de estado do registro tende a ficar espalhada. |
| Modularidade | **Fraco** | Camadas separam **tecnicamente** (UI/lógica/dados), não por **domínio de negócio**; os quatro domínios do Apêndice A (usuários, pagamentos, captura, exportação) atravessariam todas as camadas — exatamente o oposto do que D1/D2 pedem para evoluir separadamente. |

- **Custo/complexidade/equipe:** menor custo e complexidade de todos; ideal para a v1 de 5 estudantes.
- **Riscos:** degradação em "big ball of mud" conforme cresce; evoluções D1/D2 (empresas, novos meios de pagamento — REQ 06) concentradas nos mesmos arquivos.

## 2. Monólito modular

| Característica do top 4 | Atendimento | Explicação |
|---|---|---|
| Segurança | **Forte** | Módulo de segurança com interface explícita usado por todos os demais; imposição centralizada de autenticação (REQ 01–04, seção 3) sem custo de rede. |
| Integridade da evidência | **Forte** | O módulo de evidências pode encapsular hash, metadados e o repositório imutável do relatório (Apêndice A), com transações locais fortes. |
| Confiabilidade da captura | **Médio** | A máquina de estados do registro (REQ 08–11, campo "status") fica num módulo claro; porém operações de longa duração dentro de um processo único exigem threads/filas internas bem geridas. |
| Modularidade | **Forte** | É a materialização direta da característica: módulos por domínio (usuários, pagamentos — REQ 05–07/D2, captura — REQ 08–11, exportação — REQ 12–15) com fronteiras rigorosas, permitindo evoluir D1/D2 e, no futuro, extrair serviços. |

- **Custo/complexidade/equipe:** custo baixo-moderado; única implantação simplifica operação de 5 estudantes; exige disciplina (testes de fronteira, isolamento de dependências).
- **Riscos:** falta de disciplina derrete para camadas acopladas; limites de escala futuros (porém nenhuma meta de escala existe no documento — inferência: risco de sobre-engenharia se for além dele).

## 3. Baseado em serviços

| Característica do top 4 | Atendimento | Explicação |
|---|---|---|
| Segurança | **Médio** | Autenticação centralizada em um serviço de identidade; porém cada fronteira de rede amplia a superfície de ataque e exige mTLS/controle de chamadas entre serviços. |
| Integridade da evidência | **Médio** | Serviço de evidências dedicado pode centralizar hash/imutabilidade; mas transações que cruzam serviços (compra → crédito → captura — REQ 05→08) exigem sagas/compensação, arriscando inconsistência do rastro. |
| Confiabilidade da captura | **Forte** | Captura (REQ 08–11) como serviço dedicado e isolado: crash do serviço de captura não derruba consultas (REQ 12/13); escala independente da carga de vídeo. |
| Modularidade | **Forte** | Módulos viram serviços com fronteiras físicas — a modularidade máxima do domínio, mas paga com orquestração. |

- **Custo/complexidade/equipe:** custo e complexidade moderados-altos (implantação, rede, contratos, sagas); desafiador para 5 estudantes, mas viável com poucos serviços (2–3).
- **Riscos:** inconsistência de dados entre serviços (ex.: crédito debitado sem captura concluída); sobrecarga operacional não sustentada por nenhum requisito do documento.

## 4. Microsserviços

| Característica do top 4 | Atendimento | Explicação |
|---|---|---|
| Segurança | **Médio** | Autenticação via token distribuído (JWT) funciona; mas 6+ serviços multiplicam pontos de configuração de segurança e segredos. |
| Integridade da evidência | **Médio** | Cada serviço com banco próprio dificulta a visão unificada e imutável do registro (Apêndice A exige rastro unificado: usuário, datas, arquivos); consistência eventual ameaça o não-repúdio. |
| Confiabilidade da captura | **Forte** | Serviço de captura escala isoladamente e falha isoladamente — o melhor atendimento estrutural a REQ 08–11. |
| Modularidade | **Forte** | Fronteiras físicas máximas por domínio. |

- **Custo/complexidade/equipe:** custo/complexidade altos (orquestração, observabilidade, DevOps, bancos distribuídos); **inadequado para 5 estudantes** na v1.
- **Riscos:** consistência eventual conflita com evidência probatória; sobre-engenharia sem qualquer requisito de escala no documento (lacuna registrada na análise — seção 4, atributos de qualidade).

## 5. Orientado a eventos

| Característica do top 4 | Atendimento | Explicação |
|---|---|---|
| Segurança | **Médio** | Perfil do usuário e permissões podem ser propagados por eventos; auditoria de eventos é natural. Porém o controle de acesso por tópico/fila adiciona superfície de configuração. |
| Integridade da evidência | **Forte** | Eventos imutáveis com timestamp formam um log natural da cadeia de custódia (captura iniciada → concluída → relatório gerado — REQ 08→14); o log de eventos é, por si, um registro audível **[inferência — sem requisito de cadeia de custódia no documento]**. |
| Confiabilidade da captura | **Forte** | Operações assíncronas de longa duração são o caso de uso central do estilo: `CapturaIniciada` → processamento desacoplado → `CapturaConcluida` dispara emails (REQ 10/11) e atualiza status (Apêndice A) com reentregas. |
| Modularidade | **Forte** | Produtores/consumidores desacoplados por contrato de evento; novos fluxos (novos meios de pagamento — REQ 06) adicionam assinantes sem tocar os existentes. |

- **Custo/complexidade/equipe:** custo moderado (broker — Redis/RabbitMQ/Kafka); assincronia aumenta a dificuldade de depuração, mas para 5 estudantes é viável com um broker simples e poucos tópicos.
- **Riscos:** entrega eventualmente inconsistente exige cuidado com idempotência (ex.: duplo débito de crédito — REQ 05); rastreamento entre eventos exige observação/tracing (característica #12, não dirigente).

## 6. Microkernel

| Característica do top 4 | Atendimento | Explicação |
|---|---|---|
| Segurança | **Fraco** | Nada no estilo contribui; plugins ampliam superfície de ataque se mal isolados. |
| Integridade da evidência | **Fraco** | Indiferente ao domínio da evidência. |
| Confiabilidade da captura | **Médio** | Poderia tratar "modo tela" e "modo vídeo" (REQ 08–11) como plugins, mas essa extensibilidade não é exigida por requisito. |
| Modularidade | **Médio** | Forte apenas na dimensão de extensibilidade de recursos; os requisitos não pedem plugins (D2 fixa 3 pacotes; D1 fixa 2 tipos de usuário na v1). |

- **Custo/complexidade/equipe:** custo moderado (desenho de API de plugins); adequado à equipe, mas desnecessário.
- **Riscos:** sobre-engenharia de extensibilidade sem base nos requisitos; risco de escolha errada do núcleo.

## 7. Space-based

| Característica do top 4 | Atendimento | Explicação |
|---|---|---|
| Segurança | **Fraco** | Sem contribuição; replicação de dados em múltiplos nós amplia superfície. |
| Integridade da evidência | **Fraco** | Dados voláteis/in-memory conflitam com armazenamento durável e imutável de mídia (REQ 15; Apêndice A). |
| Confiabilidade da captura | **Médio** | Bom para picos de processamento, mas a captura é operação sequencial de longa duração, não paralela em memória. |
| Modularidade | **Fraco** | Distribuição por espaço não corresponde aos domínios de negócio do Apêndice A. |

- **Custo/complexidade/equipe:** custo/complexidade altos; tecnologia especializada; inadequado para a equipe.
- **Riscos:** perda de dados em falha (antitético à evidência); totalmente desalinhado do escopo v1 (D1/D2) — nenhum requisito de escala massiva existe.

---

## Tabela comparativa

Legenda: **F** forte, **M** médio, **W** fraco. Custo/complexidade avaliados para uma equipe de 5 estudantes na v1 (D1/D2) — **[inferência]**, pois o documento não define equipe/prazo.

| Estilo | Segurança | Integridade | Confiabilidade | Modularidade | Custo | Complexidade | Adequação equipe 5 | Risco principal |
|---|---|---|---|---|---|---|---|---|
| Camadas | M | M | W | **W** | Baixo | Baixa | Alta | Domínios de D1/D2 atravessam camadas; degrada para mud acoplado |
| Monólito modular | **F** | **F** | M | **F** | Baixo–Médio | Baixa–Média | Alta | Disciplina de fronteira; escala futura |
| Baseado em serviços | M | M | **F** | **F** | Médio–Alto | Média–Alta | Baixa–Média | Sagas; inconsistência; operação pesada |
| Microsserviços | M | M | **F** | **F** | Alto | Alta | Baixa | Consistência eventual vs. evidência; sobre-engenharia |
| Orientado a eventos | M | **F** | **F** | **F** | Médio | Média | Média | Idempotência; depuração assíncrona |
| Microkernel | W | W | M | M | Médio | Média | Alta | Extensibilidade sem demanda |
| Space-based | W | W | M | W | Alto | Alta | Muito baixa | Volatilidade vs. evidência durável |

---

## Os 2 estilos mais adequados

### 1º — Monólito modular

Atende **fortemente** as quatro características do top 4: modularidade é o próprio princípio do estilo (módulos por domínio — REQ 01–15/D1/D2); integridade e segurança se beneficiam de transações locais fortes e imposição centralizada; confiabilidade da captura é o único ponto médio, resolvido com workers/filas **internas** ao monólito. É o estilo com melhor razão atendimento/custo para o escopo da v1 (D1/D2) e para uma equipe de 5 estudantes, e preserva a opção futura de extrair serviços (caminho de evolução natural).

### 2º — Orientado a eventos

Atende **fortemente** as três características de negócio mais críticas: confiabilidade da captura (assincronia e reentregas para REQ 08–11) e integridade (log de eventos como trilha da cadeia da captura), com modularidade por contratos de evento. Onde ele é médio (segurança), o problema se resolve com mecanismos padrão de autenticação, não com o estilo.

### Trade-off entre os dois

- **Monólito modular** minimiza risco de execução (5 estudantes, implantação única, transações fortes para consistência da evidência) e maximiza simplicidade operacional; paga o preço de resolver a assincronia da captura (REQ 08–11) com mecanismos internos (fila em memória/banco + workers), que exigem cuidado extra justamente na característica de maior peso técnico.
- **Orientado a eventos** resolve estruturalmente a assincronia e a rastreabilidade da captura/pagamento (REQ 06–11), mas paga com maior complexidade de depuração, exigência de idempotência e um componente de infraestrutura a mais (broker) — custo que nenhuma característica do documento justifica plenamente na v1.
- **Síntese para a decisão:** o monólito modular **pode incorporar** princípios de eventos internamente (mensageria local/filas) sem adotar o estilo por inteiro — capturando o melhor dos dois com risco de execução menor. A escolha final (e sua justificativa por ADR, entregável 5) cabe à equipe; se a equipe valorizar simplicidade e evolução gradual, o monólito modular é o caminho; se valorizar robustez assíncrona desde o dia 1 e aceitar o custo de um broker, o orientado a eventos ganha força.
