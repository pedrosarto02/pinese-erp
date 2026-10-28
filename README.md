# Pinese ERP

ERP próprio, multiempresa e multiconta, para vender no Mercado Livre. Substitui o Olist ERP e o Mercado Turbo e é o dono da integração com o ML, que também alimenta o agente de IA de atendimento.

## Estrutura
- `erp/docs/`: contexto do ERP
  - `01-mapa-de-features-olist.md`: 177 features mapeadas (engenharia reversa do Olist + Bling, Omie, hubs e API do ML)
  - `02-contexto-erp.md`: operação, dores, escopo MVP / Fase 2 / Fora, regras de negócio, integração, riscos
  - `_entrevista.md`: respostas da entrevista de escopo
  - `_pesquisa/`: pesquisas de apoio
  - `contexto-erp-pinese.html`: página visual de validação
- `erp/prototipo/`: protótipo HTML modular
  - `pinese-erp.html`: protótipo consolidado (só o que já foi aprovado)
  - `00-layout-login.html`: módulo 0 (aprovado)
  - `m02-empresas/`: 10 versões do módulo 2 (Empresas e contas ML), aguardando escolha
- `agent/docs/`: documentos do agente de IA de atendimento
- `.claude/skills/`: skill REA usada na engenharia reversa

## Fase atual
Contexto validado em 27/10/2026. Protótipo HTML módulo a módulo; nenhum código de produção nem banco de dados ainda.
