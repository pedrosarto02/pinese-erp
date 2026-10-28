# 02 · Contexto do ERP Pinese

> **Fase:** contexto **validado pelo cliente em 27/10/2026**. Próxima etapa: protótipo HTML modular (sem código de produção nem schema até a aprovação de cada módulo).
> **Versão:** v1 · 27/10/2026 · **Base:** `01-mapa-de-features.md` (engenharia reversa do Olist + mercado) e `_entrevista.md` (9 rodadas).
> Os IDs entre parênteses (CAD-03, FIN-07…) apontam para o mapa de features.

---

## 1. Visão em uma frase

Um ERP próprio, **multiempresa e multiconta** (sem limite de CNPJs e contas), feito para vender no **Mercado Livre** primeiro. Ele **substitui o Olist ERP e o Mercado Turbo** e é o **dono da integração com o ML**: a mesma camada alimenta o **agente de IA de atendimento**, que roda no mesmo painel.

## 2. A operação

| Item | Situação |
|---|---|
| Empresas | **4 CNPJs**: 3 no Simples Nacional e 1 MEI (todas devem migrar para o Simples). Cada empresa tem **1 conta ML**. |
| Conta principal | **Mercado Mestre**. As outras 3 são secundárias. Só a principal está no Olist hoje. |
| Por que várias contas | Reduzir impostos e **aparecer mais vezes** no ML (catálogo **parcialmente igual** entre contas). |
| Canais que vendem | **Só Mercado Livre**: Full + coleta/agência. Shopee, Amazon, Magalu e TikTok estão integrados no Olist, mas não vendem. Futuro: **B2B, licitação, site próprio**. |
| Volume | ~**1.079 pedidos/mês** (~36/dia) só na conta conectada ao Olist; ~116 anúncios ativos para 45 produtos. |
| Catálogo | 45 produtos (37 simples, 7 kits multipack, 1 com variação de voltagem). Nicho: casa, jardim, decoração e ferramentas. |
| Estoque físico | **Um galpão para todas as empresas** + o Full de cada conta. Boa parte do estoque vive no Full, sempre com reserva no galpão. |
| Entrada de mercadoria | Mista: compra com NF por CNPJ **e** transferência entre empresas sem NF (hoje lançada como perda na origem). |
| Notas fiscais | **100% emitidas pelo Faturador do ML** (Full e coleta). O Olist só importa as cerca de 2.000 notas por mês. O desejo é emitir tudo pelo ERP. |
| Financeiro e margem | **Mercado Turbo** (margem por venda e anúncio, repasses MP, métricas de anúncio e concorrência, fluxo de caixa, contas a pagar e a receber, DRE, BP). O financeiro do Olist está parado. |
| Contas financeiras | 1 Mercado Pago por empresa + banco (1 por empresa, ou 1 único para todas, ainda não decidido). |
| Equipe | Dono + 1 pessoa (logística). A contabilidade é externa e acessa o ERP. |
| Rotina diária | Separar, embalar e imprimir etiquetas (**impressora TSC TE200**) · preparar reposição do Full · tratar devoluções e conferir estoque. O atendimento fica com a IA. |
| Devoluções | Processo grande: 822 registros no Olist, com depósito próprio "Devoluções". |

## 3. Dores que o ERP precisa resolver

1. **Mensalidade do Olist** (plano Domine, R$ 948/mês) **+ Mercado Turbo**, com funções importantes presas a planos caros e bloqueio de escrita quando a conta passa do limite de armazenamento.
2. **Integração ML instável e opaca**: erros de mapeamento, token da Shopee expirado sem alerta, mudanças sem aviso, relatos de pedidos Full duplicados e de estoque zerado ou somado errado (Full + galpão).
3. **Só 1 das 4 contas está integrada.** As outras ficam fora do controle de estoque, pedidos e margem.
4. **Margem real e repasses** fora do ERP, numa ferramenta à parte; não existe conferência de repasse × pedido.
5. **Estoque de segurança único por integração**, sem regra por conta ou anúncio.
6. **Sem visão por empresa** (CNPJ) de estoque, faturamento, impostos e contabilidade.
7. **O agente de atendimento não tem fonte de conhecimento**: o Olist não guarda manual, vídeo e FAQ por produto.
8. **Reposição do Full no feeling**, sem sugestão por giro e cobertura.

