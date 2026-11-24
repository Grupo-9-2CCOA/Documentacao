# Documentação do Projeto de Extensão — Doces com Amor

**Grupo 9 — Pesquisa e Inovação**<br>
**Integrantes:** Gabriela Vilegas, Guilherme Santos, João Carmo e Nicolas Javed<br>
**Instituição:** São Paulo Tech School<br>
**Ano:** 2026<br>
**Atualização:** novembro de 2026 — revisão feita a partir do estado atual dos quatro repositórios de código: Frontend, Backend, Banco-de-Dados e infraestrutura.

## Sumário

1. Contexto
2. Objetivos
3. Justificativa
4. Escopo
5. Requisitos
6. Premissas
7. Restrições
8. Artefatos relacionados às regras de negócio
9. Artefatos relacionados às arquiteturas
10. Estrutura dos repositórios
11. Product backlog
12. Ferramentas de planejamento e organização
13. Links do projeto no Git
14. Referências bibliográficas e técnicas

## Visão rápida

O Doces com Amor é um sistema interno para organizar os pedidos de uma doceria.
A ideia é deixar em um só lugar as informações que antes podiam ficar espalhadas
em mensagens, anotações e agendas. O sistema permite cadastrar clientes,
endereços e pedidos, acompanhar datas pelo calendário e consultar informações de
pagamento e entrega.

O frontend foi desenvolvido com React 19 e Vite 8, com interface em CSS e CSS
Modules. O backend usa Java 17 e Spring Boot 3.5, com autenticação JWT em cookie
HttpOnly, API REST documentada com Swagger e publicação de comandos no RabbitMQ.
O banco é MySQL 8, com esquema, usuário e dados de exemplo versionados em um
repositório próprio. A integração com o Google Calendar foi desacoplada: uma
Lambda em Python consome os comandos do RabbitMQ e registra os eventos na agenda.
Toda a aplicação roda em containers Docker e a infraestrutura AWS é criada com
Terraform, incluindo backup diário do banco e monitoramento no CloudWatch.

---

## 1. Contexto

A Doces com Amor trabalha com pedidos que precisam ser preparados e entregues
em datas e horários combinados. Quando esse controle é feito de forma manual,
fica mais fácil esquecer uma entrega, perder uma informação do cliente ou não
ter clareza sobre o pagamento.

O projeto surgiu para apoiar a rotina da proprietária e da equipe da doceria.
Ele não é uma loja virtual para o cliente final. É uma ferramenta de uso interno
para consultar o que precisa ser feito e manter os pedidos organizados.

Os principais problemas trabalhados foram:

- informações de pedidos espalhadas em lugares diferentes;
- dificuldade para visualizar os próximos pedidos;
- risco de esquecer datas, horários e cobranças;
- falta de um histórico centralizado de clientes e endereços;
- pouca visão sobre pedidos cancelados, reagendados e concluídos;
- ausência de rotina automática de backup do banco de dados.

## 2. Objetivos

### Objetivo geral

Desenvolver um sistema web que ajude a Doces com Amor a cadastrar, consultar e
acompanhar seus pedidos de forma simples.

### Objetivos específicos

- centralizar clientes, endereços e pedidos em um banco de dados;
- permitir a consulta dos pedidos por data em um calendário;
- mostrar os próximos pedidos em ordem de entrega;
- acompanhar o status de pagamento e de entrega;
- permitir edição, cancelamento e reagendamento de pedidos;
- apresentar indicadores básicos em um dashboard;
- registrar os pedidos na agenda do Google de forma assíncrona, sem acoplar a
  API ao serviço externo;
- garantir backup diário do banco e possibilidade de restauração;
- disponibilizar o sistema em uma infraestrutura AWS reproduzível.

## 3. Justificativa

A organização dos pedidos interfere diretamente na produção e no atendimento.
Um horário esquecido ou um pagamento sem acompanhamento pode gerar prejuízo e
desgaste com o cliente.

A solução proposta é pequena o suficiente para ser usada no dia a dia, mas já
resolve pontos importantes: reúne os dados, facilita a consulta por data e reduz
a dependência de anotações manuais. Para o grupo, o projeto também permitiu
aplicar conhecimentos de frontend, backend, banco de dados, cloud e trabalho em
equipe em um problema real.

