# 🤖 Case Study Arquitetural — Clube Siga Agent Platform (Multi-Tenant AI Control Plane)

![Status](https://img.shields.io/badge/Status-Em_Produção-10B981?style=for-the-badge)
[![Command Center](https://img.shields.io/badge/Portal_SaaS-agentes.clubesiga.com.br-4F46E5?style=for-the-badge)](https://agentes.clubesiga.com.br)
![Stack](https://img.shields.io/badge/Stack-Python_3.14_%7C_Google_ADK_%7C_LangGraph_%7C_Laravel_13-2563EB?style=for-the-badge)

> **Nota de Governança:** Este repositório público apresenta a arquitetura de engenharia da **Clube Siga Agent Platform**. O repositório principal permanece privado.

---

## 🎯 Visão Geral

A **Clube Siga Agent Platform** é uma plataforma **SaaS Multi-Tenant de Governança e Orquestração de Agentes de IA** projetada para operar dois planos de controle desacoplados sobre a mesma infraestrutura:

1. **Plano Executivo Corporativo (`Beto IA`):** Orquestrador executivo que governa múltiplos produtos, frotas de VPS/Clusters, custos (FinOps), incidentes críticos L4/L5, borda Cloudflare e um **Esquadrão Autônomo de Engenharia de Software**.
2. **Plano Multi-Tenant de Clientes (`Justino`, `Luís` e Agentes Dedicados):** Cada empresa cliente possui seu próprio orquestrador, RAG isolado (`pgvector`), canais sociais (`Meta Graph API`, `WhatsApp`, `Telegram`), políticas de modelos versionadas e limites de orçamento.

---

## 🏗️ Arquitetura de Referência (Control Plane, RAG & Self-Healing DevOps)

```mermaid
flowchart TB
    subgraph Portal["Portal SaaS Multiempresa (Laravel 13 Octane + React 19 / TypeScript)"]
        CC["Command Center do Owner (/platform) & Workspace do Tenant (/dashboard)"]
        UE["Motor FinOps & Unit Economics (Custo Medido vs Franquia & Margem Bruta)"]
        INC["Central de Incidentes Críticos L4/L5 & Aprovações Humanas (HITL)"]
    end

    subgraph Runtime["Runtime Agêntico (Python 3.14 + Google ADK 2.12 + LangGraph 1.2)"]
        OC["OpenClaw 2.0 Gateway + Transcrição Nativa de Áudio/Voz (Vertex AI)"]
        BETO["Beto Executive Orchestrator + Risk Policy Gate (L1 a L5)"]
        SQUAD["Esquadrões Especialistas: Infra/SRE · PO/SM Ágil · Engenharia GitHub · Social 360"]
        BROKER["Credential Broker Isolado (Resolução Opaca Fail-Closed por Tenant)"]
    end

    subgraph Data["Dados, Memória em 3 Níveis & Observabilidade"]
        PG["PostgreSQL 18 + pgvector HNSW (RAG Híbrido + BM25 + RRF k=60)"]
        RD["Redis 8 (Semantic Cache + Filas + Circuit Breaker)"]
        LGTM["Grafana Unified Alerting ➔ Webhook SRE ➔ PO Issue ➔ Gate Humano ➔ PR Automático"]
    end

    CC & UE & INC <--> BETO
    OC --> BETO --> SQUAD
    SQUAD --> BROKER
    SQUAD --> PG & RD
    LGTM --> INC
```

---

## 🛡️ Diferenciais de Engenharia Sênior

- **Isolamento Multi-Tenant *Fail-Closed*:** Nenhuma operação de memória, RAG, modelo ou canal social executa sem `tenant_id`, política versionada e credencial escopada válida no `credential-broker`.
- **Defesa em 5 Camadas contra *Indirect Prompt Injection*:** Sanitização de documentos, quarentena em Cloudflare R2, avaliação automatizada da **Tríade RAG** (*Context Relevance*, *Groundedness*, *Answer Relevance*) e rollback de conhecimento em 1 clique.
- **Pipeline Autônomo DevOps ➔ PO ➔ Gate Humano ➔ Developer PR:**
  1. O **Agente DevOps/SRE** inspeciona logs de produção e recebe webhooks do **Grafana Unified Alerting**;
  2. O **Agente Product Owner (PO)** valida o *Definition of Ready (DoR)*, sanitiza segredos e abre a tarefa no **GitHub Issues** (ou Portal/Notion);
  3. A **Central de Incidentes L4/L5** exige confirmação humana explícita do Owner;
  4. Após aprovado, o **Agente Desenvolvedor** cria uma branch isolada (`fix/beto-issue-N`), aplica o patch + teste automatizado via *GitHub Git Data API* (sem clone em disco e sem shell) e abre o Pull Request para `dev`.
- **Qualidade & Arquitetura Limpa:** Limite estrito de $\le 300$ linhas por módulo Python de produção, tipagem estrita (`mypy --strict`), `Ruff`, `pytest` (+560 testes) e `Pest PHP` + `PHPStan Level 7` (+135 testes).