## 4. Matriz de escopo

Critério: **MVP** é o mínimo para desligar o Olist **e** o Mercado Turbo e para alimentar o agente; **Fase 2** vem logo depois; **Fora** é o que a sua operação não precisa ou que outra ferramenta resolve. Decidido na entrevista; os itens sem pergunta específica foram classificados pelo consultor com a justificativa ao lado.

### 4.1 MVP

| Módulo | Entra no MVP | Por quê |
|---|---|---|
| **Núcleo / plataforma** | Empresas (CNPJs) e contas de marketplace **sem limite** (CAD-28) · usuários e **4 perfis** (dono, logística, contador, tributário) com escopo por empresa (OUT-01/02) · auditoria geral (OUT-04) · notificações e alertas (OUT-08) · exportação/backup (OUT-11) · tema claro/escuro (OUT-14) · **migração do Olist** (OUT-15) | 4 CNPJs desde o primeiro dia; o contador e o perfil tributário já acessam o sistema; sair do Olist exige migração. |
| **Cadastros** | Produto simples, variações e kit (CAD-01/02/03) · modelo **Produto → User Product → N anúncios em N contas** (06) · dados fiscais (07) · ficha técnica (09) · **base de conhecimento do produto** (manual PDF, vídeo, garantia, medidas, materiais, FAQ) (10) · imagens (11) · descrição/SEO (13) · categorias (14) · marcas e tags (15) · custos e histórico (19) · fornecedores do produto (20) · contatos (22) · cliente automático via pedido (23) · intermediadores (27) · planilhas e ações em lote (29) · importar produtos a partir dos anúncios (30) | A base que o agente consome e a origem do vínculo com os anúncios das 4 contas. |
| **Estoque** | Saldo por **local físico × empresa dona** (EST-01/04) · lançamentos e histórico auditado (02/03) · transferências, inclusive **entre empresas com ou sem NF** (05) · reserva (06) · gatilhos de baixa e de entrada (07/08) · política de negativo (09) · **segurança por anúncio/conta** (10) · **sync com todas as contas** (11) · espelho e histórico do Full (12/13) · **conferência Full × ERP** (14) · inventário (15) · custo médio (16) · localização (18) · alertas (19) · **triagem de devoluções** (20) · pausa/reativação (22) · multiorigem ML (23) | Pool físico único com propriedade fiscal por CNPJ; corrige as dores de estoque do Olist. |
| **Vendas e pedidos** | Importação automática das 4 contas (VEN-01) · fila de pedidos crus/reprocesso (02) · carrinhos (03) · situações e mapeamento (04) · automações entre situações (05) · marcadores (07) · prazo de despacho (08) · cancelamentos (09) · **devoluções completas** (10) · reclamações (11) · descontos/cupons (15) · **painel completo de atendimento** (inbox, histórico, contexto do pedido, anexos, SLA; é a interface do agente) (12/13) | Coração da operação e interface do agente. |
| **Fiscal** | **Importar todas as notas do ML** (venda, devolução, remessa) (FIS-01) · **alimentar o Faturador** (dados fiscais + regras tributárias por empresa) (02) · naturezas de operação e regras por empresa (06) · notas de entrada por XML (10) · exportação para contador/tributário (17) · **base contábil**: cada evento gera lançamento em partidas dobradas (plano de contas por empresa) | No MVP o Faturador do ML continua emitindo; o ERP registra e contabiliza tudo desde o primeiro dia (sem retrabalho de histórico). |
| **Compras** | Ordem de compra simples (COM-01) · entrada por XML (02) · custo de reposição na entrada (04) · **sugestão de compra pelo ponto de pedido de cada conta**, consolidada na reposição do galpão (05) · **sugestão de reposição do Full** por conta (06) | Pedido explícito: "quero que o ERP sugira". |
| **Financeiro** | Contas a pagar e a receber (FIN-01/02) · contas por empresa (MP + banco, compartilhado ou exclusivo) (03) · categorias (04) · **taxas do ML gravadas em cada venda** (06) · **conferência de repasses do Mercado Pago** (07) · **conferência das cobranças do ML** (08) · **DRE gerencial** por empresa, conta, produto e consolidado (10) · fluxo de caixa (11) · custos do Full (13) · simulador de custo de vender (14) · fechamento de período (15) | Substitui o Mercado Turbo. Despesas comuns são lançadas na principal (Mercado Mestre). |
| **Marketplaces / hub** | **Conexão das contas com token broker** (MKT-01) · **receptor de webhooks + fila de eventos** (02) · recuperação/reconciliação (03) · gestão de anúncios (04) · vínculo anúncio ↔ produto (05) · **publicar anúncio a partir do produto** (06) · **replicar entre contas com diferenciação** (07) · edição em massa (08) · sync de preço e regras por conta (09) · calculadora e proteção de margem (10) · saúde dos anúncios (15) · **monitor de integração** (16) · mapeamentos (17) · sugestão de preço ML (18) · **métricas de anúncio: visitas, conversão, Product Ads** (19) · reputação por conta (20) | "Estar mais vezes anunciado" é o objetivo; as métricas substituem o Mercado Turbo. |
| **Logística** | Separação (LOG-01) · **conferência com bipagem** (02) · **etiquetas em lote na TSC TE200**, impressão direta (03/12) · agrupamento de coleta e romaneio (04) · coleta/agência (05) · Full (07) · rastreio (08) · devoluções/reversa (09) | Rotina diária do galpão. |
| **Relatórios / BI** | **Margem de contribuição por pedido, SKU, anúncio, conta, empresa e canal** (BI-01) · dashboard por conta/empresa (02) · curva ABC (03) · giro e cobertura (04) · ruptura (05) · relatórios de exceção (produto sem anúncio, Full pausado, erro de vínculo) (06) · **devoluções por motivo, custo e SKU** (07) · saldos em data passada (10) | Margem real é a dor nº 1 que o Mercado Turbo cobre hoje. |

