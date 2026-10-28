# Olist ERP (ex-Tiny) — mapeamento da conta via REA

> Engenharia reversa de UI, **somente leitura**, na conta do dono (autorizado). Coletado em 2026-10-16 com `mcp__rea__capture_browser_scenario` (modo connect, snapshot de acessibilidade por passo). Cada bloco cita o `evidence_id` do cenário REA.
> Regra: só estrutura (telas, abas, ações, colunas, campos, opções) + **sinais agregados** de uso. Nenhum dado pessoal/comercial registrado.
> Complementa o que já estava mapeado (menu principal, relatórios, lista de configurações, lista de integrações, anúncios por canal, lista de produtos/estoques, Envios Fulfillment, aba "Dados Gerais" do produto).

## Resumo executivo — como o dono usa o Olist hoje
- **Operação 100% marketplace**, com forte peso do **Mercado Livre Full**: ~1.079 pedidos/30 dias, ~1.000 NF de saída e ~1.000 de entrada/30 dias quase todas "Fulfillment" emitidas pelo ML e importadas.
- **Catálogo pequeno** (45 produtos: 37 simples, 7 kits multipack, 1 com variação) anunciado em vários canais (ML, ML Full, Shopee, TikTok; Amazon/Magalu integrados).
- **Estoque multi-depósito** (Galpão + Devoluções + ML/Amazon/Shopee Full), reserva de estoque ligada, saída ao "marcar como enviado", estoque negativo permitido, estoque de segurança 3 no ML.
- **Devolução é processo grande** (822 registros).
- **Não usa**: compras/OC, conferência, financeiro (receber vazio, caixa parado, 7 pagar atrasadas), CRM, propostas, vendedores/comissões, metas, margem, expedição, PDV, multiempresa, API/webhooks.
- **Usuários**: dono (admin), 2 da contabilidade, 1 logística (perfil customizado), 1 gerente.
- Automações ligadas: saída de estoque ao enviar; enviar para separação e expedição ao autorizar NF. Emissão/autorização automática de NF **desligada**.

---

## Marketplaces / hub de integrações

### Lista de integrações — `https://erp.olist.com/integracoes`
- Ações: **Adicionar integração**, busca "Pesquise pelo nome da integração", alternância de visualização (2 botões ícone).
- Cards "Minhas Integrações" (cada um com link **gerenciar**): Mercado Livre (Marketplace), Shopee, Mercado Livre Fulfillment, Amazon, Amazon FBA Classic, Magalu Marketplace, TikTok Shop.
- Evidência: `ev_7b92cc6eb301ebb695f9927140b2bd516dae050fa56e748fb2b82b67a5d739af`

### Integração Mercado Livre — `integracoes#/ecommerce/edit/9398`
Evidências: `ev_a43448d5beed6d5fe68a4926646e692af6b77ae52bccf3ca51484f3d7cd9eea1` (abas), `ev_786e0a5c531e472e4e813da82dcc68531bbe63544e45ddf933c27b6b184a7071` (sub-abas Produtos/Pedidos), `ev_376a171f65ff31172b28f073797f69bc11af13ec746cb87c6fb5215e1523d780` (Mapeamentos).
Botão global: **Salvar** (não clicado). Link "Acesse a ajuda sobre a integração".

Abas: **Conexão · Canais de Venda · Produtos · Pedidos · Mapeamentos**

**Conexão**
- Campo "Nome da integração no ERP" (texto). Card do canal com status **conectada** + link **gerenciar** (abre "Conectar Integração": nome, combo de conta-mãe, bloco "Autenticação" com botão "conectar com …").

**Canais de Venda**
- "Adicionar canais de venda" — permite subdividir pedidos de uma mesma integração por canal. Canal disponível: **Mercado Shops** (link "adicionar"). Nenhum canal extra adicionado.

**Produtos** (sub-abas)
- *Enviar Produtos*: Enviar imagens ao sincronizar (Sim / **Apenas para novos produtos**); [ ] ao atualizar anúncio enviar descrição complementar; Identificador da loja oficial (busca, desabilitado); Garantia fornecida por (**Garantia do vendedor** / Garantia de fábrica); [ ] Enviar dias para preparação ao sincronizar.
- *Receber Produtos*: Ao importar produtos (**Importar imagens como anexos** / Apenas URL externa); [ ] ao atualizar produto, atualizar Código (SKU); [x] Importar anúncios com uma variação como anúncios simples; [x] Importar apenas produtos ativos no ML.
- *Regras de Estoque*: Atualizar estoque na loja a partir do ERP (Não atualizar / Enviar saldo físico / **Enviar saldo disponível** = físico − reservas); tabela **multi-depósito** (Depósito no ERP · ID no e-commerce · Depósito no e-commerce); **Depósito padrão da integração** (opções: Galpão [selecionado], Devoluções, "Todos próprios"); **Lançamento de estoque para saídas** (Usar configuração padrão da conta [sel.] / Manual / Automático ao autorizar NF / Automático ao salvar NF / Automático ao salvar o pedido / Automático ao marcar pedido como enviado) — aviso: conta tem **reserva de estoque habilitada** (reserva automática conforme situação do pedido); [x] **Utilizar estoque de segurança** = 3,00 (anúncio vai a zero quando atinge a margem).
- *Preços*: Regra de preço para o e-commerce (**Preço fixo** / Conforme lista de preços / Conforme regras de preço / Calculadora de preços); [ ] Sugestão de preços (beta).
- *Tipo do Anúncio*: padrão (Grátis / **Clássico** / Premium), alterável por anúncio.

**Pedidos** (sub-abas)
- *Fluxo de Pedidos*: [x] Importar valor do frete; [ ] Formatar nome dos contatos (iniciais maiúsculas); [x] **Sincronização automática de pedidos**; [x] Importar pedidos sem dados de entrega; Completar endereço pelo CEP (Não / **Apenas se bairro não informado** / Sempre); [x] Adicionar endereço do cliente nas observações; [x] **Utiliza Mercado Envios**; [ ] Utiliza Mercado Envios Coletas; [x] Importar parcelas dos pedidos; Método de impressão de etiquetas (**Etiqueta fornecida pelo ML** / Etiqueta com DANFE unificada (beta) — exige impressora térmica + PDF no ML); [x] Enviar rastreamento ao mudar situação no ERP para enviado; [x] Marcar como entregue ao alterar situação no ERP; Valores divergentes NF × pagamento integrado (**Usar valor do pagamento integrado (recomendado)** / Considerar "Não integrado").
- *Dados Fiscais*: Envio de dados da NF-e ao ML (**Automático ao autorizar NF** / Manual); conteúdo (**Dados por mensagem e XML** / Apenas XML / Apenas mensagem); Natureza de operação p/ transferência (sel.: "Outras Saída - Remessa para Depósito Temporário"); Natureza de operação p/ venda (sel.: "Venda de mercadorias"); [ ] natureza diferente para contribuintes do ICMS.

**Mapeamentos** (sub-abas)
- *Situações de Pedidos*: botão "Adicionar situações padrão", [ ] Permite retroceder situação na sincronização; tabela Código no e-commerce → Situação no ERP. Mapeamento atual: pending→Em aberto · handling→Preparando envio · paid→Aprovado · shipped→Enviado · delivered→Entregue · not_delivered (Enviado não entregue) · cancelled→Cancelado · ready_to_ship (Pronto para envio). Botão "Adicionar mapeamento de situação".
  - **Situações do pedido no ERP** (opções do combo): Em aberto, Faturado, Cancelado, Aprovado, Preparando envio, Enviado, Entregue, Pronto para envio, Dados incompletos, Não entregue, Em devolução.
- *Formas de Recebimento*: [x] Importar forma de recebimento; **Forma de recebimento padrão** (opções da conta: Não definida, Dinheiro, Boleto, Cheque, Pix, Link de pagamento, **Mercado Pago** [sel.], Shopee); **Categoria financeira padrão dos pedidos** (sel.: "Vendas Mercado Livre"); tabela Forma de recebimento · Meio de recebimento · Código na integração.
- *Formas de Frete*: tabela Forma de envio · Forma de frete · Código na integração. Formas de envio da conta: Correios, Transportadora, Retirar pessoalmente, **Mercado Envios**, Shopee Envios, Amazon DBA, Magalu Fulfillment, Olist Envios. Formas de frete mapeadas (Mercado Envios): Coleta Normal, Coleta Express, Coleta Correios PAC Agência, Coleta Correios Sedex Agência, Shipping Network Fast, Mercadoenvios Next Day (~8 códigos).
- *Categorias*: Categoria no ERP ↔ código de categoria ML (MLBxxxx) — **~34 categorias mapeadas**.
- *Variações*: botões "Relacionar valores pelo nome", "Excluir variações não vinculadas"; tabela Variação no e-commerce · Variação no ERP · Opção no e-commerce · Opção no ERP. Tipos de variação no ERP: **Cor, Aroma, Voltagem** (opções de aroma, cor, 127V/220V — há duplicatas/erros de digitação nas opções, ex. "Ganito", "Peto", "127 V").
- *Meus horários de envio*: prazos de despacho por forma de frete cadastrada no ML — "Nenhum horário de despacho cadastrado".

