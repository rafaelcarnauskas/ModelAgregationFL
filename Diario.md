# 28/10/2026

"clusterizar clientes pelos updates e agregar com peso igual entre clusters melhora o desempenho e a justiça em relação ao FedAvg sob dados heterogêneos?"

-estive meio distante da pesquisa ultimamente por demanda no poliquant e diversas provas aulas e labs.

-vou querer me aprofundar na melhora da acurácia do modelo por escolher melhor os pesos de acordo com uma clusterização dos mesmos. o fato do fedavg direcionar o modelo pra quem tem mais dados é algo que provavelmente tira acurácia

-a pergunta é: será que mesmo se eu fizer um modelo mais "justo", pensando no sentido de considerar também clientes com menos dados, eu consigo uma acurácia maior ou eu acabo perdendo acurácia por sacrificar acurácia em clientes com mais dados e não conseguir compensar em clientes com menos dados justamente por essa falta de dados?

-conversando com o claude decidi que o ideal é medir varias coisas para verificar se nao ha um enviesamento de metrica na resposta, o que ele disse foi "vale reportar as três: acurácia global, média por cliente e pior cliente (ou percentil 10), mais a variância entre clientes."

-o ganho de um unico modelo global é que tem coisas que da pra praticamente aperfeiçoar no aprendizado pois são comum a todos os clientes, como borda nas imagens aqui no flwr

-to com uma ideia de fazer um modelo personalizado por cluster de cliente. o claude disse para eu chamar de "aprendizado federado personalizado por agregação ponderada". minha ideia é agregar o modelo global de forma diferente para cada cluster, podendo ser o cluster algo mutável, e o modelo que retorna ao cluster é agregado com mais peso para os pesos que aquele cluster gerou. pensei em achar uma divisão de alguma forma estatística, tipo determinar a quantidade de informação nova gerada para x% do peso para os dados do cluster e (100-x)% dos dados dos outros clusters. preciso validar se uma abordagem como essa de nao deixar uma porcentagem fixa funciona ou so deixa ruidoso.

-nisso tem dois trabalhos parecidos, na seção 4.1.1, FedAMP [74] e FedFomo [75]

-papers sobre clusterização: HACCS [106]: clusteriza por metadados de distribuição de dados.
[107] (Lu et al.): clusteriza analisando os gradientes locais.
CFL (Sattler et al., clustered FL, não está no survey, mas é a referência clássica): separa clientes por similaridade de cosseno entre updates.

-todos esses papers sao referencias do doc que o prof mandou, vou colocar no repo também

-o projeto precisa primeiro tirar as metricas do baseline fedavg, depois testar a minha estratégia nas mesmas condições e validar isso

## formalização da ideia (rascunho)

- notação: K clusters, $S_k$ os clientes do cluster k, $n_c$ o número de amostras do cliente c, $w_k^t$ o modelo entregue ao cluster k na rodada t, e $Δ_c$ = $w_c - w_k^t$ o update do cliente c (vetor achatado)

- passo 1, clusterizar: depois de umas $T_0$ rodadas de fedavg com todos os clientes participando, agrupo os clientes pela similaridade de cosseno entre os $Δ_c$ (kmeans ou hierárquico). como o cluster pode ser mutável, dá pra refazer isso de tempos em tempos

- passo 2, agregar dentro do cluster: $\bar{w}_k = \sum_{c \in S_k} n_c^{\alpha} w_c / \sum_{c \in S_k} n_c^{\alpha}$, com α entre 0 e 1 (α=1 é fedavg dentro do cluster (media ponderada), α=0 todo mundo pesa igual (media simples)). serve pra cluster pequeno não ficar ruidoso demais -> dado que $\sum_{c \in S_k} n_c^{\alpha = 0} = n$ da o número de clientes e fica peso 1 para cada cliente

- passo 3, relevância entre clusters: $s_{kj} = \cos(\bar{\Delta}_k, \bar{\Delta}_j)$ e $r_{kj} = \exp(s_{kj}/\tau) / \sum_{i \neq k} \exp(s_{ki}/\tau)$, ou seja, quanto o cluster j é relevante pro cluster k

- passo 4, mistura: o x% da minha ideia é o λ. o cluster k recebe $w_k^{t+1} = \lambda \bar{w}_k + (1-\lambda) \sum_{j \neq k} r_{kj} \bar{w}_j$ onde esse lâmbda vai ser determinado pela fórmula mais abaixo. Aliás a determinação desse lâmbda é um objeto de estudo interessante para melhorias no algoritmo, existem diversas formas de escolher esse peso. Vou usar shrinkage mas tem vários outros.

- a parte de não deixar a porcentagem fixa é deixar o λ variar por cluster e por rodada, por exemplo em função de quão parecido o cluster é dos outros (se os updates são muito parecidos, λ menor, se são muito diferentes, λ maior). isso é o que preciso validar contra o λ fixo, porque pode só adicionar ruído

- casos extremos pra servir de controle: λ=1 é um modelo isolado por cluster, $λ=1/K$ com pesos iguais é um modelo global com peso igual por cluster, e K=1 com α=1 é o fedavg

- hiperparâmetros pra variar: K, $T_0$, α, τ
- risco conhecido: cluster pequeno gera $\bar{w}_j$ ruidoso que entra na mistura dos outros. e clusterizar por pesos é só um proxy, pesos parecidos não implicam dados iguais
- no flwr isso mexe também na parte que monta o que é enviado a cada cliente, não só na agregação, porque cada cluster recebe parâmetros diferentes
- Cálculo do $\lambda$ em cada cluster: $\lambda_k = \frac{n_k}{n_k + n_0}, n_k = \sum_{c \in S_k} n_c$
- lambda no cluster k confia mais nos seus dados quando eles são muitos ($\lambda$ próximo de 1) e confia nos dados dos outros quando tem poucos dados ($\lambda$ próximo de 0)
- $n_o$ é o cluster que confia 50% em si e 50% nos outros. precisa pesquisar mais para ter um método de achar esse cara