### 4.2 Fase 2

| Item | Por quê não no MVP |
|---|---|
| **Emissor NF-e próprio** (FIS-03), envio do XML ao ML (04), emissão após liberação da etiqueta (05), certificado A1 (07), eventos fiscais (08), emissão em lote (09), DF-e/manifestação (11), DANFE simplificado (12), DIFAL (14), **reforma tributária IBS/CBS** (15), correção de rejeição com IA (18) | Escolha do cliente: **construir o emissor próprio** (sem API de terceiros), mas sem travar o MVP. Até lá o Faturador do ML emite. |
| **Relatórios contábeis**: DRE e BP contábeis, conferência com o contador | A base (lançamentos) nasce no MVP; os relatórios ficam para depois. |
| Kit com variações (CAD-04), sugestão de NCM por IA (08), tratamento de imagem por IA (12), tabela de medidas (16), campos customizados (17), **listas de preço** (18, vai servir ao B2B), embalagens (21), consulta CNPJ (24), validação pré-publicação (31) | Úteis, não bloqueiam a saída do Olist. |
| Lotes e validade (EST-17) | Não usado hoje. |
| **Pedido manual** (VEN-06), feedback de venda (14), alerta de fraude (16), faturamento parcial/agrupado (21) | Ainda não há venda fora do marketplace. |
| Conferência de recebimento com bipagem (COM-03) | Compras ainda são raras. |
| Conciliação bancária OFX (FIN-09), centro de custo (05) | O Mercado Pago concentra o dinheiro. |
| Central de promoções ML (MKT-11), campanhas do ERP (12), catálogo (13), kit virtual (14), **novos canais: B2B, licitação, site próprio, Shopee etc.** (21) | Depois da base estável no ML. |
| **Flex** (LOG-06), horários de despacho (10), split de pacote (13) | Não usado hoje. |
| Vendas por UF/horário (BI-08), relatório personalizado (09), MCP/BI externo (11) | Conforto analítico. |
| 2FA e restrições de acesso (OUT-03), API pública/webhooks de saída (06), MCP do ERP (07), assistente de IA no ERP (09), envio de documentos por e-mail/WhatsApp (12) | O agente usa o mesmo banco: não precisa de API pública no MVP. **Recomendação:** 2FA do dono já no MVP, por causa dos dados financeiros. |