A evolução da integração com o Google Calendar mostrou outro ponto importante:
ao desacoplar a API do serviço externo por meio de fila e consumidor, o cadastro
do pedido deixa de depender da disponibilidade da agenda e a solução passa a ser
mais fácil de manter e evoluir.

## 4. Escopo

### Incluído nesta versão

- login administrativo e troca obrigatória da senha inicial (admin único);
- cadastro, consulta, edição, ativação e inativação de clientes;
- cadastro, consulta, edição e exclusão lógica de endereços;
- cadastro e listagem paginada de pedidos;
- calendário mensal para filtrar pedidos por data;
- consulta dos próximos pedidos;
- detalhes, edição, cancelamento e reagendamento de pedidos;
- atualização dos status de pagamento e entrega;
- relatórios e dashboard com filtros de período e fidelização;
- publicação de comandos de calendário no RabbitMQ (`calendar.exchange`);
- consumidor Lambda em Python que cria, atualiza e exclui eventos no Google
  Calendar e devolve o ID do evento ao backend;
- registro dos eventos crus do Google Calendar em bucket S3;
- backup diário do MySQL em instância EC2 dedicada, com bucket S3 versionado e
  notificação por SNS;
- API REST documentada com Swagger;
- containers Docker para frontend (Nginx) e backend;
- publicação das imagens no Amazon ECR pelo GitHub Actions;
- infraestrutura AWS criada com Terraform, com seleção automática da imagem mais
  recente e parâmetros de rollback;
- dashboard, métricas e alarmes no CloudWatch;
- listener HTTPS opcional no ALB;
- simulação local dos buckets Bronze, Silver e Gold com LocalStack;
- scripts SQL versionados para criar o banco, o usuário local e dados de exemplo;
- documentação do projeto mantida em Markdown e exportada em PDF.

### Fora do escopo desta versão

- aplicativo para o cliente fazer o próprio pedido;
- pagamento online dentro do sistema;
- integração com aplicativos de entrega;
- emissão de nota fiscal;
- controle de estoque e compra de ingredientes;
- aplicativo mobile nativo;
- funcionamento offline;
- cluster RabbitMQ ou fila gerenciada (SQS) — o broker roda em uma única EC2;
- Data Lake integrado de ponta a ponta;
- suporte e manutenção após o encerramento do projeto acadêmico.

## 5. Requisitos

### Requisitos funcionais

| ID | Requisito |
| --- | --- |
| RF01 | O administrador deve conseguir entrar com usuário e senha. |
| RF02 | No primeiro acesso, o administrador deve trocar a senha inicial. |
| RF03 | O sistema deve permitir cadastrar, consultar e editar clientes. |
| RF04 | O sistema deve permitir cadastrar e gerenciar endereços dos clientes. |
| RF05 | O sistema deve permitir cadastrar um pedido para uma data e horário. |
| RF06 | O sistema deve impedir o cadastro de pedido em data passada. |
| RF07 | O sistema deve listar os próximos pedidos em ordem de data. |
| RF08 | O usuário deve conseguir filtrar pedidos pelo calendário. |
| RF09 | O usuário deve conseguir editar, cancelar e reagendar um pedido. |
| RF10 | O usuário deve conseguir atualizar os status de pagamento e entrega. |
| RF11 | O sistema deve mostrar relatórios e indicadores dos pedidos. |
| RF12 | O backend deve publicar um comando no RabbitMQ para criar, atualizar ou excluir o evento do pedido no Google Calendar. |
| RF13 | Rotas de negócio devem exigir autenticação. |
| RF14 | O sistema deve armazenar o ID do evento do Google Calendar devolvido pelo consumidor. |
| RF15 | O banco deve ser copiado diariamente e a cópia deve poder ser restaurada. |

### Requisitos não funcionais

| ID | Requisito |
| --- | --- |
| RNF01 | A interface deve funcionar em computador e possuir responsividade básica. |
| RNF02 | O backend deve disponibilizar uma API REST documentada com Swagger. |
| RNF03 | As senhas devem ser armazenadas com BCrypt. |
| RNF04 | A autenticação deve usar JWT no cookie HttpOnly `authToken` (`SameSite=Strict`); o frontend mantém a sessão com `credentials: 'include'` e validação em `/admin/sessao`. |
| RNF05 | O banco utilizado deve ser MySQL 8. |
| RNF06 | Frontend e backend devem poder ser executados em containers Docker. |
| RNF07 | A infraestrutura AWS deve ser versionada com Terraform. |
| RNF08 | Backend e banco não devem ficar expostos diretamente à internet na AWS. |
| RNF09 | Credenciais e senhas não devem ser salvas nos repositórios. |
| RNF10 | O projeto deve possuir testes automatizados: 14 classes no backend (H2/Mockito) e teste unitário de filtro no frontend, executados no CI. |
| RNF11 | A integração com o Google Calendar deve ser assíncrona; falha na publicação da mensagem não deve desfazer o cadastro do pedido. |
| RNF12 | O backup do banco deve ter retenção de 30 dias e notificar sucesso ou falha por e-mail. |
| RNF13 | As imagens publicadas no ECR devem ter tag imutável baseada no commit. |