Evidência Mapeamentos: `ev_376a171f65ff31172b28f073797f69bc11af13ec746cb87c6fb5215e1523d780`.

### Integração Mercado Livre Fulfillment — `integracoes#/ecommerce/edit/10681`
Evidências: `ev_7b92cc6eb301ebb695f9927140b2bd516dae050fa56e748fb2b82b67a5d739af`, `ev_4f60bff894780be641f8c98b8fdc1e07dfaa32fe5ad4e6a3b2eeadea700fd19b`.
Abas: **Conexão · Produtos · Pedidos · Mapeamentos · Outros** (sem Canais de Venda).
- *Conexão/gerenciar*: "Conectar Integração" com combo **"Integração com o MercadoLivre"** → única opção "Mercado Livre" (o Full é vinculado à integração ML principal). Botão "conectar com Mercado Livre Fulfillment" (não clicado).
- *Produtos*: mesmas sub-abas do ML; Enviar imagens = **Sim**. *Regras de Estoque*: "Depósito a ser considerado" (opções: **ML Full** [sel.], Amazon Full, Shopee Full, Todos próprios); Lançamento de estoque p/ saídas Fulfillment = **Automático ao marcar pedido como enviado**.
- *Pedidos/Fluxo*: [x] frete; [x] **Sincronização automática de pedidos e notas fiscais**; [x] formatar nome; [x] pedidos sem dados de entrega.
- *Pedidos/Dados Fiscais*: igual ML + **Modelo fiscal no Mercado Envios Full** (**Self storage – Portaria CAT 59/2018** [sel.] / Modelo filial); [x] Importar automaticamente NF-e de devolução (emitidas via fulfillment); [x] Importar outros documentos fiscais (transferência) (CT-e e CC-e não); Parcelas (XML / venda).
- *Mapeamentos/Situações*: no Full, pending/handling/paid/ready_to_ship/shipped → **Enviado**, delivered→Entregue, cancelled→Cancelado. Sub-abas: Situações, Formas de Recebimento, Formas de Frete, Categorias, Variações.
- *Outros*: "Configurações Adicionais" — Parcelas presentes no XML / na venda.

**Sinais de uso (marketplaces)**
- **1 conta Mercado Livre** conectada (só um card ML; o Full aponta para a única integração ML). ML Full ativo.
- 7 integrações: ML, ML Full, Shopee, Amazon, Amazon FBA Classic, Magalu, TikTok Shop.
- Depósitos citados: **Galpão** (padrão ML), **Devoluções**, **ML Full**, **Amazon Full**, **Shopee Full** (+ agregado "Todos próprios") → operação multi-fulfillment (ML, Amazon, Shopee).
- Estoque enviado = saldo disponível com estoque de segurança 3; reserva de estoque habilitada na conta.
- Pedidos ML importados automaticamente; etiqueta do próprio ML; envio automático dos dados da NF-e ao autorizar (config. pronta para emissão pelo ERP). Porém o módulo Fiscal mostra que o grosso das NF são do **ML Full emitidas pelo ML e importadas** — ver Fiscal.
- Recebimento padrão ML = Mercado Pago; categoria financeira "Vendas Mercado Livre"; formas de recebimento cadastradas incluem Shopee.
- Variações usadas: Cor, Aroma, Voltagem (catálogo com aromas/cores/voltagem).
- Natureza "Remessa para Depósito Temporário" usada para transferência ao Full.

---

## Cadastros — Produtos

### Lista de produtos — `produtos#list`
Evidências: `ev_48e537788ba23d2b14794da32ce211b2573e446ef9a3edb9362087c39c3aa4bd`, `ev_5ab3664eaa118232741cbfb6b63cab5223f6cf476feaaac94d3003a8fef0ec50`
- Ações topo: **Receber do e-commerce** (não clicado), **Incluir produto**, **Mais ações**.
- **Mais ações**: Receber produtos do e-commerce · Imprimir relatório · Iniciar histórico de custos · Reajustar preço dos produtos · Editar CEST dos produtos em lote · Produtos com problemas fiscais · Gerenciar preços dos anúncios · Consultar último lote de sugestão de NCM · Exportar produtos para planilha · Exportar composição de kits para planilha · Exportar estrutura de fabricados para planilha · Importar produtos de uma planilha.
- Busca "nome, código (SKU) ou GTIN/EAN"; filtros rápidos: ordenar por nome, "produtos ativos", filtros, limpar filtros.
- Abas de tipo (com contadores): **Todos · Simples · Kits · Variações · Matéria-prima**.
- Colunas: checkbox · (ações) · imagem · Descrição · Código (SKU) · Unidade · Preço (mostra preço riscado + promocional) · Estoque físico · Estoque disponível · Integrações (ícones por canal + ícone "Mapeamento com erro").

### Produto — visualização `produtos#edit/{id}` (modo leitura) e edição (botão **Editar**)
Evidências: kit `ev_031705e673289cd3036f00b89917d20d98ddbbe4108a9120c438f1d1aa466a27` (leitura), `ev_268fcfd4ecf4662f4c1e12243c4ae2ae4b876ad27734ef1f357784afc3dab281` (edição); variação `ev_db5bdcc00eb1dbb31007ebe0758b9ab62ee1767f7b4a65505d157393331a31d5`
- Abre primeiro em **modo leitura** (rótulo + valor). Botões: **Enviar para o e-commerce**, **Editar**, **Mais ações**.
- **Mais ações (produto)**: Enviar para o e-commerce · Enviar preços para o e-commerce · Enviar estoque ao e-commerce · Enviar dados fiscais para o e-commerce · Consultar Estoque Multiempresa · Imprimir etiquetas · Visualizar histórico de vendas · Criar um pedido · Clonar produto · Editar tags · Enviar produto para empresas (multiempresa).
- Abas: **Dados Gerais · Dados Complementares · Ficha técnica · Anúncios · [Kits | Variações] · Preços · Custos · Outros** (a aba Kits/Variações aparece conforme o tipo).
- Rodapé da edição: **Salvar** · **Alterar regras** · **Cancelar** (nenhum clicado).

**Dados Gerais (complemento do já mapeado — visto no produto com variações)**
- Tipo do Produto (Simples / Kit / Com variações / Matéria-prima — bloqueado após criado); **Tipo do item SPED** (Mercadoria para Revenda, Matéria-Prima, Embalagem, Produto em Processo, Produto Acabado, Subproduto, Produto Intermediário, Material de Uso e Consumo, Ativo Imobilizado, Serviços, Outros insumos, Outras); Nome; GTIN; **Origem ICMS** (0 a 8); Unidade; NCM (botão "Sugerir" + "Utilize a Lis para sugerir o NCM"); SKU; CEST; Preço de venda; Preço promocional.
- Dimensões e peso: Peso líquido/bruto; **Tipo da embalagem** (Envelope / Pacote-Caixa / Rolo-Cilindro); **Embalagem** (Embalagem customizada, Caixas de Encomenda Correios Flex/CE-01/02/03/07/5B/6B/Vai e Vem/B/2B/4B/Temáticas 01-03, "Criar nova embalagem…"); Largura/Altura/Comprimento.
- Estoque: Controlar estoque (Sim/Não); **Depósito padrão** (Amazon Full, Devoluções, Galpão, ML Full, Shopee Full); Estoque mínimo; Estoque máximo; Sob encomenda (Sim/Não); **Controlar lotes**; Localização; Dias para preparação.

**Dados Complementares**
- Categorização: Categoria (árvore de categorias, ex. caminho de 6 níveis), **Marca** (busca), **Tabela de medidas** (busca).
- Descrição complementar (editor rico + "Editar HTML") — usada em propostas, pedidos e descrição do e-commerce.
- **Imagens e anexos**: gerenciador ("Gerenciar imagens"; imagens compartilhadas entre pai e variações).
- Campos adicionais: Link do vídeo, Slug, Keywords, Título SEO, Descrição SEO.
- **Tags** (classificação livre; botão "Editar tags").

**Ficha técnica**: tabela Atributo · Valor; "Sugerir atributos"; "Adicionar atributo" (busca de atributo).

