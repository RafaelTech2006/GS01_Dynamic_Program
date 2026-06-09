# Monitoramento de Riscos Ambientais com Árvores, Grafos e Algoritmos
**FIAP — Global Solution 2026 | 1º Semestre | Disciplina: Dynamic Program**  
**Tema:** Economia Espacial — Dynamic Programming | **Prof.:** André Marques

---

## Identificação do Grupo

| RM | Nome |
|----|------|
| 563487 | Rafael Tavares Santos |
| 556170 | Felipe Yamaguchi Mesquita |
| 563872 | Gabriel Oliveira Amaral |


---

## Sumário

1. [Contextualização e Justificativa do Cenário](#1-contextualização-e-justificativa-do-cenário)
2. [Modelagem das Estruturas de Dados](#2-modelagem-das-estruturas-de-dados)
   - 2.1 [Grafo de municípios — justificativa da representação](#21-grafo-de-municípios--justificativa-da-representação)
   - 2.2 [BST por índice de risco — operações e balanceamento](#22-bst-por-índice-de-risco--operações-e-balanceamento)
   - 2.3 [Tabela de estruturas de dados](#23-tabela-de-estruturas-de-dados-justificada)
3. [Algoritmos Implementados](#3-algoritmos-implementados)
   - 3.1 [Força Bruta — backtracking e explosão combinatória](#31-força-bruta--backtracking-e-explosão-combinatória)
   - 3.2 [Algoritmo Guloso — Dijkstra e prova de corretude local](#32-algoritmo-guloso--dijkstra-e-prova-de-corretude-local)
   - 3.3 [Comparação entre os algoritmos](#33-comparação-entre-os-algoritmos)
4. [Monitoramento de Desempenho](#4-monitoramento-de-desempenho)
5. [Escala de Decisão](#5-escala-de-decisão)
6. [Figuras Obrigatórias — interpretações](#6-figuras-obrigatórias--interpretações)
7. [Estrutura do Repositório e Módulos](#7-estrutura-do-repositório-e-módulos)
8. [Instalação e Execução](#8-instalação-e-execução)
9. [Testes Automatizados](#9-testes-automatizados)
10. [Fontes de Dados e Referências](#10-fontes-de-dados-e-referências)
11. [Conexão com os ODS da ONU](#11-conexão-com-os-ods-da-onu)

---

## 1. Contextualização e Justificativa do Cenário

### O problema

O Brasil é um dos países mais vulneráveis às mudanças climáticas. Eventos como a **seca histórica no Rio Amazonas (2023)**, as **enchentes no Rio Grande do Sul (2024)** e o **desmatamento contínuo na Amazônia** exigem sistemas capazes de organizar, percorrer e analisar redes complexas de dados geoespaciais em tempo real. Satélites como os da constelação **Sentinel (ESA)** e o **GOES-16** transmitem dados contínuos de temperatura, precipitação e cobertura vegetal — informações que precisam ser estruturadas e processadas com eficiência computacional para subsidiar decisões de defesa civil.

Este projeto foi desenvolvido para um consórcio de defesa civil e cooperativas agrícolas, com o objetivo de construir um **sistema de monitoramento e triagem de riscos ambientais em municípios brasileiros**, capaz de:

- Representar municípios e suas conexões como um grafo ponderado;
- Organizar dados de risco em uma BST para consultas eficientes;
- Encontrar rotas ótimas de atendimento a partir de um hub de recursos;
- Comparar Força Bruta e algoritmo Guloso em qualidade, tempo e memória.

### Cenário escolhido — Enchentes no Rio Grande do Sul (Cenário A)

**Justificativa da escolha com base em disponibilidade de dados e relevância social:**

O Cenário A foi selecionado por duas razões principais. Do ponto de vista da **relevância social**, as enchentes de abril–maio de 2024 no RS constituem o maior desastre climático da história do estado, afetando 478 municípios, deslocando mais de 581 mil pessoas e causando ao menos 169 mortes — um evento que evidencia, com dados concretos e recentes, a necessidade urgente de sistemas de triagem e roteamento de emergência. Do ponto de vista da **disponibilidade de dados**, a malha viária federal está disponível no DNIT em formato shapefile georreferenciado, e a Defesa Civil do RS publicou relatórios municipais de impacto que permitem calibrar índices de risco por município. Os dados utilizados neste projeto são **sintéticos, porém calibrados** sobre essas fontes reais: as distâncias entre municípios refletem a malha viária DNIT e os índices de risco refletem o grau de afetação registrado nos boletins da Defesa Civil. A escolha de dados sintéticos se deu pela limitação de acesso a dados georreferenciados abertos em granularidade municipal com todos os atributos exigidos pelo modelo (risco, custo, população) em uma única fonte estruturada.

O sistema foi instanciado com **Porto Alegre como hub de recursos** (origem das equipes de atendimento) e o objetivo é encontrar a **rota de menor custo até cada município afetado**, priorizando os de maior risco ambiental via consulta à BST.

---

## 2. Modelagem das Estruturas de Dados

### 2.1 Grafo de municípios — justificativa da representação

O grafo `G = (V, E)` representa a rede de municípios. Cada **vértice** `v ∈ V` é armazenado como uma **tupla imutável**:

```python
# (id_municipio, nome, indice_risco, custo_atendimento_R$mil, populacao)
v = (9, 'Mucum', 0.95, 350, 10000)
```

Cada **aresta** `e = (u, v, peso) ∈ E` representa uma rota entre municípios, com peso = distância em km.

**Justificativa — lista de adjacência vs. matriz de adjacência:**

A escolha da estrutura de representação impacta diretamente as operações que o sistema realiza. As operações centrais deste projeto são: (1) **iterar sobre todos os vizinhos de um vértice** — feito repetidamente pelo Dijkstra e pela Força Bruta ao percorrer arestas; e (2) **verificar se dois vértices são adjacentes** — feito pelo controle de ciclos no backtracking.

| Operação | Lista de Adjacência | Matriz de Adjacência |
|---|---|---|
| Espaço total | `O(V + E)` | `O(V²)` |
| Iterar vizinhos de `v` | `O(grau(v))` ✅ | `O(V)` — percorre linha inteira |
| Verificar aresta (u,v) | `O(grau(u))` | `O(1)` |
| Adicionar aresta | `O(1)` | `O(1)` |
| Adequação | Grafos **esparsos** ✅ | Grafos densos |

Para este problema, a iteração sobre vizinhos ocorre a cada relaxamento de aresta no Dijkstra — ela é a operação dominante. Com a lista de adjacência, essa iteração custa `O(grau(v))`, tipicamente pequeno (4–6 vizinhos por município na malha viária). Com a matriz, custaria `O(V)` mesmo que o município tenha apenas 3 vizinhos, percorrendo `V − 3` zeros desnecessariamente. Em termos de espaço, para `V = 478` municípios com ~5 vizinhos cada: a **lista usa ~2.390 entradas**; a **matriz usaria `478² ≈ 228.000` entradas**, sendo ~95% delas zeros. A lista de adjacência é portanto a representação correta para grafos esparsos como a malha viária brasileira.

```python
class Grafo:
    def __init__(self):
        self.vertices    = {}   # dict: id -> (id, nome, risco, custo, pop)
        self.adjacencias = {}   # dict: id -> [(vizinho, peso), ...]

    def adicionar_municipio(self, vertice_tupla):
        id_mun = vertice_tupla[0]
        self.vertices[id_mun] = vertice_tupla
        if id_mun not in self.adjacencias:
            self.adjacencias[id_mun] = []

    def adicionar_rota(self, origem, destino, peso):
        """Aresta bidirecional — grafo não-direcionado."""
        self.adjacencias[origem].append((destino, peso))
        self.adjacencias[destino].append((origem, peso))
```

---

### 2.2 BST por índice de risco — operações e balanceamento

A BST é construída sobre os vértices do grafo usando o **índice de risco** como chave de ordenação. Permite identificar municípios críticos em `O(log N)` durante triagem de emergência, sem uso de bibliotecas externas.

**Propriedade BST mantida:** `risco_esquerda < risco_pai ≤ risco_direita`

**Operações implementadas do zero (classes `Node` e `BinarySearchTree`):**

| Operação | Descrição | Complexidade Média | Pior Caso |
|---|---|---|---|
| `inserir(nó)` | Insere mantendo propriedade BST por risco | O(log N) | O(N) — árvore degenerada |
| `buscar(r_min, r_max)` | Retorna municípios no intervalo de risco | O(log N + k) | O(N) |
| `percurso_in_order()` | Municípios em ordem crescente de risco | O(N) | O(N) |
| `altura()` | Calcula altura e avalia balanceamento | O(N) | O(N) |
| `remover(id)` | Remove nó e reequilibra ponteiros (substitui pelo sucessor in-order) | O(log N) | O(N) |

**Sobre o balanceamento:** a `altura()` é usada para avaliar se a árvore está degenerando em lista encadeada. Uma BST com N nós tem altura ideal `⌊log₂ N⌋` (árvore perfeitamente balanceada). Se a altura medida for muito superior a `log₂ N`, significa que os municípios foram inseridos em ordem quase-crescente ou quase-decrescente de risco, produzindo uma cadeia linear com busca `O(N)` em vez de `O(log N)`. Para os 10 municípios do cenário RS, `log₂(10) ≈ 3.3`; a altura observada é 5, indicando eficiência de busca **boa, porém não ótima** — aceitável para o tamanho da instância.

```python
class Node:
    def __init__(self, vertice_tupla):
        self.municipio = vertice_tupla   # (id, nome, risco, custo, pop)
        self.left  = None
        self.right = None

class BinarySearchTree:
    def __init__(self):
        self.root = None

    def inserir(self, vertice_tupla):
        novo_no = Node(vertice_tupla)
        if self.root is None:
            self.root = novo_no
        else:
            self._inserir_recursivo(self.root, novo_no)

    def _inserir_recursivo(self, atual, novo_no):
        if novo_no.municipio[2] < atual.municipio[2]:
            if atual.left is None: atual.left = novo_no
            else: self._inserir_recursivo(atual.left, novo_no)
        else:
            if atual.right is None: atual.right = novo_no
            else: self._inserir_recursivo(atual.right, novo_no)

    def buscar(self, risco_min, risco_max):
        """
        Busca por intervalo em O(log N + k).
        Poda a subárvore esquerda se risco_atual <= risco_min
        e a direita se risco_atual >= risco_max.
        """
        resultados = []
        self._buscar_recursivo(self.root, risco_min, risco_max, resultados)
        return resultados

    def percurso_in_order(self):
        """In-order: esquerda → raiz → direita → produz ordem crescente de risco."""
        resultado = []
        self._in_order_recursivo(self.root, resultado)
        return resultado

    def altura(self):
        """Altura da árvore. Comparar com log₂(N) para avaliar balanceamento."""
        return self._altura_recursiva(self.root)

    def remover(self, id_municipio):
        """Remove por id. Substitui nó com dois filhos pelo sucessor in-order (mínimo da subárvore direita)."""
        municipio = self._buscar_por_id(self.root, id_municipio)
        if municipio is not None:
            self.root = self._remover_recursivo(self.root, municipio[2])
```

**Integração BST → Dijkstra:** antes de executar o roteamento, a BST é consultada para identificar os destinos prioritários:

```python
# Municípios de alto risco (≥ 0.80) identificados em O(log N)
alto_risco = bst_rs.buscar(0.80, 1.00)
# Resultado: Lajeado (0.85), Roca Sales (0.88), Encantado (0.90), Mucum (0.95)
# → esses são os destinos para os quais o Dijkstra calcula rotas prioritárias
```

---

### 2.3 Tabela de estruturas de dados justificada

O projeto usa explicitamente cada estrutura abaixo. A coluna de justificativa explica **por que a estrutura foi escolhida**, não apenas onde foi usada.

| Estrutura | Onde foi usada | Aplicação concreta | Justificativa de complexidade |
|---|---|---|---|
| `list` | Adjacência do grafo; resultado de busca BST; caminho reconstruído | Lista de vizinhos de cada município; sequência de nós no caminho ótimo | `O(V+E)` espaço; `O(grau(v))` para iterar vizinhos — ideal para grafos esparsos |
| `tuple` | Vértices e arestas do grafo | `(id, nome, risco, custo, pop)` — atributos de cada município | Imutabilidade garante integridade dos dados; menor overhead de memória que `dict` |
| `dict` | Grafo (adjacências, vértices); distâncias e predecessores no Dijkstra | `adjacencias[id] → [(viz, peso)]`; `distancias[id] → float` | `O(1)` acesso médio — essencial para que o Dijkstra não pague `O(V)` por lookup |
| `set` | Vértices visitados no Dijkstra; controle de ciclos na Força Bruta | `visitados.add(v)` e `v in visitados` a cada iteração | `O(1)` pertencimento vs. `O(N)` em lista — crítico no backtracking com milhares de chamadas |
| `heapq` | Fila de prioridade do Dijkstra | `heappush(fila, (dist, v))` e `heappop(fila)` | `O(log V)` extração do mínimo — viabiliza complexidade total `O((V+E) log V)` |
| `BST (Node)` | Triagem de municípios por risco antes do roteamento | Busca por intervalo `[0.80, 1.00]`; in-order para listar por criticidade | `O(log N)` busca/inserção vs. `O(N)` em lista não-ordenada — decisivo em emergências |
| `Grafo (dict of lists)` | Rede de municípios e rotas de atendimento | Grafo ponderado não-direcionado; suporte ao Dijkstra e ao backtracking | `O(V+E)` espaço — representação mais eficiente para a malha viária brasileira (esparsa) |

---

## 3. Algoritmos Implementados

### 3.1 Força Bruta — backtracking e explosão combinatória

A Força Bruta enumera **todos os caminhos simples** entre origem e destino por recursão com backtracking, garantindo o **ótimo global**. Ela serve como **oráculo de validação**: para toda instância com `N ≤ 12`, o custo retornado pela Força Bruta é a referência contra a qual o Dijkstra é comparado.

**Papel de oráculo:** se o Dijkstra retornar o mesmo custo que a Força Bruta, o gap é 0% e a corretude do Guloso é confirmada empiricamente. Se retornar custo maior, o gap positivo quantifica a perda de otimalidade. Para Dijkstra com pesos não-negativos, o gap observado é sempre 0% — o que é consistente com a prova teórica.

**Complexidade de tempo:** `O(N!)` no pior caso — cada nó acrescenta um fator multiplicativo ao número de caminhos a explorar.  
**Complexidade de espaço:** `O(N)` — pilha de recursão proporcional ao comprimento do caminho atual.

```python
def forca_bruta(grafo, origem, destino):
    """
    Enumeração completa por backtracking.

    Estruturas:
      - set  (visitados): O(1) para checar ciclos — evita revisitar nós
      - list (caminho):   sequência atual sendo construída

    Returns: (melhor_caminho, melhor_custo, chamadas_recursivas, caminhos_avaliados)
    """
    contador = [0, 0]           # [chamadas_recursivas, caminhos_completos_avaliados]
    melhor   = [None, float('inf')]

    def _backtracking(atual, visitados, caminho, custo_atual):
        contador[0] += 1
        visitados.add(atual)
        caminho.append(atual)

        if atual == destino:
            contador[1] += 1
            if custo_atual < melhor[1]:
                melhor[0] = caminho.copy()
                melhor[1] = custo_atual
        else:
            for vizinho, peso in grafo.adjacencias[atual]:
                if vizinho not in visitados:
                    _backtracking(vizinho, visitados, caminho, custo_atual + peso)

        caminho.pop()          # backtrack: desfaz a escolha
        visitados.remove(atual)

    _backtracking(origem, set(), [], 0)
    return melhor[0], melhor[1], contador[0], contador[1]
```

**Por que a Força Bruta se torna inviável e a partir de qual N:**

O número de caminhos simples em um grafo cresce, no pior caso, como `N!` (fatorial). Cada nó adicional multiplica o espaço de busca pelo número de vértices restantes não visitados. Isso é fundamentalmente diferente do crescimento polinomial ou `O(N log N)` — não existe melhoria de hardware capaz de compensar crescimento fatorial para instâncias maiores. Empiricamente, o **cruzamento das curvas de tempo** ocorre em torno de `N ≈ 8–10`: abaixo disso, a Força Bruta é comparável ao Dijkstra; acima, o gap cresce rapidamente. Para `N = 12`, a FB já demora ~15ms vs. ~0.1ms do Dijkstra. Para `N = 20`, extrapolando o crescimento observado, a Força Bruta levaria **horas**. A fronteira prática é `N ≤ 12` para uso como baseline de validação — instâncias maiores devem obrigatoriamente usar o algoritmo Guloso.

| N | Chamadas recursivas | Caminhos avaliados | Tempo FB (ms) | Tempo Dijkstra (ms) |
|---|---|---|---|---|
| 5  | ~30       | ~5     | < 0.1  | ~0.05 |
| 8  | ~1.000    | ~40    | ~0.5   | ~0.08 |
| 10 | ~10.000   | ~200   | ~2.0   | ~0.10 |
| 12 | ~100.000  | ~1.200 | ~15.0  | ~0.12 |
| 20 | > 10¹⁸   | —      | **inviável** | ~0.18 |

---

### 3.2 Algoritmo Guloso — Dijkstra e prova de corretude local

O **Dijkstra** foi escolhido como variante Greedy por ser a solução exata para o problema de **caminho mínimo de fonte única** com pesos não-negativos — exatamente o problema do cenário RS (distâncias em km ≥ 0). Prim seria mais adequado se o objetivo fosse construir uma MST de cobertura total; Kruskal também, mas ordenando todas as arestas. Dijkstra é a escolha certa quando se quer a rota mais curta de **um hub específico** (Porto Alegre) até um ou todos os municípios afetados.

**Critério de escolha local (razão Greedy):** a cada iteração, o algoritmo extrai do heap o vértice `u` com a **menor distância acumulada ainda não finalizado** e relaxa todas as suas arestas — ou seja, atualiza a estimativa dos vizinhos se um caminho mais curto for encontrado passando por `u`. A decisão local é sempre "processe o vértice mais próximo da origem ainda pendente".

**Prova informal de corretude da escolha local:** ao extrair `u` do heap com distância `d[u]`, já sabemos que `d[u]` é mínima. Por quê? Qualquer caminho alternativo até `u` obrigatoriamente passa por algum vértice `w` ainda no heap, com `d[w] ≥ d[u]` (pois o heap sempre retorna o menor). Como os pesos das arestas são não-negativos, estender o caminho de `w` até `u` só poderia aumentar (ou manter) o custo — nunca diminuir. Logo, não existe caminho mais curto que `d[u]`, e a decisão local de finalizar `u` neste momento é sempre correta. Esse argumento **não se manteria** se existissem pesos negativos, razão pela qual o Dijkstra requer `peso ≥ 0`.

**Complexidade de tempo:** `O((V + E) log V)` com heap binário.  
**Complexidade de espaço:** `O(V + E)` — grafo + heap + dicionários de distâncias e predecessores.

```python
def dijkstra(grafo, origem):
    """
    Caminho mínimo de fonte única — O((V+E) log V).

    Estruturas:
      - dict (distancias):    O(1) acesso à estimativa atual de cada vértice
      - dict (predecessores): O(1) para reconstrução do caminho
      - heapq (fila):         O(log V) extração do vértice mais próximo
      - set (visitados):      O(1) para checar se vértice já foi finalizado

    Returns: (distancias, predecessores, arestas_relaxadas)
    """
    distancias    = {v: float('inf') for v in grafo.adjacencias}
    predecessores = {v: None         for v in grafo.adjacencias}
    distancias[origem] = 0

    fila      = [(0, origem)]   # heap: (distancia_acumulada, id_vertice)
    visitados = set()
    operacoes = 0               # contador de arestas relaxadas

    while fila:
        dist_atual, v_atual = heapq.heappop(fila)   # extrai o mais próximo

        if v_atual in visitados:
            continue            # já finalizado — descarta entrada obsoleta do heap
        visitados.add(v_atual)

        for vizinho, peso in grafo.adjacencias[v_atual]:
            operacoes += 1
            nova_dist = dist_atual + peso
            if nova_dist < distancias[vizinho]:     # relaxamento
                distancias[vizinho]    = nova_dist
                predecessores[vizinho] = v_atual
                heapq.heappush(fila, (nova_dist, vizinho))

    return distancias, predecessores, operacoes


def reconstruir_caminho(predecessores, origem, destino):
    """Reconstrói o caminho ótimo percorrendo o dict de predecessores de trás para frente."""
    caminho, atual = [], destino
    while atual is not None:
        caminho.append(atual)
        atual = predecessores[atual]
    caminho.reverse()
    return caminho if caminho[0] == origem else []  # vazio se destino inacessível
```

**Resultado no cenário RS:**

```
Municípios de alto risco identificados pela BST (risco >= 0.80):
  Lajeado (0.85), Roca Sales (0.88), Encantado (0.90), Mucum (0.95)

Rota ótima Porto Alegre → Muçum (maior risco = 0.95):
  Porto Alegre → Santa Cruz do Sul → Lajeado → Encantado → Muçum
  Custo total: 4.30 km  |  Arestas relaxadas: 18
```

---

### 3.3 Comparação entre os algoritmos

| Critério | Força Bruta | Dijkstra (Guloso) |
|---|---|---|
| Garantia de otimalidade | ✅ Sempre ótimo (exaustão) | ✅ Ótimo para pesos ≥ 0 (provado) |
| Complexidade de tempo | O(N!) | O((V+E) log V) |
| Complexidade de espaço | O(N) | O(V+E) |
| Viável para N = 12 | ✅ ~15ms | ✅ ~0.1ms |
| Viável para N = 478 | ❌ Impossível | ✅ Milissegundos |
| Gap de otimalidade | 0% (referência) | 0% (confirmado empiricamente) |
| Papel no sistema | Validação / testes unitários | Produção / instâncias reais |

---

## 4. Monitoramento de Desempenho

O módulo `performance_monitor.py` envolve qualquer algoritmo e registra três métricas independentes:

```python
def monitorar(funcao, *args):
    """
    Métricas coletadas:
      - Tempo: time.perf_counter() — precisão de nanosegundos, sem overhead de I/O
      - Memória: tracemalloc — rastreia alocações Python, retorna pico em bytes
      - Operações: retornadas pelo próprio algoritmo (arestas relaxadas / chamadas recursivas)
    """
    tracemalloc.start()
    inicio = time.perf_counter()

    resultado = funcao(*args)

    fim = time.perf_counter()
    _, memoria_pico = tracemalloc.get_traced_memory()
    tracemalloc.stop()

    return {
        "resultado":  resultado,
        "tempo_ms":   (fim - inicio) * 1000,
        "memoria_mb": memoria_pico / (1024 * 1024),
    }
```

**Tamanhos testados:** `N ∈ {5, 8, 10, 12, 20, 50, 100}`

### Resultados empíricos

| N | Dijkstra (ms) | FB (ms) | Chamadas FB | Gap (%) | Memória Dijkstra (MB) | Memória FB (MB) |
|---|---|---|---|---|---|---|
| 5  | ~0.05 | ~0.08  | ~30      | 0.0% | < 0.01 | < 0.01 |
| 8  | ~0.08 | ~0.50  | ~1.000   | 0.0% | < 0.01 | ~0.01  |
| 10 | ~0.10 | ~2.00  | ~10.000  | 0.0% | < 0.01 | ~0.05  |
| 12 | ~0.12 | ~15.0  | ~100.000 | 0.0% | < 0.01 | ~0.30  |
| 20 | ~0.18 | N/A    | —        | —    | < 0.01 | —      |
| 50 | ~0.40 | N/A    | —        | —    | ~0.01  | —      |
| 100| ~1.20 | N/A    | —        | —    | ~0.02  | —      |

### Análise do cruzamento das curvas e significado prático

O **cruzamento das curvas de tempo** ocorre empiricamente em `N ≈ 8–10`. Abaixo desse ponto, ambos os algoritmos executam em frações de milissegundo e a diferença é imperceptível na prática. **A partir de N = 10**, a curva da Força Bruta passa a crescer visivelmente mais rápido que a do Dijkstra. Para `N = 12`, a FB já é ~125× mais lenta. O significado prático desse cruzamento é direto: **para qualquer instância com mais de ~10 municípios, o algoritmo Guloso é obrigatório**. No contexto de resposta a desastres — onde decisões precisam ser tomadas em segundos — a Força Bruta é inaceitável mesmo para instâncias moderadas. O Dijkstra, com crescimento `O((V+E) log V)`, permanece viável mesmo para os 478 municípios do cenário real do RS.

O **gap de 0%** em todos os tamanhos testados (N ≤ 12) confirma empiricamente que o Dijkstra não sacrifica qualidade para ganhar velocidade: ele encontra exatamente o mesmo caminho ótimo que a Força Bruta, mas em fração ínfima do tempo. Esse resultado é consistente com a prova teórica apresentada na seção 3.2.

---

## 5. Escala de Decisão

A escala abaixo ordena as alternativas considerando **simultaneamente** os quatro critérios exigidos: qualidade da solução, custo computacional, adequação das estruturas de dados e aplicabilidade prática no cenário brasileiro.

| Nível | Solução | Qualidade da solução | Custo computacional | Adequação das estruturas | Aplicabilidade prática |
|---|---|---|---|---|---|
| ★★★★ **Ótimo** | Força Bruta — N ≤ 8 | 100% — ótimo global por exaustão | `O(N!)` — viável só para testes | `set` para ciclos + `list` para backtracking — corretas, porém não escalam | Validação de corretude e testes unitários |
| ★★★☆ **Excelente** | Dijkstra puro | 100% — ótimo provado para pesos ≥ 0 | `O((V+E) log V)` — viável para N = 100k+ | `heapq` + `dict` + `set` — estruturas ideais para o problema | Recomendado para o cenário RS real (478 municípios) |
| ★★☆☆ **Adequado** | Dijkstra + BST para triagem | 100% na rota + priorização por criticidade | Overhead da BST: `O(log N)` por consulta — mínimo | Adiciona BST à solução anterior: estrutura correta para triagem por chave contínua | Ideal quando há restrição de orçamento — garante que equipes atinjam primeiro os municípios de maior risco |
| ★☆☆☆ **Inviável** | Força Bruta — N > 12 | Ótimo em teoria | `O(N!)` — horas para N = 20; anos para N = 50 | Estruturas corretas, mas a lógica de enumeração não escala independentemente das estruturas | Completamente inaplicável para resposta a desastres em tempo real |

**Recomendação fundamentada:** para o cenário real de enchentes no RS com 478 municípios e necessidade de resposta em minutos, a solução **Dijkstra + BST para triagem (★★☆☆)** é a mais adequada. O Dijkstra garante o caminho de menor custo; a BST adiciona a capacidade de priorizar, em `O(log N)`, os municípios de maior risco ambiental antes mesmo de iniciar o roteamento — funcionalidade diretamente alinhada com a realidade operacional de defesa civil, onde recursos são escassos e a ordem de atendimento importa tanto quanto a rota.

**Sobre o gap de otimalidade:** para todos os tamanhos testados com N ≤ 12, o gap entre Dijkstra e Força Bruta foi de **0%**. Isso ocorre porque o Dijkstra é um algoritmo Guloso **exato** — sua decisão local (processar o vértice mais próximo) é globalmente ótima para pesos não-negativos. Isso o diferencia de outros algoritmos gulosos aproximados (como greedy para cobertura de conjuntos ou para o problema da mochila fracionária), que podem apresentar gap positivo em relação ao ótimo e precisariam ser avaliados com mais cautela na escala de decisão.

---

## 6. Figuras Obrigatórias — interpretações

Todas as figuras são geradas em `src/visualizations.py` com título, legenda e fonte dos dados. As interpretações abaixo acompanham cada figura no relatório e no notebook.

**Fig. 1 — `fig1_grafo_rs.png`**  
Grafo de municípios do RS com nós coloridos por índice de risco (escala YlOrRd: amarelo = baixo risco, vermelho = alto risco) e rota ótima Porto Alegre → Muçum destacada em vermelho. Os nós mais escuros (Muçum, Encantado, Roca Sales) concentram-se na extremidade do Vale do Taquari, o que é geograficamente consistente com os dados reais das enchentes de 2024. A rota em vermelho percorre 4,3 km passando por Santa Cruz do Sul, Lajeado e Encantado — demonstrando que o Dijkstra identifica corretamente o caminho que evita o desvio pela rota direta Porto Alegre–Pelotas, mais longa. A estrutura visual confirma que a modelagem em grafo captura adequadamente a topologia da malha viária regional.

**Fig. 2 — `fig2_bst.png`**  
Representação hierárquica da BST construída com os 10 municípios do cenário RS, usando o índice de risco como chave de ordenação. A raiz da árvore corresponde ao primeiro município inserido; nós à esquerda têm risco menor que o pai e nós à direita têm risco maior, respeitando a propriedade BST. A altura observada é 5, enquanto o ideal para 10 nós seria `⌊log₂ 10⌋ = 3`— indicando que a inserção em ordem não-aleatória produziu uma árvore levemente desbalanceada. Ainda assim, a busca por intervalo `[0.80, 1.00]` retorna os 4 municípios críticos sem percorrer todos os nós, evidenciando a eficiência da estrutura em relação a uma lista linear.

**Fig. 3 — `fig3_tempo.png`**  
Gráfico comparativo de tempo de execução (ms) × N para Força Bruta e Dijkstra, com linha vertical tracejada em N = 12 marcando o limite prático de viabilidade da FB. As duas curvas se cruzam em torno de N = 8–10, ponto a partir do qual o crescimento fatorial da Força Bruta supera definitivamente o crescimento `O((V+E) log V)` do Dijkstra. O significado prático desse cruzamento é que, para qualquer instância com mais de 10 municípios — e o cenário real tem 478 —, o algoritmo Guloso é a única opção computacionalmente viável para tomada de decisão em tempo real. A curva do Dijkstra permanece essencialmente plana na escala do gráfico mesmo para N = 100.

**Fig. 4 — `fig4_tabela_estruturas.png`**  
Tabela visual das estruturas de dados utilizadas no projeto, com a estrutura, seu uso concreto no sistema e a justificativa de complexidade. Cada linha representa uma estrutura diferente, coloridas alternadamente para facilitar a leitura. A tabela evidencia que as escolhas não foram arbitrárias: cada estrutura foi selecionada pela operação dominante que precisa suportar — `heapq` pelo `O(log V)` de extração mínima no Dijkstra, `set` pelo `O(1)` de pertencimento no controle de ciclos, `dict` pelo `O(1)` de acesso às distâncias acumuladas.

**Fig. 5 — `fig5_gap.png`**  
Gráfico de barras do gap de otimalidade (%) entre Dijkstra e Força Bruta para N = 5, 8, 10, 12. Todas as barras são 0%, confirmando empiricamente que o Dijkstra é um algoritmo Guloso exato para pesos não-negativos: ele encontra o mesmo custo ótimo que a enumeração completa em todos os casos testados. Esse resultado é de alta relevância prática — significa que, ao escalar para o cenário real com 478 municípios, a solução Gulosa não introduz nenhuma perda de qualidade em relação ao ótimo teórico. O gráfico também serve como evidência de corretude da implementação: se houvesse bug no Dijkstra, os gaps seriam positivos.

**Fig. 6 — `fig6_explosao_combinatoria.png`**  
Gráfico de linhas do número de chamadas recursivas e caminhos completos avaliados pela Força Bruta em função de N. Ambas as curvas crescem de forma claramente superlinear, evidenciando a explosão combinatória característica de algoritmos de enumeração exaustiva. O número de chamadas recursivas cresce mais rápido que o de caminhos completos porque inclui todos os nós intermediários visitados durante o backtracking — a diferença entre as duas curvas representa o "desperdício" inerente à exploração de ramos que não levam ao destino. A partir de N = 12, as curvas apontam para valores que tornam a execução impraticável mesmo com hardware moderno.

---

## 7. Estrutura do Repositório e Módulos

```
global-solution-2026-fund/
│
├── README.md                         ← Este arquivo
├── requirements.txt                  ← Dependências Python
│
├── data/
│   ├── raw/                          ← Dados brutos: NDVI, pluviometria, malha viária
│   └── processed/                    ← Grafos e árvores serializados (JSON/pickle)
│
├── src/
│   ├── data_structures.py            ← Grafo, Node, BinarySearchTree + dados cenário RS
│   ├── brute_force.py                ← Força Bruta com backtracking e contadores
│   ├── greedy.py                     ← Dijkstra + reconstrução de caminho
│   ├── performance_monitor.py        ← monitorar() — tempo, memória, operações
│   └── visualizations.py             ← Geração das 6 figuras obrigatórias
│
├── notebooks/
│   └── analise_resultados.ipynb      ← Análise interativa e escala de decisão
│
├── tests/
│   └── test_algorithms.py            ← Testes unitários com pytest
│
└── report/
    └── relatorio_final.pdf           ← Relatório técnico (máx. 12 páginas)
```

**`src/data_structures.py`** — classes `Grafo`, `Node` e `BinarySearchTree` implementadas do zero. Inclui os dados sintéticos do cenário RS (10 municípios, 11 rotas).

**`src/brute_force.py`** — `forca_bruta()` com backtracking recursivo, instrumentada com dois contadores (chamadas recursivas e caminhos completos avaliados). Uso restrito a `N ≤ 12`.

**`src/greedy.py`** — `dijkstra()` com `heapq` e `reconstruir_caminho()` com dicionário de predecessores. Integrado à BST para triagem de municípios de alto risco antes do roteamento.

**`src/performance_monitor.py`** — `monitorar()` envolve qualquer função e retorna tempo (ms), memória pico (MB) e operações elementares. Executa para `N ∈ {5, 8, 10, 12, 20, 50, 100}` e produz os dados das figuras 3, 5 e 6.

**`src/visualizations.py`** — gera as 6 figuras obrigatórias com `networkx` e `matplotlib`. Não substitui as implementações dos algoritmos — usa os resultados já calculados pelos módulos anteriores.

---

## 8. Instalação e Execução

### Pré-requisitos

- Python 3.10+
- pip

### Instalação

```bash
git clone https://github.com/SEU_USUARIO/global-solution-2026-fund.git
cd global-solution-2026-fund
pip install -r requirements.txt
```

### Executando os módulos

```bash
# 1. Estruturas de dados (Grafo + BST) e cenário RS
python src/data_structures.py

# 2. Força Bruta — instâncias N ≤ 12
python src/brute_force.py

# 3. Dijkstra — solução Greedy
python src/greedy.py

# 4. Monitor de desempenho — todos os tamanhos de N
python src/performance_monitor.py

# 5. Gerar as 6 figuras obrigatórias (salvas como PNG)
python src/visualizations.py

# 6. Notebook de análise interativa
jupyter notebook notebooks/analise_resultados.ipynb
```

### Dependências (`requirements.txt`)

```
networkx>=3.0
matplotlib>=3.7
seaborn>=0.12
pytest>=7.0
jupyter>=1.0
geopandas>=0.13   # opcional — manipulação geoespacial
shapely>=2.0      # opcional
```

> `heapq` e `tracemalloc` são módulos nativos do Python — não requerem instalação.

---

## 9. Testes Automatizados

```bash
pytest tests/test_algorithms.py -v
```

Os testes cobrem:

| Teste | O que valida |
|---|---|
| `test_bst_inserir_e_in_order` | In-order retorna municípios em ordem crescente de risco |
| `test_bst_buscar_intervalo` | `buscar(0.80, 1.00)` retorna exatamente os municípios esperados |
| `test_bst_altura` | Altura calculada é ≥ `⌊log₂ N⌋` e ≤ N |
| `test_bst_remover` | Remoção mantém a propriedade BST |
| `test_forca_bruta_otimo` | FB retorna o custo mínimo para instâncias conhecidas |
| `test_dijkstra_igual_fb` | Dijkstra coincide com FB (gap = 0%) para N ≤ 12 |
| `test_dijkstra_inacessivel` | Retorna lista vazia para destino sem caminho |
| `test_grafo_bidirecional` | `adicionar_rota` cria aresta nos dois sentidos |

---

## 10. Fontes de Dados e Referências

### Fontes de dados

| Fonte | Dados utilizados | URL |
|---|---|---|
| DNIT | Malha viária federal — pesos das arestas | dnit.gov.br |
| Defesa Civil RS | Índices de risco por município (enchentes 2024) | defesacivil.rs.gov.br |
| NASA Earthdata | MODIS NDVI e SRTM | earthdata.nasa.gov |
| INPE PRODES/DETER | Desmatamento Amazônia | terrabrasilis.dpi.inpe.br |
| ANA | Pluviometria e hidrologia | hidroweb.ana.gov.br |
| INMET | Dados climáticos históricos | bdmep.inmet.gov.br |
| IBGE | Malha municipal e dados socioeconômicos | ibge.gov.br/geociencias |
| ANATEL | Cobertura de internet | mapa.anatel.gov.br |

### Referências bibliográficas

- CORMEN, T. H. et al. *Introduction to Algorithms*, 4ª ed. MIT Press, 2022. Caps. 22–25: Grafos e Algoritmos Gulosos.
- SEDGEWICK, R.; WAYNE, K. *Algorithms*, 4ª ed. Addison-Wesley, 2011. Parte 4: Grafos.
- SKIENA, S. *The Algorithm Design Manual*, 3ª ed. Springer, 2020.
- Carta Internacional Space and Major Disasters. Disponível em: disasterscharter.org.

---

## 11. Conexão com os ODS da ONU

| ODS | Conexão direta com o projeto |
|---|---|
| **ODS 2** — Fome Zero e Agricultura Sustentável | O sistema de triagem por risco agrícola (Cenário B — MATOPIBA) auxilia cooperativas a identificar municípios em risco de seca e planejar distribuição de recursos antes de perdas de safra |
| **ODS 9** — Indústria, Inovação e Infraestrutura | O uso de dados de satélites (Sentinel, GOES-16) integrados a algoritmos eficientes de grafos representa exatamente a inovação tecnológica em infraestrutura de resposta a desastres que o ODS 9 propõe |
| **ODS 11** — Cidades e Comunidades Sustentáveis | A triagem automatizada de municípios em risco e o roteamento eficiente de equipes de defesa civil contribuem diretamente para respostas mais rápidas e planejamento urbano mais resiliente |
| **ODS 13** — Ação Climática | O sistema é construído sobre dados climáticos reais (NDVI, precipitação, temperatura) e fornece subsídio computacional para decisões de adaptação climática em escala municipal — nível onde as intervenções têm maior impacto imediato |

---

*FIAP — Global Solution | 1º Semestre de 2026 | Dynamic Programming — Prof. André Marques*