## 6. Premissas

Para o projeto funcionar como planejado, consideramos que:

- a proprietária ou alguém da equipe fará o cadastro dos pedidos;
- os clientes não acessarão o sistema diretamente;
- a equipe terá acesso à internet e a um computador;
- as informações do pedido serão conferidas antes do cadastro;
- a conta administrativa inicial será usada apenas uma vez e a senha será trocada;
- a integração com agenda depende de uma conta de serviço válida do Google, com
  credencial armazenada no Secrets Manager;
- o RabbitMQ está disponível na EC2 do backend A para receber os comandos;
- o ambiente AWS Academy estará disponível durante testes e apresentações;
- as imagens do frontend e backend estarão publicadas no ECR antes do deploy;
- os dados usados em demonstrações não devem expor informações pessoais reais.

## 7. Restrições

- o projeto foi desenvolvido durante o período letivo e por uma equipe ainda em formação;
- a AWS Academy possui tempo de sessão e permissões limitadas;
- as credenciais da AWS Academy expiram e precisam ser renovadas;
- alguns serviços podem gerar custos, por isso o ambiente deve ser destruído após o uso;
- o Object Lock do S3 não pode ser consultado pelo perfil da Academy, então o Data Lake ficou desativado no Terraform (`enable_data_lake = false`);
- o sistema depende de internet para acessar a AWS e o Google Calendar;
- o RabbitMQ roda em uma única instância (backend A), sem alta disponibilidade; se o broker estiver fora, a publicação falha e o erro é apenas registrado em log;
- o consumidor não recebe um push do RabbitMQ: a Lambda é invocada por uma regra do EventBridge a cada 1 minuto e faz polling da fila;
- a credencial do Google fica em texto puro no `terraform.tfstate`, portanto esse arquivo precisa ser protegido;
- a versão atual utiliza MySQL em EC2, não Amazon RDS;
- a variável `google_calendar_secret_arn` está declarada, mas ainda não é usada pelo módulo da Lambda;
- configurações antigas `google.calendar.*` e `app.google-calendar.enabled` permanecem no `application.properties` do backend, mas não são lidas pelo código;
- o módulo de backup está na branch `feature/implementacao-backup-banco`, ainda pendente de merge na `main`;
- artefatos sensíveis versionados no repositório de infraestrutura (`terraform.tfvars.bak` e `tfplan2`) devem ser removidos do histórico e as credenciais expostas devem ser rotacionadas.

## 8. Artefatos relacionados às regras de negócio

### Usuário principal

O usuário principal é a proprietária ou integrante da equipe da Doces com Amor.
Ela precisa consultar rapidamente os pedidos, cadastrar informações e atualizar
o andamento do trabalho.

### Fluxo principal do pedido

```text
Login (admin único, troca de senha obrigatória no primeiro acesso)
  ↓
Consulta dos próximos pedidos e seleção de data no calendário
  ↓
Cadastro do pedido: cliente, endereço, data, horário, descrição e valor
  ↓
Gravação no MySQL e histórico do pedido
  ↓
Publicação de calendar.event.create no RabbitMQ (backend → exchange)
  ↓
Lambda acionada pelo EventBridge: leitura da fila e chamada ao Google Calendar
  ↓
ID do evento devolvido ao backend por PATCH /pedidos/calendar-event/{id}
  ↓
Acompanhamento, edição, reagendamento ou cancelamento
  (comandos calendar.event.update e calendar.event.delete)
```

### Regras de negócio principais

