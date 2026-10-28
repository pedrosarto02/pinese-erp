# 01 · Mapa de features: engenharia reversa do Olist ERP + referências de mercado

> **Projeto:** ERP próprio (Pinese), substituto do Olist ERP (ex-Tiny), nascido para marketplaces, com o Mercado Livre primeiro.
> **Fase:** contexto e descoberta. **Este documento não tem código, schema nem protótipo.**
> **Data:** 16/10/2026 · **Status:** rascunho para a entrevista de escopo (a classificação MVP/Fase 2/Fora sai da entrevista e vai para o `02-contexto-erp.md`).

---

## 0. Como este mapa foi feito

| Fonte | Método | Arquivo de apoio |
|---|---|---|
| **Sua conta Olist** (referência principal) | Engenharia reversa **somente leitura** com o REA: Chrome com perfil dedicado, análise do bundle JS, snapshot de acessibilidade tela a tela (menus, abas, botões, campos, combos, configurações). Nada foi salvo e nenhum dado pessoal foi registrado, só contagens agregadas. | `_pesquisa/olist-menu-rea.md`, `_pesquisa/olist-conta-rea.md` |
| Documentação Olist/Tiny | Central de ajuda, páginas de planos, API v2/v3, changelog, Reclame Aqui (trechos) | `_pesquisa/tiny-olist.md` |
| Bling e Omie | Central de ajuda inteira baixada e indexada (cerca de 3.000 artigos de cada), OpenAPI do Bling, lista de APIs do Omie | `_pesquisa/bling-omie.md` |
| Hubs + API do ML | Ideris, Anymarket, Plugg.to, Magis5 e cerca de 50 páginas oficiais do devsite do Mercado Livre e do Mercado Pago | `_pesquisa/hubs-e-api-ml.md` |

### Legenda das colunas
- **Origem:** `OL` = visto na sua conta Olist (REA) · `OD` = documentação Olist/Tiny · `BL` = Bling · `OM` = Omie · `HB` = hubs (Ideris, Anymarket, Plugg.to, Magis5) · `ML` = recurso da API do Mercado Livre/Mercado Pago.
- **Na sua conta hoje:** ✅ usa · ⚪ existe no Olist, mas você não usa · ❌ não existe no Olist (só nas referências) · ❓ não deu para saber, vai para a entrevista.
- **Complexidade** para reconstruir num ERP próprio (estimativa): **B** baixa · **M** média · **A** alta (com o motivo).
- **Dependência de marketplace:** **Nenhuma** · **Parcial** (regra interna que lê ou grava no canal) · **Total** (só existe por causa do canal).
- O `ID` serve para referenciar a feature na entrevista e na matriz de escopo.

---

## 1. O que a engenharia reversa revelou sobre a sua operação

| Sinal (agregado, últimos 30 dias) | Valor observado | Implicação para o ERP |
|---|---|---|
| Pedidos | **~1.079/mês (~36/dia)**; numeração já passa de 48,9 mil | Volume pequeno/médio. Nada de big data, mas a importação precisa ser confiável e idempotente. |
| Canais integrados | ML, **ML Fulfillment**, **Shopee**, Amazon, Amazon FBA Classic, Magalu, TikTok Shop | Multicanal **já é realidade**, não é só futuro. |
| Anúncios ativos | ML 52 · ML Full 45 · Shopee 19 · outros poucos | ~116 anúncios para 45 produtos: N anúncios por produto. |
| Contas ML conectadas | **Só 1** (o Full aponta para ela) | ❓ Onde está a 2ª conta? |
| Produtos | 45 (37 simples, 7 kits "multipack", 1 com variação por voltagem) | Catálogo pequeno. A complexidade está nos vínculos com anúncios, não no volume. |
| Depósitos | **Galpão** (padrão), **Devoluções**, **ML Full**, **Amazon Full**, **Shopee Full** | Multidepósito com Full por canal. |
| Estoque | Reserva ligada · saída "ao marcar como enviado" · **estoque negativo permitido** · segurança = 3 no ML | Regras que vamos reproduzir (ou corrigir). |
| NF-e | ~1.018 saídas e ~1.021 entradas/mês, **quase todas "Fulfillment" emitidas pelo próprio ML** e importadas. Emissão própria pequena (série 4, nº ~867). Automação de NF desligada. **Simples Nacional.** | Emitir NF não é o gargalo. O peso está em **importar e conciliar** as notas do Full. ❓ Quem emite as notas das vendas fora do Full? |
| Devoluções | **822 registros** (812 finalizadas, 10 abertas) | Devolução é processo central, com depósito próprio. |
| Pós-venda / perguntas | Em uso no ML (com reclamações). Shopee com **erro de token** | Liga direto com o agente de atendimento. |
| Compras | **Zero** ordens de compra, conferências, serviços tomados, FCI | ❓ Como a mercadoria entra hoje? |
| Financeiro | Contas a receber vazio · caixa sem movimento · 7 contas a pagar atrasadas · 2 contas (Olist Conta Digital, Mercado Pago) | Financeiro **feito fora do Olist** (ou não feito). Grande oportunidade: conciliação de repasses. |
| CRM, propostas, vendedores, comissões, metas, PDV, multiempresa, API/webhooks | Vazios ou desligados | Fortes candidatos a **Fora**. |
| Margem de contribuição / custos do e-commerce | Telas existem, mas estão **vazias/zeradas** | ❓ Como você controla margem hoje? |
| Usuários | 5: você (admin), 3 "gerente" (2 da **contabilidade externa**), 1 perfil "Logística" | RBAC: dono, operador de logística e contador (somente leitura/exportação). |
| Plano Olist | **Domine** (R$ 948/mês na tabela 2026) | Referência de custo evitado. |
| Arquitetura do Olist | PHP legado (jQuery + xajax) + "ilhas" React, assets v3.92.62 | Ajuda a entender por que integrações quebram e mudanças vêm sem aviso. |

---

## 2. Mapa de features por módulo

### 2.1 Cadastros

