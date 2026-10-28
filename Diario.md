28/10/2026
"clusterizar clientes pelos updates e agregar com peso igual entre clusters melhora o desempenho e a justiça em relação ao FedAvg sob dados heterogêneos?"
-estive meio distante da pesquisa ultimamente por demanda no poliquant e diversas provas aulas e labs.

-vou querer me aprofundar na melhora da acurácia do modelo por escolher melhor os pesos de acordo com uma clusterização dos mesmos. o fato do fedavg direcionar o modelo pra quem tem mais dados é algo que provavelmente tira acurácia

-a pergunta é: será que mesmo se eu fizer um modelo mais "justo", pensando no sentido de considerar também clientes com menos dados, eu consigo uma acurácia maior ou eu acabo perdendo acurácia por sacrificar acurácia em clientes com mais dados e não conseguir compensar em clientes com menos dados justamente por essa falta de dados?

-conversando com o claude decidi que o ideal é medir varias coisas para verificar se nao ha um enviesamento de metrica na resposta, o que ele disse foi "vale reportar as três: acurácia global, média por cliente e pior cliente (ou percentil 10), mais a variância entre clientes."

-o ganho de um unico modelo global é que tem coisas que da pra praticamente aperfeiçoar no aprendizado pois são comum a todos os clientes, como borda nas imagens aqui no flwr

-to com uma ideia de fazer um modelo personalizado por cluster de cliente. o claude disse para eu chamar de "aprendizado federado personalizado por agregação ponderada". minha ideia é agregar o modelo global de forma diferente para cada cluster, podendo ser o cluster algo mutável, e o modelo que retorna ao cluster é agregado com mais peso para os pesos que aquele cluster gerou. pensei em achar uma divisão de alguma forma estatística, tipo determinar a quantidade de informação nova gerada para x% do peso para os dados do cluster e (100-x)% dos dados dos outros clusters. preciso validar se uma abordagem como essa de nao deixar uma porcentagem fixa funciona ou so deixa ruidoso.

-nisso tem dois trabalhos parecidos, na seção 4.1.1, FedAMP [74] e FedFomo [75]
