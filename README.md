# Recursive Backtracking & BFS Maze System

## 📖 Overview

Flashlight 프로젝트에서 사용한 미로 생성 및 경로 탐색 시스템입니다.

미로는 **Recursive Backtracking** 알고리즘을 사용하여 매번 다른 형태로 생성되며, 생성이 완료된 후 **BFS(Breadth-First Search)**를 이용해 입구부터 출구까지의 최단 경로를 탐색합니다. 탐색된 경로를 기반으로 발자국 오브젝트를 배치하여 플레이어가 자연스럽게 진행 방향을 파악할 수 있도록 구현했습니다.

---

## 🛠 Tech Stack

* Unity
* C#
* Recursive Backtracking
* Breadth-First Search (BFS)
* Queue
* 2D Array
* Recursion

---

# 🌲 Maze Generation (Recursive Backtracking)

## Algorithm

1. 모든 칸을 벽(1)으로 초기화합니다.
2. 시작 위치를 길(0)로 변경합니다.
3. 이동 방향을 무작위로 섞습니다.
4. 두 칸 떨어진 미방문 칸을 선택합니다.
5. 현재 칸과 다음 칸 사이의 벽을 제거합니다.
6. 다음 위치에서 `Carve()`를 재귀 호출합니다.
7. 더 이상 이동할 수 없으면 이전 호출 위치로 돌아가 남은 방향을 탐색합니다.
8. 모든 칸을 방문하면 미로 생성이 완료됩니다.

### Flow

```text
Initialize Maze
        │
        ▼
Mark Current Cell
        │
        ▼
Shuffle Directions
        │
        ▼
Find Unvisited Cell
        │
        ▼
Remove Wall
        │
        ▼
Recursive Call
        │
        ▼
No More Paths?
        │
        ▼
Backtrack
```

### Core Code

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

---

# 🚪 Path Finding (BFS)

미로 생성 후 BFS를 사용하여 입구부터 출구까지의 최단 경로를 탐색했습니다.

Queue를 이용해 가까운 칸부터 순차적으로 탐색하고, 각 칸의 이전 위치를 `parent` 배열에 저장합니다. 출구를 찾으면 `parent`를 역추적하여 최종 경로를 복원합니다.

### Algorithm

1. 시작 좌표를 Queue에 추가합니다.
2. Queue에서 현재 좌표를 하나 꺼냅니다.
3. 상하좌우의 이동 가능한 칸을 확인합니다.
4. 방문하지 않은 길이면 Queue에 추가합니다.
5. 이전 좌표를 `parent` 배열에 저장합니다.
6. 출구를 찾으면 탐색을 종료합니다.
7. `parent`를 역추적하여 경로를 복원합니다.
8. `Reverse()`를 통해 시작점 → 출구 순으로 정렬합니다.

### Flow

```text
Start
  │
  ▼
Enqueue Start
  │
  ▼
Dequeue Current
  │
  ▼
Explore Neighbors
  │
  ▼
Mark Visited
  │
  ▼
Store Parent
  │
  ▼
Enqueue Next
  │
  ▼
Exit Found?
  │
  ▼
Reconstruct Path
```

### Core Code

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

---

# 👣 Result

* Recursive Backtracking을 이용해 매번 다른 형태의 미로를 생성했습니다.
* BFS를 이용해 입구부터 출구까지의 최단 경로를 탐색했습니다.
* 탐색한 경로를 기반으로 발자국을 배치하여 플레이어가 진행 방향을 자연스럽게 인지할 수 있도록 구현했습니다.

---

# 📷 Preview

> 미로 생성 과정 또는 결과 GIF를 추가하면 프로젝트를 이해하기 쉽습니다.

```md
![Maze Demo](Images/Maze.gif)
```
