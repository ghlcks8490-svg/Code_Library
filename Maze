using System.Collections.Generic;
using UnityEngine;

public class MazeGenerator : MonoBehaviour
{
    [SerializeField] RespawnController _respawnCtrl;

    [Header("Maze Set")]
    [SerializeField] private int _width = 21;
    [SerializeField] private int _height = 21;
    [SerializeField] private Transform _startPosition;
    [SerializeField] private Transform _mazeParent;

    [Header("Prefabs")]
    [SerializeField] private GameObject _floorPrefab;
    [SerializeField] private GameObject _wallPrefab;

    [Header("Foot steps")]
    [SerializeField] private GameObject _pathPrefab;

    private int[,] _maze;

    private Vector2Int _startCell = new Vector2Int(1, 0);
    private Vector2Int _exitCell;
    private Vector2Int _lastCell;
    private Vector3 _exitCellPosition;

    public Vector3 ExitCellPos { get => _exitCellPosition; }

    private readonly Vector2Int[] _dirs =
    {
        new Vector2Int(0, 2),
        new Vector2Int(0, -2),
        new Vector2Int(2, 0),
        new Vector2Int(-2, 0),
    };

    private void Awake()
    {
        GenerateMaze();
    }

    void Start()
    {
        List<Vector2Int> answerPath = FindPath(_startCell, _exitCell);

        if (answerPath.Count < 2)
        {
            Debug.Log("경로를 찾지 못함");
            BuildMaze();
            return;
        }

        _lastCell = answerPath[answerPath.Count - 2];
        _exitCellPosition = new Vector3(_exitCell.x, 1f, _exitCell.y) + _startPosition.position;

        BuildMaze();
        SpawnFootprintsOnPath(answerPath);
    }

    private void Update()
    {
        if (_respawnCtrl.IsDead)
        {
            CreatNewMaze();
            _respawnCtrl.ResetDead();
        }
    }

    private void GenerateMaze()
    {
        if (_width % 2 == 0) _width++;
        if (_height % 2 == 0) _height++;

        _maze = new int[_width, _height];

        for (int x = 0; x < _width; x++)
        {
            for (int y = 0; y < _height; y++)
            {
                _maze[x, y] = 1;
            }
        }

        Carve(1, 1);
        CreatEnteranceAndExit();


    }

    private void CreatEnteranceAndExit()
    {
        int centerX = (_width - 1) / 2;
        if (centerX % 2 == 0) centerX -= 1;

        _startCell = new Vector2Int(centerX, 0);
        _exitCell = new Vector2Int(centerX, _height - 1);

        _maze[_startCell.x, _startCell.y] = 0;
        _maze[_exitCell.x, _exitCell.y] = 0;

        _maze[centerX, 1] = 0;
        _maze[centerX, _height - 2] = 0;
    }

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

                Carve(nx, ny); // 재귀 백트랙킹
            }
        }
    }

    private void Shuffle(List<Vector2Int> list)
    {
        for (int i = 0; i < list.Count; i++)
        {
            int r = Random.Range(i, list.Count);
            (list[i], list[r]) = (list[r], list[i]);
        }
    }

    private bool Inside(int x, int y)
    {
        return x > 0 && y > 0 && x < _width - 1 && y < _height - 1;
    }

    private void BuildMaze()
    {
        for (int x = 0; x < _width; x++)
        {
            for (int y = 0; y < _height; y++)
            {
                Vector3 wallPos = new Vector3(x, 2, y) + _startPosition.position;
                Vector3 floorPos = new Vector3(x, 0, y) + _startPosition.position;

                if (_maze[x, y] == 1)
                {
                    Instantiate(_wallPrefab, wallPos, Quaternion.identity, _mazeParent);
                }
                else
                {
                    Instantiate(_floorPrefab, floorPos, Quaternion.identity, _mazeParent);
                }
            }
        }
    }

    private List<Vector2Int> FindPath(Vector2Int start, Vector2Int exit)
    {
        Queue<Vector2Int> queue = new Queue<Vector2Int>();
        bool[,] visited = new bool[_width, _height];
        Vector2Int?[,] parent = new Vector2Int?[_width, _height];

        queue.Enqueue(start);
        visited[start.x, start.y] = true;

        Vector2Int[] dirs =
        {
            new Vector2Int(0,1),
            new Vector2Int(0, -1),
            new Vector2Int(-1,0),
            new Vector2Int(1, 0)
        };

        while (queue.Count > 0)
        {
            Vector2Int current = queue.Dequeue();

            if (current == exit) break;

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

        List<Vector2Int> path = new List<Vector2Int>();

        if (!visited[exit.x, exit.y]) return path;

        Vector2Int? step = exit;
        while (step != null)
        {
            path.Add(step.Value);
            step = parent[step.Value.x, step.Value.y];
        }

        path.Reverse();
        return path;
    }

    private bool IsInsidePath(int x, int y)
    {
        return x >= 0 && y >= 0 && x < _width && y < _height;
    }

    private void SpawnFootprintsOnPath(List<Vector2Int> path)
    {
        for (int i = 0; i < path.Count - 1; i += 1)
        {
            Vector2Int current = path[i];
            Vector2Int next = path[i + 1];

            Vector3 pos = new Vector3(current.x, 0, current.y) + _startPosition.position;
            Vector2Int dir = next - current;

            Quaternion rot = GetFootprintRotation(dir);

            GameObject pathFloor = Instantiate(_pathPrefab, pos, rot, _mazeParent);

            MazeFloorController floorCtrl = pathFloor.GetComponent<MazeFloorController>();

            if (floorCtrl != null && current == _lastCell)
            {
                floorCtrl.IsLastFloor = true;
                floorCtrl.KeyNumPos = _exitCellPosition;
            }
        }
    }

    private Quaternion GetFootprintRotation(Vector2Int dir)
    {
        if (dir == new Vector2Int(0, 1)) return Quaternion.Euler(0, 0, 0);
        if (dir == new Vector2Int(1, 0)) return Quaternion.Euler(0, 90, 0);
        if (dir == new Vector2Int(0, -1)) return Quaternion.Euler(0, 180, 0);
        if (dir == new Vector2Int(-1, 0)) return Quaternion.Euler(0, 270, 0);

        return Quaternion.identity;
    }

    private void CreatNewMaze()
    {
        ClearMaze();

        GenerateMaze();

        List<Vector2Int> answerPath = FindPath(_startCell, _exitCell);

        if (answerPath.Count < 2)
        {
            Debug.Log("경로를 찾지 못함");
            BuildMaze();
            return;
        }

        _lastCell = answerPath[answerPath.Count - 2];
        _exitCellPosition = new Vector3(_exitCell.x, 1f, _exitCell.y) + _startPosition.position;

        BuildMaze();
        SpawnFootprintsOnPath(answerPath);
    }

    private void ClearMaze()
    {
        for (int i = _mazeParent.childCount - 1; i >= 0; i--)
        {
            Destroy(_mazeParent.GetChild(i).gameObject);
        }
    }
}
