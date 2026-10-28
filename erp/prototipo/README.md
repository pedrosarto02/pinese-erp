# Protótipo do ERP Pinese

## Como o protótipo cresce
- `pinese-erp.html` é o **protótipo consolidado**: o sistema inteiro, só com o que já foi aprovado. Cada módulo aprovado entra nele no lugar do placeholder correspondente. No fim da prototipagem, este arquivo é o sistema completo, só visual.
- `00-layout-login.html` é o módulo 0 (shell, login, tokens e componentes). **Aprovado em 27/10/2026.** É a base de tudo.
- `mNN-nome/` guarda as variações de cada módulo (`v01.html` … `v10.html`). O cliente escolhe os elementos de cada uma e é montada a versão final, que é então incorporada ao consolidado.

## Regras para qualquer módulo
- Partir **sempre** do shell aprovado (cópia do consolidado atual): mesmos tokens, menu, topo, seletor de empresa, alertas, busca, perfis e responsividade. Pode acrescentar componentes, não alterar os existentes.
- Sem backend: estado em memória (JS), com `localStorage` só para conveniências. Comportamento de produção: validações, toasts, modais, confirmações, estados vazio/carregando/erro, métricas.
- Restrições de artifact: nada de `alert/confirm/prompt`, `window.print`, downloads ou iframes; `localStorage` sempre em try/catch; só os CDNs permitidos (de preferência nenhum).
- Breakpoints: 360 / 390 / 768 / 1366 px; zero rolagem horizontal da página; alvos de toque ≥ 40 px.
- pt-BR, BRL, máscaras (CNPJ, CEP, telefone, moeda), dados de exemplo marcados como exemplo.

## Status
| Módulo | Status | Link |
|---|---|---|
| 0 · Layout, login, menu | ✅ aprovado | https://claude.ai/artifact/WfFkKkLrL4bnsF8vU7z1N9 (consolidado) |
| 2 · Empresas e contas ML | 🔄 10 variações em `m02-empresas/` | — |
