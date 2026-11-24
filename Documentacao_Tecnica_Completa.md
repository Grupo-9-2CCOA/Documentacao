# Dossiê Técnico Completo — Projeto Doces com Amor

**Gerado em:** 24/11/2026  
**Escopo:** os 4 repositórios de código do projeto — `Frontend`, `Backend`, `Banco-de-Dados` e `infraestrutura`. O repositório `Documentacao` é apenas o destino deste arquivo e não foi usado como fonte.  
**Regra de ouro:** em caso de conflito, a fonte de verdade é o código nos caminhos abaixo, depois os READMEs dos repositórios, e só então este dossiê. Antes de alterar qualquer comportamento, abra o arquivo citado como evidência.  
**Legenda:** `[OK]` verificado no código · `[DIVERGÊNCIA]` fontes discordantes · `[RISCO]` pode causar falha ou problema de segurança · `[PENDENTE]` não implementado ou não mesclado.

---

## 1. Mapa do projeto

| Repositório | Caminho local | Papel | Stack principal |
| --- | --- | --- | --- |
| Frontend | `C:\Users\guilh\Downloads\frontend\Frontend` | Interface web administrativa | React 19.2 + Vite 8 + Nginx |
| Backend | `C:\Users\guilh\IdeaProjects\Backend` | API REST, regras de negócio, relatórios e publicação de eventos | Java 17 + Spring Boot 3.5.13 |
| Banco de Dados | `C:\Users\guilh\Downloads\banco\Banco-de-Dados` | DDL, usuário local e dados de exemplo | MySQL 8 (scripts SQL estáticos) |
| Infraestrutura | `C:\Users\guilh\OneDrive\Documentos\terraform-PI\infraestrutura` | IaC da AWS, Lambda consumer e backup | Terraform >= 1.5 + Python 3.12 |

**Domínio:** sistema interno (não é e-commerce) para a doceria cadastrar clientes/endereços/pedidos, consultar por calendário, acompanhar pagamento e entrega, gerar relatórios e registrar eventos no Google Calendar. O único usuário é a proprietária/equipe (admin).

**Fluxo de negócio resumido:** login → próximos pedidos/calendário → cadastro de pedido (cliente + endereço + data/hora + descrição + valor) → gravação no MySQL → publicação no RabbitMQ → Lambda consome e cria evento no Google Calendar → ID do evento devolvido ao backend → acompanhamento/edição/reagendamento/cancelamento.

---

## 2. Repositório Frontend

### 2.1 Stack e versões `[OK]`

Fonte: `C:\Users\guilh\Downloads\frontend\Frontend\package.json`

| Tipo | Pacote | Versão declarada |
| --- | --- | --- |
| runtime | `react`, `react-dom` | `^19.2.6` |
| runtime | `react-router-dom` | `^7.17.0` |
| runtime | `@mui/material`, `@mui/icons-material` | `^9.0.1` |
| runtime | `@emotion/react`, `@emotion/styled` | `^11.14.0` / `^11.14.1` |
| dev | `vite` | `^8.0.12` |
| dev | `@vitejs/plugin-react` | `^6.0.1` |
| dev | `eslint` | `^10.3.0` |
| dev | `eslint-plugin-react-hooks` | `^7.1.1` |
| dev | `eslint-plugin-react-refresh` | `^0.5.2` |
| dev | `@eslint/js`, `globals` | `^10.0.1` / `^17.6.0` |

Versões resolvidas no lock: React 19.2.6, Vite 8.2.2, MUI 9.0.1, router 7.18.3, plugin-react 6.0.2. **Não há TypeScript, Vitest nem Jest.** `vite.config.js` contém apenas o plugin React; ESLint usa flat config (`eslint.config.js`, ignora `dist/`).

`[OK]` MUI e Emotion estão instalados, mas a interface é CSS/CSS Modules com ícones da fonte "Material Symbols Outlined" carregada em `index.html`. Não assumir que os componentes MUI são usados.

### 2.2 Scripts `[OK]`

Fonte: `package.json`

| Comando | Ação |
| --- | --- |
| `npm run dev` | Vite dev server (`http://localhost:5173`) |
| `npm run build` | Build de produção em `dist/` |
| `npm run preview` | Serve o build localmente |
| `npm run lint` | `eslint .` |
| `npm run test:unit` | `node tests/filter.test.js` |

### 2.3 Rotas, telas e componentes `[OK]`

Fonte: `src/App.jsx`, `src/pages/`, `src/components/`

| Rota | Arquivo | Protegida | Função |
| --- | --- | --- | --- |
| `/` | `src/pages/Login.jsx` | não | Login; redireciona para `/trocar-senha` se `trocaSenhaObrigatoria`, senão `/pedidos` |
| `/trocar-senha` | `src/pages/TrocaSenha.jsx` | sim (permitirTrocaSenha) | Nova senha, validação local de 6–20 caracteres + confirmação |
| `/novo-pedido` | `src/pages/NovoPedido.jsx` | sim | `[RISCO]` tela estática/mock: usuário fixo "Karla Muniz", links `#`, sem chamadas à API. A criação real está no modal de pedidos |
| `/clientes` | `src/pages/ClientesList.jsx` | sim | Lista ativos/inativos, busca por nome/telefone, criar/editar/inativar/reativar cliente, modais de endereço |
| `/pedidos` | `src/pages/PedidosList.jsx` | sim | Próximos pedidos (`dataPedido >= hoje`), calendário mensal, consulta por data, detalhes, criar/editar/cancelar, troca rápida de status, formatação BRL/pt-BR |
| `/dashboard` | `src/pages/Dashboard.jsx` | sim | Presets 7 dias/30 dias/6 meses/1 ano/personalizado, filtro de mínimo para fidelização, cards com variação vs. período anterior e tabelas |

Componentes em `src/components/`: `ClienteTable`, `NewClienteModal`, `EnderecosModal`, `NovoEnderecoModal`, `EditEnderecoModal` (CEP 8 dígitos, número só dígitos), `ConfirmModal`, `SearchToggle`, `PedidoFormModal`, `Sidebar`, `RotaProtegida`, calendário. `src/assets/components/Sidebar.jsx` está vazio; `Checkbox.jsx` não é usado. No Sidebar, "Calendário" e "Configurações" têm `rota: null` (desabilitados).

