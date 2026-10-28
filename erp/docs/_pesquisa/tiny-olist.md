# Olist ERP (ex-Tiny ERP): mapa completo de funcionalidades

> Pesquisa feita em 16/10/2026 para o projeto ERP Pinese (substituir o Olist). Fase de pesquisa: este documento não tem código.
> Fontes principais: Central de Ajuda (ajuda.olist.com, que substituiu ajuda.tiny.com.br), páginas comerciais (olist.com/sistema-erp, /planos, /hub-de-integracao), docs da API v2/v3 (api-docs.erp.olist.com, tiny.com.br/api-docs), Reclame Aqui (só pelos trechos da busca, porque a página bloqueia leitura direta) e reviews de terceiros. Também cruzei com os mapeamentos via REA da conta do dono (`olist-menu-rea.md` e `olist-conta-rea.md`, nesta pasta).

## Legenda

- **[F Sx]**: fato confirmado na fonte Sx (lista de fontes no item (a)).
- **[R1]/[R2]**: confirmado na conta real do dono via REA (R1 = `olist-menu-rea.md`, R2 = `olist-conta-rea.md`).
- **[I]**: inferência minha (não está escrito na fonte).
- **Complexidade** (Baixa/Média/Alta): estimativa **minha** [I] do esforço para reconstruir num ERP próprio (Next.js + Supabase). "Alta" vem com o motivo.
- **Dependência de marketplace**: Nenhuma / Parcial / Total, e de qual canal. "ML" = Mercado Livre.

## Planos (modelo de preços de 2026)

Planos atuais: **Avance** (R$ 66/mês) · **Construa** (R$ 177) · **Impulsione** (R$ 390) · **Domine** (R$ 948) · **Protagonize** (R$ 5.584 ou sob consulta) [F S1, S2]. Os planos anteriores (Começar / Crescer / Evoluir / Potencializar) e os ainda mais antigos (Free / Teen / Premium / Profissional / Essencial / Grande) ainda aparecem em artigos da ajuda [F S35, S51, S68]. A correspondência que uso a seguir é Começar≈Avance, Crescer≈Construa, Evoluir≈Impulsione, Potencializar≈Domine [I].

| Limite | Avance | Construa | Impulsione | Domine | Protagonize |
|---|---|---|---|---|---|
| Usuários | ilimitado | ilimitado | ilimitado | ilimitado | ilimitado |
| Armazenamento de dados | 60 MB | 200 MB | 400 MB | 2 GB | personalizado |
| Anexos/imagens | 12 GB | 12 GB | 12 GB | 12 GB | — |
| Anúncios | 2.000 | 5.000 | 10.000 | 20.000 | personalizado |
| Pedidos/mês | ilimitado | ilimitado | ilimitado | ilimitado | ilimitado |
| Sincronizações de estoque/mês | 150 mil | 300 mil | 600 mil | 900 mil | personalizado |
| API (requisições/min) | 30 | 60 | 120 | personalizado | personalizado |
| Créditos de IA (Lis)/mês | 10 | 20 | 30 | 40 | 50/personalizado |
| **OMS (importação automática de pedidos)** | **não** | sim | sim | sim | sim |

Fonte: [F S1, S2]. Ao bater o limite de armazenamento, **as funções de escrita (novos pedidos, emissão de NF, cadastros) ficam suspensas**, e só leitura e exportação continuam funcionando [F S2]. Há pacotes extras de armazenamento de R$ 34 (50 MB) a R$ 391 (1 GB) [F S2]. Uma mesma linha da página de planos traz 30 req/min para o Avance e "API de integração" só a partir do Construa [F S1]. A doc da API v2 dá 0 req/min para o "Começar" [F S35]. As duas informações se contradizem.

---

## 1. Cadastros

| Feature | O que faz | Plano/observação | Complexidade | Dep. marketplace |
|---|---|---|---|---|
| Produto simples | Cadastro de um item único com SKU, GTIN, preço, preço promocional, unidade, peso, dimensões e estoque inicial [F S13, R1]. | Todos | Baixa | Nenhuma |
| Produto com variações (grade) | O produto pai agrupa variações (cor, tamanho, voltagem, aroma…). Só as variações têm estoque. Grades padrão ficam em Configurações > Cadastros [F S78, R2]. | Depende da extensão "Variações" [F S78] | Média | Nenhuma (mas é mapeada para as variações do ML) |
| Ajustar/mover variações | Move uma variação de um grupo para outro e transforma variação em simples e vice-versa [F S75, S78]. | — | Média | Nenhuma |
| Kit | Conjunto de componentes. A venda do kit baixa o estoque de cada componente [F S13, S14]. | Importação de kits por planilha própria [F S14] | Média | Nenhuma |
| Kit com variações | Cada variação do produto pai pode ser um kit com composição própria. O marketplace só vê o pai com as variações, nunca os componentes [F S74]. | — | Média | Parcial (ML recebe só o "produto vendável") |
| Fabricado / Matéria-prima | Tipo "Fabricado" tem estrutura (BOM) na aba Produção. "Matéria-prima" é insumo [F S13, S44]. | Controle de Produção: Construa+ [F S1] | Média | Nenhuma |
| Custo de kit | Configuração: custo do kit = última compra dos componentes ou custo médio (aba Custos) [F S15]. | Custo médio exige a extensão DRE [F S15] | Média | Nenhuma |
| Código SKU automático | SKU manual, sequencial automático ou sequencial EAN-12 [F S15]. | — | Baixa | Nenhuma |
| GTIN/EAN | Campo GTIN, validado [F S13, S38]. É usado para relacionar anúncios e para bipagem [F S21]. | — | Baixa | Parcial (ML exige GTIN ou exceção) |
| Casas decimais | Quantidade e preço com 0 a 4 casas [F S15]. | — | Baixa | Nenhuma |
| Dados fiscais do produto | Origem ICMS (0–8), NCM, CEST, unidade tributável, Tipo do item SPED, EX TIPI, campos ANVISA [F S13, R1]. | Tipo do item SPED: Impulsione+ [F S13] | Média | Nenhuma |
| NCM padrão / sugestão de NCM | NCM e origem padrão para novos produtos, mais uma sugestão automática de NCM (pela Lis/IA) [F S8 cadastros, S15, R1]. | — | Média (a sugestão por IA é opcional) | Nenhuma |
| Edição de CEST em lote por NCM | Preenche o CEST em massa a partir do NCM, opcionalmente só nos campos vazios [F S75]. | — | Baixa | Nenhuma |
| Peso/dimensões/embalagem | Peso líquido e bruto, nº de volumes, tipo de embalagem (caixa/envelope/rolo), cadastro de embalagens com medidas e L×A×C [F S13, R1]. | Menu Cadastros > Embalagens [R1] | Baixa | Parcial (frete ML/Flex usa as dimensões) |
| Cálculo automático de peso e volume na NF | Soma o peso e o volume dos itens nas notas [F S8 cadastros, S15]. | — | Baixa | Nenhuma |
| Estoque mín./máx., localização, sob encomenda, dias de preparação | Parâmetros de controle por produto. "Sob encomenda" deixa o estoque virtual [F S13]. | — | Baixa | Parcial (dias de preparação e sob encomenda vão para o ML) |
| Controle de lotes/validade por produto | Flag "Controlar lotes" [F S13]. | Construa+ (na tela de produtos) e "todos" (na página de planos) [F S1, S13] | Média | Nenhuma |
| Categorias de produtos | Árvore de categorias, também exposta na API [F S34, R1]. | — | Baixa | Parcial (mapeada para as categorias MLB) |
| Marcas | Cadastro de marcas (API v3: criar/listar/atualizar) [F S34]. | — | Baixa | Nenhuma |
| Tags e grupos de tags | Marcadores para filtrar e montar relatórios [F S13, S34]. | — | Baixa | Nenhuma |
| Tabela de medidas | Para vestuário. Pode ser criada ou vinculada às tabelas do marketplace [F S7, S13]. | — | Média | Parcial (ML) |
| Descrição complementar / SEO | Texto longo do produto e campos de SEO [F S13]. | — | Baixa | Parcial |
| Imagens e anexos | Fotos e anexos do produto (API: CRUD de anexos). No plano, as imagens contam no limite de 12 GB [F S1, S13, S34]. | — | Média (storage + resize) | Parcial |
| Ficha técnica (atributos universais) | Atributos genéricos que preenchem automaticamente os anúncios de todos os canais. Os atributos específicos de cada canal vão no anúncio [F S13]. | — | **Alta**: modelo de atributos por categoria de cada marketplace | Parcial (ML/Shopee) |
| Cadastro de atributos + sugestão por categoria | Configurações > Cadastros > Atributos, com mapeamento manual e sugestão automática por categoria ML [F S4]. | — | Alta (depende da API de atributos do ML) | Total (ML) |
| Aba Anúncios no produto | Lista os anúncios vinculados ao produto (canal, ID, título, histórico de sincronizações) [F S13]. | — | Média | Total (por canal) |
| Aba Preços (listas de preços) | Preço por lista (atacado, varejo, canal) [F S13]. | — | Baixa | Nenhuma |
| Aba Custos (histórico de custo) | Histórico de custo médio/de compra por período. Precisa "Iniciar histórico de custos" [F S13, S30]. | Extensão DRE (Construa+) [F S30] | Média | Nenhuma |
| Markup | Multiplicador sobre o custo, global ou por produto, aplicado na formação de preço da entrada [F S13, S52]. | — | Baixa | Nenhuma |
| Linha de produto | Agrupador que define a alíquota de comissão do vendedor [F S13, S61]. | — | Baixa | Nenhuma |
| Garantia, unidade por caixa, "permitir inclusão nas vendas" | Campos complementares. "Permitir inclusão" restringe o produto a e-commerce [F S13]. | — | Baixa | Parcial (garantia vai ao ML) |
| Fornecedores do produto | Vínculo produto ↔ fornecedor (com código do fabricante, usado para casar o XML de entrada) [F S13, S28]. | — | Baixa | Nenhuma |
| 5 métodos de cadastro | Manual, planilha em massa, planilha de kits/fabricados, importação via integração (espelha anúncios) e criação via NF de entrada [F S14]. | — | Média | Parcial (importar do ML) |
| Cadastro automático | Cria sozinho os produtos citados em compra/venda/OS que ainda não existem [F S15]. | — | Baixa | Nenhuma |
| Ações em lote | Exportar (CSV/XLS até 500 linhas), exportar composição de kits e estrutura de fabricados, imprimir lista, editar em massa (unidade, CEST, origem, categoria, linha, localização, fornecedor, marca, garantia, tipo SPED) [F S75]. | — | Média | Nenhuma |
| Clonar / transformar / unificar / excluir lógico / recuperar | Clonar (com anexos e custos), transformar produto↔serviço e variação↔simples, unificar cadastros duplicados, excluir sem apagar e recuperar [F S75]. | — | Média | Nenhuma |
| Clientes e fornecedores (contatos) | Cadastro único com tipos (cliente, fornecedor, transportadora, vendedor, tipos customizados), endereço principal e de cobrança, telefones, pessoas de contato [F S76]. | — | Baixa | Nenhuma |
| Consulta CNPJ na Receita/SEFAZ | Preenche dados pelo CNPJ e UF [F S76]. | Exige certificado A1 [F S76] | Média | Nenhuma |
| Limite de crédito | Campo no contato que só administrador altera. Pode bloquear pedidos de clientes com contas em atraso [F S19, S76]. | Exige o módulo Contas a Receber | Baixa | Nenhuma |
| Importar/exportar contatos | XLS/CSV, até 500 por arquivo, unificação de duplicados, impressão de ficha [F S76]. | — | Baixa | Nenhuma |
| Cadastro automático de clientes via pedido de marketplace | Pedido importado cria o cliente, com opção de formatar o nome e completar o endereço pelo CEP [F S4, S10, R2]. | — | Média | Total (dados vêm do ML) |
| Vendedores | Cadastro de vendedores ligado a comissões, metas e permissões [F S8 cadastros, S60, S61]. | Gestão de vendedores: Impulsione+ [F S1] | Baixa | Nenhuma |
| Transportadoras | São um tipo de contato. Formas de envio configuráveis [F S76, S23]. | — | Baixa | Nenhuma |
| Listas de preços | Regra geral + exceções por produto, sem alterar o preço do cadastro. Podem ser vinculadas a uma integração [F S77, S34]. | API v3 cria/edita listas [F S34] | Média | Parcial |
| Intermediadores | Cadastro de intermediadores (ex.: ML CNPJ 03.361.252/0001-34) para a NT 2020.006 [F S10, S8 fiscal]. | — | Baixa | Parcial (todo marketplace) |
| Cadastro de serviços | Serviços para OS, contratos e NFS-e, com código LC 116 [F S8 serviços, S38]. | — | Baixa | Nenhuma |

