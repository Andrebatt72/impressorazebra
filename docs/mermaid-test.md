---
title: Teste Mermaid
---

<!-- Carrega o Mermaid direto do CDN -->
https://cdn.jsdelivr.net/npm/mermaid@10/dist/mermaid.min.js</script>
<script>
  mermaid.initialize({ startOnLoad: true });
</script>

# Teste Mermaid

Se este gráfico aparecer desenhado (e não como texto), o Mermaid está funcionando corretamente no GitHub Pages.

---

## 🔹 Diagrama de Fluxo

```mermaid
flowchart LR
    A[Início] --> B[Processar]
    B --> C{OK?}
    C -- Sim --> D[Finalizar]
    C -- Não --> B
```

---

## 🔹 Diagrama de Sequência

```mermaid
sequenceDiagram
    participant U as Usuário
    participant S as Site
    U->>S: Acessa página
    S-->>U: Renderiza Mermaid
```
