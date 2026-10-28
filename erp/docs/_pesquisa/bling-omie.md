# Mapa de funcionalidades: Bling e Omie (referência para o ERP Pinese)

> **Fase:** pesquisa, sem código. **Data da coleta:** 16/10/2026.
> **Contexto do uso:** ERP próprio para substituir o Olist ERP (ex-Tiny) numa operação focada em Mercado Livre: 2 contas, Full + coleta/agência, nicho casa/jardim/decoração/ferramentas, equipe de 2 pessoas.

## Como este documento foi feito (metodologia)

- **Bling:** baixei a **central de ajuda inteira** pela API pública do Zendesk (`ajuda.bling.com.br/api/v2/help_center/pt-br/articles.json`). São **3.030 artigos em 348 seções**, convertidos para texto e indexados. Li na íntegra os artigos citados nas fontes e fiz buscas por palavra-chave no texto completo de todos eles. Também baixei e resumi o **OpenAPI da API v3** (`developer.bling.com.br/build/assets/openapi-*.json`) e li as páginas de limites e webhooks do portal developer.
- **Omie:** rastreei a **central de ajuda inteira** (Intercom): **350 coleções e ~2.900 artigos**, todos convertidos para texto e indexados. Li na íntegra os artigos citados e fiz buscas por palavra-chave no corpus completo. Também li a lista de serviços da API (`developer.omie.com.br/service-list`), a página de preços e as páginas da Omie.Store (Omie.Hub, Mercado Livre, TrackCash).
- **Limitação:** as páginas `ajuda.bling.com.br/hc/...` devolvem 403 para leitura direta, por isso usei a API. O Reclame Aqui e o Capterra também devolvem 403: as dores relatadas pelos usuários vêm de **trechos de busca**, não de leitura integral.
- **Fato x inferência:**
  - O que está nas tabelas é **fato documentado** pelo fabricante, salvo marcação.
  - **(inf.)** marca inferência minha.
  - **(beta)** marca recurso que o fabricante diz estar em teste ou liberação gradual.
  - **★** marca um **candidato a diferencial em relação ao Tiny/Olist**. É sempre inferência, porque o Tiny não foi pesquisado neste documento. Vale confirmar no levantamento do Tiny.
- **Complexidade estimada para reconstruir:**
  - **Baixa:** CRUD, telas e regras simples.
  - **Média:** regras de negócio não triviais, integração simples ou cálculos.
  - **Alta:** integração externa complexa, fiscal/SEFAZ, conciliação, motores de regra ou alto volume. O motivo vem entre parênteses.
- **Dependência de marketplace:**
  - **Nenhuma:** funciona sem marketplace.
  - **Parcial:** funciona sem, mas ganha muito com a integração.
  - **Total:** só existe por causa do marketplace.

---

## 1. Cadastros

| Feature | O que faz (concreto) | Sistema | Complexidade | Dep. marketplace |
|---|---|---|---|---|
| Cadastro de produtos simples | Nome, SKU, GTIN/EAN, GTIN tributário, NCM, CEST, origem, peso líquido/bruto, dimensões, unidade, imagens, descrição curta/longa, marca, situação. | ambos | Baixa | Nenhuma |
| SKU automático | Gera o código SKU automaticamente no cadastro. | Bling | Baixa | Nenhuma |
| Código DUN (embalagem) | Código de barras da caixa de embarque. Pode ser bipado no check-in de recebimento. | Bling | Baixa | Nenhuma |
| Produtos com variação (grade) | Produto-pai + variações (cor/tamanho), com imagens por variação, cadastro por planilha, exclusão/restauração de variação e transferência de vínculos entre variações. Variação pode ser "composta" (variação que é kit). | ambos (Bling mais detalhado) | Média | Nenhuma |
| Produto com composição (kit) | Kit formado por componentes. Baixa estoque dos componentes. O custo e o preço do kit podem ser recalculados pelos componentes, e o peso do kit é calculado pelos componentes. Opção de **desmembrar o kit na NF** e no checkout. | Bling (Omie tem "Produtos - Kit" e "Estrutura" na API e relatório "Kit e Componentes") | Média | Nenhuma |
| Rateio de custo do kit na entrada | Na NF de entrada de um kit, distribui o custo entre os componentes proporcionalmente ao preço de venda na estrutura. | Bling | Média | Nenhuma |
| Estoque mínimo/máximo por produto | Limites usados em alertas e relatórios de compra. | ambos | Baixa | Nenhuma |
| Tempo de crossdocking por produto | Prazo de preparação enviado ao canal. | Bling | Baixa | Parcial |
| Artigo perigoso (ANAC) | Flag no produto para restrição de transporte. | Bling | Baixa | Parcial |
| Etiqueta de produto | Impressão de etiquetas de código de barras do produto. Também há etiquetas customizadas. | Bling | Baixa | Nenhuma |
| Remoção de fundo de imagem | Ferramenta no cadastro para remover o fundo da foto. | Bling | Média (depende de serviço de IA externo) | Nenhuma |
| Grupos, tags e categorias de produto | Agrupadores. As categorias podem ser vinculadas à categoria da loja virtual. | ambos (Omie: Famílias de Produto, Características) | Baixa | Parcial |
| Linhas de produto / comissão por produto | Comissão diferente por linha de produto. | Bling | Baixa | Nenhuma |
| Listas de preço | Várias tabelas de preço, clonagem, edição por planilha, vínculo em massa e uso na venda. | ambos (Omie: Tabela de Preços) | Baixa | Nenhuma |
| Preço diferente por loja/canal | Preço por integração ("carrinho multiloja"): preço e preço promocional por canal, cadastráveis por planilha. | Bling | Média | Parcial |
| Formação de preço | Atualiza o preço de custo e o preço de venda por fórmula (custo + markup etc.). | Bling | Média | Nenhuma |
| Margem na publicação | Margem aplicada sobre custo, sobre preço de venda ou sem margem, ao publicar o anúncio. O preço é atualizado quando muda no ERP. | Omie.Hub | Média | Total |
| Validação do cadastro de produto | Checa campos obrigatórios exigidos pela integração antes de publicar. | Bling | Média | Parcial |
| Campos customizados | Tipos texto, texto longo, inteiro, decimal, data, lista e lógico, em produtos, contatos e OS. Com agrupador, vínculo a categoria, preenchimento por planilha, permissão por usuário e mapeamento para atributos do marketplace. | Bling | Média | Parcial |
| Contatos (clientes, fornecedores, transportadoras) | Cadastro unificado com tipos. Campos obrigatórios configuráveis, importação de endereço da SEFAZ, inativação, recuperação de excluído, segmentos com desconto/acréscimo fixo e LGPD. | ambos | Baixa | Nenhuma |
| Limite de crédito do cliente | Bloqueio/alerta de crédito. | Omie | Baixa | Nenhuma |
| Vendedores e comissões | Cadastro de vendedor, % de comissão, vínculo contato x vendedor, comissão paga via contas a pagar, relatório de comissão com estratégias de cálculo e restrições de permissão por vendedor. | ambos | Média | Nenhuma |
| Funcionários (folha simples) | Cadastro de funcionário, pagamentos e histórico, com acesso do contador. | Bling (Omie tem folha completa no OneFlow/G-Click, fora do escopo) | Média | Nenhuma |
| Embalagens | Cadastro de caixas. Vínculo automático da embalagem na venda e **cálculo automático da melhor embalagem** para reduzir o frete (Omie: com Loggi, Mandaê e Melhor Envio). | ambos | Média (bin-packing simples) | Nenhuma |
| Departamentos / centros de custo e projetos | Rateio de receitas e despesas (pedido, OS, contrato, conta a pagar/receber) por departamento. Projetos como 2º eixo. Relatórios por departamento/projeto. | **Omie** (no Bling não encontrei: 0 ocorrências de "centro de custo" na ajuda) | Média | Nenhuma |
| Importação/edição em massa por planilha | Produtos, variações, kits, contatos, preços, imagens, lotes, categorias, contas e pedidos de compra/venda. | ambos | Média | Nenhuma |
| Alteração e exclusão em lote | Alteração em lote de produtos, clientes, preços, estoque, fotos e etapas de pedidos. Exclusão em lote. | ambos (Omie muito explícito) | Baixa | Nenhuma |
| Transferir produtos entre contas | Copia o catálogo de uma conta Bling para outra. | Bling | Média | Nenhuma |
| Compartilhamento de cadastros entre CNPJs | Sincroniza produtos e clientes/fornecedores entre "aplicativos" (empresas) Omie. **Não** sincroniza estoque nem tabela de preço. | Omie | Média | Nenhuma |
| Cadastro Anatel | Homologação Anatel exigida por marketplaces para eletrônicos. | ambos | Baixa | Parcial |
| Tabela de medidas e compatibilidade | Tabela de tamanhos (moda/calçados) e compatibilidade de autopeças nos anúncios ML. | ambos | Média | Total |
| Ficha técnica / atributos ML | Preenchimento de atributos da categoria do ML (inclusive "campos escondidos" de nicho). | ambos | Média | Total |