**Anúncios**: texto "ao enviar ao e-commerce um anúncio é criado; anúncios podem ser adicionados manualmente para vincular a já publicados ou vários anúncios do mesmo produto". Tabela: E-commerce · Identificador (com link "ver anúncio") · Descrição (título do anúncio por canal) · ações. Status visível (ex. "Rejeitado" no TikTok Shop). Configurações avançadas: **Notificação de estoque** (Sim/Não).

**Kits** (produto tipo Kit): tabela Produto/SKU · Quantidade/Unidade · Preço Un./Custo · Preço Promocional · Preço Total · Ações (Editar/remover); linha Total (preço e custo total). Botões: **Adicionar produto no kit**, **Atualizar preço do produto** (a partir dos componentes), **Atualizar peso do produto**. Estoque do kit é derivado (Controle de estoque avisa "Não é possível atualizar o estoque de um kit").

**Variações** (produto "Com variações"): bloco "Variações do produto" — tipo (ex. Voltagem) + valores digitados separados por vírgula/tab (ex. 220V, 127V), "Adicionar outra variação"; **Grade das variações**: Variação · Código (SKU) · Preço · Estoque · Imagens · editar/remover. Variações sem imagem herdam as do pai.

**Preços**: tabela Lista de preço · Preço · Preço promocional; "Informar novo preço" (combo de listas — **nenhuma lista de preço cadastrada**).

**Custos** (histórico de custo): A partir de · Saldo atual · Saldo anterior · Impostos recuperáveis · Preço custo · Custo médio · Preço venda; "Informar novo preço de custo"; editar/excluir linha.

**Outros**: Unidade por caixa; Linha de produto (combo — nenhuma cadastrada); Garantia (texto, ex. "7 dias"); Markup; Data de criação (somente leitura); Permitir inclusão nas vendas (Sim/Não). *Informações tributárias adicionais*: GTIN/EAN tributável, Unidade tributável, Fator de conversão, Código de Enquadramento IPI, Valor do IPI fixo, EX TIPI. *Fornecedores*: tabela Nome · Código no fornecedor ("Adicionar fornecedor"). Observações gerais (uso interno).

**Sinais de uso (produtos)**
- **45 produtos**: 37 simples, 7 kits, 1 com variações, 0 matéria-prima.
- Kits montados a partir de unidades (ex. kit de 5 = 5× um item unitário) — kits funcionam como "multipacks" do mesmo SKU base, além de combos.
- Produtos anunciados em vários canais simultaneamente (ML, ML Full, Shopee, TikTok Shop); vários com ícone "Mapeamento com erro" (Shopee).
- Sem listas de preço, sem linha de produto, sem fornecedor vinculado no produto observado; histórico de custo com 1 registro (custo inicial). Mix: ferramentas/jardinagem/cerca elétrica/construção.
- Variações usadas só por voltagem (127V/220V) no único produto com variações.

## Cadastros — Contatos, vendedores, embalagens, promoções, categorias

Evidências: `ev_fa47cfa6a6fab12132a50fd5a3d3772b59dc5326c6611ae942305fc86f39fcd4` (lista de contatos), `ev_8945339a537644f136e78837d54ac548f70a84700ae7e460d1475c18a2f152d1` (form de contato, vendedores, embalagens, promoções, categorias)

### Clientes e fornecedores — `contatos#/`
- Topo: imprimir, **incluir cadastro**, Mais ações (menu não abriu via automação). Busca por nome/código/fantasia/e-mail/CPF-CNPJ; ordenação por data do cadastro / nome; filtro por situação; filtros.
- Abas por **tipo de contato**: **Todos · Cliente · Fornecedor · Transportador · Outro**. Colunas: Nome · CPF/CNPJ · Cidade · Contato. Paginação.

### Novo contato — `contatos#/add`
- Abas: **Dados Gerais · Dados Complementares · Anexos · Observações**.
- Dados gerais: Nome/Razão social, Fantasia, Código (opcional), **Tipo de pessoa** (PJ / PF / Estrangeiro / Estrangeiro no Brasil), CNPJ/CPF (com consulta), **Contribuinte** (1 – Contribuinte ICMS / 2 – Isento / 9 – Não contribuinte [padrão]), Inscrição Estadual, Inscrição Municipal, **Tipo de contato** (multi-seleção).
- Endereço: CEP, Município, UF, Endereço, Bairro, Número, Complemento; [ ] endereço de cobrança diferente.
- Contato: Telefone, Telefone adicional, Celular, WebSite, E-mail, **E-mail para envio de NF-e**, Observações do contato.
- **Pessoas de contato**: Nome · Setor · E-mail · Telefone · Ramal (Adicionar contato). Rodapé salvar/cancelar (não clicado).

### Vendedores — `vendedores#list`
- Incluir vendedor; busca nome/código; abas **Ativos com acesso ao sistema · Todos · Ativos · Inativos · Excluídos**. **Nenhum vendedor.**

### Embalagens — `embalagens`
- **Criar embalagens Correios**, Incluir embalagem; colunas Descrição · Tipo (Pacote/Caixa, Envelope, Rolo) · Dimensões · Peso. **37 embalagens** — praticamente só o catálogo padrão Correios (Caixas Flex/CE/B/Temáticas, Envelopes).

### Campanhas promocionais — `promocoes`
- Incluir campanha; busca; filtros "campanhas criadas", período. **Nenhuma campanha.**

### Categorias dos produtos — `produto_categorias`
- **Informar atributos**, Incluir grupo de categorias; colunas Descrição · Árvore. Árvore com ~14 raízes no padrão do Mercado Livre (Acessórios para Veículos, Agro, Alimentos e Bebidas, Arte/Papelaria, Beleza, Casa/Móveis/Decoração, Celulares, Construção, Eletrônicos, Esportes, Ferramentas, Festas, Indústria e Comércio, Saúde…), todas com subcategorias.

**Sinais de uso (cadastros)**
- **46.610 contatos** (≈ 933 páginas) — praticamente todos clientes criados automaticamente pela importação de pedidos dos marketplaces (muitos com CNPJ de MEI do comprador). Base de clientes é subproduto, não CRM.
- Sem vendedores, sem campanhas; embalagens só padrão Correios; árvore de categorias espelhada do ML (usada no mapeamento de categorias da integração).

---

## Estoque

### Controle de estoques — `estoques`
Evidências: `ev_db5bdcc00eb1dbb31007ebe0758b9ab62ee1767f7b4a65505d157393331a31d5`, `ev_d7fff98e9b1c13ad15e227670ed05a0d8970f8a2691925aa61bb095a1ae74677`, `ev_95ccd4b6f44ae89cc3b86a974305832b4f224982b6b1b7e51c2a8f82f143cfb7`, `ev_e0d8a055e84425eaf6ef9c7287471b7157e56f7f27e894b646ccef7614d69834`
- Botões topo: **Inventário de estoque**, **Transferir entre depósitos**, **Gerenciar produtos**, **Mais ações** (Transferir entre depósitos · Inventário de estoque · Imprimir relatório).
- Abas: Todos · Simples · Kits · Matéria-prima. Colunas: Produto · SKU · Preço · **Custo médio** · Estoque físico · **Estoque reservado** · Estoque disponível · Unidade · **Localização**.
- **Transferir entre depósitos** (modal): Depósito de origem → Depósito de destino → **Continuar** (não clicado) / Cancelar.

### Estoque do produto — `estoque?buscaid={id}&deposito=true`
- Botões: **Incluir reserva**, **Incluir lançamento**, Ações, Transferir entre depósitos.
- Indicadores: Saldo físico · Total reservado · Total disponível · **Disponível multiempresa** · Lançamentos (Entradas/Saídas) · Quantidade reservada.
- Filtros: por depósito (ex. "depósito Galpão"), por período, por situação/tipo; abas **Lançamentos** e **Reservas**.
- Tabela de lançamentos: Data e hora · Entrada · Saída · Valor · Observação · Tipo.
- **Incluir lançamento** (modal): Tipo (**Entrada / Saída / Balanço**), Data, Hora, Quantidade, Preço unitário (pré-preenchido com custo), Observações → Salvar/Cancelar (não salvo).
- Kits não aceitam lançamento direto ("Não é possível atualizar o estoque de um kit").

### Inventário — `inventario` → `relatorio_estoque_inventario`
- Três modos: **Gerar inventário** (visualização, edição rápida e download), **Importar por planilha** (.xls/.xlsx/.csv), **Importar por coletor** (.txt de bipagem).
- Gerar inventário: Busca, filtro por tags, **Depósito**, Exibir (Todos / Com saldo / Sem saldo), Situação do produto (Ativos / Inativos / Todos exceto excluídos), Fornecedor, Valor baseado em, [ ] Exibir estoque disponível, [ ] Exibir imagem → **Gerar** (não clicado).

