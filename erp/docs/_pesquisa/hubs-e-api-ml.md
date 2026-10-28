# Hubs de marketplace e API do Mercado Livre — mapa de funcionalidades para o ERP Pinese

> **Fase:** pesquisa (nenhum código). **Data da coleta:** 16/10/2026.
> **Escopo:** levantar o que hubs de mercado (Ideris, Anymarket e, como apoio, Plugg.to, Magis5 e o próprio Olist ERP) entregam para quem vende no Mercado Livre, e mapear os recursos da API do Mercado Livre (ML) que um ERP precisa usar sendo **o dono da integração** (OAuth, tokens, webhooks, fila de eventos), com um agente de IA de atendimento consumindo essa camada.
> **Operação de referência:** 2 contas de vendedor ML (catálogo parcialmente igual), cerca de 40 anúncios ativos e mais 40 a publicar, logística Full + Mercado Envios (coleta/agência), nicho casa/jardim/decoração/ferramentas, equipe de 2 pessoas.

**Legenda usada no documento**

- **[F]** = fato, lido na documentação oficial ou na página do fornecedor (a fonte está na seção "Fontes lidas").
- **[I]** = inferência ou recomendação minha, sem confirmação documental.
- **Complexidade** é sempre uma estimativa (inferência), pensando numa equipe pequena e no padrão Next.js + Supabase.
- **Dependência de marketplace:** *Nenhuma* = feature 100% interna do ERP; *Parcial* = regra interna que lê ou grava dados no ML; *Total* = só existe por causa do ML e depende da API dele.

---

## 0. Sumário executivo — o que muda a arquitetura

1. **[F] Token:** o access token dura 6 h. O refresh token é **de uso único**, só o último gerado vale, e ele expira em 6 meses. Só o **administrador** da conta pode autorizar: se um operador ou colaborador tentar, a API devolve `invalid_operator_user_id`. São **2 grants independentes** (um por conta) → **[I]** é preciso um "token broker" central com lock por conta, porque dois workers renovando ao mesmo tempo derrubam a conexão.
2. **[F] Webhook:** o ERP precisa responder **HTTP 200 em até 500 ms**, ou os tópicos são **desativados**, e as notificações desse período não ficam no `missed_feeds`. O ML tenta de novo por 1 hora (8 tentativas); `missed_feeds` só guarda **2 dias**. → **[I]** o desenho obrigatório é: receptor burro → fila persistente → worker que faz GET no recurso, mais reconciliação periódica por polling.
3. **[F] O modelo de anúncio mudou para User Products (UP):** preço por variação, `family_name`, sem o array `variations` para quem já tem a tag `user_product_seller`, estoque por UP/local. → **[I]** o cadastro do ERP tem de nascer modelado como *Produto (SKU) → User Product → N anúncios (condições de venda)*, e não "1 anúncio = 1 produto com variações".
4. **[F] Estoque Full não é editável via API.** Só entra mandando mercadoria pelo painel do ML; a API apenas **consulta** estoque e operações. Criar o envio de reposição (inbound) pela API **não está disponível** para o Brasil: o fluxo público de Inbound FBM existe só para Fulfillment China. → O ERP pode **planejar e sugerir** a reposição e controlar a NF de remessa, mas o envio em si é criado no Seller Center.
5. **[F] NF-e no Full é emitida pelo Faturador do ML (obrigatório).** Nas demais logísticas (coleta/agência), quem emite NF-e fora do ML precisa **enviar o XML ao ML com o envio em `ready_to_ship / invoice_pending`**, antes da etiqueta. → **[I]** o ERP tem dois fluxos fiscais: (a) alimentar o Faturador (dados fiscais + regras tributárias) e importar as notas emitidas pelo ML; (b) emitir a própria NF-e e subir o XML.
6. **[F] Mensageria pós-venda MLB passou a ser intermediada por "Agentes de Mensageria [IA]" do ML desde 02/02/2026**, começando pelo Full. O `to.user_id` passa a ser o ID do agente (MLB = 3037675074), 1 mensagem por vez, 48 h úteis para resolver antes do bloqueio, limite de 350 caracteres, e mensagens automáticas/repetitivas são proibidas. → impacto direto no agente de IA do ERP.
7. **[F] Pedidos ficam guardados só 12 meses na API**, e as operações do Full também só 12 meses. → **[I]** o ERP é a fonte histórica (BI, fiscal, DRE).
8. **[F] Política de duplicados:** o ML não permite o mesmo produto com as mesmas condições de venda em mais de um anúncio. Os hubs indicam variar conta, título, tipo de anúncio, forma de envio ou frete grátis. Existir uma 2ª conta com o mesmo catálogo é um **risco de política** a validar com o cliente. **[I]**: a replicação entre contas precisa de regras de diferenciação.

---

## Parte 1 — Hubs de marketplace (visão geral)

| Hub | Posicionamento (declarado pelo fornecedor) | Destaques relevantes para a operação Pinese | Fonte |
|---|---|---|---|
| **Ideris** | Hub "para grandes operações", Parceiro Platinum ML, "bicampeã MeLi XP". | Replicação de anúncios entre contas ML (só anúncios **ativos** e **vinculados a produto**; os com tag "Sem controle de estoque" não replicam); criação/edição em massa; kit com variação; calculadora de preço por marketplace (4 tipos de regra, método markup ou margem); Central de promoções; respostas pré-programadas para perguntas/mensagens/reclamações; resposta por áudio com IA; app mobile; monitor de impressão (etiqueta + DANFE em sequência); ME1/ME2/Coleta/Places/Full/Flex; Faturador ML; "copiar anúncio de concorrente"; integração com conciliadores (plano Pro+); planejamento de envios ao Full com "sugestão de estoque", data limite e histórico semanal. Planos: Lite 500 pedidos/mês e 3 canais, Pro 1000 e 6 canais, Plus 3000 e canais ilimitados. | [F] ideris.com.br, /planos, /mercado-livre, blog, Central de Parceiros ML |
| **Anymarket** | "Mais de 200 funcionalidades", API-first, multicanal. | Regras de estoque (Mínimo = trava de segurança, Máximo por canal, Compartilhado em % por canal); Smart Product Pricing (markup por produto/categoria × custo de venda de cada canal); Smart Shipping Pricing (embute o frete no preço); Safe Sell / Price Guard (limite de desconto, bloqueio na importação do pedido); kit com estoque pela menor parte e suspensão automática; FMS (painel de fulfillment multicanal); Smart Product Match (vincular catálogo a anúncios existentes em lote); Product Categories Predictor; Ad Writer A.I.; Massive Operation; etiquetas em lote; Picking & Packing com conferência (até 200 pedidos por lote); Multi Accounts Central; Smart Warehouse Selection; OCRS (score de risco de cancelamento); Return Monitoring; Order Timeline; Monitoring/Integration Health; curva ABC; mapa de calor de conversão por estado. Exige **XML** (não JSON) para faturar pedido ME2, aceito só em `ready_to_ship/invoice_pending`. | [F] anymarket.com.br/funcionalidades, blog marketplace.anymarket.com.br, suporte Anymarket |
| **Plugg.to** | Hub multicontas. | Várias contas do mesmo marketplace com CNPJs diferentes: catálogo e estoque centralizados, mas **tabela de preço, promoção e regra de envio por conta**; regras de preço por tipo de anúncio/frete; simulador de preços e custos (taxas, comissões, logística); kits virtuais ML; IA para imagens (remover fundo, redimensionar); tela de expedição por bip; Fulfillment Manager (notas fiscais de Full). | [F] plugg.to/funcionalidades |
| **Magis5** | Hub com forte foco em ML. | Precificação automática (custo do ERP + custos operacionais + margem + comissões padrão); Estoque Mínimo (margem de segurança); Preço por Variação e Multi Estoque ML; Kit Virtual ML; horários do Flex; "Horário de Despacho"; Central de Promoções; infrações de anúncios; elegibilidade de catálogo; qualidade/experiência de compra; "Copiar anúncios"; relatórios "Produtos sem anúncio", "Anúncios Fulfillment pausados", "Produtos com estoque infinito"; vários anúncios do mesmo produto no mesmo marketplace; denúncias BPP; SAC de perguntas com respostas rápidas; Multi Origem ML. | [F] ajuda.magis5.com.br/magis5-hub |
| **Olist ERP (sistema atual)** | ERP + hub. | Integração "Mercado Livre Fulfillment" separada: **depósito exclusivo** do canal Full (o saldo é controlado pelo ML e "desconsiderado" para outras vendas); transferência automática de estoque ao emitir NF para o CNPJ do Full; envio de dados fiscais e natureza de operação (Simples × Regime Normal) ao Faturador; importação de anúncios (SKU, estoque, imagens etc.); multi-CNPJ/multi-conta. | [F] ajuda.olist.com (Fulfillment) |

---

## Parte 1 + 2 — Mapa de funcionalidades por módulo