## 2. Estoque

| Feature | O que faz (concreto) | Sistema | Complexidade | Dep. marketplace |
|---|---|---|---|---|
| Multi-depósito | Vários depósitos ou locais de estoque, com padrão definido. Omie: até **200 locais**, com histórico de alterações. | ambos | Média | Nenhuma |
| Transferência entre depósitos | Movimenta saldo entre depósitos. Bling define o preço considerado na transferência. Omie também via remessa/retorno com NF. | ambos | Média | Nenhuma |
| Depósito por canal / multi-origem no ML | Vincula um depósito do ERP a cada depósito do ML (filial "Multi Origem"), com opção de agrupar saldo de vários depósitos. Horário de despacho e capacidade diária de envio **por depósito** (beta no ML). | Bling | Alta (regras de sincronização por origem e capacidade) | Total |
| Saldo Full x Flex | Mostra o saldo do anúncio no depósito Flex e no Fulfillment. No Omie.Hub, ícones indicam anúncio com Full e Flex: o estoque do **Full é controlado pelo ML** e o do **Flex pelo Hub**. | ambos | Média | Total |
| Consulta de operações de estoque Full | Histórico de operações de inventário nos CDs do ML (entradas, saídas, ajustes) por ID de inventário. | Bling | Média | Total |
| Estoque Full separado no ERP | Locais de estoque separados para o Full vinculados ao produto (configuração feita **com o suporte**). | Omie.Hub | Média | Total |
| Reserva de estoque | **Bling:** pedidos em situações configuradas (até 25 situações) saem do saldo disponível como "reservado" até o lançamento. **Omie:** reserva manual e reserva para venda, remessa e ordem de produção, com consulta das reservas por produto e local. | ambos | Média | Nenhuma |
| Bloquear estoque negativo | Impede saída sem saldo. Omie documenta o impacto contábil do estoque negativo. | ambos | Baixa | Nenhuma |
| Bloqueio de edição/exclusão de lançamentos | Impede alterar lançamentos de estoque passados (Bling). Omie: **período limite** que bloqueia NF, estoque e contas fora de uma janela de datas. | ambos | Baixa | Nenhuma |
| Custo unitário na entrada | **Bling:** custo = (qtd × (unitário − desconto/qtd) + frete + ICMS-ST + FCP + seguro − ICMS presumido + outras despesas rateadas + IPI) / qtd. | Bling | Média | Nenhuma |
| Custo médio | **Bling:** custo médio ponderado das entradas desde o último balanço, configurável pelo preço do balanço ou pelo custo de lançamento, usado no relatório de estoque e no lucro de vendas. **Omie:** **CMC (Custo Médio Contábil)** recalculado a cada entrada: ((saldo × CMC anterior) + (qtd × valor)) / (saldo + qtd). Ajuste manual de CMC, CMC de kit e componentes, e CMV/CPV. | ambos | Média (dinheiro: precisa ser no servidor) | Nenhuma |
| Inventário / conferência por leitura | **Bling:** conferência de estoque por depósito com leitor de código de barras ou câmera do celular, mostrando contado x atual x diferença, gerando **balanço**. Recomenda lotes de até 500 itens, com salvamento em nuvem. **Omie:** ajuste e movimentação manual ou por planilha (não achei conferência por bip). | Bling (forte), Omie (básico) | Média | Nenhuma |
| Lotes e validade | **Bling:** lote por produto e variação, pela NF de entrada, pelo pedido de compra ou por planilha. Obrigatoriedade de fabricação/validade, alerta de vencimento, transferência de lote entre depósitos, lote no PDV, NF, OS, pedido e checkout, e relatório. **Omie:** método de consumo **Manual, FEFO, FIFO ou LIFO** automático na saída e relatório de rastreabilidade. | ambos | Média | Nenhuma |
| Localização no depósito | Endereço/localização exibido na conferência. | Bling | Baixa | Nenhuma |
| Kardex / movimentos detalhados | Extrato de movimentações por produto. | ambos | Baixa | Nenhuma |
| Posição e visão financeira do estoque | Saldo valorizado por custo médio ou por preço. | ambos | Baixa | Nenhuma |
| Multiempresas (estoque unificado) | **Bling:** "Depósito Virtual" que **soma** os saldos de várias contas/CNPJs. A conta principal sincroniza e exporta produtos para as demais. Limite por plano: Titânio 3, Diamante 5, Elite 10. **Omie:** não sincroniza estoque entre CNPJs (só cadastros). | Bling | Alta (consistência de saldo entre tenants) | Parcial |
| Ordem de produção | OP a partir de pedidos, calendário de produção, baixa de matéria-prima e entrada do acabado, e "gerar OP sob demanda". | ambos | Média | Nenhuma |
| Estoque ilimitado | Produto marcado como estoque ilimitado (sob encomenda/serviço). | Omie.Hub | Baixa | Parcial |

## 3. Vendas e pedidos

| Feature | O que faz (concreto) | Sistema | Complexidade | Dep. marketplace |
|---|---|---|---|---|
| Pedido de venda | Inclusão, edição, clonagem, cancelamento, exclusão, impressão/envio ao cliente, peso somado, vendedor obrigatório, colunas customizáveis, busca por CEP e bloqueio de produto não cadastrado. | ambos | Média | Nenhuma |
| Situações customizáveis + gerenciador de transições ★ | Situações próprias de pedido (e de compra, OS e OP). Cada transição origem→destino dispara ações: **lançar/estornar estoque, lançar/estornar contas, gerar NF-e/NFC-e, adicionar à remessa de postagem**. Também exposto na API. | Bling | Alta (motor de workflow com efeitos colaterais) | Nenhuma |
| Kanban de vendas | Etapas visuais e renomeáveis (orçamento → … → faturar → faturado), mudança de etapa em lote e filtros. | Omie | Média | Nenhuma |
| Orçamento / proposta comercial | Proposta com versões, modelo, envio ao cliente e conversão em pedido. | ambos | Baixa | Nenhuma |
| Faturamento parcial | Fatura só parte dos itens do pedido. | Bling | Média | Nenhuma |
| Agrupar pedidos numa NF | Vários pedidos do mesmo cliente numa única NF (agrupa item se preço, descrição, comissão e vendedor forem iguais). | Bling | Média | Nenhuma |
| Faturamento em lote | Seleciona N pedidos e fatura. Bling: envio de NF em **segundo plano** com status "Enviando autorização". | ambos | Média | Nenhuma |
| Venda consignada | Fluxo de consignação ("venda agenciada"). | Bling | Média | Nenhuma |
| Devolução/troca sem nota | Registro de troca sem NF. | Bling | Baixa | Nenhuma |
| Devolução de venda (com NF) | Gera NF de devolução a partir da venda. Omie cria automaticamente uma **conta a pagar** do valor devolvido. | ambos | Média (fiscal) | Nenhuma |
| Aprovação de desconto | Desconto acima do limite exige aprovação. | Bling | Baixa | Nenhuma |
| Conferência pré-nota | Valida o pedido e mostra o DANFE de pré-visualização antes de emitir. | Omie | Baixa | Nenhuma |
| Importação de pedidos dos canais | Importação automática (pedido pago) e manual/retroativa, com mapeamento de status canal→ERP (ex.: Pago→Em andamento, Cancelado/Devolução→Cancelado com estorno). | ambos | Alta (webhooks + reconciliação de estado) | Total |
| Filtro de pedidos Full | Importar todos, só Full ou exceto Full. | Bling | Baixa | Total |
| Pedidos pendentes / em revisão | **Omie.Hub:** pedido com cadastro divergente vai para "Pendentes" (editar e reprocessar). Pedido com item sem vínculo de anúncio vai para "Em revisão" até o anúncio ser importado. | Omie.Hub | Média | Total |
| ID do carrinho (pack) | Envia o ID Pack do ML no campo "nº do pedido do cliente". Bling mostra o ID do carrinho na conferência de recebíveis. | ambos | Baixa | Total |
| Bairro padrão e e-mail padrão | Preenche dados que o marketplace não envia, necessários para a NF. | Omie.Hub | Baixa | Total |
| Importar taxas e juros nos pedidos | Comissão, frete e taxa fixa do canal gravados no pedido (ver Financeiro). Juros de parcelamento de lojas. | Bling | Média | Total |
| Mensagens ao comprador ML | Mensagem automática de recebimento do pedido e envio da NF pelo chat (só ME1, a combinar e Flex; bloqueado para Full, coleta e drop-off). | Bling | Média | Total |
| Reclamações e devoluções ML (beta) | No pedido: ver a reclamação, conversar com o comprador ou mediador, oferecer devolução, reembolsar parcial/total (padrão 50%), anexar evidências, enviar rastreio ou promessa de envio, confirmar ou reportar problema na devolução recebida. | Bling | Alta (APIs de claims do ML, muitos estados) | Total |
| Central de atendimento ML | Responder e excluir perguntas, bloquear usuários, mensagens pós-venda, proteção de marca. | Bling (no Omie.Hub não encontrei) | Média | Total |
| Alerta de fraude ML | Sinaliza pedidos com alerta de fraude do ML. | Bling | Baixa | Total |
| PDV / frente de caixa | Caixa, sangria/suprimento, troco, PIX/QR, POS integrado (Stone, Cielo, Rede…) e NFC-e. | ambos | Alta (fora do escopo Pinese) | Nenhuma |
| Contratos e OS | Contratos recorrentes com faturamento em lote e ordens de serviço. | ambos | Média (fora do escopo) | Nenhuma |
| CRM | Oportunidades, funil, tarefas, kanban. | Omie | Média (fora do escopo) | Nenhuma |
| Loja virtual própria | Bling Loja Virtual (tema, SEO, domínio, frete, pagamentos). | Bling | Alta (fora do escopo) | Nenhuma |

