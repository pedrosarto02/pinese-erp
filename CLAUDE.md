# Pinese ERP: instruções do projeto

ERP próprio, **multiempresa e multiconta**, para vender no **Mercado Livre**. Substitui o **Olist ERP e o Mercado Turbo** e é o **dono da integração com o ML** (OAuth, tokens, webhooks, fila), camada que também alimenta o **agente de IA de atendimento** (mesmo monorepo, mesmo banco, painel único). Cliente/mentorado: Pedro (GitHub `pedrosarto02`).

## Regras deste projeto (valem sobre o CLAUDE.md global)
- **Não usar DevContext nem vault** neste projeto. Tudo fica em arquivos aqui.
- **Fase atual: protótipo HTML.** Nada de código de produção, schema ou banco até o protótipo do módulo ser aprovado.
- **Não perguntar "aprova?" sobre escopo.** O cliente aprova **vendo o módulo em HTML**. Perguntas de contexto (fatos da operação) continuam ok.
- Responder sempre em português do Brasil.
- Commit/push só quando o cliente pedir. O repositório local já está configurado com a identidade do Pedro (user.name/email locais) e com remote `github.com/pedrosarto02/pinese-erp`.

## Ler nesta ordem ao começar
1. `erp/docs/02-contexto-erp.md`: **fonte da verdade** (operação, escopo MVP/Fase 2/Fora, regras de negócio, integração ML + agente, riscos, perguntas pendentes, ordem dos módulos).
2. `erp/prototipo/README.md`: como o protótipo cresce e regras técnicas de todo módulo.
3. `erp/docs/_entrevista.md`: respostas do cliente (o "porquê" das decisões).
4. Sob demanda: `erp/docs/01-mapa-de-features-olist.md` (177 features com IDs CAD-, EST-, VEN-…), `erp/docs/_pesquisa/` (Olist via REA, Bling/Omie, hubs, API ML), `agent/docs/` (agente de atendimento).

## Processo por módulo (definido pelo cliente)
1. Escrever `erp/prototipo/mNN-nome/_briefing.md` (contexto, escopo funcional, comportamento obrigatório, regras técnicas, 10 conceitos de UX diferentes). Usar `m02-empresas/_briefing.md` como modelo.
2. Gerar **10 versões** via **subagentes em paralelo** (2 por agente). Cada agente usa **sua própria subpasta no scratchpad** (ex.: `scratchpad\v0102\`), porque nomes genéricos na raiz colidem.
3. Publicar as 10 como Artifacts e registrar os links em `mNN-nome/_links.md`.
4. O cliente escolhe os elementos de cada versão → montar a **versão final** do módulo.
5. Incorporar a final ao consolidado `erp/prototipo/pinese-erp.html` (substitui o placeholder da rota) e republicar o consolidado.
6. Atualizar o status no `erp/prototipo/README.md` e seguir para o próximo módulo.

Toda versão parte do shell aprovado (cópia do consolidado), sem backend, mas com comportamento de produção: toasts, modais, validações, métricas, estados vazio/carregando/erro, responsivo 360/390/768/1366.

## Onde paramos (28/10/2026)
- Módulo 0 (layout, login, menu por perfil): ✅ aprovado, é o `pinese-erp.html`.
- **Módulo 2 (Empresas e contas ML): 10 versões prontas em `erp/prototipo/m02-empresas/v01…v10.html`, aguardando o cliente escolher os elementos.** Links em `m02-empresas/_links.md`.
  - Ao montar a final, padronizar no shell o que algumas versões acrescentaram: cores `--m2-acc-5…8` para empresas novas, contador dinâmico do sino/selo de integrações, rota `#empresas/<id>`.
  - Confirmar com o cliente a regra da validade do refresh token (versões contaram 180 dias desde a autorização).
- Próximos módulos (ordem): 3 Produtos + base de conhecimento → 4 Anúncios (vincular, publicar, replicar) → 5 Estoque e Full → 6 Pedidos → 7 Separação/etiquetas/expedição → 8 Devoluções → 9 Atendimento (inbox) → 10 Compras e reposição → 11 Financeiro (repasses, DRE) → 12 Dashboards e margem → 13 Fiscal e contábil → 14 Configurações e usuários.
- Depois do protótipo completo: roadmap/tarefas → schema Supabase → código módulo a módulo (Next.js + Supabase, tudo em Vercel + Supabase Cloud).

## Fatos que mudam o desenho (resumo; detalhe no 02)
- 4 CNPJs / 4 contas ML (principal **Mercado Mestre**; 3 Simples + 1 MEI), contas ilimitadas criadas pelo admin.
- Galpão único: estoque físico compartilhado, propriedade fiscal por CNPJ; transferência entre empresas com ou sem NF (escolha a cada vez, as sem NF ficam como pendência).
- MVP: Faturador do ML emite todas as NF e o ERP importa; **emissor próprio e relatórios contábeis na Fase 2** (base contábil em partidas dobradas já no MVP).
- Perfis: Dono, Logística (sem custo/margem), Contador (leitura + exportação), Tributário (confere todas as NF, fechamento).
- Impressora TSC TE200 imprime ZPL. Despesas comuns vão para a principal.
- Pendências com o contador: banco por empresa ou único; plano de contas. Flex em aberto.

## Observações de ambiente
- Os Artifacts publicados até 27/10 foram criados em outra conta do claude.ai. Se não der para atualizar um link existente, publicar um novo e registrar o link no README/`_links.md`.
- O repositório no GitHub foi criado público temporariamente; o cliente vai torná-lo privado.