### 4.3 Fora

| Item | Motivo |
|---|---|
| Produção/BOM (CAD-05), limite de crédito (25), vendedores (26), propostas (VEN-17), CRM (18), comissões/metas (19), PDV (20), OS/contratos (22) | Não fazem parte da operação (tudo vazio no Olist). As propostas voltam se a licitação/B2B pedir. |
| NFC-e, SAT, NFS-e, CT-e, MDF-e (FIS-13), SPED (16) | Não há varejo físico nem transporte próprio; SPED fica com o contador. |
| Requisição/aprovação de compra (COM-07), nota de importação (08) | Equipe de 2; não importa diretamente. |
| Boletos/CNAB (FIN-12) | Não há venda a prazo. |
| Correios/transportadoras próprias (LOG-11) | A logística é do ML; volta com o site próprio. |
| Loja de extensões (OUT-13) | ERP próprio não precisa de cobrança por módulo. |

## 5. Regras de negócio

### Empresas, contas e permissões
1. O ERP aceita **N empresas** (CNPJ, regime: MEI/Simples/Normal, UF) e **N contas de marketplace** por empresa. Hoje são 4 empresas × 1 conta ML.
2. Uma conta é marcada como **principal** (Mercado Mestre) e as demais como **secundárias**. A principal recebe as despesas comuns.
3. Perfis: **Dono** (tudo) · **Logística** (separação, etiquetas, Full, devoluções, estoque; **não vê custo nem margem**) · **Contador** (leitura + exportação) · **Tributário** (analisa NFs e fechamento contábil, conduz a NF de devolução; sem editar o resto). As permissões valem por empresa.

### Produto e anúncios
4. **Um produto (SKU) pode ter N anúncios em N contas.** O vínculo é o `SELLER_SKU` + o modelo User Product do ML.
5. Kits são estoque derivado: disponível = mín(saldo do componente ÷ quantidade). A venda baixa os componentes.
6. A ficha do produto (atributos, medidas, materiais, o que acompanha, garantia, manual PDF, vídeo, FAQ) é a **base de conhecimento oficial do agente**.
7. Replicar um anúncio para outra conta **exige diferenciação** (título, tipo de anúncio e/ou frete) e respeita limites de ritmo, para não cair na política de duplicados nem nas sanções por ação em massa.

### Estoque
8. Duas dimensões: **local** (Galpão, Devoluções, Full da conta X) × **empresa dona**. O galpão é físico e único; a propriedade é fiscal e separada.
9. **Saldo publicado** em cada anúncio = disponível do galpão (físico − reservas) − **estoque de segurança** do anúncio/conta. É o mesmo saldo em todas as contas, atualizado a cada venda.
10. O **estoque Full é do ML** (só leitura via API). O ERP espelha, guarda o histórico e aponta divergências; o envio de reposição é criado no painel do ML.
11. Quando uma conta vende uma unidade de outra empresa, a transferência é registrada **com NF entre empresas** ou como **movimentação interna sem NF**, escolhida a cada vez. As sem NF ficam **marcadas como pendência fiscal**.
12. A reserva acontece na entrada do pedido; a baixa no gatilho configurado (hoje: "marcar como enviado"); o cancelamento devolve a reserva.
13. Devolução entra no depósito **Devoluções**, passa por triagem (vendável / avaria / descarte) e é transferida.