> Formato pedido: **Feature | O que faz | Sistema de origem | Complexidade estimada | Dependência de marketplace**. Quando a feature usa a API do ML, o endpoint/tópico principal aparece em `código` na coluna de origem.

### 1. Cadastros

| Feature | O que faz | Sistema de origem | Complexidade estimada | Dependência de marketplace |
|---|---|---|---|---|
| Produto mestre (SKU) | Cadastro único: SKU, EAN/GTIN, marca, dimensões/peso, custo, NCM/CEST/origem, fotos-mestre. É a âncora de estoque, preço e fiscal. | Ideris, Anymarket, Plugg.to, Olist (catálogo centralizado); API ML aceita SKU **só no atributo `SELLER_SKU`** (não em `seller_custom_field`) [F] | **Média**: muitos campos, validação de GTIN/NCM e dados fiscais compartilhados com o Faturador | Nenhuma |
| Modelo Produto → User Product → Anúncios | Representa a família (`family_name`), o UP (variação física, `user_product_id`) e N itens (condições de venda: preço, tipo, logística). Até 30 condições de venda por UP [F]. | API ML `/user-products/{id}`, `/sites/MLB/user-products-families/{family_id}`, `/users/{id}/items/search?user_product_id=` [F] | **Alta**: modelo legado (`variations`) e UP coexistem; seller com tag `user_product_seller` não pode publicar no modelo antigo (400) [F] | Total |
| Variações / grade | Variação por cor, tamanho, voltagem etc. No legado: até 100 variações por categoria (250 em moda/autopeças) [F]. No UP: cada variação é um item, agrupado por família via atributos `PARENT_PK`/`CHILD_PK` [F]. | Ideris ("anúncios com variação", "kit com variação"), Magis5, API ML `/items`, `/categories/{id}/attributes` (`allow_variations`) | **Alta**: lógica de família, título gerado pelo ML no modelo UP (o integrador não envia `title`) [F] | Total |
| Kit físico / composição (no ERP) | Kit composto de SKUs. O estoque do kit = mínimo(estoque da parte ÷ quantidade da parte); o anúncio é suspenso quando falta um componente. | Anymarket (Product Kit), Ideris, Magis5 | **Média**: cálculo derivado e baixa dos componentes na venda | Nenhuma (regra) / Parcial (publicação) |
| Kit virtual nativo ML | Kit publicado no ML a partir de UPs existentes: 2 a 6 produtos, até 10 unidades de cada; imutável na composição; só `condition=new`; kit de Full **não tem `inventory_id`**; preço via `/sale_price` [F]. | API ML `/users/{seller}/kits/components/search` + `POST /items` [F]; Plugg.to, Magis5 | **Alta**: regras próprias e estoque calculado pelo ML | Total |
| Categoria e atributos ML | Escolher a categoria/domínio e preencher os atributos obrigatórios, inclusive os atributos de envio exigidos pelo domínio para ME2. | API ML `/categories/{id}/attributes`, `/catalog_domains/{domain}/shipping_attributes` [F]; Anymarket (Product Categories Predictor) | **Média**: atributos variam por categoria; vale cache | Total |
| Fotos e descrição | Upload e ordenação de imagens; descrição enviada à parte (`POST /items/{id}/description`, já que o campo no `/items` foi desativado em 2021) [F]; edição em lote de imagens de todos os anúncios do produto. | API ML; Magis5; Plugg.to (IA remove fundo/redimensiona) | **Média**: storage MinIO + URLs públicas estáveis para o ML baixar | Parcial |
| Dados fiscais do produto para o ML | Enviar NCM, origem, CEST etc. por SKU para o Faturador; consultar se o item pode ser faturado. | API ML `POST /items/fiscal_information`, `GET /items/{id}/fiscal_information/detail`, `GET /can_invoice/items/{id}` [F]; Olist | **Média** | Total |
| Custos e fornecedores | Custo de aquisição/reposição por SKU (alimenta margem e precificação). | Ideris/Anymarket/Magis5 (custo como base da precificação) | **Baixa** | Nenhuma |
| Clientes (compradores) | Cadastro a partir do pedido. Dados fiscais vêm de `billing-info` (endereço fiscal ≠ endereço de entrega) [F]. | API ML `/orders/billing-info/MLB/{billing_info_id}` [F] | **Média**: LGPD, endereço ofuscado até o pagamento [F] | Parcial |
| Contas de marketplace (multi-conta / multi-CNPJ) | Cadastro das 2 contas ML, cada uma com grant, preferências de envio e regras próprias. | Plugg.to, Anymarket (Multi Accounts Central), Olist, Ideris | **Média** | Total |

### 2. Estoque

| Feature | O que faz | Sistema de origem | Complexidade estimada | Dependência de marketplace |
|---|---|---|---|---|
| Estoque multi-depósito | Saldos por depósito: próprio (galpão), Full da conta A, Full da conta B. O Full fica "desconsiderado" para outras vendas. | Olist (depósito exclusivo Full) [F]; Anymarket (estoque virtual por CD) [F] | **Média** | Nenhuma / Parcial |
| Sync de estoque multi-conta | O mesmo estoque físico alimenta os anúncios das 2 contas. Ao vender, o ERP recalcula e envia o disponível para todos os anúncios vinculados. | Todos os hubs; API ML `PUT /items/{id}` (`available_quantity`) **ou** `PUT /user-products/{id}/stock/type/seller_warehouse` (multiorigem, header `x-version`) [F] | **Alta**: concorrência/latência (overselling), escolha do endpoint por tipo de conta, 409 de versão, assíncrono [F]. **[I]** O ML só sincroniza sozinho os itens do **mesmo** UP da mesma conta; entre contas, o ERP faz a sincronização. | Total |
| Estoque de segurança / mínimo / máximo / % por canal | Publica menos do que o físico (trava), limita o máximo por canal ou reparte em %. | Anymarket (Regras de Estoque) [F]; Magis5 (Estoque Mínimo) [F] | **Média** | Parcial |
| Reserva por pedido | Reserva ao confirmar o pedido, baixa na expedição/NF, libera no cancelamento. | Inferência a partir de orders_v2 e status de pack/shipment [I] | **Média**: eventos fora de ordem, carrinho (pack) | Parcial |
| Espelho do estoque Full | Saldo Full por `inventory_id`: disponível e não disponível (damaged, lost, withdrawal, internal_process, transfer, noFiscalCoverage, not_supported) com condições detalhadas. | API ML `GET /inventories/{inventory_id}/stock/fulfillment` (+`?include_attributes=conditions`) [F] | **Média**: `inventory_id` por variação [F]; só leitura | Total |
| Movimentações Full (kardex Full) | Histórico de operações: inbound_reception, sale_confirmation, sale_cancelation, sale_return, withdrawal_*, transfer_*, fiscal_coverage_ajustment. | API ML `GET /stock/fulfillment/operations/search` (janela ≤ 60 dias, dados de 12 meses, paginação por scroll de 5 min) + tópico `fbm_stock_operations` [F] | **Média** | Total |
| Planejamento de reposição Full | Sugere quanto enviar por SKU/conta (cobertura em dias, giro semanal, data limite para não perder o selo de envio rápido); gera a lista e a NF de remessa. | Ideris ("Planejamento de envios", "sugestão de estoque", data limite, histórico semanal) [F]; Anymarket FMS | **Alta**: previsão de demanda. O envio é criado no Seller Center, porque a API não cria inbound no Brasil [F] | Parcial |
| Estoque distribuído / multiorigem | Contas com `warehouse_management`: estoque por `seller_warehouse` (store_id/network_node_id); Full (`meli_facility`) é independente; `selling_address` (Flex separado do Full) só existe em MLA/MLC, **não no MLB** [F]. | API ML `/user-products/{id}/stock`, `/stock/type/seller_warehouse` (100 rpm) [F]; Magis5 (Multi Origem) | **Alta**: modos diferentes por conta; um PUT em `/items` pode responder 200 e não alterar nada [F] | Total |
| Inventário, ajustes e kardex | Contagem, ajuste, transferência entre depósitos, histórico de movimentos. | Olist/ERPs [I] | **Média** | Nenhuma |
| Retorno de devoluções ao estoque | Acompanha a devolução até voltar ao estoque vendável (ou avaria). | Anymarket (Return Monitoring) [F]; API ML returns | **Média** | Parcial |
| Pausa/reativação automática | Estoque 0 → item `paused/out_of_stock`; >0 → reativa (exceto `paused_by_seller`) [F]. | API ML (comportamento de `available_quantity`) [F]; Anymarket, Magis5 | **Baixa** | Total |

### 3. Vendas e pedidos

