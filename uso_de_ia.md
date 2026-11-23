# Tabela de Uso de IA

| Registro | Data | Integrante | Ferramenta | Etapa e o que foi pedido | O que foi aproveitado | O que a equipe alterou ou verificou | Link da Conversa|
| --- | --- | --- | --- | --- | --- | --- | --- |
| 1-5 | 23/11 | Álvaro Lúcio | DeepSeek | Exploração, limpeza e tratamento de dados | Pipelines de auditoria e limpeza de dados | Pipelines de auditoria e limpeza de dados, e o registro 1. | [https://chat.deepseek.com/share/untmkblbbmukdbqkqf](https://chat.deepseek.com/share/t8b0jrlb840zq0eblr)|
## Caso de IA inadequada — registro 1

Contexto: IA sugeriu remover outliers de price/freight via Q3+3IQR
no pipeline de limpeza.

Por que foi rejeitado:

- Pois outliers nesse caso não são necessariamente dados errados, podendo ser pedidos reais, promoções, e afins.

Como percebemos:

- Checagem visual da função auditar_order_items.

O que fizemos:

- Remoção da retirada de outliers.

## Registro 2: Criação de duas pipelines para order_items, uma de auditoria e outra de limpeza

O que foi adicionado:

- Pipeline de verificação de df_orders_items
- Pipeline de limpeza de df_orders_items

## Registro 3: Criação de duas pipelines para orders, uma de auditoria e outra de limpeza

O que foi adicionado:

- Pipeline de verificação de df_orders
- Pipeline de limpeza de df_orders

## Registro 4: Criação de duas pipelines para order_reviews, uma de auditoria e outra de limpeza

O que foi adicionado:

- Pipeline de verificação de order_reviews
- Pipeline de limpeza de order_reviews

## Registro 5: Criação de duas pipelines para products, uma de auditoria e outra de limpeza

O que foi adicionado:

- Pipeline de verificação de products
- Pipeline de limpeza de products