### Pedidos, fiscal e financeiro
14. Pedidos chegam por webhook, são **idempotentes** (pack → orders → envio) e são reconciliados periodicamente. O ML só guarda 12 meses; **o ERP é a fonte histórica**.
15. MVP: o **Faturador do ML emite todas as NFs** (Full obrigatório); o ERP envia os dados fiscais e as regras tributárias de cada empresa e **importa todas as notas**.
16. Cada venda registra **bruto, comissão, custo fixo, frete do vendedor, descontos, impostos e custo do produto**, o que dá a **margem de contribuição**.
17. O contas a receber nasce da venda e é **conciliado com a liberação no Mercado Pago**. Diferenças viram alerta.
18. Todo evento financeiro e de estoque gera **lançamento contábil em partidas dobradas** (plano de contas por empresa) desde o MVP.
19. Dinheiro e estoque são **sempre calculados no servidor**, em transações atômicas.

## 6. Integração com o Mercado Livre e com o agente de atendimento

### Princípios
- **1 aplicação ML por titular**: o ERP é o dono dela (OAuth, tokens, webhook, fila). O agente **nunca** renova token nem recebe webhook diretamente.
- **Mesmo monorepo, mesmo banco (Supabase), painel único.** Schemas separados por domínio (`erp`, `ml`, `agente`) com tipos e esquemas Zod compartilhados.
- O agente lê produtos, base de conhecimento, pedidos, envios e reclamações **direto do banco** e consome os eventos `questions`, `messages` e `post_purchase` da mesma fila.

### Fluxo técnico
1. **Conexão**: o admin de cada conta autoriza → o ERP guarda os tokens criptografados por `user_id` → o **token broker** renova antes de vencer, **com trava por conta** (o refresh é de uso único) → se receber `invalid_grant`, marca "reconectar" e avisa.
2. **Webhook**: receptor mínimo (valida `application_id`/`user_id`, grava na fila, responde 200 em < 500 ms).
3. **Workers** consomem a fila: fazem GET no recurso, deduplicam, aplicam no ERP e roteiam por tópico para o módulo certo (pedidos, estoque, anúncios, NF, atendimento/agente).
4. **Reconciliação**: `missed_feeds` (≤ 2 dias) + varreduras periódicas (pedidos alterados, envios, reclamações abertas, perguntas sem resposta, saldo Full).
5. **Escritas no ML** (estoque, preço, anúncio, dados fiscais) saem por uma fila de saída com **limite de ritmo por conta e por recurso** e retentativa.
6. **Testes**: não existe sandbox. Usar usuários de teste em produção + respostas gravadas para os testes automatizados.

### Hospedagem
- **Tudo em Vercel + Supabase Cloud** (escolha do cliente). Fila em Postgres (pgmq), agendamentos com **pg_cron**, workers em Supabase Edge Functions e/ou Vercel Functions/Cron. Arquivos (manuais, vídeos, imagens) no Supabase Storage com URLs assinadas; as imagens enviadas ao ML usam URLs públicas estáveis.

## 7. Riscos