| Feature | O que faz | Sistema de origem | Complexidade estimada | Dependência de marketplace |
|---|---|---|---|---|
| Importação de pedidos em tempo real | Recebe a notificação, busca o pack e as orders, cria o pedido no ERP de forma idempotente. | API ML tópico `orders_v2` (recomendado) → `GET /packs/{pack_id}` → `GET /orders/{id}` [F] | **Alta**: idempotência, reprocessamento, eventos repetidos/fora de ordem, 2 contas | Total |
| Carrinho (pack) | 1 pack : N orders (1 order = 1 item/variação com N unidades); pack : shipment 0..1; order : payments 1..N [F]. Todos os pedidos têm `pack_id` (migração de 2024) [F]. | API ML `/packs/{id}` [F] | **Média** | Total |
| Busca/conciliação de pedidos | Polling de segurança e backfill (por status, datas de criação/atualização). | API ML `GET /orders/search?seller=&order.date_last_updated.from=` [F] | **Baixa** | Total |
| Status e timeline do pedido | Linha do tempo unificada: pago → NF → etiqueta → despachado → entregue/devolvido. | Anymarket (Order Timeline) [F]; API ML orders + `/shipments/{id}/history` [F] | **Média** | Parcial |
| Cancelamentos e mediações | Trata pedido cancelado (libera reserva, cancela NF) e pedido com mediação. | API ML `orders_v2`, `mediations` em order, `post_purchase` [F] | **Média** | Total |
| Descontos aplicados | Partes do desconto (vendedor × ML), cupons, cashback por pedido. | API ML `GET /orders/{id}/discounts` [F] | **Baixa** | Total |
| Pedidos Full | Importa vendas Full (sem baixar o estoque próprio; baixa o depósito Full) e as NF emitidas pelo ML. | Olist (integração Fulfillment) [F]; API ML | **Média** | Total |
| Risco de cancelamento | Score para agir antes do envio. | Anymarket (OCRS) [F] | **Alta**: opcional/futuro | Parcial |
| Feedback de venda | Lê e responde avaliações. | API ML tópico `orders feedback`; `feedback-de-uma-venda` [F] | **Baixa** | Total |

### 4. Fiscal

| Feature | O que faz | Sistema de origem | Complexidade estimada | Dependência de marketplace |
|---|---|---|---|---|
| Emissão de NF-e própria (coleta/agência/Flex) | O ERP emite NF-e de venda na SEFAZ (certificado A1, regras de ICMS/DIFAL). | Olist, Bling, Anymarket (via ERP) | **Alta**: SEFAZ, contingência, regras tributárias, cancelamento/CC-e | Nenhuma (SEFAZ) |
| Envio do XML da NF-e ao ML | Sobe o XML autorizado para o envio; obrigatório no MLB para cross docking/coleta [F]. Só é aceito com shipment `ready_to_ship` + `invoice_pending`; depois da etiqueta impressa não dá mais para alterar [F]. | Anymarket, Bling, Tecnospeed [F]; endpoint oficial **não localizado** na doc lida (ver "Não confirmado") | **Média**: máquina de estados + retentativa ao mudar o substatus | Total |
| Faturador ML (Full obrigatório) | O ML emite as NF-e: o ERP cadastra as regras tributárias (Regime Normal: SALE e SALE_RETURN) e os dados fiscais dos itens; emissão via API também para outras logísticas. | API ML `/users/{id}/invoices/tax_rules`, `POST /items/fiscal_information`, `POST /users/{id}/invoices/orders`, `/users/invoices/errors/MLB/{code}` [F]; Ideris, Olist | **Média/Alta**: setup fiscal correto, conferido com a contabilidade | Total |
| Importação das NF-e emitidas pelo ML | Baixa XML/DANFE (SALE, SALE_RETURN, DEVOLUTION, INBOUND, SYMBOLIC_INBOUND, REMOVAL, INPUT/OUTPUT, CTE…) por nota, pedido, envio ou em lote por período. | API ML tópico `invoices`; `/users/{id}/invoices/{id}`, `/invoices/orders/{order}`, `/invoices/shipments/{id}`, `/invoices/sites/MLB/batch_request/period/...` [F] | **Média** | Total |
| NF de remessa ao Full e retornos | Controla remessa para depósito temporário, retorno simbólico e retirada (casando com o kardex Full). | API ML (tipos INBOUND/SYMBOLIC_INBOUND etc.) [F]; Olist (transferência automática por CNPJ do Full) [F] | **Alta**: regra fiscal específica do Full por regime | Parcial |
| NF de devolução (entrada) | Entrada da mercadoria devolvida, referenciando a NF de saída. | ERPs; ML emite DEVOLUTION no Full [F] | **Média** | Parcial |
| Dados fiscais do comprador | Identifica CPF/CNPJ, IE e endereço fiscal; o tipo de nota fica a cargo do integrador (`invoice_type` removido) [F]. | API ML `/orders/billing-info/MLB/{id}` (o legado `/orders/{id}/billing_info` foi depreciado) [F] | **Baixa** | Total |
| Monitor de impressão | Fila sequencial de DANFE + etiqueta. | Ideris [F]; Anymarket (lote) [F] | **Média** | Parcial |

### 5. Financeiro

| Feature | O que faz | Sistema de origem | Complexidade estimada | Dependência de marketplace |
|---|---|---|---|---|
| Recebível por pedido | Para cada venda: bruto, tarifa de venda, custo fixo, frete pago pelo vendedor, descontos, impostos → líquido previsto e data de liberação. | API ML `/orders`, `/shipments/{id}/costs`, `/orders/{id}/discounts`; Mercado Pago `GET /v1/payments/{id}` (indicado na doc de packs) [F] | **Alta**: composição vem de várias fontes, sem campo único [F] (FAQ billing-info) | Parcial |
| Conciliação de repasses Mercado Pago | Bate o previsto com o liberado e com a conta (Liberações = após a liberação; Dinheiro em conta/Settlement = antes) [F]. | Mercado Pago `POST/GET /v1/account/release_report` (+ `/config`, `/list`) [F]; settlement report [I]; Ideris ("integração com conciliadores", plano Pro) | **Alta**: credenciais MP, relatórios assíncronos (202), casamento por payment_id | Total |
| Conciliação das faturas ML | Confere as cobranças do ML (comissões, envios, Full, publicidade) por período: periods → documents → summary → details; regra: soma de `detail_amount` por `detail_sub_type` = `amount` do summary [F]. | API ML `/billing/integration/monthly/periods`, `/periods/key/{key}/documents`, `/summary/details`, `/group/ML/details`, `/group/MP/details`, `/group/ML/payment/details`, `/payment/{id}/charges` [F] | **Alta**: volume; summary 1×/dia por usuário, não usar em batch [F]; `payment_id` é **string** [F] | Total |
| Simulador de custo por vender | Calcula comissão e custo fixo por categoria/preço/tipo de anúncio/logística antes de publicar. Desde 2026 o `fixed_fee` depende da logística: sem `logistic_type`/`shipping_mode` o valor não bate (MLB ativado em 02/03) [F]. | API ML `GET /sites/MLB/listing_prices?...` [F]; Plugg.to (simulador) | **Média** | Total |
| Custo de frete do vendedor | Custo de envio grátis por dimensões/preço/logística; custo real por envio. | API ML `/users/{id}/shipping_options/free?...`, `/shipments/{id}/costs` [F] | **Média** | Total |
| Custos do Full | Armazenagem, retirada, etc. (via detalhes de faturamento). | API ML billing details [I]; Ideris blog ("Tempo de estoque") | **Média** | Total |
| Contas a pagar/receber, fluxo de caixa, DRE gerencial | Visão financeira consolidada (marketplace + despesas). | ERPs [I] | **Média** | Nenhuma |

### 6. Marketplaces / hub (camada de integração + gestão de anúncios)

