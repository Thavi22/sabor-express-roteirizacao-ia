# Descrição do problema, desafio proposto e objetivos

O projeto aborda a roteirização de entregas em ambiente urbano, com o objetivo de reduzir o tempo total e a distância percorrida para atender múltiplos pedidos distribuídos pela cidade. O desafio consiste em encontrar rotas eficientes quando há muitos pontos de entrega, cenário em que soluções manuais tendem a produzir percursos longos e pouco produtivos. Para enfrentar esse problema, a cidade foi modelada como um grafo: os locais de interesse (por exemplo, Centro, Estação e Mercado) são representados como nós com coordenadas geográficas (latitude e longitude), e as vias entre esses locais são representadas como arestas ponderadas por duas métricas: tempo em minutos e distância em quilômetros. O objetivo geral foi transformar dados simples (CSVs) em decisões operacionais reprodutíveis, capazes de comparar alternativas de roteirização com base em métricas objetivas.

# Explicação detalhada da abordagem adotada

A solução foi estruturada em etapas. Inicialmente, os arquivos nodes.csv e edges.csv são carregados para construir o grafo ponderado. Cada aresta armazena tempo_min (minutos, métrica principal de otimização) e distancia_km (quilômetros, usada como métrica de análise). Em seguida, é calculada uma rota com múltiplos pontos utilizando o algoritmo A*, concatenando menores caminhos na sequência origem → parada1 → parada2 → … e registrando as métricas de tempo total e distância total. A heurística do A* baseia-se na distância Haversine entre dois nós (linha reta sobre a esfera), convertida para minutos por meio de um fator “min/km” estimado a partir das próprias arestas do grafo; com isso, a unidade da heurística é compatível com o peso que está sendo otimizado.

Quando o número de pedidos é grande, aplica-se clustering com K-Means sobre pedidos.csv (latitude e longitude) para particionar a demanda em k.
k regiões geográficas. Cada pedido é associado ao nó do grafo mais próximo, e, para cada região, é construída uma rota específica. A ordem de visita interna adota a heurística do Vizinho Mais Próximo (Nearest Neighbor) pela métrica de tempo, enquanto os trechos entre pontos continuam sendo calculados com A*. Por fim, comparam-se dois cenários: (i) uma rota única sem clustering e (ii) a soma das rotas por região com clustering, sempre nas mesmas métricas (tempo e distância), permitindo avaliar o impacto do particionamento da demanda.

# Algoritmos Utilizados

A solução utiliza A* e K-Means. O A* (A-star) é empregado para o cálculo de menores caminhos em grafos ponderados, combinando custo acumulado com uma heurística admissível. A heurística adotada é a distância Haversine convertida para minutos, garantindo compatibilidade com o peso principal (tempo_min) e maior eficiência do que Dijkstra quando a heurística é informativa. O K-Means é utilizado para agrupar geograficamente os pedidos em k
k regiões, reduzindo a complexidade do problema quando a quantidade de pontos cresce. Como referência metodológica, Dijkstra foi considerado como baseline sem heurística e BFS/DFS como técnicas de exploração/alcance, mas não foram necessários para o objetivo central do trabalho.

# Diagrama do grafo/modelo usado na solução (gerado por código ou imagem estática)

O modelo é documentado por imagens geradas automaticamente pelo notebook, garantindo reprodutibilidade. O arquivo docs/grafo.png apresenta o grafo da cidade com nós e arestas. A figura docs/rota_multiplos_pontos.png mostra a rota A* com múltiplas paradas numeradas (0 = origem; 1..N = paradas), acompanhada das métricas de tempo e distância. O arquivo docs/clusters.png exibe o particionamento dos pedidos por K-Means com os centróides de cada região. No comparativo final, docs/rota_sem_cluster.png ilustra a rota única sem particionamento, enquanto docs/rota_regiao_*.png apresenta as rotas específicas por região.

# Análise dos resultados, eficiência da solução, limitações encontradas e sugestões de melhoria

Na execução atual, o cenário “sem cluster” resultou em 35,0 minutos e 9,20 km, enquanto a soma das rotas “com cluster” registrou 41,0 minutos e 11,20 km. Portanto, para esta configuração de dados e parâmetro k
k, o particionamento não produziu ganhos; isso pode ocorrer quando o valor de 
𝑘
k não reflete bem a estrutura espacial dos pedidos, quando todas as rotas partem do mesmo ponto (por exemplo, o Centro) e quando a sequência interna usa uma heurística simples como o Vizinho Mais Próximo, que não é ótima em termos globais. Em termos de eficiência, o A* foi adequado por utilizar uma heurística na mesma unidade do custo (minutos), reduzindo a expansão de nós frente a abordagens sem heurística, e o K-Means foi eficiente para agrupar rapidamente os pedidos em regiões exploráveis. As principais limitações da abordagem atual são: (i) ponto de partida único para todas as regiões, (ii) sequenciamento interno por NN (não ótimo como um solucionador clássico de TSP/VRP), (iii) ausência de informações de trânsito em tempo real e janelas de entrega, e (iv) escolha fixa de k
k. Como melhorias, recomenda-se: testar sistematicamente valores de k
k e selecionar aquele que optimize as métricas; definir um ponto inicial por região (por exemplo, o nó mais próximo do centróide) para reduzir deslocamentos iniciais; empregar melhoradores de rota como 2-opt para refinar a sequência interna; e incorporar, em extensões futuras, restrições de janelas de tempo e dados de trânsito para aproximar o problema de cenários operacionais reais.
