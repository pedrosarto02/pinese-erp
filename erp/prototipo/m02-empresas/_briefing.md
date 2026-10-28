# Briefing: Módulo 2 · Empresas e contas ML

Leia antes de escrever qualquer coisa: `C:\Users\tocub\pinese\erp\prototipo\README.md`, `C:\Users\tocub\pinese\erp\prototipo\pinese-erp.html` (shell aprovado; **é a base de cada versão**) e, para contexto, as seções 2, 5 e 6 de `C:\Users\tocub\pinese\erp\docs\02-contexto-erp.md`.

## Contexto do negócio
ERP próprio de um vendedor do Mercado Livre: **4 empresas (CNPJs), cada uma com 1 conta ML**, todas operando do **mesmo galpão**. As contas e empresas são **ilimitadas**, criadas pelo admin (perfil Dono) na interface.
- **Mercado Mestre**: conta **principal**, Simples Nacional. As despesas comuns são lançadas nela.
- **Conta 2**: secundária, Simples Nacional, conectada.
- **Conta 3**: secundária, Simples Nacional, **token expirado: precisa reconectar**.
- **Conta 4**: secundária, **MEI** (vai migrar para o Simples). Não usa Full nem Faturador por enquanto.
- Todas no endereço do galpão (endereço ainda **a informar**).
- Cores das contas (tokens do shell): `--acc-1` … `--acc-4`.
- Use **CNPJs fictícios com dígito verificador válido**, marcados como exemplo.

O ERP é o **dono da integração com o ML** (1 aplicação ML por titular). Ele guarda os tokens de cada conta, renova o refresh token (uso único, válido 6 meses; access token de 6 h) com trava por conta, recebe as notificações (webhook com resposta em até 500 ms, fila, `missed_feeds` de 2 dias) e alimenta o agente de IA de atendimento. **Só o admin de cada conta ML pode autorizar.** Não existe sandbox.

## Escopo funcional (todas as versões precisam cobrir; a forma muda)
1. **Empresas**
   - Listar, incluir, editar e **inativar** (nunca excluir; inativar exige confirmação e explica o efeito: contas pausadas, histórico preservado).
   - Campos: razão social, nome fantasia, CNPJ (máscara + validação de dígito verificador + não duplicado), inscrição estadual (ou "isento"), regime (MEI / Simples Nacional / Lucro Presumido / Lucro Real), UF, endereço (CEP com máscara; opção **"usar o endereço do galpão"**), e-mail fiscal, telefone, cor de identificação, papel (**principal** / secundária, só uma principal; trocar a principal pede confirmação e explica que as despesas comuns passam a ir para a nova principal).
   - Seção **Fiscal (Fase 2)**: certificado A1, série e numeração de NF-e, mostrada **desabilitada** com o rótulo "Fase 2: emissor próprio".
   - Monitor do **limite de faturamento**: MEI R$ 81 mil/ano, Simples R$ 4,8 milhões/ano. Barra de progresso com faturamento acumulado no ano (exemplo) e alerta ao passar de 80%.
2. **Contas de marketplace** (por empresa)
   - Canal Mercado Livre ativo. Shopee, Amazon, Magalu, TikTok, B2B e site próprio aparecem como "em breve".
   - **Conectar conta ML** simulando o OAuth: modal explicando que o admin da conta precisa entrar → estado "aguardando autorização" com spinner → sucesso (nickname, user_id, tags detectadas). Incluir também um caminho de **erro** ("quem entrou não é o administrador da conta" = `invalid_operator_user_id`).
   - **Reconectar** (Conta 3) e **Desconectar** (com confirmação: os anúncios deixam de sincronizar, nada é apagado).
   - Detalhes da conexão: nickname, user_id, empresa dona, data da autorização, validade do refresh token (dias restantes), última renovação do access token, próxima renovação, **tags da conta** (`user_product_seller`, `warehouse_management`) e o efeito de cada uma ("estoque publicado via seller_warehouse"), reputação (termômetro), MercadoLíder.
   - **Saúde da integração** por conta: notificações recebidas nas últimas 24 h por tópico (orders_v2, items, questions, messages, post_purchase, shipments, invoices…), tempo médio de resposta do webhook (< 500 ms), fila pendente, falhas, eventos recuperados via missed_feeds, última reconciliação. Botão **"Reconciliar agora"** (simula processamento com toast).
   - **Depósitos ligados à conta**: o Galpão é compartilhado (com saldo por empresa dona); o "Full da conta X" é criado automaticamente quando a conta usa Full (a Conta 4 não usa).
   - **Configuração por conta**: estoque de segurança padrão dos anúncios, tipo de anúncio padrão (Clássico/Premium), se a conta usa Full / coleta / Flex (Flex "em aberto").
