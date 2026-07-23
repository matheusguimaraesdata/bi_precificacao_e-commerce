# Documentação do Modelo DAX — Precificação Ferrasoldas (Mercado Livre)

Descrição de cada medida e coluna calculada do modelo, para uso no campo **Description** do Power BI e como referência de consulta rápida sobre as regras de negócio por trás dos cálculos.

## dim_produtos

| Item | Tipo | Pasta | Descrição |
|---|---|---|---|
| `anuncio (display)` | Coluna | Formatação | Junta nome do anúncio e SKU em duas linhas, pra caber em cartão sem precisar de duas colunas separadas. |

## fact_anuncios
| Item | Tipo | Pasta | Descrição |
|---|---|---|---|
| `pesquisa (display)` | Coluna | Navegação | Campo de busca combinando anúncio e SKU, usado no slicer de seleção do simulador. |
| `atacado I (base)` | Medida | Simulação \| Atacado | Preço de atacado nível 1. Só retorna valor com um único anúncio selecionado — evita agregação errada quando o filtro pega mais de um item. |
| `atacado II (base)` | Medida | Simulação \| Atacado | Segundo nível de atacado, mesma trava de seleção única do nível 1. |
| `preco atacado I (display)` | Medida | Formatação | `atacado I (base)` formatado em R$. |
| `preco atacado II (display)` | Medida | Formatação | `atacado II (base)` formatado em R$. |
| `preco minimo (base)` | Medida | Simulação | Piso de venda cadastrado por anúncio — abaixo disso a venda passa a comprometer a margem. |
| `preco minimo (display)` | Medida | Formatação | `preco minimo (base)` formatado em R$. |
| `preco sugerido ajustado (base)` | Medida | Simulação | Preço sugerido com a margem extra aplicada por cima (parâmetro `Valor parametro margem extra`), usado como ponto de partida antes de qualquer desconto. |
| `preco sugerido` | Medida | Formatação | Versão formatada de `preco sugerido ajustado (base)`. |

## fact_estoque
| Item | Tipo | Pasta | Descrição |
|---|---|---|---|
| `estoque de anúncio (display)` | Coluna | Estoque | Estoque alocado ao anúncio, cruzado com `fact_anuncios` pelo SKU. É o que aparece disponível pro cliente no Mercado Livre. |
| `estoque de sistema (display)` | Coluna | Estoque | Estoque físico total no sistema interno — independe do que está alocado a anúncios, serve pra comparar com o estoque comercial. |

## Medidas gerais
| Item | Tipo | Pasta | Descrição |
|---|---|---|---|
| `preco simulado` | Medida | Simulação | Preço cheio com o desconto do slicer (`param_desconto`) aplicado — a simulação de preço promocional em si. |
| `preco cheio (base)` | Medida | Simulação | Espelha o preço sugerido ajustado; é a referência antes de qualquer desconto. |
| `preco final simulado` | Medida | Simulação | Trava de segurança: usa o maior valor entre `preco simulado` e `preco minimo`, então o desconto nunca derruba o preço abaixo do piso definido. |
| `preco minimo` | Medida | Simulação | Referência direta a `preco minimo (base)`, usada nas comparações e nos flags. |
| `valor comissao` | Medida | Custos | Comissão do marketplace em R$, calculada sobre o preço final simulado com o percentual do slicer `param_comissao`. |
| `custo total` | Medida | Custos | Soma de produto, taxas e frete por anúncio — vem direto da base de anúncios. |
| `receita liquida` | Medida | Resultado | Preço final simulado menos comissão do marketplace. |
| `lucro` | Medida | Resultado | Receita líquida menos custo total. |
| `margem %` | Medida | Resultado | Lucro sobre preço final simulado — o indicador que alimenta o status de margem. |
| `Atacado I (Simulado)` | Medida | Simulação \| Atacado | Preço de atacado nível 1 com desconto, comissão e redução adicional de 10%, típica desse tier de venda. |
| `Atacado II (Simulado)` | Medida | Simulação \| Atacado | Mesmo cálculo do nível 1, com redução de 20% pra refletir o volume maior de compra. |
| `diferenca %` | Medida | Resultado | Distância percentual entre preço final simulado e preço mínimo — mostra a folga (ou o risco) em relação ao piso. |
| `flag preco minimo` | Medida | Indicadores | Alerta quando o preço simulado cai abaixo do mínimo cadastrado. |
| `cor flag` | Medida | Indicadores | Hex de cor pra formatação condicional, com base na comparação entre preço simulado e mínimo. |
| `status margem` | Medida | Indicadores | Classifica a margem em três faixas: abaixo de 10% é prejuízo, entre 10% e 20% é margem baixa, acima disso é saudável. |
| `cor margem` | Medida | Indicadores | Hex de cor correspondente à faixa de `margem %`. |

## param_comissao
| Item | Tipo | Pasta | Descrição |
|---|---|---|---|
| `param_comissao` | Tabela | Parâmetros | Série de 0% a 30% (passo de 1%) pra simular diferentes comissões do marketplace via slicer. |
| `Valor Param Comissão` | Medida | Parâmetros | Comissão selecionada no slicer; padrão 15% quando nada é selecionado. |

## param_desconto
| Item | Tipo | Pasta | Descrição |
|---|---|---|---|
| `param_desconto` | Tabela | Parâmetros | Série de 0% a 50% (passo de 1%) pra simular percentuais de desconto promocional. |
| `Valor param_desconto` | Medida | Parâmetros | Desconto selecionado no slicer; padrão 0% quando nada é selecionado. |

## parametro_margem_extra
| Item | Tipo | Pasta | Descrição |
|---|---|---|---|
| `parametro_margem_extra` | Tabela | Parâmetros | Série de 0% a 20% (passo de 1%) pra testar reajuste sobre o preço sugerido antes do desconto. |
| `Valor parametro margem extra` | Medida | Parâmetros | Margem extra selecionada no slicer. |
| `preco mínimo (display)` | Medida | Formatação | Nome muito próximo de `preco minimo (display)` (fact_anuncios). |