**Sinais de uso (estoque)**
- **5 depósitos**: **Galpão** (padrão), **Devoluções**, **ML Full**, **Amazon Full**, **Shopee Full**.
- Reserva de estoque habilitada; estoque disponível = físico − reservado.
- Saída de estoque do Full lançada "ao marcar pedido como enviado"; ML normal usa padrão da conta.
- Multiempresa existe como recurso (indicador "Disponível multiempresa", "Enviar produto para empresas") — uso a confirmar.

---

## Vendas e pedidos

### Pedidos de venda — `vendas#list`
Evidências: `ev_c60aeae14496f29e518732ee478d0a8b8ec860fcbabc396ca899fc6295fe502e`, `ev_d436232e8f44b79c1ed3df1330b6e17bcca635f5ba5dfe8ba1001d38bc8603e8`
- Topo: **Imprimir**, **Receber do e-commerce** (não clicado), **Incluir pedido**, **Mais ações**.
- **Mais ações**: Receber pedidos do e-commerce · Importar pedido do e-commerce (por nº) · Atualizar situação de envio dos pedidos · Consultar pedidos multiempresa · Imprimir relatório · Exportar pedidos para planilha · Importar pedidos de uma planilha.
- Busca "cliente ou número"; período (padrão **Últimos 30 dias**); **Filtros**: Vendedor · **Marcador** · E-commerce (integração / "Sem vínculo com e-commerce") · Forma de recebimento · Forma de envio · Produto (nome/SKU) · Estados (multi) · Cidade.
- Abas de situação (com contador): **Todos · Em aberto · Aprovado · Preparando envio · Faturado · Pronto para envio · mais ▾** (as demais: Enviado, Entregue, Não entregue, Cancelado, Dados incompletos, Em devolução — conforme combo de situações).
- Colunas: checkbox · ações · Nº · Data · Previsto · **Data limite de despacho** · Cliente · CNPJ/CPF · Total · Nº pedido (do canal) · Rastreamento · **Marcadores** · Integrações. Rodapé: Quantidade e Valor total (R$) da seleção.
- **Marcadores existentes na conta**: 1ª venda · Devolvido · Etiqueta com data programada · Fulfillment · Importado sem informação de comissão de venda · Magalu Logística · Shopee coleta · Shopee Fulfillment · TikTok Shipping (+ "Sem marcadores"). Marcadores são aplicados automaticamente pelas integrações.
- **Formas de envio da conta**: Correios, Transportadora, Retirar pessoalmente, Mercado Envios, Shopee Envios, Amazon DBA, Magalu Entregas, Magalu Fulfillment, TikTok Shipping, Olist Envios.
- **Formas de recebimento**: Dinheiro, Boleto, Cheque, Pix, Link de pagamento, Mercado Pago, Shopee, Múltiplas.

### Novo pedido de venda — `vendas#add`
Evidência: `ev_3dbc0019fdcf1d7a276c9258c44a6aed13be0008ccba10a537698d9e7433e8e1`
- Banner "Antecipação Imediata" (crédito Olist).
- Cabeçalho: **Natureza da operação** (padrão "Venda de mercadorias"), **Número** (sequencial automático), **Cliente** (busca por nome/CPF-CNPJ/e-mail; botões "Dados do cliente", "Ver últimas vendas", "Limite de crédito"), **Vendedor**, **Lista de preço** (só "Padrão"), [ ] endereço de entrega diferente do de cobrança.
- Abas de itens: **Itens de produtos ou serviços · Comissões · Impostos**. Grade: Nº · Descrição (busca por descrição/SKU/GTIN) · SKU · Qtde · UN · Preço un · Preço total · Ações; "Adicionar outro item", "busca avançada de itens".
- **Totais**: Nº de itens, Soma das qtdes, Peso bruto/líquido, Total produtos, Valor IPI, Valor ICMS ST + FCP ST, Valor ICMS + FCP, Valor PIS, Valor COFINS, **Difal Não Contribuinte**, Total da venda.
- **Detalhes da venda**: Data da venda, Data prevista de entrega, Data de envio (data+hora), **Data máxima de despacho**, Nº pedido (do e-commerce/OC), Identificador do pedido e-commerce, Nº do pedido no canal de venda, **Intermediador** (botão "Sem intermediador"), Desconto (valor ou %), **Frete pago pelo cliente** (vai para NF), **Frete pago pela empresa** (informativo/custo), Despesas.
- Abas **Pagamento · Pagamento integrado**: Forma de recebimento; acesso rápido "Link de pagamento Olist", "Boleto Olist", "PIX Olist" (parcelas).
- **Transportador / Volumes**: Forma de envio; **Enviar para expedição** (Sim/Não).
- **Dados adicionais**: **Depósito** (Padrão, Amazon Full, Devoluções, Galpão, ML Full, Shopee Full), Observações (vão para a NF), Observações internas, Anexos (≤2 MB), **Marcadores** (tags livres).
- Rodapé: Salvar / Cancelar (não clicados).

### Pedidos no e-commerce — `lista_pedidos_ecommerce`
Evidência: `ev_9ae853aef5df9fef78ec4dd7608e3e6a62ab5729f81b6b8db7042fe7fa7b3530`
- Fila de pedidos crus vindos dos canais, antes/depois da importação. Filtros: "Ativos", período (ex. 06–16 out), filtros. Abas por integração: **Mercado Livre · Shopee · Mercado Livre Fulfillment · Magalu Marketplace · TikTok Shop**.
- Colunas: checkbox · Número · Cliente · Data Pedido · **Situação no e-commerce** · **Situação no Sistema ERP** · Valor · Nº Pedido (link para o pedido do ERP). Indicadores: Pedidos (qtde) e Valor.

### Separação — `separacao`
- Botões com atalhos: **Embalar fulfillment (ctrl+h)**, **Separar fulfillment (ctrl+g)**, **Embalar pedidos (ctrl+e)**, **Separar pedidos (ctrl+s)**.
- Busca destinatário/número; ordenação "mais antigas"; filtro por forma de envio.
- Abas: **Aguardando separação · Em separação · Separadas · Embaladas checkout**.

### Expedição — `expedicao`
- Aviso: "Os serviços dos correios ainda não foram configurados" (link Formas de Envio).
- Botões: **Integrar com a Intelipost**, Imprimir, **Incluir expedição**, Mais ações. Busca nº/pedido/NF/destinatário; filtros Agrupamentos e mês.
- Abas: Todas · Pendentes · Concluídas. Colunas: Agrupamento · Data emissão · Forma de Envio · Qtde pedidos · Peso Bruto (kg).

### Devoluções de venda — `devolucoes_vendas`
- Botões: **Gerenciar vales-troca**, **Incluir devolução**. Busca cliente/pedido/NF; filtro por período.
- Abas: **Todas · Em aberto · Em andamento · Finalizadas**. Colunas: Origem (links V=venda, N=nota, D=devolução) · Data devolução · Cliente · CPF/CNPJ · Total · Marcadores · Meio · Integrações. Indicadores Quantidade e Valor total.

### Custos do e-commerce — `custos_ecommerce#list`
- Link **Relatório** (`relatorio_custos_ecommerce`), botão **Ações**, filtro período (padrão "últimos 6 meses"). Lista vazia/sem dados no período observado.

### Margem de contribuição — `margem_contribuicao`
Evidência: `ev_afa48d79bcf2b498ab6bf1aaf4a01e0ad7564bb1a445e4260805744a9620ff3b`
- Botões **configurar cálculo**, Mais ações; período (padrão Últimos 7 dias), ordenação "Mais rentáveis". Visões: **Canais de venda · Produtos · Pedidos de venda · Visão geral** (+ "Glossário").
- DRE da margem: (+) Faturamento · Frete das vendas · (−) Custo adicional com frete · (−) Comissões · (−) Taxas e tarifas · (−) Custos de compras · (−) Impostos das vendas · (+) Incentivos · (+) Créditos de impostos · (−) Valores adicionais → **Valor total da margem** e **Índice total da margem (%)**; cards de detalhe por canal/produto/pedido.

### Comissões — `comissoes#list`
- Compartilhar, Imprimir, filtro por mês. Indicadores: Vendedores · Saldo anterior · Comissões do mês · Débitos de devoluções · Outros créditos · Outros débitos · Pagamentos · Comissões pendentes · Saldo. Sem vendedores cadastrados ("acesse Cadastros → Vendedor").