## 2. Estoque

| Feature | O que faz | Plano/observação | Complexidade | Dep. marketplace |
|---|---|---|---|---|
| Controle de estoque | Lista com SKU, preço, **custo médio**, físico, **reservado**, disponível, unidade e localização [F S16, R1]. | Todos | Média | Nenhuma |
| Lançamentos manuais | Entrada, Saída e **Balanço** (substitui o saldo), com histórico por produto [F S16]. | Editar/excluir movimentos exige a extensão "Edição de Registros de Estoque" [F S16] | Baixa | Nenhuma |
| Gatilho de baixa (saídas) | Manual, ao autorizar NF, ao salvar NF, ao salvar pedido ou ao marcar como enviado. Global, mas pode ser sobrescrito por integração [F S17, S10, R2]. | — | Média | Parcial (no Full é "ao marcar enviado") |
| Gatilho de entrada | Manual, ao autorizar NF, ao salvar NF, ao salvar OC ou ao finalizar a conferência de compra [F S17]. | — | Média | Nenhuma |
| Estorno automático | Estorna o estoque ao cancelar NF ou pedido [F S17]. | — | Média | Nenhuma |
| Estoque negativo | Permitir, bloquear em qualquer depósito ou bloquear só no depósito selecionado. Também bloqueia NF com saldo ≤ 0 [F S17]. | — | Média | Nenhuma |
| **Reserva de estoque** | O pedido criado reserva; o lançamento baixa o físico e libera a reserva; o cancelamento devolve. Aceita reserva manual. Saldo disponível = físico − reservado [F S41]. | Extensão "Reserva de estoque" [F S41] | **Alta**: concorrência e consistência com vários canais e situações | Parcial (alimenta o "saldo disponível" enviado ao ML) |
| Bloquear pedido sem estoque disponível | Impede criar pedido manual/PDV sem saldo. Não vale para API, app e e-commerce [F S19, S41]. | Exige Reserva [F S19] | Baixa | Nenhuma |
| **Múltiplos depósitos** | Extensão "Depósitos de Estoque". O tipo **Próprio** pode "desconsiderar saldo" (avarias). O tipo **Exclusivo canal de venda** serve ao Full [F S42]. | Construa+ [F S42] | Média | Parcial |
| Transferência entre depósitos | Origem → destino, vários itens por vez [F S16, S42]. | — | Baixa | Nenhuma |
| Transferência automática por CNPJ | Ao identificar NF com o CNPJ do CD configurado, transfere para o depósito do Full [F S3, S42]. | — | Média | Total (ML Full) |
| Reserva individual por depósito | Reserva controlada por depósito de origem da transação [F S42]. | — | Média | Nenhuma |
| Estoque enviado ao canal por depósito | Envia o saldo de um depósito específico ou de "todos os próprios". Tabela multi-depósito ERP ↔ e-commerce [F S42, R2]. | — | Média | Total (por canal) |
| Estoque de segurança por integração | Zera o anúncio quando o saldo chega à margem. Para variações, considera a soma [F S4, S10, R2]. | Vale igual para todos os anúncios da integração [F S10] | Baixa | Total |
| Estoque Full separado | Depósito "ML Full" (também Amazon Full e Shopee Full), alimentado por NFs de transferência e devolução. O ML controla o saldo [F S3, R2]. | — | Alta: reconciliar com o saldo do ML e importar documentos fiscais | Total (ML Full) |
| Inventário | Gerar contagem (imprimir/Excel/na tela), importar planilha ou importar arquivo de coletor (GTIN/SKU, 2 MB), com validação igual/divergente/não encontrado [F S16, S72]. | Edição de quantidades importadas é configurável [F S17] | Média | Nenhuma |
| Custo médio | Recalculado na formação de preço das entradas (NF/OC): (saldo × custo médio anterior + qtd comprada × custo atual) / saldo total [F S30, S52]. | Extensão DRE para o histórico [F S30] | Média | Nenhuma |
| Lotes e validades (FEFO) | Lote/validade na entrada. Saída automática pelo vencimento mais próximo (FEFO) ou manual. Lote vai ao XML da NF. Cores por status. Relatório de lotes [F S43]. | Todos ou Construa+ (as fontes divergem) | Média | Nenhuma |
| Alerta de estoque mínimo | E-mail quando o produto chega ao mínimo [F S17]. Relatório "abaixo do mínimo" [R1]. | — | Baixa | Nenhuma |
| Quantidade padrão sob encomenda | Quantidade exibida no canal para itens sob encomenda [F S17]. | — | Baixa | Parcial |
| Log de movimentação (auditoria) | Usuário, origem (manual, planilha, inventário, integração, automático, API, webhook) e data/hora. Não editável. Consulta também pela Lis [F S40]. Exposto na API v3 [F S34]. | Permissão específica [F S40] | Média | Nenhuma |
| Giro de estoque | Vendas ÷ estoque médio (período até 550 dias). Classifica normal (>1), baixo giro (0–1) e parado (=0) [F S93, R1]. | — | Baixa | Nenhuma |
| Relatórios de estoque | Saldos em data passada, entradas e saídas, visão financeira, maior circulação, sem movimentação, abaixo do mínimo, movimentação por produto, estoque multiempresa [F S65, R1]. | — | Média (saldo retroativo exige ledger) | Nenhuma |
| Estoque compartilhado multiempresa | Soma depósitos "Próprio" de outras empresas do grupo no saldo enviado ao canal [F S42, S80]. | Multiempresa: Construa+ [F S1, S80] | Alta: sincronização entre contas/CNPJs | Parcial |
| FCI | Item de menu "FCI" em Suprimentos (Ficha de Conteúdo de Importação) [R1]. | Detalhe não lido [I] | Média | Nenhuma |

## 3. Vendas e pedidos