| ID | Regra |
| --- | --- |
| RN01 | Somente usuário autenticado pode acessar clientes, endereços, pedidos e relatórios. |
| RN02 | No primeiro acesso, a troca da senha inicial é obrigatória; a API responde `403` até a troca. |
| RN03 | Um pedido deve estar relacionado a um cliente e a um endereço ativos. |
| RN04 | Não é permitido cadastrar pedido em uma data anterior ao dia atual. |
| RN05 | Para o dia atual, o horário deve ser posterior ao horário do cadastro. |
| RN06 | Os próximos pedidos são exibidos do mais próximo para o mais distante. |
| RN07 | Pedidos entregues continuam visíveis para facilitar a conferência. |
| RN08 | Um pedido cancelado deixa de fazer parte do fluxo ativo, mas permanece no histórico e nos relatórios. |
| RN09 | O reagendamento deve registrar a nova data e marcar o pedido como reagendado. |
| RN10 | O status de pagamento cancelado não deve ser usado como atualização comum de um pedido ativo. |
| RN11 | Cliente com pedido ativo não deve ser removido sem tratamento do vínculo. |
| RN12 | Dados de relatório devem respeitar o período informado pelo usuário, em `America/Sao_Paulo`. |
| RN13 | A falha ao publicar no RabbitMQ é registrada em log e não desfaz o cadastro do pedido. |
| RN14 | O ID do evento do Google Calendar é vinculado ao pedido pelo consumidor. |
| RN15 | Edições e mudanças de status geram registro no histórico do pedido. |

### Jornada resumida

| Momento | Ação da usuária | Resposta esperada do sistema |
| --- | --- | --- |
| Entrada | Faz login. | Valida os dados, cria o cookie de sessão e abre a área de pedidos. |
| Planejamento | Consulta calendário e próximos pedidos. | Mostra datas e pedidos em ordem. |
| Cadastro | Escolhe cliente, endereço, data e dados do pedido. | Valida, salva no banco e publica o comando de calendário. |
| Acompanhamento | Abre um pedido e consulta seus detalhes. | Exibe cliente, endereço, valor e status. |
| Atualização | Altera status, edita ou reagenda. | Salva a mudança, atualiza a listagem e publica o comando quando necessário. |
| Análise | Abre o dashboard. | Exibe indicadores do período selecionado. |

## 9. Artefatos relacionados às arquiteturas

### Arquitetura da aplicação

```text
Navegador
   | HTTP/JSON (cookie HttpOnly)
   v
Frontend React 19 (Nginx, proxy /api)
   |
   | HTTP/JSON
   v
API Spring Boot 3.5 (JWT em cookie)
   |             \
   v              v
MySQL 8       RabbitMQ (calendar.exchange)
                     |
                     | EventBridge (a cada 1 min)
                     v
              Lambda Python 3.12
              |       |        \
   Google Calendar  S3 (raw)   PATCH /pedidos/calendar-event/{id}
```

O frontend não manipula o token diretamente: ele envia `credentials: 'include'`,
consulta `/admin/sessao` para validar a sessão e redireciona para `/trocar-senha`
quando o backend responde `403`.

### Integração assíncrona com o Google Calendar

| Item | Definição |
| --- | --- |
| Exchange | `calendar.exchange` (topic, durável) |
| Fila | `calendar.events.queue` (durável) |
| Routing keys | `calendar.event.create`, `calendar.event.update`, `calendar.event.delete` |
| Payload | `pedidoId`, `tipo`, `produto`, `descricao`, `dataInicio`, `dataFim` (+1 hora) e `eventoGoogleCalendarId` |
| Publicação | `PedidoProducer` após criar, editar ou excluir um pedido; não publica em mudança de status |
| Consumo | Lambda Python 3.12, acionada por `rate(1 minute)`, com `basic_get` de até 50 mensagens ou 50 segundos |
| Google Calendar | API v3, escopo `calendar`, fuso `America/Sao_Paulo`, título `Pedido #<id> - <produto>` |
| Resultado | `ack` no sucesso; `nack(requeue=True)` apenas em erros transitórios (429/5xx/quota); demais erros são descartados |
| Efeitos | grava JSON bruto em `s3://<bucket>/raw/google-calendar/YYYY/MM/DD/` e chama `PATCH /pedidos/calendar-event/{pedidoId}` |
| Segredo | conta de serviço do Google no Secrets Manager, lida pela Lambda via `GOOGLE_SECRET_ARN` |

### Arquitetura de deploy na AWS

