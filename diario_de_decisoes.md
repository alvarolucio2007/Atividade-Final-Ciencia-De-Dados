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

# Diário de Decisões — Frente 2 transformações (codificação, normalização, seleção de atributos) e
feature engineering;

**Responsável:** Leticia Leandro\
**Data de início:** 26/11/2026

**Última atualização:** 27/11/2026


| Nº | Etapa                  | Problema encontrado                                              | Decisão tomada                                                           | Justificativa                                                              | Impacto nos dados                                                                                                               |
| -- | ---------------------- | ---------------------------------------------------------------- | ------------------------------------------------------------------------ | -------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| 1  | Codificação categórica | Categorias dos produtos estavam em formato textual.              | Aplicação do One-Hot Encoding, com `drop_first=True`.                    | Transformar as categorias em valores numéricos para facilitar as análises. | Criação de colunas binárias com prefixo `categoria_`.                                                                           |
| 2  | Normalização           | Variáveis numéricas apresentavam escalas diferentes.             | Aplicação do método Min-Max.                                             | Colocar os valores em uma escala de 0 a 1.                                 | Criação de quatro colunas: `price_minmax`, `freight_value_minmax`, `frete_percentual_preco_minmax` e `valor_total_item_minmax`. |
| 3  | Padronização           | Variáveis numéricas apresentavam médias e dispersões diferentes. | Aplicação do método Z-Score.                                             | Facilitar a comparação entre variáveis considerando média e desvio-padrão. | Criação de quatro colunas padronizadas com sufixo `_zscore`.                                                                    |
| 4  | Seleção de atributos   | Existiam diversas informações nas quatro bases utilizadas.       | Seleção das colunas necessárias para a análise e cruzamento das tabelas. | Manter os dados relevantes para estudar entregas, avaliações e fretes.     | Integração de informações de pedidos, itens, produtos e avaliações.                                                             |
| 5  | Feature Engineering    | Necessidade de identificar pedidos entregues com atraso.         | Criação da variável `pedido_atrasado`.                                   | Identificar se a entrega aconteceu depois da data estimada.                | Criação de uma coluna indicadora: 1 para atraso e 0 para entrega no prazo.                                                      |
| 6  | Feature Engineering    | Necessidade de identificar avaliações negativas dos clientes.    | Criação da variável `avaliacao_baixa`.                                   | Identificar avaliações com notas iguais ou inferiores a 2.                 | Criação de uma coluna indicadora para avaliações baixas.                                                                        |
| 7  | Feature Engineering    | Necessidade de analisar o peso do frete no preço dos produtos.   | Criação da variável `frete_percentual_preco`.                            | Calcular a porcentagem do preço que corresponde ao frete.                  | Criação de uma coluna com o percentual do frete em relação ao preço.                                                            |
| 8  | Feature Engineering    | Necessidade de conhecer o custo total de cada item.              | Criação da variável `valor_total_item`.                                  | Considerar o preço do produto junto com o valor do frete.                  | Criação de uma coluna somando `price` e `freight_value`.                                                                        |

---

# Diário de Decisões — Frente 3: Pipeline de Pré-processamento e Reprodutibilidade

**Projeto:** Ciência de Dados

**Integrantes da Frente 3:** Marcos Vinycius Cavalcante

**Data de início:** 26/11/2026

**Última atualização:** 28/11/2026

| Nº | Etapa                | Problema encontrado                                      | Decisão tomada                                      | Justificativa                                      | Impacto nos dados                                  |
|----|----------------------|----------------------------------------------------------|-----------------------------------------------------|----------------------------------------------------|----------------------------------------------------|
| 1  | Reprodutibilidade    | Resultados poderiam variar entre execuções.              | Definição de uma semente fixa (`42`).               | Garantir maior consistência nos resultados.       | Facilita a repetição do processamento.             |
| 2  | Controle de ambiente | Necessidade de registrar as versões das bibliotecas.     | Registro das versões do Pandas e NumPy.             | Documentar o ambiente utilizado.                  | Facilita a reprodução do projeto.                  |
| 3  | Validação            | Necessidade de verificar a consistência da pipeline.     | Comparação de duas execuções.                       | Confirmar se os resultados são iguais.            | Verificação da reprodutibilidade.                  |
| 4  | Integridade          | Possíveis alterações nas bases originais.                | Geração de hashes SHA-256.                          | Identificar mudanças nos dados.                   | Maior rastreabilidade das bases.                   |
| 5  | Documentação         | Falta de registro das configurações utilizadas.          | Criação do arquivo `registro_reprodutibilidade.json`. | Registrar as condições de processamento.        | Facilita consultas e futuras reproduções.          |
| 6  | Verificação de arquivos | Necessidade de conferir os arquivos finais.            | Verificação da existência dos arquivos.             | Garantir a disponibilidade dos arquivos da entrega.| Identificação de arquivos encontrados ou ausentes. |