## 4. Fiscal

| Feature | O que faz (concreto) | Sistema | Complexidade | Dep. marketplace |
|---|---|---|---|---|
| NF-e (modelo 55) | Emissão, cancelamento (inclusive regras fora do prazo), carta de correção, inutilização, clonagem, contingência, DANFE normal, simplificado e simplificado tipo 2, e envio automático de DANFE+XML ao cliente. | ambos | Alta (SEFAZ, schemas, certificados, rejeições: o Bling tem 254 artigos só de rejeições) | Nenhuma |
| NFC-e / SAT / NFS-e | Cupom de consumidor (com CSC e contingência), SAT e NFS-e (prefeituras e ambiente nacional). | ambos | Alta | Nenhuma |
| CT-e / MDF-e | Emissão/importação de CT-e (Bling e Omie). MDF-e no Omie. | ambos | Alta | Nenhuma |
| Natureza de operação / regras tributárias | **Bling:** natureza de operação com regras de cálculo automático (CFOP dentro/fora do UF, CST/CSOSN, ICMS, ST, IPI, PIS/COFINS, DIFAL, FCP, benefícios cBenef). **Omie:** **Cenário Fiscal** por NCM, atualizável a partir de um XML existente e importável de outro app. Regras de DIFAL por estado. | ambos | Alta (matriz tributária por UF, regime e NCM) | Nenhuma |
| Cenário fiscal por perfil de cliente no hub | Associa o tipo de operação (perfil do comprador) ao cenário fiscal: pedidos do marketplace já entram com os impostos preenchidos. | Omie.Hub | Média | Total |
| Emissão automática de NF dos pedidos de canal | Gera e transmite a NF ao importar o pedido. Exige **certificado A1**: com A3 só gera. Integrações com essa opção no Bling: ML, Amazon, Magalu, Americanas, Shopee, Temu, Kwai e TikTok. | ambos | Alta | Total |
| Emitir NF só após liberar a etiqueta ML ★ | "Gerar e emitir NF ao disponibilizar o pedido para envio": evita NF antes da data de envio definida pelo ML (vendas com despacho futuro). | Bling (ago/2025) | Média | Total |
| Envio da NF ao ML | Envia os dados fiscais (chave/XML) ao ML automaticamente ao emitir. Se o pedido ainda não foi liberado, reenvia sozinho quando liberar. Importação manual de XML no ML. Trata o substatus `invoice_pending` do xd_drop_off. | ambos (Omie: documentado para ME1 e para NF/etiqueta no Hub) | Média | Total |
| NF do Full emitida pelo ML (Faturador) | Importa os XMLs das NFs que o **ML emite** nas vendas Full e vincula ao pedido. Cancelamento sincronizado. Importa NF de entrada e de **transferência Full** (sem alterar estoque). Exige filial "fulfillment", 4 casas decimais e nome de natureza igual ao do ML. | Bling | Alta (conciliação NF-ML x pedido x estoque) | Total |
| Faturador ML (envio de dados fiscais) | Exporta NCM, CEST, GTIN, origem, CSOSN, CFOP, ICMS, IPI e COFINS do produto para o Faturador do ML emitir. Lança contas e estoque ao importar o XML. ICMS partilha. **Só para SP** e só para produto simples/variação (kit não). | Bling | Alta | Total |
| NFs adicionais Amazon | Acesso a NFs extras emitidas pela Amazon. | Omie.Hub | Média | Total |
| Remessa e retorno para armazém | NF de remessa (CFOP 5.905/6.905, CST 41) e retorno, usadas para armazém de terceiros. Também remessa para conserto, demonstração, feira e locação, além de bonificação, transferência, ajuste, complementar (ICMS, ST, IPI, frete, DIFAL), importação, exportação e anulação de frete. | ambos | Média–Alta | Parcial (útil no envio ao Full, inf.) |
| Devolução de compra / venda | Naturezas de devolução, NF de devolução ao fornecedor a partir da compra, NF de entrada de devolução de venda e IPI devolvido. | ambos | Média | Nenhuma |
| Manifesto do destinatário + captura de NF-e | **Bling:** consulta as NF-e emitidas contra o CNPJ (até 3 meses), com manifesto individual e em massa e ciência automática (exige A1). **Omie:** "Agente de NF-e" (consulta SEFAZ-AN e importa NF-e/CT-e no Kanban de Compras) + **e-mail exclusivo de NF-e** que captura XMLs enviados. Ambos alertam sobre o conflito de NSU com outro sistema. | ambos | Alta (DF-e/NSU, certificado) | Nenhuma |
| Certificado A1/A3 | A1 no servidor. A3 via aplicativo local (Bling NF-e/JDK). Compra de certificado dentro do sistema. Alerta de vencimento. | ambos | Média | Nenhuma |
| SPED / Sintegra / GNRE / DARE | Bling: SPED Fiscal e Contribuições, Sintegra, GNRE e DARE-SP. Omie: SPED EFD, Contribuições, ECD, Sintegra, integração contábil e Painel do Contador. | ambos (Omie mais contábil) | Alta | Nenhuma |
| Reforma tributária (IBS/CBS) | Campos IBS/CBS em NF-e, NFC-e e NFS-e, eventos novos (112110, 112130, 211110…), notas de débito/crédito e CNPJ alfanumérico. | ambos | Alta (regulatório em movimento) | Nenhuma |
| Intermediador (NT 2020.006) | Campo intermediador do marketplace na NF. | ambos (inf. Omie) | Baixa | Total |
| Classificação fiscal por IA | "Omie.IA Fiscal": classificação automática de produtos (NCM/tributação). | Omie (add-on) | Alta (IA + responsabilidade fiscal) | Nenhuma |
| Espaço do contador | Troca de arquivos com o contador e exportação de XMLs. | ambos | Baixa | Nenhuma |

## 5. Compras

| Feature | O que faz (concreto) | Sistema | Complexidade | Dep. marketplace |
|---|---|---|---|---|
| Pedido de compra | Criação, importação por planilha, preço padrão por produto no PC, relatório, lançar/estornar estoque e contas pelo PC, e lote informado no PC. Omie: vencimentos específicos, numeração própria, encerrar e cancelar. | ambos | Média | Nenhuma |
| Requisição de compra | Pedido interno antes do PC. | Omie | Baixa | Nenhuma |
| Aprovação de pedido de compra | Permissão "Aprovar Pedidos de Compra": quem não tem solicita aprovação, e os aprovadores recebem e-mail. O PC só se associa à NF depois de aprovado. | Omie | Média | Nenhuma |
| Kanban de compras | Etapas de requisição → pedido → faturado pelo fornecedor → recebido, renomeáveis. | Omie | Média | Nenhuma |
| Entrada por XML | Importa o XML da NF-e de compra (um ou vários), concilia os itens da nota com os produtos do estoque (de-para por fornecedor), vincula ao PC (inclusive na importação) e gera contas a pagar a partir das parcelas. | ambos | Média–Alta (de-para de itens, conversão de unidade) | Nenhuma |
| Check-in de recebimento (beta) ★ | Confere a NF de entrada (só XML) antes de lançar estoque: manual, por bip GTIN/DUN ou check-in em lote. Registra pendências (faltando, danificado, incorreto, recusado). | Bling | Média | Nenhuma |
| Sugestão de compra | **Omie:** ponto de pedido = estoque mínimo + (consumo médio × lead time), considerando previsões de entrada e saída, por local de estoque. **Bling:** dashboard "Sugestão de Compra" por período de venda, dias de cobertura, depósito, reserva, PCs abertos, fornecedor e marca. | ambos | Média | Parcial |
| Produtos abaixo do mínimo | Relatório de reposição. | ambos | Baixa | Nenhuma |
| Nota de importação | NF de importação com custo complementar e fechamento de valores. | ambos | Alta | Nenhuma |
| CT-e de entrada | Recebimento de CT-e (frete) e relatórios por transportadora. | ambos | Média | Nenhuma |