```text
                         Internet
                            |
                            v
              Application Load Balancer (HTTP / HTTPS opcional)
                            |
              +-------------+-------------+
              |                           |
        Frontend A                   Frontend B      (subnets públicas)
        (Nginx :80)                  (Nginx :80)
              |                           |
        Backend A                    Backend B       (subnets privadas)
        (API :8080 + RabbitMQ)       (API :8080)
              |                           |
              +-------------+-------------+
                            v
                       MySQL 8 EC2                   (subnet privada do banco)
                            |
              +-------------+-------------+
              |                           |
      EC2 backup (Cron)            Bucket S3 versionado (30 dias)
      mysqldump diário  ────────►   + notificação SNS
                            |
                     VPC Endpoint S3 (data lake opcional Bronze/Silver/Gold)

      Lambda calendar-consumer (subnets privadas) ⟷ RabbitMQ (backend A)
      EventBridge (1 min) → Lambda → Google Calendar → S3 (raw) → PATCH backend
```

Outros componentes usados:

- Amazon ECR para armazenar as imagens Docker, com estado Terraform separado em
  `bootstrap/` e tags imutáveis por commit;
- NAT Gateway para saída das instâncias privadas (somente na subnet pública A);
- Security Groups para limitar a comunicação entre camadas;
- Secrets Manager para a credencial do Google Calendar;
- CloudWatch para dashboard, 14 alarmes e log group da Lambda;
- SNS como destino dos alarmes e das notificações de backup;
- IAM Instance Profile para leitura do ECR e escrita do backup no S3;
- Terraform para criar e destruir o ambiente.

### Organização do banco de dados

```text
Admin (único)

Cliente 1 ─── N Endereço
   |
   └──── 1 ─── N Pedido
                    |
                    +── Entrega (estado: Pendente, Em trânsito, Entregue, Cancelado)
                    +── Pagamento (estado: Pendente, Pago, Cancelado)
                    +── Histórico do Pedido
                    └── eventoGoogleCalendarId
```

Entidades atuais do backend:

- `Admin`;
- `Cliente`;
- `Endereco`;
- `Pedido`;
- `Entrega`;
- `Pagamento`;
- `HistoricoPedido`.

Os status são tabelas de domínio com campo `estado` textual, semeadas por
`data.sql`. Não existem entidades separadas para status do pedido, tipo de entrega
ou meio de pagamento. Cliente, Endereço e Pedido usam exclusão lógica (`isAtivo`),
e o pedido registra `isReagendado`, `dataModificacao` e o ID do evento do Google.

### Backup do banco de dados

O backup roda em uma EC2 dedicada, sem alterar o `user_data` da instância MySQL:

```text
mysqldump completo -> mysql-backup-AAAA-MM-DD.sql -> upload S3 -> e-mail SNS
```

- agendamento padrão: `0 3 * * *` em UTC (meia-noite em Brasília);
- retenção padrão: 30 dias (configurável por `backup_retention_days`);
- dump com `--single-transaction` e `--no-tablespaces`;
- caminho: `s3://<bucket>/mysql-backups/AAAA-MM-DD/mysql-backup-AAAA-MM-DD.sql`;
- log em `/var/log/mysql-backup.log`;
- a assinatura SNS precisa ser confirmada por e-mail.

Restauração:

```bash
aws s3 cp s3://<bucket>/mysql-backups/AAAA-MM-DD/mysql-backup-AAAA-MM-DD.sql .
mysql -h <host> -u <usuario> -p < mysql-backup-AAAA-MM-DD.sql
```

### Tecnologias

| Camada | Tecnologias |
| --- | --- |
| Frontend | React 19.2, Vite 8, React Router 7, JavaScript, CSS Modules, ESLint 10, Nginx |
| Backend | Java 17, Spring Boot 3.5.13, Spring Security, JJWT 0.12.6, Spring Data JPA, springdoc/Swagger 2.8.16, Spring AMQP (RabbitMQ), Lombok, JaCoCo |
| Banco | MySQL 8 (H2 em memória nos testes) |
| Integração | RabbitMQ, Google Calendar API v3, consumidor Lambda em Python 3.12 |
| Containers | Docker (multi-stage), Nginx, ECR |
| Cloud | AWS EC2, ALB, VPC, NAT Gateway, VPC Endpoint S3, ECR, S3, Lambda, EventBridge, Secrets Manager, CloudWatch, SNS, IAM |
| Infraestrutura como código | Terraform (>= 1.5, provider AWS 5.x) |
| Automação | GitHub Actions |

### CI/CD atual

