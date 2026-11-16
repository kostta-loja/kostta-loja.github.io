# Kostta Loja

Aplicativo web (PWA) de operação de loja do Mercado Kostta, com dois módulos sobre a mesma Grade do ERP:

- **Recebe** — conferência de recebimento: lê o XML da NF-e, cruza com a Grade e com o pedido, e resume tudo em quatro blocos (Faturamento, Cadastro, Margem, Conferidos).
- **Pedidos** — compra para reposição: importa o pedido de uma conversa, PDF ou foto, associa cada item à Grade, compara o custo pedido com o do ERP e gera o espelho do pedido.
- **Modo Blecaute** — venda sem energia: lê o código de barras do produto, monta a lista com quantidades, fecha vendas e gera o relatório do período.

- **Repositório:** https://github.com/kostta-loja/kostta-loja.github.io
- **Aplicativo:** https://kostta-loja.github.io/
- **Versão:** 2026.11.14 (mostrada em Ajustes) · arquivo único `index.html`, sem servidor, dados no aparelho e compartilhados pela nuvem

## 1. Fluxo de uso

1. **Grade** (computador): aba *Grade* > Carregar Grade (relatório do ERP). No celular hospedado, o mesmo botão aparece na aba *Grade*.
2. **Chave** (opcional): *Tirar foto da chave* / *Foto da galeria*. A leitura é automática e tenta, nesta ordem, código de barras, QR Code e os 44 números impressos (OCR, baixa uma biblioteca na primeira vez). Também aceita colar os 44 dígitos. A chave é validada e copiada.
3. **XML**: *Abrir Meu DANFE* (manual), baixar o XML, depois *Buscar XML* (dispositivo ou Drive).
4. **Conferir**: a tela abre com o resumo da nota e quatro blocos. Cada um abre com um toque:
   - **Faturamento** — nota × pedido: custo acima do negociado, quantidade diferente, item não entregue, item fora do pedido, item repetido no pedido. Sem pedido vinculado, o bloco pergunta de qual pedido é a nota.
   - **Cadastro** — itens sem código, EAN novo, unidades por embalagem e perfil de quantidade do fornecedor.
   - **Margem** — preço fora da faixa e variação de custo contra o ERP, item a item, com *Preço ajustado*.
   - **Conferidos** — o que já está liberado.
   No rodapé: marcar como recebido, exportar CSV, enviar para a nuvem, compartilhar lista, outra nota e limpar.

## 2. Regras de negócio

**Cruzamento com a Grade**
- EAN da unidade (`cEANTrib`), depois `cEAN`. EAN-14 (DUN-14) gera o EAN da unidade pelo dígito verificador.
- Sem correspondência: busca manual na Grade por código ou nome, com associação salva por EAN.

**Custo unitário** = (`vProd` + ICMS-ST + IPI) ÷ quantidade tributável. *Somar ICMS-ST e IPI* é parâmetro.
- Unidade de embalagem (Fd, Cx, Display etc.): divide pelas unidades por embalagem, na ordem: valor salvo (manual) > fator da descrição ("x 30", "C/12", "12x500g") > sugestão pelo custo do ERP (exige confirmação) > informar manualmente.
- Unidade em KG: custo por kg × peso por unidade da descrição ("170g"). Item vendido por kg (Grade com "KG", ou peso = caixa) fica em custo por kg. Peso editável (`56`, `56g`, `0,056kg`; `0` = por kg).
- Perfil por fornecedor (CNPJ): Automático, Embalagem ou Unidade. Valores aprendidos são salvos por fornecedor + EAN.

**Margem e preço**
- Lucro sobre a venda = 1 − custo ÷ preço (igual ao "Lucro sobre a venda" do ERP). Markup atual = preço ÷ custo − 1.
- Faixa esperada padrão: **37,5% a 40,5%** (equivale a markup de 60% e margem líquida de 30% a 33% com 7,5 pontos de deduções).
- Preço sugerido = custo × (1 + markup), com arredondamento opcional (final ,90).

**Status do item**

| Status | Condição | Ação |
|---|---|---|
| Cadastrar Item | Não está na Grade | Cadastrar no ERP |
| Informar embalagem | Unidade de embalagem sem fator | Informar unidades por embalagem |
| Ajustar | Lucro abaixo da faixa | Ajustar o preço para o sugerido |
| Atenção | Lucro acima da faixa | Verificar o preço |
| Liberado | Dentro da faixa | Liberar para venda |

**Variação de custo vs ERP:** custo da nota ÷ custo com encargos da Grade. Acima de 1% (parâmetro): subiu ou caiu. Acima de 40%: aviso para conferir unidades. Item com marca "OF" na Grade: aviso de oferta.