### Painel de automação — `automacoes` (via Configurações)
Evidência: `ev_afa48d79bcf2b498ab6bf1aaf4a01e0ad7564bb1a445e4260805744a9620ff3b`
Automações disponíveis (checkbox + gatilho "Quando"):
| Automação | Gatilhos possíveis | Estado na conta |
|---|---|---|
| Lançar saída de estoque | Ao autorizar NF / Ao salvar NF / Ao salvar o pedido / Ao marcar pedido como enviado | **Ligada — ao marcar pedido como enviado** |
| Emitir NF-e | Ao aprovar o pedido | Desligada |
| Autorizar NF-e na Sefaz | É emitida automaticamente a partir de uma venda / Ao separar pedidos / Ao concluir embalagem | Desligada |
| Imprimir etiqueta de envio | Ao concluir embalagem | Desligada |
| Enviar para separação | Ao autorizar NF / Ao salvar NF / Ao salvar o pedido / Ao aprovar o pedido | **Ligada — ao autorizar a NF** |
| Enviar para expedição | Ao autorizar NF / Ao salvar NF / Ao salvar o pedido / Ao aprovar o pedido | **Ligada — ao autorizar a NF** |
Botão "Salvar automações" (não clicado).

### Perguntas do e-commerce — `perguntas_ecommerce`
Evidência: `ev_5cb933c917b54eedda1a95635254ff9aa6687d3fca4466992f87df975836bb05`
- Filtro "não respondidas", seletor de integrações (3 selecionadas), Ações. Erro exibido: **"Não foi possível buscar as conversas da integração Shopee — não foi possível obter o token de acesso"**.

### Pós-venda do Mercado Livre — `ecommerce_pos_venda`
- Seletor de integrações (2 selecionadas: ML e ML Full), filtros, **Nova mensagem**. Lista de conversas por pedido (imagem do item, trecho da mensagem, nº pedido, hora), selo **"Este pedido possui uma reclamação!"**, "Carregar mais conversas", painel de mensagens; alerta "Há novas mensagens pendentes".

### Propostas comerciais — `orcamentos#list`
- Imprimir, **Incluir proposta**, Mais ações; abas Todas · Em aberto · Rascunhos · Pendentes · Aguardando · Aprovadas · mais. Fluxo: criar proposta → atualizar status da negociação → gerar pedido a partir da proposta. **Nenhuma proposta na conta.**

### CRM — `crm#list`
- **Incluir assunto**; busca contato/CPF-CNPJ/descrição; visões **Lista / por estágio**; abas Todos · Com ações pendentes · Encerrados · Com estrela · Arquivados. **Vazio.**

### Performance de vendas (metas) — `metas_venda#list`
- Filtros mês, por natureza de operação, Ativo. Colunas: Vendedor · Clientes atendidos · Clientes com vendas · Meta do mês · Vendas no mês · Faturamento no mês · Realizado (%). **Zerado (sem vendedores).**

**Sinais de uso (vendas)**
- Volume: aba "Todos" = **1.079 pedidos nos últimos 30 dias** (~36/dia); próximo nº de pedido ≈ 48,9 mil (histórico longo). Na fila do e-commerce, ~465 pedidos em 11 dias na aba ML.
- Pedidos chegam 100% via integrações (marcadores automáticos por canal); só 2 "Em aberto", 20 "Aprovado", 1 "Pronto para envio" no momento — fluxo rápido.
- **Devoluções: 822 registros (812 finalizadas, 10 em aberto)** — devolução é processo relevante (há depósito "Devoluções").
- **Não usa**: propostas comerciais, CRM, vendedores/comissões, metas, margem de contribuição (zerada), expedição (sem agrupamentos; Correios não configurados), separação (vazia no momento, mas automação "enviar para separação ao autorizar NF" está ligada).
- Pós-venda ML em uso (conversas e reclamações); integração Shopee com **erro de token** nas perguntas.
- Automação de NF-e (emitir/autorizar) **desligada** → emissão de NF provavelmente manual em lote ou feita pelos marketplaces (ver Fiscal).

---

## Fiscal

### Notas fiscais de saída — `notas_fiscais#list`
Evidência: `ev_4117b304387e8d94cd8d51cdea24117be4c7b9235517a0cac957cec05b514c00`
- Topo: **guias de pagamento**, **Imprimir DANFEs**, **Autorizar pendentes**, **Incluir nota fiscal**, **Mais ações**. Banner de crédito ("antecipar recebíveis").
- **Mais ações**: Enviar notas pendentes · Imprimir DANFEs autorizadas · gerenciar guias de pagamento · Receber pedidos do e-commerce · Importar pedido do e-commerce · **Importar XML da NFe** · Imprimir relatório · **Inutilizar numeração** · **Ativar o modo de contingência NFe** · Verificar NFes aguardando protocolo · Exportar notas fiscais · **Importar notas Magalu Fulfillment** · **Importar notas Amazon FBA Classic** · **Importar notas Shopee Fulfillment** · **Importar notas Mercado Livre** · Verificar notas geradas incorretamente.
- Busca por cliente (botão de campo "Cliente"); período (Últimos 30 dias), por data de emissão, filtros.
- Abas: **Todas · Pendentes · Emitidas · Canceladas** (+ contadores). Colunas: Nº · Data emissão · Cliente · UF · Valor · Marcadores · Integrações (links de origem V = venda, E = e-commerce).

### Nova NF-e — `notas_fiscais#add`
- Cabeçalho: **Tipo de Saída** (Emissão própria / Exportação), **Série** (4), **Número** (sequencial — próximo ≈ 000867), Data/Hora emissão, **Natureza da operação**, Data/Hora saída, **Finalidade** (NF-e normal, complementar, de ajuste, Devolução, Nota de débito, NF-e com cupom referenciado, NF-e com chave de acesso referenciada, NF-e normal – pagamento antecipado), **Código de regime tributário** (Simples nacional [sel.], Simples – excesso de sublimite, Regime normal, MEI), Consumidor final (Sim/Não), Intermediador, **Indicador de presença** (não se aplica / presencial / **não presencial – Internet** [sel.] / teleatendimento / presencial fora do estabelecimento / outros).
- **Destinatário**: Nome (busca), Tipo de pessoa (Jurídica/Física/Estrangeiro/Estrangeiro no Brasil), Contribuinte (Contribuinte ICMS / Isento / Não contribuinte), CNPJ (consulta), IE, CEP, Cidade, UF, Endereço, Bairro, Número, Complemento, Fone, E-mail, Suframa, Vendedor, Lista de preço, endereço de entrega diferente.
- Abas **Produtos ou serviços · Impostos**; grade Nº · Descrição · SKU · Un · Qtde · Preço un · Total.
- **Cálculo do imposto**: Total produtos, Total serviços, Frete, Seguro, Base/Valor ICMS, Base/Valor ICMS ST, IPI, IPI devolvido, ISSQN, Despesas, Desconto, **Funrural**, Nº itens, Valor aprox. impostos, FCP, FCP ST, FCP ST retido anteriormente, Total da nota; [x] **Cálculo automático de imposto**.
- Transportador/Volumes (forma de envio, pesos, enviar para expedição); Pagamento / Pagamento integrado; Dados adicionais: **Depósito**, Observações, Observações do sistema (geradas pelas Operações Fiscais), Informações ao fisco, Marcadores.

### Notas fiscais de entrada — `notas_entrada#list`
- Topo: **Buscador de notas** (manifesto/DF-e), **Importar XML da NFe**, Incluir nota fiscal, Mais ações.
- Abas: **Todas · Pendentes · Registradas · Emitidas · Canceladas**. Colunas: Nº · Data emissão · **Remetente** · UF · Valor · Marcadores · Integrações.

