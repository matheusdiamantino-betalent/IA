# 🧠 Arquitetura Técnica — Agente de Geração de Questões com IA

> Documento técnico de arquitetura, dados e fluxos do serviço de IA responsável por transformar PDFs de provas em questões no formato Verdadeiro/Falso, com classificação, fundamentação legal, validação e publicação controlada.

---

## 📚 Sumário

- [1. Objetivo](#1-objetivo)
- [2. Contexto do Problema](#2-contexto-do-problema)
- [3. Objetivos do Sistema](#3-objetivos-do-sistema)
- [4. Visão Geral da Solução](#4-visão-geral-da-solução)
- [5. Princípios Arquiteturais](#5-princípios-arquiteturais)
- [6. Estado Atual da Base Principal](#6-estado-atual-da-base-principal)
- [7. Modelo de Dados Atual](#7-modelo-de-dados-atual)
- [8. Diagrama ER do Modelo Atual](#8-diagrama-er-do-modelo-atual)
- [9. Interpretação Técnica do Modelo Atual](#9-interpretação-técnica-do-modelo-atual)
- [10. Arquitetura Proposta para o Agente](#10-arquitetura-proposta-para-o-agente)
- [11. Fluxo Macro do Sistema](#11-fluxo-macro-do-sistema)
- [12. Pipeline Multiagente](#12-pipeline-multiagente)
- [13. Classificação Contextual e Taxonomia](#13-classificação-contextual-e-taxonomia)
- [14. Fluxo de Revisão e Publicação](#14-fluxo-de-revisão-e-publicação)
- [15. Modelo de Dados Futuro](#15-modelo-de-dados-futuro)
- [16. Diagrama ER do Modelo Futuro](#16-diagrama-er-do-modelo-futuro)
- [17. Camadas da Aplicação](#17-camadas-da-aplicação)
- [18. Segurança, Robustez e Resiliência](#18-segurança-robustez-e-resiliência)
- [19. Erro como Parte do Fluxo](#19-erro-como-parte-do-fluxo)
- [20. Normalização e Qualidade de Dados](#20-normalização-e-qualidade-de-dados)
- [21. Regras Arquiteturais](#21-regras-arquiteturais)
- [22. Estratégia de Testes e Qualidade](#22-estratégia-de-testes-e-qualidade)
- [23. Roadmap Técnico](#23-roadmap-técnico)
- [24. Próximos Passos](#24-próximos-passos)
- [25. Conclusão](#25-conclusão)

---

# 1. Objetivo

Este documento descreve a arquitetura técnica do serviço de IA responsável por automatizar a geração de questões para a plataforma, convertendo PDFs de provas e gabaritos em questões no formato **Verdadeiro/Falso**, classificadas, explicadas e prontas para revisão.

A proposta do sistema não é apenas gerar texto, mas operar como um **pipeline técnico robusto**, com foco em:

- qualidade dos dados;
- rastreabilidade;
- segurança operacional;
- desacoplamento;
- tolerância a falhas;
- revisão humana;
- integração segura com a base principal.

---

# 2. Contexto do Problema

Atualmente, a criação de questões para a plataforma depende de um processo manual e operacionalmente caro. Esse fluxo envolve:

- leitura de PDFs de provas;
- transcrição das questões;
- adaptação para o formato Verdadeiro/Falso;
- escrita manual do gabarito comentado;
- classificação e publicação no sistema principal.

Esse modelo é lento, pouco escalável e cria gargalos diretos na velocidade de publicação de novos conteúdos.

O objetivo do novo serviço é transformar esse processo em uma esteira automatizada, auditável e segura.

---

# 3. Objetivos do Sistema

O serviço de IA deve ser capaz de:

## 3.1 Processar PDFs de prova e gabarito
Interpretar arquivos enviados, extrair o conteúdo e organizar as questões de forma estruturada.

## 3.2 Classificar metadados automaticamente
Identificar contexto como:

- banca;
- lei;
- artigo;
- disciplina;
- matéria;
- submatéria;
- demais classificações relevantes.

## 3.3 Adaptar questões para o formato da plataforma
Converter o conteúdo original da prova para o formato Verdadeiro/Falso adotado pelo sistema.

## 3.4 Gerar gabarito comentado minimalista
Produzir justificativas curtas, consistentes e alinhadas à regra do produto, priorizando fundamentação legal.

## 3.5 Persistir com segurança
Salvar o resultado em uma camada intermediária de staging, permitindo revisão antes da publicação final.

---

# 4. Visão Geral da Solução

A solução será implementada como um **novo serviço desacoplado**, separado da aplicação principal, utilizando:

- **NestJS + TypeScript**
- **PostgreSQL + PGVector**
- **Redis**
- **Bull/BullMQ**
- **MySQL (somente integração com o serviço principal)**

O serviço atuará como uma esteira assíncrona orientada a jobs e agents especializados, responsável por transformar entradas brutas em saídas estruturadas e classificadas.

---

# 5. Princípios Arquiteturais

A arquitetura proposta deve seguir os seguintes princípios.

## 5.1 Desacoplamento
O serviço de IA não deve depender estruturalmente do framework, nem da implementação de banco ou provider de IA.

## 5.2 Erro como parte do sistema
Falhas de parsing, classificação, busca ou validação não devem ser tratadas como exceções imprevisíveis, mas como estados esperados do fluxo.

## 5.3 Rastreabilidade
Toda questão gerada deve poder ser auditada, explicada e reconstruída a partir de sua origem.

## 5.4 Evolução segura
A arquitetura deve permitir adicionar novos agents, novas fontes e novos fluxos sem reescrever o núcleo.

## 5.5 Segurança por padrão
Uploads, integrações, prompts e persistência devem operar sob princípio de mínimo privilégio e validação explícita.

---

# 6. Estado Atual da Base Principal

A aplicação principal já possui uma base estrutural importante e útil para a integração do agente.

## 6.1 Entidade central
- `questions`

## 6.2 Classificação contextual
- `filter_types`
- `filters`
- `question_filters`

## 6.3 Classificação pedagógica
- `disciplines`
- `matters`
- `sub_matters`
- `question_sub_matters`

## 6.4 Relacionamentos auxiliares
- `client_questions`
- `notebook_questions`
- `mock_questions`

A leitura correta da arquitetura atual é que a plataforma já possui um bom núcleo de questão, mas ainda não possui uma camada própria de staging e rastreabilidade para IA.

---

# 7. Modelo de Dados Atual

## 7.1 Tabela `questions`

A tabela `questions` representa a entidade principal da questão já disponível na plataforma.

### Campos identificados

| Campo | Tipo | Finalidade |
|------|------|------------|
| `id` | int | Identificador da questão |
| `user_id` | uuid | Usuário/admin responsável |
| `title` | text | Título da questão |
| `description` | text | Enunciado principal |
| `explanation` | text | Explicação / gabarito |
| `is_true` | boolean | Resposta correta no modelo V/F |
| `is_accepted` | boolean | Estado de aceitação |
| `is_from_client` | boolean | Origem externa |
| `ia_generated` | boolean | Indicador de geração por IA |
| `reason_refused` | text | Motivo da rejeição |
| `created_at` | timestamp | Criação |
| `updated_at` | timestamp | Atualização |

### Interpretação técnica

Essa tabela já atende bem a publicação final, mas **não é suficiente para suportar todo o pipeline de IA**, porque não contém:

- origem do processamento;
- score de confiança;
- vínculos com evidências;
- rastreio de execução;
- payload bruto do pipeline.

---

# 8. Diagrama ER do Modelo Atual

## 8.1 Relacionamentos principais da base existente

```mermaid
erDiagram
    ADMINS ||--o{ QUESTIONS : creates
    QUESTIONS ||--o{ QUESTION_FILTERS : has
    FILTERS ||--o{ QUESTION_FILTERS : classifies
    FILTER_TYPES ||--o{ FILTERS : categorizes

    DISCIPLINES ||--o{ MATTERS : contains
    MATTERS ||--o{ SUB_MATTERS : contains
    QUESTIONS ||--o{ QUESTION_SUB_MATTERS : has
    SUB_MATTERS ||--o{ QUESTION_SUB_MATTERS : classifies

    QUESTIONS ||--o{ CLIENT_QUESTIONS : assigned_to
    CLIENTS ||--o{ CLIENT_QUESTIONS : owns

    QUESTIONS ||--o{ NOTEBOOK_QUESTIONS : included_in
    NOTEBOOKS ||--o{ NOTEBOOK_QUESTIONS : contains

    QUESTIONS ||--o{ MOCK_QUESTIONS : included_in
    MOCK_EXAMS ||--o{ MOCK_QUESTIONS : contains
```

---

# 9. Interpretação Técnica do Modelo Atual

O principal ponto técnico identificado é que o sistema **não modela o contexto da questão como colunas fixas**.

Ou seja, hoje o sistema **não depende de campos como**:

- `bank_id`
- `year_id`
- `organization_id`
- `position_id`

Em vez disso, utiliza um modelo mais flexível baseado em:

```txt
question_filters → filters → filter_types
```

Essa é uma decisão boa e deve ser mantida.

## 9.1 Classificação contextual

Atributos como:

- banca;
- ano;
- órgão;
- cargo;
- categoria contextual;

devem continuar sendo representados por filtros.

## 9.2 Classificação pedagógica

A estrutura acadêmica da questão é feita via:

```txt
disciplines → matters → sub_matters
```

e vinculada à questão por:

```txt
question_sub_matters
```

Essa separação entre **contexto** e **conteúdo pedagógico** é correta e deve ser preservada no novo serviço.

---

# 10. Arquitetura Proposta para o Agente

O agente deve operar como um **microserviço independente**, conectado à aplicação principal, mas sem depender diretamente da lógica interna dela.

## 10.1 Responsabilidades do novo serviço

O novo serviço será responsável por:

- receber PDFs;
- orquestrar o pipeline de processamento;
- extrair questões;
- classificar contexto;
- buscar fundamentação;
- gerar adaptação V/F;
- gerar explicação;
- validar saída;
- persistir rascunhos;
- publicar conteúdo aprovado.

## 10.2 Serviços externos e internos

A arquitetura envolverá os seguintes componentes:

- **MySQL** → base principal da aplicação
- **PostgreSQL + PGVector** → base de busca semântica e materiais
- **Redis** → cache e filas
- **Bull/BullMQ** → execução assíncrona
- **NestJS** → orquestração e APIs
- **LLM / IA** → classificação, adaptação e geração
- **Dashboard Admin** → revisão e aprovação

---

# 11. Fluxo Macro do Sistema

## 11.1 Fluxo funcional de ponta a ponta

```mermaid
flowchart TD
    A["📄 Upload de PDFs<br/>Prova + Gabarito"] --> B["⏳ Fila de Processamento<br/>(Background)"]
    B --> C["🤖 Pipeline de Agents"]

    C --> D["1. Extração de PDF<br/>Extrai questões"]
    D --> E["2. Classificação<br/>Extrai metadados"]
    E --> F["3. Resolução de IDs<br/>Busca IDs no MySQL + Cache Redis"]
    F --> G["4. Busca Legal<br/>Busca fundamentação"]
    G --> H["5. Adaptação<br/>Reformula para V/F"]
    H --> I["6. Gabarito<br/>Gera justificativa"]
    I --> J["7. Validação<br/>Regras de negócio"]

    J -->|Válida| K["💾 Salva no banco principal"]
    J -->|Inválida| L["❌ Remove da lista / reprova"]

    K --> M["🖥️ Dashboard Admin<br/>Validação Humana"]
    M --> N["✅ Aprova / Rejeita / Edita"]
```

---

# 12. Pipeline Multiagente

A esteira será composta por agentes especializados. Cada agente deve ter uma responsabilidade única, contrato claro e tratamento explícito de falhas.

## 12.1 PDF Extraction Agent
Responsável por:

- ler PDFs de prova e gabarito;
- extrair texto;
- segmentar questões;
- estruturar conteúdo bruto.

### Saída esperada
- lista de questões estruturadas;
- metadados mínimos do documento;
- marcação de falhas de parsing.

---

## 12.2 Classification Agent
Responsável por:

- identificar banca;
- lei;
- artigo;
- disciplina;
- matéria;
- submatéria;
- demais atributos classificáveis.

### Saída esperada
- metadados extraídos;
- nível de confiança;
- campos ambíguos marcados.

---

## 12.3 ID Resolution Agent
Responsável por:

- buscar IDs no MySQL;
- resolver filtros existentes;
- consultar cache Redis;
- reduzir ambiguidade.

### Saída esperada
- IDs de filtros resolvidos;
- IDs de taxonomia pedagógica;
- falhas de resolução quando aplicável.

---

## 12.4 Search Agent
Responsável por:

- buscar fundamentação legal;
- consultar base vetorial;
- complementar com fontes externas controladas.

### Saída esperada
- evidências legais;
- trechos recuperados;
- referências associadas à questão.

---

## 12.5 Adaptation Agent
Responsável por:

- transformar a questão original no formato Verdadeiro/Falso;
- remover ruído;
- reescrever com consistência estrutural.

### Saída esperada
- `title`
- `description`
- `is_true`

---

## 12.6 Answer Key Agent
Responsável por:

- gerar justificativa curta;
- formatar saída final;
- manter explicação compatível com a regra de produto.

### Saída esperada
- `explanation`

---

## 12.7 Validation Agent
Responsável por:

- validar estrutura;
- validar coerência;
- validar campos obrigatórios;
- aplicar regras de negócio.

### Saída esperada
- válida / inválida;
- erros estruturados;
- score final de confiança.

---

# 13. Classificação Contextual e Taxonomia

A classificação da questão é dividida em dois eixos independentes.

## 13.1 Eixo 1 — Contexto

Representa atributos contextuais da questão, como:

- banca;
- ano;
- órgão;
- cargo;
- tipo de prova;
- demais metadados de filtro.

Esse eixo usa:

```txt
filter_types → filters → question_filters
```

## 13.2 Eixo 2 — Conteúdo pedagógico

Representa a estrutura didática da questão:

- disciplina;
- matéria;
- submatéria.

Esse eixo usa:

```txt
disciplines → matters → sub_matters → question_sub_matters
```

Essa separação é importante porque evita mistura entre classificação educacional e contexto operacional.

---

# 14. Fluxo de Revisão e Publicação

O sistema não deve publicar diretamente a saída da IA na tabela final.

A arquitetura correta é:

```txt
agente → staging → revisão → publicação
```

## 14.1 Fluxo recomendado

```mermaid
flowchart TD
    A["🤖 Saída do Agente"] --> B["🗃️ Salvar em question_drafts"]
    B --> C["🏷️ Aplicar filtros"]
    C --> D["📚 Aplicar sub_matters"]
    D --> E["👨‍⚖️ Revisão Editorial"]

    E --> F{"Questão aprovada?"}

    F -->|Sim| G["✅ Inserir em questions"]
    G --> H["🔗 Inserir em question_filters"]
    H --> I["🔗 Inserir em question_sub_matters"]
    I --> J["🚀 Disponibilizar na plataforma"]

    F -->|Não| K["❌ Registrar motivo"]
    K --> L["🔁 Corrigir ou descartar"]
```

---

# 15. Modelo de Dados Futuro

Para suportar IA de forma profissional, será necessário criar uma camada própria de staging e auditoria.

## 15.1 Novas tabelas recomendadas

### `processing_jobs`
Responsável por rastrear o ciclo de execução do pipeline.

### `question_drafts`
Responsável por armazenar a saída do agente antes da aprovação.

### `question_sources`
Responsável por guardar evidências, referências e fontes usadas.

### `draft_filters`
Relaciona drafts com filtros.

### `draft_sub_matters`
Relaciona drafts com classificação pedagógica.

---

## 15.2 Responsabilidades por tabela

### `processing_jobs`
Deve armazenar:

- tipo de entrada;
- status;
- progresso;
- origem do arquivo;
- timestamps;
- payload técnico.

### `question_drafts`
Deve armazenar:

- conteúdo gerado;
- score de confiança;
- status editorial;
- payload bruto;
- vínculo com job;
- revisor.

### `question_sources`
Deve armazenar:

- origem da fundamentação;
- trecho utilizado;
- referência legal;
- tipo de fonte.

---

# 16. Diagrama ER do Modelo Futuro

```mermaid
erDiagram
    PROCESSING_JOBS ||--o{ QUESTION_DRAFTS : generates
    ADMINS ||--o{ QUESTION_DRAFTS : reviews
    QUESTION_DRAFTS ||--o{ QUESTION_SOURCES : has
    QUESTION_DRAFTS ||--o{ DRAFT_FILTERS : classifies
    QUESTION_DRAFTS ||--o{ DRAFT_SUB_MATTERS : classifies

    FILTERS ||--o{ DRAFT_FILTERS : used_by
    SUB_MATTERS ||--o{ DRAFT_SUB_MATTERS : used_by

    QUESTION_DRAFTS ||--o| QUESTIONS : publishes_to
```

---

# 17. Camadas da Aplicação

A implementação deve seguir uma separação clara por responsabilidades.

## 17.1 Domain
Contém:

- entidades;
- value objects;
- regras puras;
- contratos de negócio.

## 17.2 Application
Contém:

- casos de uso;
- orquestração de fluxo;
- comandos e queries;
- serviços de aplicação.

## 17.3 Infrastructure
Contém:

- banco;
- Redis;
- BullMQ;
- LLM providers;
- web clients;
- storage;
- adapters.

## 17.4 Interfaces
Contém:

- controllers;
- DTOs;
- workers;
- listeners;
- APIs.

---

# 18. Segurança, Robustez e Resiliência

A arquitetura deve considerar segurança e robustez desde o início.

## 18.1 Uploads
O sistema deve:

- validar MIME real;
- validar tamanho;
- validar extensão;
- impedir arquivos maliciosos;
- armazenar temporários fora de exposição pública;
- aplicar TTL de limpeza.

## 18.2 Integração com banco principal
A integração com MySQL deve:

- usar credenciais segregadas;
- aplicar princípio de menor privilégio;
- separar leitura e escrita quando possível;
- proteger acesso com conexão segura.

## 18.3 Execução de jobs
Os jobs devem:

- ser idempotentes;
- possuir retry controlado;
- possuir timeout;
- suportar dead-letter;
- registrar falhas estruturadas.

## 18.4 Segurança do uso de IA
O sistema deve:

- validar saída antes de persistir;
- impedir propagação de conteúdo malformado;
- registrar versão de prompt;
- registrar provider/modelo utilizado;
- limitar superfície de prompt injection.

---

# 19. Erro como Parte do Fluxo

Um dos princípios centrais da arquitetura é tratar falhas como parte natural do sistema.

## 19.1 O que isso significa

O sistema não deve assumir que todo PDF será bem formado, que toda classificação será resolvível ou que toda geração será válida.

Em vez disso, cada etapa deve produzir resultados explícitos como:

- sucesso;
- sucesso parcial;
- falha recuperável;
- falha permanente;
- revisão manual necessária.

## 19.2 Benefícios dessa abordagem

Essa modelagem evita:

- pipelines frágeis;
- efeitos cascata;
- exceções silenciosas;
- perda de contexto técnico.

---

# 20. Normalização e Qualidade de Dados

A camada de IA deve normalizar os dados antes de qualquer persistência.

## 20.1 Normalizações obrigatórias

### Texto
- trim;
- normalização de espaços;
- remoção de lixo estrutural;
- normalização de pontuação.

### Metadados
- nomes de banca;
- anos;
- órgãos;
- artigos;
- referências legais.

### Estrutura
- padronização do payload entre agents;
- tipagem consistente;
- campos obrigatórios sempre presentes.

## 20.2 Qualidade mínima de persistência

Nenhum dado deve ser salvo sem:

- schema válido;
- classificação coerente;
- explicação minimamente consistente;
- validação estrutural final.

---

# 21. Regras Arquiteturais

## 21.1 `questions` deve permanecer como entidade final
A tabela final deve representar apenas conteúdo pronto para consumo.

## 21.2 O agente não publica direto
Toda geração passa primeiro por staging.

## 21.3 Contexto continua usando filtros
Não devemos introduzir colunas paralelas para banca, ano, órgão e cargo.

## 21.4 Classificação pedagógica deve ser preservada
A hierarquia acadêmica atual deve ser reutilizada e não substituída.

## 21.5 Toda saída de IA precisa ser auditável
Nada deve entrar em produção sem contexto técnico suficiente para rastreio.

---

# 22. Estratégia de Testes e Qualidade

A estratégia de qualidade deve ser distribuída em múltiplas camadas.

## 22.1 Testes unitários
Para:

- parsers;
- normalizadores;
- resolvedores;
- validadores.

## 22.2 Testes de integração
Para:

- MySQL;
- PostgreSQL;
- Redis;
- filas;
- providers externos.

## 22.3 Testes end-to-end
Para:

- upload;
- processamento;
- geração;
- validação;
- publicação.

## 22.4 Testes de qualidade de IA
Devem existir testes específicos para:

- consistência estrutural;
- aderência ao formato V/F;
- qualidade mínima da explicação;
- regressão de prompts.

---

# 23. Roadmap Técnico

## 23.1 Fase 1 — Infraestrutura
- setup NestJS + TypeScript
- PostgreSQL + PGVector
- conexão MySQL
- Redis + cache
- base inicial de legislação

## 23.2 Fase 2 — Agentes básicos
- PDF Extraction Agent
- Classification Agent
- ID Resolution Agent
- Search Agent

## 23.3 Fase 3 — Agentes avançados
- Adaptation Agent
- Answer Key Agent
- Validation Agent
- Orchestrator Agent

## 23.4 Fase 4 — Processamento assíncrono
- APIs
- filas
- status de jobs
- progresso

## 23.5 Fase 5 — Interface administrativa
- upload
- listagem
- validação
- edição
- métricas

## 23.6 Fase 6 — Hardening
- testes
- logs
- dashboards
- performance
- deploy

---

# 24. Próximos Passos

## Banco
Criar:

- `processing_jobs`
- `question_drafts`
- `question_sources`
- `draft_filters`
- `draft_sub_matters`

## Backend
Implementar:

- pipeline base;
- contratos entre agents;
- resolvedores;
- validadores;
- publicação controlada.

## Produto
Alinhar:

- critérios de aprovação;
- formato final do gabarito;
- estratégia de revisão;
- métricas mínimas do MVP.

---

# 25. Conclusão

A base atual da aplicação já oferece um bom ponto de partida para integração com IA, especialmente por já possuir:

- entidade central de questões;
- modelo flexível de filtros;
- taxonomia pedagógica;
- estrutura editorial básica.

A evolução correta não é reescrever a base, mas **adicionar uma camada de staging, rastreabilidade e orquestração** que permita ao agente operar com segurança.

A arquitetura proposta garante:

- desacoplamento;
- escalabilidade;
- auditabilidade;
- segurança;
- tolerância a falhas;
- e evolução sustentável do sistema.

> O caminho correto é: **pipeline robusto, staging intermediário, classificação reaproveitando o modelo atual e publicação controlada**.