| Feature | O que faz | Sistema de origem | Complexidade estimada | Dependência de marketplace |
|---|---|---|---|---|
| Conexão OAuth multi-conta | Fluxo authorization code (+PKCE opcional), `state`, `redirect_uri` fixa; guarda os tokens por `user_id`; renova perto da expiração com lock; detecta a revogação (`invalid_grant`). | API ML `auth.mercadolivre.com.br/authorization`, `POST /oauth/token` [F] | **Alta**: refresh de uso único + concorrência + 2 contas | Total |
| Receptor de webhooks + fila | Endpoint público que só valida e enfileira (200 em < 500 ms); workers consomem, fazem GET no `resource` e roteiam por `topic`/`user_id`; dedupe pelo `_id`. | API ML Notificações [F]; Anymarket ("150 mil alertas retentados/dia") [F] | **Alta**: núcleo da arquitetura | Total |
| Recuperação de eventos perdidos | Job que lê `missed_feeds` (≤ 2 dias; `items` exige `site_id`) + polling de reconciliação. | API ML `GET /missed_feeds?app_id=&topic=&site_id=` [F] | **Média** | Total |
| Criação de anúncio | Publica item (UP ou legado), descrição, fotos, atributos, logística, tipo (clássico/premium). | API ML `POST /items`, `POST /items/{id}/description` [F]; todos os hubs | **Média/Alta** | Total |
| Criação/edição em massa | Planilha ou lote para publicar/alterar preço, estoque, campanha, imagens. | Ideris, Anymarket (Massive Operation), Magis5 [F] | **Média**: fila + rate limit | Total |
| Replicação de anúncio entre contas | Copia um anúncio da conta A para a B (só ativos e vinculados a SKU, na Ideris). | Ideris [F]; Magis5 ("Copiar anúncios") [F] | **Média**. **[I]** risco de política de duplicados entre contas ligadas | Total |
| Múltiplos anúncios do mesmo produto | Clássico + premium, frete grátis ou não, outra logística. | Anymarket (suporte), Magis5 [F] | **Baixa** | Total |
| Catálogo ML | Elegibilidade, optin de item tradicional (`POST /items/catalog_listings`), monitorar a competição (`/items/{id}/price_to_win`, tópico `catalog_item_competition_status`); o auto-optin do ML gera a tag `catalog_boost`; as condições de venda entre o item de catálogo e o tradicional sincronizam sozinhas (sem opt-out) [F]. | API ML [F]; Ideris, Magis5 [F] | **Alta** | Total |
| Vincular anúncios existentes a SKUs | Importa os anúncios já publicados (migração do Olist) e casa com o SKU por `SELLER_SKU`/GTIN. | Anymarket (Smart Product Match) [F]; Olist (importar anúncios) [F]; API ML `/users/{id}/items/search?search_type=scan` (>1000), `/items/bulk?ids=` [F] | **Média**: crítico na migração | Total |
| Regras de preço por canal/conta | Preço por conta/tipo de anúncio: markup ou margem, acréscimo fixo/%, limites. | Ideris (4 regras; markup: `(custo+fixo)/(1-%total)`) [F]; Anymarket (Smart Pricing) [F]; Plugg.to (tabela por CNPJ) [F]; Magis5 [F] | **Média** | Parcial |
| Atualização de preço | Edita o preço standard. Itens com **automatização de preço** ativa: o PUT só de preço é rejeitado (400) desde 18/03/2026 [F]. | API ML `GET /items/{id}/prices`, `/items/{id}/sale_price?context=`, `POST /items/{id}/prices/standard` (anunciado como "ainda não disponível") [F]; PUT `/items` | **Média** | Total |
| Proteção de margem | Bloqueia preço ou desconto abaixo do limite e alerta. | Anymarket (Safe Sell/Price Guard) [F] | **Baixa/Média** | Parcial |
| Central de promoções | Lista e aceita campanhas: DEAL, MARKETPLACE_CAMPAIGN, PRICE_DISCOUNT, LIGHTNING, DOD, VOLUME, PRE_NEGOTIATED, campanhas e cupons do vendedor. | API ML `/seller-promotions/...`; tópicos `public_offers`, `public_candidates` [F]; Ideris, Magis5 [F] | **Alta**: muitos tipos e regras | Total |
| Sugestão de preço ML | Recebe e exibe o preço sugerido pelo ML. | API ML tópico `price_suggestion`, `/suggestions/items/{id}/details` [F] | **Baixa** | Total |
| Saúde dos anúncios | Infrações/moderações, `under_review`/`warning`/`inactive`, qualidade e experiência de compra. | API ML `/moderations/infractions`, status/substatus do item [F]; Magis5 [F] | **Média** | Total |
| Monitor de integração | Painel de erros por anúncio/pedido, retentativas, alertas. | Anymarket (Monitoring, Integration/Seller Health) [F] | **Média** | Parcial |
| Product Ads (leitura) | Métricas de anúncios patrocinados. Os endpoints legados foram desligados em 27/05/2026 [F]. | API ML `product-ads-leitura` [F] | **Média**: opcional | Total |

### 7. Logística

| Feature | O que faz | Sistema de origem | Complexidade estimada | Dependência de marketplace |
|---|---|---|---|---|
| Fila de expedição (picking/packing) | Lista o que separar por pedido/produto, confere por bip e libera em lote. | Anymarket (até 200 pedidos/lote), Plugg.to (bip), Ideris [F] | **Média** | Parcial |
| Etiquetas PDF/ZPL em lote | Gera etiqueta com envio `ready_to_ship` + `ready_to_print`/`printed` (permite reimpressão); máx. **50** `shipment_ids` por chamada; template do ML **não pode** ser alterado [F]. | API ML `GET /shipment_labels?shipment_ids=&response_type=pdf|zpl2` [F] | **Baixa/Média** | Total |
| Prazo de despacho (SLA) | Data/hora máxima de despacho; atrasos; `estimated_handling_limit` depreciado (13/05/2025) → usar `/sla` [F]. | API ML `/shipments/{id}/sla`, `/delays`, `/lead_time` [F]; Magis5 ("Horário de Despacho") | **Baixa** | Total |
| Coleta e agência | Logísticas `cross_docking` (coleta), `xd_drop_off` (Coletas e Places), `drop_off` (agência) [F]. | API ML `/shipments/{id}` (header `x-format-new: true` obrigatório desde 12/10/2025) [F] | **Média** | Total |
| Flex | Zonas de cobertura, horário de corte por zona, feriados, assinatura; tópico `flex-handshakes`. O app de entregas do ML **não** é integrável; limite de 1000 rpm; preço por peso/zona no MLB a partir de 24/08/2026 [F]. | API ML `/flex/sites/MLB/users/{id}/...` [F]; Magis5, Ideris | **Média** | Total |
| Full (operação) | Painel de estoque/operações, anúncios Full pausados, itens não disponíveis (avariados, sem cobertura fiscal…). | API ML fulfillment [F]; Anymarket FMS, Magis5, Ideris | **Média** | Total |
| Split de pacote | Divide um envio em mais pacotes (produtos que não cabem juntos). | API ML `POST /shipments/{id}/split` [F] | **Baixa** | Total |
| Rastreio | Transportadora/URL e histórico de status. | API ML `/shipments/{id}/carrier`, `/history`, `/shipment_statuses` [F] | **Baixa** | Total |
| Devoluções / reversa | Consulta a devolução, custo, revisão do produto devolvido (OK/falha unificados em `return-review`) [F]. | API ML `/post-purchase/v2/claims/{id}/returns`, `/post-purchase/v1/returns/{id}/return-review`, `/claims/{id}/charges/return-cost` [F] | **Média** | Total |

### 8. Relatórios / BI

| Feature | O que faz | Sistema de origem | Complexidade estimada | Dependência de marketplace |
|---|---|---|---|---|
| Dashboard de vendas por conta/canal | Faturamento, pedidos, ticket, por conta A/B, por logística. | Anymarket (Dashboard), Magis5 (Dashboard Intelligence) [F] | **Média** | Parcial |
| Margem real por SKU/anúncio | Preço − tarifas − frete − impostos − custo = margem de contribuição. | Inferência dos hubs (calculadoras) + billing [I] | **Alta**: depende da conciliação | Parcial |
| Curva ABC / giro | Classifica os produtos por venda/margem. | Anymarket (ABC Curve) [F]; Ideris (blog) | **Baixa** | Nenhuma |
| Mapa de calor de conversão por estado | Visitas × vendas por UF. | Anymarket (Heat Map & Conversion) [F] | **Média**: depende de visitas/UF | Parcial |
| Relatórios de exceção | Produtos sem anúncio, anúncios Full pausados, estoque "infinito", anúncios com precificação automática. | Magis5 [F] | **Baixa** | Parcial |
| Reputação e métricas de atendimento | Termômetro (`level_id`), MercadoLíder, reclamações/cancelamentos/atrasos; período de 60 dias ou 365 dias conforme o volume [F]; tempo de resposta de perguntas. | API ML `/users/{id}` (seller_reputation), `/users/{id}/questions/response_time`, `/claims/{id}/affects-reputation` [F] | **Baixa** | Total |
| Tempo de estoque no Full | Idade do estoque Full (custo de armazenagem). | Painel ML "Tempo de estoque" (Ideris blog) [F]; via API: inferir pelas operações [I] | **Média** | Total |

### 9. Outros — Atendimento (SAC), automações e IA