### Configurações fiscais — `preferencias_notas_fiscais`
Evidências: `ev_6994a2c607f159a0d62528964b72e67fc623ba96e4ea78833d3e9ec781f7f13b`, `ev_2c61ac6d90279efd8a1404ef840095f514335269a9f3dfa981c5559a227cfe41`
- Gerais: Dados da empresa · Configuração do certificado digital · **Ambiente das notas fiscais (Produção)** · Naturezas de operação de entrada · Naturezas de operação de saída.
- NF de venda: Configuração da NF-e (`parametros_nota_fiscal`) · ICMS DIFAL p/ não contribuinte · Cálculo diferenciado de ST (DIFAL) · **GNRE** · Cadastro de intermediadores · Marcadores nas NF de saída.
- NF de entrada: Configurações de NF de entrada · Marcadores nas NF de entrada. NFS-e: marcadores das notas de serviço.
- `parametros_nota_fiscal` — Automações: [ ] Gerar NF automaticamente ao aprovar pedido; [ ] Autorizar automaticamente NF geradas a partir da venda; [ ] **Cancelar automaticamente NF ao cancelar venda vinculada** — **todas desligadas**. + "Configurações adicionais".
- **Naturezas de operação de saída** (`operacoes_fiscais`; colunas Descrição · Padrão · Série · Consumidor final; ações: Configurações, Incluir): Devolução de compra para fornecedor (Lucro Presumido) · Lançamento a título de baixa de estoque por perda/roubo · **Outras Saída – Remessa para Depósito Temporário** (consumidor final Não) · Remessa em Bonificação, Doação ou Brinde · **Venda de mercadorias** (padrão para venda) · Venda de mercadorias PJ (série 1). → 6 naturezas, quase todas série 4.
- **Naturezas de entrada** (`operacoes_fiscais_entrada`): **Compra de mercadorias** (padrão, série 1) · **Devolução de Venda de Mercadoria** (série 4).
- **Tributação (RTC)** — `preferencias_tributacao`: guia da Reforma Tributária do Consumo; NF-e/NFC-e: Configuração dos tributos e códigos de classificação, **Habilitar cálculo de tributos da Reforma Tributária (desabilitado)**, Cadastro de regras tributárias; NFS-e: mesmos itens (desabilitado).
- **Intermediadores** (`intermediadores`; colunas Nome · CNPJ · Canal de venda): **7 cadastrados** — Amazon, Mercado Livre, Mercado Mestre, Shopee, TikTok Shop (Bytedance) + 2 registros nomeados só com código numérico (duplicatas de Shopee e ML por CNPJ).

**Sinais de uso (fiscal)**
- Saída: **1.018 NF nos últimos 30 dias (1.002 emitidas, 16 canceladas)**; entrada: **1.021 (1.006 emitidas, 1 registrada, 14 canceladas)**.
- Na 1ª página, **100% das NF (saída e entrada) têm marcador "Fulfillment"** e integração ML Fulfillment, numeração na faixa ~177 mil e remetente/destinatário Mercado Livre/Ebazar → são as **notas do fluxo ML Full emitidas pelo próprio Mercado Livre e importadas automaticamente** (venda a partir do CD do ML, remessa/retorno de armazenagem), não emissão própria.
- **Emissão própria pelo ERP é pequena**: série 4 com próximo número ≈ 867 (acumulado histórico). Automações de NF desligadas; certificado/ambiente de produção configurados.
- Regime **Simples Nacional**; venda não presencial pela internet; natureza "Remessa para Depósito Temporário" para enviar ao Full (modelo *self storage* CAT 59/2018).
- RTC (reforma tributária) ainda não habilitada.

---

## Compras / Suprimentos

Evidências: `ev_57698f32fd5515127336cb0508abd0deb9b0278d023b45f81ef136306220d231`, `ev_5254442be90b9bb78e13a60c7bb1dc5f2a287e9c39516ad985ffd2eb37fef33b`

### Ordens de compra — `pedidos_compra#list`
- Imprimir, **Incluir ordem de compra**, Mais ações; busca fornecedor/nº pedido/OC; filtro período. Abas: **Todos · Em aberto · Em andamento · Atendidas · Canceladas**. **Nenhuma ordem de compra cadastrada.**

### Nova ordem de compra — `pedidos_compra#add`
- Fornecedor (busca; "Dados do fornecedor", "Ver últimas compras", "Pessoas de contato"), Número (controle, sequencial = 1).
- Itens: Item · Cód (SKU) · GTIN/EAN · Qtde · UN · Preço un · **IPI %** · Preço total · Ações; "Adicionar item", "busca avançada".
- Totais: Nº itens, Soma qtdes, Total produtos, Desconto (valor/%), Frete, Total IPI, Total ICMS ST, Total geral.
- Detalhes: Nº no fornecedor, Data da compra, Data prevista. **Pagamento**: Condição de pagamento (ex. "30 60", "3x", "15 +2x") + "Gerar parcelas"; Categoria (financeira).
- Transportador: Nome, **Frete por conta** (CIF / FOB / Terceiros / Próprio remetente / Próprio destinatário / Sem ocorrência). Observações, Marcadores, Observações internas.

### Conferência de compra (entrada de mercadorias) — `entrada_de_mercadorias`
- Botões com atalho: **Importar XML (ctrl+x)**, **Conferir itens (ctrl+i)**, **Receber mercadorias (ctrl+e)**. Busca fornecedor/número; período; ordenação.
- Abas: **Aguardando entrada · Pronto para conferir · Em conferência · Conferidos · Com problema**. Vazio no período (17/set–16/out).

### Necessidades de compra — `relatorio_necessidade_compra`
- Parâmetros: Calcular cobertura de estoque (Sim/Não); Baseado em (Último mês / 2 meses / 3 meses / Personalizado); Calcular necessidade para N meses; Exibir produtos (Todos / Apenas com necessidade / Para atender estoque mínimo); Exibir estoque mín/máx; **Situações de venda para saídas previstas** (padrão Aprovado, Faturado); Tags; Fornecedores; Produto; Informações adicionais; Natureza de operação; Considerar saídas previstas dos últimos N meses; **Depósitos** (Todos 5); Marca; Tipo de produto (Simples / Fabricado / Matéria-prima); [ ] Considerar lançamentos de transferência; [ ] Considerar saídas previstas → Gerar.

### Giro de estoque — `giro_estoque`
- Período De/Até (padrão 6 meses), Categoria, Tipo (Simples / Com variações / Fabricado / Matéria-prima), Tags, [ ] considerar produtos com saldo final zerado → Gerar (abre em pop-up).

### Serviços tomados — `servicos_tomados` (beta)
- "incluir nota" (NFS-e de serviços tomados); filtros por período, situação, data de emissão. **Nenhum registro.**

### FCI — `fci`
- Fichas de Conteúdo de Importação: Configurações, **Incluir FCI**, filtro por mês. **Nenhuma.**

**Sinais de uso (compras)**
- **Módulo de compras não é usado**: zero ordens de compra, zero conferências, zero serviços tomados, zero FCI. Entradas de estoque não passam por OC (provavelmente lançamento manual/XML de entrada ou só transferências do Full).
- Há relatórios de necessidade/giro disponíveis, mas sem fornecedores vinculados aos produtos observados.

### Configurações de estoque — `parametros_estoque` / `depositos`
Evidência: `ev_aa95e32914d5b3b74dd4857e646d0d12c5da92bf9813028f71382af0966ff37c`
- **Depósitos** (`depositos`; Configurações, Incluir depósito, filtro Ativos): Amazon Full · Devoluções · **Galpão (Depósito Padrão)** · ML Full · Shopee Full → **5 depósitos ativos**.
- Permite estoque negativo: **Sim** (outras opções: validar saldo de todos os depósitos / do depósito da venda).
- Lançamento p/ saídas: Manual / Ao autorizar NF / Ao salvar NF / Ao salvar pedido / **Ao marcar pedido como enviado** [sel.]; aviso de **reserva de estoque habilitada**.
- Lançamento p/ entradas: **Manual** [sel.] / Ao autorizar NF / Ao salvar NF / Ao salvar OC / Ao finalizar conferência de compra.
- Lançamento p/ serviços: **Manual** / ao salvar OS / ao finalizar OS. Estorno de estoques: **Manual** / automático ao cancelar NF.
- Checkboxes (todos desmarcados salvo indicado): contratar extensão de limite de sincronização de estoque; exibir estoque na lista de preços; estornar estoque ao cancelar pedido; e-mail de produtos abaixo do mínimo (+ e-mail destino); Qtde enviada ao e-commerce p/ sob encomenda = 99; considerar saldo no sob encomenda; [x] **Confirmar lotes e validades ao lançar estoque individualmente**; permitir editar qtde lida no inventário.

---

## Financeiro

Evidências: `ev_5254442be90b9bb78e13a60c7bb1dc5f2a287e9c39516ad985ffd2eb37fef33b` (caixa), `ev_6e2a1229c9ca148f97433ac7c8a7ba893978591ceff961f09722728f2f6d1daa` (caixa/pagar/receber), `ev_6771413af9f4d6a527287e9684ec7216ee29332ddf03875962a6aaaeb2c5ad8d` (cobranças/extratos/preferências), `ev_aa95e32914d5b3b74dd4857e646d0d12c5da92bf9813028f71382af0966ff37c` (categorias/formas/contas)

