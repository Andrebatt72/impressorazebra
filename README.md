# 📘 Documentação Oficial — Arquitetura de Impressão Zebra (Saveinformatica)

**Versão 1.0 — Formato Markdown **  
**Data:** 18/02/2026  
**Autor:** André Di Battista 

---

## 1. 🎯 Resumo Executivo

A Saveinformatica necessita substituir o aplicativo local “**savecloud**”, que acessa diretamente o banco de dados de produção e cria tabelas temporárias com privilégios elevados, por uma arquitetura **segura**, **multicliente** e **totalmente desacoplada** do banco.

A nova solução utiliza:

- **Aplicativo Cloud (App A)** — responsável por orquestrar jobs, registrar impressoras, e atualizar o ERP.  
- **Agente Local (App B)** — rodando no cliente, responsável por imprimir localmente o ZPL/PRN e confirmar execução.

---

## 2. 🧱 Arquitetura Geral da Solução

### 2.1 Visão Geral — Diagrama (Mermaid)

```mermaid
flowchart LR
    ERP["ERP - App Service -FrontEnd- Backend-Job Sql"]
    DB["Azure SQL - Tabelas SPOOL_ZEBRA e portalImpressora"]
    AppA["App A - Cloud API - Orquestracao e Multicliente"]
    AppB["App B - Agente Local - Cliente"]
    Zebra["Impressora Zebra - ZPL via TCP 9100"]

    ERP -->|Gera Job P| DB
    DB -->|Consulta via DAO interno| AppA
    AppA -->|Distribui Job P| AppB
    AppB -->|Envia ZPL TCP 9100| Zebra
    AppB -->|ACK OK ou ERRO| AppA
    AppA -->|Atualiza status E ou ERRO| DB
```

---

## 3. 🖨️ Fluxo Completo de Impressão

### 3.1 Diagrama de Fluxo do Job

```mermaid
sequenceDiagram
    participant ERP as ERP
    participant DB as Azure SQL
    participant AppA as App A (Cloud)
    participant AppB as App B (Cliente)
    participant Printer as Zebra (TCP 9100)

    ERP->>DB: Insere job em SPOOL_ZEBRA (status=P)
    AppA->>DB: Le jobs novos (P)
    AppA->>AppB: Envia job para o cliente correto
    AppB->>Printer: Envia ZPL via socket TCP 9100
    Printer-->>AppB: Impressao concluida
    AppB->>AppA: ACK (OK ou ERRO)
    AppA->>DB: Atualiza status (X -E ou F)
```

---

## 4. 🏗️ Componentes da Solução

### 4.1 App A — Cloud

Responsável por:

- Autenticação multicliente.  
- Registro e health das impressoras.  
- Consulta da tabela `SPOOL_ZEBRA` (status = P).  
- Distribuição de jobs para o agente local correto.  
- Recebimento do ACK local e atualização do banco.

### 4.2 App B — Agente Local

Responsável por:

- Registrar a impressora local com o App A.  
- Solicitar jobs pendentes.  
- Imprimir via **ZPL na porta TCP 9100** (caminho oficial Zebra).  
- Enviar ACK ao App A com status final.  
- Implementar retries/queue local.

> **Nota sobre ZPL/PRN:** para Zebra, “PRN” normalmente significa o **conteúdo ZPL bruto** que pode ser enviado diretamente para a impressora (rede → TCP 9100) ou por utilitários/driver; é a forma mais simples e performática em ambiente de rede.

---

## 5. 🛡️ Segurança e Multitenancy

```mermaid
flowchart TD
    subgraph Tenant_1
        A1[Agente Local 1]
        P1[Impressora 1]
    end

    subgraph Tenant_2
        A2[Agente Local 2]
        P2[Impressora 2]
    end

    AppA[App A - Cloud API]

    A1 -->|Autenticacao JWT / chave| AppA
    A2 -->|Autenticacao JWT / chave| AppA
    AppA -->|Jobs do Tenant 1| A1
    AppA -->|Jobs do Tenant 2| A2
```

**Regras principais:**

- Nenhum cliente acessa o banco diretamente; todo IO passa pela API (App A).  
- Isolamento forte por **tenant** (escopos e claims).  
- Impressoras Zebra com protocolos legados desativados (via PrintSecure/SGD):  
  - `ip.telnet.enable=off`  
  - `ip.ftp.enable=off`  
  - `ip.snmp.enable=off`

---

## 6. 🧪 Observabilidade e Confiabilidade

- Idempotência por **CHAVE** única (já existente no SPOOL).  
- Retentativas com backoff exponencial no agente.  
- Dead-letter queue no App A.  
- Logs estruturados: tenant, impressora, job, latência, erros.  
- Métricas: tempo até imprimir, falhas, status das impressoras.

---


## 7. 📮 Contatos

- **Responsável técnico:** André Di Battista  
- **Stack:** Azure App Service (App A), Agente Local (Windows/Linux), Impressoras Zebra (ZPL/9100)

