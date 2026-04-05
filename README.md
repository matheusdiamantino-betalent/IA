# Plataforma de Questões com IA

<div align="center">

![Status](https://img.shields.io/badge/status-Fase%201%20%E2%80%94%20Funda%C3%A7%C3%A3o%20Segura-0f766e?style=for-the-badge&logo=shield&logoColor=white)
![Roadmap](https://img.shields.io/badge/roadmap-Incremental%20Delivery-2563eb?style=for-the-badge&logo=target&logoColor=white)
![Arquitetura](https://img.shields.io/badge/architecture-Mon%C3%B3lito%20Modular-1d4ed8?style=for-the-badge&logo=stackshare&logoColor=white)
![Backend](https://img.shields.io/badge/backend-NestJS%20%2B%20TypeScript-e11d48?style=for-the-badge&logo=nestjs&logoColor=white)
![Persistência](https://img.shields.io/badge/persistence-PostgreSQL%20%2B%20PGVector-334155?style=for-the-badge&logo=postgresql&logoColor=white)
![Integração](https://img.shields.io/badge/integration-MySQL%20Legado-475569?style=for-the-badge&logo=mysql&logoColor=white)
![Coordenação](https://img.shields.io/badge/coordination-Redis%20%2B%20BullMQ-b91c1c?style=for-the-badge&logo=redis&logoColor=white)
![Auth](https://img.shields.io/badge/auth-Reuso%20da%20api%2Fv1-7c3aed?style=for-the-badge&logo=auth0&logoColor=white)
![Security](https://img.shields.io/badge/security-Secure%20by%20Default-065f46?style=for-the-badge&logo=vercel&logoColor=white)
![Observability](https://img.shields.io/badge/observability-Logs%20%2B%20Tracing%20%2B%20Metrics-7c3aed?style=for-the-badge&logo=datadog&logoColor=white)

</div>

<br />

<div align="center">

## 🏛️ Arquitetura orientada a domínio para processamento, governança, revisão e evolução incremental de questões com suporte a IA

Projeto desenhado para crescimento em fases, com foco em **segurança**, **modularidade**, **rastreabilidade operacional**, **baixo acoplamento**, **processamento assíncrono**, **reaproveitamento de infraestrutura existente** e **expansão sustentável até produção completa**.

</div>

---

# 📚 Sumário

- [1. Visão Executiva](#1-visão-executiva)
- [2. Status Atual do Projeto](#2-status-atual-do-projeto)
- [3. Decisão Arquitetural Oficial](#3-decisão-arquitetural-oficial)
- [4. Objetivo da Plataforma](#4-objetivo-da-plataforma)
- [5. Roadmap Oficial por Fases](#5-roadmap-oficial-por-fases)
- [6. Interpretação Arquitetural do Roadmap](#6-interpretação-arquitetural-do-roadmap)
- [7. Mapa Completo de Entregáveis por Fase](#7-mapa-completo-de-entregáveis-por-fase)
- [8. Escopo Real da Fase 1](#8-escopo-real-da-fase-1)
- [9. O que ainda não pertence à Fase 1](#9-o-que-ainda-não-pertence-à-fase-1)
- [10. Princípios Arquiteturais](#10-princípios-arquiteturais)
- [11. Visão Arquitetural de Alto Nível](#11-visão-arquitetural-de-alto-nível)
- [12. Reaproveitamento da Autenticação da `api/v1`](#12-reaproveitamento-da-autenticação-da-apiv1)
- [13. Estrutura Arquitetural por Módulo](#13-estrutura-arquitetural-por-módulo)
- [14. Regras de Dependência Obrigatórias](#14-regras-de-dependência-obrigatórias)
- [15. Bounded Contexts da Plataforma](#15-bounded-contexts-da-plataforma)
- [16. Fluxo Operacional da Fase 1](#16-fluxo-operacional-da-fase-1)
- [17. Fluxo Completo de Evolução do Produto](#17-fluxo-completo-de-evolução-do-produto)
- [18. Fluxo de Processamento de Questões (MVP)](#18-fluxo-de-processamento-de-questões-mvp)
- [19. Fluxo de Expansão para Materiais Didáticos](#19-fluxo-de-expansão-para-materiais-didáticos)
- [20. Tree View Arquitetural Proposta](#20-tree-view-arquitetural-proposta)
- [21. Modelo de Dados Conceitual da Fase 1](#21-modelo-de-dados-conceitual-da-fase-1)
- [22. Segurança](#22-segurança)
- [23. Observabilidade](#23-observabilidade)
- [24. Resiliência e Confiabilidade](#24-resiliência-e-confiabilidade)
- [25. ACL e Isolamento do Legado](#25-acl-e-isolamento-do-legado)
- [26. Pipeline Futuro e Compatibilidade Evolutiva](#26-pipeline-futuro-e-compatibilidade-evolutiva)
- [27. MVP vs Produção Completa](#27-mvp-vs-produção-completa)
- [28. Riscos Técnicos e Trade-offs](#28-riscos-técnicos-e-trade-offs)
- [29. Critérios de Pronto da Fase 1](#29-critérios-de-pronto-da-fase-1)
- [30. Stack Técnica](#30-stack-técnica)
- [31. Conclusão](#31-conclusão)

---

# 1. Visão Executiva

A **Plataforma de Questões com IA** foi concebida como uma base arquitetural robusta para suportar, com segurança e governança, o ciclo de vida de processamento, estruturação, revisão, enriquecimento e evolução de questões a partir de materiais-base e fluxos inteligentes.

A solução foi desenhada para evoluir progressivamente até um pipeline completo envolvendo:

- ingestão de PDFs;
- extração de conteúdo;
- classificação e enriquecimento de metadados;
- resolução de identificadores e vínculos;
- busca contextual e semântica;
- adaptação e transformação assistida por IA;
- geração de justificativas;
- validação automatizada;
- orquestração por agentes;
- processamento assíncrono;
- interface administrativa;
- revisão humana;
- observabilidade operacional;
- hardening para produção completa.

Entretanto, **o projeto não deve ser descrito como se todas essas capacidades já estivessem implementadas em produção**.

A implementação atual deve ser entendida dentro da **Fase 1 — Infraestrutura / Fundação Segura**, que representa a base correta sobre a qual as fases posteriores serão construídas.

## Tese arquitetural central

> **A arquitetura não foi desenhada para “resolver apenas o agora”. Ela foi desenhada para suportar o roadmap inteiro, sem que a base precise ser refeita a cada nova fase.**

Isso significa que:

- a Fase 1 precisa ser sólida o suficiente para sustentar as fases seguintes;
- o README precisa separar com precisão **estado atual**, **MVP** e **versão completa**;
- o roadmap precisa ser refletido como **evolução arquitetural real**, e não apenas como backlog visual.

---

# 2. Status Atual do Projeto

## 🟢 Situação correta do projeto

O projeto encontra-se na etapa de **fundação estrutural**, correspondente à **Fase 1**, ainda que a arquitetura global já esteja desenhada para sustentar todas as fases seguintes.

## O que isso significa tecnicamente

A base atual precisa estar pronta para suportar:

- persistência principal;
- integração com legado;
- cache;
- contratos internos;
- módulos coesos;
- autenticação reaproveitada;
- expansão futura com filas, agentes, indexação vetorial e operação assíncrona.

## O que não deve ser comunicado de forma incorreta

Ainda **não deve ser tratado como já pronto**:

- pipeline multiagente completo;
- orquestração plena em produção;
- busca semântica completa em produção;
- observabilidade madura;
- interface operacional completa;
- endurecimento final de produção.

---

# 3. Decisão Arquitetural Oficial

A direção arquitetural aprovada para a plataforma é:

## ✅ **Monólito Modular Pragmático por Domínio**

com:

- **NestJS + TypeScript**;
- organização por **módulos de domínio**;
- estrutura interna por módulo em:
  - `infra/`
  - `model/`
  - `lib/`
- uso disciplinado de `shared/`;
- reaproveitamento da autenticação já existente da `api/v1`;
- integração com base existente via **MySQL / ACL**;
- persistência principal em **PostgreSQL + PGVector**;
- coordenação e evolução assíncrona com **Redis + BullMQ**;
- uso seletivo de conceitos de Clean/Hexagonal **sem excesso de abstração**.

## Motivos da decisão

Essa arquitetura oferece o melhor equilíbrio entre:

- simplicidade operacional;
- clareza de ownership;
- baixa fricção de evolução;
- facilidade de manutenção;
- observabilidade centralizada;
- capacidade de crescer sem fragmentação prematura.

## O que foi conscientemente evitado

### ❌ Microsserviços prematuros
Porque aumentariam:
- custo operacional;
- overhead de tracing;
- contratos distribuídos;
- dificuldade de troubleshooting.

### ❌ Abstração excessiva no core
Porque geraria:
- boilerplate desnecessário;
- lentidão de evolução;
- custo cognitivo desproporcional.

### ❌ Duplicação de autenticação
Porque criaria:
- drift de identidade;
- inconsistência de escopo;
- divergência de regras de acesso.

---

# 4. Objetivo da Plataforma

A plataforma existe para suportar, com segurança e governança, o ciclo de vida de transformação de insumos em questões processadas, validadas, enriquecidas e operacionalmente utilizáveis, preservando compatibilidade futura com IA e com o ecossistema existente.

## Objetivos principais

- estruturar o pipeline de processamento de questões;
- centralizar metadados e classificações;
- suportar agentes de enriquecimento e validação;
- preparar o sistema para processamento assíncrono;
- permitir operação humana assistida;
- sustentar expansão para materiais didáticos e busca semântica.

---

# 5. Roadmap Oficial por Fases

O roadmap oficial da plataforma, conforme definido, está organizado da seguinte forma:

1. **Fase 1 — Infraestrutura**
2. **Fase 2 — Agentes Básicos**
3. **Fase 3 — Agentes Avançados**
4. **Fase 4 — API e Processamento Assíncrono**
5. **Fase 4B — Front-end Admin**
6. **Pré-MVP — CI/CD e Testes**
7. **🚀 Launch MVP — Processamento de Questões**
8. **Fase 5 — Materiais Didáticos**
9. **Fase 6 — Monitoramento**
10. **Fase 7 — Qualidade Final**
11. **🎉 Launch Produção — Versão Completa**

## Fluxograma executivo do roadmap

```mermaid
%%{init: {
  "theme": "base",
  "themeVariables": {
    "background": "#050816",
    "primaryColor": "#111827",
    "primaryTextColor": "#e5e7eb",
    "primaryBorderColor": "#38bdf8",
    "secondaryColor": "#0f172a",
    "secondaryTextColor": "#e5e7eb",
    "tertiaryColor": "#111827",
    "tertiaryTextColor": "#e5e7eb",
    "lineColor": "#94a3b8"
  }
}}%%
flowchart TD
    START["🚩 Início do Projeto"]

    F1["Fase 1<br/>Infraestrutura"]
    F2["Fase 2<br/>Agentes Básicos"]
    F3["Fase 3<br/>Agentes Avançados"]
    F4["Fase 4<br/>API e Processamento Assíncrono"]
    F4B["Fase 4B<br/>Front-end Admin"]
    PRE["Pré-MVP<br/>CI/CD e Testes"]
    MVP["🚀 Launch MVP<br/>Processamento de Questões"]
    F5["Fase 5<br/>Materiais Didáticos"]
    F6["Fase 6<br/>Monitoramento"]
    F7["Fase 7<br/>Qualidade Final"]
    PROD["🎉 Launch Produção<br/>Versão Completa"]

    START --> F1 --> F2 --> F3 --> F4 --> F4B --> PRE --> MVP --> F5 --> F6 --> F7 --> PROD
```

---

# 6. Interpretação Arquitetural do Roadmap

O roadmap deve ser lido em dois planos simultâneos.

## Plano A — Estado atual da implementação
A implementação atual está concentrada na **fundação correta** do sistema, com foco em infraestrutura, contratos, organização modular, persistência, integração, autenticação e preparo do pipeline.

## Plano B — Arquitetura alvo já decidida
Mesmo antes da implementação completa das próximas fases, a arquitetura precisa nascer compatível com:

- agentes;
- orquestração;
- filas;
- processamento assíncrono;
- indexação vetorial;
- busca semântica;
- observabilidade;
- endurecimento para produção.

> **A base atual deve respeitar o roadmap inteiro, mesmo que a implementação ainda esteja nos estágios iniciais.**

## Interpretação correta do roadmap

### O que já precisa existir na fundação
- boundaries corretos;
- contratos claros;
- persistência principal;
- compatibilidade com integração legada;
- cache;
- auth reaproveitada;
- desenho compatível com jobs e workers.

### O que só deve ser considerado como evolução
- pipeline multiagente completo;
- UI operacional madura;
- observabilidade plena;
- expansão vetorial;
- qualidade final de produção.

---

# 7. Mapa Completo de Entregáveis por Fase

## 🟦 Fase 1 — Infraestrutura

### Objetivo
Estabelecer a fundação técnica e operacional mínima correta da plataforma.

### Entregáveis
- Setup **NestJS + TypeScript**
- **PostgreSQL + PGVector**
- **MySQL Connection**
- **Redis + Cache**
- base inicial de **legislação / conhecimento primário**
- configuração de ambiente
- bootstrap do projeto
- contratos estruturais iniciais
- camada de integração com auth existente

### Resultado esperado
Ao final da Fase 1, o projeto precisa ter **uma fundação sólida e extensível**, ainda que sem pipeline avançado operacional.

---

## 🟪 Fase 2 — Agentes Básicos

### Objetivo
Introduzir os primeiros componentes automatizados do pipeline.

### Entregáveis
- **PDF Extraction Agent**
- **Classification Agent**
- **ID Resolution Agent**
- **Search Agent**

### Resultado esperado
A plataforma passa a possuir os primeiros blocos funcionais para:

- extrair conteúdo;
- enriquecer metadados;
- resolver vínculos;
- buscar contexto relevante.

---

## 🟧 Fase 3 — Agentes Avançados

### Objetivo
Adicionar inteligência editorial e validação semântica ao fluxo.

### Entregáveis
- **Adaptation Agent**
- **Answer Key Agent**
- **Validation Agent**
- **Orchestrator Agent**

### Resultado esperado
A solução passa a suportar:

- reformulação;
- geração de justificativas;
- validação semântica;
- coordenação entre agentes.

---

## 🟩 Fase 4 — API e Processamento Assíncrono

### Objetivo
Transformar o pipeline em um sistema operacional escalável.

### Entregáveis
- **REST API Endpoints**
- **Bull / BullMQ Queues**
- **Job Status e Progress Tracking**

### Resultado esperado
O sistema deixa de ser apenas um conjunto de blocos lógicos e passa a operar como uma plataforma de execução assíncrona.

---

## 🟥 Fase 4B — Front-end Admin

### Objetivo
Entregar a superfície humana de operação do MVP.

### Entregáveis
- dashboard principal
- upload e listagem
- interface de validação
- visualização de questões
- edição manual
- métricas e logs operacionais

### Resultado esperado
A operação humana passa a interagir diretamente com o fluxo do produto.

---

## 🟨 Pré-MVP — CI/CD e Testes

### Objetivo
Garantir lançabilidade mínima antes do MVP.

### Entregáveis
- testes unitários básicos
- pipeline de CI/CD
- deploy inicial

### Resultado esperado
O produto torna-se lançável com previsibilidade operacional mínima.

---

## 🚀 Launch MVP — Processamento de Questões

### Objetivo
Entregar a primeira versão funcional e utilizável da plataforma.

### Resultado esperado
Nesse ponto, a plataforma já deve ser capaz de:

- receber insumos;
- processar questões;
- coordenar etapas essenciais;
- disponibilizar uma operação mínima utilizável.

---

## 🟦 Fase 5 — Materiais Didáticos

### Objetivo
Expandir a plataforma para materiais-base e busca semântica.

### Entregáveis
- upload de PDFs didáticos
- chunking inteligente
- indexação vetorial
- API de busca semântica

### Resultado esperado
A solução passa a operar também como plataforma de conhecimento contextualizado.

---

## 🟪 Fase 6 — Monitoramento

### Objetivo
Consolidar visibilidade operacional madura.

### Entregáveis
- métricas
- dashboards
- logs estruturados

### Resultado esperado
A plataforma torna-se observável de forma consistente.

---

## 🟩 Fase 7 — Qualidade Final

### Objetivo
Endurecimento final antes da versão completa de produção.

### Entregáveis
- testes automatizados
- otimização de performance
- deploy final

### Resultado esperado
A plataforma alcança um patamar de robustez adequado para operação completa.

---

## 🎉 Launch Produção — Versão Completa

### Objetivo
Consolidar a versão completa da solução em produção.

### Resultado esperado
A plataforma opera com:

- pipeline completo;
- superfície operacional;
- observabilidade madura;
- expansão vetorial;
- qualidade endurecida.

---

# 8. Escopo Real da Fase 1

A Fase 1 é a camada fundacional da plataforma.

## Entregas corretas da Fase 1

### 🧱 Infraestrutura Base
- Setup **NestJS + TypeScript**
- estrutura de bootstrap
- configuração de ambiente
- base de módulos

### 🐘 Persistência Principal
- **PostgreSQL** como banco principal
- **PGVector** para compatibilidade futura com indexação vetorial
- estrutura inicial de entidades e migrações

### 🗄️ Integração com Base Existente
- conexão **MySQL** para leitura e/ou integração com base legada
- preparação para ACL

### ⚡ Cache e Coordenação
- **Redis**
- base para cache
- compatibilidade futura com filas e jobs

### 📚 Base Inicial de Conhecimento
- estrutura inicial para legislação e referência canônica

### 🔐 Segurança Estrutural
- autenticação reaproveitada
- autorização
- proteção básica de borda
- validação e sanitização

---

# 9. O que ainda não pertence à Fase 1

Os itens abaixo pertencem ao roadmap da solução, mas **não devem ser tratados como entregues agora**.

- agentes completos em produção;
- orquestração completa do pipeline;
- filas operacionais maduras;
- interface admin completa;
- upload e gestão avançada de materiais didáticos;
- chunking inteligente;
- indexação vetorial operacional;
- API semântica madura;
- dashboards completos;
- otimização final de performance;
- endurecimento final de produção.

---

# 10. Princípios Arquiteturais

## 10.1 Domain First
O domínio e o roadmap do produto definem a arquitetura.

## 10.2 Secure by Default
Toda entrada e integração é tratada como potencialmente insegura até validação explícita.

## 10.3 Modular by Responsibility
Cada módulo representa uma responsabilidade clara.

## 10.4 Shared with Discipline
`shared/` existe apenas para transversalidade real.

## 10.5 Async-Ready by Design
Mesmo antes da operação assíncrona completa, a base já deve nascer compatível com jobs, filas e workers.

## 10.6 Everything Auditable
Toda operação relevante deve poder ser rastreada.

## 10.7 Incremental Evolution
A arquitetura precisa evoluir sem reescrita estrutural.

## 10.8 Legacy Isolation
O legado deve ser consumido via ACL, e não internalizado diretamente no domínio novo.

---

# 11. Visão Arquitetural de Alto Nível

```mermaid
%%{init: {
  "theme": "base",
  "themeVariables": {
    "background": "#0b1220",
    "primaryColor": "#111827",
    "primaryTextColor": "#e5e7eb",
    "primaryBorderColor": "#38bdf8",
    "secondaryColor": "#0f172a",
    "secondaryTextColor": "#e5e7eb",
    "tertiaryColor": "#111827",
    "tertiaryTextColor": "#e5e7eb",
    "lineColor": "#94a3b8"
  }
}}%%
flowchart TB
    user["Usuário / Operador"]
    admin["Admin / Editorial"]
    legacyAuth["Auth existente — api/v1"]
    redis["Redis / BullMQ"]
    postgres["PostgreSQL / PGVector"]
    mysql["MySQL / Base Existente"]
    obs["Observabilidade"]

    subgraph app["Plataforma Backend"]
        api["API Core"]
        auth["Auth Adapter"]
        catalog["Catalog / Metadata"]
        questions["Questions / Processing Core"]
        staging["Staging / Review Buffer"]
        audit["Audit / Trail"]
        governance["Governance / Policies"]
        agents["Agents Layer (Futuro)"]
        retrieval["Semantic Retrieval (Futuro)"]
    end

    user --> api
    admin --> api
    api --> auth
    auth --> legacyAuth

    api --> catalog
    api --> questions
    api --> staging
    api --> audit
    api --> governance

    catalog --> postgres
    questions --> postgres
    staging --> postgres
    audit --> postgres

    api --> redis
    api --> obs
    governance --> mysql

    agents -. evolução .-> api
    retrieval -. evolução .-> postgres
```

## Leitura técnica

A arquitetura separa desde a base:

- entrada HTTP;
- identidade e autorização;
- núcleo de processamento;
- catálogo e metadados;
- staging e revisão futura;
- auditoria;
- governança;
- integração com legado;
- compatibilidade com agentes e busca semântica.

---

# 12. Reaproveitamento da Autenticação da `api/v1`

A nova plataforma **não deve implementar um sistema paralelo de autenticação**.

Ela deve:

- reaproveitar a infraestrutura de identidade existente;
- validar contexto e escopo via camada de adaptação;
- manter consistência com o ambiente atual;
- reduzir custo de integração e risco de drift.

## Fluxo arquitetural da autenticação

```mermaid
%%{init: {
  "theme": "base",
  "themeVariables": {
    "background": "#0b1220",
    "primaryColor": "#111827",
    "primaryTextColor": "#e5e7eb",
    "primaryBorderColor": "#8b5cf6",
    "secondaryColor": "#0f172a",
    "secondaryTextColor": "#e5e7eb",
    "tertiaryColor": "#111827",
    "tertiaryTextColor": "#e5e7eb",
    "lineColor": "#94a3b8"
  }
}}%%
flowchart LR
    A["Request autenticado"] --> B["Auth Module (Adapter)"]
    B --> C["Validação de token / contexto"]
    C --> D["Integração com api/v1"]
    D --> E["Resolução de identidade e permissões"]
    E --> F["Acesso liberado ao domínio"]
```

---

# 13. Estrutura Arquitetural por Módulo

Cada módulo segue uma organização interna simples, previsível e disciplinada.

```text
modules/<modulo>/
├── infra/
├── model/
└── lib/
```

## `model/`
Contém:
- DTOs;
- enums;
- interfaces;
- schemas;
- contratos;
- tipos;
- validações.

## `infra/`
Contém:
- controllers;
- services;
- repositories;
- processors;
- gateways;
- clients;
- adapters.

## `lib/`
Contém:
- helpers;
- parsers;
- mappers;
- normalizers;
- factories;
- utilitários do módulo.

---

# 14. Regras de Dependência Obrigatórias

## Permitido
- `infra` usar `model`;
- `infra` usar `lib`;
- `lib` usar `model`.

## Proibido
- `model` depender de `infra`;
- `lib` acessar integrações externas diretamente;
- `shared` virar “depósito genérico”;
- domínio absorver semântica do legado.

## Diagrama de dependência permitida

```mermaid
%%{init: {
  "theme": "base",
  "themeVariables": {
    "background": "#0b1220",
    "primaryColor": "#111827",
    "primaryTextColor": "#e5e7eb",
    "primaryBorderColor": "#14b8a6",
    "secondaryColor": "#0f172a",
    "secondaryTextColor": "#e5e7eb",
    "tertiaryColor": "#111827",
    "tertiaryTextColor": "#e5e7eb",
    "lineColor": "#94a3b8"
  }
}}%%
flowchart TD
    infra["infra"] --> model["model"]
    infra --> lib["lib"]
    lib --> model
```

---

# 15. Bounded Contexts da Plataforma

## Contextos estruturais da solução

### Núcleo atual / fundacional
- `auth`
- `organizations`
- `catalog`
- `questions`
- `staging`
- `audit`
- `governance`

### Evolução prevista
- `ingestion`
- `extraction`
- `classification`
- `resolution`
- `search`
- `adaptation`
- `answer-key`
- `validation`
- `orchestration`
- `materials`
- `monitoring`
- `quality`
- `publication`

---

# 16. Fluxo Operacional da Fase 1

```mermaid
%%{init: {
  "theme": "base",
  "themeVariables": {
    "background": "#0b1220",
    "primaryColor": "#111827",
    "primaryTextColor": "#e5e7eb",
    "primaryBorderColor": "#22c55e",
    "secondaryColor": "#0f172a",
    "secondaryTextColor": "#e5e7eb",
    "tertiaryColor": "#111827",
    "tertiaryTextColor": "#e5e7eb",
    "lineColor": "#94a3b8"
  }
}}%%
flowchart TD
    A["Entrada HTTP / Request"] --> B["Auth / Policy Guard"]
    B --> C["Validation / Sanitization"]
    C --> D["Use Case / Application Service"]
    D --> E["Regra de domínio"]
    E --> F["Persistência"]
    F --> G["Audit / Trail"]
    G --> H["Resposta padronizada"]
```

## Interpretação

A Fase 1 não é “vazia” apenas por não conter ainda todo o pipeline futuro.  
Ela já precisa garantir:

- segurança;
- estruturação;
- previsibilidade;
- persistência;
- rastreabilidade;
- base correta para expansão.

---

# 17. Fluxo Completo de Evolução do Produto

O fluxo abaixo traduz o roadmap em uma leitura arquitetural de ponta a ponta, sem cortes de texto e com nomes completos.

```mermaid
%%{init: {
  "theme": "base",
  "themeVariables": {
    "background": "#050816",
    "primaryColor": "#111827",
    "primaryTextColor": "#e5e7eb",
    "primaryBorderColor": "#f59e0b",
    "secondaryColor": "#0f172a",
    "secondaryTextColor": "#e5e7eb",
    "tertiaryColor": "#111827",
    "tertiaryTextColor": "#e5e7eb",
    "lineColor": "#94a3b8"
  }
}}%%
flowchart TD
    START["Início do Projeto"]

    subgraph P1["FASE 1 — Infraestrutura"]
        P1A["Setup NestJS + TypeScript"]
        P1B["PostgreSQL + PGVector"]
        P1C["MySQL Connection"]
        P1D["Redis + Cache"]
        P1E["Base de Legislação Inicial"]
        P1A --> P1B --> P1C --> P1D --> P1E
    end

    subgraph P2["FASE 2 — Agentes Básicos"]
        P2A["PDF Extraction Agent"]
        P2B["Classification Agent"]
        P2C["ID Resolution Agent"]
        P2D["Search Agent"]
        P2A --> P2B --> P2C --> P2D
    end

    subgraph P3["FASE 3 — Agentes Avançados"]
        P3A["Adaptation Agent"]
        P3B["Answer Key Agent"]
        P3C["Validation Agent"]
        P3D["Orchestrator Agent"]
        P3A --> P3B --> P3C --> P3D
    end

    subgraph P4["FASE 4 — API e Processamento Assíncrono"]
        P4A["REST API Endpoints"]
        P4B["Bull / BullMQ Queues"]
        P4C["Job Status e Progress Tracking"]
        P4A --> P4B --> P4C
    end

    subgraph P4B["FASE 4B — Front-end Admin"]
        P4B1["Dashboard Principal"]
        P4B2["Upload e Listagem"]
        P4B3["Interface de Validação"]
        P4B4["Visualização e Edição Manual"]
        P4B5["Métricas e Logs"]
        P4B1 --> P4B2 --> P4B3 --> P4B4 --> P4B5
    end

    subgraph PRE["PRÉ-MVP — CI/CD e Testes"]
        PRE1["Testes Unitários Básicos"]
        PRE2["Pipeline de CI/CD"]
        PRE3["Deploy"]
        PRE1 --> PRE2 --> PRE3
    end

    MVP["Launch MVP — Processamento de Questões"]

    subgraph P5["FASE 5 — Materiais Didáticos"]
        P5A["Upload de PDFs Didáticos"]
        P5B["Chunking Inteligente"]
        P5C["Indexação Vetorial"]
        P5D["API de Busca Semântica"]
        P5A --> P5B --> P5C --> P5D
    end

    subgraph P6["FASE 6 — Monitoramento"]
        P6A["Métricas e Dashboards"]
        P6B["Logs Estruturados"]
        P6A --> P6B
    end

    subgraph P7["FASE 7 — Qualidade Final"]
        P7A["Testes Automatizados"]
        P7B["Otimização de Performance"]
        P7C["Deploy Final"]
        P7A --> P7B --> P7C
    end

    PROD["Launch Produção — Versão Completa"]

    START --> P1 --> P2 --> P3 --> P4 --> P4B --> PRE --> MVP --> P5 --> P6 --> P7 --> PROD
```

---

# 18. Fluxo de Processamento de Questões (MVP)

Este diagrama representa o caminho lógico do produto até o **Launch MVP**.

```mermaid
%%{init: {
  "theme": "base",
  "themeVariables": {
    "background": "#050816",
    "primaryColor": "#111827",
    "primaryTextColor": "#e5e7eb",
    "primaryBorderColor": "#06b6d4",
    "secondaryColor": "#0f172a",
    "secondaryTextColor": "#e5e7eb",
    "tertiaryColor": "#111827",
    "tertiaryTextColor": "#e5e7eb",
    "lineColor": "#94a3b8"
  }
}}%%
flowchart LR
    A["PDF / Insumo"] --> B["Extraction Agent"]
    B --> C["Classification Agent"]
    C --> D["ID Resolution Agent"]
    D --> E["Search Agent"]
    E --> F["Adaptation Agent"]
    F --> G["Answer Key Agent"]
    G --> H["Validation Agent"]
    H --> I["Orchestrator Agent"]
    I --> J["REST API + Queues"]
    J --> K["Admin Review / Dashboard"]
    K --> L["MVP Operacional"]
```

## Objetivo desse fluxo
Esse pipeline representa a primeira cadeia funcional do produto, combinando:

- extração;
- classificação;
- enriquecimento;
- adaptação;
- validação;
- operação humana.

---

# 19. Fluxo de Expansão para Materiais Didáticos

Após o MVP, a solução evolui para ingestão de materiais-base e busca semântica.

```mermaid
%%{init: {
  "theme": "base",
  "themeVariables": {
    "background": "#050816",
    "primaryColor": "#111827",
    "primaryTextColor": "#e5e7eb",
    "primaryBorderColor": "#3b82f6",
    "secondaryColor": "#0f172a",
    "secondaryTextColor": "#e5e7eb",
    "tertiaryColor": "#111827",
    "tertiaryTextColor": "#e5e7eb",
    "lineColor": "#94a3b8"
  }
}}%%
flowchart LR
    A["PDF Didático"] --> B["Upload"]
    B --> C["Chunking Inteligente"]
    C --> D["Indexação Vetorial"]
    D --> E["API de Busca Semântica"]
    E --> F["Suporte ao pipeline de agentes"]
```

---

# 20. Tree View Arquitetural Proposta

```text
src/
├── main.ts
├── app.module.ts
│
├── bootstrap/
│   ├── app.bootstrap.ts
│   ├── config.bootstrap.ts
│   ├── logger.bootstrap.ts
│   ├── validation.bootstrap.ts
│   ├── exception-filters.bootstrap.ts
│   ├── metrics.bootstrap.ts
│   ├── tracing.bootstrap.ts
│   ├── queues.bootstrap.ts
│   ├── swagger.bootstrap.ts
│   └── shutdown.bootstrap.ts
│
├── config/
│   ├── app.config.ts
│   ├── auth.config.ts
│   ├── db.config.ts
│   ├── redis.config.ts
│   ├── queue.config.ts
│   ├── storage.config.ts
│   ├── llm.config.ts
│   ├── vector.config.ts
│   ├── observability.config.ts
│   ├── security.config.ts
│   └── feature-flags.config.ts
│
├── modules/
│   ├── auth/
│   │   ├── infra/
│   │   ├── model/
│   │   └── lib/
│   ├── organizations/
│   ├── catalog/
│   ├── questions/
│   ├── staging/
│   ├── audit/
│   ├── governance/
│   ├── ingestion/
│   ├── extraction/
│   ├── classification/
│   ├── resolution/
│   ├── search/
│   ├── adaptation/
│   ├── answer-key/
│   ├── validation/
│   ├── orchestration/
│   ├── materials/
│   ├── monitoring/
│   ├── quality/
│   └── publication/
│
├── shared/
│   ├── infra/
│   ├── model/
│   └── lib/
│
├── docs/
│   ├── architecture/
│   ├── adr/
│   ├── contracts/
│   └── runbooks/
│
└── test/
    ├── unit/
    ├── integration/
    ├── contract/
    ├── e2e/
    └── load/
```

---

# 21. Modelo de Dados Conceitual da Fase 1

## Entidades centrais esperadas

- `users`
- `roles`
- `permissions`
- `organizations`
- `collections`
- `disciplines`
- `topics`
- `legal_references`
- `questions`
- `question_versions`
- `audit_logs`

## Diagrama conceitual

```mermaid
%%{init: {
  "theme": "base",
  "themeVariables": {
    "background": "#0b1220",
    "primaryColor": "#111827",
    "primaryTextColor": "#e5e7eb",
    "primaryBorderColor": "#f59e0b",
    "secondaryColor": "#0f172a",
    "secondaryTextColor": "#e5e7eb",
    "tertiaryColor": "#111827",
    "tertiaryTextColor": "#e5e7eb",
    "lineColor": "#94a3b8"
  }
}}%%
erDiagram
    ORGANIZATIONS ||--o{ USERS : contains
    USERS ||--o{ AUDIT_LOGS : generates
    ORGANIZATIONS ||--o{ COLLECTIONS : owns
    COLLECTIONS ||--o{ DISCIPLINES : groups
    DISCIPLINES ||--o{ TOPICS : contains
    TOPICS ||--o{ QUESTIONS : classifies
    QUESTIONS ||--o{ QUESTION_VERSIONS : versions
    QUESTIONS }o--o{ LEGAL_REFERENCES : links
```

---

# 22. Segurança

## Controles mínimos da fundação

- autenticação obrigatória;
- autorização por escopo e política;
- validação forte de payload;
- sanitização de entrada;
- proteção contra vazamento de erro sensível;
- segregação de credenciais;
- trilha mínima de auditoria.

## Regras obrigatórias

1. Nenhuma rota sensível sem autenticação.
2. Nenhuma operação crítica sem autorização explícita.
3. Nenhum payload entra no domínio sem validação.
4. Nenhuma exceção técnica deve vazar stack em produção.
5. Nenhuma integração com legado deve bypassar a ACL.

---

# 23. Observabilidade

A observabilidade deve crescer junto com o roadmap.

## Na fundação
- logs estruturados;
- correlation id;
- tracing básico;
- health checks.

## No MVP
- status de jobs;
- visibilidade de pipeline;
- falhas por etapa.

## Na produção completa
- dashboards operacionais;
- métricas de throughput;
- métricas de falha;
- visibilidade por agente;
- logs estruturados maduros.

---

# 24. Resiliência e Confiabilidade

A base já deve nascer preparada para evolução operacional.

## Fundamentos
- tratamento consistente de erro;
- previsibilidade operacional;
- isolamento de responsabilidade;
- contratos estáveis;
- compatibilidade com retries e jobs.

## Evolução posterior
- retries por etapa;
- DLQ;
- reprocessamento;
- tolerância a falhas em agentes;
- observabilidade de degradação.

---

# 25. ACL e Isolamento do Legado

A base principal / legado **não deve contaminar o domínio novo**.

## Regra obrigatória
Toda integração com a base existente deve passar por uma **ACL (Anti-Corruption Layer)**.

## Fluxo conceitual

```mermaid
%%{init: {
  "theme": "base",
  "themeVariables": {
    "background": "#0b1220",
    "primaryColor": "#111827",
    "primaryTextColor": "#e5e7eb",
    "primaryBorderColor": "#ef4444",
    "secondaryColor": "#0f172a",
    "secondaryTextColor": "#e5e7eb",
    "tertiaryColor": "#111827",
    "tertiaryTextColor": "#e5e7eb",
    "lineColor": "#94a3b8"
  }
}}%%
flowchart LR
    A["Domínio interno canônico"] --> B["ACL de integração / publicação"]
    B --> C["Base principal / legado"]
```

---

# 26. Pipeline Futuro e Compatibilidade Evolutiva

A arquitetura atual já nasce preparada para o pipeline completo futuro.

```mermaid
%%{init: {
  "theme": "base",
  "themeVariables": {
    "background": "#050816",
    "primaryColor": "#111827",
    "primaryTextColor": "#e5e7eb",
    "primaryBorderColor": "#06b6d4",
    "secondaryColor": "#0f172a",
    "secondaryTextColor": "#e5e7eb",
    "tertiaryColor": "#111827",
    "tertiaryTextColor": "#e5e7eb",
    "lineColor": "#94a3b8"
  }
}}%%
flowchart LR
    A["PDF Extraction"] --> B["Classification"]
    B --> C["ID Resolution"]
    C --> D["Search"]
    D --> E["Adaptation"]
    E --> F["Answer Key"]
    F --> G["Validation"]
    G --> H["Orchestration"]
    H --> I["Async Processing"]
    I --> J["Human Review"]
    J --> K["Publication"]
```

## Interpretação
Esse fluxo **não significa que tudo já está pronto hoje**.  
Ele existe para mostrar que a fundação foi desenhada para receber esse pipeline sem exigir reestruturação radical.

---

# 27. MVP vs Produção Completa

## 🚀 MVP — Processamento de Questões
O MVP representa a primeira versão operacional utilizável da plataforma, com pipeline funcional mínimo, API, processamento assíncrono e superfície administrativa inicial.

## 🎉 Produção Completa
A versão completa adiciona:

- materiais didáticos;
- chunking;
- indexação vetorial;
- busca semântica;
- observabilidade madura;
- qualidade endurecida;
- performance otimizada;
- operação sustentável em escala.

## Tabela comparativa

| Dimensão | MVP | Produção Completa |
|---|---|---|
| Pipeline de agentes | Básico / funcional | Completo / robusto |
| API | Operacional | Endurecida |
| Filas | Essenciais | Maturidade operacional |
| Admin | Superfície inicial | Operação madura |
| Busca semântica | Não obrigatória | Ativa |
| Observabilidade | Básica | Completa |
| Qualidade | Mínima lançável | Hardening final |

---

# 28. Riscos Técnicos e Trade-offs

## Riscos controlados pela arquitetura

### 1. Acoplamento com legado
**Mitigação:** ACL + boundaries explícitos.

### 2. Crescimento desordenado do monólito
**Mitigação:** modularização por domínio + regras de dependência.

### 3. Explosão prematura de complexidade
**Mitigação:** roadmap incremental + fases bem separadas.

### 4. Duplicação de identidade/autorização
**Mitigação:** reuso da auth da `api/v1`.

### 5. Reescrita estrutural no meio do roadmap
**Mitigação:** arquitetura preparada desde a fundação para filas, vetores, agentes e evolução assíncrona.

---

# 29. Critérios de Pronto da Fase 1

A Fase 1 é considerada consistente quando possuir:

- setup estrutural estável;
- PostgreSQL + PGVector configurados;
- integração MySQL controlada;
- Redis funcional;
- autenticação integrada;
- contratos internos claros;
- logs mínimos;
- validação e sanitização;
- documentação coerente com a fase.

---

# 30. Stack Técnica

## Backend
- **NestJS**
- **TypeScript**

## Persistência
- **PostgreSQL**
- **PGVector**

## Integração
- **MySQL**

## Coordenação
- **Redis**
- **BullMQ**

## Observabilidade
- **Pino**
- **OpenTelemetry**

## Infraestrutura
- **Docker**
- **Docker Compose**

---

# 31. Conclusão

A arquitetura definida para esta plataforma não existe apenas para “organizar o código”.

Ela existe para garantir que o produto possa crescer corretamente ao longo das fases planejadas, preservando:

- segurança;
- consistência;
- governança;
- modularidade;
- rastreabilidade;
- compatibilidade com evolução futura.

A **Fase 1** deve ser entendida como a **fundação correta do produto**, e não como um recorte improvisado.

A partir dela, as fases seguintes podem ser adicionadas sem comprometer a integridade da solução.

<div align="center">

## 🚀 A fundação correta reduz reescrita, reduz acoplamento e acelera a evolução segura do produto

</div>