| Feature | O que faz | Sistema de origem | Complexidade estimada | Dependência de marketplace |
|---|---|---|---|---|
| Perguntas pré-venda | Caixa unificada (2 contas); responder; status UNANSWERED/ANSWERED/BANNED/CLOSED_UNANSWERED; bloqueio de usuários. | API ML tópico `questions`; `/questions/search?seller_id=&api_version=4`, `POST /answers`, `/my/received_questions/search` [F]; Ideris, Magis5 | **Média** | Total |
| Mensagens pós-venda | Conversa por pack. Em MLB/MLC a conversa passa pelos **agentes de IA do ML** (desde 02/02/2026): `to.user_id` = agente MLB **3037675074**, 1 mensagem por vez, 48 h úteis antes do bloqueio; 350 caracteres; anexos JPG/PNG/PDF/TXT até 25 MB (associar em 48 h); `mark_as_read=false`; o vendedor não inicia conversa (exceto pelos "motivos"); motivos de bloqueio (`blocked_by_time`, `blocked_by_mediation`, `blocked_by_fulfillment`…) [F]. | API ML tópico `messages` (created/read); `/messages/packs/{pack}/sellers/{seller}?tag=post_sale`, `/messages/attachments` [F]; Ideris | **Alta**: regras de moderação e da nova intermediação | Total |
| Reclamações e mediações | Lista e trata claims (reasons, detail, status/actions-history, expected-resolutions: refund, partial-refund, allow-return; open-dispute; impacto na reputação). | API ML tópico `post_purchase` (claims, claims_actions; `resource` com prefixo `/post-purchase`) [F]; `/post-purchase/v1/claims/search?...` [F]; Ideris | **Alta** | Total |
| Respostas pré-programadas | Modelos de resposta com variáveis. | Ideris, Magis5 [F] | **Baixa**. **[F]** o ML proíbe mensagens automáticas/repetitivas (moderação `AUTOMATIC_MESSAGE`) | Parcial |
| Agente de IA de atendimento | Consome a camada de eventos (perguntas, mensagens, claims) e o contexto do ERP (pedido, estoque, rastreio) para sugerir ou enviar respostas. | Ideris (resposta por áudio com IA) [F]; Anymarket (Ad Writer A.I.); objetivo do projeto | **Alta**: guardrails de política ML, humano no loop, limites de taxa | Total |
| Denúncias de propriedade intelectual (BPP) | Responder notificações de marca. | Magis5 [F]; API ML Brand Protection | **Baixa** | Total |
| Automações | Pausar/reativar por estoque, alertar SLA, aplicar regra de preço, mover pedido de status, retentar integração. | Todos os hubs [F] | **Média** | Parcial |
| Usuários, permissões e auditoria | Perfis internos (2 pessoas hoje), log de quem alterou preço/estoque/anúncio. | Padrão ERP [I] | **Média** | Nenhuma |
| App mobile | Atender e acompanhar pelo celular. | Ideris (Android/iOS) [F] | **Alta**: fora do MVP; [I] PWA responsivo cobre | Nenhuma |

---

## Parte 2 — Mapa da API do Mercado Livre (recurso → endpoint/tópico)

| Domínio | Recurso | Endpoint / tópico principal | Notas [F]/[I] |
|---|---|---|---|
| Autenticação | Autorização (quem autoriza: **administrador** da conta) | `https://auth.mercadolivre.com.br/authorization?response_type=code&client_id=&redirect_uri=&state=&code_challenge=` | [F] `redirect_uri` idêntica à cadastrada, sem parte variável (usar `state`); operador/colaborador → `invalid_operator_user_id`. |
| Autenticação | Troca do code e refresh | `POST https://api.mercadolibre.com/oauth/token` (`grant_type=authorization_code` / `refresh_token`) | [F] `expires_in: 21600` (6 h); `scope: offline_access read write`; refresh de uso único, só o último vale, expira em 6 meses; token enviado só no header `Authorization: Bearer`. |
| Autenticação | Revogação/invalidação | — | [F] Invalida o token: troca de senha, troca do client secret, revogação pelo usuário, **4 meses sem chamadas à API**, fluxos internos (fraude, desvinculação de dispositivos). |
| Notificações | Configuração | DevCenter (callback URL + tópicos) | [F] POST JSON com `resource`, `user_id`, `topic`, `application_id`, `attempts`, `sent`, `received` (subtópicos via `actions`). Horários em UTC. |
| Notificações | Tópicos relevantes | `orders_v2`, `orders feedback`, `items`, `items prices`, `questions`, `messages` (created/read), `post_purchase` (claims/claims_actions), `shipments`, `fbm_stock_operations`, `stock-location`, `flex-handshakes`, `payments` (`/collections/{id}`), `invoices`, `price_suggestion`, `catalog_item_competition_status`, `catalog_suggestions`, `public_offers`, `public_candidates`, `user_products` (created/updated/purged), `user_products_families` | [F] Os tópicos estão migrando para o modelo com subtópicos. |
| Notificações | Perdidas | `GET /missed_feeds?app_id=&topic=&site_id=&limit=&offset=` | [F] Até 2 dias; `items` exige `site_id` (senão 400); padrão de 10 por página. |
| Usuários | Conta, tags, reputação | `GET /users/me`, `GET /users/{id}` (`seller_reputation`, tags `user_product_seller`, `warehouse_management`), `/users/bulk?ids=` | [F] `/users?ids=` em descontinuação (migrar até 25/10/2026). |
| Usuários | Usuários de teste | `POST /users/test_user` | [F] Sem sandbox; até 10 usuários de teste; expiram após 60 dias sem uso; só interagem com outros usuários de teste; credenciais não recuperáveis. |
| Itens | Busca de itens do vendedor | `GET /users/{id}/items/search` (`search_type=scan` para > 1000), `GET /items/{id}`, `GET /items/bulk?ids=` | [F] `/items?ids=` em descontinuação até 25/10/2026. |
| Itens | Criar/editar/pausar/encerrar | `POST /items`, `PUT /items/{id}`, `POST /items/{id}/description` | [F] Com vendas, alguns campos ficam bloqueados; `available_quantity=0` pausa (`out_of_stock`). |
| Itens | Categorias/atributos | `/categories/{id}`, `/categories/{id}/attributes`, `/catalog_domains/{domain}/shipping_attributes`, `/categories/{id}/shipping_preferences` | [F] |
| Itens | Variações (legado) | array `variations` em `/items` | [F] Máx. 100 (250 em moda/acessórios de celular/autopeças); proibido para sellers UP. |
| Itens | User Products / famílias | `GET /user-products/{id}`, `GET /sites/{site}/user-products-families/{family_id}`, `POST /user-products-families/{family_id}/tasks` (editor de família, assíncrono) | [F] Não existe endpoint para listar todas as famílias do vendedor. |
| Itens | Preços | `GET /items/{id}/prices`, `GET /items/{id}/sale_price?context=`, `POST /items/{id}/prices/standard` | [F] `price`/`base_price`/`original_price` de `/items` serão descontinuados; preço por quantidade absoluto (`/prices/standard/quantity`) sai em 27/10/2026. |
| Itens | Catálogo | `POST /items/catalog_listings` (optin), elegibilidade, `GET /items/{id}/price_to_win`, `/public/buybox/sync` | [F] |
| Itens | Kits virtuais | `/users/{seller}/kits/components/search` | [F] |
| Itens | Dados fiscais | `POST /items/fiscal_information`, `GET /items/{id}/fiscal_information/detail`, `GET /can_invoice/items/{id}` | [F] |
| Estoque | Estoque por UP / multiorigem | `GET /user-products/{id}/stock`, `PUT /user-products/{id}/stock/type/seller_warehouse` (header `x-version`; 409 em conflito) | [F] `selling_address` só em MLA/MLC; 100 rpm neste recurso. |
| Estoque | Full | `GET /inventories/{inventory_id}/stock/fulfillment`, `GET /stock/fulfillment/operations/search`, `GET /stock/fulfillment/operations/{id}` | [F] Só leitura; inbound pelo Seller Center. |
| Pedidos | Pack, orders, busca | `GET /packs/{id}`, `GET /orders/{id}`, `GET /orders/search?seller=...`, `GET /orders/{id}/discounts`, `GET /orders/{id}/shipments` (header `X-New-Domain: true`) | [F] Pedidos guardados 12 meses; a "vista atual" de `/orders/{id}/shipments` foi descontinuada no fim de set/2026. |
| Pedidos | Dados fiscais do comprador | `GET /orders/billing-info/MLB/{billing_info_id}` | [F] O legado `/orders/{id}/billing_info` foi depreciado. |
| Envios | Shipment | `GET /shipments/{id}` (+ `/items`, `/costs`, `/payments`, `/sla`, `/delays`, `/lead_time`, `/history`, `/carrier`), `/shipment_statuses` | [F] `x-format-new: true` obrigatório; `order_id`/`external_reference` removidos dos shipments (12/10/2025); endereço ofuscado até o pagamento. |
| Envios | Tipos (`logistic_type`) | `fulfillment`, `cross_docking`, `xd_drop_off`, `drop_off`, `self_service` (Flex); modos `me1`, `me2`, `custom`, `not_specified` | [F] Agora um mesmo UP pode ter várias logísticas ao mesmo tempo (`/customers/marketplace/sites/{site}/user-products/{up}/contracts/shippability/services`). |
| Envios | Etiquetas | `GET /shipment_labels?shipment_ids=…&response_type=pdf|zpl2` | [F] ≤ 50 ids por chamada. |
| Envios | Preferências/custos | `/users/{id}/shipping_preferences`, `/users/{id}/shipping_options/free`, `/items/{id}/shipping_options?zip_code=` | [F] |
| Envios | Flex | `/flex/sites/{site}/users/{id}/subscriptions/v1`, `.../configurations/coverage/zones/v1`, `.../holidays/v1`, `/flex/sites/{site}/shipments/{id}/assignment/v1` | [F] 1000 rpm. |
| Fiscal | Faturador ML | `/users/{id}/invoices/tax_rules`, `POST /users/{id}/invoices/orders`, `/users/invoices/errors/{site}/{code}` | [F] Obrigatório no Full; opt-in feito pelo seller (PJ, IE, certificado digital). |
| Fiscal | Consulta e download de NF | `/users/{id}/invoices/{invoice_id}`, `/invoices/orders/{order_id}`, `/invoices/shipments/{shipment_id}`, `/invoices/documents/xml/{id}/authorized`, `/invoices/sites/MLB/batch_request/period/...` | [F] Tópico `invoices`. |
| Fiscal | Envio do XML de NF própria (ME2 coleta/agência) | não localizado na doc oficial lida | [F, por terceiros] Exige shipment `ready_to_ship/invoice_pending`, XML autorizado, `Content-Type: application/xml`, assinatura válida e UF do emitente igual à UF de origem do envio. Veja "Não confirmado". |
| Fiscal | Upload de NF ao pack (`/packs/{id}/fiscal_documents`) | `POST /packs/{pack_id}/fiscal_documents` | [F] **Disponível em todos os sites exceto MLB.** Não serve para o Brasil. |
| Financeiro | Faturas ML/MP | `/billing/integration/monthly/periods`, `/billing/integration/periods/key/{key}/documents`, `/summary/details`, `/group/ML/details`, `/group/MP/details`, `/group/ML/payment/details`, `/billing/integration/payment/{id}/charges` | [F] Parâmetro `group` ML/MP; uso de pós-venda/conciliação, não operacional. |
| Financeiro | Custos por vender | `GET /sites/MLB/listing_prices?price=&category_id=&listing_type_id=&logistic_type=&shipping_mode=...` | [F] |
| Financeiro | Pagamentos e repasses | `GET https://api.mercadopago.com/v1/payments/{id}`; `/collections/{id}` (tópico `payments`); relatórios MP `/v1/account/release_report` | [F] endpoints; [I] credenciais exigidas pelos relatórios MP. |
| Pós-venda | Perguntas | `/questions/search`, `/questions/{id}`, `POST /answers`, `/users/{id}/questions/response_time` | [F] |
| Pós-venda | Mensagens | `/messages/packs/{pack}/sellers/{seller}?tag=post_sale`, `POST` de mensagem, `/messages/attachments` | [F] 500 rpm GET e 500 rpm POST/PUT (compartilhados). |
| Pós-venda | Reclamações | `/post-purchase/v1/claims/search`, `/claims/{id}`, `/detail`, `/status-history`, `/actions-history`, `/expected-resolutions(/refund|/partial-refund|/allow-return)`, `/actions/open-dispute`, `/affects-reputation`, `/claims/reasons/{id}` | [F] |
| Pós-venda | Devoluções e trocas | `/post-purchase/v2/claims/{id}/returns`, `/post-purchase/v1/returns/{id}/return-review`, `/returns/reasons`, `/claims/{id}/charges/return-cost`, `/claims/{id}/changes` | [F] |
| Promoções | Seller promotions | `/seller-promotions/...` (offers, candidates, campanhas, cupons) | [F] |
| Ads | Product Ads | endpoints novos de leitura | [F] Os legados foram desligados em 27/05/2026 (404). |