3. **Métricas e indicadores** no topo: empresas ativas, contas conectadas × com problema, notificações processadas em 24 h, pedidos dos últimos 30 dias por conta, faturamento do ano × limite do regime.
4. **Histórico de alterações** (auditoria): quem mudou o quê e quando (ex.: "Boris trocou a conta principal", "Token da Conta 3 expirou").
5. **Alertas do shell** coerentes: o alerta "Conta 3 desconectada" leva a esta tela; ao reconectar com sucesso, o alerta some, o contador do sino diminui e o selo do topo "1 conta para reconectar" vira "Integrações ok".
6. **RBAC**: só o Dono acessa (os outros perfis já veem a tela "sem acesso" do shell; manter).

## Comportamento obrigatório ("como em produção", sem backend)
- Estado em memória com dados de exemplo (as 4 empresas/contas acima + números plausíveis). Tudo que é criado ou editado aparece imediatamente na lista, nas métricas, no seletor de empresa do topo e no histórico.
- Modais e drawers acessíveis: foco preso, ESC fecha, retorno do foco, `aria-*`.
- Toast em toda ação ("Empresa incluída", "Conta 3 reconectada", "Alterações salvas"…).
- Validação inline com mensagens claras (CNPJ inválido, CNPJ já cadastrado, campo obrigatório).
- Estados: vazio (ex.: empresa sem conta conectada), carregando (skeleton/spinner ~1 s nas simulações), erro (caminho de falha no OAuth).
- Busca/filtro na lista (por nome, CNPJ, regime, status).
- Responsivo 360 / 390 / 768 / 1366; tabelas largas com rolagem própria ou cards no celular; nada de rolagem horizontal da página.
- Formatação pt-BR: `toLocaleString('pt-BR')`, BRL, datas dd/mm/aaaa, CNPJ 00.000.000/0000-00.

## Regras técnicas de cada arquivo
- **Copie o `pinese-erp.html` inteiro** e implemente o módulo no lugar do placeholder da rota `#empresas`. Não mude os tokens nem os componentes existentes; acrescente classes novas com prefixo `m2-`.
- O arquivo deve **abrir já logado como Dono, na rota `#empresas`** (pule o login no carregamento, mas mantenha o código dele e o "Sair").
- Atualize o rodapé do menu para "Protótipo · módulo 2 · vNN".
- Um único arquivo HTML, sem `<!doctype>/<html>/<head>/<body>` (o publicador envolve), começando por `<title>Empresas e contas vNN</title>`.
- Sem `alert/confirm/prompt`, `window.print`, downloads, iframes, `mailto:`. `localStorage` sempre em try/catch. Sem bibliotecas externas (se for indispensável, só cdnjs com versão fixa).
- Antes de terminar, **valide a sintaxe** extraindo o `<script>` principal para um arquivo temporário no scratchpad e rodando `node --check`. Corrija até passar. Não abra navegador nem rode testes visuais repetidos.
- Português do Brasil com acentuação correta em toda a interface.

## As 10 versões (cada uma com um conceito de UX realmente diferente)
| v | Conceito |
|---|---|
| v01 | **ERP clássico denso**: tabela de empresas com contas aninhadas (linhas expansíveis) + drawer lateral de edição com abas. Prioriza densidade em 1366×768. |
| v02 | **Cards por empresa**: grid de cartões grandes (identidade, regime, conta ML com status, limite de faturamento) + inclusão por modal em etapas (wizard de 3 passos). |
| v03 | **Master–detail**: lista de empresas à esquerda e painel de detalhe com abas (Dados · Conta ML · Saúde · Depósitos · Histórico) à direita; no celular vira navegação em duas telas. |
| v04 | **Onboarding guiado**: foco em "colocar as 4 contas no ar". Checklist de prontidão por empresa (dados ✔, conta conectada ✔, tags lidas ✔, depósitos ✔, limite configurado) + stepper para incluir nova empresa e conectar. |
| v05 | **Monitor de integração primeiro**: página estilo "status page" (uptime, fila, latência do webhook, tokens com contagem regressiva, timeline de eventos por tópico); o cadastro de empresas fica numa aba secundária. |
| v06 | **Árvore da estrutura**: Grupo → Empresas → Contas → Depósitos (Galpão compartilhado, Full por conta) desenhados como organograma/árvore interativa; clicar num nó abre o painel de edição. |
| v07 | **Quadro por estado**: colunas Conectada · Atenção · Desconectada · Inativa (estilo kanban); arrastar não é necessário, mas as ações mudam o cartão de coluna com animação. |
| v08 | **Página de configurações**: uma página longa por empresa com seções verticais e navegação lateral de âncoras, edição inline com salvamento automático e indicador "salvo há X s". |
| v09 | **Visão fiscal e de limites**: tabela comparativa por empresa (regime, faturamento do ano, % do limite, projeção até dezembro, data estimada de estouro) com destaque para MEI; a gestão das contas ML aparece como coluna/ação. |
| v10 | **Centro de comando compacto**: tiles por conta com métricas ao vivo (pedidos hoje, reputação, contagem regressiva do token, fila), ações rápidas por atalho de teclado, paleta de comandos própria do módulo. |

Saída: `C:\Users\tocub\pinese\erp\prototipo\m02-empresas\vNN.html`. Ao terminar, responda **só** com: arquivos gerados, 3–5 linhas por versão com os destaques, e qualquer limitação.
