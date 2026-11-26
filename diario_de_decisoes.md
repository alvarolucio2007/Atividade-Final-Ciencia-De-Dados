# Diário de Decisões — Frente 1: Limpeza e Tratamento de Dados

**Projeto:** Ciência de Dados

**Integrantes da Frente 1:** Álvaro Lúcio

**Data de início:** 23/11/2026

**Última atualização:** 24/11/2026

**Versão:** v4

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

| Dataset               | Linhas recebidas | Linhas entregues | Removidas (%) |
| --------------------- | ---------------- | ---------------- | ------------- |
| `olist_orders`        | 99.441           | 98.021           | 1,427%        |
| `olist_order_reviews` | 99.224           | 99.224           | 0%            |
| `olist_order_items`   | 112.650          | 112.650          | 0%            |
| `olist_products`      | 32.951           | 32.339           | 1,8%          |

---

## 2. Visão geral das decisões

| Nº  | Etapa | Problema encontrado                                                                                 | Decisão tomada                                             | Justificativa (resumo)                                                                     | Impacto nos dados                      |
| :-- | :---: | :-------------------------------------------------------------------------------------------------- | :--------------------------------------------------------- | :----------------------------------------------------------------------------------------- | :------------------------------------- |
| 1   |   1   | Valores monetários são `float64` ao invés de `int64` em centavos.                                   | Avisar para a frente 2.                                    | Pode dar erros de aritmética de pontos flutuantes, tornando a análise errada.              | Nulo.                                  |
| 2   |   1   | Datetimes inconsistentes em `df_orders`.                                                            | Drop de todas as linhas que têm a inconsistência.          | Tais dados podem ser não recuperáveis, e são uma parte minúscula dos dados, menos de 1,5%. | Redução no tamanho total do dataframe. |
| 3   |   1   | Quantidade, comprimento, descrição, largura e altura inválidos (nulos ou iguais a 0) em `products`. | Drop de todas as linhas que possuem esses valores errados. | São valores logicamente impossíveis, e são menos de 2% dos dados.                          | Redução no tamanho total do dataframe. |

---