---

## Restrições técnicas da API ML relevantes para a arquitetura

| # | Restrição | Implicação para o ERP | Tipo | Fonte |
|---|---|---|---|---|
| R1 | Access token de 6 h; refresh token **uso único**, só o último vale, expira em 6 meses; `invalid_grant` se for reutilizado ou revogado. | Token broker central: uma linha por conta, renovação com lock (ex.: `SELECT … FOR UPDATE` / advisory lock) e escrita atômica do novo par; alertar a reconexão. Nunca renovar em paralelo. | F + I | autenticacao-e-autorizacao |
| R2 | Só o **administrador** da conta autoriza; colaborador gera `invalid_operator_user_id`. | O onboarding das 2 contas exige o login do titular de cada uma. | F | autenticacao-e-autorizacao |
| R3 | Token invalidado por troca de senha, troca do secret, revogação, **4 meses sem uso**, eventos internos de segurança. | Health-check do grant + fluxo de reconexão na UI; alerta proativo. | F + I | autenticacao-e-autorizacao |
| R4 | `redirect_uri` estática e idêntica; `state` recomendado (o ML não o valida); PKCE opcional mas obrigatório se for habilitado. | Uma rota de callback; o `state` carrega a conta-alvo e o nonce anti-CSRF. | F | autenticacao-e-autorizacao |
| R5 | Webhook: responder **200 em ≤ 500 ms**, senão os tópicos são desativados (perdendo o que ocorreu, sem registro em missed_feeds); retentativas por 1 h (8 tentativas); depois disso a notificação é apagada. | Receptor mínimo (validar, gravar na fila, 200) separado do processamento. A fila (ex.: tabela Postgres/pgmq) com dedupe garante entrega ao worker. | F + I | produto-receba-notificacoes |
| R6 | A notificação só traz o `resource`; é preciso fazer GET no recurso. Há eventos internos não visíveis que também disparam notificação, e mudanças podem vir do Seller Central ou de outros apps. | O worker sempre relê o estado, compara com o snapshot e aplica. Idempotência total. Esperar notificações "vazias". | F | produto-receba-notificacoes |
| R7 | `missed_feeds` guarda **2 dias**; o tópico `items` exige `site_id`. | Job diário de missed_feeds + reconciliação por polling (orders/search por `date_last_updated`, shipments, claims). | F | produto-receba-notificacoes |
| R8 | IPs de origem das notificações são publicados (lista longa, AWS/GCP). | Filtrar IP é opcional; preferir validar `application_id` + `user_id` conhecido. | F + I | produto-receba-notificacoes |
| R9 | Rate limits documentados por recurso: mensagens 500 rpm (GET) e 500 rpm (escrita); `user-products/{id}/stock` 100 rpm; Flex 1000 rpm; 429 `local_rate_limited`, com cotas que podem ser compartilhadas além do app. Limite global por app **não documentado** na doc lida. | Cliente HTTP com token bucket por conta e por recurso, backoff exponencial com jitter, fila de escrita com concorrência limitada. | F + I | mensagens-post-venda, convivencia-full-e-flex, envios-flex, autenticacao (429), FAQ rate limit Global Selling |
| R10 | **Sem sandbox**: só usuários de teste em produção (até 10, expiram com 60 dias sem uso, só negociam entre si, credenciais não recuperáveis). Testar UP/kits exige formulário de ambientação. | Cofre de usuários de teste; testes e2e contra a produção com contas de teste; mocks/gravação de respostas para testes automatizados. | F + I | start-testing, user-products, kits-virtuais |
| R11 | Modelo **User Products**: com a tag `user_product_seller` não se publica com `variations` (400); o título é gerado pelo ML; o PUT em atributos compartilhados replica de forma assíncrona para todos os itens do UP; máx. 30 condições de venda por UP. | Modelo de dados do ERP: Produto/UP/Item; tratar sincronização assíncrona e eventos `user_products`. | F | preco-variacao, user-products |
| R12 | Estoque: sem multiorigem → `PUT /items available_quantity` (o ML replica para todos os itens do UP); com multiorigem → só `seller_warehouse` (PUT em `/items` "funciona" e é ignorado); exige `x-version` (409 em conflito); `selling_address` não existe no MLB. | Roteador de escrita de estoque por tipo de conta/anúncio; GET da versão antes do PUT; retry em 409. | F | estoque-distribuido, gestao-de-estoque-multiorigem |
| R13 | Full: estoque só leitura; inbound só pelo Seller Center (a API Inbound FBM existe apenas para Fulfillment China); operações com janela de 60 dias e histórico de 12 meses; scroll de 5 min. | O ERP planeja a reposição e emite a NF de remessa, mas a criação do envio é manual no ML; guardar o kardex Full localmente. | F | en_us/fulfillment, envios-fulfillment, estoque-distribuido, Global Selling inbound |
| R14 | Pedidos guardados 12 meses na API; buscas como vendedor filtram cancelados. | O ERP persiste tudo (fonte histórica). | F | gerenciamento-de-vendas |
| R15 | Shipments exigem `x-format-new: true`; `order_id` saiu do shipment; o endereço do comprador fica ofuscado até o pagamento; `estimated_handling_limit` depreciado → `/sla`. | Relacionar sempre pack → order → shipment pelo lado de orders/packs; ouvir o tópico `shipments`. | F | gerenciamento-de-envios |
| R16 | Etiqueta só com `ready_to_ship` + `ready_to_print`/`printed`; ≤ 50 ids por chamada; template inalterável. | Lotes de ≤ 50; impressão ZPL direta (Zebra) ou PDF. | F | mercado-envios-2, boas-praticas |
| R17 | NF-e própria no MLB (coleta) é obrigatória e o XML só entra em `invoice_pending`, antes da etiqueta; depois de impressa, não se altera. | Ordem do fluxo: pago → emitir NF → subir XML → liberar etiqueta. Máquina de estados com retentativa por substatus. | F (terceiros + FAQ ML) | Bling, Anymarket, Tecnospeed, FAQ billing-info, shipment-handling |
| R18 | Full exige o Faturador ML (emissão automática); Regime Normal precisa de regras tributárias SALE + SALE_RETURN. | Módulo fiscal "duas vias": configurar o Faturador e importar as NF emitidas pelo ML; definir o regime com a contabilidade. | F | api-fiscal-faturamento-de-venda, envio-regras-tributarias |
| R19 | Mensageria: 350 caracteres, ISO-8859-1, o vendedor não inicia (exceto por motivos), mensagens automáticas proibidas, bloqueios por tempo/mediação/fulfillment; nova intermediação por agentes de IA do ML em MLB (ID 3037675074), 1 mensagem por vez, 48 h úteis. | O agente de IA do ERP precisa: limitar o texto, não fazer spam, detectar `conversation_status` bloqueado, usar o ID do agente no `to`, enfileirar 1 mensagem por vez por conversa, priorizar SLA de 48 h. | F | mensagens-post-venda, mensagens-bloqueadas, boas-praticas |
| R20 | Ações em massa podem gerar sanção nas contas; clonar publicações/imagens não é recomendado; itens elegíveis devem ir ao catálogo; não fazer web crawling. | Replicação entre contas com diferenciação e throttling; nada de scraping de concorrentes (o "copiar concorrente" da Ideris é arriscado). | F + I | boas-praticas-para-usar-a-plataforma |
| R21 | Preço: com automatização de preço ativa, PUT só de `price` → 400 (desde 18/03/2026); campos de preço de `/items` em descontinuação; usar a API de preços. | Detectar automatização ativa antes de aplicar a regra de preço do ERP; usar `/prices`/`/sale_price` para leitura. | F | api-de-precos, produto-sincronizacao-de-publicacoes |
| R22 | Billing: summary 1×/dia por usuário, não usar em batch; `payment_id` string; os relatórios servem à conciliação, não à operação. | Job diário (aberto: 2×/dia; fechado: 1×) e tipos corretos no schema. | F | boas-praticas-…-faturamento, pagamentos |
| R23 | Pedidos e mensagens são vistos só para vendedores autorizados no app; em packs com vários vendedores, só se veem as orders do próprio. | Um app ML (client_id) para as 2 contas; rotear tudo por `user_id`. | F + I | gestao-packs |
| R24 | IP allowlist para o app só para integradores na whitelist. | Não contar com isso; proteger o secret no servidor. | F | gerenciar-ips-de-um-aplicativo |

