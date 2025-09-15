# Descrição do problema, desafio e objetivos

A Sabor Express enfrenta atrasos e rotas ineficientes em horários de pico (almoço/jantar). O objetivo é reduzir tempo e distância das entregas, aumentando a satisfação do cliente e reduzindo custos. Para isso, a malha urbana foi modelada como um grafo: nós representam locais (Centro, Estação, Mercado etc., com latitude/longitude) e arestas representam ruas com dois pesos: tempo (min) e distância (km). A solução compara uma rota única (todos os pedidos juntos) com rotas por região (pedidos agrupados), usando métricas objetivas e reprodutíveis.

# Abordagem Adotada

1. Construção do grafo: leitura de nodes.csv (nós com nome, lat, lon) e edges.csv (arestas com tempo_min, distancia_km).

2. A* multipontos:
   
- para visitar uma sequência de paradas, A* é usado em cada trecho otimizando tempo (min); a heurística é a distância Haversine convertida para minutos por um fator médio “min/km” calculado a partir das arestas do grafo. Assim, a unidade da heurística é coerente com o custo.

4. Clustering (K-Means):
   
- quando há muitos pedidos (data/pedidos.csv), aplicamos K-Means em (lat, lon) para dividir em k regiões. Cada pedido recebe um rótulo cluster e é salvo em data/pedidos_clusterizados.csv.

5. Rotas por região:

- para cada região, os pedidos são associados ao nó de grafo mais próximo e a ordem de visita é definida por uma heurística simples (Vizinho Mais Próximo). Os trechos entre pontos continuam sendo calculados com A* (peso tempo_min).

7. Comparação de cenários:

- Sem cluster: rota única com todos os pedidos.

- Com cluster: uma rota por região; o tempo/distância totais são a soma das rotas regionais.

- Evidências são salvas em /docs e um resumo em CSV facilita a leitura.

# Algoritmos Utilizados

- A* (A-star): busca de menor caminho em grafo ponderado, combinando custo acumulado com heurística admissível. A heurística adotada é Haversine→minutos para compatibilidade com o peso de tempo.

- K-Means: agrupamento geográfico de pedidos em k regiões, reduzindo a complexidade quando a demanda cresce.

- Referências consideradas: Dijkstra (baseline sem heurística) e BFS/DFS (exploração/conectividade). Não foram necessários para a otimização principal com pesos.

# Diagrama do grafo/modelo usado na solução (gerado por código ou imagem estática)

O grafo e os resultados são gerados por código e exportados em /docs:

- grafo.png — nós/arestas da cidade;
- rota_multiplos_pontos.png — exemplo de rota A* com múltiplas paradas (no teste: 29 min e 7,80 km);
- clusters.png — regiões criadas por K-Means;
- rota_sem_cluster.png — rota única;
- rota_regiao_*.png — rotas por região;
- resumo_comparacao_final.csv e resumo_por_regiao_final.csv — tabelas para leitura rápida.
  
# Análise dos resultados, eficiência da solução, limitações encontradas e sugestões de melhoria

Em uma execução típica com k = 3, obtivemos: 

- Sem cluster: 35,0 min | 9,20 km.
- Com cluster (soma): 41,0 min | 11,20 km, com o detalhamento por região:
- Região 0: 21,0 min | 5,90 km
- Região 1: 11,0 min | 2,80 km
- Região 2: 9,0 min | 2,50 km
  
Para este conjunto de dados, o particionamento não trouxe ganho (tempo/distância aumentaram). Isso pode ocorrer quando:

- o valor de k não reflete a distribuição espacial da demanda;
- todas as rotas partem do mesmo ponto (ex.: Centro), gerando deslocamentos extras;
- a ordem interna usa Vizinho Mais Próximo (heurístico, não ótimo globalmente).

Eficiência: A* com heurística em minutos expande menos nós que Dijkstra e é rápido para rotas urbanas; K-Means é eficiente para particionar pedidos.
Limitações: ponto de partida único, ausência de trânsito/janelas de entrega, escolha fixa de k e ordenação NN.

Melhorias sugeridas:

- testar sistematicamente k (selecionar por métrica);
- definir um ponto inicial por região (ex.: nó mais próximo do centróide);
- aplicar 2-opt (oumeta-heurísticas) para refinar a ordem;
- incorporar trânsito/janelas de tempo para aproximar o cenário real.

#Sabor Express — Roteirização Inteligente de Entregas (Grafos + A* + K-Means)

Projeto acadêmico para otimizar rotas de entrega da Sabor Express na região central da cidade.
A cidade é modelada como grafo, usamos A* para encontrar menores caminhos em minutos e K-Means para dividir pedidos por regiões quando a demanda cresce. O repositório inclui dados em /data, evidências em /docs e notebook reprodutível em /src.

#Como executar:

1. Google Colab (recomendado)

2. Abra src/Grafo.ipynb no Colab.

3. No painel de arquivos, envie para a raiz (/content): data/nodes.csv, data/edges.csv, data/pedidos.csv.

4. Menu Ambiente de execução → Reiniciar e executar tudo.

5. Ao final, os resultados estarão em:

6. /docs/ (imagens de rotas e clusters);

7. /data/pedidos_clusterizados.csv (pedidos com a coluna cluster).

Execução local: 

1. pip install -r requirements.txt

2. jupyter notebook e abra src/Grafo.ipynb.

3. Garanta os mesmos CSVs em data/ e execute todas as células.