### 2.4 Serviços HTTP `[OK]`

Fonte: `src/services/`. Base: `(import.meta.env.VITE_API_BASE_URL || import.meta.env.VITE_API_URL || 'http://localhost:8080').replace(/\/$/, '')`. Todas as requisições usam `fetch` com `credentials: 'include'`, tratamento de erro com `erro.status` e `fieldErrors` do Spring.

| Arquivo | Método e caminho |
| --- | --- |
| `authService.js` | `POST /admin/login`, `GET /admin/sessao`, `POST /admin/trocar-senha`, `POST /admin/logout` |
| `clienteService.js` | `GET /clientes`, `GET /clientes/inativos`, `POST /clientes`, `GET/PUT /clientes/{id}`, `PATCH /clientes/{id}`, `PATCH /clientes/{id}/reativar` |
| `clienteService.js` (endereços) | `GET/POST /enderecos/{clienteId}`, `PUT/DELETE /enderecos/{id}`; normaliza defensivamente `id`, `enderecoId`, `idEndereco`, `codigo`, `_id`; trata `409` com `pedidoIds` |
| `pedidoService.js` | `GET/POST /pedidos`, `GET /pedidos/listarData?dataInicio=&dataFim=`, `GET/PUT/DELETE /pedidos/{id}`, `PATCH /pedidos/{id}/status` com `{pagamentoId, entregaId}` |
| `relatorioService.js` | `GET /relatorio`, `/relatorio/cancelados`, `/relatorio/reagendados`, `/relatorio/clientes` com `dataInicio`, `dataFim`, `minimoPedidosFidelizacao` e datas com offset explícito |

### 2.5 Autenticação no cliente `[OK]`

`RotaProtegida.jsx` chama `GET /admin/sessao` a cada montagem, mostra "Validando sessão...", redireciona para `/` se não autenticado e para `/trocar-senha` se o backend responder `403`. **O frontend não armazena token**: não há `localStorage`, `Authorization` ou `Bearer` em `src/`. A sessão é o cookie HttpOnly do backend. Logout chama `POST /admin/logout` e navega para `/`.

### 2.6 Constantes de status `[DIVERGÊNCIA]`

Fonte: `src/constants/statusPedido.js`

- `STATUS_PAGAMENTO`: `{1: Pendente}, {2: Pago}` — **não inclui Cancelado**.
- `STATUS_ENTREGA`: `{1: Pendente}, {2: Em trânsito}, {3: Entregue}` — **não inclui Cancelado**.
- `ENTREGA_INICIAL = STATUS_ENTREGA[0]`.

O backend e o SQL semeiam também o status `Cancelado` (entrega id 4, pagamento id 3). Ao documentar ou mexer em status, verificar os três repositórios.

### 2.7 Testes `[OK]`

- `tests/filter.test.js`: 4 asserções com `node:assert` sobre `filterClientes` (nome, case-insensitive, substring de telefone, termo vazio). Baseado em `src/utils/filterClientes.js`.
- `tests/test_clientes_page.py` e `tests/test_pedidos_page.py`: Selenium/Chrome headless contra `http://localhost:5173`, com backend ativo, `webdriver-manager`, screenshots em `telas_erros/`. **`[RISCO]` não são executados no CI e estão untracked no Git local.**
- `tests/test-pesquisa-pets-final-aula.py`: script didático legado, sem relação com o projeto.

### 2.8 CI/CD `[OK]`

Fonte: `.github/workflows/publish-ecr.yml`. Gatilhos: `push` em `main` e `workflow_dispatch`. Um job `test-and-publish` executa `npm ci` e, em sequência, `npm run lint && npm run build && npm run test:unit && npm audit --omit=dev`. Depois publica `doces-com-amor-frontend:$GITHUB_SHA` no ECR (`linux/amd64`, cache GHA), pulando o build se a tag já existir. Credenciais: OIDC via `secrets.AWS_ROLE_ARN` ou fallback temporário da AWS Academy.

### 2.9 Docker / Nginx `[OK]`

`Dockerfile`: build multi-stage em `node:22-alpine` com `ARG VITE_API_BASE_URL=/api`, runtime `nginx:1.27-alpine` com `dist` em `/usr/share/nginx/html`, healthcheck `wget --spider http://127.0.0.1/health` (30s/5s/3). `docker/nginx.conf.template`: porta 80, `/health` → `healthy`, proxy `/api/` → `http://${BACKEND_HOST}:${BACKEND_PORT}/` (padrão `backend:8080`) com headers `Host`, `X-Real-IP`, `X-Forwarded-For`, `X-Forwarded-Proto`, e fallback SPA `try_files $uri $uri/ /index.html`. `.dockerignore` exclui `.env*`, `node_modules/`, `dist/`, `.git/`, `.github/`.

`[OK]` O CI não passa `--build-arg VITE_API_BASE_URL`, então a imagem usa `/api` e depende do proxy do Nginx; localmente o `.env` usa `http://localhost:8080`.

---

## 3. Repositório Backend

### 3.1 Stack `[OK]`

Fonte: `C:\Users\guilh\IdeaProjects\Backend\pom.xml`

- `spring-boot-starter-parent` **3.5.13**; `java.version` **17**; group `school.sptech`, artifact `projeto-extensao`, versão `0.0.1-SNAPSHOT`.
- Starters: `web`, `data-jpa`, `security`, `validation`, `actuator`; `spring-security-test` e `spring-boot-starter-test` (test).
- `springdoc-openapi-starter-webmvc-ui` **2.8.16** (Swagger).
- `jjwt-api`/`jjwt-impl`/`jjwt-jackson` **0.12.6**.
- `spring-boot-starter-amqp` + `spring-rabbit-test` (test).
- `mysql-connector-j` (runtime), `h2` (test), `lombok` **1.18.46** (compile).
- `jacoco-maven-plugin` **0.8.14**: `prepare-agent`, `report` e `check` na fase `verify`, regra BUNDLE/BRANCH `COVEREDRATIO` mínimo `0,70` para `school/sptech/*`.
- **Não há MapStruct** (mappers são classes static manuais), **não há Google API client** e **não há roles/perfis**.