| # | Risco | Impacto | Mitigação |
|---|---|---|---|
| 1 | **Fiscal/tributário**: várias empresas do mesmo dono, com galpão e estoque compartilhados, para reduzir imposto podem ser tratadas como **grupo econômico de fato** (exclusão do Simples, cobrança retroativa). | Alto | Validar o modelo com contador/advogado. O ERP registra a propriedade por CNPJ e as transferências, o que dá sustentação documental. |
| 2 | **Transferências entre empresas sem NF** | Alto | Modo "com NF" a um clique; as sem NF ficam como pendência visível; relatório para o contador. |
| 3 | **Política de duplicados / ações em massa do ML** entre contas | Alto (suspensão de conta) | Replicação com diferenciação obrigatória, limite de ritmo, e confirmar se as contas são "vinculadas" para o ML. |
| 4 | **Escopo do MVP grande** (multiempresa, publicação/replicação, métricas, conciliação MP, base contábil) | Prazo | Desenvolvimento modular com protótipo validado por módulo; ordem guiada pelo que desliga o Olist primeiro. |
| 5 | **Workers só em serverless** (Vercel/Edge): limite de duração, cold start, webhook de 500 ms | Perda de eventos | Receptor mínimo, fila em Postgres, reconciliação diária; monitorar e, se preciso, mover os workers para um container. |
| 6 | **Refresh token de uso único** + 4 contas | Desconexão de conta | Token broker com trava e alerta de reconexão. |
| 7 | **Sem sandbox** no ML | Bug em produção | Usuários de teste + respostas gravadas + modo "somente leitura" nas primeiras semanas. |
| 8 | **Conciliação Mercado Pago** pode exigir credencial MP própria (não confirmado) | Atraso no financeiro | Validar logo no início com a conta real. |
| 9 | **Métricas de concorrência**: o ML proíbe scraping; a API só dá dados próprios e catálogo | Expectativa | Deixar claro o que o ERP mede (visitas, conversão, Ads, `price_to_win`). |
| 10 | **Emissor próprio** (Fase 2): SEFAZ, contingência, reforma tributária | Alto esforço | Fase isolada, com testes em homologação SEFAZ por empresa. |
| 11 | **Migração**: dados do Olist com erros (opções de variação duplicadas, mapeamentos com erro) | Retrabalho | Migração com etapa de limpeza e conferência. |
| 12 | **Dado sensível**: custo e margem para o perfil logística; certificados A1 (Fase 2) | Vazamento | RLS + views por perfil; segredos só no servidor. |
| 13 | **API do ML muda com frequência** (vários endpoints descontinuados em 2025–2026) | Quebras | Camada ML isolada, monitor de integração, revisão periódica do changelog. |
| 14 | **TSC TE200** pode não emular ZPL | Etiqueta não imprime | Testar já no protótipo: ZPL nativo ou PDF via driver. |

## 8. Perguntas em aberto

Respondidas em 27/10/2026: todas as empresas ficam no endereço do galpão · a MEI não usa Full/Faturador por enquanto · as contas não são vinculadas no ML (nunca houve aviso) · migrar **todo o histórico** do Olist · o tributário confere **todas as NF de todas as plataformas** · as políticas da loja ficam no ERP (já mapeadas nos documentos do agente) · usuários e contas **ilimitados**, criados pelo admin na interface · a TSC TE200 **imprime ZPL**.

Ainda pendentes:
1. **Banco**: 1 por empresa ou 1 único compartilhado (pergunta ao contador).
2. **Plano de contas**: qual usar (pergunta ao contador).
3. **Flex**: em aberto.
4. Endereço do galpão (informar depois).
5. Tags das contas ML (`user_product_seller`, `warehouse_management`): consultar na API ao conectar.

## 9. Processo daqui para frente (definido pelo cliente)

1. ~~Validar este contexto~~ ✅ validado em 27/10/2026.
2. **Protótipo HTML modular** ⬅️ *estamos aqui*: primeiro o **layout base + login + sidebar**; depois um módulo por vez (~10 iterações em HTML atômico/isolado por módulo). **A aprovação é feita vendo o protótipo**, não por perguntas.
3. Roadmap e tarefas depois do protótipo aprovado.
4. Estrutura de dados (schema Supabase).
5. Código modular (front + back juntos), módulo a módulo, até o MVP. Depois a Fase 2 (emissor e relatórios contábeis) e o agente de IA.

Sugestão de ordem dos módulos no protótipo (para validar): **Layout/Login → Empresas e Contas (conexão ML) → Produtos + Base de conhecimento → Anúncios (vincular, publicar, replicar) → Estoque (galpão × empresas, Full) → Pedidos → Separação/Etiquetas/Expedição → Devoluções → Atendimento (inbox) → Compras e reposição → Financeiro (repasses, DRE) → Dashboards/margem → Fiscal (importação e contábil) → Configurações/Usuários.**