## 6. Financeiro

| Feature | O que faz (concreto) | Sistema | Complexidade | Dep. marketplace |
|---|---|---|---|---|
| Contas a pagar/receber | Inclusão, baixa total/parcial, estorno, recorrentes (com tipos), juros e multa automáticos, cartas de cobrança, nota promissória, ficha financeira, baixa e alteração em lote, e-mail de vencimento. | ambos | Média | Nenhuma |
| Lançamento automático de contas na venda | Gera conta a receber ao incluir o pedido ou ao importar a NF (no Omie.Hub: "Gerar financeiro ao incluir pedido/importar NF"). | ambos | Média | Parcial |
| Caixas e bancos / contas financeiras | Contas internas e bancárias, transferências, ajuste de saldo e origem dos lançamentos. | ambos | Baixa | Nenhuma |
| Categorias de receita/despesa | Plano de categorias, editável em massa e importável. Categoria padrão para taxas, juros e descontos. | ambos | Baixa | Nenhuma |
| **Taxas de marketplace no pedido** ★ | **Automático para ML, Shopee e Amazon.** Grava no pedido a comissão + frete do vendedor + taxa fixa. O financeiro recebe uma **entrada pelo bruto e uma saída pela comissão** ("Taxas de Marketplace"). Ex.: produtos 100, cupom ML −10, frete pago pelo ML +20 → pago pelo comprador 110, base 100 − 5% = líquido 95, comissão registrada 15. Semi-automático (% fixo) para outros canais. Limitação: não permite baixa parcial quando a conta tem taxa. | Bling | Alta (modelo de taxas do ML muda, frete, cupons) | Total |
| **Conferência de Recebíveis ML** ★ | Audita o **repasse real do ML contra o líquido esperado** por pedido: bruto, líquido esperado, repasse, comissão, taxa fixa, frete do vendedor e do comprador, diferença ±, situação financeira (a liberar, retido, reembolso, contestação), situação da conferência (conferido/com diferença) e aba de análise por período. | Bling (Meu Negócio) | Alta (API de pagamentos/liberações do ML/Mercado Pago, casar pedido↔movimento) | Total |
| Conciliação de marketplace (terceiros) | TrackCash na Omie.Store: cruza vendas, meios de pagamento e repasses dos intermediários com os títulos no Omie e dá baixa automática. | Omie (via app de terceiro) | Alta | Total |
| Conciliação bancária por OFX | **Bling:** importa OFX, sugere correspondência por valor (contas a pagar/receber e lançamentos), vincula vários títulos a um extrato, cria lançamento, ignora/oculta, concilia em massa e reverte. **Omie:** OFX/Money 2000 (CSV só alguns bancos), "carteiras virtuais" (**Mercado Pago**, PicPay…) por OFX/planilha, 1 extrato ↔ N lançamentos, atualização em lote e desconciliação. | ambos | Média–Alta (matching) | Parcial |
| Extrato automático via API bancária / Open Finance ★ | **Omie:** extrato via API (BTG, Cora, Inter, Santander, Sicoob, BB…) + **Open Finance** (consentimento com prazo e captura automática do extrato). **Bling:** só OFX (0 ocorrências de "Open Finance"). | Omie | Alta (Open Finance exige parceiro regulado, inf.) | Nenhuma |
| Boletos / CNAB / PIX / link de pagamento | Integração bancária para boleto registrado (remessa/retorno), DDA, PIX e link de pagamento, contas digitais próprias (Bling Conta, Omie.CASH) com conciliação automática, envio automático de boleto/PIX ao cliente, pagamentos por CNAB. | ambos | Alta (fora do núcleo Pinese, inf.) | Nenhuma |
| Faturamento agrupado | Agrupa várias contas do cliente numa cobrança. | Bling | Média | Nenhuma |
| Cashback | Gestão de cashback de clientes. | Bling | Média | Nenhuma |
| Taxas de forma de pagamento | Taxa por forma de pagamento (cartão) e baixa considerando o prazo de recebimento. | Bling | Média | Parcial |
| DRE gerencial | **Bling:** DRE por **competência** com grupos (receita bruta, deduções, despesas operacionais, financeiras, IR/CSLL), via vínculo categoria→grupo (só na categoria-mãe). **Omie:** DRE por data de registro ou emissão, opção de **incluir o CMC das remessas** e dos produtos usados em serviços, juros e descontos no DRE, e relatório DRE x faturamento. | ambos | Média–Alta (precisão contábil) | Nenhuma |
| Fluxo de caixa | Padrão, resumido, por categoria e por cliente/fornecedor (Omie). Futuras entradas e saídas (Bling). Orçamento de caixa (Omie API). | ambos | Média | Nenhuma |
| Comissão de vendedores via contas a pagar | Gera contas a pagar das comissões. | Bling | Média | Nenhuma |
| Antecipação / empréstimo / desconto de duplicatas | Ofertas de crédito embutidas. | ambos | n/a (serviço financeiro) | Nenhuma |

## 7. Marketplaces / hub (com foco no Mercado Livre)

> **Arquitetura.** No **Bling**, o hub é nativo: integrações na "Central de Extensões" (250+ integrações, ~1.117 artigos de ajuda). No **Omie**, o hub é **produto separado (Omie.Hub)**, com login próprio, ligado ao ERP por App Key/Secret. É **pago à parte**: R$109,90 (2.000 anúncios, 2 canais) até R$850 + 1% do faturamento (sem limites), com implantação de R$300, e grátis no plano Multivarejo (a partir de R$419). A integração ML no Omie.Hub **exige o suporte para finalizar a configuração**, e o Full também.

| Feature | O que faz (concreto) | Sistema | Complexidade | Dep. marketplace |
|---|---|---|---|---|
| Autenticação OAuth com ML | Conecta a conta do ML. Várias integrações = várias contas. Erro comum: duas integrações logadas na mesma conta ML causam **pedidos duplicados**. | ambos | Média | Total |
| Multi-conta ML | Várias lojas ML no mesmo ERP. Bling **clona anúncios para múltiplas contas ML** (até 5.000 de uma vez). Omie.Hub conta o limite por "marketplaces ou lojas" no plano (inf.: 2 contas = 2 lojas). | ambos | Média | Total |
| Gestão de anúncios ML (tela própria) | Importa anúncios próprios (vínculo automático/manual por SKU), **importa anúncios de terceiros** como modelo, cria anúncio simples ou com variação com categoria sugerida, rascunhos, edição pós-publicação, exclusão, filtros avançados e edição em massa por planilha. | Bling (Omie.Hub: publicar, importar em lote, alterar preço, data, categoria, reenviar imagens, excluir, recuperar ignorado) | Alta (API de itens, atributos e moderações do ML) | Total |
| Modelo User Product (UP) | Suporte ao novo modelo do ML: cada variação com preço, modalidade (Clássico/Premium), envio e estoque próprios, organizada em "famílias" (sem produto-pai). Bling migra automaticamente. | ambos | Alta (mudança estrutural no vínculo SKU↔anúncio) | Total |
| Sincronização de estoque | ERP→ML por depósito escolhido. Zerar estoque pausa/inativa o anúncio (opção). No Full, o ML gerencia o estoque e o ERP só pode inativar. | ambos | Alta (concorrência, latência, Full x Flex) | Total |
| Sincronização de preço | Ordem de prioridade (Bling): preço promocional do canal > preço do canal > preço de venda do cadastro. Alteração de preço em massa. | ambos | Média | Total |
| Regras de preço automáticas ML | Sugestão de preço do ML. **Preço por quantidade (PxQ/atacado B2B)**: até 5 faixas, quantidade mínima ≥2, preços decrescentes, "com ou sem impostos" (preço líquido por região). | ambos | Média | Total |
| Central de ofertas / promoções ML | Aprova campanhas e co-participação do ML (valor pago pelo vendedor x pelo ML), aplica a sugestão dentro da faixa mín./máx., ofertas 100% financiadas vinculadas sozinhas, cupons de desconto ML. | Bling | Alta (API de promoções) | Total |
| Kits de anúncios / kits virtuais ML (beta) | Agrupa até 6 anúncios publicados numa oferta "kit" (só UP, só item novo, sem duplicar composição). Preço automático (desconto % sobre a soma, sincronizado) ou manual. Diferente do kit de estoque. | ambos | Média–Alta | Total |
| Catálogo ML | Vincular ao catálogo, sugerir cadastro novo, consultar preços do catálogo, sincronizar publicações de catálogo, destaque. | Bling | Alta | Total |
| Experiência de compra / moderações | Mostra a nota de experiência de compra, moderações e moderações com pausa. | ambos (Omie: experiência de compra) | Média | Total |
| Pesos e dimensões em massa | Atualiza dimensões para evitar suspensões no ML. | Bling | Baixa | Total |
| Migrar tipo de envio do anúncio | Troca a modalidade logística (ex.: para ME1, Flex) pelo hub. | Omie.Hub (e Bling no produto: ativar/desativar Flex) | Média | Total |
| Full (Fulfillment) | Pedidos Full importados (ou filtrados), NF do Full via Faturador, transferências Full, consulta de inventário Full e desativação guiada. | ambos | Alta | Total |
| Flex | Serviço Flex no pedido/NF, etiquetas Flex, rastreio e ativação Flex por produto. | Bling (Omie: só sinalização Full+Flex) | Média | Total |
| ME1 (envio próprio) | Ativa ME1 no anúncio, envia dimensões, atualiza status logísticos por etapa, envia NF e rastreio ao ML e informa motivo de insucesso de entrega. | ambos | Média | Total |
| Programa Decola / reputação | Benefício de reputação para novos vendedores. | Bling | Baixa | Total |
| Enriquecimento de anúncio por IA | Serviços pagos de enriquecimento de dados e tratamento de imagens do anúncio com IA. | Bling | Alta (IA) | Total |
| Margem por pedido/SKU (ver BI) | Margem de contribuição por pedido ML/Shopee/Amazon e por SKU (ML). | Bling | Média | Total |
| Outros canais | Bling: dezenas (Amazon, Shopee, Magalu, Shein, TikTok, Temu, Kwai, Leroy, MadeiraMadeira, Casas Bahia…). Omie.Hub: Amazon (FBA/DBA/Self/Smart Connect), Americanas, Magalu, MadeiraMadeira, Casas Bahia, Netshoes, Shopee (com Full), Shein, TikTok, Leroy, além de lojas (Nuvemshop, Tray, Shopify, Woo…). | ambos | Alta (cada uma) | Total |

