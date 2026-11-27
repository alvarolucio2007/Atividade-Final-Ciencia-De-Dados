# Relatório Final — Precificação de Frete no E-commerce Brasileiro

> **Rascunho de trabalho.** Os campos entre colchetes precisam ser preenchidos e conferidos pela equipe. Conforme as regras da disciplina, as respostas de negócio, o sumário executivo, as justificativas do diário e as conclusões devem ser escritos pela equipe com base nos resultados que ela calculou.

**Equipe:** [nome da equipe]  
**Integrantes e frentes:** [nomes e frentes]  
**Disciplina:** Ciência de Dados — Bacharelado em Engenharia de Software  
**Data:** [data de entrega]  
**Repositório:** [link]  
**Painel publicado:** [link]

---

## Parte A — Sumário executivo

*Limite: uma página. Redigir depois da validação dos indicadores e das respostas. Não preencher com suposições.*

### Pergunta 1 — Em quais estados o frete pesa mais no valor total do pedido?

**Resposta da equipe:** [informar estados e indicador calculado, com valores e volumes de pedidos.]  
**Recomendação:** [ação proposta pela equipe, sustentada pelos resultados.]

### Pergunta 2 — O peso e as dimensões do produto justificam o frete cobrado?

**Resposta da equipe:** [descrever o padrão observado nos dados e as evidências.]  
**Recomendação:** [ação proposta pela equipe, sustentada pelos resultados.]

### Pergunta 3 — Onde uma revisão da política de frete teria maior impacto para os clientes?

**Resposta da equipe:** [indicar região ou critério de priorização, com evidências.]  
**Recomendação:** [ação proposta pela equipe, sustentada pelos resultados.]

---

## Parte B — Relatório técnico

### 1. Contextualização do problema

O briefing recebido é da Área de Precificação de Frete. A área relata reclamações de que o frete seria caro em algumas regiões e quer avaliar o peso do frete no valor do pedido, a relação entre as características físicas dos produtos e o valor cobrado, e as regiões em que uma revisão da política poderia afetar mais clientes.

**Contexto complementar pesquisado pela equipe:** [inserir fontes, data de acesso e síntese escrita pela equipe.]

### 2. Perguntas de negócio e tradução para análise

As perguntas do cliente são:

1. Em quais estados o frete pesa mais no valor total do pedido?
2. O peso e as dimensões do produto justificam o frete cobrado?
3. Onde uma revisão da política de frete teria maior impacto para os clientes?

Antes de interpretar os resultados, a equipe deve declarar a unidade de análise, os denominadores e o tratamento de pedidos com vários itens. A base de trabalho existente está no nível do item do pedido; portanto, agregações por pedido ou estado precisam ser definidas explicitamente para evitar que pedidos com mais itens tenham peso implícito maior.

### 3. Datasets e relacionamentos

| Dataset | Uso na análise | Chave de relacionamento |
|---|---|---|
| `olist_orders_dataset.csv` | Identificação do pedido, cliente e datas/status disponíveis | `order_id`; `customer_id` para clientes |
| `olist_order_items_dataset.csv` | Preço e frete por item, produto e vendedor | `order_id`, `product_id`, `seller_id` |
| `olist_products_dataset.csv` | Peso, dimensões e categoria do produto | `product_id` |
| `olist_customers_dataset.csv` | Estado do cliente (destino) | `customer_id` |
| `olist_sellers_dataset.csv` | Estado do vendedor (origem declarada no cadastro) | `seller_id` |

O enunciado exige os três primeiros datasets e permite até dois complementares. Para este briefing, clientes e vendedores são os complementares sugeridos. O arquivo de geolocalização não deve ser utilizado.

**Datasets efetivamente usados e validação das junções:** [confirmar arquivos, tipo de junção, cardinalidade, linhas antes/depois e chaves sem correspondência.]

### 4. Ciclo de Vida da Ciência de Dados (CRISP-DM)

| Fase | Como foi aplicada neste projeto |
|---|---|
| Compreensão do negócio | Briefing de precificação de frete e três perguntas do cliente. |
| Compreensão dos dados | Inspeção de estrutura, tipos, ausências, distribuições e relações entre tabelas. |
| Preparação dos dados | Limpeza, conversões, junções e criação da tabela analítica. |
| Modelagem | Não se aplica: o enunciado proíbe modelos de IA, classificação e previsão. |
| Avaliação | [Descrever como a equipe conferiu indicadores, agregações, filtros e limitações.] |
| Entrega | Painel Streamlit publicado e relatório final. |

### 5. Exploração dos dados (EDA)

O notebook da equipe registra inspeções iniciais das tabelas de pedidos, itens, produtos e avaliações. Nos resultados disponíveis, `olist_order_items_dataset.csv` tem 112.650 linhas e 7 colunas; `olist_orders_dataset.csv`, 99.441 linhas e 8 colunas; e `olist_products_dataset.csv`, 32.951 linhas e 9 colunas. Esses números devem ser recalculados a partir dos arquivos efetivamente entregues e atualizados se a versão dos dados mudar.