**Chave da nota:** confere a chave informada com o XML carregado e avisa se a nota já foi carregada antes (neste aparelho).

## 3. Telas

| Aba | Conteúdo |
|---|---|
| Vendas | Modo Blecaute (ver seção 4) |
| Pedidos | Compra para reposição (ver seção 5) |
| Conferir | Painel de recebimento (Pedidos / Notas fiscais) e, com a nota aberta, a tela de conferência em quatro blocos. O painel fica oculto enquanto a nota está aberta; volta pelo botão *Painel* |
| Grade | Produtos, data, origem, idade (alerta após 7 dias), margem da Grade (% dentro/abaixo/acima) com histórico de cargas; carregar Grade e salvar página com a Grade (computador) |
| Ajustes | Dois cartões: **Nuvem** (login, sincronização, versão e atualização) e **Configurações**, com Lojas, Prazos de pagamento, Parâmetros de margem, Embalagens salvas, Limpar dados de teste e exemplo fictício — cada um abre quando clicado |

A lista resumida mostra código interno, descrição e status. O texto compartilhado é agrupado por status e inclui a seção de custo alterado.

## 4. Modo Blecaute (venda sem energia)

Objetivo: registrar vendas no celular quando o caixa está fora do ar, usando a mesma Grade.

1. **Ler código do produto** (câmera ou galeria): EAN-13, EAN-8, UPC, Code 128, Code 39 e ITF. Ou digitar o código de barras ou o código interno; o campo aceita apenas números e abre o teclado numérico.
2. Cada item entra com **quantidade 1**, editável (aceita fração, ex.: `2,5` para granel). Ler o mesmo produto de novo soma a quantidade e o traz para o topo.
3. **Caixa de seleção** por item: só os marcados entram no total. O **✕** remove o item.
4. **Totalizador**: produtos, itens e total em destaque.
5. **Fechar venda**: grava a venda no período e limpa os itens marcados (os desmarcados ficam na lista). **Compartilhar** e **Exportar CSV** valem para a lista atual; **Zerar lista** limpa tudo.

**Fechamento do período** (bloco abaixo da lista):
- Cartão com vendas, itens, **total vendido**, produtos distintos, ticket médio e período.
- Detalhes: vendas uma a uma (hora, produtos, valor, com ✕ para excluir) e produtos vendidos consolidados.
- **Resumo do período**: texto curto para WhatsApp com vendas, itens, produtos distintos, ticket médio, maior venda, **total vendido** e os 5 mais vendidos — é o que se usa para confrontar com o caixa.
- **Relatório detalhado**: as vendas uma a uma e todos os produtos consolidados.
- **Exportar CSV**: uma linha por item, com o número da venda e a linha de total (`Kostta_blecaute_AAAAMMDD_HHMM.csv`).
- **Encerrar período**: apaga as vendas fechadas, com confirmação. Compartilhe ou exporte antes.

Preço usado: `P.VENDA` da Grade. A lista e as vendas fechadas ficam guardadas no aparelho e sobrevivem a fechar o app ou ficar sem bateria.

## 5. Pedidos (compra para reposição)

Feito no celular, no chão de loja. O **fornecedor** é sempre digitado. Três modos, nesta ordem de preferência:

1. **Manual (preferido):** ler o código de barras pela câmera ou buscar na Grade por nome ou código (setas ↑ ↓ e Enter no computador). O item entra com quantidade 1; informe quantidade e custo negociado.
2. **Planilha (CSV):** peça o pedido em Excel/CSV ao vendedor. O app reconhece as colunas pelo cabeçalho (descrição, quantidade, preço, código de barras, código) e também lê sem cabeçalho, pela ordem das colunas.
3. **PDF/texto (último caso):** cola de conversa, leitura de PDF ou foto (OCR). Leitura por heurística: identifica a linha pela conta quantidade × valor unitário = total, pega o EAN quando existe e descarta cabeçalhos e linhas de coluna repetida. **O app não deixa salvar sem marcar "Conferi todos os itens".**

Em qualquer modo: cada item é associado à Grade (*confirmado* pelo EAN ou pela sua escolha, *sugerido* pelo nome), e as associações ficam guardadas por descrição. Por item: **un. por embalagem** (detectada por "C/N" ou pela unidade), **custo unitário**, **custo do ERP**, **variação**, **preço de venda atual** e **lucro se comprar**.

