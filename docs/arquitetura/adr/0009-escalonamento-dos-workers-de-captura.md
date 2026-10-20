# ADR-0009: Escalar a captura com pool de workers sem estado e autoescalonamento pela fila

- **Status:** Aceita
- **Data:** 2026-10-15
- **Decisores:** <nomes da equipe>
- **Características arquiteturais relacionadas:** Escalabilidade (elasticidade), Disponibilidade, Custo
- **Requisitos e restrições relacionados:** REQ08–REQ11 (captura de tela e vídeo)

## Contexto

Cada sessão de captura precisa de um navegador dedicado (ordem de 1 vCPU e 1–2 GB de
memória, mais na codificação de vídeo) durante minutos. A demanda tende a ser irregular, com
picos em horário comercial e perto de prazos processuais. Os demais serviços são leves.
Dimensionar tudo para o pico desperdiça recursos; dimensionar para a média gera filas longas
no pico.

## Decisão

- O serviço de Captura roda como **workers sem estado** em contêineres, gerenciados por um
  orquestrador (ex.: Kubernetes; em desenvolvimento, Docker Compose com réplicas fixas).
- Pedidos de sessão entram na fila `captura.solicitada`. Cada worker atende **uma sessão por
  vez** e só busca a próxima quando fica livre (*prefetch* = 1).
- Um **autoescalador** (ex.: KEDA) ajusta o número de workers pela profundidade da fila e pelo
  número de sessões ativas, entre um mínimo (**pool aquecido**, ex.: 2) e um máximo (teto de custo).
- **Limites de demanda:** até 2 sessões simultâneas por usuário e duração máxima por sessão.
- Os artefatos vão para uma área temporária no armazenamento de objetos; o worker **não acessa
  o banco de dados**.

## Alternativas consideradas

| Alternativa | Prós | Contras |
|---|---|---|
| Escalar o sistema inteiro junto (monólito) | Uma única unidade para operar | Desperdício: replica partes leves para atender à captura |
| Funções serverless (ex.: AWS Lambda) | Escala a zero; paga por uso | Limites de tempo e memória; streaming interativo difícil; cold start |
| Pool fixo dimensionado para o pico | Previsível | Custo ocioso alto fora do pico |
| Pool fixo pequeno | Barato | Filas longas no pico; experiência ruim |

## Justificativa

Workers sem estado alimentados por fila são a forma mais direta de obter elasticidade só onde
ela é necessária, compensando a elasticidade limitada do estilo baseado em serviços
(ADR-0002). A fila funciona como buffer de carga, e o pool aquecido reduz o tempo de espera
pela primeira sessão. Como o worker não acessa o banco, ele forma um quantum arquitetural
independente, com características operacionais próprias.

## Consequências

**Positivas**

- Custo proporcional à demanda.
- Falha de um worker afeta só uma sessão; o orquestrador reinicia o contêiner.

**Negativas / trade-offs**

- Exige infraestrutura de orquestração de contêineres e conhecimento da equipe para operá-la.
- Partida de novos nós leva tempo (mitigado pelo pool aquecido).

**Riscos**

- Custos fora de controle em um pico anômalo. Mitigação: teto de workers e limites por usuário.

## Conformidade

- Teste de carga: com 10x sessões, tempo de espera p95 por sessão ≤ 30 s após o escalonamento.
- Métrica com alerta: tempo de espera na fila `captura.solicitada`.
- Função de aptidão: a imagem do worker não contém credenciais nem driver de banco.

## Notas

- ADRs relacionadas: 0002, 0003, 0007.
- Elaborada em sala com apoio de ferramenta de IA; registro da interação em <local>.