Para o briefing de frete, documentem também:

- tipos, ausências, duplicatas e estatísticas de `price` e `freight_value`;
- distribuição de peso e dimensões dos produtos, incluindo extremos e unidades;
- número de pedidos, itens e clientes por estado;
- distribuição da proporção de frete no total do item/pedido;
- cobertura das junções e quantos itens ficam sem estado, peso ou dimensões;
- gráficos e tabelas que ajudem a equipe a validar os padrões antes de escrever conclusões.

**Achados descritivos calculados pela equipe:** [inserir tabela/gráficos e descrição sem inferir causalidade.]

### 6. Preparação e tratamento dos dados

#### 6.1 Limpeza e valores ausentes

O diário existente da Frente 1 registra decisões de tratamento para datas inconsistentes em pedidos e valores ausentes/inválidos em produtos. Antes de reutilizar esses dados no relatório, a equipe deve confirmar as regras diretamente no código executado, registrar o impacto em linhas e percentuais e verificar se essas exclusões fazem sentido para a análise de frete.

**Decisões efetivamente adotadas e impacto:** [referenciar números do diário consolidado e resultados recalculados.]

#### 6.2 Conversão e padronização de tipos

O notebook converte campos de data para datetime em pedidos, itens e avaliações. Para a tabela final, documentar também os tipos numéricos de preço, frete, peso e dimensões e conferir as unidades (por exemplo, gramas e centímetros nos campos do dataset de produtos).

#### 6.3 Categóricos e textos

O notebook trata categoria ausente com uma categoria explícita em trechos do pipeline. A equipe deve confirmar se essa regra permanece no produto final e explicar o tratamento dos estados e categorias usados nos filtros e gráficos.

#### 6.4 Codificação de dados categóricos

O notebook existente cria variáveis one-hot para categoria de produto. Descrever se essa codificação é necessária para as análises e para a tabela final; para apresentar resultados em painel, preservar também rótulos legíveis para gestores.

#### 6.5 Normalização e padronização

O notebook cria versões Min-Max e Z-score de variáveis monetárias e derivadas. Informar quais colunas foram escaladas, com quais parâmetros, e se os parâmetros são calculados em toda a base ou fixados para que o pipeline seja reprodutível em uma amostra.

#### 6.6 Seleção de atributos

O escopo exige examinar correlação, baixa variância e filtros simples. Registrar atributos examinados, critério aplicado e decisão da equipe. Não excluir variáveis relevantes ao briefing sem justificar a escolha.

#### 6.7 Feature engineering

O enunciado exige pelo menos quatro técnicas diferentes, relevantes às perguntas. Para cada atributo criado, documentar fórmula, origem e interpretação. Possíveis medidas a avaliar pela equipe (não são conclusões):

1. proporção do frete no valor total do item ou pedido;
2. volume geométrico do produto a partir de comprimento, largura e altura;
3. faixas de peso ou de volume para comparação descritiva;
4. agregações de valores e volumes por estado e período.

Confirmar que as quatro técnicas escolhidas são distintas, fazem sentido para os dados e estão presentes no pipeline entregue. Evitar chamar uma simples renomeação ou cópia de coluna de nova técnica.

### 7. Plano de análise das perguntas de negócio

> Esta seção define como calcular e comunicar evidências. A equipe deve executar as análises, interpretar os resultados e escrever as respostas com suas próprias palavras.

#### 7.1 Estados em que o frete pesa mais

**Definição a escolher e registrar:**

- Numerador: [soma do frete, média por pedido ou outra definição]
- Denominador: [valor total de itens + frete, valor dos itens ou outro]
- Unidade de cálculo: [item/pedido; explicar como pedidos com vários itens são consolidados]
- Estado considerado: estado do cliente (`customer_state`)
- Volume mínimo ou sinalização para grupos pequenos: [critério]

**Saída recomendada para a análise:** tabela ordenável por estado com indicador escolhido, número de pedidos, número de clientes e frete total. Mostrar mais de uma medida quando isso ajudar a distinguir participação percentual de impacto financeiro absoluto.

**Resultado calculado:** [preencher]  
**Resposta redigida pela equipe:** [preencher]

#### 7.2 Relação entre frete, peso e dimensões

**Variáveis a avaliar:** `freight_value`, `product_weight_g`, comprimento, largura, altura e volume calculado. A equipe pode comparar distribuições e associações, estratificando por estado, preço ou vendedor quando houver suporte suficiente nos dados.

**Verificações a registrar:** valores ausentes; produtos sem dimensões válidas; valores extremos; escala/unidade; forma de agregação; e resultados com e sem extremos, se isso for relevante.