**Fluxo de digitação:** ao escolher o produto, ele abre num cartão de edição (un. por embalagem, quantidade e custo, nessa ordem; o custo usa máscara de dinheiro, só números). Ao concluir (tecla OK ou *Concluir item*), o item vira uma linha compacta — código, descrição, quantidade × custo, valor unitário e subtotal, com ✕ — a lista fica em ordem alfabética e o cursor volta para a busca. Tocar na linha reabre o item.

**Salvar pedido** guarda no aparelho, envia para a nuvem e **limpa a tela** (itens, fornecedor e busca) para o próximo pedido; **Espelho** gera o texto para o WhatsApp; **Exportar CSV** inclui custo da embalagem, unidades e custo unitário. A lista **Pedidos realizados** mostra data, fornecedor, itens, total e status (**Aberto**, **Em conferência**, **Recebido**), com o botão *enviar para conferência*. Trocar de modo limpa os itens em edição.

**Loja e prazo:** todo pedido começa pela **Loja** (lista editável em Ajustes) e pelo **prazo de pagamento** (lista editável, ou digitado). Ambos vão para o espelho, para o CSV e para a nuvem, e aparecem na conferência.

**Sem duplicar:** ao buscar um produto que já está no pedido, o app traz aquela linha para edição em vez de criar outra.

**Histórico na hora da negociação:** na lista de busca e no cartão do item aparece *última compra R$ X · há N dias · variação*, e no cartão também o volume médio de reposição e o intervalo entre compras.

Tabela na nuvem: `pedidos` (loja, fornecedor, prazo, nf, total, status, itens). SQL em `supabase-pedidos.sql`. Os pedidos salvos na nuvem são baixados pelos outros aparelhos ao entrar e em *Sincronizar agora*, com a regra de que o status mais avançado vence (aberto → aguardando NF → NF vinculada → recebido).

## 5.1 Histórico de compras (base de negociação)

Cada item recebido vira uma linha na tabela `entregas` da nuvem: loja, fornecedor, CNPJ, NF, data de emissão, código interno, EAN, descrição, unidades, custo unitário, total e pedido de origem. A chave `nf|cnpj|código|loja` impede duplicar.

**Quando grava:** ao marcar o pedido como *recebido* com a nota vinculada, e no botão *Enviar para a nuvem* da conferência.

**Onde consultar:** aba *Pedidos* > **Histórico de compras**. Filtros por loja, fornecedor e produto. Mostra, por produto:
- **dias desde a última entrega** e a data;
- **variação de preço** contra a entrega anterior, com o valor das duas;
- **volume de reposição** (média de unidades por entrega) e o intervalo médio entre compras;
- por fornecedor: última entrega, número de entregas, produtos distintos e total comprado.

O histórico é baixado ao entrar na nuvem e em *Sincronizar agora*, e fica no aparelho para consulta sem internet (últimas 1.200 linhas). SQL em `supabase-entregas.sql`.

## 6. Dados

**Grade (entrada):** relatório do ERP em texto com tabulação (`grade_DDMM.xls`) ou CSV. Colunas usadas: `CODIGO`, `DESCRICAO DO PRODUTO`, `P.VENDA`, `*C/ENC.` (ou `C.COMP`), `COD. BARRA` (8+ dígitos) e a coluna sem nome com "OF" (oferta). O cabeçalho é localizado sozinho.

**Guardado no aparelho** (navegador): Grade, parâmetros, lojas, prazos, embalagens e pesos por fornecedor, associações EAN > código, perfis de fornecedor, histórico de cargas, chaves já carregadas, pedidos, notas guardadas (resumo e XML das 12 últimas, para reabrir a conferência) e o histórico de compras baixado.

**Compartilhado na nuvem (Supabase):** Grade publicada (`grade.json`), vendas do Modo Blecaute (`vendas`), conferências de recebimento (`conferencias`), embalagens e pesos (`embalagens`), associações EAN > código (`associacoes`), pedidos (`pedidos`) e histórico de compras (`entregas`). A sincronização é automática ao entrar e ao abrir o app; em Ajustes há os botões *Sincronizar agora* e *Embalagens*. Regra de conflito: um valor definido à mão vence o detectado automaticamente; entre dois iguais, vence o mais recente.

**Exportação (CSV, 29 colunas, `;`, UTF-8):** data, NF, fornecedor, código interno, EANs, descrição, custos, variação, preço, markup, lucro, faixa, preço sugerido, status, preço ajustado, oferta, `chave_linha`, `chave_nfe`. Nome: `Kostta_conferencia_AAAAMMDD_HHMM.csv`.

## 7. Histórico e indicadores (Excel)

