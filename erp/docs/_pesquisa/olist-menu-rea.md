# Olist ERP — árvore de menus e relatórios (engenharia reversa via REA)

Coletado em 16/10/2026 na conta do dono, em modo somente leitura (Chrome com perfil dedicado + REA `capture_browser_scenario` / `analyze_web_bundle`).

## Arquitetura observada
- Aplicação **híbrida**: base legada em PHP com jQuery 1.11 + **xajax** (chamadas `*.server.php` / `*.services.php`) e "ilhas" novas em **React** (chunks `build/*.js`, roteamento compatível com React Router). Versão dos assets: `3.92.62`.
- Cada tela é uma página do servidor (`/produtos`, `/vendas`, `/estoques` …) com sub-rotas por hash (`#list`, `#add`, `#/ecommerce/edit/{id}`).
- Chunks React expostos (pistas de features recentes): `MapeamentoCategoriasEcommerceModal`, `MapeamentoProdutosRegraPrecoEnum` (regra de preço por mapeamento), `PrecosAnuncioView` (preço por anúncio), `TipoAnuncioEnum`, `IntegracaoEcommerceEnum`, `IntegracaoRecursosEnum`, `SituacaoVendaEnum`, `PesoCorteCubadoEnum` (peso cubado), `EventosPDVEnum`, `ContaDigital`/`OlistPay`, `usePermissaoACL` (ACL de permissões), `featureFlags`, `TesteAB`.
- Autenticação: `auth.services.php`, `multi.empresa.auth.services.php` (multiempresa), Keycloak citado no bundle.
- Evidências REA: `ev_c974ae69…f891` (bundle), `ev_fc0f3ccc…89e4` (targets).

## Barra de apps (topo)
ERP · PDV · Envios (Olist Envios) · Ecommerce (loja virtual Olist) · Lis (agente de IA da Olist, `agents.olist.com`) · Crédito (antecipação de recebíveis).

## Menu do ERP
| Módulo | Itens |
|---|---|
| **Início** | Dashboard: "Pedidos nos últimos 7 dias" por canal (ex.: Mercado Livre), widgets configuráveis, "Extensões do Sistema ERP" (loja de aplicativos) |
| **Cadastros** | Clientes e Fornecedores · Produtos · Anúncios · Campanhas Promocionais · Categorias dos Produtos · Vendedores · Embalagens · Relatórios |
| **Suprimentos** | Controle de Estoques · Envios Fulfillment · Ordens de Compra · Serviços Tomados · Notas de Entrada · Conferência de compra · Necessidades de Compra · Giro de Estoque · FCI · Relatórios |
| **Vendas** | Crédito da Olist · CRM · Painel de Automações · PDV · Propostas Comerciais · Pedidos de Venda · Notas Fiscais · Comissões · Performance de Vendas · Margem Contribuição · Custos do e-commerce · Separação · Expedição · Devoluções de venda · Pedidos no e-commerce · Perguntas do e-commerce · Pós-venda Mercado Livre · Relatórios |
| **Finanças** | Caixa · Contas a Pagar · Contas a Receber · Cobranças Bancárias · Extratos Bancários · Relatórios |
| **Configurações** | Abas Geral · Notas Fiscais · E-commerce · Tributação (RTC, novo). Itens: dados da empresa, dados do usuário, cadastro de usuários, servidor de e-mail, envio de documentos, etiquetas, agenda; Outras: interface, central de notificações, impressão PrintNode, Multiempresa, Aplicativos, Token API, Configurações de API, Webhooks |
| Rodapé | Canal de ideias · Indique e ganhe · tema claro/escuro/auto |

## Relatórios por módulo
**Cadastros:** sincronização de produtos com e-commerce (log de interações produto/estoque) · preços dos produtos (lista de preço por tags) · relatório personalizado e "criar relatório com a Lis" (IA).

**Suprimentos:** *personalizado da conta:* "Inventário Galpão" · Estoque Multi Empresas · Entradas e Saídas · Saldos em Estoque (em data qualquer) · Produtos com Maior Circulação · Produtos sem Movimentação · Abaixo do Estoque Mínimo · Visão Financeira do Estoque (última compra × preço de venda) · Movimentação de um Produto · Lotes e validades · NF de compra por operação / por fornecedor / por produto / evolução / por fornecedor (fantasia) · Ordens de compra · Serviços tomados.

**Vendas:** Vendas (agrupável por cliente/produto/vendedor, com lucratividade por custo ou última compra) · Financeiro de Vendas (taxas, tarifas, meio e forma de pagamento) · Curva ABC (clientes ou produtos) · CRM · Expedição · Consulta de itens de NF · NF por operação / cliente / produto · Evolução do faturamento · Partilha de ICMS (DIFAL) · Controle de caixa do PDV e movimentações · Pedidos para produção (ficha técnica) · Propostas comerciais · Desempenho da separação.

**Finanças:** Balancete · DRE · Fluxo de Caixa (projeção) · Entradas e saídas por categoria / por cliente · Contas a pagar · Recebimentos · Contas a receber.

## Sinais de uso da conta (agregados)
- Integrações ativas: **Mercado Livre**, **Mercado Livre Fulfillment**, **Shopee**, Amazon, Amazon FBA Classic, Magalu Marketplace, TikTok Shop. Só **um** card "Mercado Livre" aparece, então é preciso confirmar onde está a 2ª conta (multiempresa?).
- Anúncios ativos: ML 52 · ML Fulfillment 45 · Shopee 19 · demais canais com poucos ou nenhum.
- Produtos: 45 (37 simples, 7 kits, 1 com variações); há itens tipo "Matéria-prima" disponíveis no filtro.
- Depósito padrão: "Galpão". Relatório personalizado "Inventário Galpão" e relatório "Estoque Multi Empresas" no menu.
- Colunas do controle de estoque: produto, SKU, preço, **custo médio**, estoque físico, **reservado**, disponível, unidade, **localização**.
- Envios Fulfillment: abas Em aberto · Separação · Embalagem · Envios Prontos · Envios finalizados. Sem registros nos últimos 30 dias.

## Novo produto, aba "Dados Gerais" (campos)
Tipo (Simples · Kit · Com variações · Matéria-prima) · Tipo do item SPED · Nome · GTIN · Origem ICMS (0–8) · Unidade · NCM (com sugestão da Lis) · SKU · CEST · Preço de venda · Preço promocional · Peso líquido/bruto · Tipo de embalagem (envelope, caixa, rolo) · Embalagem (customizada ou caixas padrão Correios/Flex) · L×A×C · Controlar estoque · Estoque inicial (por depósito) · Estoque mín./máx. · Sob encomenda · Controlar lotes · Localização · Dias para preparação.
Abas do produto: Dados Gerais · Dados Complementares · Ficha técnica · Anúncios · (2 abas condicionais) · Preços · Custos · Outros.

## Integração Mercado Livre (`integracoes#/ecommerce/edit/{id}`)
Abas: Conexão · Canais de Venda · Produtos · Pedidos · Mapeamentos. Status "conectada". O agente de varredura detalha o conteúdo de cada aba.

Evidências REA: `ev_9e3645aa…e463`, `ev_697b3ea1…78de` e cenários de menu e relatórios.