### Caixa e bancos — `caixa`
- Imprimir, **Transferir**, **Incluir lançamento**, Mais ações (Transferir entre contas · Imprimir relatório · Exportar lançamentos para planilha · Importar lançamentos de planilha · **Importar extrato OFX** · **Gerenciar fechamento financeiro** · Movimentações financeiras bloqueadas).
- Busca por cliente; período (Últimos 30 dias); filtros. Seletor de conta (aba "Caixa"); indicador **Saldo atual**. Banner de antecipação de recebíveis.

### Contas a pagar — `contas_pagar`
- **Incluir conta a pagar**, Ações, Imprimir, **Gerenciar pagamentos**, Mais ações (Gerenciar pagamentos · Imprimir relatório · Imprimir agrupado por fornecedor · Exportar/Importar lançamentos por planilha).
- Busca fornecedor/nº doc. Abas: **Todas · Em aberto · Emitidas · Pagas · Atrasadas · Canceladas**. Colunas: Fornecedor · Histórico · Nº documento · Vencimento · **Competência** · Valor · Saldo · Pago · Marcadores · Forma · Integrações. Rodapé quantidade/valor.

### Contas a receber — `contas_receber`
- **Gerenciar recebimentos**, **Incluir conta a receber**, Mais ações (Gerenciar recebimentos · Imprimir · Imprimir agrupado por cliente · Exportar/Importar planilha).
- Busca cliente / nº no banco / nº doc; filtro "por meio de recebimento". Abas: **Em aberto · Emitidas · Previstas · Recebidas · Atrasadas**.

### Cobranças bancárias — `cobrancas_registradas#remessas`
- **Incluir remessa**; filtros por banco e período; abas **Remessas · Retornos bancários** (CNAB). Vazio.

### Extratos bancários (conciliação) — `extratos_bancarios`
- **Importar extrato bancário**; lista de extratos. Vazio.

### Configurações financeiras — `preferencias_financeiro`
- Geral: Configurações gerais · **Categorias de receita e despesa**. Contas e caixa: **Cadastro de contas bancárias** · Cadastro de contas financeiras. Recebimentos e pagamentos: **Formas de recebimento** · **Formas de pagamento** · Cadastro de gateways · Maquininhas de cartão. Avisos e e-mails: config. contas a pagar / a receber / envio de documentos. Marcadores financeiros.
- **Categorias** (`categorias`; colunas Descrição · Categoria no DRE · Competência padrão; Grupos, Incluir, Mais ações), agrupadas: *Insumo de Escritório* → Etiquetas (Despesas operacionais, mês da emissão, padrão compra); *VENDAS* → **Vendas Mercado Livre**; *Sem grupo* → Água/luz, Aluguéis e condomínio, Compras, Impostos/taxas, Segurança, Serviços gerais, Telecomunicação/internet, Veículos/transportes (≈ 10 categorias, a maioria padrão do sistema).
- **Formas de recebimento** (colunas Forma · Banco/Gateway preferencial · Status; Preferências): Dinheiro, Cartão de crédito, Cartão de débito, Boleto, Cheque, Depósito, Crediário, Vale-troca, Pix, Link de pagamento, **Mercado Pago**, **Shopee** (12).
- **Formas de pagamento** (18, padrão SEFAZ): Boleto, Cheque, Crediário, Cartão crédito/débito, Depósito, Depósito bancário, Dinheiro, Duplicata mercantil, Fidelidade/Cashback/Crédito virtual, Pix, Vale-troca, Vale alimentação/combustível/presente/refeição, Transferência/Carteira digital, Outros.
- **Contas bancárias** (colunas Descrição · Banco · Agência · Conta): **2 contas — "Olist Conta Digital" e "Mercado Pago"** (números omitidos).

**Sinais de uso (financeiro)**
- **Financeiro praticamente não usado**: caixa sem movimentos nos últimos 30 dias (saldo histórico negativo, sinal de lançamentos antigos sem conciliação), **contas a receber vazio no filtro padrão** (pedidos de marketplace não geram títulos/repasses), **7 contas a pagar em aberto, todas atrasadas**, sem remessas de cobrança, sem extratos importados.
- Categorias quase todas padrão; só "Vendas Mercado Livre" e "Etiquetas" criadas pelo usuário.
- Contas: Olist Conta Digital e Mercado Pago (onde caem os repasses do ML).

---

## Logística

Evidências: `ev_9ae853aef5df9fef78ec4dd7608e3e6a62ab5729f81b6b8db7042fe7fa7b3530` (separação/expedição), `ev_d1b727f5a7fa9f7a8508e06d8f2423bb7660570ef2675bf3f6a5cd67cf4b2e7e` (Envios/PDV/loja)

- **Separação** e **Expedição**: ver seção Vendas (abas, atalhos de teclado, agrupamentos). Configurações em `preferencias_vendas` → Expedição e Logística: Formas de envio (`formas_envio`), Gateways logísticos, Configurações da expedição, Configurações da separação, Marcadores na separação, Configurações da Intelipost.
- **Olist Envios** — `olist_envios` (app lateral): status **"Envios ativado"**; Créditos disponíveis (carteira pré-paga, valor oculto) + "adicionar créditos"; "ir para plataforma Envios"; Guia (integrar e-commerce, adicionar créditos, forma de cotação/contratação de etiquetas, várias etiquetas por pedido/multivolume, cotar frete manual, etiqueta PDF/ZPL, regras de frete grátis, comprar etiqueta direto da Separação/Expedição, acompanhar coleta); **Calculadora de frete** (CEP remetente/destinatário, valor declarado, A×L×C, peso → simular); **Endereço de coleta** (link "atualizar dados de coleta").
- Formas de envio presentes na conta: Correios, Transportadora, Retirar pessoalmente, Mercado Envios, Shopee Envios, Amazon DBA, Magalu Entregas, Magalu Fulfillment, TikTok Shipping, Olist Envios.

**Sinais de uso (logística)**
- Logística é essencialmente do marketplace: **Mercado Envios (etiqueta do ML)** + **Fulfillment** (ML/Amazon/Shopee Full). Marcadores "Etiqueta com data programada", "Shopee coleta", "TikTok Shipping", "Magalu Logística" indicam coletas/etiquetas dos próprios canais.
- Olist Envios está ativado (há crédito/carteira), mas Correios não configurados na expedição e não há agrupamentos de expedição no mês → uso marginal ou nulo.
- Separação vazia no momento, embora a automação "Enviar para separação ao autorizar NF" esteja ligada.

---

## Relatórios / BI

- Listas de relatórios por módulo já mapeadas anteriormente (não repetido). Vistos nesta rodada: **Inventário de estoque** (`relatorio_estoque_inventario`), **Necessidades de compra** (`relatorio_necessidade_compra`), **Giro de estoque** (`giro_estoque`, abre em pop-up), **Relatório de custos do e-commerce** (`relatorio_custos_ecommerce`), **Margem de contribuição** (dashboard), **Comissões**, **Performance de vendas**.
- Relatórios geralmente abrem em nova janela/pop-up (bloqueado pelo navegador na automação — "Pop-up bloqueado pelo navegador").

---

## Outros / Configurações

Evidências: `ev_6994a2c607f159a0d62528964b72e67fc623ba96e4ea78833d3e9ec781f7f13b` (índice), `ev_6771413af9f4d6a527287e9684ec7216ee29332ddf03875962a6aaaeb2c5ad8d` (abas Vendas/Cadastros/Suprimentos/Finanças), `ev_31c2255d3a4e2014ecdcaeeb8f94ff64431595fce091df2082e663d5f93655f3` (usuários/multiempresa/webhooks/API/apps/e-commerce), `ev_0eced29186435ded60f57900c96326424fa2b550d1dee9df3fc540212780b681` + `ev_8967cb583f11bd723987adfbfe0d7f18157dba672467fac5a5e9ea91975ddbac` (perfis/permissões), `ev_d1b727f5a7fa9f7a8508e06d8f2423bb7660570ef2675bf3f6a5cd67cf4b2e7e` (loja de extensões)

