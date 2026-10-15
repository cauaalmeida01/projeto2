# Documento de Requisitos — Veridt

> Transcrição fiel do documento `veridit-req.pdf` fornecido pelo professor, convertida para Markdown sem alterações de conteúdo.

- **Data de Criação:** 20/4/2026
- **Versão:** 01.00
- **Responsável:** Eduardo Almeida – Engenheiro de Requisitos (eduardo.almeida@ufba.br)
- **Organização:** RiSE Labs – Veridit

## Histórico de Revisões

| Data | Responsável | Mudança |
|---|---|---|
| 20/4/2026 | Eduardo Almeida | Adição dos requisitos iniciais (REQ01 até REQ15) |

## 1. Introdução

Este documento apresenta os requisitos funcionais e os atributos de qualidade definidos para o projeto.

## 2. Requisitos Funcionais

Esta seção apresenta o conjunto de requisitos funcionais especificados para o produto, assim como as prioridades dos mesmos. A prioridade de um requisito é classificada como alta (crucial para o produto), média (importante para o produto) e baixa (desejável, mas a aplicação pode existir sem o mesmo).

### REQ 01 – Cadastrar Usuário

O sistema deve permitir o cadastro de usuários para acesso ao sistema com base nos campos definidos no apêndice A.

- **Decisões registradas:** D1
- **Prioridade:** Alta

### REQ 02 – Recuperar Senha

O sistema deve permitir que o usuário possa recuperar a senha de acesso ao sistema.

- **Prioridade:** Média

### REQ 03 – Logar no Sistema

O sistema deve permitir que o usuário efetue log-in no sistema.

- **Prioridade:** Alta

### REQ 04 – Sair do Sistema

O sistema deve permitir que o usuário efetue log-out no sistema.

- **Prioridade:** Alta

### REQ 05 – Comprar Créditos

O sistema deve permitir que o usuário possa comprar créditos para realizar o registro de ocorrências com base em diferentes pacotes definidos no Apêndice A. A realização da compra de créditos envolve os passos de selecionar o pacote de créditos, informar os dados de faturamento e realizar o pagamento.

- **Decisões registradas:** D2
- **Prioridade:** Alta

### REQ 06 – Efetuar Pagamento de Créditos

O sistema deve permitir que o usuário realize o pagamento dos créditos através do mercado pago ou pela plataforma do Veridit. Ao solicitar o pagamento pela plataforma do Veridit, o sistema deve gerar o QR Code ou o número do pix para pagamento.

- **Prioridade:** Alta

### REQ 07 – Confirmar pagamento por email

O sistema deve informar o usuário por email da compra dos créditos realizada.

- **Prioridade:** Alta

### REQ 08 – Iniciar Registro de conteúdo

O sistema deve permitir que o usuário realize a captura de conteúdo através de um site da internet com base nos campos definidos no apêndice A. O conteúdo pode ser uma tela de um site ou vídeo de navegação.

- **Prioridade:** Alta

### REQ 09 – Concluir gravação de conteúdo

O sistema deve permitir a conclusão da gravação de conteúdo.

- **Prioridade:** Alta

### REQ 10 – Confirmar gravação de conteúdo por email

O sistema deve informar ao usuário por email da conclusão da gravação do conteúdo.

- **Prioridade:** Alta

### REQ 11 – Concluir captura de Tela

O sistema deve permitir a conclusão da captura de tela do conteúdo.

- **Prioridade:** Alta

### REQ 11 – Confirmar captura de tela por email

O sistema deve informar ao usuário por email da conclusão da captura de tela realizada.

- **Prioridade:** Alta

### REQ 12 – Listar Registros realizados

O sistema deve listar os registros realizados pelo usuário com base nos campos definidos no Apêndice A.

- **Prioridade:** Alta

### REQ 13 – Visualizar Detalhes dos Registros realizados

O sistema deve disponibilizar os dados do registro selecionado pelo usuário com base nos campos definidos no Apêndice A.

- **Prioridade:** Alta

### REQ 14 – Gerar Relatório do Registro realizado

O sistema deve gerar um relatório do registro selecionado pelo usuário apresentando as informações com base nos campos definidos no Apêndice A.

- **Prioridade:** Alta

### REQ 15 – Gerar Arquivo zip do Registro realizado

O sistema deve gerar um arquivo zip para download com os arquivos capturados durante o registro.

- **Prioridade:** Alta

## 3. Atributos de Qualidade

**Segurança.** O sistema deve permitir o acesso ao sistema apenas por usuários cadastrados.

## 4. Decisões

Esta seção apresenta as principais decisões consideradas com base nos requisitos do produto.

**D1.** O sistema deve permitir o cadastro de advogados, usuários comuns e empresas. Na primeira versão do sistema, apenas advogados e usuários comuns serão cadastrados.

**D2.** O sistema deve permitir a escolha do pacote de créditos pelo usuário. Na primeira versão do sistema, os pacotes básico, médio e premium são oferecidos.

## Apêndice A

Os seguintes campos foram definidos paras as entidades do sistema:

- **Advogado:** Nome completo, email, senha, CPF e número da OAB.
- **Usuário comum:** Nome completo, email, senha, CPF
- **Pacote de créditos:** Nome, quantidade de créditos, valor por crédito, descrição dos benefícios.
- **Dados da compra do pacote de créditos:** Nome completo, email, telefone, cpf, cep, endereço, número, complemento, bairro, cidade, estado.
- **Registro de conteúdo:** Endereço do site, título do registro do conteúdo
- **Registros realizados:** Título, Data/hora início, Data/hora fim, Detalhes, status
- **Detalhes dos registros:** id gerado pelo sistema, Título, Data/hora início, Data/hora fim, tipos de dados registrados, informações dos dados (número de imagens e vídeos), usuário responsável pelo registro
- **Relatório do registro de conteúdo:** Título, id gerado pelo sistema, Usuário, CPF, id do registro gerado pelo sistema, Data/hora início, Data/hora fim, duração, imagens capturadas, site da imagem, url do site navegado, nome do arquivo gerado pelo sistema, tamanho do arquivo.
