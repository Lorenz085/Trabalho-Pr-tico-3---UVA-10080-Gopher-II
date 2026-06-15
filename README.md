# Grupo F — UVA 10080: Gopher II

Trabalho Prático 3 — Unidade 3
Disciplina: Resolução de Problemas com Grafos
Orientador: Prof. Me. Ricardo Carubbi

## Problema

- **Nome:** UVA 10080 — Gopher II
- **Link:** <https://onlinejudge.org/external/100/10080.pdf>
- **Plataforma de submissão:** <https://onlinejudge.org/>
- **Foco:** emparelhamento bipartido modelado como fluxo máximo com capacidades unitárias.

## Vídeo de apresentação

- **Link:** <https://youtu.be/DBA77k34Pz4>

## Integrantes do grupo

-  Vinni Lorenzo Gomes
-  Igor Romero Guerra
-  João Gabriel Leite Borges
-  Pedro Maia Araujo Costa

## Linguagem utilizada

Python 3 (sem bibliotecas externas de grafos; toda a lógica de fluxo foi implementada pelo grupo).

## Como executar a solução

A entrada é lida da entrada padrão (`stdin`) e a saída é escrita na saída padrão (`stdout`).

```bash
# executar passando um arquivo de entrada
python3 src/main.py < dados/entradas_do_problema.txt

# ou digitando a entrada manualmente e encerrando com Ctrl+D
python3 src/main.py
```

Não é necessário compilar nem instalar dependências. Basta ter o Python 3 instalado.

## Descrição do problema

Há `n` toupeiras (*gophers*) e `m` buracos, cada um em coordenadas `(x, y)`
distintas. Quando o falcão chega, uma toupeira que **não alcançar um buraco em
`s` segundos** fica vulnerável a ser comida. Cada buraco abriga **no máximo uma**
toupeira e todas correm à mesma velocidade `v`. O objetivo é **minimizar o número
de toupeiras vulneráveis**.

Como cada toupeira percorre no máximo `s · v` metros no tempo disponível, uma
toupeira **pode** usar um buraco se a distância entre eles for `≤ s · v`. Isso
define um grafo bipartido entre toupeiras e buracos, e o melhor aproveitamento
possível dos buracos é exatamente o **emparelhamento bipartido máximo**.

## Modelagem como rede de fluxo

A modelagem usa uma rede com origem, sorvedouro e duas camadas de vértices
(toupeiras e buracos).

### Vértices

- `source` (origem): vértice `0`.
- Uma camada com `n` vértices, **uma para cada toupeira**: vértices `1 .. n`.
- Uma camada com `m` vértices, **um para cada buraco**: vértices `n+1 .. n+m`.
- `sink` (sorvedouro): vértice `n+m+1`.

### Arestas e capacidades

- `source → toupeira_i`, capacidade `1`.
  Garante que **cada toupeira é salva por no máximo um buraco** (uma unidade de
  fluxo por toupeira).
- `toupeira_i → buraco_j`, capacidade `1`, **somente se** a toupeira `i` alcança
  o buraco `j` a tempo, isto é, se `dist(i, j) ≤ s · v`.
  Representa a compatibilidade "esta toupeira consegue chegar neste buraco".
- `buraco_j → sink`, capacidade `1`.
  Garante que **cada buraco salva no máximo uma toupeira**.

### Por que essas capacidades representam as restrições

Toda capacidade é **unitária** porque cada toupeira é uma entidade indivisível e
cada buraco comporta exatamente uma toupeira. Uma unidade de fluxo que sai da
origem, passa por uma toupeira, atravessa uma aresta de compatibilidade, entra
em um buraco e chega ao sorvedouro corresponde, no enunciado, a **uma toupeira
salva em um buraco alcançável**. Como as capacidades de entrada e de saída são
`1`, nenhuma toupeira é contada duas vezes e nenhum buraco recebe duas toupeiras.

### Comparação de distâncias

Para evitar erros de ponto flutuante, a verificação de alcance é feita **com
distâncias ao quadrado**, sem calcular raiz quadrada:

```
(gx − hx)² + (gy − hy)² ≤ (s · v)²
```

A comparação usa `≤`, então uma toupeira que está exatamente à distância máxima
do buraco ainda é considerada capaz de alcançá-lo.

## Algoritmo utilizado

**Edmonds-Karp** — que é o método de Ford-Fulkerson usando **BFS** para escolher
o caminho aumentante.

A escolha do BFS torna o número de iterações previsível e independente das
capacidades, e como o grafo é pequeno (`n, m < 100`) e de capacidades unitárias,
o desempenho é instantâneo. Ford-Fulkerson com DFS também resolveria, mas
Edmonds-Karp é a opção mais previsível, como sugerido pelo enunciado do trabalho.

## Papel do grafo residual

Cada aresta direta `u → v` possui uma **aresta reversa** `v → u` de capacidade
inicial `0`. No código, a reversa de uma aresta de índice `eid` é obtida por
`eid ^ 1` (as arestas são inseridas em pares).

A cada caminho aumentante encontrado:

1. calcula-se o **gargalo** (menor capacidade residual do caminho — aqui sempre `1`);
2. **subtrai-se** o gargalo das arestas diretas e **soma-se** nas reversas.

As arestas reversas permitem que o algoritmo **desfaça** uma escolha anterior:
se uma toupeira já ocupava um buraco que seria melhor aproveitado por outra, o
fluxo pode ser redirecionado pela aresta reversa, encontrando um emparelhamento
maior. A busca para quando **não existe mais caminho da origem ao sorvedouro** no
grafo residual.

## Como o resultado do fluxo vira a resposta

O **valor do fluxo máximo** é o número máximo de toupeiras que conseguem ocupar
buracos distintos — ou seja, o **emparelhamento bipartido máximo**. As toupeiras
restantes não têm buraco e ficam vulneráveis:

```
vulneráveis = n − fluxo_máximo
```

## Reconstrução / emparelhamento

O problema pede apenas a **quantidade** de toupeiras vulneráveis, portanto basta
o valor do fluxo. Caso fosse necessário recuperar o emparelhamento em si,
bastaria percorrer as arestas `toupeira_i → buraco_j` cuja capacidade residual
ficou `0` (fluxo positivo): cada uma indica uma toupeira salva em um buraco.

## Análise de complexidade

- Construção da rede: `O(n · m)` para testar todos os pares toupeira/buraco.
- Edmonds-Karp: `O(V · E²)` no pior caso. Com `V = n + m + 2` e `E = O(n · m)`,
  e `n, m < 100`, o custo é pequeno; na prática a execução é instantânea.
- Memória dominada pela **lista de arestas residuais**: `O(n · m)`.

## Casos especiais tratados

- **Vários casos de teste** na mesma entrada, lidos até o fim do arquivo (`EOF`).
- **Nenhum buraco alcançável:** o fluxo é `0` e todas as toupeiras ficam
  vulneráveis (`vulneráveis = n`).
- **Borda exata:** distância igual a `s · v` conta como alcançável (uso de `≤`).
- **Mais toupeiras do que buracos** (ou o contrário): a capacidade unitária nas
  duas pontas garante que o emparelhamento nunca excede `min(n, m)`.
- **Coordenadas em ponto flutuante:** comparação por distância ao quadrado evita
  `sqrt` e imprecisões.

## Evidência de Accepted

Ver a pasta `evidencias/` (`accepted.png` ou `accepted.pdf`).