## 8. Logística

| Feature | O que faz (concreto) | Sistema | Complexidade | Dep. marketplace |
|---|---|---|---|---|
| **Checkout de pedidos (bipagem)** ★ | Abre o pedido por nº, rastreio ou chave da NF. Bipa SKU/GTIN (ou câmera do celular). Quantidade antes do bip. Trava "Em separação" com nome do usuário (evita 2 pessoas no mesmo pedido). Filtra por loja, logística e etiqueta disponível. Desmembra kit. Mostra o lote. | Bling (Omie: **não encontrei** checkout por bip nativo, só apps WMS de terceiros na Store, ex. Wemov) | Média | Parcial |
| Automações pós-checkout | Ao finalizar a conferência (configuração **por usuário**): imprimir DANFE, DANFE simplificado, etiqueta, DANFE simplificado + etiqueta, declaração de conteúdo (1 ou 2 vias), **gerar NF, enviar à SEFAZ, enviar remessa de postagem**. Lançar estoque após o checkout. | Bling | Média–Alta (orquestra NF + etiqueta + impressão) | Parcial |
| **Checkout por produto único** ★ | Para pedidos de 1 item: soma todos os pedidos do mesmo SKU e confere de uma vez (bipa 50 canetas = 50 pedidos). Alterna com o modo "por pedido" mantendo o progresso. | Bling | Média | Parcial |
| Relatório de separação (picking list) | Lista consolidada de itens a separar, com filtros. | Bling | Baixa | Nenhuma |
| Etiquetas em lote | Imprime etiquetas de vários pedidos ou NFs (uma aba por logística) ou por objetos de uma logística. **PDF ou ZPL** (Zebra). DANFE simplificado + etiqueta na mesma página, com tamanho configurável. Impressão automática de etiqueta. | ambos (Omie: etiqueta ZPL ML via Hub/Store) | Média | Parcial |
| Objetos de postagem / remessas | Cada pedido vira um "objeto" com status. Remessas geradas e enviadas automaticamente. Correção de remessa com erro. Status ML: etiqueta disponível, aguardando etiqueta, postagem atrasada, prazo de despacho etc. | Bling | Média | Parcial |
| Prazo de despacho ML | Mostra a data/hora limite de despacho por objeto. Filtra "etiqueta disponível" para saber o que já pode postar. | Bling | Baixa | Total |
| **Agrupamento de coleta ML (bundle)** ★ | Junta vários pacotes num grupo com **uma etiqueta única** (o coletor não bipa pacote a pacote). Estados aberto → fechado (ao imprimir) → finalizado (coletado). CSV detalhado. Pacote não pode estar em 2 grupos. Só para contas habilitadas pelo ML. | ambos | Média | Total |
| Horários de despacho e capacidade | Tempo de processamento e capacidade máxima diária de envios por serviço e depósito. | Bling | Média | Total |
| Romaneio | Relatório de romaneio por serviço logístico, cidade/UF, peso e frete, com campo de assinatura do coletor. Omie: romaneio de frete no Painel de Frete. | ambos | Baixa | Nenhuma |
| Cotação de frete multi-transportadora | Compara preço e prazo em tempo real. Omie: Correios, Loggi, Mandaê, Melhor Envio, Frenet, Intelipost, Datafrete, com parâmetros de precificação e troca automática de etapa do pedido após a cotação. Bling: idem por logística + "cotação por produto". | ambos | Alta (N APIs) | Nenhuma |
| Gateway de frete próprio | Bling Envios (via Melhor Envio): saldo/carteira, compra automática de etiquetas, mapeamento automático de serviço, logística reversa, ocorrências, comprovante de entrega e painel de métricas. | Bling | Alta | Nenhuma |
| Integrações logísticas | Correios (contrato, PLP, valor declarado, suspensão de entrega, reversa), Jadlog, Loggi, Total Express, Jamef, Mandaê, Intelipost, Frenet, além das logísticas de marketplace (Mercado Envios, Shopee, Amazon DBA, Magalu, Shein, TikTok, Temu…). Logística personalizada via API. | ambos | Alta | Parcial |
| Rastreamento + e-mail ao cliente | Rastreio por objeto, URL de rastreio personalizável e e-mail automático por mudança de status. | Bling | Média | Nenhuma |
| Declaração de conteúdo (DC-e) | Emite a declaração de conteúdo eletrônica para envios sem NF. | Bling | Média | Nenhuma |
| Logística reversa | Gera, cancela e refaz a reversa (Correios, Bling Envios). | Bling | Média | Nenhuma |
| Diferenciar objeto Full x normal | Identifica postagens geridas pelo Full. | Bling | Baixa | Total |

## 9. Relatórios / BI

| Feature | O que faz (concreto) | Sistema | Complexidade | Dep. marketplace |
|---|---|---|---|---|
| Relatórios padrão por módulo | Vendas (avançado, por produto com **lucro** por custo médio, curva ABC), compras, estoque, financeiro. Omie: dezenas de relatórios por categoria, departamento, projeto, fornecedor, transportadora, período. | ambos | Média | Nenhuma |
| Relatório customizado | Monta um relatório escolhendo colunas e filtros (coluna = valor), agrupamento, gráfico e clonagem. | Bling | Média–Alta (query builder seguro) | Nenhuma |
| **Margem de contribuição por pedido** ★ | Margem = faturamento − frete do vendedor − custo do produto − impostos − taxas do marketplace (não inclui ADS). ML, Shopee e Amazon. Planos Titânio+. | Bling | Média | Total |
| **Margem por SKU (ML)** ★ | Mesma fórmula agregada por produto. | Bling | Média | Total |
| Previsão de estoque (giro) | Dias de cobertura pelo histórico de vendas: produto, variação, kit ou "desmembrado" (componentes). Vendas por canal e por dia. | Bling | Média | Parcial |
| Produtos sem movimentação | Itens parados. | Bling | Baixa | Nenhuma |
| Dashboards diário, mensal e 24 meses por canal | Faturamento e quantidade por período e canal, novos x recorrentes, por vendedor, metas, atividade da carteira. | Bling | Média | Parcial |
| Consolidação multi-conta | Dashboards de até 3/5/7 contas Bling. Omie: "Relatórios Consolidados" entre CNPJs. | ambos | Média | Nenhuma |
| Meu Negócio no WhatsApp | Consulta faturamento, estoque e vendas pelo WhatsApp. | Bling (Omie: "Omie ERP no WhatsApp") | Média | Nenhuma |
| Exportação para BI externo | Omie documenta Power BI, Excel e Google Sheets consumindo a API, com paginação e exemplos de query. | Omie | Baixa (doc) | Nenhuma |
| Assistente de IA no ERP | Chat lateral com LLM: consulta dados, executa ações (cadastrar, alterar), aceita anexo de imagem ou planilha, por créditos. | Bling | Alta | Nenhuma |
| **MCP oficial** ★ | **Bling MCP** (beta) e **Omie MCP** (`https://api.omie.com.br/mcp`, OAuth com o login do usuário, respeita as permissões, pede confirmação para excluir/cancelar, multi-empresa, "skills" prontas). | ambos | Média (expor a API própria como MCP) | Nenhuma |