| Feature | O que faz | Plano/observação | Complexidade | Dep. marketplace |
|---|---|---|---|---|
| Pedido de venda manual | Cliente, itens com tributação por item (CFOP, NCM, alíquotas), totais, frete, desconto, condição de pagamento (vários meios, Pix), transportador, depósito, observações, marcadores [F S20]. | — | Média | Nenhuma |
| **OMS: pedidos do e-commerce** | Importa automaticamente os pedidos dos canais ("Pedidos no e-commerce"). Ativação "Automático/Manual/Não importar" por integração [F S1, S10, R1]. | **Construa+** [F S1] | **Alta**: polling/webhook de vários canais, idempotência, packs | Total |
| Situações do pedido | Em aberto, Aprovado, Preparando envio, Faturado, Pronto para envio, Enviado, Entregue, Não entregue, Dados incompletos, Em devolução (só ML), Cancelado [F S18, R2]. | — | Média | Parcial |
| Mapeamento de situações por canal | Tabela código do canal → situação ERP (ML: pending, paid, handling, ready_to_ship, shipped, delivered, not_delivered, cancelled). Pode permitir retroceder a situação [R2, F S4]. | No Full, quase tudo vira "Enviado" [R2] | Média | Total |
| Ícones de fluxo E/N/C | O pedido mostra se o estoque foi lançado, se a NF foi emitida e se as contas foram lançadas [F S18]. | — | Baixa | Nenhuma |
| Aprovação automática | Muda de "em aberto" para "aprovado" quando o pagamento é identificado [F S19]. | — | Baixa | Parcial |
| Lançar/estornar estoque e contas | Individual ou em lote [F S20]. | — | Média | Nenhuma |
| Faturamento parcial | Divide o pedido em vários faturamentos [F S20]. | — | Média | Nenhuma |
| Gerar NF / NFC-e / NFS-e do pedido | Individual, em lote ou automaticamente ao aprovar [F S20, S68]. | — | Média | Nenhuma |
| Clonar, editar alguns dados, excluir | Pedidos com estoque ou contas lançados só podem ser editados parcialmente [F S20]. | — | Baixa | Nenhuma |
| Marcadores | Tags coloridas em pedidos, em lote. Repassadas à NF. Marcador automático de 1ª venda [F S19, S20]. | — | Baixa | Parcial (ex.: "coleta rápida", "etiqueta com data programada") |
| Configurações de pedido | ~30 opções: vendedor obrigatório, desconto máximo do vendedor/gerente, bloqueio por inadimplência, forma de pagamento obrigatória, numeração, volumes padrão, depósito de devoluções, observação padrão, impressão (código de barras, imagens, impostos) [F S19]. | — | Média | Nenhuma |
| Impressões | Pedido (layout customizável), recibo, carnê, etiquetas, pedido para produção, detalhado [F S20]. | — | Média | Nenhuma |
| Compartilhar documento | E-mail, link ou WhatsApp, com boleto anexo [F S20, S8 minha-conta]. | — | Baixa | Nenhuma |
| Exportar pedidos | XLS/CSV [F S20]. | — | Baixa | Nenhuma |
| Agrupar pedidos de carrinho (pack) em 1 NF | Itens de um carrinho ML vão para uma única nota [F snippet de busca S99]. | Não li o artigo oficial | Média | Total (ML packs) |
| Propostas comerciais (orçamentos) | Orçamento com situações e modelos, convertido em pedido [F S34, R1]. | — | Média | Nenhuma |
| Enviar pedidos para outra empresa | Pedido repassado a outro CNPJ do grupo [F S20]. | Multiempresa | Alta (multi-tenant entre contas) | Nenhuma |
| **Separação** | Abas Aguardando, Em separação, Separadas e Embaladas. Separação por lista (agrupada por pedido ou produto, ordenada por localização/SKU), por pedido ou por produto. Filtros por forma de envio e quantidade de itens [F S21]. | Separação/embalagem em lote: Impulsione+ [F S1] | Média | Nenhuma |
| Gatilho de envio para separação | Manual, ao salvar, ao aprovar, ao salvar NF ou ao autorizar NF [F S22]. | — | Baixa | Nenhuma |
| **Embalagem/checkout com bipagem** | Lê código de barras (SKU, GTIN-13, nº de série), marca "item lido", confere, atualiza volumes e imprime etiqueta/DANFE ao finalizar [F S21, S22]. | Impulsione+ [F S1, S18] | Média | Parcial (etiqueta do ML) |
| Impressão automática ao finalizar embalagem | DANFE (padrão, simplificado ou etiqueta), etiqueta de envio ou ambos [F S22]. | Impulsione+ [F S1] | Média (impressão térmica: PrintNode) | Parcial |
| Lock de separação/embalagem | Pedido preso ao operador. Gerente libera. A listagem mostra operador e tempo [F S73]. | — | Baixa | Nenhuma |
| Autorizar NF na separação/embalagem | Opções para autorizar a NF durante a separação ou ao concluir a embalagem [F S22]. | — | Baixa | Nenhuma |
| Relatório de desempenho da separação | Relatório de produtividade [R1]. | — | Baixa | Nenhuma |
| **Expedição** | Agrupamentos (coletas) por forma de envio, com edição do endereço só na expedição, objetos avulsos, conferência por bipagem, romaneio/ordem de coleta, DANFE em lote, relatório [F S23, S24]. | Controle de expedição: todos [F S1] | Média | Parcial |
| Gatilho de envio para expedição | Manual, ao salvar, ao aprovar, ao salvar NF ou ao autorizar NF [F S24]. | — | Baixa | Nenhuma |
| Etiquetas | 4 ou 2 por página, bobina, ZPL térmica, logo P&B, layouts por transportadora [F S23]. Etiqueta ML: PDF ou ZPL [F S47]. | — | Média | Parcial (a etiqueta ML vem pronta da API ML) |
| Etiqueta ML com DANFE unificada (beta) | Junta a DANFE simplificada na etiqueta PDF 10×15. Não funciona com ZPL [F S47, R2]. | Exige impressora térmica + PDF no painel ML | Média | Total (ML) |
| Marcar enviado/entregue + rastreio | Envia o rastreio por e-mail e ao canal, e marca entregue no canal [F S23, S24, S4]. | — | Baixa | Parcial |
| Atualizar situação de envio em lote | Consulta transportadoras (Correios, Frenet, Jadlog, Kangu, Mandaê, Melhor Envio, Total Express…) e classifica em enviado, em trânsito, entregue, atrasado, não entregue ou atenção [F S91]. | Intervalo máximo de 30 dias | Média | Nenhuma |
| Devoluções de venda | Parte do pedido: devolver itens com reembolso sem crédito, em dinheiro, em vale-troca ou como conta a pagar. Gera NF de devolução, lança estoque (depósito de devoluções), debita comissão e permite logística reversa Correios [F S57, S19]. | Impulsione+ [F S1] ou Construa+ [F S57] | Média | Parcial |
| Vale-troca | Gera código, gerencia saldo e emite novo vale com o restante [F S57]. | — | Baixa | Nenhuma |
| **PDV (frente de caixa)** | Venda balcão, pré-venda, abertura/fechamento de caixa, crediário, repasse de juros, devolução/vale-troca, maquininhas integradas (Stone POS etc.), NFC-e/SAT/PAF-SC [F S98 pdv, S1]. | Todos. SAT e PAF: Impulsione+ [F S1] | Alta: offline, fiscal de consumidor e TEF | Nenhuma |
| Desconto com senha de gerente | Libera desconto acima do limite com código do gerente. Relatório [F S19, S39, S59]. | Construa+ [F S1] | Baixa | Nenhuma |
| CRM | Assuntos/negociações com ações, anotações, estrela, marcadores e funil de até 10 estágios (ex.: Prospecção → Fechado) [F S79, S34]. | Impulsione+ [F S1] | Média | Nenhuma |
| Comissões de vendedores | Alíquota por linha de produto (fixa ou por desconto). Liberação no faturamento, por parcela paga, após a 1ª parcela ou 50/50. Base = total dos produtos. Pagamento lançado no caixa [F S61]. | Impulsione+ (gestão de vendedores) | Média | Nenhuma |
| Metas e performance de vendas | Meta mensal por vendedor, clientes atendidos/com venda, vendas × faturamento, comparação com o ano anterior [F S60]. | Impulsione+ [F S60] | Baixa | Nenhuma |
| Painel de Automações | Item de menu em Vendas [R1]. | Conteúdo não lido [I] | — | — |
| Crédito da Olist / antecipação | Antecipação de recebíveis, no menu [R1, S1]. | Serviço financeiro Olist | Fora do escopo [I] | Nenhuma |

## 4. Fiscal

| Feature | O que faz | Plano/observação | Complexidade | Dep. marketplace |
|---|---|---|---|---|
| NF-e modelo 55 (saída) | Emissão a partir do pedido ou manual: venda, nota mista (produto + serviço), exportação, devolução de compra [F S8 fiscal, S62]. | Todos [F S1] | **Alta**: motor tributário, schema SEFAZ por UF, contingência | Nenhuma |
| NF-e de entrada (emissão própria) | Nota de importação, devolução de venda, entrada própria [F S8 fiscal]. | — | Alta (mesmo motivo) | Nenhuma |
| NFC-e modelo 65 | Nota de consumidor com ID/CSC, devolução de NFC-e e PAF-NFCe para SC [F S8 fiscal, S1]. | PAF-SC: Impulsione+ | Alta | Nenhuma |
| SAT-CFe (SP) | Cupom fiscal SAT [F S8 fiscal, S1]. | Impulsione+ | Alta (hardware) | Nenhuma |
| NFS-e | Nota de serviço por prefeitura e no padrão nacional (com IBS/CBS) [F S8 fiscal, S8 serviços]. | 20 (Construa) e 200 (Impulsione/Domine) por mês [F S1] | Alta: cada município tem um layout | Nenhuma |
| CT-e | Só **registro de entrada** (não emite CT-e) [F S62]. Sintegra exporta o registro 70 [F S66]. | — | Média | Nenhuma |
| MDF-e | **Não encontrado**. Um comparativo de terceiro diz que não há suporte [F S97]. | Não confirmado | — | — |
| DC-e (Declaração de Conteúdo Eletrônica) | Obrigatória desde 06/04/2026 para envio sem NF. Emitida via Correios, Melhor Envio, Frenet ou SuperFrete. Imprime o DACE [F S90]. | — | Média | Nenhuma |
| Certificado digital A1 | Upload do .pfx, armazenado nos servidores Olist. Necessário para emitir, consultar CNPJ e usar o buscador de notas [F S8 fiscal, S68]. A3 aparece na página comercial [F S69]. | — | Média (guardar com segurança) | Nenhuma |
| Credenciamento SEFAZ / ambiente | Produção ou homologação, com checklist de emissão [F S8 fiscal, S38]. | — | Baixa | Nenhuma |
| Natureza de operação (saída e entrada) | Abas Simples/CSOSN, ICMS/CST, IPI, ISSQN, PIS, COFINS. CFOP com curinga "?" (5/6/7). Considerar no DRE, lançar/reservar estoque, enviar à expedição, produtor rural/Funrural, ZFM, redução de base ST [F S63]. | — | **Alta**: é o coração do motor fiscal | Nenhuma |
| Regras de exceção na natureza | Exceções por UF, produto, origem e NCM. Origem tem precedência [F S63, S67]. | — | Alta | Nenhuma |
| ICMS-ST, DIFAL, FCP, partilha | Cálculo de ST, DIFAL para contribuinte e não contribuinte, FCP. Relatório de partilha de ICMS [F S8 fiscal, S38, R1]. | — | Alta | Nenhuma |
| Benefícios fiscais / desoneração | Configuração de benefícios de ICMS [F S8 fiscal]. | — | Alta | Nenhuma |
| Reforma Tributária (IBS/CBS) | Aba "Tributação (RTC)": cClassTrib por produto, cálculo de IBS/CBS em NF-e/NFC-e/NFS-e, exceções por NCM [F S8 fiscal, S38, R1]. | 2026 | Alta (regras mudando) | Nenhuma |
| Alteração de regime tributário | Simples, Normal (Presumido/Real) e IVA [F S1, S8 fiscal]. | — | Média | Nenhuma |
| Autorização automática de NF | Gera a NF ao aprovar o pedido e transmite sozinha. Pode ser limitada a canais específicos [F S68]. | Planos pagos | Média | Parcial |
| Emissão em lote | NF em lote a partir dos pedidos [F S20, S6]. | — | Média | Nenhuma |
| Cancelamento com estorno | Cancela a NF e estorna estoque e contas automaticamente [F S8 fiscal]. | — | Média | Nenhuma |
| Contingência | Emissão em contingência [F S8 fiscal]. | — | Alta | Nenhuma |
| Carta de correção | Exportada junto com os XMLs [F S65]. Não confirmei a tela de emissão. | — | Média | Nenhuma |
| Inutilização de numeração | Relatório de numeração de notas [F S8 fiscal]. Não confirmei a tela de inutilização. | — | Baixa | Nenhuma |
| Correção de NF rejeitada com IA | Explica a rejeição (ex.: erro 213) e guia a correção [F S71]. | Liberação gradual | Média (opcional) | Nenhuma |
| DANFE | Padrão, simplificado, etiqueta, unificado com etiqueta ML [F S20, S22, S47]. | — | Média | Parcial |
| Envio da NF ao marketplace | Chave + XML (ou só mensagem, ou só XML) ao ML, automático ao autorizar ou manual [F S4, S10, R2]. | — | Média | Total (ML) |
| Envio de dados fiscais do produto ao ML | NCM, CSOSN, CEST, origem e GTIN por produto ou em lote, para o ML faturar Full/Coletas [F S3, S49]. | Simples: todos os campos são obrigatórios [F S3] | Média | Total (ML Full) |
| Mensagens fiscais ao ML Full | Até 30 mensagens. ISSQN e Simples não são enviadas [F S3]. | — | Baixa | Total (ML Full) |
| Modelo fiscal do Full | Self storage CAT 59/2018 (SP) ou modelo filial. Natureza de transferência, por exemplo "Remessa p/ depósito temporário" [F S10, R2]. | — | Alta (regras do Full por UF) | Total (ML Full) |
| Importação de NF emitidas pelo ML (Full) | Importa automaticamente NFs de venda, devolução e transferência emitidas no Full [F S3, R2]. | — | Média | Total (ML Full) |
| Notas de entrada por XML | Upload unitário (2 MB), ZIP em lote (5 MB) ou buscador SEFAZ [F S28]. | — | Média | Nenhuma |
| Buscador de notas (DF-e) + manifestação | Baixa as NFs emitidas contra o CNPJ (20/dia) e permite manifestar: ciência, confirmação, desconhecimento ou operação não realizada [F S28]. | Exige A1 | Média (Distribuição DF-e) | Nenhuma |
| Vínculo XML ↔ produto | Prioridade: EAN → fornecedor + código do fabricante + descrição → … → descrição. Sem correspondência, cria produto "Novo" [F S28]. | — | Média | Nenhuma |
| GNRE e guias | Gera guias GNRE (manual ou automática ao autorizar NF interestadual), cria a conta a pagar, paga pela Conta Digital, baixa ZIP por UF [F S70]. | Gratuitas por mês: 10 (Construa), 30 (Impulsione), 60 (Domine), 200 (Protagonize) [F S1]. Também DARF e IPVA [F S8 fiscal] | Média | Nenhuma |
| SPED Fiscal (com Bloco K) | Gera o SPED Fiscal com K200, K230, K235 e K280 [F S64]. | Impulsione+ [F S1] | **Alta**: layout extenso e validação PVA | Nenhuma |
| Sintegra | Exporta registros 10…, 70 (CT-e) e 74 (inventário) [F S66]. | — | Média | Nenhuma |
| Extração para a contabilidade | XMLs (com CC-e e canceladas, por UF) em XML, planilha ou PDF. Relatórios de saldo, entradas e saídas e visão financeira do estoque [F S65]. "Painel de Contadores Olist" existe (FAQ) [F S8 finanças]. | — | Baixa | Nenhuma |
| Relatórios fiscais | NF por operação, cliente e produto, consulta de itens de NF, evolução do faturamento, numeração de notas [F S8 fiscal, R1]. | — | Baixa | Nenhuma |
| Serviços tomados | Registro de NFS-e de serviços tomados em Suprimentos [F S8 serviços, R1]. | — | Baixa | Nenhuma |

