---
title: Teste Mermaid
---

{% include head-custom.html %}

# Teste Mermaid

```mermaid
flowchart LR
    A[Início] --> B[Processar]
    B --> C{OK?}
    C -- Sim --> D[Finalizar]
    C -- Não --> B
```