## 10. Outros (usuários, auditoria, API, automações, apps)

| Feature | O que faz (concreto) | Sistema | Complexidade | Dep. marketplace |
|---|---|---|---|---|
| Usuários e permissões | **Bling:** um admin único, demais com permissões por módulo e por ação (incluir/editar, excluir, funções específicas como "Importar taxas de marketplace"), restrição de tipos de contato visíveis, limite por plano (5/50/100/ilimitado). **Omie:** **Grupos de Acesso** (padrão Omie copiáveis + "Meu Grupo" editável) e usuários ilimitados. | ambos | Média | Nenhuma |
| Auditoria | **Bling:** "Registro de atividades" (só admin) com filtros por período, módulo, operação, usuário, código e descrição, somente consulta. **Omie:** histórico de alterações nos cadastros + **histórico de exclusões** por módulo (usuário, data/hora, registro), sem restauração. Histórico de APIs e webhooks. | ambos | Média | Nenhuma |
| Segurança | MFA (Bling e Omie.Hub 2FA), OTP por e-mail, link externo com senha e anonimização, camada de segurança no DANFE. Omie: **Consultor de Segurança** (score e recomendações). | ambos | Média | Nenhuma |
| Alertas | E-mail de estoque baixo, contas em atraso, lote vencendo etc. Central de notificações (Omie). | ambos | Baixa | Nenhuma |
| API | **Bling v3:** REST/JSON, OAuth2 (code expira em 1 min, refresh_token 30 dias), **3 req/s e 120 mil req/dia**, 429 ao estourar. Cobre produtos, variações, estruturas, lotes, depósitos, estoques, pedidos de venda e compra (lançar/estornar estoque e contas, gerar NF-e/NFC-e), NF-e, NFC-e, NFS-e, contas a pagar e receber, caixas, categorias, formas de pagamento, logísticas (objetos, remessas, etiquetas), **anúncios** (criar, publicar, pausar), canais de venda, situações e transições, campos customizados, contratos, OP, OS, vendedores, notificações. **Omie:** SOAP ou JSON-RPC via POST (REST "em desenvolvimento"), App Key + Secret, **960 req/min por IP, 240 por IP+app+método, 4 simultâneas**, consulta repetida do mesmo ID em 60 s bloqueada, 10 erros seguidos = bloqueio de 30 min. Cobre geral, CRM, finanças (extrato, orçamento de caixa, PIX, boletos, DRE), estoque, vendas, cupom fiscal, serviços, contador. | ambos | n/a (referência de design) | Nenhuma |
| Webhooks | **Bling:** pedido de venda, produto, estoque, estoque virtual, produto-fornecedor, NF/NFC-e (created/updated/deleted), assinatura **HMAC-SHA256** (`X-Bling-Signature-256`), 2xx em até 5 s, retentativas por até 3 dias, sem garantia de ordem, desativa o recurso se falhar. **Omie:** eventos agrupados por app+endpoint, **FIFO**, timeout de 7 s, 3 tentativas e depois **DLQ** com 1 tentativa a cada 10 min por até 5 dias. Uma falha **bloqueia a fila** do grupo. | ambos | Média | Nenhuma |
| Apps / loja de integrações | Bling: Central de Extensões (250+), apps públicos e privados, homologação de apps, contrato comercial. Omie: **Omie.Store** (TrackCash, WMS etc.), Pluga e similares. | ambos | n/a | Parcial |
| Automações configuráveis | Bling: "80+ automações" (importação automática, regras de operação, meio de pagamento por identificador, lançamento automático de contas, reserva, NF automática ao importar, ao criar ou ao mudar situação, transições, checkout). Omie: troca automática de etapa pós-cotação, envio automático de documentos, DDA e CNAB. | ambos | Alta (motor de regras) | Parcial |
| Configuração "One-Step" de canal | Assistente rápido para integrar um canal com padrões recomendados. | Bling | Média | Total |
| Backup / cópia de dados | Exportação de backup dos dados e dos produtos vinculados à loja. Planos com cota de **armazenamento em MB** (60 MB a 5 GB de dados). | Bling | Baixa | Nenhuma |
| App mobile | Bling app (checkout e conferência por câmera). App Omie (iOS/Android, painel de tarefas e do contador). | ambos | Média | Nenhuma |
| Ambiente beta / casas decimais | Bling permite configurar as casas decimais de preço e quantidade (exigido para importar NF do Full com 4 casas). | Bling | Baixa | Parcial |

---

## (a) Fontes efetivamente lidas

**Corpora completos baixados e pesquisados (texto integral):**
- Bling, central de ajuda via API Zendesk: `https://ajuda.bling.com.br/api/v2/help_center/pt-br/articles.json` (3.030 artigos, 31 páginas), mais seções e categorias.
- Omie, central de ajuda (Intercom): `https://ajuda.omie.com.br/pt-BR/` (350 coleções, ~2.900 artigos rastreados).

**Bling, artigos lidos na íntegra:**
- https://ajuda.bling.com.br/hc/pt-br/articles/4414429622167 — Configurações do Mercado Livre
- https://ajuda.bling.com.br/hc/pt-br/articles/4835261790999 — Configurações do Mercado Livre Fulfillment
- https://ajuda.bling.com.br/hc/pt-br/articles/360047013493 — Faturador do Mercado Livre
- https://ajuda.bling.com.br/hc/pt-br/articles/28205897101463 — Múltiplos depósitos no Mercado Livre
- https://ajuda.bling.com.br/hc/pt-br/articles/8930132309911 — Taxas de marketplaces nas vendas
- https://ajuda.bling.com.br/hc/pt-br/articles/13061204299799 — Cálculo da importação automática de taxas
- https://ajuda.bling.com.br/hc/pt-br/articles/41957704803607 — Conferência de Recebíveis (repasses ML)
- https://ajuda.bling.com.br/hc/pt-br/articles/1500000328521 — Operações de estoque fulfillment
- https://ajuda.bling.com.br/hc/pt-br/articles/17655343661591 — Saldo dos depósitos Flex e Fulfillment
- https://ajuda.bling.com.br/hc/pt-br/articles/360046384474 — Desativação do Fulfillment
- https://ajuda.bling.com.br/hc/pt-br/articles/30574574392215 — Guia da Gestão de Anúncios ML
- https://ajuda.bling.com.br/hc/pt-br/articles/43955854380439 — Kits de anúncios ML
- https://ajuda.bling.com.br/hc/pt-br/articles/35812079768087 — Modelo User Product (UP)
- https://ajuda.bling.com.br/hc/pt-br/articles/43910635031447 — Clonar anúncios para outras contas
- https://ajuda.bling.com.br/hc/pt-br/articles/25502549789847 — Reclamações e devoluções ML
- https://ajuda.bling.com.br/hc/pt-br/articles/40335643368855 — Agrupamento de coleta Mercado Envios
- https://ajuda.bling.com.br/hc/pt-br/articles/20316191778839 — Regras de sincronização de preço ML
- https://ajuda.bling.com.br/hc/pt-br/articles/27090649543447 — Automatizar preços ML (PxQ)
- https://ajuda.bling.com.br/hc/pt-br/articles/360040935414 — Pedidos de venda do ML
- https://ajuda.bling.com.br/hc/pt-br/articles/1500003971762 — Central de Ofertas ML
- https://ajuda.bling.com.br/hc/pt-br/articles/360047021393 — Central de Atendimento ML
- https://ajuda.bling.com.br/hc/pt-br/articles/360040936414 — Envio das notas para o ML
- https://ajuda.bling.com.br/hc/pt-br/articles/24767304981399 — Prazo de despacho Mercado Envios
- https://ajuda.bling.com.br/hc/pt-br/articles/360047027253 — Integração com o Mercado Envios
- https://ajuda.bling.com.br/hc/pt-br/articles/34188805749399 — NF só após liberar a etiqueta ML
- https://ajuda.bling.com.br/hc/pt-br/articles/10978998085527 — Pedidos ML duplicados
- https://ajuda.bling.com.br/hc/pt-br/articles/18134856833943 — Checkout de pedidos
- https://ajuda.bling.com.br/hc/pt-br/articles/24112035701143 — Automações do checkout
- https://ajuda.bling.com.br/hc/pt-br/articles/39430448474519 — Checkout por produto único
- https://ajuda.bling.com.br/hc/pt-br/articles/360039130893 — 9 automações do Bling
- https://ajuda.bling.com.br/hc/pt-br/articles/360036512833 — Reserva de estoque
- https://ajuda.bling.com.br/hc/pt-br/articles/360056494793 — Cálculo do custo unitário
- https://ajuda.bling.com.br/hc/pt-br/articles/360056093354 — Relatório de custo médio
- https://ajuda.bling.com.br/hc/pt-br/articles/37029128283927 — Check-in de recebimentos
- https://ajuda.bling.com.br/hc/pt-br/articles/360050781314 — Conferência de estoque por leitura
- https://ajuda.bling.com.br/hc/pt-br/articles/38601853643287 — Multiempresas (estoque unificado)
- https://ajuda.bling.com.br/hc/pt-br/articles/360057118174 — Consultar, manifestar e importar NF de entrada
- https://ajuda.bling.com.br/hc/pt-br/articles/21819268802071 — Gestão de Compras
- https://ajuda.bling.com.br/hc/pt-br/articles/360046395213 — NF de remessa e retorno para armazém
- https://ajuda.bling.com.br/hc/pt-br/articles/360039127833 — Gerar notas automaticamente
- https://ajuda.bling.com.br/hc/pt-br/articles/13101706677527 — Emissão automática de NF dos pedidos de lojas
- https://ajuda.bling.com.br/hc/pt-br/articles/35864097628311 — Envio de NF em segundo plano
- https://ajuda.bling.com.br/hc/pt-br/articles/360037858134 — Uma NF para vários pedidos
- https://ajuda.bling.com.br/hc/pt-br/articles/360051642334 — Conciliação bancária
- https://ajuda.bling.com.br/hc/pt-br/articles/360044185094 — Relatório DRE
- https://ajuda.bling.com.br/hc/pt-br/articles/35658275718551 — Dashboard Margem por Pedido
- https://ajuda.bling.com.br/hc/pt-br/articles/44218974266775 — Dashboard Margem por SKU
- https://ajuda.bling.com.br/hc/pt-br/articles/26792590540951 — Dashboard Sugestão de Compra
- https://ajuda.bling.com.br/hc/pt-br/articles/26776902193687 — Dashboard Previsão de Estoque
- https://ajuda.bling.com.br/hc/pt-br/articles/360036059673 — Relatório customizado
- https://ajuda.bling.com.br/hc/pt-br/articles/7581499239447 — Etiquetas em lote
- https://ajuda.bling.com.br/hc/pt-br/articles/360039266373 — Relatório de romaneio
- https://ajuda.bling.com.br/hc/pt-br/articles/360035558634 — Gerenciar usuários
- https://ajuda.bling.com.br/hc/pt-br/articles/360039073174 — Registro de atividades
- https://ajuda.bling.com.br/hc/pt-br/articles/360039070654 — Gerenciador de transições (pedidos de venda)
- https://ajuda.bling.com.br/hc/pt-br/articles/43357622051607 — Bling MCP
- https://ajuda.bling.com.br/hc/pt-br/articles/40765473905815 — Assistente de IA
- https://ajuda.bling.com.br/hc/pt-br/articles/19966749418391 — Limites de pedidos importados por plano
- https://ajuda.bling.com.br/hc/pt-br/articles/30224184866583 — Planos e preços de abril de 2026
- A lista de títulos de todas as seções (cadastros, estoque, NF, logística, financeiro etc.) também foi usada como evidência de que a feature existe.

