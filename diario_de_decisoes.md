# Diário de Decisões — Frente 1: Limpeza e Tratamento de Dados

**Projeto:** Ciência de Dados

**Integrantes da Frente 1:** Álvaro Lúcio

**Data de início:** 23/11/2026

**Última atualização:** 24/11/2026

**Versão:** v3

---

## 1. Escopo da Frente 1

**O que esta frente faz:**

- Remove linhas objetivamente inválidas (violação de regra dura de domínio).
- Normaliza tipos (`object → datetime`, cast de numéricos, `strip` de IDs).
- Sinaliza ambiguidades via colunas `flag_*` para as frentes seguintes.

**O que esta frente NÃO faz (e delega):**

- Decisões de modelagem (agregação, granularidade analítica) → Frente de Análise.
- Respostas às perguntas de negócio → Frente de Análise.
- Escolha de visualizações → Frente de Visualização.

**Datasets tratados nesta frente:**

| Dataset | Linhas recebidas | Linhas entregues | Removidas (%) |
| --------- | ------------------ | ------------------ | --------------- |
| `olist_orders` | 99.441 | 98.036 | 1,413% |
| `olist_order_reviews` | 99.224 | 98.410 | 0,820% |
| `olist_order_items` | 112.650 | 112.650 | 0% |
| `olist_products` | 32.951 | 32.951 | 0%* |

\* Recebidas 65.902 em memória por import duplicado (ver Decisão 8); removidas 32.951 duplicatas antes da entrega.

---

## 2. Visão geral das decisões

| Nº | Etapa | Problema encontrado | Decisão tomada | Justificativa (resumo) | Impacto nos dados | Registro IA |
| ---- | ------- | --------------------- | ---------------- | --------------------------- | ------------------- | ------------- |
| 1 | Piso temporal | Datas absurdas contaminariam séries | `DATA_MIN = 2000-01-01` em todas as colunas temporais | O ano de 2010 é muito recente para lojas, 2000 é um período maior e mais realista. | 0 linhas removidas | — |
| 2 | Nulos em `orders.order_approved_at` | 14 pedidos com status que exige aprovação, sem `approved_at` | Remover os pedidos | Poucos pedidos, e possuem estados inválidos | 14 linhas (0,014%) | 3 |
| 2-b | Outras invalidezes em `orders` | Inconsistências temporais e de status não cobertas pela Decisão 2 | Remoção via `mask_ruim` (regras detalhadas na seção 3) | Remoção de dados que não valeriam a pena recuperar.| 1.391 linhas (1,399%) | 3 |
| 3 | `order_id` com múltiplos reviews | 1.098 `order_id` (~1,11%) em >1 review | Mantidas todas; `flag_order_multi_review`; agregação delegada à Frente 2 | Pode ser regra de negócio, como edições | 0 removidas; 1.098 sinalizadas | 4 |
| 3-b | Outras invalidezes em `reviews` | Score fora do domínio, resposta antes da criação, PK duplicada, datas inválidas | Remoção via `mask_ruim` (regras detalhadas na seção 3) | Remoção de dados que não valeriam a pena recuperar. | 814 linhas (0,820%) | 4 |
| 4 | `order_items` aderente | Nenhuma invalidez dura detectada | Apenas cast de tipos + strip | Não foi necessário retirar linhas — base já sem valores impossíveis; só normalização | 0 removidas (0,000%) | 2 |
| 5 | Outliers em `price` / `freight_value` | IA sugeriu remoção via `Q3+3IQR` | **Rejeitada.** Nenhuma remoção | Outliers não são necessariamente erros — podem ser pedidos reais, promoções, itens de alto valor (ver Decisão 5) | 0 removidas | **1** |
| 6 | Nulos em `reviews.review_comment_title` / `_message` | 88% / 59% de nulos respectivos | Mantidos como `pd.NA`; strings vazias → `NA` | Apenas ausência de comentários, não torna os dados inválidos | 0 removidas | 4 |
| 7 | Erro `ValueError` em `limpar_orders` | `Can only compare identically-labeled Series` ao calcular flags após filtro | Reordenado: flags antes do filtro + `reset_index` inicial | Erro de sintaxe no Pandas após geração de código com IA | Correção de código | **6** |