**Limite de interpretação:** o dataset não informa distância real percorrida nem modalidade de transporte. Uma associação entre peso/dimensões e frete é evidência descritiva; sozinha, não prova que o valor cobrado é justo, que o peso causou o preço, ou que a política atual está correta.

**Método e resultados calculados:** [preencher]  
**Resposta redigida pela equipe:** [preencher]

#### 7.3 Região de maior impacto potencial

Definir previamente o que significa “impacto”. A equipe pode comparar, por estado, o volume de pedidos/clientes, o valor total de frete e a participação do frete no total pago. Se usar um índice composto, documentar variáveis, escalas, pesos e análise de sensibilidade; não apresentar um ranking como objetivo sem explicar a regra escolhida.

**Regra de priorização adotada:** [preencher]  
**Estados/regiões e evidências:** [preencher]  
**Resposta redigida pela equipe:** [preencher]

### 8. Pipeline de pré-processamento e reprodutibilidade

O notebook contém a função `preparar_dados(...)` e uma tentativa de fixar categorias e parâmetros de escala como referência. O relatório deve descrever a versão final realmente entregue e explicar como recebe os DataFrames brutos e devolve a tabela analítica, sem edição manual célula a célula.

**Registrar:** arquivos de entrada; validações; ordem das junções; tratamento de chaves ausentes/duplicadas; parâmetros de transformação; saídas; e resultado da execução com a amostra bruta usada na apresentação.

**Comando/instruções de execução:** [preencher]  
**Artefato da tabela analítica:** [caminho e tamanho]  
**Resultado da execução na amostra:** [preencher]

### 9. Painel Streamlit e comunicação

O requisito mínimo é uma página para cada pergunta do briefing, pelo menos dois filtros interativos, conclusões em linguagem de negócio em cada página e leitura do CSV final do repositório.

| Página | Indicador principal | Filtros | Gráfico/tabela | Conclusão escrita pela equipe |
|---|---|---|---|---|
| Pergunta 1 — Estado | [preencher] | [ex.: período, estado] | [preencher] | [a equipe redige] |
| Pergunta 2 — Produto | [preencher] | [ex.: faixa de peso, categoria] | [preencher] | [a equipe redige] |
| Pergunta 3 — Priorização | [preencher] | [ex.: estado, período] | [preencher] | [a equipe redige] |

**URL publicada e verificada em:** [preencher]

### 10. A limpeza alterou o comportamento dos dados?

Comparar indicadores selecionados antes e depois do tratamento, com a mesma definição e unidade de análise. Informar mudanças de cobertura, médias/medianas, distribuições e rankings relevantes. Distinguir alteração causada por remoção de registros da alteração causada por agregação ou transformação.

**Comparação feita pela equipe:** [preencher com valores, denominadores e interpretação própria.]

### 11. Limitações

Documentar limitações observadas na análise, por exemplo: ausência de distância real e modalidade de transporte; registros sem dimensões ou estado após as junções; granularidade por item; estados com poucos pedidos; período coberto pelo dataset; e decisões de limpeza que removem observações. Manter apenas limitações que a equipe tenha confirmado.

### 12. Conclusão da equipe

[Escrever pela equipe após validar as respostas, os resultados e as limitações. Não preencher com texto gerado por IA.]

---

## Parte C — Diário de decisões

Inserir aqui o diário consolidado conforme o Anexo B, ou anexá-lo como arquivo/seção. Cada linha deve ser escrita e validada pelos responsáveis pelas decisões.

| Nº | Etapa | Problema encontrado | Decisão tomada | Justificativa da equipe | Impacto nos dados |
|---:|---|---|---|---|---|
| 1 | [preencher] | [preencher] | [preencher] | [escrever pela equipe] | [preencher] |
| 2 | [preencher] | [preencher] | [preencher] | [escrever pela equipe] | [preencher] |
| 3 | [preencher] | [preencher] | [preencher] | [escrever pela equipe] | [preencher] |

---

## Apêndice — Checklist de finalização

- [ ] Conferir integrantes, frentes e briefing no README.
- [ ] Confirmar os datasets obrigatórios e os complementares utilizados.
- [ ] Validar cardinalidade e cobertura de cada junção.
- [ ] Documentar todas as colunas da tabela analítica no dicionário de dados.
- [ ] Confirmar quatro técnicas de feature engineering relevantes e implementadas.
- [ ] Confirmar que o pipeline funciona em recorte bruto sem ajustes manuais.
- [ ] Comparar resultados antes/depois da limpeza.
- [ ] Preencher as respostas e conclusões com análise e redação próprias da equipe.
- [ ] Garantir uma página do painel por pergunta, dois ou mais filtros e conclusão por página.
- [ ] Atualizar `uso_de_ia.md` e as marcações `[IA]` no código conforme o enunciado.
- [ ] Exportar o relatório para PDF e verificar links do repositório e do painel.