### 3.2 Estrutura `[OK]`

`src/main/java/school/sptech/projeto_extensao/`: `Config/`, `controller/`, `dto/`, `mapper/`, `model/`, `repository/`, `service/`, `controller/enums/PeriodoFiltro.java` (`SEMANA, MENSAL, SEMESTRAL, ANUAL` — não usado pelos controllers). Recursos em `src/main/resources/` (`application.properties`, `data.sql`). Testes em `src/test/java/...` e `src/test/resources/application.properties`.

### 3.3 Endpoints implementados `[OK]`

Fonte: `controller/` (5 controllers). Paginação padrão tamanho 10; ordenação por `dataPedido`, `nome` ou `logradouro`.

| Controller | Método e caminho | Observação |
| --- | --- | --- |
| AdminController | `POST /admin` | criar admin (apenas um permitido, 409 se já existir) |
| AdminController | `POST /admin/login` | público |
| AdminController | `POST /admin/logout` | público |
| AdminController | `GET /admin/sessao` | validar sessão |
| AdminController | `POST /admin/trocar-senha` | liberado mesmo com troca obrigatória |
| AdminController | `GET /admin` | — |
| ClienteController | `GET /clientes` | pageable |
| ClienteController | `GET /clientes/inativos` | — |
| ClienteController | `GET /clientes/{id}` | — |
| ClienteController | `POST /clientes` | — |
| ClienteController | `PATCH /clientes/{id}` | inativação lógica |
| ClienteController | `PATCH /clientes/{id}/reativar` | — |
| ClienteController | `PUT /clientes/{id}` | edição |
| EnderecoController | `GET /enderecos/{id}` | na verdade por cliente |
| EnderecoController | `POST /enderecos/{id}` | — |
| EnderecoController | `PUT /enderecos/{id}` | — |
| EnderecoController | `DELETE /enderecos/{id}` | exclusão lógica; bloqueada se houver pedido com entrega ≠ "Entregue" |
| PedidoController | `GET /pedidos` | pageable |
| PedidoController | `GET /pedidos/listarData?dataInicio=&dataFim=` | parâmetros `ISO_DATE_TIME` |
| PedidoController | `GET /pedidos/{id}` | — |
| PedidoController | `POST /pedidos` | publica comando `create` |
| PedidoController | `PUT /pedidos/{id}` | publica comando `update` |
| PedidoController | `PATCH /pedidos/{id}/status` | body `{pagamentoId, entregaId}`; **não publica comando** |
| PedidoController | `DELETE /pedidos/{id}` | soft delete; publica comando `delete` se afetou linhas |
| PedidoController | `PATCH /pedidos/calendar-event/{id}` | público; vincula `eventoGoogleCalendarId` |
| RelatorioController | `GET /relatorio` | `dataInicio`, `dataFim` (OffsetDateTime) |
| RelatorioController | `GET /relatorio/cancelados` | idem |
| RelatorioController | `GET /relatorio/reagendados` | idem |
| RelatorioController | `GET /relatorio/clientes` | idem + `minimoPedidosFidelizacao` (default 2) |

Públicos também: Swagger UI, `/v3/api-docs/**`, `/webjars/**`, `/configuration/**`, `/api/public/**`, `/actuator/health`, `/error/**`. Todo o resto exige autenticação. Não existem controllers de Entrega, Pagamento ou HistoricoPedido.

### 3.4 Entidades e mapeamento `[OK]`

Fonte: `model/` (7 entidades, mesmas do SQL): `Admin`, `Cliente`, `Endereco`, `Pedido`, `Entrega`, `Pagamento`, `HistoricoPedido`.

- Status **não são enums nem entidades separadas**: são strings no campo `estado` de `Entrega` e `Pagamento`, referenciadas por `@ManyToOne` em `Pedido` e `HistoricoPedido`.
- `Pedido`: `produto` → coluna `descricao`; `descricao` → coluna `detalhes`; `valor` → coluna `valor_total`; `isAtivo`, `isReagendado`, `dataPedido`, `dataModificacao`, `dataCriacao`, `eventoGoogleCalendarId`.
- `HistoricoPedido` guarda versão antiga de pedido/cliente/entrega/pagamento + `dataCriacao`, gravada em `editar()` e em `atualizarStatus()` quando há mudança.
- Delete lógico via `isAtivo` em Cliente/Endereco/Pedido.
- `PedidoResponseDto.linkEvento` existe, mas o mapper nunca preenche (sempre `null`; há teste que afirma isso).

### 3.5 Seeds `[OK]`

`src/main/resources/data.sql`: entrega `1 Pendente, 2 Em trânsito, 3 Entregue, 4 Cancelado`; pagamento `1 Pendente, 2 Pago, 3 Cancelado`; admin `admin` com `troca_senha_obrigatoria = true`.  
`[DIVERGÊNCIA]` o repositório de banco também semeia esses status em `inserts.sql` e o admin com senha BCrypt — há duplicação de seeds entre Backend e Banco-de-Dados. `spring.sql.init.mode=always` e `spring.jpa.hibernate.ddl-auto=update` no `application.properties`.

### 3.6 Segurança `[OK]`

Fonte: `Config/SecurityConfiguracao.java`, `AdminController.java`, `GerenciadorTokenJwt.java`.

- Stateless, CSRF desabilitado, CORS por `CORS_ALLOWED_ORIGINS` (default `localhost:5173,localhost:3000`), BCrypt, filtro JWT antes de `UsernamePasswordAuthenticationFilter`.
- Cookie **`authToken`**: `HttpOnly`, `SameSite=Strict`, `path=/`, `maxAge=jwt.validity`, `secure=${COOKIE_SECURE:true}`; logout expira o cookie. O filtro também aceita `Authorization: Bearer ...`.
- Token HMAC com `jwt.secret` em Base64; validade padrão `1209600` s (14 dias, `application.properties`), mas `.env`/`.env.example` usam `3600`. Token expirado → 401 JSON "Sessão expirada" e remoção do cookie.
- Primeiro acesso: `Admin.precisaTrocarSenha` (default `true`); o filtro devolve **403 "Troca de senha obrigatória"** para qualquer rota exceto `/admin/trocar-senha`, `/admin/logout`, `/admin/login`.
- `[OK]` `AdminDetalhesDto.getAuthorities()` retorna `List.of()`; não há `@PreAuthorize`/`@Secured`/`hasRole`. Todos os admins autenticados têm o mesmo acesso.