---

## 3. Decisões detalhadas

### Decisão 1 — Piso temporal (`DATA_MIN = 2000-01-01`)

| Campo | Valor |
| ------- | ------- |
| Dataset(s) | `orders`, `order_items` (todas as colunas de data) |
| Data da decisão | 23/11/2026 |
| Responsável | Álvaro Lúcio |
| Registro de IA | — (decisão de negócio, não de código) |

**Problema encontrado:** Sem piso mínimo, datas absurdas (default de sistema legado) contaminariam séries temporais.

**Como foi detectado:** Auditoria automática: contagem de linhas com data anterior a referência mínima.

**Decisão:** `DATA_MIN = pd.Timestamp("2000-01-01")` em todas as validações de data.

**Justificativa:** O ano de 2010 é muito recente para diversas empresas, tendo risco de invalidar os dados. O ano 2000 é um ano de começo de coleta de dados muito mais realista.

**Impacto:** 0 linhas removidas por essa regra (validação preventiva).

**Alternativas descartadas:**

- `DATA_MIN = 2010` — Intervalo muito curto de tempo
- `DATA_MIN = min(data)` do próprio dataset — Se houvesse, no futuro, tempos menores que o mínimo atual, poderia quebrar o pipeline, e deixar um tempo estático ajuda na visualização.

---

### Decisão 2 — Nulos em `orders.order_approved_at` com status que exige aprovação

| Campo | Valor |
| ------- | ------- |
| Dataset | `olist_orders` |
| Data da decisão | 23/11/2026 |
| Responsável | Álvaro Lúcio |
| Registro de IA | 3 (`uso_de_ia.md`) |

**Problema encontrado:** 14 pedidos (0,014%) têm `order_status ∈ {approved, invoiced, processing, shipped, delivered}` mas `order_approved_at` nulo.

**Como foi detectado:** Auditoria condicional (status × presença de data).

**Decisão:** Remoção.

**Justificativa:** Sendo apenas 0,014% dos dados, não seria relevante tentar tratar tais dados.

**Impacto:** 14 linhas (0,014%) removidas.

---

### Decisão 2-b — Demais invalidezes em `orders` (decomposição da `mask_ruim`)

| Campo | Valor |
| ------- | ------- |
| Dataset | `olist_orders` |
| Data da decisão | 23/11/2026 |
| Responsável | Álvaro Lúcio |
| Registro de IA | 3 (`uso_de_ia.md`) |

**Problema encontrado:** Além dos 14 casos da Decisão 2, a `mask_ruim` remove linhas por outras 8 regras de invalidez dura (timestamps invertidos, `delivered` sem data, status fora do domínio, etc.).

**Como foi detectado:** Decomposição individual de cada regra da `mask_ruim`.

**Decisão:** Aplicar a `mask_ruim` completa, removendo linhas que violam qualquer regra dura.

**Justificativa:** Tais linhas são irrecuperáveis , e pela quantidade de 1.413%, não valendo a pena serem inferidas.

**Impacto nos dados:** 1.391 linhas (1,399%) removidas.

---

### Decisão 3 — `order_id` com múltiplos reviews

| Campo | Valor |
| ------- | ------- |
| Dataset | `olist_order_reviews` |
| Data da decisão | 23/11/2026 |
| Responsável | Álvaro Lúcio |
| Registro de IA | 4 (`uso_de_ia.md`) |

**Problema encontrado:** 1.098 `order_id` (~1,11%) em >1 linha. PK é `review_id`.

**Como foi detectado:** `df["order_id"].duplicated(keep=False).sum()`.

**Decisão:** Manter todas; `flag_order_multi_review`; agregação delegada à Frente 2.

**Justificativa:** Podem ser regra de negócio, como edição ou logs para fins jurídicos.

**Impacto:** 0 removidas; 1.098 sinalizadas.

**Alternativas descartadas:**

- Agregar por `order_id` — modelagem, não limpeza
- Remover duplicatas — perderia avaliações legítimas

---

### Decisão 3-b — Demais invalidezes em `reviews` (decomposição da `mask_ruim`)

| Campo | Valor |
| ------- | ------- |
| Dataset | `olist_order_reviews` |
| Data da decisão | 23/11/2026 |
| Responsável | Álvaro Lúcio |
| Registro de IA | 4 (`uso_de_ia.md`) |