## 5. Compras / Suprimentos

| Feature | O que faz | Plano/observação | Complexidade | Dep. marketplace |
|---|---|---|---|---|
| Ordens de compra | Fornecedor, itens (busca por descrição, SKU, GTIN ou código do fabricante), condição de pagamento, transportador, ICMS ST por item. Situações: Em aberto, Em andamento, Atendido, Cancelado [F S26]. | Gestão de compras: Impulsione+ [F S1] | Média | Nenhuma |
| Recebimento parcial | Recebe uma OC aos poucos [F S26]. | — | Média | Nenhuma |
| Lançar estoque/contas da OC | Individual ou em lote, com criação e vínculo de produtos e estorno [F S26]. | — | Média | Nenhuma |
| Gerar NF a partir da OC | Ex.: NF de entrada própria [F S26]. | — | Média | Nenhuma |
| Clonar, marcadores, compartilhar, importar/exportar planilha, PDF | Utilidades da OC [F S26]. | — | Baixa | Nenhuma |
| **Necessidades de compra (sugestão)** | Sugestão = média mensal (1–3 meses ou período livre) × cobertura (1–3 meses) − físico − entradas previstas (+ mínimo, + saídas previstas). Filtra por fornecedor, marca, depósito, tags e situações de venda. Gera uma OC por fornecedor [F S25]. | "Planejamento de compras": todos [F S1] | Média | Nenhuma |
| Conferência de compra | Check-in de volumes (bipa a chave de acesso) e de itens (bipa GTIN ou código do fornecedor). Registra problemas (divergência, avaria, não recebido). Pode lançar estoque ao finalizar [F S27]. | Impulsione+ [F S27] | Média | Nenhuma |
| Conferência cega | Oculta a quantidade esperada [F S38, terceiro]. | Não confirmado na ajuda oficial | Baixa | Nenhuma |
| Formação de preço na entrada | Custo = unitário + II + frete + seguro + ST + IPI + despesas − descontos − recuperáveis, mais DIFAL. Aplica markup e pode atualizar o preço nos canais [F S52]. | Não cobre notas de importação [F S52] | Média | Parcial (atualiza o ML) |
| Custo de reposição / última compra | Custo da última compra usado em kits, DRE e visão financeira [F S15, S30, R1]. | — | Baixa | Nenhuma |
| Relatórios de compra | NF de compra por operação, fornecedor, produto e evolução; ordens de compra; serviços tomados [R1]. | — | Baixa | Nenhuma |
| Envios Fulfillment (inbound) | Monta o envio ao CD do ML: escolhe integração e depósito, adiciona anúncios e quantidades, reserva estoque, gera planilha no layout ML, registra nº do envio e data de coleta, vincula NF de transferência, divide por CD. Abas Em aberto → Separação → Embalagem → Prontos → Finalizados [F S45, R1]. | — | **Alta**: fluxo misto ERP + painel ML, divisão por CD | Total (ML Full). Também Amazon FBA [F S8 suprimentos] |
| Produção (ordens de produção) | OP a partir da BOM do fabricado, cálculo de necessidade de materiais, baixa de matéria-prima, etapas, OP gerada a partir de pedidos [F S44]. | Construa+ [F S1] | Média | Nenhuma |
| Produtos sob encomenda | Fluxo de itens sem estoque físico [F S8 suprimentos, S1]. | Todos | Baixa | Parcial |

## 6. Financeiro

| Feature | O que faz | Plano/observação | Complexidade | Dep. marketplace |
|---|---|---|---|---|
| Contas a pagar | Fornecedor, vencimento, valor, categoria obrigatória, forma de pagamento, chave Pix. Ocorrência única, semanal, mensal, trimestral, semestral, anual ou parcelada. Pagamento parcial ou em lote (até 50), agrupamento por fornecedor, estorno, anexos de 2 MB, marcadores, competência [F S82]. | Construa+ [F S1] | Média | Nenhuma |
| Contas a receber | Mesmo modelo, com baixa, boletos, marcadores e "obter recebimentos" (API) [F S34, S31]. | Construa+ | Média | Parcial |
| Lançamento automático de contas | A receber: ao autorizar NF, salvar NF ou salvar pedido. A pagar: ao autorizar, salvar NF ou salvar OC. Serviços: por OS/NFS-e [F S32]. | — | Média | Nenhuma |
| Formas de recebimento com taxas | MDR %, tarifa fixa, comissão e prazo D+ por forma (cartão, Pix, boleto, marketplace). O contas a receber nasce com o valor líquido. Mapeadas para tPag/xPag na NF-e [F S37 terceiro, S8 PDV]. | — | Média | Parcial (taxa do ML) |
| Formas de pagamento | Cadastro de meios de pagamento [F S34]. | — | Baixa | Nenhuma |
| Categorias de receita/despesa | Plano de contas gerencial com flag "considerar no DRE" e "CMV" [F S30, S82]. | — | Baixa | Nenhuma |
| Centro de custo | **Não encontrado** na documentação [F S82]. | — | — | — |
| Caixa e bancos | Lançamentos (entrada/saída, competência, anexos, marcadores), contas financeiras e bancárias, transferências entre contas, recibo, exportação XLS/CSV [F S83]. | Construa+ | Baixa | Nenhuma |
| Conciliação bancária (OFX) | Importa OFX/XML de extrato, vincula movimentações a contas a pagar/receber (com baixa parcial, juros, desconto, taxa), transforma em lançamento de caixa, ignora ou estorna. Evita duplicados [F S81, S35 changelog 3.1.10, R1]. | Construa+ [F S1]. API desde set/2026 | Média | Nenhuma |
| Conciliação Mercado Pago (Lis) | Automação diária de IA: lê pagamentos do gateway MP (pack_ID/order_ID, taxa, líquido) e baixa os contas a receber correspondentes [F S36]. | **Exclui vendas diretas do marketplace ML** e não baixa taxas/estornos [F S36] | Alta: casar repasses do ML com pedidos, taxas e frete | Total (MP/ML) |
| Cobranças bancárias (CNAB) | Remessa e retorno para bancos, baixa automática, boletos via Pagar.me, envio por e-mail [F S31]. | Extensão "Financeiro Intermediário", Construa+ [F S31] | Alta (layouts CNAB por banco) | Nenhuma |
| Boletos e Pix (Conta Digital Olist) | Emissão gratuita de boletos (tarifa só quando liquidado), Pix QR com tarifa por plano, link de pagamento, maquininha no celular [F S1]. | Serviço financeiro Olist | Fora do escopo [I] | Nenhuma |
| Gateway financeiro / pagamento integrado | Integração com gateways de pagamento [F S8 finanças, S8 integrações]. | Construa+ | Média | Parcial |
| Configurações financeiras | Data de recebimento (pagamento ou crédito), baixa agrupada ou individual, tarifas no caixa, atualizar vencimentos ao emitir NF, categoria padrão de retorno [F S32]. | — | Baixa | Nenhuma |
| DRE | Por NF ou pedido, custo de compra ou custo médio, anual, agrupável por empresa. Receita − impostos (ICMS, IPI, ISS, PIS, COFINS, ST) − CMV − comissões − despesas por categoria [F S30]. | Extensão DRE, Construa+ [F S30] | Média | Nenhuma |
| Fluxo de caixa (projeção) | Relatório de projeção [F S85, R1]. Também no dashboard financeiro [F S86]. | — | Média | Nenhuma |
| Balancete | Relatório [F S8 finanças, R1]. | — | Média | Nenhuma |
| Relatórios financeiros | Entradas e saídas por categoria ou cliente, contas a pagar, recebimentos, contas a receber, financeiro de vendas (taxas, tarifas, meios) [R1, S20]. | — | Baixa | Nenhuma |
| Custos do e-commerce | Por pedido: comissão do integrador, comissão do marketplace, frete (pago pela empresa × total), taxas, custo do produto, total líquido. **Valores do ML importados automaticamente** [F S92]. | Impulsione+ [F S92]. Não inclui impostos nem Ads [F S92] | Média | Total (ML automático, outros canais manual) |
| Margem de contribuição | Relatório de lucro real por venda, canal e produto [F busca S92b, R1]. | Impulsione+ | Média | Parcial |
| Comissões a pagar | Pagamento de comissões lançado no caixa [F S61]. | — | Baixa | Nenhuma |

