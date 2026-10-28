# Entrevista de escopo: respostas registradas

> Registro bruto das rodadas. Alimenta o `02-contexto-erp.md`.

## Rodada 1: operação (27/10/2026)

| Pergunta | Resposta |
|---|---|
| Onde está a 2ª conta ML? | **São 3 contas ML no total**; só 1 está no Olist por enquanto. |
| Quem emite a NF fora do Full? | **Todas as NF são emitidas pelo marketplace** (Faturador do ML). O **sonho é emitir tudo pelo ERP**. |
| Canais que vendem de verdade | **Só Mercado Livre** (Full + coleta). Shopee, Amazon, Magalu e TikTok estão integrados, mas não vendem. |
| Controle de financeiro/margem | **Mercado Turbo** hoje; quer **controlar tudo pelo ERP**. |

Implicações:
- O multiconta é real desde o dia 1, com **3 contas ML**, não 2.
- Fiscal: hoje 100% Faturador ML. Conflito a tratar: no **Full o Faturador é obrigatório** (doc ML), então "emitir tudo pelo ERP" só vale para coleta/agência/Flex.
- Margem: o ERP precisa cobrir o que o Mercado Turbo entrega (concorrente direto a substituir).
- Os outros canais saem do MVP, mas a arquitetura continua multicanal.

## Rodada 2: contas, impostos e rotina (27/10/2026)

| Pergunta | Resposta |
|---|---|
| CNPJs das contas | **Cada conta tem um CNPJ.** |
| Regime | Correção: **são 4 contas ML / 4 CNPJs**. 3 no Simples Nacional, **1 MEI** (todas devem ir para o Simples). O ERP deve ser **multiconta sem limite** ("quantas contas eu quiser ter ativas"). |
| Uso do Mercado Turbo | Margem/lucro por venda e anúncio · repasses/financeiro do Mercado Pago · métricas de anúncio/concorrência · **fluxo de caixa, contas a pagar, contas a receber, DRE e BP** (balanço patrimonial). |
| Rotina diária | Separar/embalar e imprimir etiquetas · preparar reposição ao Full · tratar devoluções e conferir estoque. **Impressora térmica TSC TE200** (precisa de integração). Perguntas, mensagens e reclamações ficam com a **IA de atendimento** (projeto paralelo); no ERP só interessa um **módulo de mensageria/Kanban** para acompanhar as conversas. |

Implicações:
- **Multiempresa (multi-CNPJ) é requisito do MVP**: 4 empresas, cada uma com certificado, numeração de NF, regime e conta ML próprios. O que no Olist estava "sem uso" (multiempresa) vira núcleo.
- MEI: limites de faturamento e regras fiscais próprias (CSOSN, sem destaque de imposto); o desenho fiscal precisa suportar MEI e Simples.
- O Mercado Turbo vira o benchmark de financeiro e BI. BP (balanço) é contábil: avaliar se é gerencial ou se fica com o contador.
- TSC TE200: linguagem nativa TSPL; o ML entrega etiquetas em PDF ou ZPL2. **Verificar se a TE200 emula ZPL**; se não, imprimir PDF via driver ou converter.
- Atendimento sai do ERP, mas a camada ML (webhooks, tokens) continua no ERP, junto com um Kanban de conversas para acompanhamento.

## Rodada 3: estoque entre empresas, compras, catálogo, contábil (27/10/2026)

| Pergunta | Resposta |
|---|---|
| Estoque físico | **Um galpão para todas as empresas.** Boa parte dos produtos fica no Full, mas sempre com estoque também no galpão. |
| Entrada de mercadoria | **Misto**: às vezes cada CNPJ compra com NF própria; outras vezes há **transferência entre empresas sem NF**, lançada como **perda no estoque** da empresa de origem. |
| Catálogo | **Parcialmente igual** entre as contas. |
| DRE/BP | **Contábil completo no ERP.** O contador continua, mas o ERP deve produzir a contabilidade para conferir os números do contador. |

Implicações:
- O estoque precisa de **duas dimensões**: local físico (Galpão, Full da conta X) × **empresa proprietária** (CNPJ). Um galpão físico com 4 saldos fiscais.
- **Risco fiscal/contábil**: transferir mercadoria entre CNPJs sem NF e lançar como perda deixa a empresa de destino com estoque sem entrada documentada e a de origem com perda que não aconteceu. Isso aparece em fiscalização e quebra a contabilidade "sistemática" que você quer comparar com a do contador. Recomendação: o ERP gerar a **NF de transferência/venda entre empresas** (com um clique); tratar com o contador.
- Contábil completo = livro razão em partidas dobradas, plano de contas por empresa, BP, DRE contábil, integração automática dos eventos (venda, compra, repasse, taxa). Escopo grande.