**Bling, outras páginas:**
- https://www.bling.com.br/funcionalidades
- https://www.bling.com.br/planos-e-precos
- https://developer.bling.com.br/referencia e o OpenAPI `https://developer.bling.com.br/build/assets/openapi-Dw6cY8yQ.json` (todos os endpoints)
- https://developer.bling.com.br/limites
- https://developer.bling.com.br/webhooks

**Omie, artigos lidos na íntegra:**
- https://ajuda.omie.com.br/pt-BR/articles/4701742-configurando-o-marketplace-mercado-livre
- https://ajuda.omie.com.br/pt-BR/articles/12051064-configurando-o-fulfillment-da-loja-mercado-livre-via-omie-hub
- https://ajuda.omie.com.br/pt-BR/articles/12607706-conhecendo-as-opcoes-de-configuracao-da-sua-loja-no-omie-hub
- https://ajuda.omie.com.br/pt-BR/articles/12986931-configurando-kits-virtuais-do-mercado-livre-via-omie-hub
- https://ajuda.omie.com.br/pt-BR/articles/11588389-configurando-cenarios-fiscais-no-omie-hub
- https://ajuda.omie.com.br/pt-BR/articles/10206157-gerenciando-pedidos-no-omie-hub
- https://ajuda.omie.com.br/pt-BR/articles/15021027-agrupando-pacotes-para-coleta-no-mercado-livre
- https://ajuda.omie.com.br/pt-BR/articles/7939984-seja-bem-vindo-a-ao-omie-hub
- https://ajuda.omie.com.br/pt-BR/articles/14659796-configurando-precos-liquidos-pxq-no-mercado-livre-pelo-omie-hub
- https://ajuda.omie.com.br/pt-BR/articles/11824589-buscando-nf-es-no-omie-hub
- https://ajuda.omie.com.br/pt-BR/articles/9762305-associando-o-tipo-de-frete-para-geracao-de-etiquetas-de-envio-via-omie-hub
- https://ajuda.omie.com.br/pt-BR/articles/15452425-configurando-o-mercado-envios-1-me1-no-mercado-livre-via-omie-hub
- https://ajuda.omie.com.br/pt-BR/articles/4701756-configurando-a-atualizacao-automatica-do-valor-de-venda-via-omie-hub
- https://ajuda.omie.com.br/pt-BR/articles/9279485-acessando-o-painel-de-frete
- https://ajuda.omie.com.br/pt-BR/articles/10838485-utilizando-o-calculo-automatico-de-embalagens-de-transporte
- https://ajuda.omie.com.br/pt-BR/articles/15459744-configurando-o-open-finance
- https://ajuda.omie.com.br/pt-BR/articles/4112418-conciliacao-bancaria-conciliando-carteiras-virtuais
- https://ajuda.omie.com.br/pt-BR/articles/499175-conciliacao-bancaria-importando-o-extrato-no-omie
- https://ajuda.omie.com.br/pt-BR/articles/498779-o-que-e-um-departamento
- https://ajuda.omie.com.br/pt-BR/articles/6672272-relatorios-financas-gerando-e-analisando-dre
- https://ajuda.omie.com.br/pt-BR/articles/6597224-entendendo-como-e-feito-o-calculo-do-cmc
- https://ajuda.omie.com.br/pt-BR/articles/1350609-importando-a-nf-e-de-fornecedor-automaticamente-via-agente-e-e-mail-de-nf-e
- https://ajuda.omie.com.br/pt-BR/articles/16803648-configurando-as-etapas-do-kanban-de-compras
- https://ajuda.omie.com.br/pt-BR/articles/6328913-aprovando-pedidos-de-compra
- https://ajuda.omie.com.br/pt-BR/articles/5386253-sugestao-de-compra
- https://ajuda.omie.com.br/pt-BR/articles/15264121-consultando-movimentacoes-de-reserva-de-estoque
- https://ajuda.omie.com.br/pt-BR/articles/4164410-cadastrando-locais-de-estoque
- https://ajuda.omie.com.br/pt-BR/articles/11962585-configurando-o-controle-de-lote-e-validade
- https://ajuda.omie.com.br/pt-BR/articles/5833962-curva-abc-como-emitir-entender-e-analisar
- https://ajuda.omie.com.br/pt-BR/articles/2199357-configurando-as-etapas-do-kanban-de-vendas
- https://ajuda.omie.com.br/pt-BR/articles/498882-faturando-pedidos-de-venda-em-lote
- https://ajuda.omie.com.br/pt-BR/articles/499170-gerando-uma-devolucao-de-venda
- https://ajuda.omie.com.br/pt-BR/articles/1742396-cadastrando-ou-excluindo-um-cenario-fiscal
- https://ajuda.omie.com.br/pt-BR/articles/2274214-conferindo-o-pedido-de-venda-e-visualizando-a-pre-nota-antes-de-emiti-la
- https://ajuda.omie.com.br/pt-BR/articles/9768942-acessando-o-historico-de-exclusoes
- https://ajuda.omie.com.br/pt-BR/articles/9169328-configurando-o-periodo-limite-para-registrar-lancamentos
- https://ajuda.omie.com.br/pt-BR/articles/9760272-configurando-o-omie-para-utilizacao-em-multiplas-empresas
- https://ajuda.omie.com.br/pt-BR/articles/5409954-compartilhamento-de-cadastros-entre-aplicativos
- https://ajuda.omie.com.br/pt-BR/articles/4826048-configurando-grupos-de-acesso-e-perfis-de-usuarios-no-omie
- https://ajuda.omie.com.br/pt-BR/articles/9345398-acessando-o-consultor-de-seguranca
- https://ajuda.omie.com.br/pt-BR/articles/5412721-caracteristicas-e-recomendacoes-das-apis-do-omie
- https://ajuda.omie.com.br/pt-BR/articles/8112984-limites-de-consumo-da-api-do-omie
- https://ajuda.omie.com.br/pt-BR/articles/9565655-caracteristicas-e-recomendacoes-dos-webhooks
- https://ajuda.omie.com.br/pt-BR/articles/5412754-utilizando-os-webhooks-no-omie
- https://ajuda.omie.com.br/pt-BR/articles/17173343-conectando-o-omie-a-um-assistente-de-ia-mcp
- https://ajuda.omie.com.br/pt-BR/articles/8333437-setembro-2023 (novidades: app Wemov WMS na Store)

