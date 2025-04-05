---
title: Graph Search Algorithm
date: 2025-03-20 12:33:25 +09:00
categories: ['algorithm']
tags: ['graph', 'BFS', 'DFS', 'Dijkstra']
mermaid: true
---

그래프는 노드(정점)와 간선으로 구성된 자료구조로, 다양한 실세계 상황을 모델링하는 데 사용됩니다. 
소셜 네트워크, 도로 체계, 컴퓨터 네트워크 등 많은 시스템이 그래프로 표현될 수 있습니다. 
그래프 탐색 알고리즘은 이러한 구조를 체계적으로 순회하고 분석하는 방법을 제공합니다.


그래프 표현 방식은 크게 인접 행렬(Adjacency Matrix)과 인접 리스트(Adjacency List)로 나눌 수 있습니다. 
인접 행렬은 모든 노드 쌍의 연결 상태를 2차원 배열로 표현하여 연결 여부를 O(1) 시간에 확인할 수 있으나, 희소 그래프에서는 공간 낭비가 발생합니다. 
반면 인접 리스트는 각 노드에 연결된 노드들만 리스트로 저장하여 메모리를 효율적으로 사용하지만, 특정 두 노드의 연결 여부를 확인하는 데 O(V) 시간이 필요합니다.


그래프 탐색의 주요 알고리즘으로는 **깊이 우선 탐색(DFS)과 너비 우선 탐색(BFS)**이 있습니다. 

1. DFS는 가능한 깊이 탐색한 후 백트래킹(backtracking)하는 방식으로, 미로 탐색이나 퍼즐 해결과 같은 문제에 적합합니다. 
2. BFS는 시작점에서 가까운 노드부터 단계적으로 탐색하여 최단 경로 문제에 유용합니다. 
3. 가중치가 있는 그래프에서는 다익스트라(Dijkstra) 알고리즘이 특정 노드로부터 다른 모든 노드까지의 최단 경로를 찾는 데 사용됩니다.

> 본 글에서는 다양한 그래프 표현 방식과 탐색 알고리즘의 구현 방법을 자세히 살펴보고, 각 알고리즘의 특성과 적용 사례를 분석합니다.

## 1. 깊이 우선 탐색 (DFS)

깊이 우선 탐색은 그래프의 깊은 부분을 우선적으로 탐색하는 알고리즘입니다. 한 경로를 끝까지 탐색한 후 다른 경로를 탐색하는 방식으로 동작합니다.

### 1.1 스택을 이용한 DFS - 인접 행렬

인접 행렬은 그래프의 연결 상태를 2차원 배열로 표현한 것입니다. 행렬의 각 요소 `graph[i][j]`가 1이면 노드 `i`에서 노드 `j`로 가는 간선이 있음을 의미합니다.

```python
def dfs_matrix(graph, start):
    visited = [False] * len(graph)
    stack = [start]

    while stack:
        node = stack.pop()
        if not visited[node]:
            visited[node] = True
            print(node, end=" ")
            # 인접한 노드를 스택에 추가
            for i in range(len(graph)):
                if graph[node][i] == 1 and not visited[i]:
                    stack.append(i)
```

이 구현에서는 스택을 사용하여 방문할 노드를 관리합니다. 현재 노드에서 인접한 모든 노드를 확인하고, 방문하지 않은 노드를 스택에 추가합니다. 순환이 있는 그래프에서는 `visited` 배열을 사용하여 이미 방문한 노드를 다시 방문하지 않도록 합니다.

### 1.2 스택을 이용한 DFS - 인접 리스트

인접 리스트는 각 노드에 연결된 노드들의 목록을 저장하는 방식입니다. 이 방식은 그래프의 간선이 상대적으로 적은 경우(희소 그래프) 메모리 사용 측면에서 효율적입니다.

```python
def dfs_list(graph, start):
    visited = [False] * len(graph)
    stack = [start]

    while stack:
        node = stack.pop()
        if not visited[node]:
            visited[node] = True
            print(node, end=" ")
            # 인접한 노드를 스택에 추가
            for neighbor in graph[node]:
                if not visited[neighbor]:
                    stack.append(neighbor)
```

인접 리스트를 사용할 경우, 현재 노드에서 연결된 노드들을 직접 접근할 수 있어 탐색 시간이 단축됩니다.

### 1.3 재귀를 이용한 DFS - 인접 행렬

재귀 함수를 사용하여 DFS를 구현할 수도 있습니다. 이 방식은 코드가 더 간결하고 직관적이지만, 깊은 그래프에서는 스택 오버플로우의 위험이 있습니다.