## Rodada 4: emissor NF-e, transferências, contábil, reposição Full (27/10/2026)

| Pergunta | Resposta |
|---|---|
| Emissor NF-e | **Emissor próprio**, sem depender de API de terceiros, mas **na Fase 2**. O MVP é urgente por dois motivos: (1) a IA de atendimento precisa consumir os produtos cadastrados no ERP; (2) abandonar o Tiny/Olist por completo. |
| Transferência entre empresas | **Os dois modos, escolhidos a cada vez** (NF entre empresas ou movimentação interna sem NF), com marcação do que ficou sem documento. |
| Contábil completo | **Base no MVP** (todo evento já gera lançamento em partidas dobradas); **DRE/BP contábil e conferência com o contador na Fase 2**. |
| Reposição do Full | **O ERP deve sugerir** (giro e cobertura). |

Implicações:
- MVP fiscal = continuar no Faturador do ML (Full e coleta) e **importar** todas as notas; o ERP alimenta o Faturador (dados fiscais e regras tributárias por empresa).
- Emissor próprio (Fase 2): NF-e 55 com assinatura A1, transmissão SEFAZ (SP), eventos (cancelamento, CC-e, inutilização), contingência, DANFE, multi-CNPJ (4 certificados/séries).
- Há dois "clientes" do MVP: o agente de IA (precisa de cadastro + base de conhecimento + camada ML) e a saída do Olist (precisa de pedidos, estoque, Full, etiquetas, devoluções).

## Rodada 5: arquitetura e perfis (27/10/2026)

| Pergunta | Resposta |
|---|---|
| ERP + agente | **Mesmo monorepo, mesmo banco, painel único.** |
| Hospedagem | **Tudo Vercel + Supabase Cloud.** |
| Fatiar o MVP | **MVP único, modular.** Ficam fora do MVP só a **emissão de notas** e os **relatórios contábeis complexos**. Ordem: **primeiro o ERP, depois o agente**. Processo: **cada módulo é prototipado (~10 versões em HTML atômico/isolado)** antes de qualquer código. |
| Perfis | **Dono/admin · Operador de logística · Contador (leitura + exportação) · "Tributário"**: perfil próprio que gera a NF de devolução, analisa as NFs e o fechamento contábil, sem editar mais nada. |

Implicações:
- Workers sem servidor próprio: fila em Postgres (pgmq) + **pg_cron** + Supabase Edge Functions ou Vercel Cron/Functions para consumir. Atenção ao limite de duração das funções e à latência de 500 ms do webhook (o receptor só grava e responde).
- O perfil Tributário gera NF de devolução, mas a emissão própria é Fase 2. No MVP isso significa importar/conferir a devolução emitida pelo ML; a geração vem com o emissor.
- RBAC com 4 perfis, mais escopo por empresa (CNPJ).

## Rodada 6: Cadastros e Estoque (27/10/2026)

| Pergunta | Resposta |
|---|---|
| Saldo que cada conta usa | **O mesmo estoque do galpão para todas as contas**, mas com **ponto de pedido por conta** para repor o galpão. **Conta principal: "Mercado Mestre"**; as outras são secundárias. Objetivo das multicontas: **pagar menos imposto e estar mais vezes anunciado.** |
| Partilha do saldo entre contas | **Mesmo saldo em todas, menos o estoque de segurança.** |
| Proposta de Cadastros | **Aprovada.** |
| Proposta de Estoque | **Aprovada.** |

Implicações:
- Estoque físico único (pool), com a **propriedade fiscal por CNPJ** rastreada em paralelo. Quando uma conta secundária vende uma unidade que é de outra empresa, o ERP registra a transferência (com NF ou interna, conforme a escolha) para que estoque, fiscal e contábil fechem.
- Demanda e ponto de pedido calculados **por conta**, consolidados na reposição do galpão.
- Risco a registrar (para o contador/advogado): dividir o faturamento de uma operação única entre vários CNPJs do mesmo dono, com estoque e galpão compartilhados, pode ser enquadrado pela Receita como **grupo econômico de fato / fracionamento** (exclusão do Simples com cobrança retroativa). Pelo lado do ML, há a **política de anúncios duplicados entre contas vinculadas**. O ERP não resolve isso, mas registrar corretamente a propriedade e as transferências de cada empresa é o que dá sustentação documental.