- `Kostta_Historico.xlsx`: painel com margem da Grade por data, preços e custos que subiram/caíram entre Grades, conferências por status e mês, ajustes confirmados (preço mudou na Grade seguinte) e itens recorrentes.
- `consultas_power_query.txt`: 6 consultas (Conferencias, GradeHistorico, GradeResumo, PrecoMudancas, AjustesConfirmados, Recorrencia). Leem a pasta de CSVs exportados e a pasta de Grades (`grade_DDMM.xls`).
- Parâmetros da planilha: pastas, faixa de lucro, tolerância de custo (aba *Parametros*, tabela `tblParam`).

## 8. Implantação e atualização

- **Arquivos** (mesma pasta): `index.html`, `manifest.webmanifest`, `sw.js`, `icon-192.png`, `icon-512.png`, `icon-512-maskable.png`, `apple-touch-icon.png`.
- **Hospedagem:** GitHub Pages (repositório público, branch `main`, raiz). O repositório não contém Grade, custos nem preços.
- **Atualizar:** substituir `index.html` (e `sw.js`, se mudar) no repositório. O app pega a versão nova ao abrir com internet; se necessário, abrir duas vezes. Se continuar antigo, usar **Ajustes > Buscar atualização** (limpa o cache e o service worker, sem apagar Grade, embalagens ou vendas). A versão instalada aparece em Ajustes.
- **Instalar (Android/Chrome):** abrir o endereço github.io, menu ⋮ > Adicionar à tela inicial > **Instalar** (a opção "Criar atalho" não instala o app). iPhone (Safari): Compartilhar > Adicionar à Tela de Início.
- **Alternativa sem hospedagem:** *Salvar página com a Grade* gera um HTML com a Grade e as configurações embutidas, para abrir no Chrome do celular (não pela visualização do Drive). **Contém custos e preços: manter privado.**

## 9. Etapas do projeto

| # | Etapa | Situação |
|---|---|---|
| 1 | Leitura do XML da NF-e e cruzamento com a Grade do ERP | Concluída |
| 2 | Custo unitário por embalagem, peso e perfil de fornecedor | Concluída |
| 3 | Margem, preço sugerido e status por item | Concluída |
| 4 | Variação de custo entre a nota e o ERP | Concluída |
| 5 | Aplicativo instalável (PWA) hospedado no GitHub Pages | Concluída |
| 6 | Modo Blecaute: venda, fechamento e relatórios | Concluída |
| 7 | Nuvem (Supabase): Grade publicada, vendas e conferências compartilhadas | Concluída |
| 8 | Histórico e indicadores no Excel/Power Query | Entregue, em uso inicial |
| 9 | Painel gerencial no Power BI sobre a base da nuvem | A fazer |
| 10 | Módulo Pedidos: manual, planilha, PDF/texto, associação à Grade, espelho e status | Concluída |
| 11 | Confronto automático do pedido com a nota no módulo Recebe | Concluída |
| 13 | Histórico de compras por loja, fornecedor e produto, com painel de consulta | Concluída |
| 14 | Transferências entre lojas (Separado > Coletado > Recebido) | A fazer |
| 12 | Conferência fiscal (NCM, CEST, CST) | A fazer: depende de exportação do ERP com esses campos |

## 10. Limitações conhecidas

- Cada aparelho guarda uma cópia local de tudo para funcionar sem internet; Grade, vendas, conferências, embalagens e associações são compartilhadas pela nuvem.
- O download do XML no Meu DANFE é manual (o site usa verificação anti-robô). O destino do download é configuração do navegador.
- Leitura por foto: exige imagem nítida e pouco inclinada; na primeira vez baixa uma biblioteca (internet).
- Não há conferência de NCM, CEST ou CST (a Grade exportada não traz esses campos).
- Variação de custo compara com o custo com encargos do ERP, que pode incluir frete ou outras despesas.
- Em Pedidos, a leitura de PDF/texto é heurística: alguns layouts saem por colunas e podem perder ou trocar itens. Use Manual ou Planilha; o botão *Copiar texto para suporte* envia o texto extraído para análise.
- Preços escritos à mão no PDF não são lidos: ajuste o custo na linha do item.
- Modo Blecaute não emite documento fiscal nem baixa estoque: é um registro para conferir o caixa e lançar no ERP depois.
- As vendas ficam no aparelho em que foram feitas; com vários celulares, cada um gera o seu relatório.
- O XML das notas fica guardado só no aparelho que o carregou (12 últimas). Em outro aparelho, a nota aparece na lista mas precisa do XML para o confronto item a item.
- O histórico de compras começa a valer a partir da primeira nota enviada para a nuvem; entregas anteriores ao app não entram.
- A variação de preço do histórico compara o custo unitário de venda (já convertido por embalagem ou peso) entre as duas últimas entregas do mesmo produto.