### 3.7 RabbitMQ — contrato `[OK]`

Fontes: `Config/RabbitMQConfig.java` (lido diretamente), `service/PedidoProducer.java`, `dto/CalendarCommand.java`.

| Item | Valor |
| --- | --- |
| Exchange | `calendar.exchange` (topic), declarada no código |
| Fila | `calendar.events.queue`, `new Queue(QUEUE, true)` (durável) |
| Bindings | `calendar.event.create`, `calendar.event.update`, `calendar.event.delete` |
| Conversor | `Jackson2JsonMessageConverter` |
| Payload | `pedidoId`, `tipo`, `produto`, `descricao`, `dataInicio` (= `dataPedido`), `dataFim` (= `dataPedido + 1h`), `eventoGoogleCalendarId` |
| Publicação | `PedidoProducer.publicarCriacao` após `cadastrar()`; `publicarAtualizacao` após `editar()`; `publicarExclusao` só se o soft delete afetou linhas |
| Não publica | em `atualizarStatus()` (teste verifica `verifyNoInteractions(producer)`) nem no vínculo do evento |
| Falha | `logger.warn` — a gravação no banco **não** é desfeita |
| Config | `spring.rabbitmq.host/port/username/password`, defaults `localhost:5672`, `guest/guest` |

### 3.8 Google Calendar — desacoplado `[OK]`

O backend **não chama mais a API do Google**. O código antigo (`GoogleCalendarService`, `EventoCalendarDto`, dependências `google-api-*`) está preservado apenas em `reference/google-calendar/README.md` como referência histórica. Commits reais do desacoplamento: `321b57b` e `0e2ff69`. O contrato atual é o RabbitMQ da seção 3.7 + a Lambda da seção 5.7.

`[RISCO]` Propriedades obsoletas em `application.properties` linhas 24–26 (`google.calendar.id`, `google.calendar.credentials-path`, `app.google-calendar.enabled`) não são lidas por nenhum código. O README desse commit de referência cita `c421766`, mas o log mostra que esse commit é de padronização de `isAtivo`, não do desacoplamento.

### 3.9 Relatórios `[OK]`

`service/RelatorioServiceTest` é a maior suíte: vendas, comparação de períodos, cancelados, reagendados, clientes e limite 15. Conversão para `America/Sao_Paulo` via `app.timezone` (default `America/Sao_Paulo`). Filtro de fidelização padrão 2 pedidos.

### 3.10 Testes `[OK]`

14 classes em `src/test/java/school/sptech/projeto_extensao/`: `ProjetoExtensaoApplicationTests`, `Config/SecurityConfiguracaoTest`, `repository/PedidoRepositoryTest`, `service/PedidoProducerTest`, `service/PedidoServiceTest`, `service/RelatorioServiceTest`, `service/AdminServiceTest`, `service/AutenticacaoServiceTest`, `service/ClienteServiceTest`, `service/EnderecoServiceTest`, `controller/PedidoControllerTest`, `controller/RelatorioControllerTest`, `dto/pedido/PeriodoFiltroDtoTest`, `mapper/PedidoMapperTest`. Maioria Mockito; 3 usam H2. `src/test/resources/application.properties`: H2 `jdbc:h2:mem:projeto_extensao_test`, `ddl-auto=create-drop`, `sql.init.mode=never`, `jwt.validity=3600` — sem dependência de MySQL.

### 3.11 CI e Docker `[OK]`

Workflow `.github/workflows/publish-ecr.yml`: `push` em `main` ou `workflow_dispatch`, checkout@v4, setup-java@v4 Temurin 17 com cache Maven, **`./mvnw test`**, credenciais AWS via OIDC ou Academy, login ECR, buildx/build-push-action@v6 `linux/amd64` para `doces-com-amor-backend:$GITHUB_SHA`, cache GHA, pula se a tag existe. **`[PENDENTE]` não roda `mvn verify`**, logo o `check` do JaCoCo não é aplicado no CI.

`Dockerfile`: build `eclipse-temurin:17-jdk-alpine`, runtime `17-jre-alpine`, usuário não-root, `TZ=America/Sao_Paulo`, `EXPOSE 8080`, healthcheck em `/actuator/health`.

### 3.12 Configuração `[OK]`

`application.properties`: datasource `jdbc:mysql://localhost:3306/doces_com_amor?useSSL=false&allowPublicKeyRetrieval=true&serverTimezone=UTC`, usuário `docescomamor`, senha vazia por default, driver `com.mysql.cj.jdbc.Driver`, `ddl-auto=update`, `spring.config.import=optional:file:.env[.properties]`. Variáveis externas: `JWT_SECRET`, `JWT_VALIDITY`, `COOKIE_SECURE`, `CORS_ALLOWED_ORIGINS`, `SPRING_RABBITMQ_HOST/PORT/USERNAME/PASSWORD`, `SPRING_DATASOURCE_URL/USERNAME/PASSWORD`, `APP_TIMEZONE`.

---

## 4. Repositório Banco-de-Dados

### 4.1 Arquivos `[OK]`

`C:\Users\guilh\Downloads\banco\Banco-de-Dados` contém exatamente 4 arquivos versionados, na raiz: `README.md`, `doces_com_amor.sql`, `usuario_banco.sql`, `inserts.sql`. Não há `.github/`, Docker, migrations, testes, lint ou `data.sql`. Branch `main` em `1ca9081` ("Add 'evento_google_calendar_id' to pedido table").

### 4.2 DDL `[OK]`

Fonte: `doces_com_amor.sql` (lido diretamente). `CREATE DATABASE doces_com_amor; USE doces_com_amor;`