---

## Mudanças recentes na API (2025–2026) que afetam um ERP

| Data | Mudança | Impacto | Fonte |
|---|---|---|---|
| out/2024 → 2025 | Ativação progressiva do **User Products** (preço por variação) para 100% dos sellers. | Publicação sem `variations`, `family_name` obrigatório, UPtin dos legados. | user-products, preco-variacao [F] |
| 2024 | Todos os pedidos passam a ter `pack_id`. | Sempre ler o pack primeiro. | gestao-packs [F] |
| 13/05/2025 | `estimated_handling_limit` depreciado → `/shipments/{id}/sla`. | Ajustar o cálculo de prazo. | gerenciamento-de-envios [F] |
| 12/10/2025 | `order_id`/`external_reference` saem do shipment; `x-format-new: true` obrigatório. | Ajustar o cliente HTTP e o relacionamento. | gerenciamento-de-envios [F] |
| 29/12/2025 (última atualização) | PKCE suportado no OAuth (opcional; obrigatório se habilitado). | Recomendado habilitar. | autenticacao-e-autorizacao [F] |
| 02/02/2026 | **Nova arquitetura de mensageria** MLB/MLC com agentes de IA do ML (`to` = ID do agente; 1 mensagem por vez; 48 h úteis). | Desenho do agente de IA do ERP. | mensagens-post-venda [F] |
| 02/03 (2026, pela data da página) | `fixed_fee` do custo por vender passa a depender da logística (`logistic_type`, `shipping_mode`). | Calculadora de margem precisa da logística. | comissao-por-vender [F]; ano [I] |
| 18/03/2026 | Itens com automatização de preço ativa: PUT só de preço → 400; com outros campos, o preço é ignorado (warning). | Regra de preço do ERP deve checar a automatização. | api-de-precos [F] |
| 27/05/2026 | Endpoints legados de Product Ads desligados (404). | Usar os endpoints novos. | busca ML devsite [F] |
| 15/07/2026 | Restrições nas compatibilidades de autopeças gerenciadas pelo catálogo. | Baixo impacto (nicho casa/jardim). | busca ML devsite [F] |
| 13/08/2026 | Tópico "quotations" de Items descontinuado (VIS). | Sem impacto (imóveis/veículos). | produto-receba-notificacoes [F] |
| 24/08/2026 | Flex MLB: `price` das zonas retorna vazio; novo atributo `pricing` (preço por peso/zona). | Ajustar se o ERP mostrar o custo do Flex. | envios-flex [F] |
| fim de set/2026 | "Vista atual" de `/orders/{id}/shipments` descontinuada. | Usar o formato novo. | gerenciamento-de-vendas [F] |
| 14/09/2026 (atualização) | Notificações migrando para subtópicos; `missed_feeds` de `items` exige `site_id`. | Ajustar o job de recuperação. | produto-receba-notificacoes [F] |
| 25/10/2026 | `/items?ids=` e `/users?ids=` → `/items/bulk?ids=` e `/users/bulk?ids=`. | Já nascer no endpoint novo. | itens-e-buscas [F] |
| 27/10/2026 | Preço por quantidade com valor absoluto (`/prices/standard/quantity`) descontinuado. | Só relevante se for usado atacado. | busca ML devsite / precos-por-quantidade [F] |
| contínuo | `post_purchase`: `resource` passa a vir com o prefixo `/post-purchase`; revisão de devolução unificada em `return-review`; `allow-return` substitui aceitar/nova resolução. | Parser de resource genérico. | produto-receba-notificacoes, gerenciar-devolucoes, gerenciar-resolucao [F] |
| contínuo | `invoice_type` removido do billing-info; endpoint legado `/orders/{id}/billing_info` depreciado. | Lógica fiscal própria para o tipo de nota. | faturamento, FAQ billing-info [F] |
| contínuo | Coexistência de logísticas para o mesmo UP (Full + coleta + Flex). | Roteamento de estoque por logística. | mercado-envios, estoque-distribuido [F] |

---

## Recomendações arquiteturais derivadas (todas inferência [I])

1. **Camada "ML Gateway" dentro do ERP** (é ela que o agente de IA consome): `ml_accounts` (grant e tokens criptografados), `ml_events` (inbox de webhooks, dedupe por `_id`/resource+topic+sent), `ml_jobs` (outbox de escritas com rate limit por conta/recurso), `ml_snapshots` (último JSON por recurso, para diff).
2. **Receptor de webhook isolado** (Route Handler/edge leve) que grava e devolve 200; os workers (container separado no Portainer) processam. Reconciliação diária: missed_feeds + `orders/search` + claims abertos + estoque Full.
3. **Token broker** com renovação proativa (ex.: aos 5 h) sob lock por conta; `invalid_grant` marca a conta como "reconectar" e notifica.
4. **Modelo de produto nativo UP:** SKU interno ↔ `user_product_id` ↔ N `item_id` por conta; `inventory_id` por item/variação no Full; `logistic_type` por item.
5. **Estoque:** fonte única no ERP; o "disponível para publicar" de cada conta = físico − reservas − segurança, com regra de partilha entre as contas A e B; o Full é saldo à parte e só leitura.
6. **Fiscal em duas vias** (Faturador ML para o Full e NF-e própria + upload de XML para coleta/agência) — decidir com a contabilidade antes do schema.
7. **Agente de IA com humano no loop** no MVP: rascunha a resposta, valida as regras (350 caracteres, nada de links/dados pessoais, nada de template repetitivo), e o humano aprova; depois, autonomia gradual só em perguntas pré-venda.

---

## Fontes lidas

**API Mercado Livre — documentação oficial (developers.mercadolivre.com.br, lida via HTTP em 16/10/2026)**