## 7. Marketplaces / hub, anúncios e precificação

| Feature | O que faz | Plano/observação | Complexidade | Dep. marketplace |
|---|---|---|---|---|
| Hub de integrações | 100 a 170+ integrações: ML, ML Fulfillment, Shopee, Amazon (+FBA), Magalu, Americanas, Casas Bahia, Madeira, TikTok Shop, Temu (via Hub), Shein, Leroy Merlin, Dafiti, plataformas (Nuvemshop, Shopify, Tray, VTEX, WooCommerce, Loja Integrada, Wix…) e hubs (HUB2B, AnyMarket, Plugg.To) [F S84, S8 integrações, S38]. | Marketplaces nativos: todos [F S1] | Alta por canal | Total |
| **Hub da Olist (nova camada ML/Temu)** | Nova camada de integração para o ML: ações em massa em segundo plano, pedidos e etiquetas "100% sincronizados", melhor suporte a variações e ao modelo **User Product** do ML, clonagem de anúncios entre integrações do mesmo canal, IA para títulos/descrições/fundo branco, tradução de erros [F S9, S10, S11]. | Recente (2025/26) | Alta | Total (ML/Temu) |
| Multicontas do mesmo canal | Cada conta ML é uma **instalação separada** da integração [F S10]. Pós-venda com várias contas ML [F S56]. | Hoje a conta do dono tem 1 só [R2] | Média | Total |
| Canais de venda por integração | Subdivide os pedidos de uma integração por canal (ex.: Mercado Shops) e define a categoria financeira por canal [F S4, R2]. | — | Baixa | Total |
| Gestão de anúncios | Lista de anúncios por canal (MLB, preço), importar (todos, novos ou um específico), editar (gerais, ficha técnica, preços), pausar/ativar (ML/Shopee, em lote), clonar até 20 [F S12, S8 cadastros]. | Limite de anúncios por plano [F S1] | **Alta**: ciclo de vida do anúncio e erros da API | Total |
| Relacionar anúncio ↔ produto (mapeamento SKU) | Por SKU, título ou GTIN. Sem correspondência: criar produto ou relacionar à mão [F S8 cadastros, S4]. | — | Média | Total |
| Publicar anúncio a partir do produto | "Enviar para e-commerce" → Hub. Regras padrão (envio, retirada, tipo, garantia), categoria + atributos obrigatórios, ficha técnica (a Lis preenche pelo GTIN/catálogo), anúncio próprio ou de catálogo, validação e painel de status [F S11, S4]. | Títulos de até 60 caracteres no ML [F S11] | Alta | Total |
| Anúncios de catálogo | Elegibilidade, vínculo ao catálogo, sincronização de catalogados. Anúncios de catálogo **não são importados** [F S4, S7]. | — | Alta | Total (ML) |
| Tipo de anúncio | Grátis, Clássico ou Premium (padrão por integração ou por anúncio). Escolher vários gera anúncios separados [F S4, S10, R2]. | — | Baixa | Total (ML) |
| Variações no ML / família | Mapeia tipos e opções de variação ERP ↔ ML. Preço por variação (PxV). Atualizar um anúncio processa a família inteira [F S12, R2]. | — | Alta | Total (ML) |
| Mapeamento de categorias | Categoria ERP ↔ categoria MLB [F S4, R2]. | ~34 mapeadas na conta [R2] | Média | Total |
| Compatibilidade de autopeças | Ficha de compatibilidades com sugestões do ML [F S4, S7]. | — | Alta | Total (ML) |
| Moderações | Lista anúncios moderados (inativo, inativo p/ revisar, ativo com perda de exposição) com histórico [F S4]. | — | Média | Total (ML) |
| Qualidade/erros de anúncios | Acompanha a qualidade e o status, com artigos de correção de erros da API ML [F S7, S8 cadastros]. | — | Média | Total |
| Sincronização de estoque | Envia saldo físico ou disponível, por depósito, com estoque de segurança. Webhook/tempo real [F S4, S10]. | Contado no limite mensal de sincronizações [F S1] | **Alta**: fila confiável, rate limit do ML, sem furo | Total |
| Sincronização de preço | Preço fixo (± ajuste), lista de preços, regras do canal ou calculadora. Preço promocional "de/por" [F S4, S10, S54]. | — | Média | Total |
| Gerenciador de preços dos anúncios | Alteração em massa: base (produto ou anúncio atual), desconto ou acréscimo % e R$, promoção on/off, arredondamento (centavos 9/0), salvar e enviar ou só salvar [F S54]. | — | Média | Total |
| Calculadora de preços | **Só ML**: base (venda ou custo), frete automático do ML ou manual (fixo ou por faixa de peso), comissão do ML consultada automaticamente (por dentro), margem, até 5 custos extras, arredondamento. Até 10 calculadoras por integração [F S55]. | Impulsione/Domine. Extensão nos menores [F S55] | Média | Total (ML) |
| Sugestão de preços (beta) | Opção na aba Preços da integração [R2]. | Beta | — | Total (ML) |
| Campanhas promocionais | Campanha com período e % de desconto + arredondamento. Envia e reverte o preço nas datas [F S53]. | Impulsione+ [F S1] | Média | Total |
| Google Shopping | Feed de produtos [F S1]. | Impulsione+ | Média | Parcial |
| Perguntas do e-commerce (pré-venda) | Perguntas do ML, Shopee (beta) e Americanas (beta) numa tela, com respostas padrão por "/tag", saudação/assinatura e notificação a cada 2 minutos [F S51, S7]. | Construa+ (Crescer+) [F S51] | Média | Total |
| Pós-venda Mercado Livre | Mensagens do pós-venda (várias contas), filtros (não respondidas, com reclamação, com devolução), respostas padrão e anexos, oferecer devolução, acompanhar e revisar devoluções, reportar problema. IA de resposta em piloto [F S56]. | — | Alta (claims/returns API) | Total (ML) |
| Dias laborais / capacidade de envio | Pontos facultativos (dias sem envio) e capacidade diária por dia da semana enviados ao ML. "Meus horários de envio" por forma de frete [F S4, R2]. | — | Baixa | Total (ML) |
| Limite e alerta de anúncios | Limite por plano. Correções de erros (UNIT_TYPE, estoque inválido, ofertas ativas) [F S8 cadastros]. | — | Baixa | Total |
| Log de sincronização de produtos | Relatório das interações produto/estoque com o e-commerce [R1]. | — | Média | Total |

## 8. Logística

| Feature | O que faz | Plano/observação | Complexidade | Dep. marketplace |
|---|---|---|---|---|
| Mercado Envios (ME2) | Forma de envio cadastrada sozinha. Etiqueta pela expedição (PDF/ZPL definido no painel ML) [F S47]. | Extensão Mercado Envios [F S47] | Média | Total (ML) |
| Etiqueta com data programada | O pedido fica "em aberto" com o marcador "etiqueta com data programada" até o ML liberar. Antes disso não dá para enviar NF nem gerar etiqueta [F S48]. | — | Média | Total (ML) |
| Mercado Envios Coletas | Habilitada por integração (SP, PR, MG, RJ). Expedição → "solicitar Coleta Mercado Envios" → imprime etiquetas de coleta [F S49]. | Extensão | Média | Total (ML) |
| Coleta Rápida | Pedidos com a tag "coleta rápida" para filtrar e priorizar [F S50]. | Só clientes selecionados | Baixa | Total (ML) |
| Mercado Envios Flex | Configura prazos, mesmo dia, horário de corte, sábado/domingo, área de cobertura e limite diário. Ativa ou desativa Flex por anúncio. Etiqueta Flex pela expedição [F S46]. | — | Média | Total (ML) |
| ME1 (envio próprio do vendedor) | Para volumosos e contratos próprios, com mapeamento de formas de frete [F S4, S10]. | — | Média | Total (ML) |
| ML Fulfillment | Ver §5 (inbound) e §4 (fiscal). Pedidos e NFs importados, estoque controlado pelo ML, devoluções (ao vendedor ou ao CD) [F S3]. | Exige contrato Full [F S3] | Alta | Total (ML) |
| Amazon DBA/FBA, Magalu Entregas, Americanas Entregas, Shopee Envios, AliExpress | Envios dos marketplaces [F S8 logística, R2]. | — | Média cada | Total (cada canal) |
| Correios (contrato próprio) | PAC/SEDEX, etiquetas, AR, logística reversa, orçamento online, declaração de conteúdo/DC-e [F S23, S8 logística]. | Senha de API Correios | Média | Nenhuma |
| Transportadoras / gateways logísticos | Frenet, Melhor Envio, Kangu, Intelipost, Loggi, Mandaê, Envvias, Buslog, FedEx, Manda Bem, Estoca, Correios Log+ [F S8 logística]. | Integrações logísticas: parcial no Avance [F S1] | Média cada | Nenhuma |
| Olist Envios | Frete próprio da Olist com coleta, créditos por plano e cashback [F S1, S84]. | Serviço Olist | Fora do escopo [I] | Nenhuma |
| Cotação de frete | Cotação integrada. Webhook de cotação para a loja [F S5, S33]. | — | Média | Nenhuma |
| Rastreio ao cliente | E-mail de rastreio e de confirmação de entrega com variáveis. XML por e-mail à transportadora [F S24]. | — | Baixa | Nenhuma |
| Peso cubado | Enum "PesoCorteCubado" no bundle [R1]. | Inferência de uso | Baixa | Parcial |

## 9. Relatórios / BI

| Feature | O que faz | Plano/observação | Complexidade | Dep. marketplace |
|---|---|---|---|---|
| Dashboard de vendas | Valor vendido, ticket médio, vendas por UF, top produtos, desempenho por integração, horários de pico, vendas × devoluções, funil CRM, totalizador de NFs. Widgets configuráveis para toda a conta [F S86, R1]. | — | Média | Parcial |
| Dashboard de finanças | Saldos, projeção de caixa, a pagar/receber e atrasados [F S86]. | — | Média | Nenhuma |
| Relatório de vendas | Agrupável por cliente, produto ou vendedor, com lucratividade por custo médio ou última compra [R1]. | — | Média | Nenhuma |
| Curva ABC | Clientes ou produtos, por quantidade ou valor (A = 80%, B = 15%, C = 5%). Filtra por canal, situação, tag, categoria e natureza. Agrupa variações no pai [F S59]. | — | Baixa | Parcial (filtro por canal) |
| Margem de contribuição / custos do e-commerce | Ver §6 [F S92]. | Impulsione+ | Média | Total (ML) |
| Giro, ruptura e sem movimentação | Giro de estoque, abaixo do mínimo, sem movimentação, maior circulação [F S93, R1]. | — | Baixa | Nenhuma |
| Performance de vendas / comissões / desconto do gerente | §3 [F S59–S61]. | — | Baixa | Nenhuma |
| Vendas por canal | Filtros por e-commerce e por integração no dashboard e na ABC [F S59, S86]. | — | Baixa | Parcial |
| Relatórios personalizados | Fontes: Produtos, Vendas, Estoque, Contas a pagar e a receber. Colunas, filtros, agrupamento e ordenação escolhidos pelo usuário. Download e impressão [F S85]. "Criar relatório com a Lis" [R1]. | Todos [F S1] | Alta: construtor genérico de relatórios | Nenhuma |
| Relatórios do PDV e da expedição | Controle de caixa do PDV, relatório de expedição [R1, S23]. | — | Baixa | Nenhuma |
| Resumos automáticos (Lis) | Resumo financeiro diário, relatório diário de vendas, monitor de novos pedidos, pedidos parados na expedição, clientes VIP em risco [F S89, S93 busca]. | Consome créditos de IA [I] | Média | Parcial |

