# 🛠️ Architecture / Software Design Document

**Projeto:** insc.gg — Sistema de Inscrição para Campeonatos de E-sports  
**Versão:** 1.0.0  
**Última atualização:** 2026-10-02  

> 🤖 **O `prd.md` responde _o quê_ o produto faz. Este responde _onde as coisas
> moram e como se chamam_.** Detalhe de tela — rota, componente, contrato —
> **não** se decide aqui: isso é trabalho da spec de cada história.

---

## 🤖 1. Fontes de Contexto para a IA

> Onde a IDE agêntica busca a verdade. **Isto é o índice; a configuração mora
> nos arquivos** — documento não configura ferramenta.

| Fonte | Onde configurar | Serve para |
| :---- | :-------------- | :--------- |
| Constituição da IA | `.agents/rules/utf-rules.md` (via `CLAUDE.md` / `AGENTS.md`) | Regras inegociáveis: fases do SDD, 2 rodadas, revisores distintos, Git |
| Fluxos da IA | `.agents/workflows/` | PRD, backlog, jornadas e tokens, architecture, setup, ciclo por Issue, ciclo por tarefa, tutor |
| Agentes (subagentes) | `.agents/agents/` (cascas em `.claude/`, `.cursor/` e `.opencode/`) | Implementador, revisores, auditor final e tutor |
| Ficha da disciplina | `docs/checklist.md` | Regras do projeto, IDs e entregas |
| Design (Protótipo Stitch) | [Protótipo no Stitch](https://stitch.withgoogle.com/preview/11000728704193360087?node-id=d2cf76c9d7d9414cba740122ff3cab60) | Cores, tipografia, hierarquia visual e estados de tela |

---

## 📦 2. Stack Tecnológica

> Definição **estrita**: nenhuma dependência entra sem aparecer aqui. Esta
> seção e os `package.json` contam a mesma história.

- **Backend:** NestJS (v11/12) + Prisma ORM (v8) + PostgreSQL (v16).
- **Frontend:** React (v19) com Vite + TypeScript.
- **Estilo:** TailwindCSS + Shadcn/UI (alinhado a `docs/design-tokens.md`).
- **Testes:**
  - Ferramenta unificada: **Vitest** (com `@testing-library/react` no frontend e `@nestjs/testing` no backend).
  - Comandos da raiz:
    - Suíte completa: `npm run test`
    - Verificação de lint: `npm run lint`
  - Comandos do backend (`apps/api`):
    - `npm run test` (testes unitários e integração)
    - `npm run test:e2e` (testes end-to-end)
    - `npm run lint`
  - Comandos do frontend (`apps/web`):
    - `npm run test` (testes de componentes e serviços)
    - `npm run lint`

### 🧱 2.1. Backend — regras estruturais (Critérios dos Revisores)

Declaradas para atender rigorosamente aos IDs da ficha da disciplina (`docs/checklist.md`):

1. **Separação de Camadas (ID6):**
   - **Controllers:** Apenas manipulam requisições HTTP, injetam DTOs validados e delegam chamadas aos Services. Sem lógica de negócio ou queries de banco no controller.
   - **Services:** Concentram regras de negócio, validações de domínio e orquestração do `PrismaService`.
   - **Modules:** Cada domínio é encapsulado em seu próprio módulo NestJS (`TournamentsModule`, `TeamsModule`, `RegistrationsModule`, `PaymentsModule`, `AuthModule`).

2. **Blindagem e Validação de Entradas (ID7):**
   - Todo payload de entrada possui um DTO explícito tipado com decorators de `class-validator` e `class-transformer`.
   - Configuração global no `main.ts`:
     ```typescript
     app.useGlobalPipes(new ValidationPipe({
       whitelist: true,
       forbidNonWhitelisted: true,
       transform: true,
     }));
     ```

3. **Operações Relacionais com Prisma ORM (ID8):**
   - Todo acesso ao banco ocorre via `PrismaService` tipado gerado a partir de `schema.prisma`.
   - Operações compostas (ex.: verificar vaga + criar inscrição, ou processar pagamento + aprovar vaga) usam transações atômicas `prisma.$transaction`.

4. **Autenticação JWT e Controle de Acesso (ID9):**
   - Autenticação stateless via token JWT assinado (`@nestjs/jwt`).
   - Estratégia Passport (`JwtStrategy`) e guard `JwtAuthGuard`.
   - Controle de permissões por perfil (`ORGANIZER`, `CAPTAIN`, `PLAYER`) via decorator `@Roles(...)` e `RolesGuard`.

5. **Padronização de Respostas e Tratamento de Erros (ID10):**
   - **Sucesso:** Interceptor global `TransformInterceptor` encapsula respostas no formato:
     ```json
     {
       "success": true,
       "data": {},
       "timestamp": "2026-10-02T20:00:00.000Z"
     }
     ```
   - **Erros:** Exception Filter global `HttpExceptionFilter` intercepta exceções e formata:
     ```json
     {
       "success": false,
       "statusCode": 400,
       "message": "Mensagem descritiva ou array de erros de validação",
       "error": "Bad Request",
       "timestamp": "2026-10-02T20:00:00.000Z",
       "path": "/api/tournaments"
     }
     ```

6. **Variáveis Sensíveis e Configuração (ID17):**
   - Injeção obrigatória via `@nestjs/config` (`ConfigModule`, `ConfigService`).
   - Credenciais e segredos (`DATABASE_URL`, `JWT_SECRET`, `MERCADO_PAGO_ACCESS_TOKEN`) ficam em arquivos `.env` ignorados no `.gitignore`.
   - Exemplo público fornecido em `.env.example`.

7. **Gateway de Pagamento e Webhooks (ID20 e ID21):**
   - Integração com **Mercado Pago Sandbox** (geração de cobrança PIX/Cartão vinculada à `Registration`).
   - Rota de webhook `/api/payments/webhook` com validação de token/assinatura da notificação.
   - Idempotência: processamento seguro que não duplica confirmações de pagamento já aprovadas.
   - Job/Cron com `@nestjs/schedule` para verificar expiração de inscrições pendentes com prazo esgotado (resolvendo a dúvida de `docs/user-flows.md`).

### 🌐 2.2. O contrato da API (ID14)

A documentação interativa OpenAPI/Swagger é gerada diretamente dos decorators do código (`@ApiTags`, `@ApiOperation`, `@ApiResponse`) e servida pela própria API:
- **Endpoint vivo:** `/docs` (ou `/api/docs`).
- **Princípio:** O contrato vive na API em execução. Não há tabela de endpoints manual neste documento e nenhum arquivo `swagger.json` é commitado.

---

## 🗂️ 3. Estrutura do Repositório (Monorepo)

Monorepo estruturado em pastas isoladas por aplicação, orquestrado via scripts no `package.json` raiz:

```text
.
├── .agents/                      # constituição, workflows e prompts dos agentes
├── .claude/ .cursor/ .opencode/  # cascas das IDEs apontando para .agents/
├── .github/                      # templates de PR, issues e CI/CD
│   └── workflows/
│       └── ci.yml                # Esteira de CI (ID18: lint e Vitest)
├── CLAUDE.md AGENTS.md           # carregamento da constituição
├── README.md                     # a vitrine do projeto
├── docker-compose.yml            # PostgreSQL 16 local para desenvolvimento
├── docs/                         # prd.md, user-flows.md, design-tokens.md, architecture.md, checklist.md
├── specs/                        # especificações e planos de cada issue
└── apps/
    ├── api/                      # Backend NestJS
    │   ├── src/
    │   │   ├── common/           # guards, interceptors, filters, decorators
    │   │   ├── config/           # ConfigModule e validação de envs
    │   │   ├── prisma/           # schema.prisma, migrations, PrismaService
    │   │   ├── modules/          # auth, tournaments, teams, registrations, payments
    │   │   ├── app.module.ts
    │   │   └── main.ts
    │   ├── test/                 # testes e2e e fixtures
    │   ├── package.json
    │   └── vitest.config.ts
    └── web/                      # Frontend React + Vite
        ├── src/
        │   ├── assets/
        │   ├── components/       # componentes reutilizáveis e UI (Shadcn)
        │   ├── pages/            # páginas roteadas (lazy loaded)
        │   ├── services/         # camada de repositório / clientes de API
        │   ├── hooks/            # custom hooks (ex.: useAuth, useTournaments)
        │   ├── types/            # interfaces e tipos derivados da API
        │   ├── App.tsx
        │   └── main.tsx
        ├── package.json
        ├── tailwind.config.js
        └── vitest.config.ts
```

---

## 🏗️ 4. Arquitetura Frontend

### 📏 Regras de Padrão e Dados (ID15 e ID16)

1. **Regra de Ouro da Camada de Dados:**
   - **Nenhum componente React realiza chamadas diretas ao backend (nem `fetch` nem `axios` direto no componente).**
   - Todas as requisições passam pela pasta `apps/web/src/services/` (ex.: `tournamentService.ts`, `registrationService.ts`, `authService.ts`).
   - Mudanças no contrato da API alteram exclusivamente os arquivos de serviço, mantendo as telas isoladas.

2. **Componentes e Roteamento:**
   - Componentes de função modernos com TypeScript e Hooks (proibido o uso de classes).
   - Separação rígida entre **páginas** (`src/pages/`) e **componentes reutilizáveis** (`src/components/`).
   - Roteamento modular com carregamento preguiçoso (`React.lazy` e `Suspense`) por rota.

3. **Gerenciamento de Estado:**
   - Estado de UI puramente local (`useState`, `useReducer`).
   - Estado de autenticação e sessão encapsulado em Context / Hook (`useAuth`).
   - Estado assíncrono do servidor gerenciado via camada de serviços com tratamento de loading, erro e cache.

---

## 🗄️ 5. Arquitetura de Dados

### 📖 5.1. Glossário Técnico (Mapeamento)

> A ponte entre o português do negócio (PRD §2) e o inglês do código.
> **Dados e código em inglês, interface em português.**

| Termo PRD (PT-BR) | Entidade técnica (EN) | Atributos principais |
| :---------------- | :-------------------- | :------------------- |
| **Usuário / Ator** | `User` | `id`, `name`, `email`, `passwordHash`, `role` (`ORGANIZER`, `CAPTAIN`, `PLAYER`), `createdAt` |
| **Campeonato** | `Tournament` | `id`, `title`, `game`, `entryFeeCents`, `maxSlots`, `status` (`OPEN`, `CLOSED`, `FINISHED`), `organizerId`, `deadlineDate`, `createdAt` |
| **Time (Equipe)** | `Team` | `id`, `name`, `tag`, `captainId`, `createdAt` |
| **Membro / Convite** | `TeamMember` | `id`, `teamId`, `userId`, `role` (`CAPTAIN`, `PLAYER`), `status` (`PENDING`, `ACCEPTED`, `REJECTED`), `joinedAt` |
| **Inscrição** | `Registration` | `id`, `tournamentId`, `teamId`, `status` (`PENDING_PAYMENT`, `CONFIRMED`, `WAITING_LIST`, `CANCELLED`), `createdAt` |
| **Pagamento / Ordem** | `Payment` | `id`, `registrationId`, `amountCents`, `status` (`PENDING`, `APPROVED`, `REJECTED`), `gatewayPaymentId`, `qrCode`, `paidAt`, `createdAt` |
| **Log de Webhook** | `WebhookEvent` | `id`, `provider`, `externalEventId`, `payloadJson`, `processedAt` |

### 📊 5.2. Diagrama ER (Mermaid)

> Atende ao escopo mínimo da ficha (múltiplas relações 1:N e fluxo completo de pedido e pagamento).

```mermaid
erDiagram
    User ||--o{ Tournament : "cria e gerencia (organizer)"
    User ||--o{ Team : "lidera como capitão"
    User ||--o{ TeamMember : "compõe como jogador"
    Team ||--o{ TeamMember : "possui no elenco"
    Tournament ||--o{ Registration : "recebe vagas de"
    Team ||--o{ Registration : "inscreve-se via"
    Registration ||--o{ Payment : "gera cobrança"

    User {
        string id PK
        string email UK
        string passwordHash
        string name
        enum role "ORGANIZER | CAPTAIN | PLAYER"
        datetime createdAt
        datetime updatedAt
    }

    Tournament {
        string id PK
        string organizerId FK
        string title
        string game
        int entryFeeCents
        int maxSlots
        enum status "OPEN | CLOSED | FINISHED"
        datetime deadlineDate
        datetime createdAt
        datetime updatedAt
    }

    Team {
        string id PK
        string captainId FK
        string name UK
        string tag UK
        datetime createdAt
        datetime updatedAt
    }

    TeamMember {
        string id PK
        string teamId FK
        string userId FK
        enum role "CAPTAIN | PLAYER"
        enum status "PENDING | ACCEPTED | REJECTED"
        datetime createdAt
        datetime joinedAt
    }

    Registration {
        string id PK
        string tournamentId FK
        string teamId FK
        enum status "PENDING_PAYMENT | CONFIRMED | WAITING_LIST | CANCELLED"
        datetime createdAt
        datetime updatedAt
    }

    Payment {
        string id PK
        string registrationId FK
        int amountCents
        enum status "PENDING | APPROVED | REJECTED"
        string gatewayProvider "MERCADO_PAGO"
        string gatewayPaymentId
        string qrCode
        datetime paidAt
        datetime createdAt
        datetime updatedAt
    }

    WebhookEvent {
        string id PK
        string provider
        string externalEventId UK
        json payloadJson
        datetime processedAt
    }
```

### 🌍 5.3. O banco por ambiente

| Ambiente | Onde roda | Como conecta |
| :--- | :--- | :--- |
| **Local** | Docker local (`postgres:16-alpine` via `docker-compose.yml`) | `DATABASE_URL="postgresql://postgres:postgres@localhost:5432/insc_db"` |
| **CI** | GitHub Actions Service Container (`postgres:16-alpine`) | `DATABASE_URL="postgresql://postgres:postgres@localhost:5432/insc_test_db"` |
| **Produção (ID19)** | Nuvem **Neon.tech** (PostgreSQL Serverless) | `DATABASE_URL="postgresql://...neon.tech/neondb?sslmode=require&pgbouncer=true"` |

> 🔒 Credenciais **nunca** aparecem no repositório — nem em código, nem em
> YAML, nem em doc. São lidas exclusivamente das variáveis de ambiente (`.env` local ou secrets na nuvem).

---

## 🗺️ 6. Mapa de Domínios

> Este índice cresce a cada história implementada. Ele mapeia onde mora cada
> domínio, quem o protege e qual história o originou.

| Domínio | Módulo (`apps/api/src/modules/`) | Guard | Dados (Prisma) | US Relacionada |
| :------ | :------------------------------- | :---- | :------------- | :------------- |
| **Auth** | `auth/` | `JwtAuthGuard` | `User` | Base |
| **Tournaments** | `tournaments/` | `JwtAuthGuard`, `RolesGuard('ORGANIZER')` | `Tournament` | US01, US09 |
| **Teams** | `teams/` | `JwtAuthGuard`, `RolesGuard('CAPTAIN')` | `Team`, `TeamMember` | US02, US06, US07 |
| **Registrations** | `registrations/` | `JwtAuthGuard`, `RolesGuard('CAPTAIN')` | `Registration` | US03, US05, US08 |
| **Payments** | `payments/` | `JwtAuthGuard` (checkout), Público com validação de assinatura (webhook) | `Payment`, `WebhookEvent` | US04, US05 |

---

## 📅 7. Histórico

| Data | Versão | O que mudou |
| :--- | :----- | :---------- |
| 2026-10-02 | 1.0.0 | Versão inicial gerada via `/utf-architecture` cobrindo todas as decisões de stack, padrões para IDs da disciplina e modelo ER. |

---

## 🛑 O que ainda **não** está neste documento

Detalhes de funcionalidade — DTOs de endpoints específicos, validações pontuais de formulário, contratos exatos de cada tela — **não entram aqui**: nascem sob demanda no `spec.md` de cada história. Este documento guarda só as regras e padrões arquiteturais que valem para o sistema inteiro.
