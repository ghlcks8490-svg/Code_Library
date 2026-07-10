# Recursive Backtracking & BFS Maze System

## Overview

Flashlight 프로젝트의 미로 시스템입니다.

재귀 백트래킹(Recursive Backtracking) 알고리즘을 사용하여 매번 다른 형태의 미로를 생성하고, BFS(Breadth-First Search)를 사용하여 입구부터 출구까지의 최단 경로를 탐색했습니다. 탐색된 경로를 기반으로 발자국을 배치하여 플레이어가 자연스럽게 진행 방향을 파악할 수 있도록 구현했습니다.

---

# 1. Recursive Backtracking을 이용한 미로 생성

## 동작 과정

1. 모든 칸을 벽(1)으로 초기화합니다.
2. 시작 위치를 길(0)로 변경합니다.
3. 상하좌우 방향을 무작위로 섞습니다.
4. 두 칸 떨어진 미방문 칸을 선택합니다.
5. 현재 칸과 다음 칸 사이의 벽을 제거합니다.
6. 선택한 칸에서 `Carve()`를 재귀 호출합니다.
7. 더 이상 이동할 수 없으면 이전 호출 위치로 돌아가 남은 방향을 탐색합니다.
8. 모든 칸을 방문하면 미로 생성이 완료됩니다.

## 핵심 코드

```csharp
private void Carve(int x, int y)
{
    _maze[x, y] = 0;

    List<Vector2Int> dirs = new List<Vector2Int>(_dirs);
    Shuffle(dirs);

    foreach (Vector2Int dir in dirs)
    {
        int nx = x + dir.x;
        int ny = y + dir.y;

        if (Inside(nx, ny) && _maze[nx, ny] == 1)
        {
            int wallX = x + dir.x / 2;
            int wallY = y + dir.y / 2;

            _maze[wallX, wallY] = 0;
            _maze[nx, ny] = 0;

            Carve(nx, ny);
        }
    }
}
```

## 알고리즘 흐름

```text
모든 칸을 벽으로 초기화
        ↓
시작 칸을 길로 변경
        ↓
방향을 무작위로 선택
        ↓
미방문 칸 발견
        ↓
벽 제거
        ↓
Carve() 재귀 호출
        ↓
막다른 길이면 이전 호출로 복귀(Backtracking)
        ↓
모든 칸 방문 시 종료
```

---

# 2. BFS를 이용한 최단 경로 탐색

미로 생성이 완료된 후 입구에서 출구까지의 최단 경로를 탐색합니다.

Queue를 이용하여 시작점에서 가까운 칸부터 순차적으로 탐색하며, 각 칸의 이전 위치를 `parent` 배열에 저장합니다. 출구를 찾으면 `parent`를 역추적하여 최종 경로를 복원합니다.

## 동작 과정

1. 시작점을 Queue에 추가합니다.
2. Queue에서 현재 위치를 꺼냅니다.
3. 상하좌우의 이동 가능한 칸을 탐색합니다.
4. 방문하지 않은 길이면 Queue에 추가합니다.
5. 이전 위치를 `parent` 배열에 저장합니다.
6. 출구를 찾으면 탐색을 종료합니다.
7. `parent`를 따라 출구부터 시작점까지 역추적합니다.
8. 경로를 뒤집어 시작점 → 출구 순으로 정렬합니다.

## 핵심 코드

```csharp
queue.Enqueue(start);
visited[start.x, start.y] = true;

while (queue.Count > 0)
{
    Vector2Int current = queue.Dequeue();

    if (current == exit)
        break;

    foreach (Vector2Int dir in dirs)
    {
        Vector2Int next = current + dir;

        if (!IsInsidePath(next.x, next.y)) continue;
        if (visited[next.x, next.y]) continue;
        if (_maze[next.x, next.y] != 0) continue;

        visited[next.x, next.y] = true;
        parent[next.x, next.y] = current;
        queue.Enqueue(next);
    }
}
```

## 경로 복원

```text
Start → A → B → C → Exit
```

`parent` 배열에는 다음과 같이 저장됩니다.

```text
parent[A] = Start
parent[B] = A
parent[C] = B
parent[Exit] = C
```

출구부터 `parent`를 따라 역추적한 뒤 `Reverse()`를 호출하여 시작점부터 출구까지의 최종 경로를 생성합니다.

---

# 결과

* Recursive Backtracking을 활용해 매번 다른 형태의 미로를 생성했습니다.
* BFS를 이용해 입구부터 출구까지의 최단 경로를 탐색했습니다.
* 탐색한 경로를 기반으로 발자국을 배치하여 플레이어가 자연스럽게 진행 방향을 파악할 수 있도록 구현했습니다.

---