| ID | Feature | O que faz | Origem | Na sua conta hoje | Complexidade | Dependência de marketplace |
|---|---|---|---|---|---|---|
| CAD-01 | Produto simples | SKU, GTIN, nome, unidade, preço, preço promocional, peso líquido/bruto, L×A×C, embalagem | OL, OD, BL, OM | ✅ 37 produtos | B | Nenhuma |
| CAD-02 | Produto com variações (grade) | Produto-pai + variações (cor, voltagem, aroma…), SKU/preço/estoque/imagens por variação; grades padrão | OL, OD, BL, OM | ✅ 1 produto (voltagem) | M | Nenhuma (mapeada para as variações do ML) |
| CAD-03 | Kit / composição | Kit de componentes; estoque derivado (mín. componente ÷ qtd); a venda baixa os componentes; preço/peso recalculados pelos componentes | OL, OD, BL, HB | ✅ 7 kits (multipacks) | M | Nenhuma |
| CAD-04 | Kit com variações | Cada variação do pai é um kit com composição própria | OD, BL | ❓ | M | Parcial |
| CAD-05 | Matéria-prima / fabricado (BOM) | Estrutura de produção, insumos | OL, OD, OM | ⚪ 0 | M | Nenhuma |
| CAD-06 | Modelo Produto → User Product → N anúncios | SKU interno ↔ `user_product_id` ↔ N `item_id` por conta/condição de venda (até 30 por UP); família (`family_name`) | ML, BL, OD (Hub da Olist) | ❌ (o Olist está migrando) | **A**: modelo legado e UP coexistem; afeta todo o schema | Total |
| CAD-07 | Dados fiscais do produto | NCM, CEST, origem ICMS (0–8), tipo do item SPED, unidade tributável, EX TIPI, GTIN tributável | OL, OD, BL, OM | ✅ | M | Nenhuma |
| CAD-08 | Sugestão de NCM / classificação fiscal por IA | Sugere o NCM pelo nome/descrição | OL (Lis), OM (Omie.IA) | ⚪ | M (opcional) | Nenhuma |
| CAD-09 | Ficha técnica / atributos | Atributos universais (preenchem todos os canais) + atributos específicos por categoria do ML; "sugerir atributos" | OL, OD, BL, ML | ✅ (aba existe) | **A**: atributos por categoria de cada canal | Parcial |
| CAD-10 | Ficha de conhecimento do produto (manual PDF, vídeos, garantia, medidas, materiais, FAQ) | Base de conhecimento para o **agente de atendimento** | ❌ em todos (o Olist só tem "link do vídeo" e "garantia" texto) | ❌ | M | Parcial (alimenta o agente do ML) |
| CAD-11 | Imagens e anexos | Galeria por produto/variação, ordem, herança pai→variação, anexos | OL, OD, BL | ✅ | M (storage + URLs públicas estáveis para o ML baixar) | Parcial |
| CAD-12 | Remoção de fundo / tratamento de imagem por IA | Fundo branco, redimensionar | BL, HB, OD (Lis) | ⚪ | M | Nenhuma |
| CAD-13 | Descrição complementar + SEO | Editor rico/HTML, slug, keywords, título/descrição SEO | OL, OD | ✅ | B | Parcial |
| CAD-14 | Categorias (árvore) | Árvore própria ou espelhada do ML, mapeada para MLB | OL, OD | ✅ ~34 mapeadas | B | Parcial |
| CAD-15 | Marcas, tags, linhas de produto | Agrupadores para filtros, relatórios e comissão | OL, OD, BL | ⚪ quase sem uso | B | Nenhuma |
| CAD-16 | Tabela de medidas | Para vestuário/calçado, vinculada ao ML | OL, OD, BL | ⚪ | M | Parcial |
| CAD-17 | Campos customizados | Campos próprios (texto, número, lista…) em produto/contato, mapeáveis para atributos do canal | BL | ❌ | M | Parcial |
| CAD-18 | Listas de preço | Regra geral + exceções por produto, vinculáveis a canal | OL, OD, BL, OM | ⚪ nenhuma cadastrada | M | Parcial |
| CAD-19 | Custos do produto / histórico de custo | Custo de compra, custo médio, histórico por data, impostos recuperáveis, markup | OL, OD, BL, OM | ⚪ 1 registro (custo inicial) | M | Nenhuma |
| CAD-20 | Fornecedores do produto | Vínculo produto ↔ fornecedor + código no fornecedor (casa o XML de entrada) | OL, OD, BL | ⚪ sem vínculo | B | Nenhuma |
| CAD-21 | Embalagens | Cadastro de caixas/envelopes; sugestão automática da melhor embalagem (bin-packing) | OL, OD, BL, OM | ⚪ só o padrão Correios (37) | B / M (sugestão) | Parcial |
| CAD-22 | Clientes e fornecedores (contatos) | Cadastro único com tipos (cliente/fornecedor/transportador/outro), PF/PJ/estrangeiro, contribuinte ICMS, IE, endereços, pessoas de contato, e-mail para NF-e | OL, OD, BL, OM | ✅ 46.610 (quase todos automáticos) | B | Nenhuma |
| CAD-23 | Cliente automático a partir do pedido | Cria o comprador na importação, dados fiscais via `billing-info`, completa endereço pelo CEP, formata nome | OL, OD, ML | ✅ | M (LGPD, endereço ofuscado até o pagamento) | Total |
| CAD-24 | Consulta CNPJ (Receita/SEFAZ) | Preenche o cadastro pelo CNPJ | OD, BL | ❓ | M (exige certificado) | Nenhuma |
| CAD-25 | Limite de crédito do cliente | Bloqueia/alerta inadimplente | OL, OD, OM | ⚪ | B | Nenhuma |
| CAD-26 | Vendedores | Cadastro ligado a comissões, metas e permissões | OL, OD, BL, OM | ⚪ nenhum | B | Nenhuma |
| CAD-27 | Intermediadores (NT 2020.006) | CNPJ do marketplace/instituição de pagamento na NF | OL, OD, BL | ✅ 7 cadastrados | B | Total |
| CAD-28 | Contas de marketplace (multiconta/multi-CNPJ) | Cadastro de cada conta ML/Shopee etc. com grant, preferências e regras próprias | OL, OD, BL, HB, ML | ✅ 1 ML + outros canais | M | Total |
| CAD-29 | Cadastro em massa por planilha + ações em lote | Importar/exportar produtos, kits, variações, contatos; editar em massa (CEST, categoria, origem, marca…); clonar, unificar duplicados, excluir lógico | OL, OD, BL, OM | ⚪ | M | Nenhuma |
| CAD-30 | Importar produtos a partir dos anúncios | Espelha anúncios existentes, vincula por SKU/GTIN/título | OL, OD, HB, ML | ✅ ("Receber do e-commerce") | M (crítico na migração) | Total |
| CAD-31 | Validação de cadastro antes de publicar | Checa campos obrigatórios exigidos pelo canal | BL, OD | ❓ | M | Parcial |

### 2.2 Estoque