| Tabela | Colunas |
| --- | --- |
| `admin` | `id` INT AI PK, `usuario` VARCHAR(255) NOT NULL UNIQUE, `senha` VARCHAR(255) NOT NULL, `troca_senha_obrigatoria` BOOLEAN NOT NULL DEFAULT TRUE |
| `cliente` | `id` INT AI PK, `nome` VARCHAR(50) NOT NULL, `cpf` VARCHAR(11) UNIQUE (anulável), `is_ativo` BOOLEAN NOT NULL DEFAULT TRUE, `telefone` VARCHAR(11) |
| `endereco` | `id` INT AI PK, `logradouro` VARCHAR(30) NOT NULL, `numero` VARCHAR(30) NOT NULL, `cep` VARCHAR(8) NOT NULL, `complemento` VARCHAR(50), `is_ativo` BOOLEAN NOT NULL DEFAULT TRUE, `id_cliente` INT NOT NULL FK→`cliente(id)` |
| `entrega` | `id` INT AI PK, `estado` VARCHAR(20) NOT NULL (catálogo de estados) |
| `pagamento` | `id` INT AI PK, `estado` VARCHAR(20) NOT NULL (catálogo de estados) |
| `pedido` | `id` INT AI, `id_cliente` INT NOT NULL FK, `id_endereco` INT NOT NULL FK, `descricao` VARCHAR(100) NOT NULL, `detalhes` VARCHAR(200), `valor_total` DECIMAL(10,2) NOT NULL, `status_entrega` INT NOT NULL FK, `status_pagamento` INT NOT NULL FK, `is_ativo` BOOLEAN NOT NULL DEFAULT TRUE, `is_reagendado` BOOLEAN NOT NULL DEFAULT FALSE, `data_pedido` DATETIME NOT NULL, `data_modificacao` DATETIME, `data_criacao` DATETIME NOT NULL, `evento_google_calendar_id` VARCHAR(255) DEFAULT NULL, **PK composta `(id, id_cliente)`** |
| `historico_pedido` | `id` INT AI, `id_pedido` INT NOT NULL FK→`pedido(id)`, `id_cliente` INT NOT NULL FK→`cliente(id)`, `status_entrega` INT NOT NULL FK, `status_pagamento` INT NOT NULL FK, `data_criacao` DATETIME NOT NULL, **PK composta `(id, id_pedido, id_cliente)`** |

11 FKs no total, todas sem `ON DELETE/UPDATE`. Índices apenas os implícitos de PK/UNIQUE/FK. Sem charset/collation explícitos, sem `ENGINE=`, sem `CHECK`, sem `DEFAULT` em DATETIME, sem views/procedures/triggers. Convenção SQL snake_case ↔ JPA camelCase: `is_ativo`↔`isAtivo`, `is_reagendado`↔`isReagendado`, `evento_google_calendar_id`↔`eventoGoogleCalendarId`, `valor_total`↔`valorTotal`. Status são **catálogos textuais**, não formas de pagamento.

### 4.3 Seeds `[OK]`

`inserts.sql`: 1 admin `admin` com senha BCrypt `$2a$10$vK/rA0mYsNE5knULSc1aSeaSKAGV5Fh/IzcMdlLEwq3v6actLnI/m` (senha padrão "admin", `troca_senha_obrigatoria` omitido → TRUE); 5 clientes fictícios (Ana Silva…Eva Martins, CPFs 11111111111…55555555555); 5 endereços; entrega `Pendente, Em trânsito, Entregue, Cancelado` (1–4); pagamento `Pendente, Pago, Cancelado` (1–3); 10 pedidos fictícios (Bolo de Chocolate, Brigadeiro Gourmet, etc., R$ 35–120), 2 inativos, 2 reagendados, nenhum `evento_google_calendar_id`, nenhum registro em `historico_pedido`.

`[RISCO]` **Bug confirmado em `inserts.sql:32`**: o INSERT de `endereco` usa a coluna `isAtivo`, mas a tabela define `is_ativo` (`doces_com_amor.sql:25`). O script falha com "Unknown column 'isAtivo'". O commit `419f9dd` corrigiu só o DDL.

`[RISCO]` `usuario_banco.sql` cria `CREATE USER 'docescomamor' IDENTIFIED BY '';` (senha vazia, host implícito `%`) e dá `GRANT ALL ... WITH GRANT OPTION`. É configuração apenas local/demonstração.

### 4.4 Setup `[OK]`

Ordem do README: `doces_com_amor.sql` → `usuario_banco.sql` → `inserts.sql` (opcional):

```bash
mysql -u root -p < doces_com_amor.sql
mysql -u root -p < usuario_banco.sql
mysql -u root -p doces_com_amor < inserts.sql
```

`[RISCO]` Scripts não idempotentes: `CREATE DATABASE` sem `IF NOT EXISTS`, sem `DROP TABLE IF EXISTS`, sem `ALTER TABLE`. Reexecutar falha. Não há Flyway/Liquibase nem arquivos de migration numerados — a evolução foi feita editando o DDL diretamente no Git.

---

## 5. Repositório Infraestrutura

### 5.1 Organização `[OK]`

Fonte: `C:\Users\guilh\OneDrive\Documentos\terraform-PI\infraestrutura`

| Arquivo/pasta | Função |
| --- | --- |
| `main.tf` | orquestra 8 módulos (lido diretamente) |
| `variables.tf`, `outputs.tf` | variáveis e outputs (VPC, ALB DNS, URIs, Lambda, secret, backup) |
| `providers.tf`, `versions.tf` | provider AWS com `default_tags` Project/ManagedBy; Terraform >= 1.5, AWS ~> 5.0, random ~> 3.6 |
| `identity.tf` role/profile IAM condicional (`create_ec2_instance_profile`), `random_id.jwt_secret` (32 bytes), validação de profile existente |
| `images.tf` | `data.aws_ecr_repository` + `data.aws_ecr_image` `most_recent = true` → `repository_url@image_digest`; overrides `backend_image_uri`/`frontend_image_uri` |
| `network/` | VPC, subnets, IGW, NAT, route tables, VPC Endpoint S3 |
| `security/` | 7 Security Groups + regras |
| `compute/` | 6 EC2 Ubuntu 24.04 + user-data (Docker, RabbitMQ, MySQL, backup) |
| `load-balancer/` | ALB, target group, listeners |
| `storage/` | Data Lake Bronze/Silver/Gold, `count = var.enable_data_lake ? 1 : 0` |
| `observability/` | dashboard, alarmes, SNS |
| `backup/` | bucket S3 versionado, SNS, política IAM |
| `lambda-consumer/` | Lambda + EventBridge + Secrets Manager + `build.ps1`/`build.sh` |
| `bootstrap/` | estado Terraform separado com os 2 repositórios ECR persistentes |