## Rodada 7: Vendas e Fiscal (27/10/2026)

| Pergunta | Resposta |
|---|---|
| Kanban de mensageria | **Painel completo de atendimento**: caixa de entrada com histórico, contexto do pedido, anexos e métricas de SLA. |
| Venda fora do marketplace | Ainda não existe. **Futuro: B2B, licitação, site próprio.** |
| Proposta de Vendas | **Aprovada.** |
| Proposta de Fiscal | **Aprovada.** |

Processo definido pelo cliente (depois da validação do contexto): protótipo HTML **modular**, um módulo por vez. Primeiro **layout base + página de login + sidebar**; com aprovação, vai para o 1º módulo, e assim por diante até completar o MVP do ERP.

Implicações:
- O "painel de atendimento" vive no ERP (painel único) e é a interface do agente de IA: inbox, contexto, aprovação de sugestões, SLA.
- Novos canais (B2B, licitação, site próprio) confirmam a arquitetura multicanal; pedido manual, propostas e listas de preço ficam na Fase 2/3.

## Rodada 8: Compras e Financeiro (27/10/2026)

| Pergunta | Resposta |
|---|---|
| Contas financeiras | A princípio **1 Mercado Pago por empresa + 1 banco por empresa**; talvez **1 banco único para todas** e um MP por empresa. → O ERP deve permitir conta bancária compartilhada entre empresas ou exclusiva de uma. |
| Despesas comuns | **Lançar tudo na principal (Mercado Mestre).** Sem rateio. |
| Proposta de Compras | **Aprovada.** |
| Proposta de Financeiro | **Aprovada.** |

Implicações:
- Se o banco for único para várias empresas, o ERP precisa identificar a empresa dona de cada lançamento (risco de confusão patrimonial; levar ao contador).
- O DRE das secundárias fica sem despesas fixas; o consolidado continua correto.

## Rodada 9: Marketplaces e Logística (27/10/2026)

| Pergunta | Resposta |
|---|---|
| Publicar e replicar anúncios | **Publicar + replicar entre contas no MVP.** |
| Métricas de anúncio/concorrência | **Tudo no MVP** (desligar o Mercado Turbo junto com o Olist). |
| Aprovações de proposta | **Não perguntar mais "aprova?".** As perguntas de contexto já foram respondidas (e outras podem ser feitas se precisar), mas aprovar é **vendo e desenvolvendo um módulo de cada vez em HTML**, antes de código e tabelas no BD. |
| Flex | Perguntou o que é. Explicado: Mercado Envios Flex = o vendedor entrega no mesmo dia/dia seguinte na própria região (motoboy próprio ou terceirizado), com selo "Chega hoje/amanhã". Classificado na **Fase 2**, como pergunta em aberto. |

Implicações:
- MVP ganha **publicação + replicação multiconta** com regras de diferenciação (título, tipo de anúncio, frete) por causa da política de duplicados.
- Métricas no MVP limitadas ao que a API oficial dá (visitas, conversão, Product Ads, reputação, `price_to_win` em catálogo). **Não há dados de concorrentes fora do catálogo sem scraping**, e scraping é proibido pelas regras do ML.
- Classificação de BI e Outros feita pelo consultor, sem rodada de aprovação (ver 02).

## Respostas às perguntas em aberto (27/10/2026)

| # | Pergunta | Resposta |
|---|---|---|
| 1 | UF/endereço das empresas | **Todas no endereço do galpão.** O endereço será informado depois. |
| 2 | MEI no Full/Faturador | **Não por enquanto.** |
| 3 | Contas vinculadas no ML | **Não são vinculadas**; nunca houve aviso de duplicidade. |
| 4 | Banco por empresa ou único | **Pendente**: o cliente vai perguntar ao contador. |
| 5 | Histórico a migrar do Olist | **Todo o histórico.** |
| 6 | Plano de contas | **Pendente**: o cliente vai perguntar ao contador. |
| 7 | NF de devolução / tributário | O tributário **confere todas as NF emitidas, de todas as plataformas**. |
| 8 | Flex | **Em aberto.** |
| 9 | Políticas da loja | **Ficam no ERP**; já estão mapeadas nos documentos do agente. |
| 10 | Logins por perfil | **Usuários e contas ilimitados, criados pelo admin na interface.** |
| 11 | TSC TE200 | **Imprime ZPL.** As etiquetas do ML saem em ZPL direto para a impressora. |