| ID | Feature | O que faz | Origem | Na sua conta hoje | Complexidade | Dependência de marketplace |
|---|---|---|---|---|---|---|
| EST-01 | Controle de estoque por produto | Físico, **reservado**, disponível, custo médio, localização, unidade | OL, OD, BL, OM | ✅ | M | Nenhuma |
| EST-02 | Lançamentos manuais (entrada/saída/balanço) | Movimento avulso com data, hora, quantidade, preço e observação | OL, OD | ✅ | B | Nenhuma |
| EST-03 | Kardex / log de movimentação (auditoria) | Quem, quando e de onde (manual, integração, API, inventário…), sem edição | OL, OD, BL, OM | ✅ (log de estoque) | M | Nenhuma |
| EST-04 | **Múltiplos depósitos** | Depósito próprio, de devoluções e Full por canal; depósito padrão; "desconsiderar saldo" | OL, OD, BL, OM | ✅ 5 depósitos | M | Parcial |
| EST-05 | Transferência entre depósitos | Origem → destino, vários itens | OL, OD, BL, OM | ✅ | B | Nenhuma |
| EST-06 | **Reserva de estoque** | Pedido reserva conforme a situação; baixa libera a reserva; cancelamento devolve; reserva manual | OL, OD, BL, OM | ✅ ligada | **A**: concorrência entre canais e eventos fora de ordem | Parcial |
| EST-07 | Gatilho de baixa configurável | Manual / ao autorizar NF / salvar NF / salvar pedido / **marcar como enviado**, global e por integração | OL, OD | ✅ "ao marcar enviado" | M | Parcial |
| EST-08 | Gatilho de entrada configurável | Manual / NF / OC / fim da conferência | OL, OD | ✅ manual | M | Nenhuma |
| EST-09 | Política de estoque negativo | Permitir, bloquear em qualquer depósito ou só no da venda | OL, OD, BL, OM | ✅ permitido | B | Nenhuma |
| EST-10 | Estoque de segurança / mín./máx. / % por canal | Publica menos do que o físico; máximo por canal; partilha percentual entre canais/contas | OL (só 1 valor por integração), HB (por anúncio/canal) | ✅ 3 por integração | M | Parcial |
| EST-11 | **Sync de estoque multiconta/multicanal** | O mesmo físico alimenta todos os anúncios (2 contas ML + Shopee…); recalcula e publica o disponível a cada venda | OL, OD, BL, HB, ML | ✅ | **A**: overselling, rate limit, endpoint diferente por tipo de conta (UP/multiorigem), 409 de versão | Total |
| EST-12 | Espelho do estoque Full | Saldo Full por `inventory_id`: disponível e indisponível (avariado, perdido, sem cobertura fiscal…) | ML, BL | ⚪ (o Olist só importa as NFs) | M (só leitura na API) | Total |
| EST-13 | Kardex Full (operações) | Recebimento, venda, cancelamento, devolução, retirada, transferência e ajuste no CD | ML, BL | ❌ no Olist | M | Total |
| EST-14 | Reconciliação de estoque Full × ERP | Compara o saldo Full do ML com o depósito Full do ERP e aponta divergências | ❌ em todos (dor citada no Olist) | ❌ | M | Total |
| EST-15 | Inventário | Gerar contagem, importar planilha, importar coletor (bipagem), conferência pelo celular, balanço | OL, OD, BL | ✅ ("Inventário Galpão") | M | Nenhuma |
| EST-16 | Custo médio ponderado | Recalculado a cada entrada; base do CMV e da margem | OL, OD, BL, OM | ✅ | M (dinheiro: sempre no servidor) | Nenhuma |
| EST-17 | Lotes e validade (FEFO/FIFO) | Lote na entrada, saída automática por validade, alertas | OL, OD, BL, OM | ⚪ desligado | M | Nenhuma |
| EST-18 | Localização física | Corredor, prateleira | OL, OD, BL | ❓ | B | Nenhuma |
| EST-19 | Alerta de estoque mínimo / ruptura | E-mail ou painel quando atinge o mínimo | OL, OD, BL, OM | ⚪ | B | Nenhuma |
| EST-20 | Retorno de devolução ao estoque (triagem) | Devolvido → depósito "Devoluções" → triagem (vendável/avaria/descarte) → transferência | OL (depósito), HB (Return Monitoring), ML | ✅ (depósito de devoluções) | M | Parcial |
| EST-21 | Estoque compartilhado multiempresa | Soma depósitos de vários CNPJs | OL, OD, BL | ⚪ não configurado | A (multi-tenant) | Parcial |
| EST-22 | Pausa/reativação automática por estoque | Estoque 0 pausa o anúncio; volta ativa | ML, HB, BL | ✅ (comportamento do ML) | B | Total |
| EST-23 | Estoque multiorigem do ML (`seller_warehouse`) | Contas com `warehouse_management` publicam por armazém | ML, BL | ❓ (tag da conta) | A | Total |

### 2.3 Vendas e pedidos

