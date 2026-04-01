# 🧠 Arquitetura Técnica — Agente de Geração de Questões com IA

<p align="center">

![Status](https://img.shields.io/badge/status-em%20evolução-2563eb?style=for-the-badge)
![Versão](https://img.shields.io/badge/versão-2.0.0-0f172a?style=for-the-badge)
![Arquitetura](https://img.shields.io/badge/arquitetura-clean%20%2B%20hexagonal-111827?style=for-the-badge)
![Resiliência](https://img.shields.io/badge/resiliência-timeout%20%2B%20retry%20%2B%20fallback-059669?style=for-the-badge)
![Observabilidade](https://img.shields.io/badge/observabilidade-logs%20%2B%20métricas%20%2B%20tracing-7c3aed?style=for-the-badge)
![Pipeline](https://img.shields.io/badge/pipeline-multiagente-f59e0b?style=for-the-badge)
![Stack](https://img.shields.io/badge/stack-NestJS%20%2B%20Postgres%20%2B%20Redis-ef4444?style=for-the-badge)
![Segurança](https://img.shields.io/badge/segurança-RBAC%20%2B%20JWT%20%2B%20rate%20limit-16a34a?style=for-the-badge)
![Versionamento](https://img.shields.io/badge/api-versionada%20v1%20%2F%20v2-7e22ce?style=for-the-badge)

</p>

> Documento técnico completo de arquitetura, versionamento de API, roteamento, segurança, dados, observabilidade, resiliência, organização de código e fluxos do serviço de IA responsável por transformar PDFs de provas em questões no formato Verdadeiro/Falso, com classificação, fundamentação legal, validação, revisão e publicação controlada.

---

## 📚 Sumário

- [1. Objetivo](#1-objetivo)
- [2. Contexto do Problema](#2-contexto-do-problema)
- [3. Objetivos do Sistema](#3-objetivos-do-sistema)
- [4. Requisitos Funcionais](#4-requisitos-funcionais)
- [5. Requisitos Não Funcionais](#5-requisitos-não-funcionais)
- [6. Regras de Negócio](#6-regras-de-negócio)
- [7. Princípios Arquiteturais](#7-princípios-arquiteturais)
- [8. Visão Geral da Solução](#8-visão-geral-da-solução)
- [9. Estratégia de Versionamento de API](#9-estratégia-de-versionamento-de-api)
- [10. Mapeamento Completo de Rotas](#10-mapeamento-completo-de-rotas)
- [11. Segurança da API](#11-segurança-da-api)
- [12. Estado Atual da Base Principal](#12-estado-atual-da-base-principal)
- [13. Modelo de Dados Atual](#13-modelo-de-dados-atual)
- [14. Diagrama ER do Modelo Atual](#14-diagrama-er-do-modelo-atual)
- [15. Arquitetura Proposta para o Agente](#15-arquitetura-proposta-para-o-agente)
- [16. Fluxograma Geral da Aplicação](#16-fluxograma-geral-da-aplicação)
- [17. Fluxo Macro do Sistema](#17-fluxo-macro-do-sistema)
- [18. Pipeline Multiagente](#18-pipeline-multiagente)
- [19. Fluxo de Revisão e Publicação](#19-fluxo-de-revisão-e-publicação)
- [20. Modelo de Dados Futuro](#20-modelo-de-dados-futuro)
- [21. Diagrama ER do Modelo Futuro](#21-diagrama-er-do-modelo-futuro)
- [22. Dicionário Completo de Banco de Dados](#22-dicionário-completo-de-banco-de-dados)
- [23. Camadas da Aplicação](#23-camadas-da-aplicação)
- [24. Arquitetura de Pastas e Arquivos](#24-arquitetura-de-pastas-e-arquivos)
- [25. Organização por Responsabilidade](#25-organização-por-responsabilidade)
- [26. Helpers, Normalizadores e Utilitários Globais](#26-helpers-normalizadores-e-utilitários-globais)
- [27. Contratos, DTOs, Enums e Tipos](#27-contratos-dtos-enums-e-tipos)
- [28. Logs Estruturados](#28-logs-estruturados)
- [29. Observabilidade](#29-observabilidade)
- [30. Timeouts, Retries e Circuit Breaker](#30-timeouts-retries-e-circuit-breaker)
- [31. Fallbacks e Estratégias de Degradação](#31-fallbacks-e-estratégias-de-degradação)
- [32. Idempotência](#32-idempotência)
- [33. Segurança, Robustez e Resiliência](#33-segurança-robustez-e-resiliência)
- [34. Erro como Parte do Fluxo](#34-erro-como-parte-do-fluxo)
- [35. Normalização e Qualidade de Dados](#35-normalização-e-qualidade-de-dados)
- [36. Payloads, Contratos JSON e Exemplos Completos](#36-payloads-contratos-json-e-exemplos-completos)
- [37. Estratégia de Testes](#37-estratégia-de-testes)
- [38. Regras Arquiteturais](#38-regras-arquiteturais)
- [39. Roadmap Técnico](#39-roadmap-técnico)
- [40. Conclusão](#40-conclusão)

---

# 1. Objetivo

Este documento descreve a arquitetura técnica completa do serviço de IA responsável por automatizar a geração de questões para a plataforma, convertendo PDFs de provas e gabaritos em questões no formato **Verdadeiro/Falso**, classificadas, explicadas e prontas para revisão editorial.

A proposta do sistema não é apenas gerar texto, mas operar como um **pipeline técnico robusto**, com foco em:

- qualidade e consistência de dados;
- rastreabilidade ponta a ponta;
- desacoplamento entre camadas;
- resiliência operacional;
- observabilidade nativa;
- idempotência e segurança;
- revisão humana estruturada;
- integração segura e versionada com a base principal;
- versionamento explícito de API e evolução controlada.

---

# 2. Contexto do Problema

Atualmente, a criação de questões depende de um processo manual que envolve:

- leitura de PDFs de provas;
- transcrição das questões;
- adaptação para o formato Verdadeiro/Falso;
- escrita manual do gabarito comentado;
- classificação e publicação no sistema principal.

Esse fluxo é lento, sujeito a inconsistências e pouco escalável. O novo serviço deve transformar esse processo em uma esteira automatizada, auditável e segura, sem comprometer a qualidade editorial.

---

# 3. Objetivos do Sistema

## 3.1 Processar PDFs de prova e gabarito
Ler arquivos enviados, extrair texto, identificar questões e estruturar a informação bruta.

## 3.2 Classificar metadados automaticamente
Identificar contexto como:

- banca;
- lei;
- artigo;
- disciplina;
- matéria;
- submatéria;
- demais atributos necessários para filtros e taxonomia.

## 3.3 Adaptar questões para o formato da plataforma
Converter a formulação original da prova para o formato Verdadeiro/Falso.

## 3.4 Gerar gabarito comentado minimalista
Produzir explicações curtas e consistentes, priorizando fundamentação legal.

## 3.5 Persistir com segurança
Salvar a saída em uma camada intermediária de staging, permitindo revisão antes da publicação.

## 3.6 Operar com confiabilidade
Ser observável, tolerante a falhas, idempotente e testável.

---

# 4. Requisitos Funcionais

## RF-001 Upload de PDFs
O sistema deve permitir o upload de até dois arquivos PDF por job: um para a prova e um para o gabarito.

## RF-002 Validação de arquivos
O sistema deve validar MIME real, extensão, tamanho máximo e checksum antes de aceitar o arquivo.

## RF-003 Criação de jobs de processamento
O sistema deve criar um job assíncrono idempotente para cada par de arquivos recebidos.

## RF-004 Extração de questões
O agente de extração deve identificar e estruturar todas as questões presentes nos PDFs.

## RF-005 Classificação contextual e pedagógica
O agente de classificação deve identificar banca, ano, órgão, cargo, disciplina, matéria e submatéria.

## RF-006 Resolução de IDs
O agente de resolução deve converter metadados classificados em IDs válidos existentes na base principal.

## RF-007 Busca de fundamentação legal
O agente de busca deve recuperar trechos legais relevantes para fundamentar as questões.

## RF-008 Adaptação para Verdadeiro/Falso
O agente de adaptação deve reformular cada questão no formato V/F com enunciado claro.

## RF-009 Geração de gabarito comentado
O agente de gabarito deve gerar explicação minimalista com embasamento legal.

## RF-010 Validação da saída
O agente de validação deve verificar completude, consistência e score antes de persistir.

## RF-011 Persistência em staging
Toda saída gerada deve ser salva em camada intermediária antes de qualquer publicação.

## RF-012 Fila de revisão humana
Drafts com baixo score ou anomalias devem ser encaminhados para revisão editorial.

## RF-013 Aprovação e publicação
Drafts aprovados devem ser publicados na base principal com todos os vínculos necessários.

## RF-014 Rejeição e descarte
Drafts rejeitados devem registrar motivo e ser marcados como descartados.

## RF-015 Reprocessamento de jobs
Jobs com falha devem poder ser reprocessados de forma idempotente.

## RF-016 Consulta de status
O sistema deve expor endpoints para consulta de status de jobs e drafts.

## RF-017 Métricas operacionais
O sistema deve expor métricas de jobs, agents, qualidade e infraestrutura.

## RF-018 Limpeza de arquivos temporários
O sistema deve realizar limpeza periódica de arquivos e locks expirados.

## RF-019 Autenticação e autorização
Todas as rotas devem exigir autenticação JWT e autorizar por papel (RBAC).

## RF-020 Versionamento de API
Todas as rotas devem ser versionadas com prefixo `/v1` ou `/v2`.

---

# 5. Requisitos Não Funcionais

## RNF-001 Disponibilidade
O serviço deve operar com disponibilidade mínima de 99,5% em ambiente de produção.

## RNF-002 Latência de API
Endpoints síncronos devem responder em até 500ms para 95% das requisições.

## RNF-003 Throughput de jobs
O sistema deve processar no mínimo 50 jobs simultâneos sem degradação de qualidade.

## RNF-004 Timeout por etapa
Cada etapa do pipeline deve ter timeout configurado e explícito, nunca operando sem limite de tempo.

## RNF-005 Retry seguro
Retries devem ser idempotentes, com backoff exponencial e jitter configurável.

## RNF-006 Rastreabilidade
Toda operação deve ser rastreável por `trace_id`, `job_id` e `draft_id`.

## RNF-007 Logs estruturados
Todos os logs devem ser estruturados em JSON, com campos mínimos padronizados.

## RNF-008 Segurança de uploads
Arquivos devem ser validados, armazenados fora de área pública e com TTL aplicado.

## RNF-009 Escalabilidade horizontal
Workers e agentes devem ser stateless e escaláveis horizontalmente.

## RNF-010 Isolamento de falhas
A falha em um agente não deve comprometer o pipeline completo.

## RNF-011 Auditabilidade
Toda modificação em draft ou publicação deve ser auditada com timestamp e responsável.

## RNF-012 Segurança de credenciais
Credenciais não devem aparecer em logs, payloads ou respostas de API.

## RNF-013 Rate limiting
Todos os endpoints devem ter rate limiting aplicado por usuário e por IP.

## RNF-014 Versionamento de prompts
Cada prompt enviado a um provider de IA deve ter versão registrada.

## RNF-015 Testabilidade
O sistema deve ter cobertura de testes mínima de 80% para camadas de domínio e aplicação.

---

# 6. Regras de Negócio

## RN-001 Staging obrigatório
Nenhuma questão gerada por IA pode ser publicada diretamente. Toda saída passa pela camada de staging.

## RN-002 Score mínimo para publicação automática
Drafts com `confidence_score` abaixo de 0.75 devem ser encaminhados para revisão manual antes de qualquer publicação.

## RN-003 Revisão humana em casos ambíguos
Quando o agente de validação detectar ambiguidade, classificação de baixa confiança ou falha parcial, o draft deve ir para a fila de revisão manual independentemente do score.

## RN-004 Idempotência de jobs
O upload do mesmo par de arquivos, identificado pelo `input_hash`, não deve criar um novo job. Deve retornar o job existente.

## RN-005 Contexto via filtros
Metadados contextuais como banca, ano, órgão e cargo devem ser representados exclusivamente via `filters` e `filter_types`. Não devem ser adicionadas colunas paralelas na tabela `questions`.

## RN-006 Taxonomia pedagógica preservada
A hierarquia `disciplines → matters → sub_matters` deve ser reutilizada sem criação de estruturas paralelas.

## RN-007 Publicação atômica
A publicação de um draft na base principal deve ser atômica: insere a questão, os filtros e as submatérias em uma única transação.

## RN-008 Motivo de rejeição obrigatório
Ao rejeitar um draft, o revisor deve registrar obrigatoriamente o `review_reason`.

## RN-009 Reprocessamento controlado
Um job só pode ser reprocessado se estiver nos estados `failed` ou `cancelled`. Jobs em estado `processing` não podem ser reiniciados.

## RN-010 Limite de tentativas
Um job ou etapa não pode ultrapassar 3 tentativas automáticas. Após isso, vai para dead-letter e requer intervenção manual.

## RN-011 Prompt versionado
Todo agente que invoca um LLM deve registrar a versão do prompt utilizado no `llm_invocations`.

## RN-012 Arquivo temporário com TTL
Arquivos de upload devem ter TTL máximo de 7 dias. Após esse período, são elegíveis para limpeza automática.

## RN-013 Segregação de credenciais de banco
O serviço de IA deve acessar o MySQL principal com credenciais segregadas, com permissão mínima necessária para leitura de filtros, taxonomia e escrita de questões publicadas.

## RN-014 Logs sem dados sensíveis
Payloads de LLM, PDFs e dados do usuário não devem ser logados sem mascaramento.

---

# 7. Princípios Arquiteturais

## 7.1 Desacoplamento
O domínio não deve depender diretamente de framework, banco ou provider de IA. Todo acesso externo ocorre por contratos e injeção de dependência.

## 7.2 Erro como parte do sistema
Falhas devem ser modeladas como estados explícitos do fluxo, não como exceções inesperadas.

## 7.3 Rastreabilidade
Toda saída deve poder ser auditada desde a origem até a publicação, com `trace_id` propagado em todo o pipeline.

## 7.4 Evolução segura
Novos agents, fontes e regras devem poder ser adicionados sem reescrever o núcleo.

## 7.5 Segurança por padrão
Uploads, integrações, prompts e persistência devem operar com validação e mínimo privilégio.

## 7.6 Observabilidade nativa
Logs, métricas e tracing devem fazer parte da arquitetura desde o início, não como adição posterior.

## 7.7 Idempotência
Reprocessamentos não devem gerar duplicidade nem efeitos colaterais indevidos.

## 7.8 Contratos explícitos
Toda integração entre camadas deve ser feita por interface ou contrato tipado, nunca por acoplamento direto a implementações concretas.

---

# 8. Visão Geral da Solução

A solução será implementada como um **novo serviço desacoplado**, separado da aplicação principal, utilizando:

- **NestJS + TypeScript** — framework principal
- **PostgreSQL + PGVector** — banco do agente com busca semântica
- **Redis** — cache, locks distribuídos e pub/sub
- **Bull/BullMQ** — execução assíncrona orientada a jobs
- **MySQL** — integração de leitura/escrita com o banco principal
- **Dashboard Admin** — revisão editorial e gestão operacional

O serviço funcionará como uma esteira assíncrona orientada a jobs e agents especializados, com API REST versionada, autenticação JWT, autorização RBAC e observabilidade completa.

---

# 9. Estratégia de Versionamento de API

## 9.1 Filosofia de versionamento

O versionamento de API é obrigatório e adota a estratégia de **prefixo de URL**. Toda rota deve conter o segmento `/v{N}/` logo após o prefixo de serviço.

O versionamento por URL foi escolhido porque:

- é explícito e visível no log e no monitoramento;
- é compatível com proxies, gateways e balanceadores sem configuração especial;
- facilita rollback por rota específica;
- permite que v1 e v2 coexistam durante períodos de transição.

## 9.2 Convenções

- a versão atual estável é sempre `v1`;
- quando houver breaking change em um contrato, cria-se `v2` para aquela rota;
- versões descontinuadas devem retornar `410 Gone` após período de deprecação de 90 dias;
- o header `X-API-Version` deve ser retornado em toda resposta com a versão efetiva atendida;
- o header `Deprecation` deve ser incluído quando a versão acessada estiver em processo de descontinuação.

## 9.3 Estrutura de URL

```
https://ai-service.domain.com/api/v1/{recurso}
https://ai-service.domain.com/api/v2/{recurso}
```

## 9.4 NestJS: configuração de versionamento

```typescript
// main.ts
import { VersioningType } from '@nestjs/common';

app.enableVersioning({
  type: VersioningType.URI,
  prefix: 'v',
  defaultVersion: '1',
});
```

## 9.5 Decorators de versão por controller

```typescript
@Controller({ path: 'jobs', version: '1' })
export class ProcessingJobsV1Controller {}

@Controller({ path: 'jobs', version: '2' })
export class ProcessingJobsV2Controller {}
```

## 9.6 Header de resposta de versão

Implementar interceptor global para injetar o header em todas as respostas:

```typescript
@Injectable()
export class ApiVersionInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const response = context.switchToHttp().getResponse();
    response.setHeader('X-API-Version', 'v1');
    return next.handle();
  }
}
```

---

# 10. Mapeamento Completo de Rotas

## 10.1 Convenções de roteamento

- todas as rotas usam kebab-case;
- recursos no plural;
- ações específicas como subpaths;
- versão obrigatória no prefixo;
- regex de validação de parâmetro definido por tipo.

## 10.2 Rotas de Health e Infraestrutura

| Método | Rota | Versão | Autenticação | Papel | Descrição |
|--------|------|--------|-------------|-------|-----------|
| `GET` | `/api/v1/health` | v1 | Pública | — | Health check básico |
| `GET` | `/api/v1/health/detailed` | v1 | JWT | `admin` | Health check com dependências |
| `GET` | `/api/v1/health/ready` | v1 | Pública | — | Readiness probe |
| `GET` | `/api/v1/health/live` | v1 | Pública | — | Liveness probe |
| `GET` | `/api/v1/metrics` | v1 | JWT | `admin` | Métricas Prometheus |

## 10.3 Rotas de Upload

| Método | Rota | Versão | Autenticação | Papel | Descrição |
|--------|------|--------|-------------|-------|-----------|
| `POST` | `/api/v1/uploads` | v1 | JWT | `operator`, `admin` | Faz upload de PDFs e cria job |
| `GET` | `/api/v1/uploads/:uploadId` | v1 | JWT | `operator`, `admin` | Consulta metadados do upload |

**Regex de parâmetro:** `:uploadId` → `[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}` (UUID v4)

## 10.4 Rotas de Jobs de Processamento

| Método | Rota | Versão | Autenticação | Papel | Descrição |
|--------|------|--------|-------------|-------|-----------|
| `POST` | `/api/v1/jobs` | v1 | JWT | `operator`, `admin` | Cria job de processamento |
| `GET` | `/api/v1/jobs` | v1 | JWT | `operator`, `admin` | Lista jobs com paginação e filtros |
| `GET` | `/api/v1/jobs/:jobId` | v1 | JWT | `operator`, `admin` | Consulta status detalhado do job |
| `GET` | `/api/v1/jobs/:jobId/steps` | v1 | JWT | `operator`, `admin` | Lista etapas do job |
| `POST` | `/api/v1/jobs/:jobId/retry` | v1 | JWT | `operator`, `admin` | Reprocessa job com falha |
| `POST` | `/api/v1/jobs/:jobId/cancel` | v1 | JWT | `admin` | Cancela job em execução |

**Regex de parâmetro:** `:jobId` → `[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}` (UUID v4)

## 10.5 Rotas de Drafts

| Método | Rota | Versão | Autenticação | Papel | Descrição |
|--------|------|--------|-------------|-------|-----------|
| `GET` | `/api/v1/drafts` | v1 | JWT | `reviewer`, `admin` | Lista drafts com paginação e filtros |
| `GET` | `/api/v1/drafts/:draftId` | v1 | JWT | `reviewer`, `admin` | Consulta draft completo |
| `PATCH` | `/api/v1/drafts/:draftId` | v1 | JWT | `reviewer`, `admin` | Atualiza campos do draft |
| `POST` | `/api/v1/drafts/:draftId/approve` | v1 | JWT | `reviewer`, `admin` | Aprova draft para publicação |
| `POST` | `/api/v1/drafts/:draftId/reject` | v1 | JWT | `reviewer`, `admin` | Rejeita draft com motivo |
| `POST` | `/api/v1/drafts/:draftId/publish` | v1 | JWT | `admin` | Publica draft aprovado |
| `GET` | `/api/v1/drafts/:draftId/sources` | v1 | JWT | `reviewer`, `admin` | Lista fontes e evidências do draft |
| `GET` | `/api/v1/drafts/:draftId/agent-runs` | v1 | JWT | `admin` | Lista execuções de agents do draft |

**Regex de parâmetro:** `:draftId` → `[0-9]+` (bigserial)

## 10.6 Rotas de Revisão Manual

| Método | Rota | Versão | Autenticação | Papel | Descrição |
|--------|------|--------|-------------|-------|-----------|
| `GET` | `/api/v1/review-queue` | v1 | JWT | `reviewer`, `admin` | Lista fila de revisão manual |
| `GET` | `/api/v1/review-queue/:itemId` | v1 | JWT | `reviewer`, `admin` | Consulta item da fila |
| `POST` | `/api/v1/review-queue/:itemId/assign` | v1 | JWT | `admin` | Atribui item a revisor |
| `POST` | `/api/v1/review-queue/:itemId/complete` | v1 | JWT | `reviewer`, `admin` | Conclui revisão do item |

**Regex de parâmetro:** `:itemId` → `[0-9]+` (bigserial)

## 10.7 Rotas de Observabilidade

| Método | Rota | Versão | Autenticação | Papel | Descrição |
|--------|------|--------|-------------|-------|-----------|
| `GET` | `/api/v1/observability/jobs/metrics` | v1 | JWT | `admin` | Métricas de jobs |
| `GET` | `/api/v1/observability/agents/metrics` | v1 | JWT | `admin` | Métricas de agents |
| `GET` | `/api/v1/observability/quality/metrics` | v1 | JWT | `admin` | Métricas de qualidade |
| `GET` | `/api/v1/observability/llm/metrics` | v1 | JWT | `admin` | Métricas de invocações LLM |
| `GET` | `/api/v1/observability/integration-logs` | v1 | JWT | `admin` | Logs de integração |

## 10.8 Rotas de Manutenção

| Método | Rota | Versão | Autenticação | Papel | Descrição |
|--------|------|--------|-------------|-------|-----------|
| `POST` | `/api/v1/maintenance/requeue-stuck-jobs` | v1 | JWT | `admin` | Recoloca jobs travados na fila |
| `POST` | `/api/v1/maintenance/cleanup-files` | v1 | JWT | `admin` | Limpa arquivos expirados |
| `POST` | `/api/v1/maintenance/cleanup-locks` | v1 | JWT | `admin` | Remove locks distribuídos expirados |

## 10.9 Roteamento com Regex no NestJS

```typescript
// Validação de UUID em rota
@Get(':jobId([0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12})')
async getJob(@Param('jobId') jobId: string) {}

// Validação de bigserial em rota
@Get(':draftId(\\d+)')
async getDraft(@Param('draftId') draftId: string) {}
```

---

# 11. Segurança da API

## 11.1 Autenticação

Toda rota protegida exige token JWT assinado com chave privada RS256. O token deve conter:

- `sub` — identificador do usuário;
- `role` — papel do usuário;
- `iat` — momento de emissão;
- `exp` — expiração;
- `jti` — identificador único do token para revogação.

```typescript
@Injectable()
export class JwtAuthGuard extends AuthGuard('jwt') {}
```

## 11.2 Autorização por Papel (RBAC)

Os papéis disponíveis são:

| Papel | Permissões |
|-------|------------|
| `admin` | Acesso total |
| `operator` | Upload, criação e consulta de jobs |
| `reviewer` | Consulta e revisão de drafts |
| `viewer` | Consulta somente leitura de jobs e drafts |

```typescript
@UseGuards(JwtAuthGuard, RolesGuard)
@Roles('admin', 'operator')
@Post('jobs')
async createJob() {}
```

## 11.3 Rate Limiting

Aplicado globalmente e configurável por rota:

| Escopo | Limite | Janela |
|--------|--------|--------|
| IP global | 300 req | 1 minuto |
| Usuário autenticado | 120 req | 1 minuto |
| Upload de arquivos | 10 req | 5 minutos |
| Retry de job | 5 req | 10 minutos |

```typescript
@Throttle({ default: { limit: 10, ttl: 300000 } })
@Post('uploads')
async uploadFiles() {}
```

## 11.4 Validação de Input

Toda requisição passa por validação com `class-validator` e `class-transformer` antes de atingir qualquer lógica de negócio.

```typescript
@UsePipes(new ValidationPipe({ whitelist: true, forbidNonWhitelisted: true, transform: true }))
```

## 11.5 Headers de Segurança

Configurados via `helmet`:

- `Strict-Transport-Security`
- `X-Frame-Options: DENY`
- `X-Content-Type-Options: nosniff`
- `Content-Security-Policy`
- `X-XSS-Protection`
- `Referrer-Policy: no-referrer`

## 11.6 CORS

Configurado com whitelist de origens permitidas. Não usar `*` em produção.

```typescript
app.enableCors({
  origin: process.env.ALLOWED_ORIGINS?.split(',') ?? [],
  methods: ['GET', 'POST', 'PATCH', 'DELETE'],
  allowedHeaders: ['Authorization', 'Content-Type', 'X-Request-ID'],
  exposedHeaders: ['X-API-Version', 'X-Request-ID', 'Retry-After'],
});
```

## 11.7 Correlation ID

Todo request deve ser identificado por um `X-Request-ID`. Se não enviado pelo cliente, o gateway deve gerar automaticamente.

```typescript
@Injectable()
export class RequestIdMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: Function) {
    const requestId = req.headers['x-request-id'] ?? crypto.randomUUID();
    req['requestId'] = requestId;
    res.setHeader('X-Request-ID', requestId);
    next();
  }
}
```

---

# 12. Estado Atual da Base Principal

A aplicação principal já possui uma base útil para integração do agente.

## 12.1 Entidade central
- `questions` — tabela principal de questões publicadas

## 12.2 Classificação contextual
- `filter_types` — tipos de filtro (banca, ano, órgão, cargo, etc.)
- `filters` — valores dos filtros
- `question_filters` — vínculo entre questão e filtros

## 12.3 Classificação pedagógica
- `disciplines` — disciplinas
- `matters` — matérias dentro de uma disciplina
- `sub_matters` — submatérias dentro de uma matéria
- `question_sub_matters` — vínculo entre questão e submatérias

## 12.4 Relacionamentos auxiliares
- `client_questions` — questões associadas a clientes
- `notebook_questions` — questões em cadernos de estudo
- `mock_questions` — questões em simulados

A base atual já oferece um bom núcleo de publicação, mas não possui camada de staging, jobs ou evidências voltada para IA.

---

# 13. Modelo de Dados Atual

## 13.1 Tabela `questions`

Entidade final de publicação das questões.

| Campo | Tipo | Nulável | Finalidade |
|-------|------|---------|------------|
| `id` | int | Não | Identificador da questão |
| `user_id` | uuid | Não | Usuário/admin responsável |
| `title` | text | Não | Título da questão |
| `description` | text | Não | Enunciado principal |
| `explanation` | text | Sim | Explicação / gabarito comentado |
| `is_true` | boolean | Não | Resposta correta Verdadeiro/Falso |
| `is_accepted` | boolean | Não | Estado de aceitação editorial |
| `is_from_client` | boolean | Não | Indica origem externa |
| `ia_generated` | boolean | Não | Indica geração por IA |
| `reason_refused` | text | Sim | Motivo da rejeição editorial |
| `created_at` | timestamp | Não | Criação |
| `updated_at` | timestamp | Não | Atualização |

## 13.2 Tabela `filter_types`

Categorias de filtros contextuais.

| Campo | Tipo | Nulável | Finalidade |
|-------|------|---------|------------|
| `id` | int | Não | Identificador |
| `name` | varchar | Não | Nome do tipo, ex.: `banca`, `ano` |
| `slug` | varchar | Não | Slug único, ex.: `banca`, `ano` |
| `created_at` | timestamp | Não | Criação |
| `updated_at` | timestamp | Não | Atualização |

## 13.3 Tabela `filters`

Valores dos filtros contextuais.

| Campo | Tipo | Nulável | Finalidade |
|-------|------|---------|------------|
| `id` | int | Não | Identificador |
| `filter_type_id` | int | Não | Tipo de filtro pai |
| `name` | varchar | Não | Nome do valor, ex.: `FGV` |
| `slug` | varchar | Não | Slug único |
| `created_at` | timestamp | Não | Criação |
| `updated_at` | timestamp | Não | Atualização |

## 13.4 Tabela `question_filters`

Vínculo entre questão e filtros.

| Campo | Tipo | Nulável | Finalidade |
|-------|------|---------|------------|
| `id` | int | Não | Identificador |
| `question_id` | int | Não | Questão relacionada |
| `filter_id` | int | Não | Filtro relacionado |
| `created_at` | timestamp | Não | Criação |

## 13.5 Tabela `disciplines`

Disciplinas pedagógicas.

| Campo | Tipo | Nulável | Finalidade |
|-------|------|---------|------------|
| `id` | int | Não | Identificador |
| `name` | varchar | Não | Nome da disciplina |
| `slug` | varchar | Não | Slug único |
| `created_at` | timestamp | Não | Criação |
| `updated_at` | timestamp | Não | Atualização |

## 13.6 Tabela `matters`

Matérias dentro de uma disciplina.

| Campo | Tipo | Nulável | Finalidade |
|-------|------|---------|------------|
| `id` | int | Não | Identificador |
| `discipline_id` | int | Não | Disciplina pai |
| `name` | varchar | Não | Nome da matéria |
| `slug` | varchar | Não | Slug único |
| `created_at` | timestamp | Não | Criação |
| `updated_at` | timestamp | Não | Atualização |

## 13.7 Tabela `sub_matters`

Submatérias dentro de uma matéria.

| Campo | Tipo | Nulável | Finalidade |
|-------|------|---------|------------|
| `id` | int | Não | Identificador |
| `matter_id` | int | Não | Matéria pai |
| `name` | varchar | Não | Nome da submatéria |
| `slug` | varchar | Não | Slug único |
| `created_at` | timestamp | Não | Criação |
| `updated_at` | timestamp | Não | Atualização |

## 13.8 Tabela `question_sub_matters`

Vínculo entre questão e submatérias.

| Campo | Tipo | Nulável | Finalidade |
|-------|------|---------|------------|
| `id` | int | Não | Identificador |
| `question_id` | int | Não | Questão relacionada |
| `sub_matter_id` | int | Não | Submatéria relacionada |
| `position` | int | Sim | Ordem de exibição |
| `created_at` | timestamp | Não | Criação |

## 13.9 Tabela `client_questions`

Associação de questões a clientes.

| Campo | Tipo | Nulável | Finalidade |
|-------|------|---------|------------|
| `id` | int | Não | Identificador |
| `client_id` | int | Não | Cliente |
| `question_id` | int | Não | Questão |
| `created_at` | timestamp | Não | Criação |

## 13.10 Tabela `notebook_questions`

Questões inseridas em cadernos de estudo.

| Campo | Tipo | Nulável | Finalidade |
|-------|------|---------|------------|
| `id` | int | Não | Identificador |
| `notebook_id` | int | Não | Caderno |
| `question_id` | int | Não | Questão |
| `position` | int | Sim | Ordem |
| `created_at` | timestamp | Não | Criação |

## 13.11 Tabela `mock_questions`

Questões inseridas em simulados.

| Campo | Tipo | Nulável | Finalidade |
|-------|------|---------|------------|
| `id` | int | Não | Identificador |
| `mock_exam_id` | int | Não | Simulado |
| `question_id` | int | Não | Questão |
| `position` | int | Sim | Ordem |
| `created_at` | timestamp | Não | Criação |

---

# 14. Diagrama ER do Modelo Atual

```mermaid
erDiagram
    direction LR

    ADMINS {
        uuid id PK
        varchar name
        varchar email
        timestamp created_at
    }

    QUESTIONS {
        int id PK
        uuid user_id FK
        text title
        text description
        text explanation
        boolean is_true
        boolean is_accepted
        boolean is_from_client
        boolean ia_generated
        text reason_refused
        timestamp created_at
        timestamp updated_at
    }

    FILTER_TYPES {
        int id PK
        varchar name
        varchar slug
        timestamp created_at
    }

    FILTERS {
        int id PK
        int filter_type_id FK
        varchar name
        varchar slug
        timestamp created_at
    }

    QUESTION_FILTERS {
        int id PK
        int question_id FK
        int filter_id FK
        timestamp created_at
    }

    DISCIPLINES {
        int id PK
        varchar name
        varchar slug
        timestamp created_at
    }

    MATTERS {
        int id PK
        int discipline_id FK
        varchar name
        varchar slug
        timestamp created_at
    }

    SUB_MATTERS {
        int id PK
        int matter_id FK
        varchar name
        varchar slug
        timestamp created_at
    }

    QUESTION_SUB_MATTERS {
        int id PK
        int question_id FK
        int sub_matter_id FK
        int position
        timestamp created_at
    }

    CLIENTS {
        int id PK
        varchar name
        timestamp created_at
    }

    CLIENT_QUESTIONS {
        int id PK
        int client_id FK
        int question_id FK
        timestamp created_at
    }

    NOTEBOOKS {
        int id PK
        varchar name
        timestamp created_at
    }

    NOTEBOOK_QUESTIONS {
        int id PK
        int notebook_id FK
        int question_id FK
        int position
        timestamp created_at
    }

    MOCK_EXAMS {
        int id PK
        varchar name
        timestamp created_at
    }

    MOCK_QUESTIONS {
        int id PK
        int mock_exam_id FK
        int question_id FK
        int position
        timestamp created_at
    }

    ADMINS ||--o{ QUESTIONS : "cria"
    QUESTIONS ||--o{ QUESTION_FILTERS : "possui"
    FILTERS ||--o{ QUESTION_FILTERS : "classifica"
    FILTER_TYPES ||--o{ FILTERS : "categoriza"
    DISCIPLINES ||--o{ MATTERS : "contém"
    MATTERS ||--o{ SUB_MATTERS : "contém"
    QUESTIONS ||--o{ QUESTION_SUB_MATTERS : "possui"
    SUB_MATTERS ||--o{ QUESTION_SUB_MATTERS : "classifica"
    QUESTIONS ||--o{ CLIENT_QUESTIONS : "atribuída a"
    CLIENTS ||--o{ CLIENT_QUESTIONS : "possui"
    QUESTIONS ||--o{ NOTEBOOK_QUESTIONS : "incluída em"
    NOTEBOOKS ||--o{ NOTEBOOK_QUESTIONS : "contém"
    QUESTIONS ||--o{ MOCK_QUESTIONS : "incluída em"
    MOCK_EXAMS ||--o{ MOCK_QUESTIONS : "contém"
```

---

# 15. Arquitetura Proposta para o Agente

O agente deve operar como um microserviço independente, conectado à aplicação principal, sem depender da lógica interna dela.

## 15.1 Responsabilidades do novo serviço

- receber PDFs via API;
- validar e armazenar arquivos com segurança;
- orquestrar o pipeline de processamento;
- extrair questões dos PDFs;
- classificar contexto e taxonomia;
- resolver IDs da base principal;
- buscar fundamentação legal;
- adaptar para V/F;
- gerar explicações comentadas;
- validar saída dos agentes;
- persistir rascunhos em staging;
- gerenciar fila de revisão humana;
- publicar conteúdo aprovado na base principal.

## 15.2 Componentes principais

- **MySQL** — base principal, leitura de filtros e taxonomia, escrita de questões publicadas
- **PostgreSQL + PGVector** — banco exclusivo do agente com busca semântica
- **Redis** — cache, locks distribuídos e filas
- **Bull/BullMQ** — execução assíncrona e dead-letter queue
- **NestJS** — APIs versionadas e orquestração
- **LLM Provider** — classificação, adaptação e geração
- **Dashboard Admin** — revisão humana e monitoramento

---

# 16. Fluxograma Geral da Aplicação

```mermaid
flowchart TD
    subgraph UI["🖥️ Interface"]
        DASH["Dashboard Admin\nUpload · Revisão · Aprovação"]
    end

    subgraph API_LAYER["🌐 API NestJS — Versão v1"]
        API["REST API Versionada\nJWT + RBAC + Rate Limit"]
        MW["Middlewares\nCorrelation ID · Helmet · CORS"]
    end

    subgraph ASYNC["⚙️ Processamento Assíncrono"]
        JOB["📋 processing_jobs\nCriação Idempotente"]
        QUEUE["📬 BullMQ · Redis\nFila de Jobs"]
        DLQ["☠️ Dead Letter Queue\n> 3 tentativas"]
        ORCH["🎛️ Orchestrator\nCoordenação do Pipeline"]
    end

    subgraph AGENTS["🤖 Pipeline Multiagente"]
        PDF["📄 PDF Extraction\nAgent"]
        CLS["🏷️ Classification\nAgent"]
        RES["🔗 ID Resolution\nAgent"]
        SEA["📚 Search\nAgent"]
        ADP["🧠 Adaptation\nAgent"]
        ANS["📝 Answer Key\nAgent"]
        VAL["✅ Validation\nAgent"]
    end

    subgraph STORAGE["🗄️ Persistência"]
        PG["PostgreSQL\nPGVector"]
        MYSQL["MySQL\nBase Principal"]
        REDIS_CACHE["Redis\nCache · Locks"]
        S3["Storage S3\nArquivos PDF"]
    end

    subgraph STAGING["🗃️ Staging e Revisão"]
        DRAFTS["question_drafts\nRascunhos IA"]
        REVIEW["Fila de Revisão\nmanual_review_queue"]
        PUB["Publicação\nControlada"]
    end

    subgraph OBS["📈 Observabilidade"]
        LOGS["Logs Estruturados\nPino + JSON"]
        METRICS["Métricas\nPrometheus"]
        TRACE["Tracing\nOpenTelemetry"]
    end

    DASH --> API
    API --> MW
    MW --> JOB
    JOB --> QUEUE
    QUEUE --> ORCH
    QUEUE -->|falha > 3x| DLQ

    ORCH --> PDF
    ORCH --> CLS
    ORCH --> RES
    ORCH --> SEA
    ORCH --> ADP
    ORCH --> ANS
    ORCH --> VAL

    PDF --> S3
    PDF --> PG
    SEA --> PG
    RES --> MYSQL
    RES --> REDIS_CACHE

    VAL -->|válido| DRAFTS
    VAL -->|inválido ou baixo score| REVIEW
    DRAFTS --> REVIEW
    REVIEW -->|aprovado| PUB
    PUB --> MYSQL

    ORCH --> LOGS
    ORCH --> METRICS
    ORCH --> TRACE
    API --> LOGS
    PUB --> LOGS
```

---

# 17. Fluxo Macro do Sistema

```mermaid
flowchart TD
    A(["📄 Upload de PDFs\nProva + Gabarito"]) --> B["🔐 Validação\nMIME · Tamanho · Hash"]
    B -->|inválido| ERR1(["❌ 422 Unprocessable\nErro de validação"])
    B -->|válido| C["📋 Criar processing_job\nIdempotente por input_hash"]
    C -->|job já existe| EXIST(["♻️ Retornar job existente\n200 com jobId"])
    C -->|novo job| D["⏳ Enfileirar no BullMQ\nPrioridade configurável"]

    D --> E["🤖 Pipeline de Agentes"]

    E --> F["1️⃣ PDF Extraction Agent\n30s – 90s"]
    F -->|falha| RETRY1["🔁 Retry até 2x\nBackoff exponencial"]
    RETRY1 -->|esgotou| DLQ1(["☠️ Dead Letter Queue"])
    F -->|sucesso| G["2️⃣ Classification Agent\n10s – 30s"]

    G --> H["3️⃣ ID Resolution Agent\n2s – 5s"]
    H --> I["4️⃣ Search Agent\n5s – 15s"]
    I --> J["5️⃣ Adaptation Agent\n10s – 20s"]
    J --> K["6️⃣ Answer Key Agent\n10s – 20s"]
    K --> L["7️⃣ Validation Agent\n1s – 5s"]

    L -->|score >= 0.75 e válido| M["🗃️ Salvar em question_drafts\nstatus: valid"]
    L -->|score < 0.75 ou ambíguo| N["🗃️ Salvar em question_drafts\nstatus: needs_review"]
    L -->|inválido| O["❌ Registrar falha estruturada\nstatus: invalid"]

    M --> P{"Auto-publicar?"}
    P -->|não - revisão obrigatória| Q["👨‍⚖️ Fila de Revisão\nmanual_review_queue"]
    N --> Q
    P -->|sim - aprovação automática| R["✅ Publicar na base principal"]

    Q -->|aprovado pelo revisor| R
    Q -->|rejeitado| S["🔁 Ajuste ou Descarte\nreview_reason obrigatório"]

    R --> T["📝 Inserir em questions\nfiltros + submatérias\ntransação atômica"]
    T --> U(["🚀 Questão disponível\nna plataforma"])
```

---

# 18. Pipeline Multiagente

Cada agente possui responsabilidade única, contrato claro e retorno tipado. Nenhum agente conhece a implementação interna de outro.

## 18.1 Diagrama Sequencial do Pipeline

```mermaid
sequenceDiagram
    autonumber
    participant O as 🎛️ Orchestrator
    participant PDF as 📄 PDF Agent
    participant CLS as 🏷️ Classification Agent
    participant RES as 🔗 Resolution Agent
    participant SEA as 📚 Search Agent
    participant ADP as 🧠 Adaptation Agent
    participant ANS as 📝 Answer Key Agent
    participant VAL as ✅ Validation Agent
    participant DB as 🗄️ PostgreSQL

    O->>PDF: execute(uploadedFiles)
    PDF-->>O: RawQuestionsPayload | ExtractionError

    O->>CLS: execute(rawQuestions)
    CLS-->>O: ClassificationPayload | ClassificationError

    O->>RES: execute(classifiedMetadata)
    RES-->>O: ResolvedIdsPayload | ResolutionError

    O->>SEA: execute(classifiedMetadata)
    SEA-->>O: SearchResultsPayload | SearchError

    loop Para cada questão extraída
        O->>ADP: execute(rawQuestion, resolvedIds, searchResults)
        ADP-->>O: AdaptationPayload | AdaptationError

        O->>ANS: execute(adaptedQuestion, searchResults)
        ANS-->>O: AnswerKeyPayload | AnswerKeyError

        O->>VAL: execute(completePayload)
        VAL-->>O: ValidationResult

        O->>DB: save(questionDraft)
        DB-->>O: draftId
    end

    O-->>O: markJobAsCompleted
```

## 18.2 PDF Extraction Agent

Responsável por ler PDFs, extrair texto e estruturar questões brutas.

**Entrada:**
- arquivo da prova (storage path);
- arquivo do gabarito (storage path).

**Saída:**
- lista de questões brutas com número, enunciado e resposta identificada;
- segmentos identificados;
- erros de parsing por questão, quando existirem.

**Timeout:** 30s a 90s conforme tamanho do arquivo.

**Retry:** até 2 tentativas para falhas transitórias. Não retentar em erros de parsing estrutural.

## 18.3 Classification Agent

Responsável por identificar metadados contextuais e pedagógicos via LLM.

**Saída:**
- `bank_name` — nome da banca;
- `year` — ano da prova;
- `organization_name` — órgão;
- `role_name` — cargo;
- `law_name` — lei referenciada;
- `article_reference` — artigo referenciado;
- `discipline_name` — disciplina;
- `matter_name` — matéria;
- `sub_matter_name` — submatéria;
- `classification_confidence` — score de confiança da classificação.

**Timeout:** 10s a 30s por lote.

**Retry:** apenas em falhas transitórias do provider.

## 18.4 ID Resolution Agent

Responsável por transformar metadados classificados em IDs válidos da plataforma.

**Fontes de resolução:**
1. Redis cache (TTL 1h);
2. consulta MySQL por slug normalizado;
3. busca aproximada controlada;
4. marcação para revisão manual se não encontrado.

**Saída:**
- IDs resolvidos para cada metadado;
- lista de ambiguidades;
- lista de não encontrados.

**Timeout:** 2s a 5s por lookup agregado.

**Retry:** até 3 tentativas rápidas com backoff de 100ms, 300ms e 600ms.

## 18.5 Search Agent

Responsável por recuperar fundamentação legal confiável.

**Fontes:**
1. PostgreSQL com PGVector (embedding semântico);
2. índice legal estruturado;
3. busca web controlada como fallback de último recurso.

**Saída:**
- trechos legais relevantes;
- referências de fonte;
- `rank_score` por trecho;
- evidências utilizadas.

**Timeout:** 5s a 15s por consulta.

**Retry:** apenas para indisponibilidade transitória do banco vetorial.

## 18.6 Adaptation Agent

Responsável por reformular a questão original para o formato Verdadeiro/Falso.

**Saída:**
- `title` — título curto e direto;
- `description` — enunciado completo no formato V/F;
- `is_true` — resposta correta booleana.

**Timeout:** 10s a 20s por questão.

**Retry:** não retentar em erros de schema inválido. Retentar apenas em timeouts.

## 18.7 Answer Key Agent

Responsável por gerar o gabarito comentado minimalista.

**Saída:**
- `explanation` — texto HTML formatado, com `<strong>` para termos legais, citação da lei e artigo, justificativa clara e concisa.

**Timeout:** 10s a 20s por questão.

## 18.8 Validation Agent

Responsável por validar consistência, completude e regras de negócio antes da persistência.

**Verificações realizadas:**
- presença de `title`, `description`, `explanation` e `is_true`;
- `title` não pode ser idêntico a `description`;
- `explanation` deve conter ao menos uma referência legal;
- `confidence_score` calculado com base nos scores dos agentes anteriores;
- detecção de inconsistência entre `is_true` e o conteúdo do `explanation`.

**Saída:**
- `validation_status`: `valid`, `invalid`, `needs_review`;
- `confidence_score`: número entre 0 e 1;
- lista de erros estruturados;
- flag `requires_manual_review`.

**Timeout:** 1s a 5s por questão (processamento local).

---

# 19. Fluxo de Revisão e Publicação

```mermaid
flowchart TD
    A(["🤖 Saída do Agente\nquestion_draft gerado"]) --> B["🔍 Verificar validation_status"]

    B -->|valid e score >= 0.75| C["🏷️ Aplicar draft_filters\nContexto classificado"]
    B -->|needs_review ou score < 0.75| QUEUE["📋 Encaminhar para\nmanual_review_queue"]
    B -->|invalid| ERR(["❌ Registrar\nerror_code + error_message"])

    C --> D["📚 Aplicar draft_sub_matters\nTaxonomia pedagógica"]
    D --> E["👨‍⚖️ Dashboard Editorial\nRevisão humana"]

    E --> F{"Decisão do Revisor"}

    F -->|Aprovado sem alteração| G["✅ review_status: approved"]
    F -->|Aprovado com edição| H["✏️ Atualizar draft\nreview_status: approved"]
    F -->|Rejeitado| I["❌ review_status: rejected\nreview_reason obrigatório"]

    QUEUE --> E

    G --> J["🚀 Iniciar Publicação\nTransação Atômica"]
    H --> J

    J --> K["📝 INSERT em questions\n(MySQL Principal)"]
    K --> L["🔗 INSERT em question_filters\nfiltros contextuais"]
    L --> M["🔗 INSERT em question_sub_matters\ntaxonomia pedagógica"]
    M --> N["✅ COMMIT"]
    N --> O(["🟢 Questão Disponível\nna Plataforma"])

    J -->|erro na transação| P["⚠️ ROLLBACK\nregistrar integration_log"]
    P --> Q["🔁 Retentar publicação\naté 3 tentativas"]
    Q -->|esgotou| R(["☠️ Falha de Publicação\nRequer intervenção manual"])

    I --> S["🔁 Reprocessar ou Descartar\ndecisão do admin"]
```

---

# 20. Modelo de Dados Futuro

Para suportar IA de forma profissional, será necessário criar uma camada de staging e auditoria independente da base principal.

## 20.1 Novas tabelas propostas

- `processing_jobs` — unidade de processamento
- `processing_job_steps` — etapas do job
- `uploaded_files` — arquivos recebidos
- `question_drafts` — rascunhos gerados
- `question_sources` — evidências e fontes
- `draft_filters` — filtros contextuais do draft
- `draft_sub_matters` — taxonomia pedagógica do draft
- `agent_runs` — execuções individuais de agentes
- `llm_invocations` — chamadas a LLMs
- `integration_logs` — logs de integração com sistemas externos
- `manual_review_queue` — fila de revisão humana

---

# 21. Diagrama ER do Modelo Futuro

```mermaid
erDiagram
    direction TB

    PROCESSING_JOBS {
        uuid id PK
        varchar job_key UK
        varchar source_type
        varchar status
        integer progress_pct
        varchar input_hash UK
        uuid requested_by
        jsonb metadata
        timestamp started_at
        timestamp finished_at
        timestamp created_at
        timestamp updated_at
    }

    PROCESSING_JOB_STEPS {
        bigserial id PK
        uuid processing_job_id FK
        varchar step_name
        integer step_order
        varchar status
        integer attempt
        varchar error_code
        text error_message
        jsonb payload
        timestamp started_at
        timestamp finished_at
        timestamp created_at
    }

    UPLOADED_FILES {
        uuid id PK
        uuid processing_job_id FK
        varchar file_role
        varchar original_name
        varchar storage_path
        varchar mime_type
        bigint size_bytes
        varchar checksum_sha256
        boolean is_valid
        jsonb validation_errors
        timestamp created_at
    }

    QUESTION_DRAFTS {
        bigserial id PK
        uuid processing_job_id FK
        varchar source_question_number
        text title
        text description
        text explanation
        boolean is_true
        numeric confidence_score
        varchar validation_status
        varchar review_status
        uuid reviewed_by
        text review_reason
        jsonb raw_payload
        jsonb normalized_payload
        timestamp created_at
        timestamp updated_at
    }

    QUESTION_SOURCES {
        bigserial id PK
        bigint question_draft_id FK
        varchar source_type
        varchar source_ref
        varchar title
        text excerpt
        numeric rank_score
        jsonb metadata
        timestamp created_at
    }

    DRAFT_FILTERS {
        bigserial id PK
        bigint question_draft_id FK
        integer filter_id
        numeric resolution_confidence
        text resolution_note
        timestamp created_at
    }

    DRAFT_SUB_MATTERS {
        bigserial id PK
        bigint question_draft_id FK
        integer sub_matter_id
        integer position
        numeric resolution_confidence
        timestamp created_at
    }

    AGENT_RUNS {
        uuid id PK
        uuid processing_job_id FK
        bigint question_draft_id FK
        varchar agent_name
        varchar agent_version
        varchar status
        integer attempt
        jsonb input_snapshot
        jsonb output_snapshot
        varchar error_code
        text error_message
        timestamp started_at
        timestamp finished_at
        timestamp created_at
    }

    LLM_INVOCATIONS {
        uuid id PK
        uuid agent_run_id FK
        varchar provider_name
        varchar model_name
        varchar prompt_version
        varchar request_hash
        integer input_tokens
        integer output_tokens
        integer latency_ms
        varchar status
        text response_summary
        timestamp created_at
    }

    INTEGRATION_LOGS {
        bigserial id PK
        uuid processing_job_id FK
        varchar target_system
        varchar operation_name
        varchar request_id
        varchar status
        integer http_status
        integer latency_ms
        jsonb request_payload
        jsonb response_payload
        varchar error_code
        text error_message
        timestamp created_at
    }

    MANUAL_REVIEW_QUEUE {
        bigserial id PK
        bigint question_draft_id FK
        varchar queue_reason
        integer priority
        uuid assigned_to
        varchar status
        timestamp created_at
        timestamp updated_at
    }

    PROCESSING_JOBS ||--o{ PROCESSING_JOB_STEPS : "possui"
    PROCESSING_JOBS ||--o{ UPLOADED_FILES : "recebe"
    PROCESSING_JOBS ||--o{ QUESTION_DRAFTS : "gera"
    PROCESSING_JOBS ||--o{ INTEGRATION_LOGS : "registra"
    QUESTION_DRAFTS ||--o{ QUESTION_SOURCES : "possui"
    QUESTION_DRAFTS ||--o{ DRAFT_FILTERS : "classifica"
    QUESTION_DRAFTS ||--o{ DRAFT_SUB_MATTERS : "classifica"
    QUESTION_DRAFTS ||--o{ AGENT_RUNS : "processado por"
    QUESTION_DRAFTS ||--o{ MANUAL_REVIEW_QUEUE : "enfileirado em"
    AGENT_RUNS ||--o{ LLM_INVOCATIONS : "dispara"
```

---

# 22. Dicionário Completo de Banco de Dados

## 22.1 `processing_jobs`

Representa a unidade principal de processamento assíncrono disparada por um upload.

| Campo | Tipo | Nulável | Índice | Descrição |
|-------|------|---------|--------|-----------|
| `id` | uuid | Não | PK | Identificador único do job |
| `job_key` | varchar(255) | Não | UK | Chave idempotente derivada de `input_hash` + `source_type` |
| `source_type` | varchar(50) | Não | — | Tipo da origem: `pdf_upload`, `manual`, `api` |
| `status` | varchar(50) | Não | IDX | Estado atual: `pending`, `processing`, `completed`, `failed`, `cancelled` |
| `progress_pct` | integer | Não | — | Percentual de progresso de 0 a 100 |
| `input_hash` | varchar(64) | Não | UK | SHA-256 do conteúdo de entrada para idempotência |
| `requested_by` | uuid | Sim | IDX | Usuário solicitante |
| `metadata` | jsonb | Sim | — | Metadados do processamento: versão do pipeline, prioridade, clientId |
| `started_at` | timestamp | Sim | — | Momento do início efetivo do processamento |
| `finished_at` | timestamp | Sim | — | Momento do fim efetivo |
| `created_at` | timestamp | Não | IDX | Criação do registro |
| `updated_at` | timestamp | Não | — | Última atualização |

**Índices:** `idx_processing_jobs_status`, `idx_processing_jobs_created_at`, `idx_processing_jobs_requested_by`

## 22.2 `processing_job_steps`

Detalhamento das etapas de um job de processamento.

| Campo | Tipo | Nulável | Índice | Descrição |
|-------|------|---------|--------|-----------|
| `id` | bigserial | Não | PK | Identificador interno |
| `processing_job_id` | uuid | Não | FK + IDX | Job pai |
| `step_name` | varchar(100) | Não | — | Nome da etapa: `extraction`, `classification`, etc. |
| `step_order` | integer | Não | — | Ordem sequencial no pipeline |
| `status` | varchar(50) | Não | IDX | Status: `pending`, `running`, `completed`, `failed`, `skipped` |
| `attempt` | integer | Não | — | Número da tentativa (começa em 1) |
| `error_code` | varchar(100) | Sim | — | Código de erro padronizado |
| `error_message` | text | Sim | — | Mensagem resumida do erro |
| `payload` | jsonb | Sim | — | Dados relevantes da etapa para debugging |
| `started_at` | timestamp | Sim | — | Início da etapa |
| `finished_at` | timestamp | Sim | — | Fim da etapa |
| `created_at` | timestamp | Não | — | Criação do registro |

## 22.3 `uploaded_files`

Arquivos recebidos com seus metadados técnicos.

| Campo | Tipo | Nulável | Índice | Descrição |
|-------|------|---------|--------|-----------|
| `id` | uuid | Não | PK | Identificador único do arquivo |
| `processing_job_id` | uuid | Não | FK + IDX | Job relacionado |
| `file_role` | varchar(50) | Não | — | Papel do arquivo: `proof` ou `answer_key` |
| `original_name` | varchar(500) | Não | — | Nome original do arquivo enviado |
| `storage_path` | varchar(1000) | Não | — | Caminho completo no storage |
| `mime_type` | varchar(100) | Não | — | MIME validado real do arquivo |
| `size_bytes` | bigint | Não | — | Tamanho em bytes |
| `checksum_sha256` | varchar(64) | Não | UK | Hash SHA-256 do conteúdo |
| `is_valid` | boolean | Não | — | Resultado da validação |
| `validation_errors` | jsonb | Sim | — | Erros de validação, se houver |
| `created_at` | timestamp | Não | IDX | Criação do registro |

## 22.4 `question_drafts`

Rascunho intermediário gerado pelo agente, antes da publicação.

| Campo | Tipo | Nulável | Índice | Descrição |
|-------|------|---------|--------|-----------|
| `id` | bigserial | Não | PK | Identificador do draft |
| `processing_job_id` | uuid | Não | FK + IDX | Job que gerou o draft |
| `source_question_number` | varchar(20) | Sim | — | Número da questão no PDF original |
| `title` | text | Não | — | Título gerado pelo agente |
| `description` | text | Não | — | Enunciado V/F gerado |
| `explanation` | text | Não | — | Gabarito comentado em HTML |
| `is_true` | boolean | Não | — | Resposta correta |
| `confidence_score` | numeric(5,2) | Não | IDX | Score de confiança de 0.00 a 1.00 |
| `validation_status` | varchar(50) | Não | IDX | Estado de validação: `valid`, `invalid`, `needs_review` |
| `review_status` | varchar(50) | Não | IDX | Estado editorial: `pending_review`, `approved`, `rejected`, `published` |
| `reviewed_by` | uuid | Sim | — | UUID do revisor |
| `review_reason` | text | Sim | — | Motivo de rejeição ou ajuste |
| `raw_payload` | jsonb | Não | — | Payload bruto consolidado de todos os agentes |
| `normalized_payload` | jsonb | Não | — | Payload normalizado para publicação |
| `created_at` | timestamp | Não | IDX | Criação |
| `updated_at` | timestamp | Não | — | Atualização |

**Índices:** `idx_question_drafts_validation_status`, `idx_question_drafts_review_status`, `idx_question_drafts_confidence_score`, `idx_question_drafts_created_at`

## 22.5 `question_sources`

Evidências e referências utilizadas para fundamentar a questão gerada.

| Campo | Tipo | Nulável | Índice | Descrição |
|-------|------|---------|--------|-----------|
| `id` | bigserial | Não | PK | Identificador |
| `question_draft_id` | bigint | Não | FK + IDX | Draft relacionado |
| `source_type` | varchar(50) | Não | — | Tipo: `legal_text`, `pgvector`, `web_search` |
| `source_ref` | varchar(500) | Não | — | Referência: `CF88-art5`, `lei-8112-art67`, URL |
| `title` | varchar(500) | Sim | — | Título amigável da fonte |
| `excerpt` | text | Sim | — | Trecho relevante utilizado |
| `rank_score` | numeric(8,4) | Sim | — | Score da busca semântica |
| `metadata` | jsonb | Sim | — | Dados adicionais: página, parágrafo, etc. |
| `created_at` | timestamp | Não | — | Criação |

## 22.6 `draft_filters`

Vínculos do draft com os filtros contextuais da plataforma.

| Campo | Tipo | Nulável | Índice | Descrição |
|-------|------|---------|--------|-----------|
| `id` | bigserial | Não | PK | Identificador |
| `question_draft_id` | bigint | Não | FK + IDX | Draft relacionado |
| `filter_id` | integer | Não | IDX | ID do filtro resolvido na base principal |
| `resolution_confidence` | numeric(5,2) | Sim | — | Confiança da resolução de 0.00 a 1.00 |
| `resolution_note` | text | Sim | — | Observação sobre a resolução |
| `created_at` | timestamp | Não | — | Criação |

## 22.7 `draft_sub_matters`

Vínculos do draft com a taxonomia pedagógica.

| Campo | Tipo | Nulável | Índice | Descrição |
|-------|------|---------|--------|-----------|
| `id` | bigserial | Não | PK | Identificador |
| `question_draft_id` | bigint | Não | FK + IDX | Draft relacionado |
| `sub_matter_id` | integer | Não | IDX | ID da submatéria na base principal |
| `position` | integer | Sim | — | Ordem de importância |
| `resolution_confidence` | numeric(5,2) | Sim | — | Confiança da resolução |
| `created_at` | timestamp | Não | — | Criação |

## 22.8 `agent_runs`

Execuções individuais de agentes sobre drafts ou jobs.

| Campo | Tipo | Nulável | Índice | Descrição |
|-------|------|---------|--------|-----------|
| `id` | uuid | Não | PK | Identificador |
| `processing_job_id` | uuid | Não | FK + IDX | Job relacionado |
| `question_draft_id` | bigint | Sim | FK + IDX | Draft relacionado, se aplicável |
| `agent_name` | varchar(100) | Não | IDX | Nome do agente: `pdf-extraction`, `classification`, etc. |
| `agent_version` | varchar(20) | Não | — | Versão do agente: `1.0.0`, `1.1.0` |
| `status` | varchar(50) | Não | IDX | Status: `running`, `completed`, `failed`, `timeout` |
| `attempt` | integer | Não | — | Número da tentativa |
| `input_snapshot` | jsonb | Sim | — | Snapshot da entrada (sanitizado) |
| `output_snapshot` | jsonb | Sim | — | Snapshot da saída (sanitizado) |
| `error_code` | varchar(100) | Sim | — | Código de erro padronizado |
| `error_message` | text | Sim | — | Mensagem de erro |
| `started_at` | timestamp | Sim | — | Início da execução |
| `finished_at` | timestamp | Sim | — | Fim da execução |
| `created_at` | timestamp | Não | IDX | Criação |

## 22.9 `llm_invocations`

Rastreamento de chamadas a modelos de IA.

| Campo | Tipo | Nulável | Índice | Descrição |
|-------|------|---------|--------|-----------|
| `id` | uuid | Não | PK | Identificador |
| `agent_run_id` | uuid | Não | FK + IDX | Execução que disparou a chamada |
| `provider_name` | varchar(50) | Não | IDX | Provider: `anthropic`, `openai`, `groq` |
| `model_name` | varchar(100) | Não | IDX | Modelo: `claude-3-5-sonnet`, `gpt-4o` |
| `prompt_version` | varchar(20) | Não | — | Versão do prompt: `v1.2.0` |
| `request_hash` | varchar(64) | Não | — | Hash SHA-256 da entrada |
| `input_tokens` | integer | Sim | — | Tokens de entrada |
| `output_tokens` | integer | Sim | — | Tokens de saída |
| `latency_ms` | integer | Sim | — | Latência em milissegundos |
| `status` | varchar(50) | Não | IDX | Status: `success`, `timeout`, `error`, `rate_limited` |
| `response_summary` | text | Sim | — | Resumo seguro e mascarado da resposta |
| `created_at` | timestamp | Não | IDX | Criação |

## 22.10 `integration_logs`

Registro de integrações com sistemas externos.

| Campo | Tipo | Nulável | Índice | Descrição |
|-------|------|---------|--------|-----------|
| `id` | bigserial | Não | PK | Identificador |
| `processing_job_id` | uuid | Não | FK + IDX | Job relacionado |
| `target_system` | varchar(50) | Não | IDX | Sistema destino: `mysql_principal`, `s3`, `llm_provider` |
| `operation_name` | varchar(100) | Não | — | Operação: `insert_question`, `insert_filters`, `upload_file` |
| `request_id` | varchar(100) | Sim | — | Correlation ID da requisição |
| `status` | varchar(50) | Não | IDX | Resultado: `success`, `failure`, `timeout` |
| `http_status` | integer | Sim | — | Código HTTP quando aplicável |
| `latency_ms` | integer | Sim | — | Latência em milissegundos |
| `request_payload` | jsonb | Sim | — | Payload sanitizado e mascarado |
| `response_payload` | jsonb | Sim | — | Payload sanitizado e mascarado |
| `error_code` | varchar(100) | Sim | — | Código de erro padronizado |
| `error_message` | text | Sim | — | Mensagem de erro |
| `created_at` | timestamp | Não | IDX | Criação |

## 22.11 `manual_review_queue`

Fila de revisão humana para drafts que requerem atenção editorial.

| Campo | Tipo | Nulável | Índice | Descrição |
|-------|------|---------|--------|-----------|
| `id` | bigserial | Não | PK | Identificador |
| `question_draft_id` | bigint | Não | FK + UK | Draft relacionado (único por draft) |
| `queue_reason` | varchar(100) | Não | IDX | Motivo: `low_confidence`, `ambiguous_classification`, `validation_error`, `manual_request` |
| `priority` | integer | Não | IDX | Prioridade: 1=alta, 2=média, 3=baixa |
| `assigned_to` | uuid | Sim | IDX | UUID do revisor atribuído |
| `status` | varchar(50) | Não | IDX | Status: `pending`, `in_review`, `completed`, `skipped` |
| `created_at` | timestamp | Não | IDX | Criação |
| `updated_at` | timestamp | Não | — | Atualização |

---

# 23. Camadas da Aplicação

## 23.1 Domain

Camada mais interna e estável do sistema. Contém:

- **Entities** — representações das entidades de negócio com regras próprias;
- **Value Objects** — objetos imutáveis que encapsulam conceitos como `JobKey`, `ConfidenceScore`, `Checksum`;
- **Enums** — vocabulário compartilhado de estados e categorias;
- **Domain Errors** — erros específicos do domínio, tipados e semânticos;
- **Domain Rules** — regras puras de negócio verificáveis e testáveis;
- **Domain Services** — lógica de negócio que não cabe em uma entidade única.

Esta camada não conhece NestJS, banco de dados, Redis ou qualquer provider externo.

## 23.2 Application

Camada que orquestra o domínio com o mundo externo. Contém:

- **Use Cases** — casos de uso atômicos e focados em um objetivo;
- **Contracts** — interfaces de repositórios, gateways e serviços externos;
- **DTOs** — objetos de transferência de dados internos;
- **Mappers** — tradutores entre camadas;
- **Policies** — políticas de timeout, retry, circuit breaker, fallback e idempotência.

Esta camada conhece os contratos, mas não as implementações concretas.

## 23.3 Infrastructure

Camada de implementações técnicas concretas. Contém:

- **Repositories** — implementações PostgreSQL e MySQL;
- **Gateways** — Redis, BullMQ, LLM Providers, Storage, Web Search;
- **Adapters** — Logger (Pino), Tracer (OpenTelemetry), Metrics (Prometheus);
- **Resilience** — Circuit Breaker, Timeout Runner, Retry Runner.

Esta camada implementa os contratos definidos pela Application.

## 23.4 Interfaces

Camada de entrada da aplicação. Contém:

- **Controllers** — controllers REST versionados;
- **Workers** — workers BullMQ para processamento assíncrono;
- **DTOs de transporte** — validação de entrada e forma da resposta;
- **Guards** — autenticação JWT e autorização RBAC;
- **Interceptors** — logging, versionamento de API, transformação de resposta.

---

# 24. Arquitetura de Pastas e Arquivos

```
question-ai-service/
├── .github/
│   ├── workflows/
│   │   ├── ci.yml
│   │   ├── cd.yml
│   │   ├── lint.yml
│   │   ├── tests.yml
│   │   └── security-scan.yml
│   ├── pull_request_template.md
│   └── CODEOWNERS
│
├── docs/
│   ├── architecture/
│   │   ├── agent-architecture.md
│   │   ├── database-dictionary.md
│   │   ├── observability.md
│   │   ├── resilience.md
│   │   ├── api-contracts.md
│   │   ├── versioning-strategy.md
│   │   └── folder-structure.md
│   ├── adr/
│   │   ├── ADR-001-clean-architecture.md
│   │   ├── ADR-002-staging-before-publication.md
│   │   ├── ADR-003-filters-as-context.md
│   │   ├── ADR-004-idempotent-jobs.md
│   │   ├── ADR-005-uri-versioning.md
│   │   ├── ADR-006-jwt-rbac-security.md
│   │   └── ADR-007-pgvector-for-legal-search.md
│   └── runbooks/
│       ├── incident-llm-provider-down.md
│       ├── incident-mysql-unavailable.md
│       ├── incident-queue-backlog.md
│       ├── incident-circuit-breaker-open.md
│       └── manual-review-playbook.md
│
├── src/
│   ├── main.ts
│   ├── app.module.ts
│   │
│   ├── bootstrap/
│   │   ├── app.bootstrap.ts
│   │   ├── config.bootstrap.ts
│   │   ├── logger.bootstrap.ts
│   │   ├── queue.bootstrap.ts
│   │   ├── tracing.bootstrap.ts
│   │   ├── security.bootstrap.ts
│   │   └── validation.bootstrap.ts
│   │
│   ├── config/
│   │   ├── app.config.ts
│   │   ├── database.config.ts
│   │   ├── redis.config.ts
│   │   ├── queue.config.ts
│   │   ├── llm.config.ts
│   │   ├── observability.config.ts
│   │   ├── storage.config.ts
│   │   ├── security.config.ts
│   │   └── feature-flags.config.ts
│   │
│   ├── core/
│   │   │
│   │   ├── domain/
│   │   │   ├── entities/
│   │   │   │   ├── processing-job.entity.ts
│   │   │   │   ├── processing-job-step.entity.ts
│   │   │   │   ├── uploaded-file.entity.ts
│   │   │   │   ├── question-draft.entity.ts
│   │   │   │   ├── question-source.entity.ts
│   │   │   │   ├── agent-run.entity.ts
│   │   │   │   ├── llm-invocation.entity.ts
│   │   │   │   └── manual-review-item.entity.ts
│   │   │   │
│   │   │   ├── value-objects/
│   │   │   │   ├── job-key.vo.ts
│   │   │   │   ├── checksum.vo.ts
│   │   │   │   ├── confidence-score.vo.ts
│   │   │   │   ├── normalized-text.vo.ts
│   │   │   │   ├── source-reference.vo.ts
│   │   │   │   ├── filter-resolution.vo.ts
│   │   │   │   └── taxonomy-resolution.vo.ts
│   │   │   │
│   │   │   ├── enums/
│   │   │   │   ├── processing-job-status.enum.ts
│   │   │   │   ├── processing-step-status.enum.ts
│   │   │   │   ├── review-status.enum.ts
│   │   │   │   ├── validation-status.enum.ts
│   │   │   │   ├── agent-run-status.enum.ts
│   │   │   │   ├── source-type.enum.ts
│   │   │   │   ├── file-role.enum.ts
│   │   │   │   ├── fallback-strategy.enum.ts
│   │   │   │   ├── integration-target.enum.ts
│   │   │   │   ├── queue-reason.enum.ts
│   │   │   │   └── error-category.enum.ts
│   │   │   │
│   │   │   ├── constants/
│   │   │   │   ├── system-limits.constants.ts
│   │   │   │   ├── timeout-policy.constants.ts
│   │   │   │   ├── retry-policy.constants.ts
│   │   │   │   └── metric-names.constants.ts
│   │   │   │
│   │   │   ├── errors/
│   │   │   │   ├── domain-error.base.ts
│   │   │   │   ├── validation-error.ts
│   │   │   │   ├── retryable-error.ts
│   │   │   │   ├── non-retryable-error.ts
│   │   │   │   ├── timeout-error.ts
│   │   │   │   ├── integration-error.ts
│   │   │   │   ├── parsing-error.ts
│   │   │   │   └── classification-error.ts
│   │   │   │
│   │   │   ├── rules/
│   │   │   │   ├── question-draft-completeness.rule.ts
│   │   │   │   ├── publication-eligibility.rule.ts
│   │   │   │   ├── confidence-threshold.rule.ts
│   │   │   │   └── manual-review-required.rule.ts
│   │   │   │
│   │   │   └── services/
│   │   │       ├── idempotency-policy.service.ts
│   │   │       ├── publication-policy.service.ts
│   │   │       ├── review-policy.service.ts
│   │   │       └── fallback-policy.service.ts
│   │   │
│   │   ├── application/
│   │   │   ├── contracts/
│   │   │   │   ├── repositories/
│   │   │   │   │   ├── processing-job.repository.contract.ts
│   │   │   │   │   ├── question-draft.repository.contract.ts
│   │   │   │   │   ├── question-source.repository.contract.ts
│   │   │   │   │   ├── uploaded-file.repository.contract.ts
│   │   │   │   │   ├── agent-run.repository.contract.ts
│   │   │   │   │   ├── llm-invocation.repository.contract.ts
│   │   │   │   │   └── manual-review.repository.contract.ts
│   │   │   │   │
│   │   │   │   ├── gateways/
│   │   │   │   │   ├── llm.gateway.contract.ts
│   │   │   │   │   ├── vector-search.gateway.contract.ts
│   │   │   │   │   ├── mysql-question.gateway.contract.ts
│   │   │   │   │   ├── storage.gateway.contract.ts
│   │   │   │   │   ├── queue.gateway.contract.ts
│   │   │   │   │   ├── cache.gateway.contract.ts
│   │   │   │   │   ├── web-search.gateway.contract.ts
│   │   │   │   │   └── circuit-breaker.gateway.contract.ts
│   │   │   │   │
│   │   │   │   └── services/
│   │   │   │       ├── agent.contract.ts
│   │   │   │       ├── orchestrator.contract.ts
│   │   │   │       ├── logger.contract.ts
│   │   │   │       ├── tracer.contract.ts
│   │   │   │       ├── metrics.contract.ts
│   │   │   │       └── clock.contract.ts
│   │   │   │
│   │   │   ├── dto/
│   │   │   │   ├── common/
│   │   │   │   │   ├── pagination.dto.ts
│   │   │   │   │   ├── metadata.dto.ts
│   │   │   │   │   └── trace-context.dto.ts
│   │   │   │   ├── jobs/
│   │   │   │   │   ├── create-processing-job.input.ts
│   │   │   │   │   ├── create-processing-job.output.ts
│   │   │   │   │   ├── get-job-status.output.ts
│   │   │   │   │   └── retry-processing-job.input.ts
│   │   │   │   ├── drafts/
│   │   │   │   │   ├── create-question-draft.input.ts
│   │   │   │   │   ├── update-question-draft.input.ts
│   │   │   │   │   ├── publish-question-draft.input.ts
│   │   │   │   │   └── question-draft.output.ts
│   │   │   │   └── agents/
│   │   │   │       ├── extraction.payload.ts
│   │   │   │       ├── classification.payload.ts
│   │   │   │       ├── resolution.payload.ts
│   │   │   │       ├── search.payload.ts
│   │   │   │       ├── adaptation.payload.ts
│   │   │   │       ├── answer-key.payload.ts
│   │   │   │       └── validation.payload.ts
│   │   │   │
│   │   │   ├── use-cases/
│   │   │   │   ├── jobs/
│   │   │   │   │   ├── create-processing-job.use-case.ts
│   │   │   │   │   ├── start-processing-job.use-case.ts
│   │   │   │   │   ├── retry-processing-job.use-case.ts
│   │   │   │   │   ├── cancel-processing-job.use-case.ts
│   │   │   │   │   └── get-processing-job-status.use-case.ts
│   │   │   │   ├── uploads/
│   │   │   │   │   ├── validate-upload.use-case.ts
│   │   │   │   │   ├── store-uploaded-files.use-case.ts
│   │   │   │   │   └── register-upload-metadata.use-case.ts
│   │   │   │   ├── pipeline/
│   │   │   │   │   ├── orchestrate-question-generation.use-case.ts
│   │   │   │   │   ├── run-extraction-step.use-case.ts
│   │   │   │   │   ├── run-classification-step.use-case.ts
│   │   │   │   │   ├── run-resolution-step.use-case.ts
│   │   │   │   │   ├── run-search-step.use-case.ts
│   │   │   │   │   ├── run-adaptation-step.use-case.ts
│   │   │   │   │   ├── run-answer-key-step.use-case.ts
│   │   │   │   │   └── run-validation-step.use-case.ts
│   │   │   │   ├── drafts/
│   │   │   │   │   ├── create-question-draft.use-case.ts
│   │   │   │   │   ├── update-question-draft.use-case.ts
│   │   │   │   │   ├── queue-manual-review.use-case.ts
│   │   │   │   │   ├── approve-question-draft.use-case.ts
│   │   │   │   │   ├── reject-question-draft.use-case.ts
│   │   │   │   │   └── publish-question-draft.use-case.ts
│   │   │   │   ├── observability/
│   │   │   │   │   ├── register-agent-run.use-case.ts
│   │   │   │   │   ├── register-llm-invocation.use-case.ts
│   │   │   │   │   ├── register-integration-log.use-case.ts
│   │   │   │   │   └── compute-job-metrics.use-case.ts
│   │   │   │   └── maintenance/
│   │   │   │       ├── requeue-stuck-jobs.use-case.ts
│   │   │   │       ├── cleanup-expired-files.use-case.ts
│   │   │   │       └── cleanup-failed-locks.use-case.ts
│   │   │   │
│   │   │   ├── mappers/
│   │   │   │   ├── question-draft.mapper.ts
│   │   │   │   ├── processing-job.mapper.ts
│   │   │   │   ├── publication.mapper.ts
│   │   │   │   └── filter-resolution.mapper.ts
│   │   │   │
│   │   │   └── policies/
│   │   │       ├── timeout.policy.ts
│   │   │       ├── retry.policy.ts
│   │   │       ├── circuit-breaker.policy.ts
│   │   │       ├── fallback.policy.ts
│   │   │       └── idempotency.policy.ts
│   │   │
│   │   └── shared/
│   │       ├── result/
│   │       │   ├── result.ts
│   │       │   ├── success.ts
│   │       │   ├── failure.ts
│   │       │   └── error-result.ts
│   │       ├── types/
│   │       │   ├── nullable.type.ts
│   │       │   ├── optional.type.ts
│   │       │   ├── primitive.type.ts
│   │       │   └── json-value.type.ts
│   │       ├── helpers/
│   │       │   ├── object.helper.ts
│   │       │   ├── array.helper.ts
│   │       │   ├── string.helper.ts
│   │       │   ├── date.helper.ts
│   │       │   ├── number.helper.ts
│   │       │   ├── hash.helper.ts
│   │       │   ├── mask.helper.ts
│   │       │   ├── html.helper.ts
│   │       │   ├── file.helper.ts
│   │       │   └── retry.helper.ts
│   │       ├── normalizers/
│   │       │   ├── global/
│   │       │   │   ├── normalize-whitespace.ts
│   │       │   │   ├── normalize-line-breaks.ts
│   │       │   │   ├── normalize-unicode.ts
│   │       │   │   ├── normalize-punctuation.ts
│   │       │   │   ├── normalize-html-snippet.ts
│   │       │   │   ├── normalize-boolean.ts
│   │       │   │   ├── normalize-year.ts
│   │       │   │   ├── normalize-bank-name.ts
│   │       │   │   ├── normalize-organization-name.ts
│   │       │   │   ├── normalize-article-reference.ts
│   │       │   │   ├── normalize-law-name.ts
│   │       │   │   └── normalize-question-title.ts
│   │       │   ├── payload/
│   │       │   │   ├── normalize-extraction-payload.ts
│   │       │   │   ├── normalize-classification-payload.ts
│   │       │   │   ├── normalize-resolution-payload.ts
│   │       │   │   ├── normalize-search-payload.ts
│   │       │   │   ├── normalize-adaptation-payload.ts
│   │       │   │   ├── normalize-answer-key-payload.ts
│   │       │   │   └── normalize-validation-payload.ts
│   │       │   └── mappers/
│   │       │       ├── map-raw-question-to-draft.ts
│   │       │       └── map-draft-to-publication-payload.ts
│   │       ├── validation/
│   │       │   ├── schemas/
│   │       │   │   ├── create-job.schema.ts
│   │       │   │   ├── extraction-payload.schema.ts
│   │       │   │   ├── classification-payload.schema.ts
│   │       │   │   ├── resolution-payload.schema.ts
│   │       │   │   ├── search-payload.schema.ts
│   │       │   │   ├── adaptation-payload.schema.ts
│   │       │   │   ├── answer-key-payload.schema.ts
│   │       │   │   ├── validation-payload.schema.ts
│   │       │   │   └── publication-payload.schema.ts
│   │       │   └── validators/
│   │       │       ├── file.validator.ts
│   │       │       ├── question-draft.validator.ts
│   │       │       ├── html-output.validator.ts
│   │       │       └── filter-resolution.validator.ts
│   │       └── telemetry/
│   │           ├── events/
│   │           │   ├── job-events.ts
│   │           │   ├── agent-events.ts
│   │           │   ├── draft-events.ts
│   │           │   └── integration-events.ts
│   │           └── names/
│   │               ├── log-event-names.ts
│   │               ├── metric-names.ts
│   │               └── trace-names.ts
│   │
│   ├── modules/
│   │   ├── health/
│   │   │   ├── interface/
│   │   │   │   └── controllers/
│   │   │   │       └── health.controller.ts
│   │   │   └── health.module.ts
│   │   │
│   │   ├── uploads/
│   │   │   ├── interface/
│   │   │   │   ├── controllers/
│   │   │   │   │   └── uploads.controller.ts
│   │   │   │   └── dto/
│   │   │   │       ├── upload-files.request.ts
│   │   │   │       └── upload-files.response.ts
│   │   │   ├── application/
│   │   │   │   └── services/
│   │   │   │       └── upload-orchestration.service.ts
│   │   │   └── uploads.module.ts
│   │   │
│   │   ├── jobs/
│   │   │   ├── interface/
│   │   │   │   ├── controllers/
│   │   │   │   │   └── processing-jobs.controller.ts
│   │   │   │   └── dto/
│   │   │   │       ├── create-job.request.ts
│   │   │   │       ├── retry-job.request.ts
│   │   │   │       └── job-status.response.ts
│   │   │   ├── application/
│   │   │   │   └── services/
│   │   │   │       └── job-status-query.service.ts
│   │   │   └── jobs.module.ts
│   │   │
│   │   ├── pipeline/
│   │   │   ├── application/
│   │   │   │   ├── orchestrators/
│   │   │   │   │   ├── question-generation.orchestrator.ts
│   │   │   │   │   └── publication.orchestrator.ts
│   │   │   │   └── services/
│   │   │   │       └── pipeline-step-runner.service.ts
│   │   │   ├── workers/
│   │   │   │   ├── process-job.worker.ts
│   │   │   │   ├── retry-job.worker.ts
│   │   │   │   └── publish-draft.worker.ts
│   │   │   └── pipeline.module.ts
│   │   │
│   │   ├── agents/
│   │   │   ├── extraction/
│   │   │   │   ├── application/
│   │   │   │   │   ├── extraction-agent.service.ts
│   │   │   │   │   ├── extraction-runner.service.ts
│   │   │   │   │   └── extraction-response.builder.ts
│   │   │   │   └── extraction.module.ts
│   │   │   ├── classification/
│   │   │   │   ├── application/
│   │   │   │   │   ├── classification-agent.service.ts
│   │   │   │   │   ├── classification-runner.service.ts
│   │   │   │   │   └── classification-response.builder.ts
│   │   │   │   └── classification.module.ts
│   │   │   ├── resolution/
│   │   │   │   ├── application/
│   │   │   │   │   ├── resolution-agent.service.ts
│   │   │   │   │   ├── resolution-runner.service.ts
│   │   │   │   │   └── resolution-response.builder.ts
│   │   │   │   └── resolution.module.ts
│   │   │   ├── search/
│   │   │   │   ├── application/
│   │   │   │   │   ├── search-agent.service.ts
│   │   │   │   │   ├── search-runner.service.ts
│   │   │   │   │   └── search-response.builder.ts
│   │   │   │   └── search.module.ts
│   │   │   ├── adaptation/
│   │   │   │   ├── application/
│   │   │   │   │   ├── adaptation-agent.service.ts
│   │   │   │   │   ├── adaptation-runner.service.ts
│   │   │   │   │   └── adaptation-response.builder.ts
│   │   │   │   └── adaptation.module.ts
│   │   │   ├── answer-key/
│   │   │   │   ├── application/
│   │   │   │   │   ├── answer-key-agent.service.ts
│   │   │   │   │   ├── answer-key-runner.service.ts
│   │   │   │   │   └── answer-key-response.builder.ts
│   │   │   │   └── answer-key.module.ts
│   │   │   ├── validation/
│   │   │   │   ├── application/
│   │   │   │   │   ├── validation-agent.service.ts
│   │   │   │   │   ├── validation-runner.service.ts
│   │   │   │   │   └── validation-response.builder.ts
│   │   │   │   └── validation.module.ts
│   │   │   └── agents.module.ts
│   │   │
│   │   ├── drafts/
│   │   │   ├── interface/
│   │   │   │   ├── controllers/
│   │   │   │   │   └── question-drafts.controller.ts
│   │   │   │   └── dto/
│   │   │   │       ├── approve-draft.request.ts
│   │   │   │       ├── reject-draft.request.ts
│   │   │   │       ├── update-draft.request.ts
│   │   │   │       └── draft.response.ts
│   │   │   ├── application/
│   │   │   │   └── services/
│   │   │   │       ├── draft-review.service.ts
│   │   │   │       └── draft-publication.service.ts
│   │   │   └── drafts.module.ts
│   │   │
│   │   ├── observability/
│   │   │   ├── interface/
│   │   │   │   └── controllers/
│   │   │   │       └── observability.controller.ts
│   │   │   ├── application/
│   │   │   │   └── services/
│   │   │   │       ├── metrics-query.service.ts
│   │   │   │       └── traces-query.service.ts
│   │   │   └── observability.module.ts
│   │   │
│   │   └── maintenance/
│   │       ├── application/
│   │       │   └── cron/
│   │       │       ├── cleanup-files.cron.ts
│   │       │       ├── requeue-jobs.cron.ts
│   │       │       └── health-audit.cron.ts
│   │       └── maintenance.module.ts
│   │
│   ├── infra/
│   │   ├── database/
│   │   │   ├── postgres/
│   │   │   │   ├── entities/
│   │   │   │   ├── repositories/
│   │   │   │   │   ├── processing-job.postgres-repository.ts
│   │   │   │   │   ├── question-draft.postgres-repository.ts
│   │   │   │   │   ├── question-source.postgres-repository.ts
│   │   │   │   │   ├── uploaded-file.postgres-repository.ts
│   │   │   │   │   ├── agent-run.postgres-repository.ts
│   │   │   │   │   ├── llm-invocation.postgres-repository.ts
│   │   │   │   │   └── manual-review.postgres-repository.ts
│   │   │   │   ├── migrations/
│   │   │   │   └── postgres.module.ts
│   │   │   ├── mysql/
│   │   │   │   ├── gateways/
│   │   │   │   │   ├── mysql-question.gateway.ts
│   │   │   │   │   ├── mysql-filter.gateway.ts
│   │   │   │   │   ├── mysql-taxonomy.gateway.ts
│   │   │   │   │   └── mysql-publication.gateway.ts
│   │   │   │   ├── queries/
│   │   │   │   │   ├── find-filter-by-type-and-name.sql.ts
│   │   │   │   │   ├── find-sub-matter-by-name.sql.ts
│   │   │   │   │   ├── insert-question.sql.ts
│   │   │   │   │   ├── insert-question-filters.sql.ts
│   │   │   │   │   └── insert-question-sub-matters.sql.ts
│   │   │   │   └── mysql.module.ts
│   │   │   └── database.module.ts
│   │   │
│   │   ├── cache/
│   │   │   ├── redis-cache.gateway.ts
│   │   │   ├── redis-lock.gateway.ts
│   │   │   └── cache.module.ts
│   │   │
│   │   ├── queue/
│   │   │   ├── bullmq.gateway.ts
│   │   │   ├── queue-names.ts
│   │   │   ├── job-options.factory.ts
│   │   │   └── queue.module.ts
│   │   │
│   │   ├── llm/
│   │   │   ├── providers/
│   │   │   │   ├── primary-llm.gateway.ts
│   │   │   │   ├── fallback-llm.gateway.ts
│   │   │   │   └── provider-router.gateway.ts
│   │   │   ├── prompts/
│   │   │   │   ├── extraction/
│   │   │   │   │   ├── extraction.prompt.ts
│   │   │   │   │   └── extraction.prompt.version.ts
│   │   │   │   ├── classification/
│   │   │   │   │   ├── classification.prompt.ts
│   │   │   │   │   └── classification.prompt.version.ts
│   │   │   │   ├── adaptation/
│   │   │   │   │   ├── adaptation.prompt.ts
│   │   │   │   │   └── adaptation.prompt.version.ts
│   │   │   │   ├── answer-key/
│   │   │   │   │   ├── answer-key.prompt.ts
│   │   │   │   │   └── answer-key.prompt.version.ts
│   │   │   │   └── shared/
│   │   │   │       ├── system-rules.prompt.ts
│   │   │   │       └── output-schema.prompt.ts
│   │   │   ├── parsers/
│   │   │   │   ├── llm-json-response.parser.ts
│   │   │   │   ├── llm-html-response.parser.ts
│   │   │   │   └── llm-error.parser.ts
│   │   │   └── llm.module.ts
│   │   │
│   │   ├── vector-search/
│   │   │   ├── pgvector.gateway.ts
│   │   │   ├── chunkers/
│   │   │   │   ├── legal-document.chunker.ts
│   │   │   │   └── study-material.chunker.ts
│   │   │   └── vector-search.module.ts
│   │   │
│   │   ├── storage/
│   │   │   ├── local-storage.gateway.ts
│   │   │   ├── s3-storage.gateway.ts
│   │   │   └── storage.module.ts
│   │   │
│   │   ├── telemetry/
│   │   │   ├── logger/
│   │   │   │   ├── pino-logger.adapter.ts
│   │   │   │   ├── logger-context.factory.ts
│   │   │   │   └── logger.module.ts
│   │   │   ├── tracing/
│   │   │   │   ├── opentelemetry-tracer.adapter.ts
│   │   │   │   └── tracing.module.ts
│   │   │   ├── metrics/
│   │   │   │   ├── prometheus-metrics.adapter.ts
│   │   │   │   ├── metrics.registry.ts
│   │   │   │   └── metrics.module.ts
│   │   │   └── telemetry.module.ts
│   │   │
│   │   ├── resilience/
│   │   │   ├── circuit-breaker/
│   │   │   │   ├── circuit-breaker.service.ts
│   │   │   │   ├── circuit-breaker.registry.ts
│   │   │   │   └── circuit-breaker.module.ts
│   │   │   ├── timeout/
│   │   │   │   ├── timeout-runner.service.ts
│   │   │   │   └── timeout.module.ts
│   │   │   ├── retry/
│   │   │   │   ├── retry-runner.service.ts
│   │   │   │   └── retry.module.ts
│   │   │   └── resilience.module.ts
│   │   │
│   │   ├── security/
│   │   │   ├── guards/
│   │   │   │   ├── jwt-auth.guard.ts
│   │   │   │   └── roles.guard.ts
│   │   │   ├── decorators/
│   │   │   │   ├── roles.decorator.ts
│   │   │   │   └── current-user.decorator.ts
│   │   │   ├── strategies/
│   │   │   │   └── jwt.strategy.ts
│   │   │   └── security.module.ts
│   │   │
│   │   └── web-search/
│   │       ├── web-search.gateway.ts
│   │       └── web-search.module.ts
│   │
│   └── test-support/
│       ├── factories/
│       │   ├── processing-job.factory.ts
│       │   ├── question-draft.factory.ts
│       │   ├── agent-run.factory.ts
│       │   └── llm-response.factory.ts
│       ├── fixtures/
│       │   ├── pdf/
│       │   ├── payloads/
│       │   └── prompts/
│       ├── mocks/
│       │   ├── llm.gateway.mock.ts
│       │   ├── queue.gateway.mock.ts
│       │   ├── cache.gateway.mock.ts
│       │   └── mysql.gateway.mock.ts
│       └── utils/
│           ├── test-app.builder.ts
│           ├── integration-db.helper.ts
│           └── queue-test.helper.ts
│
├── test/
│   ├── unit/
│   │   ├── core/
│   │   ├── modules/
│   │   └── infra/
│   ├── integration/
│   │   ├── database/
│   │   ├── queue/
│   │   ├── cache/
│   │   ├── llm/
│   │   └── vector-search/
│   ├── e2e/
│   │   ├── upload-to-draft.e2e-spec.ts
│   │   ├── draft-publication.e2e-spec.ts
│   │   └── retry-idempotency.e2e-spec.ts
│   ├── contract/
│   │   ├── agents/
│   │   └── gateways/
│   └── resilience/
│       ├── timeout.spec.ts
│       ├── retry.spec.ts
│       ├── circuit-breaker.spec.ts
│       └── fallback.spec.ts
│
├── scripts/
│   ├── seed-legal-base.ts
│   ├── reindex-vectors.ts
│   ├── requeue-failed-jobs.ts
│   └── cleanup-temp-files.ts
│
├── .env.example
├── package.json
├── tsconfig.json
├── nest-cli.json
├── eslint.config.js
├── prettier.config.js
└── README.md
```

---

# 25. Organização por Responsabilidade

## 25.1 `core/domain`

Camada mais estável do sistema. Contém o que não muda com frameworks ou infraestrutura:

- entidades com identidade e regras próprias;
- value objects imutáveis e sem identidade;
- enums que representam vocabulário operacional;
- erros de domínio tipados e semânticos;
- regras de negócio puras e testáveis;
- serviços de domínio para lógica que cruza entidades.

Nada nessa camada deve conhecer NestJS, Redis, banco de dados, HTTP ou qualquer provider externo.

## 25.2 `core/application`

Camada que coordena o domínio com o mundo externo por meio de contratos:

- casos de uso atômicos com responsabilidade única;
- contratos de repositórios, gateways e serviços (interfaces);
- DTOs de transporte interno entre camadas;
- mapeadores que traduzem entre domínio e infraestrutura;
- políticas de execução como timeout, retry, circuit breaker, fallback e idempotência.

## 25.3 `core/shared`

Recursos reutilizáveis sem regra de negócio específica:

- helpers puros e genéricos para string, data, hash, array e arquivo;
- normalizadores globais determinísticos;
- validadores e schemas de payload;
- tipos utilitários;
- abstrações de `Result<T, E>` para tratamento funcional de erros;
- nomes padronizados de eventos, métricas e traces.

## 25.4 `modules`

Contextos funcionais da aplicação, organizados por capacidade de negócio:

- `health` — verificação de saúde do serviço;
- `uploads` — recebimento e validação de PDFs;
- `jobs` — criação e consulta de jobs;
- `pipeline` — orquestração e workers assíncronos;
- `agents` — cada agente em seu próprio submodulo;
- `drafts` — revisão e publicação de rascunhos;
- `observability` — métricas e traces;
- `maintenance` — crons de limpeza e requeuing.

## 25.5 `infra`

Implementações concretas dos contratos definidos na application:

- repositórios PostgreSQL e MySQL;
- gateways Redis, BullMQ, LLM, Storage, PGVector e Web Search;
- adapters de telemetria: logger Pino, tracer OpenTelemetry, métricas Prometheus;
- módulos de resiliência: circuit breaker, timeout runner, retry runner;
- módulo de segurança: guards, strategies, decorators.

---

# 26. Helpers, Normalizadores e Utilitários Globais

## 26.1 Helpers globais permitidos

São funções puras e genéricas sem conhecimento de negócio:

- `hash.helper.ts` — cálculo de SHA-256 e MD5 de conteúdo;
- `mask.helper.ts` — mascaramento de emails, CPFs e tokens em logs;
- `string.helper.ts` — truncamento, slugificação, capitalização, remoção de acentos;
- `date.helper.ts` — formatação, comparação e cálculo de diferença entre datas;
- `number.helper.ts` — arredondamento, formatação e faixas;
- `array.helper.ts` — chunking, deduplicação, flatten e ordenação;
- `object.helper.ts` — deep merge, deep clone, pick e omit;
- `html.helper.ts` — sanitização de HTML curto e extração de texto puro;
- `file.helper.ts` — cálculo de checksum de arquivo, leitura de MIME real;
- `retry.helper.ts` — execução com backoff exponencial e jitter sem estado.

## 26.2 Normalizadores globais permitidos

São funções puras, determinísticas e sem estado:

- `normalize-whitespace.ts` — trim e colapso de múltiplos espaços;
- `normalize-line-breaks.ts` — unificação de quebras de linha;
- `normalize-unicode.ts` — normalização NFD/NFC, remoção de caracteres especiais;
- `normalize-punctuation.ts` — padronização de aspas, hífens e reticências;
- `normalize-html-snippet.ts` — sanitização de HTML do gabarito;
- `normalize-boolean.ts` — conversão de strings como `"sim"`, `"V"`, `"true"` para boolean;
- `normalize-year.ts` — extração de ano de strings como `"2024"`, `"2024/2025"`;
- `normalize-bank-name.ts` — padronização de nomes de banca: `"FGV"`, `"fgv"`, `"Fundação Getulio Vargas"`;
- `normalize-organization-name.ts` — normalização de órgãos públicos;
- `normalize-article-reference.ts` — padronização de referências como `"art. 5°"`, `"Art 5"`, `"artigo 5"`;
- `normalize-law-name.ts` — padronização de nomes de lei: `"CF/88"`, `"CF-88"`, `"Constituição Federal"`;
- `normalize-question-title.ts` — limpeza e formatação do título da questão.

## 26.3 O que não deve virar helper ou normalizador global

Pertencem ao domínio ou à camada de aplicação, não a utilitários genéricos:

- regras de publicação e elegibilidade;
- critérios para encaminhar para revisão manual;
- lógica de classificação e resolução de IDs;
- decisão de fallback entre providers;
- cálculo de score de confiança.

## 26.4 Convenções de nomenclatura

| Tipo | Onde vive | Característica |
|------|-----------|----------------|
| Helper global | `core/shared/helpers/` | Função pura, sem estado, sem regra de negócio |
| Normalizador global | `core/shared/normalizers/global/` | Função pura, determinística, sem estado |
| Normalizador de payload | `core/shared/normalizers/payload/` | Transforma payload de agente em forma canônica |
| Policy | `core/application/policies/` | Regra orquestradora com dependências |
| Domain service | `core/domain/services/` | Lógica de negócio que cruza entidades |
| Adapter | `infra/` | Implementação técnica de um contrato |

---

# 27. Contratos, DTOs, Enums e Tipos

## 27.1 Contratos de gateways

Todos os componentes externos são acessados exclusivamente por interfaces:

- `llm.gateway.contract.ts` — invoke, stream, getAvailableModels;
- `vector-search.gateway.contract.ts` — search, upsert, delete;
- `mysql-question.gateway.contract.ts` — findFilterByTypeAndName, findSubMatterByName, insertQuestion;
- `storage.gateway.contract.ts` — upload, download, delete, generatePresignedUrl;
- `queue.gateway.contract.ts` — enqueue, dequeue, getJobStatus;
- `cache.gateway.contract.ts` — get, set, del, ttl, lock, unlock;
- `web-search.gateway.contract.ts` — search;
- `circuit-breaker.gateway.contract.ts` — execute, getState, reset.

## 27.2 Contratos de repositórios

- `processing-job.repository.contract.ts` — findById, findByJobKey, save, updateStatus;
- `question-draft.repository.contract.ts` — findById, findByJobId, save, updateReviewStatus;
- `agent-run.repository.contract.ts` — save, findByDraftId;
- `llm-invocation.repository.contract.ts` — save, findByAgentRunId;
- `manual-review.repository.contract.ts` — enqueue, findPending, assign, complete.

## 27.3 Enums centrais

```typescript
// ProcessingJobStatus
enum ProcessingJobStatus {
  PENDING = 'pending',
  PROCESSING = 'processing',
  COMPLETED = 'completed',
  FAILED = 'failed',
  CANCELLED = 'cancelled',
}

// ValidationStatus
enum ValidationStatus {
  VALID = 'valid',
  INVALID = 'invalid',
  NEEDS_REVIEW = 'needs_review',
}

// ReviewStatus
enum ReviewStatus {
  PENDING_REVIEW = 'pending_review',
  APPROVED = 'approved',
  REJECTED = 'rejected',
  PUBLISHED = 'published',
}

// AgentRunStatus
enum AgentRunStatus {
  RUNNING = 'running',
  COMPLETED = 'completed',
  FAILED = 'failed',
  TIMEOUT = 'timeout',
}

// FallbackStrategy
enum FallbackStrategy {
  CACHE = 'cache',
  ALTERNATIVE_PROVIDER = 'alternative_provider',
  REDUCED_PROMPT = 'reduced_prompt',
  MANUAL_REVIEW = 'manual_review',
  STRUCTURED_FAILURE = 'structured_failure',
}

// ErrorCategory
enum ErrorCategory {
  INPUT_VALIDATION = 'input_validation',
  PARSING = 'parsing',
  CLASSIFICATION = 'classification',
  RESOLUTION = 'resolution',
  EXTERNAL_DEPENDENCY = 'external_dependency',
  VALIDATION = 'validation',
  EDITORIAL = 'editorial',
  PUBLICATION = 'publication',
  INFRASTRUCTURE = 'infrastructure',
}

// QueueReason
enum QueueReason {
  LOW_CONFIDENCE = 'low_confidence',
  AMBIGUOUS_CLASSIFICATION = 'ambiguous_classification',
  VALIDATION_ERROR = 'validation_error',
  RESOLUTION_FAILURE = 'resolution_failure',
  MANUAL_REQUEST = 'manual_request',
}
```

## 27.4 Tipos utilitários globais

```typescript
type Nullable<T> = T | null;
type Optional<T> = T | undefined;
type JsonValue = string | number | boolean | null | JsonObject | JsonArray;
type JsonObject = { [key: string]: JsonValue };
type JsonArray = JsonValue[];
type Primitive = string | number | boolean | null;
type DeepPartial<T> = { [K in keyof T]?: T[K] extends object ? DeepPartial<T[K]> : T[K] };
```

## 27.5 Result type para tratamento funcional de erros

```typescript
type Result<T, E = DomainError> = Success<T> | Failure<E>;

class Success<T> {
  readonly isSuccess = true;
  readonly isFailure = false;
  constructor(readonly value: T) {}
}

class Failure<E> {
  readonly isSuccess = false;
  readonly isFailure = true;
  constructor(readonly error: E) {}
}

function success<T>(value: T): Success<T> { return new Success(value); }
function failure<E>(error: E): Failure<E> { return new Failure(error); }
```

---

# 28. Logs Estruturados

## 28.1 Filosofia

Logs são a principal ferramenta de debugging em produção. Devem ser estruturados em JSON, com campos mínimos padronizados, correlacionáveis por `trace_id` e nunca conter dados sensíveis sem mascaramento.

## 28.2 Campos mínimos obrigatórios em todos os logs

| Campo | Tipo | Descrição |
|-------|------|-----------|
| `timestamp` | ISO 8601 | Momento exato do evento com timezone UTC |
| `level` | string | Nível: `trace`, `debug`, `info`, `warn`, `error`, `fatal` |
| `service` | string | Nome do serviço: `question-ai-service` |
| `version` | string | Versão do serviço: `1.2.0` |
| `environment` | string | Ambiente: `development`, `staging`, `production` |
| `trace_id` | string | Identificador do trace OpenTelemetry |
| `span_id` | string | Identificador do span corrente |
| `request_id` | string | Correlation ID do HTTP request |
| `processing_job_id` | string | UUID do job, quando aplicável |
| `question_draft_id` | string | ID do draft, quando aplicável |
| `agent_name` | string | Nome do agente, quando aplicável |
| `event_name` | string | Nome padronizado do evento |
| `message` | string | Mensagem resumida e humano-legível |
| `error_code` | string | Código de erro padronizado, se houver |
| `error_category` | string | Categoria do erro, se houver |
| `duration_ms` | number | Duração da operação em ms, quando mensurável |
| `metadata` | object | Dados adicionais sanitizados e mascarados |

## 28.3 Nomes de eventos padronizados

```typescript
const LOG_EVENT_NAMES = {
  // Jobs
  JOB_CREATED: 'job.created',
  JOB_STARTED: 'job.started',
  JOB_COMPLETED: 'job.completed',
  JOB_FAILED: 'job.failed',
  JOB_CANCELLED: 'job.cancelled',
  JOB_RETRIED: 'job.retried',
  JOB_DEAD_LETTERED: 'job.dead_lettered',

  // Agentes
  AGENT_STARTED: 'agent.started',
  AGENT_COMPLETED: 'agent.completed',
  AGENT_FAILED: 'agent.failed',
  AGENT_TIMEOUT: 'agent.timeout',
  AGENT_RETRIED: 'agent.retried',

  // LLM
  LLM_INVOKED: 'llm.invoked',
  LLM_COMPLETED: 'llm.completed',
  LLM_FAILED: 'llm.failed',
  LLM_RATE_LIMITED: 'llm.rate_limited',
  LLM_FALLBACK_TRIGGERED: 'llm.fallback_triggered',

  // Drafts
  DRAFT_CREATED: 'draft.created',
  DRAFT_VALIDATED: 'draft.validated',
  DRAFT_QUEUED_FOR_REVIEW: 'draft.queued_for_review',
  DRAFT_APPROVED: 'draft.approved',
  DRAFT_REJECTED: 'draft.rejected',
  DRAFT_PUBLISHED: 'draft.published',

  // Integração
  INTEGRATION_STARTED: 'integration.started',
  INTEGRATION_COMPLETED: 'integration.completed',
  INTEGRATION_FAILED: 'integration.failed',

  // Resiliência
  CIRCUIT_BREAKER_OPENED: 'circuit_breaker.opened',
  CIRCUIT_BREAKER_CLOSED: 'circuit_breaker.closed',
  CIRCUIT_BREAKER_HALF_OPEN: 'circuit_breaker.half_open',
  RETRY_SCHEDULED: 'retry.scheduled',
  FALLBACK_TRIGGERED: 'fallback.triggered',
};
```

## 28.4 Exemplo completo de log estruturado — sucesso de agente

```json
{
  "timestamp": "2024-11-15T14:32:18.423Z",
  "level": "info",
  "service": "question-ai-service",
  "version": "1.2.0",
  "environment": "production",
  "trace_id": "4bf92f3577b34da6a3ce929d0e0e4736",
  "span_id": "00f067aa0ba902b7",
  "request_id": "req_9f3a1b2c4d5e6f7a",
  "processing_job_id": "7f3b5a17-6db0-4d8b-95c0-7d65e6a8be0f",
  "question_draft_id": null,
  "agent_name": "classification",
  "event_name": "agent.completed",
  "message": "Classification agent completed successfully",
  "error_code": null,
  "error_category": null,
  "duration_ms": 4287,
  "metadata": {
    "agent_version": "1.1.0",
    "attempt": 1,
    "model_used": "claude-3-5-sonnet",
    "classification_confidence": 0.93,
    "bank_identified": "FGV",
    "discipline_identified": "Direito Constitucional"
  }
}
```

## 28.5 Exemplo completo de log estruturado — falha de agente

```json
{
  "timestamp": "2024-11-15T14:35:02.811Z",
  "level": "error",
  "service": "question-ai-service",
  "version": "1.2.0",
  "environment": "production",
  "trace_id": "4bf92f3577b34da6a3ce929d0e0e4736",
  "span_id": "f45b9c1e2d3a4b5c",
  "request_id": "req_9f3a1b2c4d5e6f7a",
  "processing_job_id": "7f3b5a17-6db0-4d8b-95c0-7d65e6a8be0f",
  "question_draft_id": null,
  "agent_name": "pdf-extraction",
  "event_name": "agent.failed",
  "message": "PDF extraction agent failed after maximum retries",
  "error_code": "PARSING_ERROR_ENCRYPTED_PDF",
  "error_category": "parsing",
  "duration_ms": 92341,
  "metadata": {
    "agent_version": "1.0.0",
    "attempt": 3,
    "max_attempts": 3,
    "file_role": "proof",
    "file_size_bytes": 4293872,
    "mime_type": "application/pdf",
    "is_encrypted": true,
    "retry_exhausted": true,
    "next_action": "dead_letter_queue"
  }
}
```

## 28.6 Exemplo completo de log estruturado — publicação

```json
{
  "timestamp": "2024-11-15T16:10:45.002Z",
  "level": "info",
  "service": "question-ai-service",
  "version": "1.2.0",
  "environment": "production",
  "trace_id": "9e3f8c2a1b4d5e6f7a8b9c0d1e2f3a4b",
  "span_id": "a1b2c3d4e5f6a7b8",
  "request_id": "req_admin_pub_001",
  "processing_job_id": "7f3b5a17-6db0-4d8b-95c0-7d65e6a8be0f",
  "question_draft_id": "4821",
  "agent_name": null,
  "event_name": "draft.published",
  "message": "Question draft published successfully to main database",
  "error_code": null,
  "error_category": null,
  "duration_ms": 312,
  "metadata": {
    "published_question_id": 18394,
    "reviewer_id": "usr_3a9f1b2c4d5e6f7a",
    "filter_ids_linked": [12, 45, 103, 207],
    "sub_matter_ids_linked": [88, 92],
    "confidence_score": 0.93,
    "ia_generated": true
  }
}
```

---

# 29. Observabilidade

## 29.1 Os três pilares

```mermaid
graph LR
    subgraph PILARES["📊 Três Pilares de Observabilidade"]
        L["📋 Logs\nEventos detalhados\nDebugging\nPino + JSON"]
        M["📈 Métricas\nAgregação operacional\nAlertas\nPrometheus"]
        T["🔍 Tracing\nLatência e encadeamento\nFluxo entre agentes\nOpenTelemetry"]
    end

    L --- M
    M --- T
    T --- L
```

## 29.2 Métricas recomendadas por categoria

### Métricas de Jobs

| Nome da Métrica | Tipo | Labels | Descrição |
|-----------------|------|--------|-----------|
| `jobs_created_total` | Counter | `source_type` | Total de jobs criados |
| `jobs_completed_total` | Counter | `source_type` | Total de jobs concluídos |
| `jobs_failed_total` | Counter | `source_type`, `error_category` | Total de jobs falhados |
| `jobs_cancelled_total` | Counter | — | Total de jobs cancelados |
| `job_duration_seconds` | Histogram | `source_type` | Duração do job do início ao fim |
| `job_step_duration_seconds` | Histogram | `step_name` | Duração por etapa do pipeline |
| `job_retry_total` | Counter | `step_name` | Total de retries por etapa |
| `dead_letter_queue_size` | Gauge | — | Tamanho atual da DLQ |

### Métricas de Agentes

| Nome da Métrica | Tipo | Labels | Descrição |
|-----------------|------|--------|-----------|
| `agent_runs_total` | Counter | `agent_name`, `status` | Total de execuções por agente |
| `agent_duration_seconds` | Histogram | `agent_name` | Duração por agente |
| `agent_attempts_total` | Counter | `agent_name` | Total de tentativas |
| `agent_timeout_total` | Counter | `agent_name` | Total de timeouts |

### Métricas de LLM

| Nome da Métrica | Tipo | Labels | Descrição |
|-----------------|------|--------|-----------|
| `llm_invocations_total` | Counter | `provider`, `model`, `status` | Total de invocações |
| `llm_latency_seconds` | Histogram | `provider`, `model` | Latência da invocação |
| `llm_tokens_input_total` | Counter | `provider`, `model` | Total de tokens de entrada |
| `llm_tokens_output_total` | Counter | `provider`, `model` | Total de tokens de saída |
| `llm_fallback_total` | Counter | `from_provider`, `to_provider` | Total de fallbacks de provider |

### Métricas de Qualidade

| Nome da Métrica | Tipo | Labels | Descrição |
|-----------------|------|--------|-----------|
| `drafts_created_total` | Counter | `validation_status` | Total de drafts criados |
| `drafts_approved_total` | Counter | — | Total de drafts aprovados |
| `drafts_rejected_total` | Counter | `queue_reason` | Total de drafts rejeitados |
| `drafts_published_total` | Counter | — | Total de drafts publicados |
| `draft_confidence_score` | Histogram | — | Distribuição do score de confiança |
| `manual_review_queue_size` | Gauge | `priority` | Tamanho da fila de revisão por prioridade |
| `filter_resolution_failure_total` | Counter | `filter_type` | Falhas de resolução por tipo de filtro |

### Métricas de Infraestrutura

| Nome da Métrica | Tipo | Labels | Descrição |
|-----------------|------|--------|-----------|
| `queue_depth` | Gauge | `queue_name` | Profundidade da fila |
| `queue_wait_seconds` | Histogram | `queue_name` | Tempo de espera na fila |
| `redis_cache_hits_total` | Counter | `cache_key_prefix` | Cache hits |
| `redis_cache_misses_total` | Counter | `cache_key_prefix` | Cache misses |
| `mysql_query_duration_seconds` | Histogram | `operation` | Latência de queries MySQL |
| `postgres_query_duration_seconds` | Histogram | `operation` | Latência de queries PostgreSQL |
| `circuit_breaker_state` | Gauge | `component` | Estado atual: 0=closed, 1=open, 2=half_open |

---

# 30. Timeouts, Retries e Circuit Breaker

## 30.1 Política de timeout por componente

| Componente | Timeout Mínimo | Timeout Máximo | Observação |
|------------|---------------|---------------|------------|
| Upload validation | 2s | 5s | Validação síncrona de MIME e hash |
| Criação de job | 1s | 2s | Operação simples de escrita |
| PDF parsing (pequeno < 1MB) | 15s | 30s | Arquivo pequeno |
| PDF parsing (médio 1–5MB) | 30s | 60s | Arquivo médio |
| PDF parsing (grande > 5MB) | 60s | 90s | Arquivo grande |
| Classification Agent | 10s | 30s | Dependente de provider e lote |
| ID Resolution (cache hit) | 50ms | 200ms | Redis hit |
| ID Resolution (MySQL) | 2s | 5s | Consulta com índice |
| Search Agent (PGVector) | 3s | 15s | Busca semântica |
| Adaptation Agent | 10s | 20s | Provider IA por questão |
| Answer Key Agent | 10s | 20s | Provider IA por questão |
| Validation Agent | 1s | 5s | Processamento local |
| Publicação MySQL | 3s | 10s | Transação com rollback |

## 30.2 Política de retry por tipo de erro

| Tipo de Erro | Deve Retentar? | Máx. Tentativas | Backoff | Jitter |
|--------------|---------------|----------------|---------|--------|
| Timeout de rede | Sim | 3 | Exponencial | Sim |
| HTTP 429 Rate Limited | Sim | 3 | Exponencial com `Retry-After` | Sim |
| HTTP 5xx Servidor | Sim | 3 | Exponencial | Sim |
| Lock Redis temporário | Sim | 5 | Linear 200ms | Não |
| Erro de parsing de PDF | Não | 0 | — | — |
| Schema inválido do LLM | Não | 0 | — | — |
| Erro de validação de input | Não | 0 | — | — |
| Arquivo inválido | Não | 0 | — | — |
| Erro de autenticação | Não | 0 | — | — |

## 30.3 Fórmula de backoff exponencial com jitter

```typescript
function calculateBackoff(attempt: number, baseMs: number, maxMs: number): number {
  const exponential = Math.min(baseMs * Math.pow(2, attempt - 1), maxMs);
  const jitter = Math.random() * exponential * 0.3; // até 30% de jitter
  return Math.floor(exponential + jitter);
}
// Tentativa 1: ~1000ms, Tentativa 2: ~2100ms, Tentativa 3: ~4200ms
```

## 30.4 Circuit Breaker por componente

| Componente | Limiar de Abertura | Janela | Tempo em Open | Observação |
|------------|-------------------|--------|---------------|------------|
| LLM Provider principal | 5 falhas em 60s | 60s | 30s | Fallback para provider alternativo |
| LLM Provider alternativo | 3 falhas em 60s | 60s | 60s | Fallback para revisão manual |
| Busca web (fallback) | 3 falhas em 120s | 120s | 120s | Fallback para índice local |
| MySQL principal | 3 falhas em 30s | 30s | 15s | Alertar imediatamente |
| Redis | 5 falhas em 60s | 60s | 10s | Operar sem cache temporariamente |

```mermaid
stateDiagram-v2
    direction LR
    [*] --> Fechado: Inicialização

    Fechado --> Aberto: Limiar de falhas atingido
    Aberto --> MeioAberto: Tempo de espera expirou
    MeioAberto --> Fechado: Requisição de sonda bem-sucedida
    MeioAberto --> Aberto: Requisição de sonda falhou

    Fechado: 🟢 CLOSED\nPassagem normal
    Aberto: 🔴 OPEN\nFalha rápida
    MeioAberto: 🟡 HALF-OPEN\nTeste de sonda
```

---

# 31. Fallbacks e Estratégias de Degradação

## 31.1 Fluxo de fallback por componente

```mermaid
flowchart TD
    subgraph RESOLUCAO["🔗 Resolução de IDs"]
        R1["1. Redis Cache\nTTL 1h"] -->|miss| R2["2. Consulta MySQL\nPor slug normalizado"]
        R2 -->|não encontrado| R3["3. Busca aproximada\nLevenshtein controlado"]
        R3 -->|ambíguo| R4["4. Revisão Manual\nFila de revisão"]
    end

    subgraph BUSCA["📚 Busca Legal"]
        S1["1. PGVector\nEmbedding semântico"] -->|indisponível| S2["2. Índice legal\nEstruturado local"]
        S2 -->|sem resultado| S3["3. Busca web\nFallback controlado"]
        S3 -->|falhou| S4["4. Revisão Manual\nSem fundamentação"]
    end

    subgraph LLM["🤖 Geração por IA"]
        L1["1. Provider principal\nex.: Anthropic"] -->|falhou ou lento| L2["2. Provider alternativo\nex.: OpenAI"]
        L2 -->|falhou| L3["3. Retry com\nprompt reduzido"]
        L3 -->|falhou| L4["4. Falha estruturada\nStatus: failed"]
    end
```

## 31.2 Regras de degradação controlada

Degradação controlada significa que o sistema continua operando, mas registra e sinaliza o fallback:

- todo fallback deve ser registrado em `agent_runs` com o campo `error_code` indicando o fallback utilizado;
- o `confidence_score` do draft deve ser penalizado quando fontes de fallback forem utilizadas;
- drafts gerados com fallback de revisão manual devem ser automaticamente encaminhados para a fila;
- alertas de observabilidade devem ser disparados quando o fallback de segundo nível ou mais for utilizado.

---

# 32. Idempotência

## 32.1 Casos que exigem idempotência

- upload do mesmo par prova/gabarito identificado pelo `input_hash`;
- reprocessamento de job com o mesmo `job_key`;
- retries automáticos de qualquer etapa do pipeline;
- publicação na base principal;
- criação de vínculos em `question_filters` e `question_sub_matters`.

## 32.2 Estratégias por contexto

| Contexto | Estratégia | Mecanismo |
|----------|-----------|-----------|
| Upload e criação de job | Hash de entrada | `UNIQUE constraint` em `input_hash` + upsert |
| Etapas do pipeline | Lock distribuído | Redis SETNX com TTL |
| Publicação MySQL | Constraint única | `UNIQUE` em `question_id` + `filter_id` para `question_filters` |
| Retries de workers | Job ID único | BullMQ `jobId` explícito derivado do `job_key` |
| Resolução de IDs | Cache idempotente | Redis com TTL e invalidação controlada |

## 32.3 Implementação de lock distribuído

```typescript
async function withDistributedLock<T>(
  cacheGateway: CacheGatewayContract,
  lockKey: string,
  ttlMs: number,
  fn: () => Promise<T>,
): Promise<T> {
  const acquired = await cacheGateway.lock(lockKey, ttlMs);
  if (!acquired) {
    throw new RetryableError('LOCK_NOT_ACQUIRED', `Lock ${lockKey} already held`);
  }
  try {
    return await fn();
  } finally {
    await cacheGateway.unlock(lockKey);
  }
}
```

---

# 33. Segurança, Robustez e Resiliência

## 33.1 Segurança de uploads

- validar MIME real usando `file-type` (não apenas extensão);
- validar tamanho máximo por arquivo (configurável, padrão 20MB);
- calcular checksum SHA-256 para detecção de duplicidade e integridade;
- armazenar fora de área pública (S3 com acesso privado ou path não exposto);
- gerar URL pré-assinada de acesso temporário quando necessário para leitura;
- aplicar TTL de 7 dias em arquivos temporários;
- nunca executar código extraído de PDFs.

## 33.2 Segurança da integração com banco principal

- usar credenciais segregadas por função: leitura de filtros, leitura de taxonomia e escrita de questões;
- aplicar mínimo privilégio por role de banco;
- usar pool de conexões com tamanho máximo configurado;
- nunca montar queries por concatenação de strings; usar sempre parâmetros preparados;
- registrar todas as operações no `integration_logs` com payload sanitizado.

## 33.3 Segurança de prompts e LLMs

- nunca incluir dados de usuário raw nos prompts sem sanitização;
- nunca logar o conteúdo completo de prompts ou respostas de LLM em produção;
- registrar apenas `response_summary` e `request_hash` no `llm_invocations`;
- versionar todos os prompts e registrar a versão no `llm_invocations`;
- aplicar limite de tokens de saída configurável por agente.

## 33.4 Execução de jobs

- timeout obrigatório por etapa, nunca operar sem limite;
- retry com backoff exponencial e jitter;
- dead-letter queue para jobs que esgotaram tentativas;
- circuit breaker para dependências externas;
- locks distribuídos para garantir processamento único por job.

## 33.5 Segurança de headers e transporte

- TLS obrigatório em produção para todos os endpoints;
- headers de segurança via `helmet`;
- CORS restrito a origens explicitamente permitidas;
- rate limiting aplicado por IP e por usuário autenticado;
- correlation ID em todas as requisições para rastreabilidade.

---

# 34. Erro como Parte do Fluxo

## 34.1 Hierarquia de erros de domínio

```typescript
abstract class DomainError extends Error {
  abstract readonly code: string;
  abstract readonly category: ErrorCategory;
  readonly isRetryable: boolean = false;
  readonly timestamp: Date = new Date();
}

class RetryableError extends DomainError {
  readonly isRetryable = true;
  constructor(readonly code: string, message: string, readonly category: ErrorCategory) {
    super(message);
  }
}

class NonRetryableError extends DomainError {
  readonly isRetryable = false;
  constructor(readonly code: string, message: string, readonly category: ErrorCategory) {
    super(message);
  }
}

class TimeoutError extends RetryableError {
  readonly category = ErrorCategory.EXTERNAL_DEPENDENCY;
  constructor(readonly code: string, readonly durationMs: number) {
    super(code, `Operation timed out after ${durationMs}ms`, ErrorCategory.EXTERNAL_DEPENDENCY);
  }
}

class ParsingError extends NonRetryableError {
  readonly category = ErrorCategory.PARSING;
}

class ClassificationError extends RetryableError {
  readonly category = ErrorCategory.CLASSIFICATION;
}

class IntegrationError extends RetryableError {
  readonly category = ErrorCategory.EXTERNAL_DEPENDENCY;
}
```

## 34.2 Estados possíveis de resultado de uma etapa

| Estado | Significado | Próxima Ação |
|--------|-------------|-------------|
| `success` | Operação completamente bem-sucedida | Continuar pipeline |
| `partial_success` | Completou com degradação aceitável | Continuar com flag de qualidade reduzida |
| `retryable_failure` | Falha transitória, pode ser tentada novamente | Retentar com backoff |
| `non_retryable_failure` | Falha permanente, não adianta retentar | Registrar e encaminhar para DLQ ou revisão |
| `manual_review_required` | Resultado ambíguo que exige julgamento humano | Encaminhar para fila de revisão |

## 34.3 Resposta padronizada de erro na API

```json
{
  "error": {
    "code": "UPLOAD_FILE_TOO_LARGE",
    "category": "input_validation",
    "message": "O arquivo enviado excede o tamanho máximo permitido de 20MB.",
    "details": {
      "max_size_bytes": 20971520,
      "received_size_bytes": 31457280,
      "field": "files[0]"
    },
    "request_id": "req_9f3a1b2c4d5e6f7a",
    "timestamp": "2024-11-15T14:32:18.423Z"
  }
}
```

---

# 35. Normalização e Qualidade de Dados

## 35.1 Normalizações obrigatórias de texto

Aplicadas a todos os campos de texto antes de qualquer persistência:

- trim de espaços no início e fim;
- colapso de múltiplos espaços internos para um único espaço;
- normalização Unicode NFD e remoção de caracteres de controle;
- padronização de pontuação: aspas, travessões e reticências;
- remoção de caracteres residuais de extração de PDF como `\f`, `\x0c` e sequências de bytes inválidas.

## 35.2 Normalizações obrigatórias de contexto

Aplicadas antes de qualquer lookup de ID:

- nomes de banca: `"fgv"`, `"FGV"`, `"Fundação Getúlio Vargas"` → `"FGV"`;
- anos: `"2024/2025"`, `"2024-2025"` → `"2024"`;
- nomes de órgão: `"TCU"`, `"t.c.u."`, `"Tribunal de Contas da União"` → `"TCU"`;
- referências de artigo: `"art. 5°"`, `"Art 5"`, `"artigo 5"` → `"art. 5º"`;
- nomes de lei: `"CF/88"`, `"CF-88"`, `"Constituição Federal de 1988"` → `"CF88"`.

## 35.3 Validações obrigatórias de schema de payload por agente

Cada agente deve validar seu payload de saída antes de retornar ao orchestrator. Schemas devem ser mantidos em `core/shared/validation/schemas/` e usados tanto no agente produtor quanto no consumidor.

---

# 36. Payloads, Contratos JSON e Exemplos Completos

## 36.1 Request: Upload de PDFs e criação de job

`POST /api/v1/uploads`

**Headers:**
```
Authorization: Bearer eyJhbGciOiJSUzI1NiJ9...
Content-Type: multipart/form-data
X-Request-ID: req_9f3a1b2c4d5e6f7a
```

**Body (multipart/form-data):**
```
files[0]: [binary PDF da prova]
files[1]: [binary PDF do gabarito]
metadata: {"priority":"normal","pipelineVersion":"v1.0.0"}
```

**Response 201 Created:**
```json
{
  "data": {
    "jobId": "7f3b5a17-6db0-4d8b-95c0-7d65e6a8be0f",
    "jobKey": "pdf_upload:sha256:1a6e0d5d08be9c2f7d8f8a725d7f6be9",
    "status": "pending",
    "sourceType": "pdf_upload",
    "uploadedFiles": [
      {
        "fileId": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
        "role": "proof",
        "originalName": "fgv-direito-constitucional-2024.pdf",
        "mimeType": "application/pdf",
        "sizeBytes": 3721842,
        "checksumSha256": "1a6e0d5d08be9c2f7d8f8a725d7f6be95d2f1986f4f95e66c1c9f462edcb8a01"
      },
      {
        "fileId": "b2c3d4e5-f6a7-8901-bcde-f12345678901",
        "role": "answer_key",
        "originalName": "fgv-direito-constitucional-2024-gabarito.pdf",
        "mimeType": "application/pdf",
        "sizeBytes": 124390,
        "checksumSha256": "9b7f4c1e2d3a5b6c7d8e9f0a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b0"
      }
    ],
    "createdAt": "2024-11-15T14:30:00.000Z"
  },
  "meta": {
    "requestId": "req_9f3a1b2c4d5e6f7a",
    "apiVersion": "v1"
  }
}
```

**Response 200 OK (job já existia — idempotência):**
```json
{
  "data": {
    "jobId": "7f3b5a17-6db0-4d8b-95c0-7d65e6a8be0f",
    "status": "completed",
    "idempotent": true,
    "message": "Job already exists for this input hash."
  },
  "meta": {
    "requestId": "req_new_request_id",
    "apiVersion": "v1"
  }
}
```

**Response 422 Unprocessable Entity (arquivo inválido):**
```json
{
  "error": {
    "code": "UPLOAD_INVALID_MIME_TYPE",
    "category": "input_validation",
    "message": "Um dos arquivos enviados não é um PDF válido.",
    "details": {
      "field": "files[1]",
      "expected_mime": "application/pdf",
      "received_mime": "application/octet-stream",
      "original_name": "gabarito.exe"
    },
    "request_id": "req_9f3a1b2c4d5e6f7a",
    "timestamp": "2024-11-15T14:30:01.000Z"
  }
}
```

---

## 36.2 Request: Consulta de status de job

`GET /api/v1/jobs/7f3b5a17-6db0-4d8b-95c0-7d65e6a8be0f`

**Response 200 OK:**
```json
{
  "data": {
    "jobId": "7f3b5a17-6db0-4d8b-95c0-7d65e6a8be0f",
    "jobKey": "pdf_upload:sha256:1a6e0d5d08be9c2f7d8f8a725d7f6be9",
    "status": "completed",
    "progressPct": 100,
    "sourceType": "pdf_upload",
    "requestedBy": "8d8b9f4a-1e6d-4d3d-8a18-59d9d74f7e11",
    "startedAt": "2024-11-15T14:30:05.000Z",
    "finishedAt": "2024-11-15T14:38:42.000Z",
    "durationMs": 517000,
    "steps": [
      {
        "stepName": "extraction",
        "stepOrder": 1,
        "status": "completed",
        "attempt": 1,
        "startedAt": "2024-11-15T14:30:05.000Z",
        "finishedAt": "2024-11-15T14:31:22.000Z",
        "durationMs": 77000
      },
      {
        "stepName": "classification",
        "stepOrder": 2,
        "status": "completed",
        "attempt": 1,
        "startedAt": "2024-11-15T14:31:22.000Z",
        "finishedAt": "2024-11-15T14:31:43.000Z",
        "durationMs": 21000
      },
      {
        "stepName": "resolution",
        "stepOrder": 3,
        "status": "completed",
        "attempt": 1,
        "startedAt": "2024-11-15T14:31:43.000Z",
        "finishedAt": "2024-11-15T14:31:46.000Z",
        "durationMs": 3000
      },
      {
        "stepName": "search",
        "stepOrder": 4,
        "status": "completed",
        "attempt": 1,
        "startedAt": "2024-11-15T14:31:46.000Z",
        "finishedAt": "2024-11-15T14:31:54.000Z",
        "durationMs": 8000
      },
      {
        "stepName": "adaptation",
        "stepOrder": 5,
        "status": "completed",
        "attempt": 1,
        "startedAt": "2024-11-15T14:31:54.000Z",
        "finishedAt": "2024-11-15T14:35:12.000Z",
        "durationMs": 198000
      },
      {
        "stepName": "answer_key",
        "stepOrder": 6,
        "status": "completed",
        "attempt": 1,
        "startedAt": "2024-11-15T14:35:12.000Z",
        "finishedAt": "2024-11-15T14:38:28.000Z",
        "durationMs": 196000
      },
      {
        "stepName": "validation",
        "stepOrder": 7,
        "status": "completed",
        "attempt": 1,
        "startedAt": "2024-11-15T14:38:28.000Z",
        "finishedAt": "2024-11-15T14:38:42.000Z",
        "durationMs": 14000
      }
    ],
    "draftsGenerated": 20,
    "draftsValid": 17,
    "draftsNeedingReview": 3,
    "createdAt": "2024-11-15T14:30:00.000Z"
  },
  "meta": {
    "requestId": "req_get_job_001",
    "apiVersion": "v1"
  }
}
```

---

## 36.3 Response: Draft completo

`GET /api/v1/drafts/4821`

```json
{
  "data": {
    "id": 4821,
    "processingJobId": "7f3b5a17-6db0-4d8b-95c0-7d65e6a8be0f",
    "sourceQuestionNumber": "12",
    "title": "Direito adquirido, ato jurídico perfeito e coisa julgada na CF/88",
    "description": "A Constituição Federal de 1988 prevê expressamente que a lei não prejudicará o direito adquirido, o ato jurídico perfeito e a coisa julgada.",
    "explanation": "<p><strong>Correta.</strong> O <strong>art. 5º, XXXVI</strong> da <strong>Constituição Federal de 1988</strong> estabelece que <em>\"a lei não prejudicará o direito adquirido, o ato jurídico perfeito e a coisa julgada\"</em>. Trata-se de garantia fundamental que protege situações jurídicas já consolidadas contra a aplicação retroativa de novas leis.</p>",
    "isTrue": true,
    "confidenceScore": 0.93,
    "validationStatus": "valid",
    "reviewStatus": "pending_review",
    "reviewedBy": null,
    "reviewReason": null,
    "filters": [
      {
        "filterId": 12,
        "filterTypeName": "banca",
        "filterName": "FGV",
        "resolutionConfidence": 0.99
      },
      {
        "filterId": 45,
        "filterTypeName": "ano",
        "filterName": "2024",
        "resolutionConfidence": 0.99
      },
      {
        "filterId": 103,
        "filterTypeName": "orgao",
        "filterName": "TRF 1ª Região",
        "resolutionConfidence": 0.97
      },
      {
        "filterId": 207,
        "filterTypeName": "cargo",
        "filterName": "Analista Judiciário",
        "resolutionConfidence": 0.95
      }
    ],
    "subMatters": [
      {
        "subMatterId": 88,
        "disciplineName": "Direito Constitucional",
        "matterName": "Direitos e Garantias Fundamentais",
        "subMatterName": "Direito Adquirido e Coisa Julgada",
        "resolutionConfidence": 0.96,
        "position": 1
      }
    ],
    "sources": [
      {
        "sourceType": "legal_text",
        "sourceRef": "CF88-art5-XXXVI",
        "title": "Constituição Federal — Art. 5º, XXXVI",
        "excerpt": "a lei não prejudicará o direito adquirido, o ato jurídico perfeito e a coisa julgada",
        "rankScore": 0.9872
      },
      {
        "sourceType": "pgvector",
        "sourceRef": "chunk_legal_4821",
        "title": "Direito Constitucional — Garantias Individuais",
        "excerpt": "O princípio da irretroatividade das leis protege situações jurídicas consolidadas...",
        "rankScore": 0.9341
      }
    ],
    "createdAt": "2024-11-15T14:38:42.000Z",
    "updatedAt": "2024-11-15T14:38:42.000Z"
  },
  "meta": {
    "requestId": "req_get_draft_001",
    "apiVersion": "v1"
  }
}
```

---

## 36.4 Request: Aprovar draft

`POST /api/v1/drafts/4821/approve`

**Body:**
```json
{
  "reviewNote": "Questão revisada e aprovada. Fundamentação legal correta e enunciado claro."
}
```

**Response 200 OK:**
```json
{
  "data": {
    "id": 4821,
    "reviewStatus": "approved",
    "reviewedBy": "usr_3a9f1b2c4d5e6f7a",
    "reviewNote": "Questão revisada e aprovada. Fundamentação legal correta e enunciado claro.",
    "updatedAt": "2024-11-15T16:05:30.000Z"
  },
  "meta": {
    "requestId": "req_approve_draft_001",
    "apiVersion": "v1"
  }
}
```

---

## 36.5 Request: Rejeitar draft

`POST /api/v1/drafts/4822/reject`

**Body:**
```json
{
  "reviewReason": "Enunciado ambíguo. O texto 'não prejudicará' pode ser interpretado de múltiplas formas sem contexto adicional. Necessita reformulação.",
  "suggestedAction": "reprocess"
}
```

**Response 200 OK:**
```json
{
  "data": {
    "id": 4822,
    "reviewStatus": "rejected",
    "reviewedBy": "usr_3a9f1b2c4d5e6f7a",
    "reviewReason": "Enunciado ambíguo. O texto 'não prejudicará' pode ser interpretado de múltiplas formas sem contexto adicional. Necessita reformulação.",
    "suggestedAction": "reprocess",
    "updatedAt": "2024-11-15T16:08:12.000Z"
  },
  "meta": {
    "requestId": "req_reject_draft_001",
    "apiVersion": "v1"
  }
}
```

---

## 36.6 Request: Publicar draft aprovado

`POST /api/v1/drafts/4821/publish`

**Response 201 Created:**
```json
{
  "data": {
    "draftId": 4821,
    "publishedQuestionId": 18394,
    "publishedAt": "2024-11-15T16:10:45.000Z",
    "linkedFilters": [12, 45, 103, 207],
    "linkedSubMatters": [88],
    "transactionStatus": "committed"
  },
  "meta": {
    "requestId": "req_publish_draft_001",
    "apiVersion": "v1"
  }
}
```

---

## 36.7 Response: Métricas de jobs

`GET /api/v1/observability/jobs/metrics`

```json
{
  "data": {
    "period": {
      "from": "2024-11-15T00:00:00.000Z",
      "to": "2024-11-15T23:59:59.999Z"
    },
    "jobs": {
      "createdTotal": 45,
      "completedTotal": 41,
      "failedTotal": 3,
      "cancelledTotal": 1,
      "inProgressCount": 0,
      "pendingCount": 0,
      "avgDurationMs": 487000,
      "p95DurationMs": 892000,
      "retryTotal": 7,
      "deadLetterCount": 1
    },
    "drafts": {
      "generatedTotal": 820,
      "validTotal": 748,
      "needingReviewTotal": 51,
      "invalidTotal": 21,
      "avgConfidenceScore": 0.87,
      "p10ConfidenceScore": 0.62
    },
    "queue": {
      "manualReviewPending": 38,
      "manualReviewHighPriority": 5
    }
  },
  "meta": {
    "requestId": "req_metrics_001",
    "apiVersion": "v1"
  }
}
```

---

## 36.8 Response: Health check detalhado

`GET /api/v1/health/detailed`

```json
{
  "data": {
    "status": "degraded",
    "version": "1.2.0",
    "uptime": 864000,
    "checkedAt": "2024-11-15T16:15:00.000Z",
    "dependencies": {
      "postgres": {
        "status": "healthy",
        "latencyMs": 12,
        "details": "Connection pool: 5/20 active"
      },
      "mysql": {
        "status": "healthy",
        "latencyMs": 8,
        "details": "Connection pool: 2/10 active"
      },
      "redis": {
        "status": "healthy",
        "latencyMs": 2,
        "details": "Used memory: 128MB / 512MB"
      },
      "bullmq": {
        "status": "healthy",
        "details": {
          "waitingJobs": 0,
          "activeJobs": 0,
          "failedJobs": 1,
          "deadLetterJobs": 1
        }
      },
      "llm_provider_primary": {
        "status": "degraded",
        "details": "Circuit breaker: half_open. Last error: timeout at 14:10:32Z"
      },
      "llm_provider_fallback": {
        "status": "healthy",
        "latencyMs": 890,
        "details": "Currently handling overflow from primary"
      },
      "storage_s3": {
        "status": "healthy",
        "latencyMs": 45,
        "details": "Bucket accessible"
      }
    }
  },
  "meta": {
    "requestId": "req_health_001",
    "apiVersion": "v1"
  }
}
```

---

## 36.9 Payload interno consolidado do orchestrator para o draft

Este é o `raw_payload` salvo em `question_drafts.raw_payload`:

```json
{
  "pipelineVersion": "v1.0.0",
  "extractionPayload": {
    "agentVersion": "1.0.0",
    "sourceQuestionNumber": "12",
    "rawText": "12. A Constituição Federal de 1988 prevê que a lei não prejudicará o direito adquirido...",
    "rawAnswer": "C",
    "parsingConfidence": 0.99
  },
  "classificationPayload": {
    "agentVersion": "1.1.0",
    "bankName": "FGV",
    "year": "2024",
    "organizationName": "TRF 1ª Região",
    "roleName": "Analista Judiciário",
    "lawName": "CF88",
    "articleReference": "art. 5º, XXXVI",
    "disciplineName": "Direito Constitucional",
    "matterName": "Direitos e Garantias Fundamentais",
    "subMatterName": "Direito Adquirido e Coisa Julgada",
    "classificationConfidence": 0.93
  },
  "resolutionPayload": {
    "agentVersion": "1.0.0",
    "resolvedFilters": [
      {"filterTypeSlug": "banca", "filterId": 12, "confidence": 0.99},
      {"filterTypeSlug": "ano", "filterId": 45, "confidence": 0.99},
      {"filterTypeSlug": "orgao", "filterId": 103, "confidence": 0.97},
      {"filterTypeSlug": "cargo", "filterId": 207, "confidence": 0.95}
    ],
    "resolvedSubMatters": [
      {"subMatterId": 88, "confidence": 0.96, "position": 1}
    ],
    "unresolvedItems": []
  },
  "searchPayload": {
    "agentVersion": "1.0.0",
    "sourcesFound": 2,
    "topRankScore": 0.9872,
    "searchStrategy": "pgvector"
  },
  "adaptationPayload": {
    "agentVersion": "1.0.0",
    "title": "Direito adquirido, ato jurídico perfeito e coisa julgada na CF/88",
    "description": "A Constituição Federal de 1988 prevê expressamente que a lei não prejudicará o direito adquirido, o ato jurídico perfeito e a coisa julgada.",
    "isTrue": true,
    "adaptationConfidence": 0.97
  },
  "answerKeyPayload": {
    "agentVersion": "1.0.0",
    "explanation": "<p><strong>Correta.</strong> O <strong>art. 5º, XXXVI</strong> da <strong>Constituição Federal de 1988</strong>...</p>",
    "answerKeyConfidence": 0.95
  },
  "validationPayload": {
    "agentVersion": "1.0.0",
    "validationStatus": "valid",
    "confidenceScore": 0.93,
    "requiresManualReview": false,
    "errors": []
  }
}
```

---

# 37. Estratégia de Testes

## 37.1 Pirâmide de testes

```mermaid
graph TD
    subgraph PIRAMIDE["🧪 Pirâmide de Testes"]
        E2E["🔵 E2E\nPoucos · Lentos · Alta cobertura de fluxo"]
        INT["🟡 Integração\nModerados · Médios · Dependências reais"]
        UNIT["🟢 Unitários\nMuitos · Rápidos · Domínio e aplicação"]
    end

    UNIT --> INT --> E2E
```

## 37.2 Testes unitários

Cobrem a camada de domínio e aplicação sem qualquer dependência externa:

- parsers de payload e normalização;
- normalizadores globais;
- validadores de schema;
- mapeadores;
- resolvedores de IDs (com fakes);
- regras de domínio;
- policies de timeout, retry e idempotência;
- value objects;
- domain errors.

**Cobertura mínima exigida:** 80% nas camadas `core/domain` e `core/application`.

## 37.3 Testes de integração

Cobrem a camada de infraestrutura com dependências reais em ambiente de teste:

- repositórios PostgreSQL com banco de teste;
- gateways MySQL com banco de teste;
- Redis com lock e cache real;
- BullMQ com queue real;
- LLM providers mockados via `llm.gateway.mock.ts`;
- PGVector com banco real e seed de materiais legais.

## 37.4 Testes end-to-end

Cobrem o fluxo completo da API até o banco:

- `upload-to-draft.e2e-spec.ts` — upload de PDF real até a criação de draft;
- `draft-publication.e2e-spec.ts` — aprovação e publicação de draft;
- `retry-idempotency.e2e-spec.ts` — reprocessamento e garantia de idempotência.

## 37.5 Testes de contrato

Garantem que os contratos de agentes e gateways são respeitados:

- cada agente produz a saída no formato correto;
- cada gateway implementa o contrato exato;
- mudanças de schema são detectadas antes do merge.

## 37.6 Testes de resiliência

Cobrem comportamento sob falha:

- `timeout.spec.ts` — operações que excedem timeout devem falhar com `TimeoutError`;
- `retry.spec.ts` — retries devem respeitar backoff e máximo de tentativas;
- `circuit-breaker.spec.ts` — circuit breaker abre após limiar e fecha após período;
- `fallback.spec.ts` — fallbacks são acionados na ordem correta.

---

# 38. Regras Arquiteturais

## 38.1 `questions` permanece entidade final
A tabela final deve conter apenas conteúdo editorial pronto para consumo dos usuários.

## 38.2 O agente nunca publica diretamente
Toda geração passa obrigatoriamente pela camada de staging em `question_drafts`.

## 38.3 Contexto continua via filtros
Não criar colunas como `bank_id`, `year`, `organization_id` em `questions` ou `question_drafts`. Usar exclusivamente a estrutura `filter_types → filters`.

## 38.4 Taxonomia pedagógica preservada
A hierarquia `disciplines → matters → sub_matters` deve ser reutilizada. Não criar estruturas paralelas.

## 38.5 Toda saída precisa ser auditável
Nenhum dado deve entrar em produção sem rastreio por `trace_id`, `processing_job_id` e `question_draft_id`.

## 38.6 Contratos obrigatórios
Toda comunicação entre camadas deve passar por interfaces tipadas. Nunca injetar implementações concretas de infraestrutura diretamente no domínio.

## 38.7 Retry exige idempotência
Todo mecanismo de retry deve ser seguro por design, garantido por `input_hash`, `job_key` ou constraint única.

## 38.8 API sempre versionada
Nenhuma rota deve ser exposta sem versionamento de URL. O versionamento nunca pode ser removido sem período de deprecação explícito.

## 38.9 Logs sem dados sensíveis
Credenciais, tokens, conteúdo bruto de PDF e prompts completos nunca devem aparecer em logs sem mascaramento.

## 38.10 Timeouts explícitos em toda dependência externa
Nunca chamar um provider externo sem timeout configurado. O timeout deve ser explícito no código, não depender do default do client HTTP.

---

# 39. Roadmap Técnico

## Fase 1 — Infraestrutura Base

Objetivo: criar o alicerce técnico do serviço.

- setup NestJS + TypeScript com versionamento de API;
- PostgreSQL + PGVector com migrations iniciais;
- conexão MySQL principal com pool e credenciais segregadas;
- Redis com suporte a cache e locks;
- BullMQ com dead-letter queue configurada;
- logging estruturado com Pino e JSON;
- tracing OpenTelemetry básico;
- métricas Prometheus básicas;
- segurança: JWT, RBAC, rate limiting, helmet, CORS;
- seed inicial da base legal para PGVector.

## Fase 2 — Agentes Básicos

Objetivo: extrair e classificar questões com confiabilidade.

- PDF Extraction Agent com suporte a arquivos de até 20MB;
- Classification Agent com prompt versionado;
- ID Resolution Agent com cache Redis e fallback para MySQL;
- Search Agent com PGVector e fallback para índice estruturado;
- contratos internos entre todos os agentes;
- persistência de `processing_jobs`, `uploaded_files` e `agent_runs`.

## Fase 3 — Agentes Avançados

Objetivo: gerar e validar questões completas.

- Adaptation Agent com geração V/F;
- Answer Key Agent com HTML estruturado;
- Validation Agent com score de confiança;
- orchestrator completo com tratamento de falhas parciais;
- persistência de `question_drafts` e `question_sources`;
- fallbacks principais por agente.

## Fase 4 — Processamento Assíncrono e APIs

Objetivo: expor a API completa e processar jobs de forma assíncrona.

- APIs versionadas v1 completas;
- workers BullMQ para todos os jobs;
- status de jobs em tempo real;
- reprocessamento seguro com idempotência;
- dead-letter queue com alerta;
- circuit breaker para providers externos.

## Fase 5 — Interface Administrativa

Objetivo: habilitar revisão humana e monitoramento operacional.

- Dashboard de upload e listagem de jobs;
- tela de revisão de drafts com aprovação e rejeição;
- edição de drafts antes da publicação;
- fila de revisão manual com priorização;
- painel de métricas operacionais;
- painel de falhas e dead-letter.

## Fase 6 — Hardening e Produção

Objetivo: garantir produção com qualidade e segurança.

- observabilidade completa: dashboards Grafana, alertas;
- testes de carga com k6;
- otimização de queries e índices;
- revisão completa de segurança;
- pentest básico;
- deploy com blue/green ou canary;
- documentação operacional e runbooks completos.

---

# 40. Conclusão

A arquitetura apresentada neste documento representa uma evolução segura, estruturada e auditável sobre a base atual da plataforma, respeitando as decisões já consolidadas como o modelo de filtros contextuais e a hierarquia pedagógica existente.

Os principais diferenciais da arquitetura proposta são:

- **Versionamento de API explícito** com prefixo URI, permitindo evolução controlada dos contratos sem impactar consumidores existentes;
- **Segurança em camadas** com JWT RS256, RBAC granular, rate limiting, validação de MIME real e headers de segurança;
- **Pipeline multiagente desacoplado** onde cada agente possui responsabilidade única, contrato tipado e tolerância a falhas independente;
- **Staging obrigatório** que separa a geração de IA da publicação em produção, com revisão humana estruturada;
- **Observabilidade nativa** com logs estruturados em JSON, métricas Prometheus e tracing OpenTelemetry desde o primeiro deploy;
- **Resiliência operacional** com timeouts explícitos, retries idempotentes, circuit breakers por dependência e fallbacks progressivos;
- **Dicionário de dados completo** cobrindo tanto as tabelas existentes quanto todas as novas tabelas de staging e auditoria;
- **Roteamento preciso** com regex de validação de parâmetros e versionamento por contexto funcional;
- **Exemplos completos de payload** representando todos os contratos da API de ponta a ponta.

O caminho recomendado é: **pipeline robusto com staging intermediário, classificação reutilizando o modelo atual de filtros e taxonomia, estrutura de código organizada por contratos e camadas, logs estruturados, observabilidade nativa, retries seguros, timeouts explícitos, circuit breakers nas dependências críticas e publicação atômica e controlada**.

---

> Documento gerado em: Abril de 2026 · Versão do documento: 1.0.0 · Serviço: `question-ai-service`