Após um merge na `main` do frontend ou backend, o GitHub Actions executa os
testes, faz o build da imagem e publica uma versão no ECR com a tag imutável do
commit (`$GITHUB_SHA`):

- **Frontend:** `npm ci`, `npm run lint`, `npm run build`, `npm run test:unit` e
  `npm audit --omit=dev`; build multi-stage com Nginx e proxy `/api` para o
  backend configurado por `BACKEND_HOST`/`BACKEND_PORT`;
- **Backend:** `./mvnw test` com Java 17; o JaCoCo está configurado com mínimo de
  70% de cobertura na fase `verify`, mas essa etapa ainda não é executada no CI.

A autenticação no AWS usa OIDC via `AWS_ROLE_ARN` quando disponível; na AWS
Academy são usadas credenciais temporárias. O build é pulado se a tag já existir
no ECR. O deploy da infraestrutura é feito separadamente pelo Terraform, e a raiz
seleciona automaticamente o digest da imagem mais recente; `backend_image_uri` e
`frontend_image_uri` permitem fixar uma versão para rollback.

## 10. Estrutura dos repositórios

### Frontend

```text
src/
├── pages/          Login, TrocaSenha, ClientesList, PedidosList, Dashboard
├── components/     tabelas, modais, calendário, sidebar, rota protegida
├── services/       authService, clienteService, pedidoService, relatorioService
├── assets/         CSS Modules, estilos e componentes visuais
├── App.jsx         rotas: /, /trocar-senha, /novo-pedido, /clientes, /pedidos, /dashboard
└── main.jsx
tests/filter.test.js
docker/nginx.conf.template
.github/workflows/publish-ecr.yml
```

A URL da API vem de `VITE_API_BASE_URL` (padrão local `http://localhost:8080`; em
container, `/api` via proxy do Nginx). Os endereços são gerenciados em modais
dentro de Clientes e o calendário fica embutido em Pedidos. A rota `/novo-pedido`
ainda é uma tela estática de demonstração; a criação real acontece no modal de
pedidos.

### Backend

```text
src/main/java/school/sptech/projeto_extensao/
├── Config/         segurança, JWT, CORS, RabbitMQ, OpenAPI
├── controller/     Admin, Cliente, Endereco, Pedido, Relatorio
├── dto/            requisições e respostas da API
├── mapper/         conversão manual entre entidades e DTOs
├── model/          7 entidades JPA
├── repository/     repositórios Spring Data
└── service/        regras de negócio, relatórios e PedidoProducer
src/test/java/...   14 classes de teste
src/main/resources/data.sql
.github/workflows/publish-ecr.yml
```

Os principais endpoints são `/admin`, `/clientes`, `/enderecos`, `/pedidos` e
`/relatorio`, além de `/actuator/health` e do Swagger em `/swagger-ui/index.html`.

### Infraestrutura

```text
bootstrap/          estado separado com os repositórios ECR persistentes
network/            VPC, subnets, IGW, NAT, route tables, VPC Endpoint S3
security/           Security Groups
compute/            6 EC2 (frontend, backend, MySQL, backup) e scripts user-data
load-balancer/      ALB, target group e listeners
storage/            Data Lake Bronze/Silver/Gold (desabilitado por padrão)
observability/      dashboard, alarmes e tópico SNS
backup/             bucket versionado, política IAM e SNS de backup
lambda-consumer/    Lambda, EventBridge, Secrets Manager e build do pacote
images.tf           consulta ao ECR e seleção da imagem mais recente
identity.tf         IAM do Instance Profile e segredo JWT
```

## 11. Product backlog