| ID | Feature | O que faz | Origem | Na sua conta hoje | Complexidade | Dependência de marketplace |
|---|---|---|---|---|---|---|
| VEN-01 | **Importação automática de pedidos (OMS)** | Webhook `orders_v2` → pack → orders → pedido idempotente no ERP; também Shopee e outros | OL, OD, BL, OM, HB, ML | ✅ 100% dos pedidos | **A**: idempotência, packs, eventos repetidos/fora de ordem, várias contas | Total |
| VEN-02 | Fila "Pedidos no e-commerce" | Pedido cru do canal × situação no ERP, por integração; reprocessar | OL, OM (pendentes/em revisão) | ✅ | M | Total |
| VEN-03 | Carrinho (pack) | 1 pack = N orders = 1 envio; NF única por carrinho | OL, OD, BL, ML | ✅ | M | Total |
| VEN-04 | Situações do pedido + mapeamento por canal | Em aberto, Aprovado, Preparando envio, Faturado, Pronto, Enviado, Entregue, Não entregue, Dados incompletos, Em devolução, Cancelado; mapa código do canal → situação | OL, OD, BL, OM | ✅ (11 situações) | M | Parcial |
| VEN-05 | Gerenciador de transições (workflow) | Cada mudança de situação dispara ações: estoque, contas, NF, separação, etiqueta | BL (completo), OL (Painel de Automações, 6 gatilhos) | ✅ 3 automações ligadas | **A**: motor de regras com efeitos colaterais | Parcial |
| VEN-06 | Pedido de venda manual | Cliente, itens, impostos por item, frete, desconto, pagamento, transportador, depósito, observações | OL, OD, BL, OM | ⚪ (tudo vem dos canais) | M | Nenhuma |
| VEN-07 | Marcadores (tags) automáticos | "Fulfillment", "1ª venda", "Etiqueta com data programada", "Shopee coleta"… | OL, OD | ✅ 9 marcadores | B | Parcial |
| VEN-08 | Data limite de despacho / SLA | Prazo máximo de despacho por pedido; alerta de atraso | OL, BL, ML (`/sla`) | ✅ (coluna existe) | B | Total |
| VEN-09 | Cancelamentos | Libera a reserva, cancela/estorna a NF e as contas | OL, OD, BL, ML | ✅ | M | Parcial |
| VEN-10 | **Devoluções de venda** | Registro (aberta → em andamento → finalizada), NF de devolução/entrada, estoque no depósito de devoluções, reembolso/vale-troca, vínculo com a reclamação do ML | OL, OD, BL, OM, ML | ✅ **822 registros** | M | Parcial |
| VEN-11 | Reclamações e mediações ML | Ver a reclamação, prazos (`due_date`), ações (reembolso, devolução), impacto na reputação | OL (pós-venda), BL (beta), ML | ✅ (pós-venda) | **A**: API de claims com muitos estados | Total |
| VEN-12 | Perguntas pré-venda | Caixa unificada multiconta, respostas padrão, notificação | OL, OD, BL, HB, ML | ✅ (Shopee com erro) | M | Total |
| VEN-13 | Mensagens pós-venda | Conversa por pack, anexos, bloqueios, agente de IA do ML intermediando (desde 02/02/2026) | OL, OD, BL, ML | ✅ | **A**: regras de moderação e nova intermediação | Total |
| VEN-14 | Feedback/avaliação de venda | Ler/responder avaliação | ML | ❓ | B | Total |
| VEN-15 | Descontos e cupons do pedido | Parte do vendedor × parte do ML, cupons | ML | ❓ | B | Total |
| VEN-16 | Alerta de fraude / risco de cancelamento | Sinaliza pedidos suspeitos | BL, HB (OCRS) | ❌ | M / A | Parcial |
| VEN-17 | Propostas comerciais (orçamentos) | Orçamento → pedido | OL, OD, BL, OM | ⚪ nenhuma | B | Nenhuma |
| VEN-18 | CRM / funil | Assuntos, estágios, ações | OL, OD, OM | ⚪ vazio | M | Nenhuma |
| VEN-19 | Comissões e metas de vendedores | Alíquotas, liberação, metas mensais | OL, OD, BL, OM | ⚪ zerado | M | Nenhuma |
| VEN-20 | PDV / frente de caixa | Venda balcão, caixa, NFC-e/SAT, maquininha | OL, OD, BL, OM | ⚪ caixa fechado | A (fiscal de consumidor/TEF) | Nenhuma |
| VEN-21 | Faturamento parcial / agrupar pedidos numa NF | Fatura parte do pedido; vários pedidos numa nota | OD, BL | ❓ | M | Nenhuma |
| VEN-22 | Ordens de serviço / contratos | Serviços, recorrência | OD, BL, OM | ⚪ | M | Nenhuma |

### 2.4 Fiscal

| ID | Feature | O que faz | Origem | Na sua conta hoje | Complexidade | Dependência de marketplace |
|---|---|---|---|---|---|---|
| FIS-01 | **Importação das NF-e emitidas pelo ML (Full)** | Baixa automaticamente XML/DANFE de venda, devolução, transferência (remessa/retorno simbólico) emitidas pelo Faturador; vincula ao pedido e ao depósito Full | OL, OD, BL, ML (`invoices`) | ✅ **~2.000 notas/mês** | M | Total |
| FIS-02 | Faturador do ML: envio de dados fiscais do produto | NCM, CSOSN, CEST, origem, GTIN por SKU para o ML faturar o Full; regras tributárias (`tax_rules`) | OL, OD, BL, ML | ✅ (modelo self storage CAT 59) | M | Total |
| FIS-03 | **Emissão de NF-e própria (modelo 55)** | Venda, devolução, remessa; série/numeração; cálculo de impostos; DANFE; envio ao cliente | OL, OD, BL, OM | ✅ pouco (série 4, nº ~867) | **A** se construída: motor tributário, SEFAZ por UF, contingência. **M** se integrada a um emissor por API | Nenhuma |
| FIS-04 | **Envio do XML/NF ao ML** (coleta/agência) | Sobe a NF quando o envio está `ready_to_ship/invoice_pending`, antes da etiqueta; automático ao autorizar | OL, OD, BL, HB | ✅ configurado | M (máquina de estados por substatus) | Total |
| FIS-05 | Emitir NF só depois que o ML libera a etiqueta | Evita NF antes da data programada | BL, OD | ✅ (marcador "data programada") | M | Total |
| FIS-06 | Natureza de operação + exceções | CFOP, CST/CSOSN, ICMS, ST, IPI, PIS/COFINS, DIFAL, FCP; exceções por UF/NCM/origem | OL, OD, BL, OM | ✅ 6 saídas + 2 entradas | **A** (núcleo do motor fiscal) | Nenhuma |
| FIS-07 | Certificado digital A1 | Upload/guarda segura, alerta de vencimento | OL, OD, BL, OM | ✅ (produção) | M (segredo sensível) | Nenhuma |
| FIS-08 | Cancelamento, carta de correção, inutilização, contingência | Eventos fiscais da NF-e | OL, OD, BL, OM | ✅ (menu existe) | A se construído | Nenhuma |
| FIS-09 | Emissão automática e em lote | Gerar e autorizar ao aprovar o pedido / em lote | OL, OD, BL | ⚪ desligada | M | Parcial |
| FIS-10 | Notas de entrada por XML | Upload unitário/ZIP; vínculo item ↔ produto (EAN, código do fornecedor); gera estoque e contas | OL, OD, BL, OM | ✅ (~1.000/mês, Full) | M | Nenhuma |
| FIS-11 | Buscador de NF-e (DF-e) + manifestação do destinatário | Baixa as notas emitidas contra o CNPJ; ciência/confirmação/desconhecimento | OL, OD, BL, OM | ❓ | **A**: DF-e/NSU, conflito com outro sistema | Nenhuma |
| FIS-12 | DANFE simplificado / etiqueta + DANFE unificada | Formatos de impressão para o pacote | OL, OD, BL | ⚪ (usa a etiqueta do ML) | M | Parcial |
| FIS-13 | NFC-e / SAT / NFS-e / CT-e / MDF-e | Outros documentos fiscais | OD, BL, OM | ⚪ | A | Nenhuma |
| FIS-14 | DIFAL / partilha / GNRE | Cálculo interestadual para não contribuinte; guias | OL, OD, BL | ❓ | A | Nenhuma |
| FIS-15 | Reforma tributária (IBS/CBS, cClassTrib) | Novos campos e cálculos a partir de 2026 | OL, OD, BL, OM | ⚪ não habilitada | A (regra em movimento) | Nenhuma |
| FIS-16 | SPED Fiscal / Contribuições / Sintegra | Obrigações acessórias | OD, BL, OM | ❓ (contador faz?) | A | Nenhuma |
| FIS-17 | Exportação para a contabilidade | XMLs do período (saída/entrada/canceladas/CC-e), relatórios; acesso do contador | OL, OD, BL, OM | ✅ (2 usuários da contabilidade) | B | Nenhuma |
| FIS-18 | Correção de rejeição com IA | Explica a rejeição da SEFAZ e guia a correção | OD | ⚪ | M (opcional) | Nenhuma |