## 10. Outros (plataforma, segurança, API, serviços)

| Feature | O que faz | Plano/observação | Complexidade | Dep. marketplace |
|---|---|---|---|---|
| Usuários e perfis | Perfis com permissões granulares por módulo, permissões extras, perfil gerente (tudo), acesso restrito por tipo de contato [F S39]. ACL no front [R1]. | Usuários ilimitados [F S1] | Média | Nenhuma |
| Restrições de acesso | 2FA (TOTP), dias da semana, horários, IP fixo, **uma sessão ativa por usuário** [F S39]. | — | Média | Nenhuma |
| Usuário de suporte | Habilita acesso temporário do suporte à conta [F S8 minha-conta]. | — | Baixa | Nenhuma |
| Logs/auditoria | Log de movimentação de estoque com autor e origem [F S40]. Ocorrências de pedido com bloqueio de exclusão [F S19]. | Não achei auditoria geral de todas as entidades [I] | Média | Nenhuma |
| Multiempresa | Liga várias contas (1 por CNPJ, cada uma com plano próprio): compartilha clientes, fornecedores, produtos e pedidos (com autorização) e estoque. Filtro por empresa nos dashboards e no DRE [F S80, S86, S30]. | Construa+ [F S1, S80] | Alta (multi-tenant) | Parcial |
| Loja de extensões | Ativa e desativa módulos: Variações, Reserva, Depósitos, DRE, Financeiro Intermediário, Campanha promocional, Calculadora, Mercado Envios, Coletas, Controle de Produção, Vendedores, Controle de Caixa, Multiempresa, Webhooks, Token API, API de estoque em tempo real, Edição de registros de estoque… Algumas grátis, outras mensais [F S88, S16, S31, S41, S42, S49, S53, S55]. | — | Média (feature flags por tenant) | Nenhuma |
| API v2 (token) | REST com token: contatos, produtos (estoque, estrutura, tags, alterados, atualizações de estoque, preços), listas de preços, vendedores, CRM, contas a pagar/receber, pedidos (gerar NF, OP, rastreio, situação, marcadores, estoque, contas), PDV, NF-e, NFS-e, expedição, frete, contratos, separação, tabelas auxiliares [F S33, S34]. | Lote de até 20 registros. Até 100 por consulta. Concorrência = 1/4 do limite. HTTP 429 [F S35] | — | Nenhuma |
| API v3 (OAuth2) | OAuth2/Keycloak, ~168 operações: caixa, categorias, categorias financeiras, contas bancárias e financeiras, contas a pagar/receber, contatos e pessoas, CRM, empresa, depósitos, estoque (+ logs), expedição (agrupamentos, etiquetas, "One Step Shipping"), extrato bancário/conciliação, formas de envio, pagamento e recebimento, intermediadores, listas de preço, marcas, NF (autorizar, cancelar, XML, DANFE, por XML de fornecedor), orçamentos, OC, OS, pedidos, produtos (kits, fabricados, variações, anexos, custos), separação, serviços, usuários, vendedores [F S34]. Changelog até 3.1.11 (out/2026) [F S35]. | Limite por conta compartilhado entre apps. Leitura tem mais cota que escrita. Headers X-RateLimit-* [F S35] | — | Nenhuma |
| Webhooks | v3: vendas criadas/alteradas, pedidos enviados, lançamentos de estoque, NF autorizada. Retenta até 10× com +5 min a cada vez. Não há webhook por aplicativo [F S35]. v2: estoque, produtos, rastreio, NF, preço, situação de pedido, cotação de frete [F S35]. | App "Webhooks" só em alguns planos [F S35] | Média | Nenhuma |
| Lis (agentes de IA) | Assistente que executa ações: automações agendadas, criação de relatórios, consulta de logs, sugestão de NCM, preenchimento de ficha técnica, títulos/descrições, fundo branco nas fotos, correção de NF rejeitada, conciliação MP [F S2, S9, S11, S36, S40, S71, S89]. | Créditos mensais por plano [F S1] | Média a Alta (opcional) | Parcial |
| Assistente da Olist (suporte) | Chat de suporte virtual gratuito, liberação gradual [F busca S100]. | — | — | — |
| Central de notificações | Notificações do sistema e do navegador (ex.: perguntas) [F S8 minha-conta, S51]. | — | Baixa | Nenhuma |
| Envio de documentos (e-mail SMTP, WhatsApp) | Servidor SMTP próprio, modelos de mensagem, envio por WhatsApp [F S8 minha-conta, S20]. | — | Baixa | Nenhuma |
| Impressão PrintNode | Impressão direta e automática em impressora térmica [F S38, R1]. | — | Média | Nenhuma |
| Backup / exportação | Exporta 9 módulos (contatos, produtos, OC, pedidos, propostas, caixa, XMLs 55/65, a receber, a pagar) em XLS/CSV. Agendamento manual, processado à meia-noite, link válido por 48 h, planilhas de até 500 linhas. **Não exporta composição de kits** [F S87]. | Só administrador | Baixa | Nenhuma |
| Gerenciar anexos / armazenamento | Controle do espaço usado (conta no plano) [F S8 minha-conta, S2]. | — | Baixa | Nenhuma |
| Ordens de serviço | OS com serviços e peças (reserva estoque), gera NF/NFS-e e lança contas e estoque [F S8 serviços, S34]. | Construa: 20/mês, Impulsione: 200 [F S1] | Média | Nenhuma |
| Contratos de serviço | Contratos recorrentes com adicionais, que geram cobranças mensais [F S8 serviços, S33]. | Impulsione+ [F S1] | Média | Nenhuma |
| E-commerce da Olist (loja própria) | Loja virtual integrada [F S1, S84]. | Produto separado | Fora do escopo [I] | Nenhuma |
| Conta Digital / Crédito / Maquininha / Link de pagamento | Serviços financeiros da Olist [F S1]. | — | Fora do escopo [I] | Nenhuma |
| Interface | Tema claro/escuro, canal de ideias, menu global, app mobile-first via navegador [R1, F busca S101]. | — | Baixa | Nenhuma |
| Calculadora de planos | Simulador de preço dos planos [F S8 minha-conta]. | — | N/A | — |

---

## (a) Fontes efetivamente lidas