- https://developers.mercadolivre.com.br/pt_br/autenticacao-e-autorizacao
- https://developers.mercadolivre.com.br/pt_br/produto-receba-notificacoes
- https://developers.mercadolivre.com.br/pt_br/gerenciamento-de-vendas
- https://developers.mercadolivre.com.br/pt_br/gestao-packs
- https://developers.mercadolivre.com.br/pt_br/gerenciamento-de-envios
- https://developers.mercadolivre.com.br/pt_br/mercado-envios-2
- https://developers.mercadolivre.com.br/pt_br/mercado-envios
- https://developers.mercadolivre.com.br/en_us/shipment-handling
- https://developers.mercadolivre.com.br/pt_br/envios-flex
- https://developers.mercadolivre.com.br/pt_br/custos-de-envio
- https://developers.mercadolivre.com.br/pt_br/envios-fulfillment
- https://developers.mercadolivre.com.br/en_us/fulfillment
- https://developers.mercadolivre.com.br/pt_br/convivencia-full-e-flex
- https://developers.mercadolivre.com.br/pt_br/estoque-distribuido
- https://developers.mercadolivre.com.br/pt_br/gestao-de-estoque-multiorigem-user-products
- https://developers.mercadolivre.com.br/pt_br/user-products
- https://developers.mercadolivre.com.br/pt_br/preco-variacao
- https://developers.mercadolivre.com.br/pt_br/variacoes
- https://developers.mercadolivre.com.br/pt_br/kits-virtuais
- https://developers.mercadolivre.com.br/pt_br/publicacao-de-produtos/
- https://developers.mercadolivre.com.br/pt_br/publicacao-no-catalogo
- https://developers.mercadolivre.com.br/pt_br/elegibilidade-de-catalogo
- https://developers.mercadolivre.com.br/pt_br/produto-sincronizacao-de-publicacoes
- https://developers.mercadolivre.com.br/pt_br/api-de-precos
- https://developers.mercadolivre.com.br/pt_br/itens-e-buscas
- https://developers.mercadolivre.com.br/pt_br/comissao-por-vender
- https://developers.mercadolivre.com.br/pt_br/obtendo-nota-fiscal
- https://developers.mercadolivre.com.br/pt_br/api-fiscal-faturamento-de-venda
- https://developers.mercadolivre.com.br/pt_br/envio-regras-tributarias
- https://developers.mercadolivre.com.br/pt_br/envio-dos-dados-fiscais
- https://developers.mercadolivre.com.br/pt_br/faturamento
- https://developers.mercadolivre.com.br/devcenter/faturamento-billing-info
- https://developers.mercadolivre.com.br/en_us/upload-invoices
- https://developers.mercadolivre.com.br/pt_br/relatorios-de-faturamento
- https://developers.mercadolivre.com.br/pt_br/boas-praticas-para-o-consumo-das-apis-de-relatorios-de-faturamento
- https://developers.mercadolivre.com.br/pt_br/pagamentos
- https://developers.mercadolivre.com.br/pt_br/gerenciar-reclamacoes
- https://developers.mercadolivre.com.br/pt_br/gerenciar-resolucao-de-reclamacoes
- https://developers.mercadolivre.com.br/pt_br/gerenciar-devolucoes
- https://developers.mercadolivre.com.br/pt_br/trocas-change
- https://developers.mercadolivre.com.br/pt_br/mensagens-post-venda
- https://developers.mercadolivre.com.br/pt_br/mensagens-bloqueadas
- https://developers.mercadolivre.com.br/pt_br/o-que-e-mensageria
- https://developers.mercadolivre.com.br/pt_br/perguntas-e-respostas
- https://developers.mercadolivre.com.br/pt_br/gerenciamento-perguntas-respostas
- https://developers.mercadolivre.com.br/pt_br/reputacao-de-vendedores
- https://developers.mercadolivre.com.br/pt_br/permissoes-funcionais
- https://developers.mercadolivre.com.br/pt_br/boas-praticas-para-usar-a-plataforma
- https://developers.mercadolivre.com.br/pt_br/gerenciar-ips-de-um-aplicativo
- https://developers.mercadolivre.com.br/pt_br/erros
- https://developers.mercadolivre.com.br/en_us/start-testing
- https://global-selling.mercadolibre.com/devsite/en_us/specific-regulations/question-3 (FAQ de rate limit, CBT)
- Resultados de busca no devsite ML (resumos das páginas automatizacoes-de-precos, product-ads-leitura, compatibilidades, precos-por-quantidade, campanhas/seller-promotions, Global Selling "Fulfilment Inbound Creation")

**Mercado Pago**

- https://www.mercadopago.com.ar/developers/en/docs/reports/released-money/api (via busca)
- https://www.mercadopago.com.ar/developers/en/reference/releases-report/create-report/post (via busca)
- https://www.mercadopago.com.br/developers/pt/docs/checkout-pro/additional-content/reports/account-money/introduction (via busca)

**Hubs e ERPs**

- https://www.ideris.com.br/ · https://www.ideris.com.br/planos/ · https://www.ideris.com.br/mercado-livre/
- https://www.ideris.com.br/blog/calculadora-de-precos/ · /blog/conciliacao-financeira-marketplace/ · /blog/mercado-livre-full/ · /blog/ideris-ajuda-voce-a-vender-mercado-livre/
- https://suporte.ideris.com.br/pt-BR/articles/5640182-replicando-anuncios-entre-contas-mercado-livre (conteúdo via snippet de busca; o domínio não resolveu no fetch)
- https://suporte.ideris.com.br/pt-BR/articles/6055348-dominando-a-calculadora-de-precos-do-ideris (via snippet de busca)
- https://centrodepartners.mercadolivre.com.br/apps/ideris (via busca)
- https://www.anymarket.com.br/funcionalidades/ · https://www.anymarket.com.br/
- https://marketplace.anymarket.com.br/regras-de-estoque-anymarket/ · /controle-de-precos/ · /5-funcionalidades-do-anymarket-que-poderao-te-ajudar-a-aumentar-suas-vendas-nos-marketplaces/
- https://suporteanymarket.freshdesk.com/pt-BR/support/solutions/articles/19000105005 (múltiplos anúncios do mesmo produto)
- https://suporteanymarket.freshdesk.com/pt-BR/support/solutions/articles/19000133323 (XML obrigatório ME2, `invoice_pending`)
- https://plugg.to/funcionalidades/
- https://ajuda.magis5.com.br/magis5-hub
- https://ajuda.olist.com/pt_BR/mercado-livre/erp-integracao-mercado-livre-fulfillment
- https://ajuda.bling.com.br/hc/pt-br/articles/360042034893 (envio de dados da nota ao ML)
- https://atendimento.tecnospeed.com.br/hc/pt-br/articles/29035043810711 (erros de envio de XML de NF ao ML)

---

## O que não foi possível confirmar

1. **Endpoint exato para enviar o XML da NF-e própria ao envio ME2 no MLB.** As regras (`invoice_pending`, `Content-Type: application/xml`, assinatura, UF de origem, erros `shipment_invoice_already_saved`, `error_parse_invoice_data`, `malformed_XML`) foram confirmadas por terceiros (Anymarket, Bling, Tecnospeed) e pela FAQ do ML, mas a página oficial com o endpoint não foi localizada. **[I]** Pela prática do mercado, o recurso é `POST /shipments/{shipment_id}/invoice_data?siteId=MLB` — validar em teste antes de implementar.
2. **Limite global de requisições por app/vendedor.** A doc lida só traz limites por recurso (500/100/1000 rpm) e o 429 genérico. Os números "240/min" e "1500/min por seller" que circulam vêm de outro provedor (developer.me.com.br) e **não** se aplicam ao ML.
3. **Credenciais dos relatórios Mercado Pago (Liberações/Settlement) com o token OAuth do ML.** A doc de packs indica `api.mercadopago.com/v1/payments/{id}`; não confirmei se o token do app ML basta para `/v1/account/release_report` ou se é preciso um app MP próprio.
4. **Regras internas da Ideris** (central de ajuda `suporte.ideris.com.br` inacessível do ambiente). Replicação, Full e calculadora foram vistos só por snippets de busca e pelo blog.
5. **Se as duas contas do cliente são "contas vinculadas" pelo ML** e qual o risco concreto de penalidade por catálogo duplicado entre elas: a política só é descrita por blogs e hubs, sem regra oficial da API.
6. **Se a conta do cliente já tem `user_product_seller` e/ou `warehouse_management`.** É preciso consultar `/users/{id}` com o token real; isso muda o endpoint de estoque.
7. **Limite de imagens por anúncio e de caracteres na resposta de pergunta** no modelo atual: não confirmados na leitura (a doc fala em "até seis imagens" no exemplo legado, provavelmente desatualizado).
8. **Ano da mudança do `fixed_fee` por logística** (a página diz apenas "Brasil: 02/03"; a última atualização é de 03/09/2026 → provavelmente 2026).
9. **Custo de armazenagem do Full via API:** presumo que vem nos detalhes de billing (`detail_sub_type`), não verificado.
10. **`POST /items/{id}/prices/standard`** consta na doc como "ainda não disponível". Não confirmei se já foi liberado no MLB.