### 2.5 Compras e fornecedores

| ID | Feature | O que faz | Origem | Na sua conta hoje | Complexidade | Dependência de marketplace |
|---|---|---|---|---|---|---|
| COM-01 | Ordem de compra | Fornecedor, itens, IPI/ST, condição de pagamento, frete CIF/FOB, situações, recebimento parcial | OL, OD, BL, OM | ⚪ zero | M | Nenhuma |
| COM-02 | Entrada por XML vinculada à OC | Importa a NF do fornecedor, casa itens, gera estoque e contas a pagar | OL, OD, BL, OM | ⚪ | M | Nenhuma |
| COM-03 | Conferência/check-in de recebimento | Bipa chave/itens, registra divergência/avaria | OL, OD, BL | ⚪ zero | M | Nenhuma |
| COM-04 | **Custo de reposição / formação de custo na entrada** | Custo = unitário + frete + ST + IPI + despesas − recuperáveis; atualiza custo médio e, opcionalmente, o preço | OL, OD, BL, OM | ❓ | M | Parcial |
| COM-05 | **Necessidade/sugestão de compra** | Média de vendas × cobertura − estoque − entradas previstas (+ mínimo, lead time) | OL, OD, BL, OM | ⚪ | M | Parcial |
| COM-06 | **Planejamento de reposição do Full** | Sugere quanto enviar ao CD por SKU/conta (cobertura, giro semanal, data limite); gera a NF de remessa. O envio é criado no painel do ML | OL (Envios Fulfillment), OD, HB (Ideris) | ⚪ vazio nos últimos 30 dias | **A**: previsão + fluxo misto ERP/ML | Total |
| COM-07 | Requisição e aprovação de compra | Pedido interno → aprovação por permissão | OM | ❌ | M | Nenhuma |
| COM-08 | Nota de importação | Custos de importação | OD, BL, OM | ❓ (produtos com origem 2?) | A | Nenhuma |

### 2.6 Financeiro

| ID | Feature | O que faz | Origem | Na sua conta hoje | Complexidade | Dependência de marketplace |
|---|---|---|---|---|---|---|
| FIN-01 | Contas a pagar | Fornecedor, vencimento, competência, categoria, recorrência, parcelas, baixa parcial/lote, anexos | OL, OD, BL, OM | ⚪ 7 atrasadas | M | Nenhuma |
| FIN-02 | Contas a receber | Títulos, baixa, previstos × recebidos | OL, OD, BL, OM | ⚪ vazio | M | Parcial |
| FIN-03 | Caixa e bancos / contas financeiras | Lançamentos, transferências, saldo por conta (Mercado Pago, banco) | OL, OD, BL, OM | ⚪ parado | B | Nenhuma |
| FIN-04 | Categorias de receita/despesa (plano de contas gerencial) | Com flag de DRE/CMV | OL, OD, BL, OM | ✅ ~10 (quase padrão) | B | Nenhuma |
| FIN-05 | Centro de custo / projeto | Rateio de despesas | OM | ❌ | M | Nenhuma |
| FIN-06 | **Taxas do marketplace no pedido** | Grava comissão, custo fixo, frete do vendedor, descontos e impostos em cada venda | BL (automático), OD ("Custos do e-commerce") | ⚪ tela vazia | **A**: composição vem de várias fontes da API | Total |
| FIN-07 | **Conciliação de repasses ML/Mercado Pago** | Compara o líquido esperado por pedido com o liberado no MP (a liberar, retido, reembolso, contestação) e aponta a diferença | BL (Conferência de Recebíveis), OM (TrackCash, terceiro), HB, ML/MP | ❌ **o Olist não tem** | **A**: relatórios MP assíncronos, casamento por payment_id | Total |
| FIN-08 | Conciliação das faturas do ML (billing) | Confere o que o ML cobrou (comissões, envios, Full, Ads) por período | ML | ❌ | A | Total |
| FIN-09 | Conciliação bancária (OFX / Open Finance) | Importa extrato, casa com títulos | OL, OD, BL, OM (Open Finance) | ⚪ vazio | M / A (Open Finance) | Nenhuma |
| FIN-10 | **DRE gerencial (por canal, conta, produto)** | Receita − impostos − CMV − taxas − frete − despesas, por competência | OL, OD, BL, OM | ⚪ | M | Parcial |
| FIN-11 | Fluxo de caixa (projeção) | A pagar + a receber + repasses futuros | OL, OD, BL, OM | ⚪ | M | Parcial |
| FIN-12 | Boletos / CNAB / Pix / link de pagamento | Cobrança bancária | OL, OD, BL, OM | ⚪ | A | Nenhuma |
| FIN-13 | Custos do Full (armazenagem, retirada) | Via detalhes de billing | ML, HB | ❌ | M | Total |
| FIN-14 | Simulador de custo por vender | Comissão + custo fixo + frete por categoria/preço/tipo/logística antes de publicar | ML, OD (calculadora), HB | ⚪ | M | Total |
| FIN-15 | Período fechado / trava de lançamentos | Bloqueia alterações fora da competência | OL ("fechamento financeiro"), OM | ⚪ | B | Nenhuma |

### 2.7 Marketplaces / hub