**Omie, outras páginas:**
- https://www.omie.com.br/precos/
- https://store.omie.com.br/apps/omie-hub
- https://store.omie.com.br/apps/mercado-livre
- https://developer.omie.com.br/service-list/
- TrackCash (só snippet de busca): https://store.omie.com.br/apps/trackcash

**Terceiros:**
- https://lojexa.com/blog/bling-ou-tiny (comparativo Bling x Tiny, lido)
- Reclame Aqui e Capterra: **só snippets de busca**, porque as páginas devolveram 403 (ver item c).

## (b) Diferenciais notáveis

### Bling (fortíssimo em e-commerce/ML)
1. **Financeiro de marketplace nativo.** Importa automaticamente as taxas de ML, Shopee e Amazon para o pedido, com fórmula documentada. A **Conferência de Recebíveis ML** audita o repasse real contra o esperado por pedido (status a liberar, retido, reembolso, contestação). Os dashboards de **margem de contribuição por pedido e por SKU** descontam frete do vendedor, custo, impostos e taxas.
2. **Expedição madura.** Checkout com bipagem e trava por usuário, **checkout por produto único** (agrupa pedidos de 1 item), automações por usuário ao fim do checkout (gerar NF → SEFAZ → etiqueta → remessa), etiqueta ZPL, DANFE simplificado + etiqueta e **agrupamento de coleta ML** (etiqueta única por lote).
3. **Profundidade no ML.** Full com importação das NFs do ML (Faturador), transferências Full e inventário Full. Multi-origem por depósito com capacidade de despacho. Modelo UP. Kits de anúncios. PxQ (atacado). Central de ofertas com co-participação. Catálogo. Reclamações e devoluções (beta). Perguntas e mensagens. NF emitida só após a etiqueta ser liberada. Gestão de anúncios com importação de anúncios de terceiros e clonagem entre contas.
4. **Workflow configurável.** O gerenciador de transições entre situações dispara estoque, contas, NF e remessa, e é exposto na API.
5. **API REST moderna** (OAuth2, OpenAPI público, webhooks com HMAC) e **MCP**.
6. **Estoque:** check-in de recebimento com pendências, conferência de inventário por bip, lotes completos, **multiempresas com estoque somado** entre CNPJs.

### Omie (fortíssimo em gestão/financeiro/contábil)
1. **Financeiro e contábil:** **departamentos/centros de custo e projetos** com rateio, DRE com opção de CMC, fluxos de caixa variados, **Open Finance** e extrato via API de vários bancos, carteiras virtuais (Mercado Pago) por OFX, CNAB e DDA, conta digital integrada, Painel do Contador e SPED/ECD.
2. **Compras com governança:** requisição → pedido → aprovação por permissão (com e-mail) → kanban. **Agente de NF-e + e-mail de NF-e** para captura automática. Sugestão de compra por **ponto de pedido** (mínimo + consumo × lead time).
3. **Custo contábil (CMC)** recalculado a cada entrada, com CMV/CPV e Kardex. **Lote com FEFO/FIFO/LIFO automático.**
4. **Governança:** grupos de acesso copiáveis, **histórico de exclusões**, **período limite de lançamentos** (trava de competência), Consultor de Segurança, usuários ilimitados.
5. **Cenário fiscal por NCM e por perfil de cliente no Hub**, cotação de frete com **cálculo automático de embalagem**.
6. **MCP oficial** já em produção (ChatGPT/Claude), respeitando as permissões do usuário.

## (c) Dores e limitações relatadas

> As dores de usuário (marcadas **[usuário]**) vêm de **snippets de busca** do Reclame Aqui, Capterra e reviews, pois as páginas não puderam ser lidas (403). Trate como sinal, não como fato verificado. As limitações **[doc]** estão na documentação oficial.

### Bling
- **[doc]** O preço escala por **pedidos importados de marketplace/API**: média móvel de 3 meses, com **upgrade automático** após um "mês de adequação" e downgrade manual. Cobalto 200 pedidos/mês, Titânio F1 500 / F2 2.000 / F3 5.000, Diamante 10.000. Desde abr/2026 os pedidos via API também contam.
- **[doc]** Limite de usuários por plano (5 / 50 / 100) e **cota de armazenamento em MB** (60 MB de dados no Cobalto).
- **[doc]** O Faturador ML (dados fiscais para o ML emitir no Full) funciona **só em SP** e não aceita kit.
- **[doc]** NF do Full exige 4 casas decimais e nome de natureza idêntico ao do ML. O Bling **não gerencia o estoque do Full**.
- **[doc]** Taxas automáticas só para ML, Shopee e Amazon. Venda ML não paga importada manualmente vem **sem taxa** (é preciso excluir e reimportar). Não permite baixa parcial com taxa.
- **[doc]** Duas integrações logadas na mesma conta ML **duplicam pedidos**.
- **[doc]** Várias features ML estão em beta ou liberação gradual (reclamações, kits, multi-origem, check-in, agrupamento de coleta, MCP).
- **[doc]** Sem centro de custo e sem Open Finance (só OFX). Inferência por ausência na documentação.
- **[usuário]** Lentidão e instabilidade recorrentes ("falhas quase toda segunda-feira"), queda de qualidade "nos últimos 2 anos", suporte lento em picos, aumentos de mensalidade vistos como abusivos e upgrade automático de plano. Reclame Aqui com cerca de 618 reclamações e 99,8% respondidas, segundo snippet.

### Omie
- **[doc]** O hub é **produto separado e pago à parte** (exceto no Multivarejo), com login próprio, e a integração ML/Full **só se finaliza com o suporte**: dependência operacional.
- **[doc]** Compartilhamento entre CNPJs **não sincroniza estoque nem preço**.
- **[doc]** API **SOAP/JSON-RPC via POST**, sem REST nem GET. Rate limit restritivo por método (240/min, 4 simultâneas) e bloqueio de consulta repetida do mesmo ID em 60 s. Uma falha de webhook **bloqueia a fila inteira** do endpoint.
- **[doc / inferência por ausência]** Não encontrei: checkout/bipagem de expedição nativo (só WMS de terceiros na Store), importação de taxas do ML no pedido, conciliação de repasse ML nativa (há o TrackCash, de terceiro), perguntas e reclamações ML no Hub, multi-origem ML.
- **[usuário]** Integrações com Shopee e Nuvemshop instáveis (etiqueta não funcionando), **estoque não confiável**, "o Hub puxa de um único depósito para todos os canais" (conflita em parte com o artigo do Full, que cria locais separados), atualização de preço manual via Excel lenta (16 mil itens em 2–3 dias), relatórios inconsistentes, bugs que duram meses e suporte superficial.

## (d) O que não foi possível confirmar

1. **Comparação direta com Tiny/Olist.** Os ★ são hipóteses e precisam ser checados contra a pesquisa do Tiny.
2. **Valor e preços exatos atuais.** Bling: o site mostra Cobalto R$60, Titânio a partir de R$96 (com promoção) e Diamante R$357,50 (promo), enquanto o artigo de abr/2026 lista Titânio F1 R$120 e F2 R$200. Os valores divergem por promoção ou ciclo. Omie: "a partir de R$309" (ERP) e R$419 (Multivarejo), com preço por faixa de faturamento não detalhada.
3. **Omie.Hub e Flex.** Não confirmei se o Omie.Hub gera etiqueta Flex, trata `invoice_pending`/drop-off, importa as NFs emitidas pelo ML no Full nem quantas contas ML o plano permite de fato.
4. **Profundidade da devolução ML em ambos.** Não confirmei se a NF de entrada de devolução é gerada automaticamente quando o ML conclui a devolução, nem o tratamento do estoque devolvido ao Full.
5. **Inventário rotativo/cíclico, endereçamento WMS e curva de giro por depósito:** não documentados como features próprias.
6. **Desempenho real das sincronizações** (latência de estoque e de preço). O Bling tem artigo "Tempo de sincronização", não lido.
7. **Conteúdo integral** das reclamações (Reclame Aqui e Capterra bloquearam o acesso, 403).
8. **Detalhes de UI** (telas, campos exatos de modais). A documentação descreve fluxos, mas as imagens interativas não foram analisadas.
9. **Lista completa das 80+ automações do Bling.** Só as 9 principais e as do checkout e das transições foram documentadas.
10. **Eventos exatos de webhook do Omie** (a lista aparece só dentro do portal developer logado).