Oficiais Olist/Tiny (lidas com WebFetch, salvo indicação):
- S1 https://olist.com/planos/
- S2 https://ajuda.olist.com/plano-e-mensalidade/novo-modelo-de-precos-do-erp-olist
- S3 https://ajuda.olist.com/pt_BR/mercado-livre/erp-integracao-mercado-livre-fulfillment
- S4 https://ajuda.olist.com/mercado-livre/mercado-livre-configuracoes-e-utilizacao
- S5 https://olist.com/sistema-erp/
- S6 https://olist.com/hub-de-integracao/mercado-livre (redirect de tiny.com.br/integracoes/mercado-livre)
- S7 https://materiais.tiny.com.br/recursos-tiny-e-mercado-livre
- S8 Índices de categoria: https://olist.helpjuice.com/erp · https://ajuda.olist.com/cadastros · /sumprimentos · /erp-vendas · /notas-fiscais-e-tributacao · /financas · /l-logistica · /e-integracoes · /primeiros-passos-no-erp · /pdv · /servicos · /minha-conta
- S9 https://ajuda.olist.com/hub-da-olist/venda-mais-e-opere-menos-com-o-hub-da-olist
- S10 https://ajuda.olist.com/hub-da-olist/como-integrar-com-o-mercado-livre-via-hub-da-olist
- S11 https://ajuda.olist.com/hub-da-olist/como-publicar-anuncios-no-hub-da-olist
- S12 https://ajuda.olist.com/cadastro-de-anuncios/mercado-livre-unificacao-de-anuncios-variacoes-de-produto-e-preco-por-variacao-pxv
- S13 https://ajuda.olist.com/produtos/modulo-de-produtos
- S14 https://ajuda.olist.com/produtos/como-cadastrar-produtos-conheca-os-5-metodos-disponiveis
- S15 https://ajuda.olist.com/produtos/configuracoes-do-cadastro-de-produtos
- S16 https://ajuda.olist.com/gestao-de-estoque/estoque
- S17 https://ajuda.olist.com/gestao-de-estoque/configuracoes-de-estoque
- S18 https://ajuda.olist.com/pedidos/como-funciona-o-fluxo-de-pedidos-do-erp
- S19 https://ajuda.olist.com/pedidos/pedidos-de-venda-configuracoes
- S20 https://ajuda.olist.com/pedidos/pedidos-de-venda-como-utilizar
- S21 https://ajuda.olist.com/separacao-de-produtos/separacao
- S22 https://ajuda.olist.com/separacao-de-produtos/separacao-configuracoes
- S23 https://ajuda.olist.com/expedicao/expedicao-utilizacao
- S24 https://ajuda.olist.com/expedicao/expedicao-configuracoes
- S25 https://ajuda.olist.com/compras/necessidades-de-compra
- S26 https://ajuda.olist.com/compras/ordens-de-compra
- S27 https://ajuda.olist.com/compras/conferencia-de-compra
- S28 https://ajuda.olist.com/tipos-de-documentos-fiscais-e-suas-operacoes/notas-de-entrada
- S29 https://ajuda.olist.com/configuracoes-financeiras/visao-geral-do-modulo-de-financas
- S30 https://ajuda.olist.com/relatorios-de-financas/demonstracao-do-resultado-do-exercicio-dre
- S31 https://ajuda.olist.com/gestao-financeira/cobrancas-bancarias
- S32 https://ajuda.olist.com/configuracoes-financeiras/financas-configuracoes-gerais
- S33 https://tiny.com.br/api-docs/api (API v2)
- S34 https://api-docs.erp.olist.com/llms.txt (índice completo da API v3 + v2)
- S35 https://api-docs.erp.olist.com/documentacao/webhooks/webhooks.md · /documentacao/comecando/limites-de-consulta.md · /api-v2/limites.md · /api-v2/webhooks/webhooks-tiny.md · /changelog/versao-3-1-8.md a 3-1-11.md
- S36 https://ajuda.olist.com/automacoes-com-ia/conciliacao-mercado-pago
- S39 https://ajuda.olist.com/gerenciamento-da-conta/usuarios-uso-e-configuracoes
- S40 https://ajuda.olist.com/acesso-e-seguranca/consulta-de-logs-no-erp
- S41 https://ajuda.olist.com/gestao-de-estoque/reserva-de-estoque
- S42 https://ajuda.olist.com/gestao-de-estoque/reserva-de-estoque-individual-por-deposito
- S43 https://ajuda.olist.com/produtos/controle-de-lotes-e-validades-de-produtos
- S44 https://ajuda.olist.com/producao/ordens-de-producao
- S45 https://ajuda.olist.com/gestao-de-envios-fulfillment/gestao-de-envios-do-fulfillment
- S46 https://ajuda.olist.com/erp-cotacao-de-fretes/mercado-envios-flex
- S47 https://ajuda.olist.com/erp-cotacao-de-fretes/mercado-envios
- S48 https://ajuda.olist.com/pt_BR/mercado-livre/mercado-livre-disponibilidade-de-etiquetas-dos-pedidos
- S49 https://ajuda.olist.com/erp-cotacao-de-fretes/mercado-envios-coletas
- S50 https://ajuda.olist.com/marketplaces/coleta-rapida-do-mercado-livre (só trecho da busca)
- S51 https://ajuda.olist.com/gestao-de-integracoes/perguntas-do-e-commerce
- S52 https://ajuda.olist.com/precificacao/formacao-de-precos
- S53 https://ajuda.olist.com/precificacao/campanhas-promocionais
- S54 https://ajuda.olist.com/precificacao/gerenciador-de-precos-dos-anuncios
- S55 https://ajuda.olist.com/precificacao/calculadora-de-precos
- S56 https://ajuda.olist.com/mercado-livre/pos-vendas-mercado-livre
- S57 https://ajuda.olist.com/pedidos/devolucoes-de-vendas
- S58 https://olist.com/blog/olist/tiny-news/devolucao-de-mercadorias-no-tiny-erp/
- S59 https://ajuda.olist.com/condicoes-comerciais/relatorio-de-curva-abc
- S60 https://ajuda.olist.com/condicoes-comerciais/performance-de-vendas
- S61 https://ajuda.olist.com/condicoes-comerciais/comissoes
- S62 https://ajuda.olist.com/tipos-de-documentos-fiscais-e-suas-operacoes/tipos-de-documentos-fiscais
- S63 https://ajuda.olist.com/primeiros-passos-para-emitir-notas-fiscais/natureza-de-operacao-de-saida-de-mercadorias
- S64 https://olist.com/blog/olist/tiny-news/spedfiscal-bloco-k/
- S65 https://ajuda.olist.com/tributos-e-obrigacoes-com-o-fisco/extracao-de-dados-para-contabilidade
- S66 https://ajuda.olist.com/tributos-e-obrigacoes-com-o-fisco/sintegra (trecho da busca)
- S67 https://ajuda.olist.com/tributos-e-obrigacoes-com-o-fisco/como-adicionar-regras-de-excecao-na-natureza-de-operacao (trecho da busca)
- S68 https://ajuda.olist.com/duvidas/autorizacao-automatica-de-nota-fiscal
- S69 https://olist.com/sistema-erp/emissao-nota-fiscal/
- S70 https://ajuda.olist.com/guias-de-pagamento-de-impostos/como-utilizar-guias-de-pagamento-de-impostos-no-erp
- S71 https://ajuda.olist.com/rejeicoes-de-notas-fiscais/novidade-correcao-de-notas-fiscais-rejeitadas-com-inteligencia-artificial
- S72 https://ajuda.olist.com/gestao-de-estoque/como-fazer-o-inventario-de-estoque-gerar-importar-planilha-ou-importar-por-coletor
- S73 https://ajuda.olist.com/separacao-de-produtos/bloqueio-de-pedidos-lock-na-separacao
- S74 https://ajuda.olist.com/produtos/como-trabalhar-com-kits-que-possuem-variacoes
- S75 https://ajuda.olist.com/produtos/acoes-em-lote-e-gerenciamento-de-produtos
- S76 https://ajuda.olist.com/cadastro-de-contatos/clientes-e-fornecedores
- S77 https://ajuda.olist.com/precificacao/listas-de-precos (trecho da busca)
- S78 https://ajuda.olist.com/produtos/variacoes-de-produtos-configuracoes (trecho da busca)
- S79 https://ajuda.olist.com/pedidos/crm-customer-relationship-management (trecho da busca)
- S80 https://ajuda.olist.com/gestao-de-dados/multiempresa-configuracoes (trecho da busca)
- S81 https://olist.com/sistema-erp/conciliacao-bancaria/
- S82 https://ajuda.olist.com/gestao-financeira/contas-a-pagar
- S83 https://ajuda.olist.com/gestao-financeira/caixa
- S84 https://olist.com/hub-de-integracao/
- S85 https://ajuda.olist.com/interface-e-personalizacao/relatorios-personalizados
- S86 https://ajuda.olist.com/interface-e-personalizacao/dashboard
- S87 https://ajuda.olist.com/interface-e-personalizacao/backup
- S88 https://ajuda.olist.com/gerenciamento-da-conta/loja-de-extensoes
- S89 https://olist.helpjuice.com/faqs (automações com a Lis)
- S90 https://ajuda.olist.com/expedicao/declaracao-de-conteudo-eletronica-dc-e
- S91 https://ajuda.olist.com/pedidos/atualizar-situacao-de-envio-dos-pedidos
- S92 https://ajuda.olist.com/pt_BR/precificacao/custos-do-e-commerce (S92b = trecho da busca sobre "margem de contribuição")
- S93 https://ajuda.olist.com/gestao-de-estoque/giro-de-estoque (trecho da busca)
- S99 trecho da busca sobre agrupamento de carrinho do ML numa NF (artigo oficial não lido)
- S100 https://ajuda.olist.com/atendimentoERP/assistente-virtual (trecho da busca)
- S101 https://olist.com/blog/olist/tiny-news/tiny-nova-interface-de-usuario/ (trecho da busca)

Terceiros:
- S37 https://gefersonalencar.com.br/2026/09/29/formas-de-recebimento-olist-erp-configurar-taxas-parcelas/
- S38 https://gefersonalencar.com.br/2026/09/28/guia-geral-configuracoes-olist-erp-passo-a-passo/
- S94 Reclame Aqui (as páginas devolvem 403. Usei títulos e trechos da busca): reclameaqui.com.br/empresa/olist-oficial/ e as reclamações citadas no item (c)
- S95 https://gosmarter.com.br/tiny-erp-para-e-commerce/ · https://lojexa.com/blog/tiny-erp-para-quem-serve-alternativa
- S96 https://escoladelojista.com.br/erp-olist-tiny-integracao-para-o-mercado-livre/ · https://gefersonalencar.com.br/2026/08/10/erp-olist-vale-a-pena-analise-cupom-2026/
- S97 https://kmee.com.br/comparativos/odoo-vs-bling/ (trecho da busca: "Bling e Tiny não suportam CT-e/MDF-e/SPED". Sobre SPED está desatualizado, porque a Olist anuncia SPED Fiscal no Impulsione+)
- https://github.com/opastorello/olist-mcp-server (contagem de 168 operações da API v3 por grupo)

Internas: R1 `olist-menu-rea.md`, R2 `olist-conta-rea.md` (REA na conta do dono, 16/10/2026).

---

## (b) Integração Tiny/Olist ↔ Mercado Livre em detalhe

**Arquitetura atual.** O ML está migrando para o **Hub da Olist**, uma nova camada de integração (também usada para Temu). A integração "antiga" continua documentada. A calculadora de preços funciona nas duas [F S9, S10, S55]. O Olist é **Parceiro Platinum** do ML [F S6].

**Conexão e multi-conta**
- OAuth ML em Início > Integrações > Mercado Livre. **Cada conta ML exige uma instalação separada** da integração [F S10].
- "ML Fulfillment" é uma integração à parte, presa à integração ML principal [R2].
- O pós-venda centraliza várias contas ML [F S56].
- A conta do dono hoje mostra **1 só conta ML** [R2]. Falta confirmar onde está a 2ª conta.

**Produtos e anúncios**
- Importar anúncios (todos, só novos ou um específico) e relacionar por SKU, título ou GTIN, com opção de atualizar SKU, preço, estoque e imagens. Imagens entram como anexo ou só como URL. Dá para importar só os ativos e tratar anúncio de uma variação como simples [F S4, R2].
- Publicar a partir do produto, via Hub: regras padrão (forma de envio, retirada, tipo Grátis/Clássico/Premium, frete grátis, garantia, loja oficial), categoria, atributos obrigatórios e ficha técnica (Lis preenche pelo GTIN ou pelo catálogo), anúncio próprio ou de catálogo, validação com ícones e painel de status [F S10, S11].
- Campos sincronizados: atributos, categoria, SKU, datas de promoção, descrição, dias de preparação, estoque, garantia, GTIN, imagens, preço normal e promocional, tabela de medidas, tipo de anúncio, variações, tipo logístico e retirada [F S10].
- Variações: mapeamento tipo/opção ERP ↔ ML, preço por variação (PxV), atualização por família [F S12, R2]. Suporte ao modelo **User Product** [F S9].
- Kits: o kit é vendido como produto. O ML vê o produto ou variação, não os componentes. A baixa no ERP é por componente [F S74]. O Hub "sinaliza kits virtuais" (trecho de busca de terceiro, não confirmado na ajuda oficial).
- Catálogo: elegibilidade, vínculo e sincronização. Anúncios de catálogo **não são importados** para o ERP [F S4].
- Também: pausar e ativar em lote, clonar (até 20, e entre integrações do mesmo canal via Hub), moderações, compatibilidade de autopeças, tabela de medidas, atributos sugeridos por categoria, correção de erros da API [F S4, S9, S12]. Desde 09/09/2024 o ML recusa links de vídeo [F S4].

**Estoque**
- Envia saldo físico ou **disponível** (físico − reservas), de um depósito, de "todos os próprios" ou de vários depósitos (tabela por depósito), com estoque de segurança por integração [F S4, S10, R2].
- Gatilho de baixa configurável por integração. Recomendado "ao autorizar NF-e" [F S10].
- Produto sob encomenda ignora o estoque de segurança [F S4].
- As sincronizações contam no limite do plano (150 mil a 900 mil/mês) [F S1].

**Preço**
- Preço fixo (± %), lista de preços, regras do marketplace ou **calculadora de preços** (só ML: comissão consultada no ML, frete automático ou por faixa de peso, margem, até 5 custos extras, até 10 calculadoras por integração, arredondamento) [F S4, S55].
- Gerenciador em massa, preço promocional e campanhas com datas [F S53, S54].
- Sugestão de preços em beta [R2].