| ID | Feature | O que faz | Origem | Na sua conta hoje | Complexidade | Dependência de marketplace |
|---|---|---|---|---|---|---|
| MKT-01 | **Conexão OAuth multiconta (token broker)** | Login de cada conta pelo **admin**; guarda tokens criptografados; renova o refresh (uso único) com **trava por conta**; detecta revogação e pede reconexão | OL, OD, BL, HB, ML | ✅ 1 conta | **A**: refresh de uso único + concorrência; **compartilhado com o agente** | Total |
| MKT-02 | **Receptor de webhooks + fila de eventos** | Endpoint que só valida e enfileira (200 em < 500 ms); workers fazem GET no recurso; dedupe; roteia por `topic`/`user_id` | ML, HB | ✅ (implícito no Olist) | **A**: núcleo da arquitetura; **o agente consome esta fila** | Total |
| MKT-03 | Recuperação de eventos perdidos + reconciliação | `missed_feeds` (≤ 2 dias) + polling periódico (pedidos, envios, claims, perguntas) | ML | ❓ | M | Total |
| MKT-04 | Gestão de anúncios | Lista por canal/conta, status, importar, editar (gerais, ficha, preço), pausar/ativar em lote, moderações | OL, OD, BL, HB, ML | ✅ ~116 anúncios | **A**: ciclo de vida e erros da API | Total |
| MKT-05 | **Vínculo anúncio ↔ produto (mapeamento de SKU)** | Por `SELLER_SKU`/GTIN/título; um produto → N anúncios em N contas; indicador "mapeamento com erro" | OL, OD, HB, ML | ✅ (erros na Shopee) | M | Total |
| MKT-06 | Publicação de anúncio a partir do produto | Categoria, atributos obrigatórios, fotos, descrição, tipo (Clássico/Premium), logística, garantia | OL, OD, BL, HB, ML | ✅ | **A** | Total |
| MKT-07 | Replicação de anúncio entre contas | Copia da conta A para a B com regras de diferenciação (título, tipo, frete) | BL, HB (Ideris, Magis5), OD (Hub) | ❌ | M (risco de política de duplicados) | Total |
| MKT-08 | Criação/edição em massa | Planilha ou lote para preço, estoque, imagens, campanhas | OL, BL, HB | ⚪ | M (fila + rate limit) | Total |
| MKT-09 | **Sync de preço + regras de preço por canal/conta** | Preço fixo, lista, markup/margem por canal, preço promocional "de/por", detecção de automatização de preço do ML | OL, OD, BL, HB, ML | ✅ preço fixo | M | Total |
| MKT-10 | **Calculadora de preço / proteção de margem** | Comissão e frete automáticos do ML, margem desejada, custos extras; bloqueia preço abaixo do piso | OD (Impulsione+), HB (Price Guard), ML | ⚪ | M | Total |
| MKT-11 | Central de promoções do ML | Aceitar campanhas, ofertas relâmpago, co-participação, cupons | BL, HB, ML | ❌ no Olist | A (muitos tipos) | Total |
| MKT-12 | Campanhas promocionais do ERP | Desconto % com período, aplica e reverte preço | OL, OD | ⚪ nenhuma | M | Total |
| MKT-13 | Catálogo do ML | Elegibilidade, optin, competição (`price_to_win`) | OD, BL, HB, ML | ❓ | A | Total |
| MKT-14 | Kit virtual nativo do ML | Kit publicado a partir de UPs (2–6 produtos) | BL, HB, ML | ❓ | A | Total |
| MKT-15 | Saúde dos anúncios / qualidade / moderações | Infrações, perda de exposição, qualidade | OD, BL, HB, ML | ⚪ | M | Total |
| MKT-16 | Monitor de integração | Painel de erros por anúncio/pedido, retentativas, alertas (ex.: "token Shopee expirou") | HB (Anymarket) | ❌ (o erro aparece solto na tela) | M | Parcial |
| MKT-17 | Mapeamentos (categorias, variações, formas de frete, recebimento) | De-para ERP ↔ canal | OL, OD | ✅ | M | Total |
| MKT-18 | Sugestão de preço do ML | Lê o preço sugerido | ML, OL (beta) | ⚪ | B | Total |
| MKT-19 | Product Ads (leitura) | Métricas de anúncios patrocinados | ML | ❌ | M | Total |
| MKT-20 | Reputação e métricas de atendimento | Termômetro, MercadoLíder, tempo de resposta | ML | ❌ no Olist | B | Total |
| MKT-21 | Outros canais (Shopee, Amazon, Magalu, TikTok, Temu…) | Conector por canal com a mesma camada de abstração | OL, OD, BL, OM, HB | ✅ Shopee; ❓ demais | A (cada canal) | Total |

### 2.8 Logística

| ID | Feature | O que faz | Origem | Na sua conta hoje | Complexidade | Dependência de marketplace |
|---|---|---|---|---|---|---|
| LOG-01 | **Separação (picking)** | Lista por pedido/produto, ordenada por localização; abas aguardando → em separação → separadas; trava por operador | OL, OD, BL, HB | ⚪ vazia (automação ligada) | M | Parcial |
| LOG-02 | **Embalagem/checkout com bipagem** | Bipa SKU/GTIN, confere, gera NF/etiqueta ao finalizar; checkout por produto único | OL, OD, BL, HB | ❓ | M | Parcial |
| LOG-03 | **Etiquetas ME2 PDF/ZPL em lote** | `shipment_labels` (≤ 50 por chamada), impressão térmica; DANFE simplificado junto | OL, OD, BL, HB, ML | ✅ (etiqueta do ML) | B / M | Total |
| LOG-04 | Expedição / agrupamento de coleta | Agrupa pacotes por forma de envio, romaneio, etiqueta única do lote (bundle ML) | OL, OD, BL, OM | ⚪ sem agrupamentos | M | Parcial |
| LOG-05 | Coleta e agência (cross_docking / drop_off / xd_drop_off) | Fluxo por tipo logístico | OL, OD, BL, ML | ✅ | M | Total |
| LOG-06 | Mercado Envios Flex | Zonas, horário de corte, limite diário, ativar por anúncio | OD, BL, ML | ❓ | M | Total |
| LOG-07 | Full (operação) | Pedidos Full, saldo Full, anúncios Full pausados, envios de reposição | OL, OD, BL, HB, ML | ✅ forte | M | Total |
| LOG-08 | Rastreio e status de envio | Histórico do envio, atraso, entregue | OL, OD, BL, ML | ✅ | B | Total |
| LOG-09 | Devoluções / logística reversa | Status da devolução no ML, revisão do produto, custo da devolução | OL, OD, BL, HB, ML | ✅ | M | Total |
| LOG-10 | Horários de despacho e capacidade | Dias sem envio, capacidade diária | OL, OD, BL, ML | ⚪ não cadastrado | B | Total |
| LOG-11 | Correios/transportadoras/gateways de frete | Contrato próprio, cotação, etiquetas | OL, OD, BL, OM | ⚪ (Olist Envios com crédito, uso marginal) | A (N APIs) | Nenhuma |
| LOG-12 | Impressão direta (PrintNode/Zebra) | Imprime sem diálogo, na térmica | OL, OD, HB | ❓ | M | Nenhuma |
| LOG-13 | Split de pacote | Divide um envio em mais volumes | ML | ❓ | B | Total |