```python
def dfs_matrix(graph, start, visited):
    visited[start] = True
    print(start, end=" ")

    for neighbor in range(len(graph)):
        if graph[start][neighbor] == 1 and not visited[neighbor]:
            dfs_matrix(graph, neighbor, visited)
```

### 1.4 재귀를 이용한 DFS - 인접 리스트

재귀와 인접 리스트를 결합한 DFS 구현은 다음과 같습니다:

```python
def dfs_list(graph, start, visited):
    visited[start] = True
    print(start, end=" ")

    for neighbor in graph[start]:
        if not visited[neighbor]:
            dfs_list(graph, neighbor, visited)
```

인접 리스트를 사용한 재귀 구현은 매우 간결하며, 각 노드와 연결된 노드들을 직접 순회할 수 있습니다.

## 2. 너비 우선 탐색 (BFS)

너비 우선 탐색은 시작 노드에서 가까운 노드부터 차례대로 탐색하는 알고리즘입니다. 같은 레벨의 노드를 모두 탐색한 후에 다음 레벨로 넘어가는 방식으로 동작합니다.

### 2.1 BFS - 인접 행렬

BFS는 큐(Queue)를 사용하여 구현됩니다. 파이썬에서는 효율적인 큐 구현을 위해 `collections` 모듈의 `deque`를 사용합니다.

```python
from collections import deque

def bfs_matrix(graph, start):
    visited = [False] * len(graph)
    queue = deque([start])
    visited[start] = True

    while queue:
        node = queue.popleft()
        print(node, end=" ")
        # 인접한 노드를 큐에 추가
        for i in range(len(graph)):
            if graph[node][i] == 1 and not visited[i]:
                queue.append(i)
                visited[i] = True
```

BFS에서는 노드를 방문할 때 즉시 `visited` 배열을 업데이트하여 같은 노드가 큐에 여러 번 추가되는 것을 방지합니다.

### 2.2 BFS - 인접 리스트

인접 리스트를 사용한 BFS 구현은 다음과 같습니다:

```python
from collections import deque

def bfs_list(graph, start):
    visited = [False] * len(graph)
    queue = deque([start])
    visited[start] = True

    while queue:
        node = queue.popleft()
        print(node, end=" ")
        # 인접한 노드를 큐에 추가
        for neighbor in graph[node]:
            if not visited[neighbor]:
                queue.append(neighbor)
                visited[neighbor] = True
```

인접 리스트를 사용하면 현재 노드와 연결된 노드들만 확인하면 되므로, 인접 행렬보다 효율적인 탐색이 가능합니다.

## 3. 다익스트라 알고리즘 (Dijkstra)

다익스트라 알고리즘은 가중치가 있는 그래프에서 시작 노드로부터 다른 모든 노드까지의 최단 경로를 찾는 알고리즘입니다. 
우선순위 큐를 사용하여 효율적으로 구현할 수 있습니다.

### 3.1 우선순위 큐를 이용한 다익스트라

```python
import heapq

def dijkstra(graph, start):
    # 거리 리스트 (무한대로 초기화)
    dist = [float('inf')] * len(graph)
    dist[start] = 0
    pq = [(0, start)]  # (거리, 노드)

    while pq:
        current_dist, node = heapq.heappop(pq)

        if current_dist > dist[node]:
            continue

        for neighbor, weight in graph[node]:
            distance = current_dist + weight
            if distance < dist[neighbor]:
                dist[neighbor] = distance
                heapq.heappush(pq, (distance, neighbor))

    return dist
```

이 구현에서는 우선순위 큐를 사용하여 현재까지 발견된 최단 경로가 가장 짧은 노드부터 처리합니다. 이를 통해 각 노드를 최대 한 번만 처리하고, 불필요한 탐색을 줄일 수 있습니다.

## 맺음말

그래프 탐색 알고리즘은 다양한 문제 해결에 활용되는 중요한 도구입니다. 
DFS는 경로 찾기, 사이클 감지, 위상 정렬 등에 사용되며, BFS는 최단 경로 찾기, 연결 구성 요소 탐색 등에 적합합니다. 
다익스트라 알고리즘은 가중치가 있는 그래프에서 최단 경로를 찾는데 효율적입니다.

> 인접 행렬과 인접 리스트는 그래프를 표현하는 두 가지 주요 방식이며, 그래프의 특성에 따라 적절한 방식을 선택하는 것이 중요합니다.
> 순환이 있는 그래프를 처리할 때는 반드시 방문 여부를 확인하는 로직을 포함하여 무한 루프를 방지해야 합니다. 

> 본 글이 그래프 탐색 알고리즘을 이해하는 데 도움이 되었기를 바랍니다.