### 5.2 Rede `[OK]`

1 VPC `10.0.0.0/16`; 5 subnets: públicas `10.0.1.0/24` (A) e `10.0.2.0/24` (B), backend privadas `10.0.3.0/24` (A) e `10.0.4.0/24` (B), DB privada `10.0.5.0/24`; 1 IGW; 1 EIP + 1 NAT Gateway apenas na pública A; 2 route tables (pública→IGW, privada→NAT); **VPC Gateway Endpoint para S3** ligado somente à route table privada.

### 5.3 Computação `[OK]`

6 instâncias **t3.micro** (default) Ubuntu Noble 24.04, IMDSv2 obrigatório, root volume criptografado, `user_data_replace_on_change = true`: `frontend-a`/`frontend-b` (públicas), `backend-a`/`backend-b` (privadas), `mysql` e `mysql-backup` (subnet DB). RabbitMQ é instalado **somente no backend A**; o backend B aponta para o IP privado de A (regra extra 5672 de B→A).

### 5.4 Segurança `[OK]`

7 SGs em `security/` (ALB, frontend A/B, backend A/B, MySQL, lambda-consumer) + 1 SG de backup em `compute/`. Fluxo: internet → ALB (80/443) → frontend (80, só do SG do ALB) → backend pareado (8080, só do frontend correspondente) → MySQL (3306, somente dos 2 backends **e da instância de backup**). Lambda: egress 5672 para backends, 8080 para backend A, 53 no VPC, 443 para internet; ingress correspondente 5672/8080 nos backends. Backend e MySQL sem exposição direta à internet.

### 5.5 ALB `[OK]`

1 ALB internet-facing, target group HTTP com health check `GET /health` (matcher `200-399`, intervalo 30s), 2 attachments, listener HTTP 80 e listener HTTPS 443 **opcional** via `alb_certificate_arn` (policy TLS13).

### 5.6 ECR e imagens `[OK]`

`bootstrap/` mantém estado separado (tag `Lifetime = "Persistent"`) com `doces-com-amor-backend` e `doces-com-amor-frontend`, tags imutáveis, `scan_on_push = true`, lifecycle mantendo 10 imagens. `terraform destroy` na raiz não apaga os repositórios. A raiz usa o digest da imagem mais recente por padrão; `backend_image_uri`/`frontend_image_uri` permitem fixar tag/digest para rollback. As EC2 fazem login no ECR via Instance Profile e rodam os containers reais.

### 5.7 Lambda `calendar-consumer` `[OK]`

Fontes: `lambda-consumer/main.tf`, `lambda-consumer/lambda-src/handler.py` (lido diretamente), `lambda-consumer/README.md`, `build.ps1`/`build.sh`.

| Item | Valor |
| --- | --- |
| Nome | `<project>-calendar-consumer`, Python 3.12, handler `handler.lambda_handler`, 60s, 256 MB |
| Gatilho | **EventBridge**, regra `<project>-calendar-consumer-schedule`, `rate(1 minute)` (default em `lambda-consumer/variables.tf`, não exposto na raiz) + `aws_lambda_permission` |
| Rede | dentro das 2 subnets backend privadas, SG dedicado |
| Broker | abre conexão AMQP para `RABBITMQ_HOST` = IP privado do backend A, porta 5672, usuário/senha `guest` por default |
| Topologia | declara `calendar.exchange` (topic, durable), `calendar.events.queue` (durable) e as 3 routing keys |
| Leitura | `basic_get` até 50 mensagens ou 50 segundos; `ack` no sucesso; `nack(requeue=True)` só para erros Google transitórios (429/500/502/503/504/rate/quota); demais erros descartados |
| Google | service account do Secrets Manager (`GOOGLE_SECRET_ARN`), escopo `https://www.googleapis.com/auth/calendar`, API v3, `America/Sao_Paulo`, título `Pedido #<pedidoId> - <produto>` |
| Ações | `create`→`events.insert`; `update`→`events.update` (exige `eventoGoogleCalendarId`); `delete`→`events.delete` (exige `eventoGoogleCalendarId`) |
| Efeitos | JSON cru (sucesso/falha, body, evento, erro, timestamp) em `s3://<raw_bucket>/raw/google-calendar/YYYY/MM/DD/<pedidoId>-<uuid>.json`; `PATCH http://<BACKEND_HOST>:<BACKEND_PORT>/pedidos/calendar-event/<pedidoId>` com `{"eventoGoogleCalendarId": id}` |
| Env | `RABBITMQ_HOST/PORT/USERNAME/PASSWORD/QUEUE`, `GOOGLE_SECRET_ARN`, `GOOGLE_CALENDAR_ID` (default `primary`), `RAW_BUCKET_NAME`, `RAW_BUCKET_ARN`, `BACKEND_HOST`, `BACKEND_PORT` |
| IAM | role criada só se `lambda_existing_role_arn` for nulo; `AWSLambdaVPCAccessExecutionRole` + read do secret; policy `s3:PutObject` em `raw/google-calendar/*` |
| Log | `/aws/lambda/<project>-calendar-consumer`, retenção 7 dias |

`[RISCO]` Com `lambda_existing_role_arn` (comum na AWS Academy), a permissão de leitura do secret **não** é anexada pelo Terraform. `[RISCO]` O pacote `lambda.zip` precisa ser gerado antes do `apply` (`build.ps1`/`build.sh`).

### 5.8 Backup do MySQL `[PENDENTE: branch]`

