# ADR-0002: Adotar arquitetura baseada em serviços com mensageria assíncrona

- **Status:** Aceita
- **Data:** 2026-10-15
- **Decisores:** <nomes da equipe>
- **Características arquiteturais relacionadas:** Segurança, Integridade, Confiabilidade, Escalabilidade (top 4, ADR-0001); Manutenibilidade; Custo
- **Requisitos e restrições relacionados:** REQ01–REQ15; restrições do projeto: equipe pequena, prazo acadêmico, etapas futuras de implementação e evolução

## Contexto

O domínio se divide naturalmente em áreas com pouca dependência entre si: identidade e
acesso, créditos e pagamentos, gestão de registros, captura, evidências (selagem, relatório
e .zip) e notificações. Essas áreas têm perfis de carga diferentes. Cadastro, login e
listagens são operações curtas de CRUD. A captura dura de segundos a minutos, consome muita
CPU e memória e executa conteúdo externo não confiável. Os e-mails (REQ07, REQ10, REQ11-B)
são assíncronos por natureza.

A equipe é pequena e o projeto terá etapas de implementação e evolução, então o estilo
precisa ser simples o bastante para ser construído e testado pela equipe.

## Decisão

Vamos adotar o estilo **baseado em serviços** (*service-based*), particionado por domínio:

- Seis serviços de domínio implantados separadamente: Identidade e Acesso, Créditos e
  Pagamentos, Registros, Captura, Evidências e Notificações, mais um serviço transversal de
  Auditoria.
- Uma aplicação web (SPA) acessa os serviços por um **API Gateway**.
- Comunicação **síncrona (REST)** para comandos e consultas da interface, e **assíncrona
  (eventos em broker de mensagens)** para captura, selagem, notificações e auditoria
  (ADR-0007).
- Um banco de dados compartilhado, com esquema isolado por serviço (ADR-0010).
- O serviço de Captura não acessa o banco e se comunica só por fila e armazenamento de
  objetos, formando um quantum arquitetural próprio, escalável de forma independente
  (ADR-0009).

## Alternativas consideradas

Classificação por estrelas de Richards & Ford (1 = fraco, 5 = forte):

| Característica | Camadas | Microkernel | Baseado em serviços | Orientado a eventos | Space-based | Microsserviços |
|---|---|---|---|---|---|---|
| Escalabilidade | 1 | 1 | 3 | 5 | 5 | 5 |
| Elasticidade | 1 | 1 | 2 | 3 | 5 | 5 |
| Tolerância a falhas | 1 | 1 | 4 | 5 | 3 | 4 |
| Confiabilidade | 3 | 3 | 4 | 3 | 4 | 4 |
| Testabilidade | 2 | 3 | 4 | 2 | 1 | 4 |
| Implantabilidade | 1 | 3 | 4 | 3 | 3 | 4 |
| Simplicidade | 5 | 4 | 3 | 1 | 1 | 1 |
| Custo (5 = barato) | 5 | 5 | 4 | 3 | 2 | 1 |

| Alternativa | Prós | Contras |
|---|---|---|
| Monólito em camadas | Mais simples e barato | Escalabilidade, elasticidade e tolerância a falhas = 1: um pico de capturas derrubaria login e pagamentos; o navegador que abre sites externos ficaria no mesmo processo que os dados sensíveis |
| Microkernel | Bom para variações por plug-in | Não resolve escalabilidade nem isolamento; o núcleo do Veridit não é um produto de plug-ins |
| Microsserviços | Escalabilidade e elasticidade máximas | Custo e simplicidade = 1; exige banco por serviço e transações distribuídas para créditos; desproporcional para 15 requisitos e uma equipe pequena |
| Orientado a eventos puro | Escalabilidade e tolerância a falhas = 5 | Testabilidade = 2, simplicidade = 1; consultas da interface (listar, detalhar) ficam mais difíceis; tratamento de erros complexo |
| Space-based | Elasticidade máxima para picos | Feito para alto volume concorrente sobre dados em memória, que não é o caso; testabilidade = 1 e alto custo |

## Justificativa

O estilo baseado em serviços tem boas notas justamente nas características prioritárias que
dependem da estrutura: tolerância a falhas e confiabilidade (4), além de testabilidade,
implantabilidade e custo (4) adequados à equipe. Seus pontos fracos, elasticidade (2) e
escalabilidade (3), aparecem quando todo o sistema precisa escalar. No Veridit, só a captura
precisa disso, então tratamos esse ponto separando a Captura em um serviço com workers sem
estado alimentados por fila (ADR-0009). Usamos eventos só onde a assincronia é natural, sem
pagar o custo de um sistema totalmente orientado a eventos.

Para segurança e integridade, a separação em serviços também isola o componente mais
arriscado (o navegador que carrega sites externos) dos dados pessoais e de pagamento, e
concentra a selagem das evidências em um único serviço auditável.

## Consequências

**Positivas**

- Falha ou sobrecarga na captura não afeta login, compras e downloads.
- Cada serviço pode ser implantado e testado separadamente.
- Caminho de evolução: um serviço pode ser dividido em microsserviços no futuro, se necessário.

**Negativas / trade-offs**

- Custos de sistema distribuído: latência de rede, consistência eventual entre Registros e
  Evidências, necessidade de observabilidade (correlation-id) e de um broker.
- O banco compartilhado acopla os serviços que o usam (mitigado pelo isolamento por esquema).

**Riscos**

- Proliferação de chamadas síncronas entre serviços. Mitigação: chamadas síncronas só para a
  reserva de créditos; o restante por eventos.

## Conformidade

- Função de aptidão no CI: nenhum serviço importa pacotes internos de outro serviço; só o
  pacote `compartilhado/contratos` (regras de dependência, ex.: dependency-cruiser ou ArchUnit).
- Cada serviço usa um usuário de banco com permissão apenas no próprio esquema.
- A imagem do serviço de Captura não contém driver nem credenciais de banco.

## Notas

- Referência: Richards, M.; Ford, N. *Fundamentals of Software Architecture*, capítulos sobre
  estilos (camadas, microkernel, baseado em serviços, orientado a eventos, space-based e
  microsserviços).
- ADRs relacionadas: 0001, 0007, 0009, 0010.
- Elaborada em sala com apoio de ferramenta de IA; registro da interação em <local>.
