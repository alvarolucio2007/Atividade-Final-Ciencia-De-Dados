
# Uso de Inteligência Artificial — Frente 1

**Projeto:** Ciência de Dados
**Integrantes:** Álvaro Lúcio
**Última atualização:** 24/11/2026

---

## Tabela de registros

| Registro | Data | Integrante | Ferramenta | Etapa e o que foi pedido | O que foi aproveitado | O que a equipe alterou ou verificou | Link da conversa |
| ---------- | ------ | ------------ | ------------ | -------------------------- | ----------------------- | ------------------------------------- | ------------------ |
| 1 | 23/11/2026 | Álvaro Lúcio | DeepSeek | Definição de regra de outliers em `price`/`freight_value` (`order_items`) | Sugestão inicial de remoção via `Q3 + 3·IQR` | **Rejeitada.** Outlier ≠ erro. Regra não distingue valor extremo legítimo (item de alto valor, promoção) de valor inválido. Nenhuma linha removida | [https://chat.deepseek.com/share/37svfdmrhp9niu97ll](https://chat.deepseek.com/share/37svfdmrhp9niu97ll) |
| 2 | 23/11/2026 | Álvaro Lúcio | DeepSeek | Pipelines de auditoria e limpeza para `order_items` | Estrutura de `auditar_items` e `limpar_items` (checks de nulos, IDs, PK composta, `order_item_id`, datas, `price`/`freight`) | Revisado; nenhuma remoção por invalidez — base aderente. Apenas cast de tipos + `strip` | [https://chat.deepseek.com/share/37svfdmrhp9niu97ll](https://chat.deepseek.com/share/37svfdmrhp9niu97ll) |
| 3 | 23/11/2026 | Álvaro Lúcio | DeepSeek | Pipelines de auditoria e limpeza para `orders`; inclusão de `DATA_MIN=2000`; verificação condicional de `approved_at` por status | Estrutura de `auditar_orders` e `limpar_orders`; colunas `flag_*`; bloco de `decompor_remocoes` | Revisado; ajustado `DATA_MIN` de 2010 para 2000; fix do `ValueError` (ver Registro 6); removidas 14 linhas por `approved_at` nulo com status inválido | [https://chat.deepseek.com/share/37svfdmrhp9niu97ll](https://chat.deepseek.com/share/37svfdmrhp9niu97ll) |
| 4 | 23/11/2026 | Álvaro Lúcio | DeepSeek | Pipelines de auditoria e limpeza para `order_reviews` | Estrutura de `auditar_reviews` e `limpar_reviews`; normalização de strings vazias em `title`/`message`; `flag_order_multi_review` | Revisado; mantidas as 1.098 linhas com `order_id` multi-review (decisão de modelagem delegada à Frente 2) | [https://chat.deepseek.com/share/37svfdmrhp9niu97ll](https://chat.deepseek.com/share/37svfdmrhp9niu97ll) |
| 5 | 23/11/2026 | Álvaro Lúcio | DeepSeek | Pipelines de auditoria e limpeza para `products` | Estrutura de `auditar_products` e `limpar_products`; `drop_duplicates` por `product_id` | Revisado; detecção e correção de import duplicado (65.902 → 32.951 linhas) | [https://chat.deepseek.com/share/37svfdmrhp9niu97ll](https://chat.deepseek.com/share/37svfdmrhp9niu97ll) |
| 6 | 23/11/2026 | Álvaro Lúcio | DeepSeek | Diagnóstico e correção de `ValueError: Can only compare identically-labeled Series objects` em `limpar_orders` | Diagnóstico da causa (flags calculadas após `reset_index`, reusando Series do `df` original) e reordenação do pipeline (flags → filtro → dedup) | Revisado; correção aplicada no código; risco identificado: em versões antigas do pandas, comparação por label poderia ter atribuído flags ao pedido errado silenciosamente | [https://chat.deepseek.com/share/37svfdmrhp9niu97ll](https://chat.deepseek.com/share/37svfdmrhp9niu97ll)|
| 7 | 23/11/2026 | Álvaro Lúcio | DeepSeek | Revisão de português e formatação de READMEs do projeto | Correções gramaticais e ajuste de clareza em trechos escritos pela equipe | Revisado; nenhum conteúdo novo (conclusões, justificativas, sumário) foi gerado por IA nos READMEs | [https://chat.deepseek.com/share/vb4uz2d7pvx884vxl9](https://chat.deepseek.com/share/vb4uz2d7pvx884vxl9) |


---

## Caso de IA inadequada — Registro 1

**Contexto:** IA sugeriu remover outliers de `price`/`freight_value` via `Q3 + 3·IQR` no pipeline de limpeza de `order_items`.

**Por que foi rejeitado:**

- Outliers nesse caso não são necessariamente dados errados, podendo ser pedidos reais, promoções, e afins.

**Como percebemos:**

- Checagem visual da função `auditar_items`: a regra não distingue valor extremo legítimo de valor inválido.

**O que fizemos:**

- Remoção da etapa de retirada de outliers do pipeline.

---

## Caso de IA inadequada — Registro 6

**Contexto:** Ao executar `limpar_orders`, a IA gerou código que calculava as `flag_*` após o filtro `~mask_ruim` + `reset_index(drop=True)`, reusando Series do `df` original (pré-filtro).

**Por que é inadequado:**

- `reset_index` troca os rótulos das linhas. A comparação entre a Series antiga (índice original) e a nova coluna do `df` filtrado (índice reindexado) levanta `ValueError: Can only compare identically-labeled Series objects`.
- Em versões antigas do pandas que alinhavam por label em vez de recusar a operação, o resultado seria silenciosamente errado: flags alinhadas por rótulo, podendo atribuir `flag_atraso` ao pedido errado.

**Como percebemos:**

- Traceback apontou `_cmp_method` em `Series`; leitura do código revelou que a Series `d` era capturada antes do `reset_index`.

**O que fizemos:**

- Reordenamos o pipeline: `flag_*` calculadas **antes** do filtro; `reset_index(drop=True)` movido para o **início** da função. Comentário `# [IA] registro 6` adicionado no trecho corrigido.

---

## Registros de pipelines (referência cruzada com o diário)

| Registro | Dataset | Decisões do diário cobertas |
| ---------- | --------- | ----------------------------- |
| 2 | `order_items` | Decisão 4 |
| 3 | `orders` | Decisões 2, 2-b |
| 4 | `order_reviews` | Decisões 3, 3-b, 6 |
| 5 | `products` | Decisão 8 |
| 6 | `orders` (fix) | Decisão 7 |
| 7 | (READMEs) | — (não é decisão de dados; registro de transparência) |

---

## Declaração

Declaro que os registros acima contemplam todos os usos de IA relevantes realizados pela equipe na Frente 1, conforme item 7 do edital.

Álvaro Lúcio — 23/11/2026