### 2.9 Relatórios e BI

| ID | Feature | O que faz | Origem | Na sua conta hoje | Complexidade | Dependência de marketplace |
|---|---|---|---|---|---|---|
| BI-01 | **Margem de contribuição por pedido / SKU / anúncio / conta / canal** | Faturamento − frete − comissões − taxas − custo − impostos (+ incentivos) | OL (vazio), OD, BL, HB | ⚪ zerada | **A**: depende de FIN-06/07 | Parcial |
| BI-02 | Dashboard de vendas por canal/conta | Faturamento, pedidos, ticket, por conta e por logística | OL, OD, BL, HB | ✅ (gráfico de 7 dias) | M | Parcial |
| BI-03 | Curva ABC | Produtos/clientes por valor/quantidade | OL, OD, BL, HB | ⚪ | B | Parcial |
| BI-04 | Giro de estoque / cobertura / previsão | Dias de cobertura, baixo giro, parado | OL, OD, BL | ⚪ | B | Nenhuma |
| BI-05 | Ruptura / abaixo do mínimo / sem movimentação | Exceções de estoque | OL, OD, BL, HB | ⚪ | B | Nenhuma |
| BI-06 | Relatórios de exceção do canal | Produtos sem anúncio, Full pausado, estoque infinito, erros de mapeamento | HB (Magis5) | ❌ | B | Parcial |
| BI-07 | Relatório de devoluções (motivos, custo, por SKU) | Por que devolvem e quanto custa | ❌ em todos (só listas) | ❌ | M | Parcial (liga com os insights do agente) |
| BI-08 | Vendas por período/UF/horário | Mapa e picos | OD, HB | ⚪ | M | Parcial |
| BI-09 | Relatório personalizado / construtor | Colunas, filtros e agrupamentos livres; "criar com IA" | OL, OD, BL | ✅ ("Inventário Galpão") | A (query builder seguro) | Nenhuma |
| BI-10 | Saldos em data passada / visão financeira do estoque | Estoque valorizado em qualquer data | OL, OD, BL | ⚪ | M (exige ledger) | Nenhuma |
| BI-11 | Exportação para BI externo / MCP | API e conector MCP para IA | BL, OM | ❌ | M | Nenhuma |

### 2.10 Outros (plataforma, segurança, integrações, automações)

| ID | Feature | O que faz | Origem | Na sua conta hoje | Complexidade | Dependência de marketplace |
|---|---|---|---|---|---|---|
| OUT-01 | Usuários e perfis (RBAC) | Permissões por tela e por ação (incluir/editar/excluir) + ações especiais | OL, OD, BL, OM | ✅ 5 usuários / perfil "Logística" | M | Nenhuma |
| OUT-02 | Acesso do contador | Perfil somente leitura/exportação | OL (perfil gerente), OM (Painel do Contador) | ✅ 2 usuários | B | Nenhuma |
| OUT-03 | Restrições de acesso / 2FA | TOTP, horário, IP, sessão única | OD, BL, OM | ❓ | M | Nenhuma |
| OUT-04 | **Auditoria geral** | Quem mudou preço, estoque, anúncio, pedido; histórico de exclusões | OD (só estoque), BL, OM | ⚪ parcial | M | Nenhuma |
| OUT-05 | Painel de automações | Gatilhos configuráveis (ver VEN-05) | OL, OD, BL | ✅ 3 ligadas | A | Parcial |
| OUT-06 | API própria + webhooks de saída | Para integrar outros sistemas (o **agente** inclusive) | OL, OD, BL, OM | ⚪ desligados | M | Nenhuma |
| OUT-07 | MCP do ERP | Expor o ERP para assistentes de IA | BL, OM | ❌ | M | Nenhuma |
| OUT-08 | Central de notificações / alertas | Alertas de erro de integração, SLA, estoque | OL, OD, OM | ✅ | B | Parcial |
| OUT-09 | Assistente de IA no ERP | Chat que consulta e executa ações | OL (Lis), BL | ⚪ | A (opcional) | Nenhuma |
| OUT-10 | Multiempresa (vários CNPJs) | Alternar empresas, compartilhar cadastros/estoque | OL, OD, BL, OM | ⚪ não configurado | A | Parcial |
| OUT-11 | Backup / exportação de dados | Exportação completa | OL, OD, BL | ❓ | B | Nenhuma |
| OUT-12 | Envio de documentos (e-mail/WhatsApp) | DANFE, boleto, rastreio ao cliente | OL, OD, BL | ❓ | B | Nenhuma |
| OUT-13 | Loja de extensões / feature flags | Ligar/desligar módulos | OL, OD | n/a | M | Nenhuma |
| OUT-14 | Tema claro/escuro, atalhos de teclado | UX | OL | ✅ | B | Nenhuma |
| OUT-15 | **Migração do Olist** | Importar produtos, kits, contatos, anúncios, saldos, histórico de pedidos/NFs (o ML só guarda 12 meses de pedidos) | ❌ | — | M | Parcial |

---

## 3. Integração com o agente de atendimento (o que o ERP precisa expor)

O documento do agente (`pinese/docs/01-documento-do-agente.md`) usava o Olist como fonte das especificações. A partir de agora, essa fonte é o ERP. Features do mapa que viram **contrato com o agente**:

| Necessidade do agente | Feature do ERP | Observação |
|---|---|---|
| Login das contas ML, tokens, renovação | MKT-01 | **1 app ML por titular**: o ERP é o dono; o agente nunca renova token sozinho. |
| Receber perguntas, mensagens e reclamações | MKT-02, MKT-03 | O webhook é único (1 callback por app). O ERP enfileira e o agente consome os tópicos `questions`, `messages` e `post_purchase`. |
| Ficha do produto (medidas, materiais, o que acompanha, manual PDF, vídeo, garantia) | CAD-09, CAD-10, CAD-11 | **CAD-10 não existe no Olist** e é um diferencial. |
| Produto ↔ anúncios nas 2 contas | CAD-06, MKT-05 | O agente recebe um `item_id` e precisa chegar ao SKU. |
| Status do pedido e do envio | VEN-01, LOG-08 | Resposta de "onde está meu pedido". |
| Reclamações, devoluções, prazos | VEN-10, VEN-11, LOG-09 | A janela de 48 h e o `due_date` vêm da mesma camada. |
| Políticas da loja (garantia de 30 dias, troca, nunca enviar peça) | (novo) OUT-xx Configurações de política | Ficam no ERP ou no agente? Pergunta da entrevista. |
| Insights de devolução/dúvida → anúncio | BI-07 | O agente gera os insights e o ERP mostra por SKU. |

