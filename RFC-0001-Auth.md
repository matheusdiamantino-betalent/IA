# 🔐 Arquitetura de Autenticação Delegada
## Reaproveitamento do Auth da API Principal (AdonisJS) na API de IA de Questions (NestJS)
### Recorte Arquitetural da Fase 1 — Auth, Identidade e Autorização Delegada

---

<div align="center">

![Status](https://img.shields.io/badge/status-fase%201%20em%20implementa%C3%A7%C3%A3o-22c55e?style=for-the-badge)
![Arquitetura](https://img.shields.io/badge/arquitetura-autentica%C3%A7%C3%A3o%20delegada-3b82f6?style=for-the-badge)
![Auth](https://img.shields.io/badge/auth-oat%20%2B%20redis-f59e0b?style=for-the-badge)
![Segurança](https://img.shields.io/badge/seguran%C3%A7a-enterprise-ef4444?style=for-the-badge)
![Integração](https://img.shields.io/badge/integra%C3%A7%C3%A3o-introspec%C3%A7%C3%A3o-a855f7?style=for-the-badge)
![Stack](https://img.shields.io/badge/stack-adonisjs%20%2B%20nestjs-06b6d4?style=for-the-badge)
![Scope](https://img.shields.io/badge/escopo-fase%201-0f172a?style=for-the-badge)

</div>

---

> [!IMPORTANT]
> Este documento representa **o recorte arquitetural do módulo de autenticação e autorização delegada da Fase 1**.
>
> Ele **não descreve toda a Fase 1 da plataforma**, mas sim a fundação de segurança e acesso que habilita a evolução segura do domínio de Questions.

> [!NOTE]
> O documento foi estruturado para funcionar como **artefato final de entrega técnica**, adequado para:
>
> - revisão arquitetural;
> - PR técnica;
> - alinhamento entre squads backend;
> - implementação incremental;
> - base para hardening de segurança.

---

# Sumário

- [1. Resumo Executivo](#1-resumo-executivo)
- [2. Escopo do Documento](#2-escopo-do-documento)
- [3. Problema Arquitetural](#3-problema-arquitetural)
- [4. Decisão de Arquitetura](#4-decisão-de-arquitetura)
- [5. Contexto Técnico Validado](#5-contexto-técnico-validado)
- [6. Objetivos do Slice](#6-objetivos-do-slice)
- [7. Princípios Arquiteturais](#7-princípios-arquiteturais)
- [8. Arquitetura de Alto Nível](#8-arquitetura-de-alto-nível)
- [9. Fluxos Principais](#9-fluxos-principais)
- [10. Boundary e Responsabilidades](#10-boundary-e-responsabilidades)
- [11. Contrato de Integração](#11-contrato-de-integração)
- [12. Endpoint de Introspecção](#12-endpoint-de-introspecção)
- [13. Arquitetura Interna do Auth Module](#13-arquitetura-interna-do-auth-module)
- [14. Estratégia de Roles, Scopes e Enforcement](#14-estratégia-de-roles-scopes-e-enforcement)
- [15. Estrutura Técnica Recomendada](#15-estrutura-técnica-recomendada)
- [16. Segurança, Resiliência e Observabilidade](#16-segurança-resiliência-e-observabilidade)
- [17. ADR — Registro da Decisão](#17-adr--registro-da-decisão)
- [18. Plano de Implementação](#18-plano-de-implementação)
- [19. Critérios de Aceite Técnico](#19-critérios-de-aceite-técnico)
- [20. Próximos Passos](#20-próximos-passos)
- [21. Conclusão Executiva](#21-conclusão-executiva)

---

# 1. Resumo Executivo

A arquitetura proposta estabelece um modelo de **autenticação delegada com introspecção controlada**, no qual a **API Principal (AdonisJS)** permanece como **fonte única de verdade de identidade administrativa**, enquanto a **API de IA de Questions (NestJS)** consome esse contexto autenticado para aplicar **autorização local por domínio**.

## Decisão central

```text
A API Principal autentica.
A API de IA consome identidade autenticada.
A API de IA autoriza localmente.
```

## Resultado esperado

- identidade única;
- sessão lógica única;
- sem duplicação de login ou token;
- autorização desacoplada por domínio;
- integração segura, auditável e evolutiva.

> [!TIP]
> Esta abordagem reduz acoplamento com o legado de autenticação e preserva independência evolutiva do domínio de Questions.

---

# 2. Escopo do Documento

## Em escopo

- reaproveitamento do auth administrativo existente;
- introspecção do token administrativo;
- resolução do contexto autenticado do `admin`;
- normalização do payload autenticado;
- mapeamento de `roles` externas para `scopes` internos;
- enforcement de autorização na API de IA;
- requisitos de segurança, resiliência e observabilidade;
- desenho técnico do `AuthModule` da IA.

## Fora de escopo

- login de frontend;
- UX ou UI de autenticação;
- pipeline completo de IA;
- OCR, embeddings e LLM orchestration;
- geração ponta a ponta de questões;
- redesign do auth legado;
- JWT migration;
- SSO externo ou federação multi-tenant.

---

# 3. Problema Arquitetural

Se a API de IA implementar autenticação própria para resolver esse acesso, o sistema passa a carregar complexidade, redundância e risco desnecessários.

## Riscos de um auth duplicado

- duplicação de identidade;
- divergência de sessão entre sistemas;
- revogação distribuída;
- permissão inconsistente;
- maior superfície de ataque;
- acoplamento indevido entre domínio de IA e domínio de identidade;
- troubleshooting mais difícil em produção.

## O problema real

A API de IA **não precisa autenticar o usuário do zero**.

Ela precisa responder, com segurança:

- quem é o usuário autenticado;
- se o token ainda é válido;
- quais roles esse usuário possui;
- o que esse usuário pode executar dentro do domínio de Questions.

> [!IMPORTANT]
> O problema não é “como fazer login na IA”.
>
> O problema correto é: **como confiar, de forma controlada, na identidade já autenticada pela plataforma principal**.

---

# 4. Decisão de Arquitetura

## Decisão oficial

# **Autenticação Delegada com Introspecção Controlada**

## Papel de cada sistema

| Sistema | Papel arquitetural |
|---|---|
| **API Principal (AdonisJS)** | Autoridade de autenticação, validação de token e resolução de identidade |
| **API de IA (NestJS)** | Consumidora de contexto autenticado e executora de autorização local |

## Regra de ouro

> [!IMPORTANT]
> **A API de IA não autentica usuários.**
>
> Ela **confia de forma controlada** na identidade resolvida pela API principal.

## Decisão operacional

```text
Auth centralizado.
Autorização distribuída.
Boundary preservado.
```

---

# 5. Contexto Técnico Validado

## Stack atual

- **Framework principal:** AdonisJS
- **Framework da API de IA:** NestJS
- **Guard administrativo:** `admin`
- **Driver:** `oat` (Opaque Access Token)
- **Persistência do token:** Redis
- **Provider de identidade:** `Admin`
- **Autorização legada:** baseada em `roles`
- **Proteção atual de rotas:** `auth:admin` + `role:*`

## Guard administrativo validado

```ts
admin: {
  driver: 'oat',
  tokenProvider: {
    type: 'api',
    driver: 'redis',
    redisConnection: 'local',
    foreignKey: 'admin_id',
  },
  provider: {
    driver: 'lucid',
    identifierKey: 'id',
    uids: ['email'],
    model: () => import('App/Models/Admin'),
  },
}
```

## Conclusão prática

A identidade correta a ser reaproveitada pela API de IA é a identidade do **perímetro administrativo**.

```text
auth:admin
```

---

# 6. Objetivos do Slice

Este slice existe para permitir que a API de IA aceite requests administrativos **sem implementar login próprio, sessão própria ou emissão de token própria**.

## Objetivos técnicos

1. receber o mesmo Bearer Token emitido pela app principal;
2. validar esse contexto contra a autoridade correta;
3. resolver o perfil autenticado com suas roles;
4. traduzir essas roles em scopes internos;
5. autorizar a operação localmente.

## Resultado arquitetural desejado

```text
Mesma identidade.
Mesma sessão lógica.
Sem duplicação de auth.
Com autorização isolada por domínio.
```

---

# 7. Princípios Arquiteturais

## Princípios norteadores

### 7.1 Single Source of Truth
A identidade administrativa deve existir em apenas um lugar confiável.

### 7.2 Security by Default
Qualquer incerteza de autenticação deve resultar em bloqueio, nunca em permissão.

### 7.3 Boundary First
A API de IA não deve conhecer detalhes internos do mecanismo de autenticação do Adonis.

### 7.4 Stable Internal Contract
O domínio da IA deve operar sobre um contrato autenticado interno estável, mesmo que o provider externo evolua.

### 7.5 Local Authorization
A semântica de acesso do domínio de Questions deve ser controlada localmente por scopes internos.

> [!NOTE]
> Estes princípios evitam que o auth vire um “vazamento estrutural” do core principal para dentro da IA.

---

# 8. Arquitetura de Alto Nível

## 8.1 Diretriz visual para diagramas

> [!TIP]
> Os diagramas abaixo foram organizados em **alto nível**, com foco em leitura executiva, clareza de ownership e comunicação arquitetural.
>
> Recomenda-se manter este padrão visual em PRs, docs técnicas e RFCs futuras:
>
> - fundo escuro;
> - contraste alto;
> - textos curtos;
> - ícones semânticos;
> - espaçamento respirado;
> - sem excesso de linhas cruzadas.

## 8.2 Visão executiva

```mermaid
%%{init: {'theme':'dark','themeVariables': {
  'primaryColor':'#111827',
  'primaryTextColor':'#e5e7eb',
  'primaryBorderColor':'#6366f1',
  'lineColor':'#94a3b8',
  'secondaryColor':'#0f172a',
  'tertiaryColor':'#1f2937',
  'fontFamily':'Inter, Segoe UI, Arial'
}}}%%
flowchart LR
    FE["🖥️ Frontend\nAdministrativo"]
    AP["🏛️ API\nPrincipal"]
    RD[("🔴 Redis")]
    IA["🤖 API de IA\nQuestions"]

    FE -->|"🔐 Login\ne sessão"| AP
    AP -->|"🗝️ Token OAT"| RD
    FE -->|"📨 Bearer\nToken"| IA
    IA -->|"🔎 Introspecção"| AP
    AP -->|"👤 Admin autenticado\ne roles"| IA
```

## 8.3 Diagrama de contexto

```mermaid
%%{init: {'theme':'dark','themeVariables': {
  'primaryColor':'#111827',
  'primaryTextColor':'#e5e7eb',
  'primaryBorderColor':'#8b5cf6',
  'lineColor':'#94a3b8',
  'secondaryColor':'#0f172a',
  'tertiaryColor':'#1f2937',
  'fontFamily':'Inter, Segoe UI, Arial'
}}}%%
flowchart TB
    subgraph CLIENTE["🧑‍💼 Cliente"]
        FE["Frontend\nAdmin"]
    end

    subgraph CORE["🏛️ Core Principal"]
        AP["API\nPrincipal"]
        AUTH["Auth\nAdmin"]
        ROLE["Roles\nAdmin"]
        REDIS[("Redis")]
    end

    subgraph IA_DOMAIN["🧠 Domínio IA Questions"]
        IAA["API\nIA"]
        GUARD["Auth\nGuard"]
        MAP["Normalizer\n+ Mapper"]
        SCOPE["Scopes\nGuard"]
        USECASE["Use\nCases"]
    end

    FE --> AP
    FE --> IAA
    AP --> AUTH
    AUTH --> REDIS
    AP --> ROLE
    IAA --> GUARD
    GUARD --> AP
    AP --> MAP
    MAP --> SCOPE
    SCOPE --> USECASE
```

## 8.4 Diagrama de ownership

```mermaid
%%{init: {'theme':'dark','themeVariables': {
  'primaryColor':'#111827',
  'primaryTextColor':'#e5e7eb',
  'primaryBorderColor':'#22c55e',
  'lineColor':'#94a3b8',
  'secondaryColor':'#0f172a',
  'tertiaryColor':'#1f2937',
  'fontFamily':'Inter, Segoe UI, Arial'
}}}%%
flowchart LR
    subgraph MAIN["🏛️ API Principal"]
        M1["Login"]
        M2["Token\nOAT"]
        M3["Validação"]
        M4["Identidade\nAdmin"]
        M5["Roles"]
    end

    subgraph AI["🤖 API de IA"]
        A1["Recepção\ndo Token"]
        A2["Introspecção"]
        A3["Normalização"]
        A4["Scopes"]
        A5["Autorização"]
        A6["Use\nCases"]
    end

    M1 --> M2 --> M3 --> M4 --> M5
    M4 --> A2
    M5 --> A2
    A1 --> A2 --> A3 --> A4 --> A5 --> A6
```

---

# 9. Fluxos Principais

## 9.1 Fluxo funcional ponta a ponta

```mermaid
%%{init: {'theme':'dark','themeVariables': {
  'primaryColor':'#111827',
  'primaryTextColor':'#e5e7eb',
  'primaryBorderColor':'#f59e0b',
  'lineColor':'#94a3b8',
  'secondaryColor':'#0f172a',
  'tertiaryColor':'#1f2937',
  'fontFamily':'Inter, Segoe UI, Arial'
}}}%%
flowchart TD
    A["🧑‍💼 Admin\nfaz login"] --> B["POST\n/api/v1/login"]
    B --> C["auth.use\nadmin.attempt"]
    C --> D["🗝️ Token OAT\nemitido"]
    D --> E["Frontend\nrecebe token"]

    E --> F["Frontend chama\nAPI IA"]
    F --> G["AuthGuard\nextrai token"]
    G --> H["AuthService"]
    H --> I["GET /profile\nou /auth/me"]
    I --> J["auth\nadmin"]
    J --> K["Redis valida\ntoken"]

    K --> L{"Token válido"}
    L -- "Não" --> M["401\nUnauthorized"]
    L -- "Sim" --> N["Admin\nautenticado"]
    N --> O["Roles\nresolvidas"]
    O --> P["Normalizer"]
    P --> Q["Mapper"]
    Q --> R["Scopes\nGuard"]
    R --> S{"Tem permissão"}
    S -- "Não" --> T["403\nForbidden"]
    S -- "Sim" --> U["✅ Executa\ncaso de uso"]
```

## 9.2 Sequência técnica de request

```mermaid
%%{init: {'theme':'dark','themeVariables': {
  'primaryColor':'#111827',
  'primaryTextColor':'#e5e7eb',
  'primaryBorderColor':'#3b82f6',
  'lineColor':'#94a3b8',
  'secondaryColor':'#0f172a',
  'tertiaryColor':'#1f2937',
  'fontFamily':'Inter, Segoe UI, Arial'
}}}%%
sequenceDiagram
    participant F as Frontend
    participant I as API IA
    participant P as API Principal
    participant R as Redis

    F->>I: Request com Bearer Token
    I->>I: Extrai token
    I->>P: GET /api/v1/profile
    Note over I,P: Evolução recomendada\nGET /api/v1/auth/me
    P->>P: auth admin
    P->>R: check token
    R-->>P: Token válido ou inválido

    alt token inválido
        P-->>I: 401
        I-->>F: 401
    else token válido
        P-->>I: Admin autenticado e roles
        I->>I: Normalizer
        I->>I: Mapper
        I->>I: ScopesGuard
        alt sem permissão
            I-->>F: 403
        else autorizado
            I-->>F: 200
        end
    end
```

## 9.3 Fluxo de decisão de autorização

```mermaid
%%{init: {'theme':'dark','themeVariables': {
  'primaryColor':'#111827',
  'primaryTextColor':'#e5e7eb',
  'primaryBorderColor':'#a855f7',
  'lineColor':'#94a3b8',
  'secondaryColor':'#0f172a',
  'tertiaryColor':'#1f2937',
  'fontFamily':'Inter, Segoe UI, Arial'
}}}%%
flowchart LR
    A["Token\nválido"] --> B["Admin\nautenticado"]
    B --> C["Roles\nexternas"]
    C --> D["Normalizer"]
    D --> E["Mapper"]
    E --> F["Scopes\ninternos"]
    F --> G["Scopes\nGuard"]
    G --> H{"Permissão suficiente"}
    H -- "Sim" --> I["Libera\nexecução"]
    H -- "Não" --> J["Bloqueia\nacesso"]
```

## Nota de decisão

> [!IMPORTANT]
> A separação entre **autenticação** e **autorização** é proposital e obrigatória:
>
> - autenticação responde **quem é**;
> - autorização responde **o que pode fazer**.

---

# 10. Boundary e Responsabilidades

## 10.1 O que cruza a fronteira entre sistemas

- token Bearer recebido na request;
- chamada de introspecção;
- payload autenticado do admin;
- roles administrativas necessárias;
- status mínimo de conta quando aplicável.

## 10.2 O que não deve cruzar a fronteira

- acesso direto ao Redis;
- detalhes internos do provider do Adonis;
- segredos internos do auth principal;
- middleware legado reaproveitado de forma acoplada;
- payload cru espalhado pela IA.

## 10.3 Boundary model

```mermaid
%%{init: {'theme':'dark','themeVariables': {
  'primaryColor':'#111827',
  'primaryTextColor':'#e5e7eb',
  'primaryBorderColor':'#06b6d4',
  'lineColor':'#94a3b8',
  'secondaryColor':'#0f172a',
  'tertiaryColor':'#1f2937',
  'fontFamily':'Inter, Segoe UI, Arial'
}}}%%
flowchart LR
    subgraph MAIN["🏛️ API Principal"]
        A1["Auth\nAdmin"]
        A2["Roles"]
        A3["Token\nValidation"]
        A4[("Redis")]
    end

    subgraph AI["🤖 API IA"]
        B1["Auth\nGuard"]
        B2["External\nGateway"]
        B3["Normalizer"]
        B4["Mapper"]
        B5["Scopes\nGuard"]
        B6["Use\nCases"]
    end

    A1 --> A3
    A3 --> A4
    A2 --> A1

    B1 --> B2
    B2 --> A1
    B2 --> B3
    B3 --> B4
    B4 --> B5
    B5 --> B6
```

## 10.4 Matriz de ownership

| Tema | API Principal | API de IA |
|---|---|---|
| Login | ✅ | ❌ |
| Emissão de token | ✅ | ❌ |
| Revogação | ✅ | ❌ |
| Introspecção | ✅ | Consome |
| Normalização de payload | ❌ | ✅ |
| Mapeamento para scopes | ❌ | ✅ |
| Autorização de domínio | ❌ | ✅ |
| Enforcement por endpoint | ❌ | ✅ |

---

# 11. Contrato de Integração

## 11.1 Estado atual utilizável

Hoje, com base no comportamento atual do `AuthController.show`, o contrato efetivamente disponível é equivalente a:

```ts
const user = auth.user as Admin

return response.ok(
  await Admin.query().preload('roles').where('id', user.id).first()
)
```

## 11.2 Exemplo de payload atual

```json
{
  "id": 10,
  "name": "Matheus Diamantino",
  "email": "admin@empresa.com",
  "roles": [
    {
      "id": 1,
      "name": "admin",
      "slug": "admin"
    },
    {
      "id": 3,
      "name": "questioncreator",
      "slug": "questioncreator"
    }
  ],
  "created_at": "2026-01-10T10:00:00.000Z",
  "updated_at": "2026-02-10T10:00:00.000Z"
}
```

## 11.3 Payload recomendado para estabilização futura

```json
{
  "id": 10,
  "name": "Matheus Diamantino",
  "email": "admin@empresa.com",
  "roles": ["admin", "questioncreator"],
  "active": true,
  "status": "active"
}
```

## 11.4 Contrato interno canônico da IA

```ts
export interface AuthenticatedUser {
  id: number
  name: string
  email: string
  roles: string[]
  scopes: string[]
  isActive: boolean
  status?: string
}
```

## 11.5 Regra de robustez

A IA pode ser tolerante a pequenas variações do payload externo, mas essa tolerância deve existir **somente na camada de normalização**.

O domínio interno deve trabalhar sempre com um contrato estável.

> [!TIP]
> Regra prática: **tolerância fora, rigidez dentro**.

---

# 12. Endpoint de Introspecção

## Estado atual utilizável

```http
GET /api/v1/profile
```

## Evolução recomendada

```ts
Route.get('/auth/me', 'AuthController.me').middleware(['auth:admin'])
```

## Controller recomendado

```ts
public async me({ response, auth }: HttpContextContract) {
  const user = auth.user as Admin

  const admin = await Admin.query()
    .preload('roles')
    .where('id', user.id)
    .first()

  return response.ok(admin)
}
```

## Requisitos do endpoint

- payload estável e canônico;
- sem dependência de UI;
- sem lógica incidental de tela;
- protegido apenas por `auth:admin`;
- contrato previsível para integração entre serviços.

## Recomendação enterprise adicional

### Headers internos sugeridos entre serviços

```http
X-Internal-Client: questions-ai-api
X-Correlation-Id: <uuid>
X-Request-Id: <uuid>
```

### Controles recomendados

- allowlist de origem interna por ambiente;
- rate limit técnico para introspecção;
- logs estruturados por request;
- versionamento de contrato quando necessário.

---

# 13. Arquitetura Interna do Auth Module

## 13.1 Princípio de implementação

O `AuthModule` da IA deve ser responsável apenas por:

- receber o token;
- validar esse token contra a API principal;
- construir um `AuthenticatedUser` interno;
- aplicar autorização por scopes.

Ele **não deve**:

- emitir token;
- persistir sessão administrativa;
- manter login próprio;
- reimplementar o guard do Adonis;
- acoplar a IA ao payload cru da API principal.

## 13.2 Diagrama interno do módulo

```mermaid
%%{init: {'theme':'dark','themeVariables': {
  'primaryColor':'#111827',
  'primaryTextColor':'#e5e7eb',
  'primaryBorderColor':'#ef4444',
  'lineColor':'#94a3b8',
  'secondaryColor':'#0f172a',
  'tertiaryColor':'#1f2937',
  'fontFamily':'Inter, Segoe UI, Arial'
}}}%%
flowchart TD
    A["📨 Request\nHTTP"] --> B["🛡️ Auth\nGuard"]
    B --> C["Auth\nService"]
    C --> D["External Auth\nGateway"]
    D --> E["Auth API\nClient"]
    E --> F["🏛️ API\nPrincipal"]
    F --> G["Payload\nexterno"]
    G --> H["External Auth Response\nNormalizer"]
    H --> I["Authenticated User\nMapper"]
    I --> J["Authenticated\nUser"]
    J --> K["Scopes\nGuard"]
    K --> L["Controller\nUse Case"]
```

## 13.3 Composição interna

```mermaid
%%{init: {'theme':'dark','themeVariables': {
  'primaryColor':'#111827',
  'primaryTextColor':'#e5e7eb',
  'primaryBorderColor':'#22c55e',
  'lineColor':'#94a3b8',
  'secondaryColor':'#0f172a',
  'tertiaryColor':'#1f2937',
  'fontFamily':'Inter, Segoe UI, Arial'
}}}%%
flowchart LR
    subgraph INFRA["Infra"]
        C1["Auth API\nClient"]
        C2["External Auth\nGateway"]
    end

    subgraph APP["Application"]
        A1["Auth\nService"]
        A2["Auth\nGuard"]
        A3["Scopes\nGuard"]
    end

    subgraph DOMAIN["Domain Contract"]
        D1["Authenticated\nUser"]
        D2["Internal\nScope"]
        D3["Role Scope\nMap"]
    end

    subgraph SUPPORT["Support"]
        S1["Normalizer"]
        S2["Mapper"]
        S3["Decorators"]
        S4["Helpers"]
    end

    A2 --> A1
    A1 --> C2
    C2 --> C1
    C1 --> S1
    S1 --> S2
    S2 --> D1
    D1 --> A3
    D2 --> A3
    D3 --> A3
    S3 --> A2
    S4 --> S1
```

## 13.4 Sequência interna de responsabilidades

| Camada | Responsabilidade |
|---|---|
| **Client** | comunicação HTTP com provider externo |
| **Gateway** | boundary técnico com a API principal |
| **Service** | orquestração do processo de autenticação delegada |
| **Normalizer** | tolerância a payloads externos variáveis |
| **Mapper** | construção do contrato canônico interno |
| **Guards** | enforcement técnico de acesso |

---

# 14. Estratégia de Roles, Scopes e Enforcement

## Regra central

```text
Role externa → Scope interno → Decisão de autorização
```

## Mapeamento inicial recomendado

```ts
export const ROLE_SCOPE_MAP: Record<string, string[]> = {
  admin: ['*'],
  contentcreator: [
    'content.read',
    'content.write',
    'documents.read'
  ],
  questioncreator: [
    'documents.read',
    'documents.upload',
    'processing.read',
    'processing.retry',
    'questions.generate',
    'questions.review'
  ],
  seller: [
    'dashboard.read'
  ],
}
```

## Estratégia de enforcement

A autorização deve ocorrer em camadas complementares:

- **AuthGuard** → valida identidade;
- **ScopesGuard** → valida permissão técnica do endpoint;
- **Application Layer / Use Case** → valida regra de negócio.

## Exemplo de uso no NestJS

```ts
@UseGuards(AuthGuard, ScopesGuard)
@RequiredScopes('questions.generate')
@Post('/questions/generate')
async generate(@CurrentUser() user: AuthenticatedUser) {
  return this.generateQuestionsUseCase.execute({
    actorId: user.id,
  })
}
```

## Nota arquitetural

> [!IMPORTANT]
> Essa separação evita que a semântica de acesso da IA fique refém da modelagem de roles do legado.

---

# 15. Estrutura Técnica Recomendada

## Tree View do módulo

```text
src/
└── modules/
    └── auth/
        ├── 📦 auth.module.ts
        ├── infra/
        │   ├── 🌐 clients/
        │   │   └── auth-api.client.ts
        │   ├── 🔌 gateways/
        │   │   └── external-auth.gateway.ts
        │   ├── ⚙️ services/
        │   │   └── auth.service.ts
        │   ├── 🛡️ guards/
        │   │   ├── auth.guard.ts
        │   │   └── scopes.guard.ts
        │   └── 🧩 decorators/
        │       ├── current-user.decorator.ts
        │       └── required-scopes.decorator.ts
        ├── model/
        │   ├── 🧾 dto/
        │   │   └── authenticated-user.dto.ts
        │   ├── 📘 interfaces/
        │   │   ├── external-admin-profile.interface.ts
        │   │   ├── authenticated-user.interface.ts
        │   │   └── role-scope-map.interface.ts
        │   ├── 🏷️ enums/
        │   │   └── internal-scope.enum.ts
        │   └── 🧠 constants/
        │       └── role-scope-map.constant.ts
        └── lib/
            ├── 🔄 mappers/
            │   └── authenticated-user.mapper.ts
            ├── 🧰 helpers/
            │   ├── extract-bearer-token.helper.ts
            │   └── normalize-role.helper.ts
            └── 🧼 normalizers/
                └── external-auth-response.normalizer.ts
```

## Legenda visual

- **🌐 Clients** → integração HTTP externa
- **🔌 Gateways** → boundary com provider externo
- **⚙️ Services** → orquestração de aplicação
- **🛡️ Guards** → enforcement técnico
- **🧾 DTO / Interfaces / Enums / Constants** → contratos e semântica interna
- **🔄 / 🧰 / 🧼** → transformação, suporte e estabilização do payload

---

# 16. Segurança, Resiliência e Observabilidade

## 16.1 Segurança por padrão

### Obrigatório

- TLS obrigatório;
- aceitar apenas `Authorization: Bearer`;
- nunca trafegar token em query string;
- negar acesso por padrão;
- falha de introspecção deve bloquear;
- nunca persistir token puro;
- nunca acessar Redis diretamente da IA.

## 16.2 Resiliência

### Regras recomendadas

- timeout entre **1000ms e 2000ms**;
- retry apenas para falhas transitórias;
- no máximo **1 retry curto**;
- nunca retry para `401`, `403` e `404`;
- preferir falha rápida a degradação silenciosa.

### Regra crítica

> [!IMPORTANT]
> **Falha de autenticação remota deve degradar para bloqueio, nunca para permissão.**

## 16.3 Observabilidade

### Logs mínimos

- `request_id`
- `correlation_id`
- `user_id`
- `user_roles`
- `auth_provider_status_code`
- `auth_provider_latency_ms`
- `endpoint`
- `method`
- `decision`

### Métricas recomendadas

- `auth_requests_total`
- `auth_success_total`
- `auth_failures_total`
- `auth_forbidden_total`
- `auth_provider_timeout_total`
- `auth_provider_latency_ms`
- `auth_guard_execution_ms`

## 16.4 Matriz de falha esperada

| Situação | Resultado esperado |
|---|---|
| Token ausente | `401 Unauthorized` |
| Token inválido | `401 Unauthorized` |
| Token revogado | `401 Unauthorized` |
| Usuário sem scope | `403 Forbidden` |
| Timeout da API principal | `503` ou bloqueio controlado |
| Payload inválido do provider | `401` ou `502`, conforme política adotada |

## 16.5 Hardening adicional recomendado

### Segurança de integração interna

- autenticação mTLS entre serviços em ambientes críticos;
- allowlist por rede privada ou service mesh;
- assinatura opcional de requests internos para rotas sensíveis;
- versionamento explícito do endpoint de introspecção.

### Proteção operacional

- circuit breaker na camada cliente;
- budget de timeout controlado;
- dashboards dedicados de auth delegada;
- alarmes por aumento de `401`, `403` e timeout.

---

# 17. ADR — Registro da Decisão

## ADR-001 — Modelo de autenticação entre API Principal e API de IA

### Status
**Aceito**

### Contexto
A API de IA precisa receber requests autenticadas do mesmo perímetro administrativo da plataforma principal, sem duplicar login, token ou estado de sessão.

### Decisão
Adotar **Autenticação Delegada com Introspecção Controlada**, usando a API Principal como autoridade de autenticação e a API de IA como consumidora de identidade autenticada com autorização local baseada em scopes.

### Consequências positivas

- elimina duplicação de auth;
- reduz risco operacional;
- preserva boundary arquitetural;
- facilita troubleshooting;
- melhora governança de identidade.

### Trade-offs assumidos

- dependência controlada da disponibilidade da API Principal;
- necessidade de normalização de payload externo;
- necessidade de observabilidade forte na integração.

---

# 18. Plano de Implementação

## API Principal

- [ ] manter `auth:admin` como fonte de verdade
- [ ] expor endpoint estável de introspecção
- [ ] garantir preload consistente de `roles`
- [ ] padronizar payload retornado
- [ ] validar `401` para token inválido
- [ ] estabilizar contrato de integração

## API de IA

- [ ] criar `AuthModule`
- [ ] implementar `AuthGuard`
- [ ] implementar `ScopesGuard`
- [ ] implementar `AuthService`
- [ ] implementar `ExternalAuthGateway`
- [ ] implementar `AuthApiClient`
- [ ] implementar `AuthenticatedUserMapper`
- [ ] implementar normalizer do payload externo
- [ ] implementar `ROLE_SCOPE_MAP`
- [ ] proteger endpoints críticos
- [ ] instrumentar logs e métricas

## Testes obrigatórios

- [ ] token ausente
- [ ] token inválido
- [ ] token expirado ou revogado
- [ ] token válido
- [ ] autorização por scope
- [ ] falha da API principal
- [ ] teste ponta a ponta entre APIs
- [ ] payload externo inconsistente

## Ordem de execução recomendada

1. estabilizar contrato de introspecção na API principal;
2. construir contrato interno canônico na IA;
3. implementar guard de autenticação;
4. implementar guard de scopes;
5. proteger endpoints críticos;
6. instrumentar observabilidade;
7. validar com testes integrados.

---

# 19. Critérios de Aceite Técnico

## Critérios funcionais

- a API de IA aceita Bearer Token emitido pela API principal;
- a API de IA rejeita token inválido ou revogado;
- a API de IA constrói corretamente o `AuthenticatedUser` interno;
- a autorização por scope protege endpoints críticos;
- requests autorizados executam normalmente.

## Critérios não funcionais

- logs e métricas suficientes para troubleshooting;
- comportamento previsível em timeout ou falha remota;
- ausência de dependência direta com Redis;
- ausência de emissão de token na IA;
- boundary preservado entre identidade e domínio de Questions.

## Critério arquitetural principal

> [!IMPORTANT]
> O módulo estará correto quando a API de IA puder confiar na identidade administrativa da API principal **sem se tornar dependente da implementação interna do auth do Adonis**.

---

# 20. Próximos Passos

## Na API Principal

- manter o login administrativo existente;
- usar `GET /api/v1/profile` como base inicial;
- criar `GET /api/v1/auth/me` como evolução correta;
- padronizar payload;
- garantir preload consistente de roles.

## Na API de IA

- criar o módulo `auth` completo;
- implementar `AuthGuard`;
- implementar `ScopesGuard`;
- criar `AuthenticatedUserMapper`;
- criar `ROLE_SCOPE_MAP`;
- proteger endpoints críticos da IA;
- escrever testes de integração ponta a ponta.

## Evolução futura sugerida

### Fase 2

- cache técnico controlado de introspecção;
- circuit breaker maduro;
- scopes mais granulares por caso de uso;
- segregação por capability do domínio.

### Fase 3

- políticas centralizadas por policy engine, se necessário;
- trilha de auditoria ampliada;
- readiness para expansão multi-serviço.

---

# 21. Conclusão Executiva

O desenho arquitetural está **coerente com o stack validado**, **compatível com o modelo técnico atual** e **adequado para implementação segura neste estágio da plataforma**.

## Síntese final

A API principal autentica.  
A API de IA confia.  
A API de IA normaliza.  
A API de IA traduz.  
A API de IA autoriza.  
A API de IA executa.

---

# Status do Documento

- **Tipo:** Documento arquitetural técnico final
- **Uso pretendido:** Entrega técnica / PR arquitetural / implementação
- **Escopo:** Slice de autenticação e autorização delegada da Fase 1
- **Status:** Pronto para revisão e implementação
- **Formato:** Markdown enterprise pronto para evolução incremental