**Pedidos**
- Importação automática pelo OMS, a partir do Construa [F S1]. Modo Automático, Manual (para testes) ou Não importar [F S10].
- Importa o frete, as parcelas e os pedidos sem dados de entrega. Completa o endereço pelo CEP. Formata o nome. Põe o endereço nas observações [F S4, S10, R2].
- Intermediador obrigatório: ML CNPJ 03.361.252/0001-34 e instituição de pagamento 10.573.521/0001-91 [F S10].
- Usa o valor do pagamento integrado quando a NF diverge [F S10, R2].
- Mapeamento de situações, formas de recebimento (padrão "Mercado Pago"), formas de frete e categoria financeira padrão ("Vendas Mercado Livre") [R2].
- Situação exclusiva "Em devolução" [F S18].
- Carrinho (pack) agrupado numa NF [F S99].

**Fiscal**
- Natureza de operação para venda, para venda a contribuinte de ICMS e para transferência ao Full [F S4, S10, R2].
- Envio da NF ao ML: automático ao autorizar ou manual, como mensagem + XML, só XML ou só mensagem [F S4, R2].
- Antes da data liberada pelo ML ("etiqueta com data programada") não se emite NF nem etiqueta [F S48].

**Envio (Mercado Envios)**
- ME2 com etiqueta do ML em PDF/ZPL (formato definido no painel ML). Etiqueta + DANFE unificada (beta) só em PDF 10×15 [F S47, R2].
- Coletas (SP/PR/MG/RJ): solicitar coleta pela expedição [F S49].
- Coleta Rápida: tag no pedido [F S50].
- **Flex**: prazos, mesmo dia, horário de corte, fim de semana, área de cobertura, limite diário, ativar/desativar por anúncio, etiqueta Flex pela expedição [F S46].
- ME1 para volumosos/contrato próprio [F S4].
- Rastreio enviado automaticamente ao marcar enviado. Marcação de entregue no ML [F S4, S10].
- Dias sem envio e capacidade diária por dia da semana [F S4].

**Full**
- Pré-requisitos: contrato Full, conta padrão e extensão Depósitos [F S3].
- Depósito "Exclusivo canal de venda" com CNPJ do CD, transferência automática ao identificar NF com esse CNPJ e "desconsiderar saldo" [F S3, S42].
- Modelo fiscal self storage (CAT 59, SP) ou filial [F S10, R2].
- Envio dos dados fiscais do produto ao ML (NCM, CSOSN, CEST, origem, GTIN) e mensagens fiscais (até 30) [F S3].
- Pedidos e NFs do Full importados automaticamente. No Full todas as situações viram "Enviado" e a baixa acontece "ao marcar enviado" [F S3, R2].
- Devoluções: (1) ao vendedor, importando a NF de entrada; (2) ao CD, onde o ML emite devolução + remessa e o ERP lança e transfere entre depósitos [F S3].
- Inbound pelo módulo "Envios Fulfillment": anúncios e quantidades, reserva, planilha no layout ML, nº do envio e coleta, vínculo da NF de transferência, divisão por CD [F S45].
- **Não faz pelo ERP**: enviar produtos ao Full (só importa), controlar o estoque Full (o ML controla), imprimir DANFE das notas do painel ML, enviar preço automaticamente (só manual) [F S3].

**Atendimento**
- Perguntas pré-venda com respostas padrão e notificação a cada 2 min [F S51].
- Pós-venda: mensagens, reclamações e devoluções (oferecer, acompanhar, revisar, reportar problema), com IA de resposta em piloto [F S56].

**Financeiro ML**
- O relatório "Custos do e-commerce" importa do ML **automaticamente** comissão, frete e taxas por pedido [F S92].
- A conciliação automática via Lis cobre o **gateway Mercado Pago do e-commerce**. Vendas diretas do marketplace ML ficam fora [F S36].
- Conciliar repasses do ML pelo extrato Mercado Pago, usando a "Conta gráfica Mercado Pago" como conta financeira, é **inferência** [I a partir de S37].

**Não encontrado na documentação:** gestão de **Mercado Ads** e de **reputação/métricas da conta**; gestão de **promoções nativas do ML** (Central de Promoções/ofertas relâmpago), pois a campanha do ERP só muda preço; **Mercado Livre Full multi-CD** com previsão de inbound; e se a integração faz cotação de frete ME2 para o anúncio (a página cita "cotação de fretes" de forma genérica [F S4]).

---

## (c) Dores e limitações conhecidas

Os títulos e trechos do Reclame Aqui vêm da busca, porque as páginas devolvem 403. A Olist tem nota 7,7/10 com cerca de 583–594 reclamações, 92,5% respondidas e 77,3% resolvidas [F S94].

1. **Estoque errado ou zerado automaticamente**: "alterações automáticas e equivocadas de estoque", zerando ou inflando saldos. Num caso, ao acionar a IA para investigar, mais SKUs foram zerados, passando de 500 afetados [F S94].
2. **Estoque Full misturado com o Galpão**: "ERROS GRAVÍSSIMOS DE ESTOQUE … COM DEPÓSITO FULL MERCADO LIVRE". O ERP somou unidades do depósito Geral com as do Full nas reservas [F S94].
3. **Divergência entre o saldo enviado e o saldo no anúncio ML**: "falha grave e recorrente na integração de estoque … com o Mercado Livre" [F S94].
4. **Pedidos duplicados do Full e problemas na importação de NFs**, sem solução há cerca de 1,5 ano. Um cliente cita "controle de estoque manual" como contorno [F S94].
5. **Mudanças na integração ML sem aviso** e sem comunicação de impacto. A migração para o Hub da Olist causou falhas de sincronização e de importação de pedidos [F S94].
6. **Exclusão de anúncio por erro de integração** com prejuízo [F S94]. Duplicidade ao vincular tabela de medidas em novas variações [F S94/S95].
7. **Etiquetas com código inválido** na agência e **peso/dimensões erradas enviados ao ML (ME1)** [F S94].
8. **Instabilidade**: sistema fora do ar, Gateway time-out 504, sem aviso [F S94].
9. **Suporte** muito criticado depois da aquisição pela Olist: respostas fora de contexto, chamados encerrados sem solução, queda de qualidade [F S94].
10. **Mudanças unilaterais de planos e preço**: o novo modelo por armazenamento (vigente desde 01/06/2026) **bloqueia a escrita** quando o limite é atingido [F S2, S94].
11. **Funções importantes presas aos planos caros**: OMS (importação automática) só a partir do Construa. Calculadora, margem de contribuição, custos do e-commerce, separação/embalagem em lote, impressão automática, SPED, CRM e devoluções só a partir do Impulsione [F S1, S95].
12. **A sincronização não é em tempo real**: ocorre em intervalos, então uma venda em outro canal pode levar minutos para refletir no ML [F S95, terceiro]. Um terceiro alega o contrário, "< 30 s" [S96]. Não confirmado.
13. **Fiscal inflexível em cenários específicos** (vários NCMs, CFOPs interestaduais, produção própria) segundo terceiro [S95]. Depende muito de configurar bem a natureza de operação [S95].
14. **Limitações estruturais encontradas na doc**: estoque de segurança único por integração [F S10]; buscador de notas limitado a 20/dia [F S28]; planilhas de importação/exportação de 500 linhas [F S75, S76, S87]; backup manual com link de 48 h e sem kits [F S87]; "uma sessão por usuário" [F S39]; não há webhook por aplicativo e o limite de API é por conta [F S35]; Full sem controle de estoque nem envio de produtos pelo ERP [F S3]; conciliação automática não cobre vendas do marketplace ML [F S36]; **não há centro de custo** [F S82].

---

## (d) O que não consegui confirmar

- **Lista completa de extensões** da Loja de Extensões, com preço de cada uma. O artigo não lista, só dá para ver dentro do sistema [F S88].
- **MDF-e e CT-e (emissão)**: a ajuda diz que não há emissão de CT-e [F S62]. Não achei MDF-e (terceiro diz que não existe [S97]).
- **SPED Contribuições (EFD-PIS/COFINS)** e **ECD/ECF**: só confirmei SPED Fiscal com Bloco K.
- **Carta de correção e inutilização de numeração**: há indícios (exportação de CC-e, relatório de numeração), mas não li as telas de emissão.
- **Conferência cega** na compra: só fonte de terceiro [S38].
- **Centro de custo** e **rateio de frete/despesas na OC**: não aparecem na documentação.
- **Agrupamento de carrinho numa NF**: só pelo trecho de busca, o artigo oficial não foi lido.
- **Gestão de Mercado Ads, reputação, promoções nativas do ML e métricas de anúncio (visitas/conversão)** dentro do ERP: não encontrei.
- **Conciliação de repasses do marketplace ML** (liberações do Mercado Pago × pedidos × tarifas × frete × estornos): não há feature dedicada. A automação da Lis exclui explicitamente as vendas do marketplace.
- **Tempo real × intervalo** da sincronização de estoque: as fontes se contradizem.
- **Auditoria geral** (log de quem alterou preço, pedido, cadastro): só confirmei o log de estoque e o bloqueio de exclusão de ocorrências de pedido.
- **Plano exato de cada recurso**: vários artigos ainda usam os nomes antigos (Crescer/Evoluir/Essencial/Grande) e às vezes contradizem a página de planos (ex.: lotes, API no Avance, devoluções Construa × Impulsione).
- **Conteúdo de "Painel de Automações", "FCI", "Margem Contribuição"** (telas): vi os itens de menu na conta [R1], mas não li os artigos de ajuda.
- **Changelog do produto (não da API)**: o repositório github.com/olist/erp-changelog aparece na busca mas devolve 404 (privado). Só li o changelog da API v3.

---

## Leitura para o projeto Pinese [I]

Isto é inferência, para orientar o escopo e não para registrar como fato. Para 2 pessoas vendendo no ML (2 contas, Full + coleta/agência) no nicho casa/jardim, o núcleo que de fato substitui o Olist é:

1. Cadastro com variações e kits.
2. Estoque multi-depósito com reserva e depósito Full.
3. Integração ML de verdade (multi-conta, pedidos/packs, estoque com margem, preço/calculadora, NF ao ML, etiquetas ME2/coleta, Full inbound e importação de NFs do Full, devoluções, perguntas e pós-venda).
4. NF-e 55 com natureza de operação e exceções.
5. Notas de entrada por XML com manifestação.
6. Necessidade de compra.
7. Contas a pagar e receber + conciliação de repasses do ML.
8. Margem real por pedido, anúncio e conta.

Ficam de fora ou para depois: PDV, NFC-e/SAT, NFS-e, OS/contratos, CRM, produção, CNAB, SPED (o contador costuma fazer), multiempresa e a maioria das integrações de outros canais. As maiores oportunidades sobre o Olist estão nas dores (c) 1–5 e no item "conciliação de repasses do ML", que o Olist não cobre.