---

## 4. Restrições técnicas do Mercado Livre que moldam o ERP

| # | Restrição (fonte: devsite ML, out/2026) | Consequência no desenho |
|---|---|---|
| R1 | Access token de 6 h; **refresh token de uso único** (só o último vale), validade de 6 meses; só o **admin** da conta autoriza | Token broker central com trava por conta. O agente nunca renova. Alerta de reconexão. |
| R2 | Webhook precisa responder **200 em ≤ 500 ms**, senão os tópicos são desativados; retentativas por 1 h; `missed_feeds` guarda 2 dias | Receptor mínimo → fila persistente → workers; reconciliação diária por polling. |
| R3 | **Sem sandbox**: só até 10 usuários de teste, que só negociam entre si | Testes contra a produção com contas de teste + gravação de respostas para testes automatizados. |
| R4 | Modelo **User Products** (preço por variação, `family_name`, sem `variations` para quem tem a tag) | O schema nasce como Produto → UP → N anúncios. |
| R5 | Estoque: `PUT /items` (conta comum) **ou** `seller_warehouse` (multiorigem, com `x-version`) | Roteador de escrita por tipo de conta; consultar as tags reais das suas contas. |
| R6 | **Estoque Full só leitura**; envio de reposição (inbound) só pelo painel do ML | O ERP planeja e emite a NF de remessa, mas não cria o envio. |
| R7 | **Full: NF-e obrigatoriamente pelo Faturador do ML**; coleta/agência: NF própria + XML no `invoice_pending` | Fiscal em "duas vias". |
| R8 | Pedidos ficam na API só **12 meses** | O ERP é a fonte histórica (BI, DRE, fiscal). |
| R9 | Mensagens: 350 caracteres; o vendedor não inicia; automático/repetitivo é proibido; **agente de IA do ML intermediando desde 02/02/2026** | Regras do agente e do ERP (fila de 1 mensagem por conversa). |
| R10 | Política de **duplicados**: o mesmo produto com as mesmas condições em mais de um anúncio pode ser penalizado; ações em massa e clonagem são desaconselhadas | Replicar o catálogo entre as 2 contas exige diferenciação. **A validar com você.** |
| R11 | Rate limits por recurso (mensagens 500 rpm, estoque UP 100 rpm, Flex 1000 rpm); limite global não documentado | Cliente HTTP com token bucket por conta/recurso e backoff. |
| R12 | Endpoint oficial de upload do XML de NF própria **não localizado** na doc (o mercado usa `/shipments/{id}/invoice_data`) | Validar em conta de teste antes de prometer. |

---

## 5. Dores conhecidas do Olist (para a entrevista)

As fontes são reclamações públicas (trechos de busca), limitações documentadas e sinais da sua conta.
1. **Estoque errado/zerado automaticamente** e **Full somado com o Galpão** nas reservas.
2. **Saldo do anúncio no ML diverge do ERP**; sincronização por intervalo (as fontes divergem).
3. **Pedidos do Full duplicados** e falhas na importação de NFs.
4. **Mudanças na integração ML sem aviso** (migração para o "Hub da Olist").
5. Instabilidade (504), suporte que piorou depois da aquisição.
6. **Recursos importantes só nos planos caros** (OMS, margem, custos do e-commerce, separação em lote, devoluções); novo modelo cobra por armazenamento e **bloqueia a escrita** ao atingir o limite.
7. **Estoque de segurança único por integração**, não por anúncio/conta.
8. **Sem conciliação de repasses do ML** e sem centro de custo.
9. Na sua conta: **erro de token da Shopee**, **"mapeamento com erro"** em produtos, opções de variação duplicadas/com erro de digitação ("Ganito", "Peto", "127 V").

---

## 6. Pontos de decisão que a entrevista precisa fechar

1. Onde está a **2ª conta ML** e se o catálogo repetido está em risco (R10).
2. **Quem emite a NF** das vendas fora do Full (ERP, Faturador do ML, ou ambos) → **construir × integrar emissor NF-e por API**.
3. **Shopee/Amazon/Magalu/TikTok**: estão ativos? Entram no MVP ou na Fase 2?
4. Como funciona hoje: **entrada de mercadoria**, **financeiro** (fora do Olist?), **controle de margem**.
5. **Fluxo de devolução** (822 registros): o que acontece fisicamente e no sistema.
6. O que a **contabilidade** extrai do ERP.
7. ERP e agente: **mesmo banco/painel ou sistemas separados**?
8. **Stack e monorepo** (Next.js na Vercel + Supabase Cloud, igual ao agente?).

---

## 7. Totais do mapa

| Módulo | Features | Usadas hoje (✅) | Existem, sem uso (⚪) | Não existem no Olist (❌) | A confirmar (❓) |
|---|---|---|---|---|---|
| Cadastros | 31 | 13 | 12 | 3 | 3 |
| Estoque | 23 | 15 | 4 | 2 | 2 |
| Vendas e pedidos | 22 | 12 | 6 | 1 | 3 |
| Fiscal | 18 | 10 | 5 | 0 | 3 |
| Compras | 8 | 0 | 5 | 1 | 2 |
| Financeiro | 15 | 1 | 10 | 4 | 0 |
| Marketplaces / hub | 21 | 8 | 5 | 5 | 3 |
| Logística | 13 | 5 | 4 | 0 | 4 |
| Relatórios / BI | 11 | 2 | 6 | 3 | 0 |
| Outros | 15 | 5 | 4 | 1 | 5 |
| **Total** | **177** | **71** | **61** | **20** | **25** |

> **Leitura rápida:** você usa de fato **cerca de 40% do Olist**, quase todo concentrado em cadastro, estoque multidepósito, importação de pedidos, Full (notas e estoque), devoluções e atendimento. **Financeiro, compras e margem** existem no Olist mas estão parados. As maiores oportunidades do ERP próprio estão justamente aí e no que o Olist **não tem**: conciliação de repasses, margem real por anúncio/conta, base de conhecimento para o agente, reconciliação do Full e regras de estoque por conta.