| Prioridade | Item | Situação |
| --- | --- | --- |
| Alta | Login e troca obrigatória de senha | Concluído |
| Alta | Proteção das rotas com JWT em cookie HttpOnly | Concluído |
| Alta | Cadastro e gestão de clientes | Concluído |
| Alta | Cadastro e gestão de endereços | Concluído |
| Alta | Listagem de pedidos e calendário | Concluído |
| Alta | Cadastro e detalhes do pedido | Concluído |
| Alta | Edição, cancelamento e reagendamento | Concluído |
| Alta | Atualização de pagamento e entrega | Concluído |
| Alta | Testes automatizados e publicação no ECR | Concluído |
| Média | Dashboard e relatórios por período | Concluído |
| Média | Integração com Google Calendar via RabbitMQ e Lambda | Concluído |
| Média | Infraestrutura AWS com Terraform | Concluído |
| Média | Backup diário do MySQL com S3 e SNS | Concluído na branch `feature/implementacao-backup-banco` |
| Média | CloudWatch, dashboard e alarmes | Concluído em nível básico |
| Alta | Separar status do pedido, status da entrega, tipo de entrega, status do pagamento e meio de pagamento | Próxima versão |
| Alta | Melhorar o tratamento de falhas do Google Calendar (retry, fila de dead letter e aviso ao usuário) | Próxima versão |
| Alta | Mesclar a branch de backup na `main` e remover artefatos sensíveis versionados | Próxima versão |
| Alta | Corrigir `google_calendar_secret_arn` não utilizado e remover propriedades `google.calendar.*` obsoletas | Próxima versão |
| Média | Docker Compose para executar todo o projeto localmente, incluindo RabbitMQ | Próxima versão |
| Média | Executar a verificação do JaCoCo no CI | Próxima versão |
| Média | Migrações de banco com Flyway | Futuro |
| Média | Centralização dos logs no CloudWatch | Futuro |
| Média | Habilitar HTTPS no ALB com certificado ACM | Futuro |
| Baixa | Integração real do Data Lake Bronze, Silver e Gold | Futuro |
| Baixa | Migração para RDS e ECS/Fargate | Futuro |

## 12. Ferramentas de planejamento e organização

| Ferramenta | Uso no projeto |
| --- | --- |
| Git e GitHub | Versionamento, branches, Pull Requests e histórico das entregas. |
| GitHub Actions | Testes, lint, build e publicação das imagens Docker; OIDC ou credenciais temporárias da Academy. |
| Trello | Organização do backlog em sprints com as colunas To Do, Doing e Done. |
| Figma | Referência visual para as telas. |
| Swagger | Consulta e teste dos endpoints do backend. |
| MySQL Workbench | Execução e conferência dos scripts do banco. |
| Terraform | Planejamento, criação e destruição da infraestrutura. |
| AWS Console | Conferência dos recursos durante testes e apresentações. |
| RabbitMQ Management | Conferência da exchange, da fila e dos comandos de calendário. |
| JaCoCo e ESLint | Cobertura de testes no backend e análise estática no frontend. |
| LocalStack | Simulação local dos buckets Bronze, Silver e Gold. |
| Reuniões e validações | Alinhamento das telas e do fluxo com a beneficiária. |

O grupo trabalhou com branches por funcionalidade, revisão por Pull Request,
commits em português e um quadro Kanban no Trello. O backlog foi separado por
sprints e acompanhado pelas colunas To Do, Doing e Done.

## 13. Links do projeto no Git

- Documentação: <https://github.com/Grupo-9-2CCOA/Documentacao>
- Frontend: <https://github.com/Grupo-9-2CCOA/Frontend>
- Backend: <https://github.com/Grupo-9-2CCOA/Backend>
- Banco de Dados: <https://github.com/Grupo-9-2CCOA/Banco-de-Dados>
- Infraestrutura: <https://github.com/Grupo-9-2CCOA/infraestrutura>

## 14. Referências bibliográficas e técnicas

- AMAZON WEB SERVICES. **Documentação da AWS**. Disponível em: <https://docs.aws.amazon.com/>.
- DOCKER. **Docker Documentation**. Disponível em: <https://docs.docker.com/>.
- FOOD CONNECTION. **Tendências para o mercado de confeitaria**. Disponível em: <https://www.foodconnection.com.br/foodservice/5-tendencias-para-o-mercado-de-confeitaria/>.
- GOOGLE. **Google Calendar API**. Disponível em: <https://developers.google.com/calendar/api>.
- HASHICORP. **Terraform Documentation**. Disponível em: <https://developer.hashicorp.com/terraform/docs>.
- MYSQL. **MySQL 8.0 Reference Manual**. Disponível em: <https://dev.mysql.com/doc/refman/8.0/en/>.
- RABBITMQ. **RabbitMQ Documentation**. Disponível em: <https://www.rabbitmq.com/docs>.
- REACT. **React Documentation**. Disponível em: <https://react.dev/>.
- SPRING. **Spring Boot Reference Documentation**. Disponível em: <https://docs.spring.io/spring-boot/>.
- SPRING. **Spring AMQP Reference**. Disponível em: <https://docs.spring.io/spring-amqp/reference/>.