### Índice de configurações — `preferencias_geral`
- Abas: **Geral · Cadastros · Suprimentos · Vendas · Notas Fiscais · Finanças · E-commerce · Tributação (RTC, novo)** + busca "funcionalidade ou dúvida".
- **Geral**: Alterar dados da empresa · Alterar dados do usuário · **Cadastro de usuários do sistema** · Servidor de e-mail · Envio de documentos · Etiquetas · Agenda · *Outras*: Interface do usuário · Central de notificações · **Impressão PrintNode** · **Multiempresa** · **Aplicativos** (API) · **Token API** · **Configurações de API** · **Webhooks**.
- **Cadastros**: clientes · produtos · variações de produtos · atributos de produtos · marcas · tabelas de medidas · tags · tipos de contato · linhas de produto · listas de preços.
- **Suprimentos**: depósitos · configurações de estoque · envio de documentos · marcadores de OC · marcadores de serviços tomados · ordens de compra · conferência de compra.
- **Vendas**: PDV · propostas · pedidos de venda · envio de documentos · marcadores (vendas, propostas, devoluções) · devoluções de vendas · *Expedição e Logística* (formas de envio, gateways logísticos, expedição, separação, marcadores na separação, Intelipost) · *CRM* (configurações, marcadores, estágios do funil). (O Painel de Automação também é acessado via Configurações.)
- **Finanças** e **Notas Fiscais**: ver seções Financeiro e Fiscal.
- **E-commerce**: Configurações gerais (`parametros_ecommerce_geral`) · Integrações · Token API. Configurações gerais: [ ] Número do pedido nas observações; **Substituir kits pelos componentes nas integrações/importação** (Sim p/ kits e fabricados / Sim só kits / **Não**); **Buscar produtos por** (Código ou descrição / **Apenas código**); Preenchimento do "Nº do pedido no e-commerce/OC" (**Identificador do pedido no e-commerce** / Nº do pedido no canal).

### Usuários do sistema — `usuarios_sistema`
- Botões: gerenciar vendedores · **perfil de usuários** · **incluir usuário** · Mais ações; busca nome/login. Card por usuário (nome, e-mail, perfil, "gerenciar").
- **Novo usuário** (`#/new`), abas **acesso do usuário · permissões e ações**: Nome; Login de acesso (sufixo @conta) ou e-mail; E-mail para comunicação com clientes; **Restrições de acesso** por horário/dias e por IP. Botão "salvar e avançar" (não clicado).
- **Perfis de usuário** (`perfil_usuario`): "criar perfil"; perfil = nome + descrição + permissões; usuário pode ter vários perfis.
  - Permissões de **menu/telas** por módulo (abas por módulo + "mais"), ex. Cadastros: Vendedores, Embalagens, Produtos, Preço promocional, Relatório de sincronização com e-commerce, Relatório de preços, Categorias, editar preço na lista de preços, Clientes e Fornecedores, **editar limite de crédito**, **exportar dados de contatos**, Campanhas promocionais; Outros: Variações, Listas de preço, Precificação inteligente, Preços competitivos ML/Amazon.
  - **Permissões de ações** (tabela Módulo × Pode incluir / Pode editar / Pode excluir): Lista de atualizações de produtos e estoque, Clientes e Fornecedores, Produtos, Vendedores, Controle de Estoques, Serviços Tomados, Ordens de Compra, Notas de Entrada, CRM, Pedidos de Venda, Notas Fiscais, Margem de Contribuição, Relatórios Personalizados.
  - **Outras ações**: alterar situação da conferência de compra manualmente; conferir volumes/itens/problemas na conferência; alterar situação da separação manualmente; **aprovar vendas com desconto de gerente**.

### Multiempresa — `multi_empresas`
- Página de apresentação com "Adicionar empresa" (centralizar CNPJs, estoques isolados por empresa, certificado por empresa, alternância no menu lateral). **Nenhuma empresa adicional configurada.**

### Webhooks — `configuracoes_webhooks_empresa`
- [ ] notificações de vendas · [ ] pedidos enviados · [ ] lançamentos de estoque · [ ] NF autorizadas — **todos desligados**.

### Configurações de API — `configuracoes_api` / Aplicativos — `aplicativos_api`
- Estoque API: [ ] forçar novo registro de balanço quando preço/qtde iguais; [x] sincronizar estoque dos anúncios ao gerar balanço via API.
- Aplicativos API: "+ novo aplicativo" (permissões + chaves). **Nenhum aplicativo cadastrado.**

### Loja de extensões — `loja_aplicativos`
- Filtros: E-commerce · Comércio · Serviço · Indústria · Conta · IA; "Minhas extensões". Plano atual: **"Domine"**.
- Extensões: Maquininhas de cartão, Correios LOG+, pacotes de **sincronizações de estoque** (+10 mil a +10 milhões) e de **anúncios** (+500 a +100 mil), Boa Vista SCPC, Cálculo automático de volumes, Cashback, Vale presente, **créditos de Agentes de IA (Lis)**, banco de dados e anexos extras; upgrade: limite adicional de chamadas de API, **Logs de Estoque**.

### Apps laterais
- **PDV** (`pdv`): tela inicial com relógio, **Abrir caixa (ctrl+enter)**, "ver detalhes do caixa" — sem sinal de uso (caixa fechado).
- **Ecommerce** (`olist_ecommerce`): página carregou apenas imagens (sem conteúdo acessível) — loja virtual própria não ativa/visível.
- **Envios**: ver Logística. Outros atalhos no topo: **Lis** (agente de IA), **Crédito** (antecipação), Notificações, Ajuda.

**Sinais de uso (configurações)**
- **5 usuários**: 1 admin (dono), 3 com perfil padrão "gerente" — **2 deles são da contabilidade externa** —, 1 com **perfil customizado "Logística"** (92 permissões). → a contabilidade acessa o ERP diretamente; há um operador logístico.
- Sem multiempresa, sem webhooks, sem apps de API; plano "Domine".

---

## Telas que não abriram / limitações
- `contatos` — botão **Mais ações** não abriu via automação (timeout); lista só carrega após ~6 s.
- `vendas#list` — link **"mais ▾"** das abas de situação não abriu (timeout); situações restantes inferidas do combo de mapeamento.
- `estoque` (produto) — "Incluir reserva" não abriu (timeout); o botão "Ações" clicado acabou abrindo o menu de Configurações/tema.
- `usuarios_sistema#/new` — aba **permissões e ações** só é acessível após "salvar e avançar" (não feito); granularidade vista pelo perfil existente.
- `giro_estoque` e relatórios — abrem em **pop-up** (bloqueado pelo navegador).
- `olist_ecommerce` — conteúdo não renderizado na árvore de acessibilidade.
- `custos_ecommerce` — lista vazia nos últimos 6 meses (pode ser não-uso ou carregamento lento).
- Menu "Mais ações" de `contatos`, `expedicao`, `orcamentos`, `margem_contribuicao`, `custos_ecommerce` (Ações) e da integração não foram abertos.

## Perguntas para o dono
1. **Contas ML**: a conta mostra **1 integração Mercado Livre** (+ ML Fulfillment vinculado a ela). Existe uma 2ª conta ML vendendo fora do Olist?
2. **NF-e**: as ~1.000 NF/mês visíveis são as do **ML Full emitidas pelo próprio Mercado Livre** e importadas. Os pedidos ML não-Full (Mercado Envios "etiqueta com data programada"), Shopee, Magalu e TikTok — quem emite a NF? O emissor do ML/marketplace ou o ERP (série 4, numeração baixa ~867)?
3. **Shopee**: a integração está com **erro de token** nas Perguntas e vários produtos com "Mapeamento com erro". A Shopee ainda vende? Precisa ser religada?
4. **Devoluções**: 822 devoluções registradas (812 finalizadas). Qual o fluxo hoje (retorno ao depósito "Devoluções", reanálise, descarte)? Precisamos de módulo próprio de devolução/triagem?
5. **Financeiro**: contas a receber vazio, caixa sem movimentos e 7 contas a pagar atrasadas — o financeiro é feito fora (planilha, Mercado Pago, contador)? Querem conciliação dos repasses do ML/Mercado Pago no novo ERP?
6. **Compras**: zero ordens de compra/conferência. Como entra mercadoria hoje (NF de fornecedor importada? lançamento manual?) e quem são os fornecedores (importados — origem ICMS 2)?
7. **Contabilidade**: 2 usuários da contabilidade com perfil gerente — o que eles extraem do ERP (XML, SPED, relatórios)? Isso vira requisito de exportação/acesso do contador.
8. **Estoque**: estoque negativo permitido e lançamento de saída "ao marcar como enviado". A reconciliação com os saldos dos Full (ML/Amazon/Shopee) é feita como? Transferências Galpão→Full são registradas por NF de remessa?
9. **Amazon / Magalu / TikTok**: estão ativos? (Na fila do e-commerce aparecem abas Magalu e TikTok; Amazon não aparece na fila de pedidos.)
10. **Kits**: kits são multipacks do mesmo item (ex. 5× lâmina). Manter o modelo de kit com estoque derivado do componente?
11. **Olist Envios**: está ativado com créditos — ainda usam para algum envio próprio?
12. **Usuário Logística**: quais tarefas ele faz no ERP hoje (separação, etiquetas, estoque)? Define os perfis/RBAC iniciais.

