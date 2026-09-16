# DS# Priorização de clientes em campanha bancária

Projeto de aprendizado de máquina em Python para responder: **usando informações disponíveis antes da ligação, é possível priorizar clientes pela chance de contratar um depósito a prazo?**

[Abrir o notebook](Priorizacao_Clientes_Bank_Marketing_GitHub.ipynb) · [Fonte dos dados](https://archive.ics.uci.edu/dataset/222/bank+marketing)

## Como executar

1. Abra `Priorizacao_Clientes_Bank_Marketing_GitHub.ipynb` no Google Colab.
2. Execute as células em ordem com acesso à internet. A primeira célula de código baixa os dados diretamente da UCI; não é necessário copiar arquivos para o Drive.
3. As dependências usadas são `pandas`, `numpy` e `scikit-learn`, normalmente disponíveis no Colab. Em outro ambiente, instale-as com `pip install pandas numpy scikit-learn`.

O notebook público foi distribuído **sem saídas de execução e sem metadados pessoais**. Os números abaixo foram obtidos na execução original e podem ser reproduzidos ao executar todas as células.

## Método

- Dados: `bank-additional-full.csv`, 41.188 registros de campanhas de um banco português, ordenados por data no arquivo original.
- Alvo: contratação do depósito a prazo (`y = yes`).
- Variáveis: atributos do cliente e histórico de campanhas; na versão com contexto, mês, dia da semana e canal do contato planejado.
- `duration` foi excluída por só ser conhecida após a ligação; `campaign` também foi excluída porque inclui o contato atual.
- Divisão cronológica: 60% treino, 20% validação e 20% teste. Preprocessamento categórico e numérico é ajustado somente no treino.
- Modelos comparados na validação: regressão logística, random forest e regressão logística com variáveis de contexto. O escolhido foi reajustado nos primeiros 80% antes da avaliação final.
- Métricas: *average precision*, ROC-AUC e taxa de contratação entre os 10% de clientes com maior pontuação.

## Resultados

| Métrica | Validação (logística com contexto) | Teste final |
|---|---:|---:|
| Taxa de contratação na amostra | 11,1% | 30,8% |
| Average precision | 0,197 | 0,493 |
| ROC-AUC | 0,643 | 0,696 |
| Taxa de contratação no top 10% | 23,7% | 51,3% |
| Contratações no top 10% | 195 | 423 |
| Probabilidade média prevista | 6,9% | 17,4% |

No teste final, os 824 registros do top 10% reuniram **423 das 2.540 contratações**. A ordenação foi útil para priorizar contatos, mas a probabilidade média prevista (17,4%) ficou abaixo da taxa observada (30,8%).

## Decisões e limites

- O dicionário descreve `pdays = 999` como ausência de contato anterior, mas há 4.110 registros com esse código, `previous > 0` e `poutcome = failure`. O notebook usa `previous` para identificar histórico anterior e `pdays != 999` apenas como indicador de intervalo de dias informado.
- A taxa de contratação muda muito entre períodos: 4,8% no treino, 11,1% na validação e 30,8% no teste. Isso limita a extrapolação das probabilidades.
- A taxa e a composição do teste foram inspecionadas durante a exploração dos dados; assim, a avaliação não foi completamente cega. A escolha do modelo usou as métricas da validação. A ablação de `month` foi feita após consultar o teste e é apenas diagnóstica.
- `month` ajudou na validação, mas a base não traz um ano explícito por linha para separar sazonalidade de mudanças entre campanhas. Interpretar o mês como causa da contratação seria incorreto.
- O uso de `contact` pressupõe que o canal já esteja definido ao preparar a lista. Se essa informação só estiver disponível após a chamada, o modelo prospectivo deverá excluí-la e ser reavaliado.
- Os dados são históricos (Portugal, 2008–2010). Os resultados não demonstram desempenho em campanhas atuais ou em outras populações.

## Créditos

Dados: Moro, S., Rita, P., & Cortez, P. (2014), [*Bank Marketing*](https://doi.org/10.24432/C5K306), UCI Machine Learning Repository. O conjunto é distribuído sob [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). O notebook baixa os dados da UCI durante a execução; eles não estão incluídos neste repositório.