`backup/` cria bucket `<project>-mysql-backups-<suffix>` (ou reusa `backup_bucket_name`), bloqueio público, SSE AES256, versionamento, lifecycle 30 dias (`backup_retention_days`) e versões antigas 7 dias; tópico SNS `<project>-mysql-backup-notifications` (`backup_alert_email` → `alarm_email`); política `s3:PutObject`/`sns:Publish` no Instance Profile. `compute/user-data/backup.sh` instala MySQL client e AWS CLI, grava `/opt/mysql-backup/backup_mysql.sh` e registra Cron `/etc/cron.d/mysql-backup` com `0 3 * * *` UTC (meia-noite em Brasília), além de uma execução no bootstrap.

Fluxo: `mysqldump --single-transaction --quick --triggers --no-tablespaces --set-gtid-purged=OFF --default-character-set=utf8mb4 --databases <db>` → `mysql-backup-YYYY-MM-DD.sql` → `s3://<bucket>/mysql-backups/YYYY-MM-DD/` → SNS de sucesso/falha. Log em `/var/log/mysql-backup.log`. Restauração:

```bash
aws s3 cp s3://<bucket>/mysql-backups/AAAA-MM-DD/mysql-backup-AAAA-MM-DD.sql .
mysql -h <host> -u <usuario> -p < mysql-backup-AAAA-MM-DD.sql
```

`[PENDENTE]` Todo esse fluxo existe na branch local `feature/implementacao-backup-banco`, **1 commit à frente de `origin/main`**. Não assumir que está no `main`.

### 5.9 Observabilidade `[OK]`

1 SNS `<project>-infrastructure-alarms` com subscrição email (`alarm_email`, requer confirmação); dashboard `<project>-infrastructure` com CPUUtilization/StatusCheckFailed por instância, `HTTPCode_ELB_5XX_Count`/`UnHealthyHostCount`/`RequestCount` do ALB e `BucketSizeBytes` do data lake quando habilitado. **14 alarmes**: CPU ≥80% (Average, 2×300s) e StatusCheckFailed ≥1 (Maximum, 2×60s) para cada uma das 6 EC2 (12), `UnHealthyHostCount ≥1` (2×60s) e 5XX ≥5 em 300s (2). `alarm_actions` e `ok_actions` → SNS. Não há alarmes para backup, Lambda, RabbitMQ, ECR nem logs.

### 5.10 Data Lake `[OK]`

`enable_data_lake = false` por padrão e em `terraform.tfvars` local → nenhum bucket Bronze/Silver/Gold criado. Motivo: AWS Academy nega `s3:GetBucketObjectLockConfiguration`, consulta exigida pelo provider. Quando habilitado, cria 3 buckets com sufixo aleatório, SSE AES256 e bloqueio público (o código não configura Object Lock). Independentemente disso, os eventos crus do Google Calendar podem ir para `raw_bucket_name`/`raw_bucket_arn` externos.

### 5.11 Variáveis e adaptações para AWS Academy `[OK]`

- `create_ec2_instance_profile` / `ec2_instance_profile_name`: quando IAM é bloqueado, usa profile existente com leitura do ECR (e do secret do Google, se aplicável).
- `lambda_existing_role_arn`: reusa role existente.
- `backup_bucket_name`: reusa bucket existente se a conta negar criação.
- `google_calendar_credentials_file`: caminho do JSON da conta de serviço **fora do repositório** (ex.: `~/.secrets/google-calendar-key.json`); o Terraform grava no Secrets Manager. Valor fica em texto puro no `terraform.tfstate`.
- `[DIVERGÊNCIA]` `google_calendar_secret_arn` existe em `variables.tf`, mas **não é passada ao módulo** nem usada em nenhum `.tf`; a Lambda sempre usa o ARN do secret criado por `aws_secretsmanager_secret.google_calendar`.
- Senha MySQL e JWT não usam Secrets Manager: vão via `user_data` (base64) e ficam no state. Somente a credencial do Google usa Secrets Manager.

### 5.12 Relação com CI/CD `[OK]`

Apenas Backend e Frontend têm `.github/workflows/publish-ecr.yml`. **O repositório de infraestrutura não tem workflows**. Criação/destruição e seleção do digest são locais:

```bash
cd bootstrap && terraform init && terraform apply   # uma vez, ECR persistentes
cd .. && cp terraform.tfvars.example terraform.tfvars
terraform init && terraform validate && terraform plan && terraform apply
terraform destroy   # após testes/apresentações, para interromper custos
```

### 5.13 Estado local e Git `[RISCO]`

Checkout local em `feature/implementacao-backup-banco`, 1 commit à frente de `origin/main`. `terraform.tfstate` local registra apenas parte dos recursos (rede + Lambda/secret), sem EC2/ALB/backup. `terraform.tfvars.bak` e `tfplan2` estão **versionados**; o `.bak` contém valores não-placeholder para `mysql_password` e `google_calendar_secret_arn`. `terraform.tfvars` e `terraform.tfstate` locais também contêm valores sensíveis (ignorados pelo `.gitignore`, mas presentes em disco).

---

## 6. Contratos entre repositórios

### 6.1 Autenticação ponta a ponta

1. `POST /admin/login` (público) com `{usuario, senha}` → backend valida BCrypt, emite JWT e define cookie `authToken` HttpOnly/SameSite=Strict.
2. Navegador reenvia o cookie automaticamente; frontend usa `credentials: 'include'` e nunca lê o token.
3. `GET /admin/sessao` valida a sessão a cada navegação protegida.
4. Se `troca_senha_obrigatoria`, o filtro responde `403`; o frontend redireciona para `/trocar-senha`, que chama `POST /admin/trocar-senha`.
5. `POST /admin/logout` expira o cookie.

### 6.2 Ciclo de vida do pedido e do evento de calendário

```
Frontend POST /pedidos
  → PedidoService.cadastrar() grava MySQL + histórico
  → PedidoProducer.publicarCriacao() → calendar.exchange / calendar.event.create
  → Lambda (EventBridge 1 min) faz basic_get em calendar.events.queue
  → Google Calendar events.insert
  → grava JSON cru em S3 + PATCH /pedidos/calendar-event/{id}
  → backend atualiza eventoGoogleCalendarId no pedido
Edição → calendar.event.update | Exclusão → calendar.event.delete | Troca de status → nenhum comando
```