**Problema encontrado:** Além das decisões 3 e 6, a `mask_ruim` de reviews remove linhas por `review_score` fora de `{1..5}`, `review_answer_timestamp < review_creation_date`, `review_id` duplicado, IDs vazios, datas inválidas.

**Como foi detectado:** Decomposição individual de cada regra da `mask_ruim`.

**Decisão:** Aplicar a `mask_ruim` completa, removendo linhas que violam qualquer regra dura.

**Justificativa:** Pela quantidade extremamente diminuta de dados inválidos, apenas 0.820%, portanto, podendo ser deletadas sem afetar as conclusões.

**Impacto nos dados:** 814 linhas (0,820%) removidas.
---

### Decisão 4 — `order_items` aderente

| Campo | Valor |
| ------- | ------- |
| Dataset | `olist_order_items` |
| Data da decisão | 23/11/2026 |
| Responsável | Álvaro Lúcio |
| Registro de IA | 2 (`uso_de_ia.md`) |

**Problema encontrado:** Nenhum. Auditoria sem invalidez dura em 112.650 linhas.

**Como foi detectado:** Auditoria completa (nulos, IDs, PK composta, `order_item_id <= 0`, datas, `price`/`freight_value < 0`).

**Decisão:** Nenhuma remoção. `limpar_items` só executa:

- cast `shipping_limit_date: object → datetime64`
- cast `order_item_id: int64 → int32`, `price/freight_value: float64 → float32`
- `strip` em IDs

**Justificativa:** Não foi necessário retirar linhas de `order_items`, pois já estava sem valores impossíveis, sendo feita apenas a normalização de tipos.

**Impacto:** 0 removidas (0,000%). Tipos normalizados.

---

### Decisão 5 — Não remover outliers em `price` / `freight_value` (caso de IA inadequada)

| Campo | Valor |
| ------- | ------- |
| Dataset | `olist_order_items` |
| Data da decisão | 23/11/2026 |
| Responsável | Álvaro Lúcio |
| Registro de IA | **1** (`uso_de_ia.md`) |

**Problema encontrado:** IA sugeriu remoção automática de outliers via `Q3 + 3·IQR`.

**Como foi detectado:** Checagem visual da função `auditar_items` — a regra não distingue valor extremo legítimo de valor inválido.

**Decisão:** **Rejeitada.** Nenhum outlier removido. Investigação manual quando relevante.

**Justificativa:** Outliers nesse caso não são necessariamente dados errados, podendo ser pedidos reais, promoções, e afins.

**Impacto:** 0 linhas removidas por outlier.

---

### Decisão 6 — Nulos em `review_comment_title` / `_message`

| Campo | Valor |
| ------- | ------- |
| Dataset | `olist_order_reviews` |
| Data da decisão | 23/11/2026 |
| Responsável | Álvaro Lúcio |
| Registro de IA | 4 (`uso_de_ia.md`) |

**Problema encontrado:** 88% sem `title`, 59% sem `message`. Nulos brutos.

**Como foi detectado:** Auditoria de nulos por coluna.

**Decisão:** Manter nulos. Normalizar strings vazias (`""`, `"   "`) para `pd.NA`.

**Justificativa:** Apenas ausência de comentários, não torna os dados inválidos.

**Impacto:** 0 removidas. Normalização de representação.

## 4. Colunas entregues com `flag_*` (handoff para Frente 2)

| Dataset | Coluna flag | Significado | Decisão de negócio (Frente 2) |
| --------- | ------------- | ------------- | ------------------------------- |
| `orders` | `flag_atraso` | Entrega após estimativa | Filtrar em análises de SLA? |
| `orders` | `flag_entrega_antes` | Entrega antes da estimativa | Considerar em NPS? |
| `orders` | `flag_sem_aprovacao` | `approved_at` nulo | Contexto por status |
| `orders` | `flag_sem_entrega` | `delivered_customer_date` nulo | Analisar por status |
| `reviews` | `flag_order_multi_review` | `order_id` com >1 review | Agregar por pedido? |
| `order_items` | (nenhuma) | — | — |
| `products` | (nenhuma) | — | — |

---