### 6.3 Nomenclatura de dados

| Conceito | SQL | Java | Frontend |
| --- | --- | --- | --- |
| Pedido ativo | `is_ativo` | `isAtivo` | filtra por data/status |
| Reagendado | `is_reagendado` | `isReagendado` | exibido em detalhes |
| ID do evento | `evento_google_calendar_id` | `eventoGoogleCalendarId` | não exposto diretamente |
| Valor | `valor_total` | `valor` (campo Java) / `valorTotal` | `valorTotal` em BRL |
| Produto | `descricao` | `produto` | `produto` |
| Detalhes | `detalhes` | `descricao` | `descricao` |
| Estado da entrega | `entrega.estado` | `Entrega.estado` (String) | `STATUS_ENTREGA` (ids 1–3) |

### 6.4 Variáveis de ambiente por repositório

| Repositório | Variáveis |
| --- | --- |
| Frontend | `VITE_API_BASE_URL` (ou `VITE_API_URL`), default `http://localhost:8080`; em Docker, `/api` |
| Backend | `SPRING_DATASOURCE_URL/USERNAME/PASSWORD`, `JWT_SECRET`, `JWT_VALIDITY`, `COOKIE_SECURE`, `CORS_ALLOWED_ORIGINS`, `APP_TIMEZONE`, `SPRING_RABBITMQ_HOST/PORT/USERNAME/PASSWORD` |
| Infraestrutura | `AWS_REGION`, `AWS_ROLE_ARN` (CI), `project_name`, `mysql_password`, `jwt_secret` gerado, `rabbitmq_username/password`, `google_calendar_credentials_file`, `alarm_email`, `backup_alert_email`, `backup_schedule`, `backup_retention_days`, `enable_data_lake`, `raw_bucket_name/arn`, `create_ec2_instance_profile`, `ec2_instance_profile_name`, `lambda_existing_role_arn`, `alb_certificate_arn`, `backend_image_uri`/`frontend_image_uri` |

---

## 7. Divergências, riscos e pendências consolidadas

| # | Tipo | Item | Evidência |
| --- | --- | --- | --- |
| 1 | DIVERGÊNCIA | Google Calendar descrito como chamada direta em documentos antigos; hoje é RabbitMQ + Lambda | `PedidoProducer.java`, `lambda-src/handler.py`, `reference/google-calendar/README.md` |
| 2 | DIVERGÊNCIA | MUI instalado mas não usado; UI é CSS Modules | `package.json`, `src/assets/styles/` |
| 3 | DIVERGÊNCIA | Frontend sem JWT explícito; sessão via cookie | `authService.js`, `RotaProtegida.jsx` |
| 4 | DIVERGÊNCIA | Status "Cancelado" existe no backend/SQL, mas falta nas constantes do frontend | `statusPedido.js`, `data.sql`, `inserts.sql` |
| 5 | RISCO | `inserts.sql` usa `isAtivo` onde a tabela tem `is_ativo` | `inserts.sql:32`, `doces_com_amor.sql:25` |
| 6 | RISCO | Scripts SQL não idempotentes e sem sistema de migração | `doces_com_amor.sql` |
| 7 | RISCO | Seeds duplicados entre Backend (`data.sql`) e Banco (`inserts.sql`) | ambos |
| 8 | RISCO | Falha ao publicar no RabbitMQ é apenas logada; pedido permanece gravado | `PedidoProducer.java` |
| 9 | RISCO | Lambda sem permissão de ler o secret quando usa role existente | `lambda-consumer/main.tf` |
| 10 | RISCO | `google_calendar_secret_arn` declarado mas não usado | `variables.tf`, `main.tf` |
| 11 | RISCO | Propriedades `google.calendar.*` obsoletas no backend | `application.properties:24-26` |
| 12 | RISCO | Artefatos sensíveis versionados (`terraform.tfvars.bak`, `tfplan2`) e state local com segredos | repositório de infraestrutura |
| 13 | RISCO | RabbitMQ sem HA, em uma única EC2; Lambda faz polling com `basic_get` | `compute/user-data/backend.sh`, `handler.py` |
| 14 | PENDENTE | Backup do banco existe apenas na branch `feature/implementacao-backup-banco` | Git local da infraestrutura |
| 15 | PENDENTE | JaCoCo configurado, mas `verify` não roda no CI | `pom.xml`, `publish-ecr.yml` |
| 16 | PENDENTE | `/novo-pedido` é tela mock | `src/pages/NovoPedido.jsx` |
| 17 | PENDENTE | Testes Selenium locais não versionados nem executados no CI | `tests/*.py`, `.github/workflows/` |
| 18 | PENDENTE | Data Lake desativado por limitação da AWS Academy | `variables.tf`, `main.tf` |
| 19 | PENDENTE | Sem roles/perfis no backend; sem MapStruct | `AdminDetalhesDto`, `pom.xml` |
| 20 | PENDENTE | Logs não centralizados; sem alarmes para Lambda/backup/RabbitMQ | `observability/main.tf` |

---

## 8. Checklist de verificação para a IA

Antes de afirmar ou alterar algo, reconcile com os comandos abaixo:

```bash
# Frontend
cd "C:\Users\guilh\Downloads\frontend\Frontend"
git status --short
npm run lint
npm run test:unit

# Backend
cd "C:\Users\guilh\IdeaProjects\Backend"
git status --short
.\mvnw.cmd test

# Banco de Dados
cd "C:\Users\guilh\Downloads\banco\Banco-de-Dados"
git status --short

# Infraestrutura
cd "C:\Users\guilh\OneDrive\Documentos\terraform-PI\infraestrutura"
git status --short
git branch --show-current
terraform validate
```

Pontos que exigem confirmação adicional antes de virar requisito: itens **5, 10, 12 e 14** da tabela de divergências (bug do SQL, variável não usada, segredos versionados e branch do backup).
