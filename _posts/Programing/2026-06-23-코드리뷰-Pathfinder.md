---
title: "코드 리뷰-Pathfinder"
categories:
  - Programing
tags:
  - Review
  - Refactoring
---

Pathfinder의 코드를 살펴보자.

<details>
<summary>Before</summary>

```csharp
using System.Collections.Generic;
using System.Linq;
using UnityEngine;

namespace Data.Map
{
    public static class Pathfinder
    {
        private class Node
        {
            public Vector2Int Position;
            public Node Parent;
            public int G;
            public int H;
            public int F => G + H;

            public Node(Vector2Int position)
            {
                Position = position;
            }
        }

        public static List<Vector2Int> FindPath(GridMap gridMap, Vector2Int start, Vector2Int end)
        {
            if (gridMap == null) return null;

            var startCell = gridMap.GetCell(start);
            var endCell = gridMap.GetCell(end);

            // Goal needs to be valid and walkable
            if (!CanMove(startCell, endCell)) return null;

            var openList = new List<Node>();
            var closedSet = new HashSet<Vector2Int>();
            
            var startNode = new Node(start);
            openList.Add(startNode);

            while (openList.Count > 0)
            {
                // 가장 낮은 F 비용을 가진 노드 찾기
                var currentNode = openList.OrderBy(n => n.F).ThenBy(n => n.H).First();
                var currentPos = currentNode.Position;
                var currentCell = gridMap.GetCell(currentPos);
                
                if (currentPos == end)
                {
                    return RetracePath(start, currentNode);
                }

                openList.Remove(currentNode);
                closedSet.Add(currentPos);

                foreach (var neighborPos in GetNeighbors(currentPos))
                {
                    if (closedSet.Contains(neighborPos)) continue;

                    var neighborCell = gridMap.GetCell(neighborPos);

                    if (!CanMove(currentCell, neighborCell)) continue;

                    var newMovementCostToNeighbor = currentNode.G + neighborCell.MoveCost; 
                    var neighborNode = openList.FirstOrDefault(n => n.Position == neighborPos);
                    
                    if (neighborNode == null || newMovementCostToNeighbor < neighborNode.G)
                    {
                        if (neighborNode == null)
                        {
                            neighborNode = new Node(neighborPos);
                            neighborNode.H = GetDistance(neighborPos, end);
                            openList.Add(neighborNode);
                        }
                        
                        neighborNode.G = newMovementCostToNeighbor;
                        neighborNode.Parent = currentNode;
                    }
                }
            }

            return null; // 경로 없음
        }

        public static bool CanMove(Cell fromCell, Cell toCell)
        {
            if (fromCell == null || toCell == null) return false;
            if (fromCell.NavigationID <= 0 || toCell.NavigationID <= 0) return false;
            if (fromCell.NavigationID != toCell.NavigationID) return false;
            return true;
        }

        private static List<Vector2Int> RetracePath(Vector2Int start, Node endNode)
        {
            var path = new List<Vector2Int>();
            var currentNode = endNode;

            while (currentNode.Position != start)
            {
                path.Add(currentNode.Position);
                currentNode = currentNode.Parent;
            }
            path.Reverse();
            return path;
        }

        private static int GetDistance(Vector2Int a, Vector2Int b)
        {
            int dstX = Mathf.Abs(a.x - b.x);
            int dstY = Mathf.Abs(a.y - b.y);
            return 10 * (dstX + dstY);
        }

        private static IEnumerable<Vector2Int> GetNeighbors(Vector2Int pos)
        {
            // 4방향 이동
            yield return new Vector2Int(pos.x + 1, pos.y);
            yield return new Vector2Int(pos.x - 1, pos.y);
            yield return new Vector2Int(pos.x, pos.y + 1);
            yield return new Vector2Int(pos.x, pos.y - 1);
        }

        public static string PathToString(List<Vector2Int> path)
        {
            if (path == null || path.Count == 0) return "No Path";
            return string.Join(" -> ", path.Select(p => $"({p.x},{p.y})"));
        }
    }
}
```

</details>

흔한 A* 알고리즘이지만 읽기에는 부담스럽다. 여기서 문제가 생겼을 때 바로 고칠 수 있는 지, 수정할 수 있는지 생각하면 어렵다.
하나하나 수정해보도록 하겠다.

```csharp
private class Node
{
    public Vector2Int Position;
    public Node Parent;
    public int G;
    public int H;
    public int F => G + H;

    public Node(Vector2Int position)
    {
        Position = position;
    }
}
```

Node 클래스는 언뜻 보면 손댈 곳이 없어보인다. A*알고리즘의 관례를 따르고 있기 때문이다.
알고리즘으로는 이해가 쉽고 빠르지만, 밑의 코드를 보면 알 수 있다.

```csharp
if (neighborNode == null || newMovementCostToNeighbor < neighborNode.G)
```

이 코드만 분리해서 봤을 때, 왜 Cost와 G가 비교되어야 하는지 쉽게 눈에 들어오지 않는다.
물론 A* 알고리즘을 아는 사람은 이해할 수 있다. 
하지만 매번 A*알고리즘의 G를 머리 속에 기억하고 코드를 읽어야 한다는 부분에서 피로는 몰려온다.

```csharp
private class Node
{
    public Vector2Int Position;
    public Node Parent;
    public int Cost;
    public int H;
    public int F => Cost + H;

    public Node(Vector2Int position)
    {
        Position = position;
    }
}
```

```csharp
if (neighborNode != null && newCost >= neighborNode.Cost) return;
```

이렇게 Cost로 바꾸면 새 Cost와 이웃 노드의 코스트를 비교한다 라는 뜻이 명확히 전달된다.
이제 진정한 문제로 넘어가보자.

```csharp
public static List<Vector2Int> FindPath(GridMap gridMap, Vector2Int start, Vector2Int end)
{
    if (gridMap == null) return null;

    var startCell = gridMap.GetCell(start);
    var endCell = gridMap.GetCell(end);

    // Goal needs to be valid and walkable
    if (!CanMove(startCell, endCell)) return null;

    var openList = new List<Node>();
    var closedSet = new HashSet<Vector2Int>();
    
    var startNode = new Node(start);
    openList.Add(startNode);

    while (openList.Count > 0)
    {
        // 가장 낮은 F 비용을 가진 노드 찾기
        var currentNode = openList.OrderBy(n => n.F).ThenBy(n => n.H).First();
        var currentPos = currentNode.Position;
        var currentCell = gridMap.GetCell(currentPos);
        
        if (currentPos == end)
        {
            return RetracePath(start, currentNode);
        }

        openList.Remove(currentNode);
        closedSet.Add(currentPos);

        foreach (var neighborPos in GetNeighbors(currentPos))
        {
            if (closedSet.Contains(neighborPos)) continue;

            var neighborCell = gridMap.GetCell(neighborPos);

            if (!CanMove(currentCell, neighborCell)) continue;

            var newMovementCostToNeighbor = currentNode.G + neighborCell.MoveCost; 
            var neighborNode = openList.FirstOrDefault(n => n.Position == neighborPos);
            
            if (neighborNode == null || newMovementCostToNeighbor < neighborNode.G)
            {
                if (neighborNode == null)
                {
                    neighborNode = new Node(neighborPos);
                    neighborNode.H = GetDistance(neighborPos, end);
                    openList.Add(neighborNode);
                }
                
                neighborNode.G = newMovementCostToNeighbor;
                neighborNode.Parent = currentNode;
            }
        }
    }

    return null; // 경로 없음
}
```

함수 하나가 50줄 이상을 차지하고 있다. 이중 반복문과 중첩된 조건문들도 많아서 아쉽다.
함수는 하나의 목적을 가지고 실행되어야 한다. 
이를 무조건 지키는 건 오히려 해가 되지만, 최소한 말단 함수는 하나의 목적을 가져야 한다. 
하나하나 분리할 수 있는 부분들을 찾아보자.

```csharp
if (gridMap == null) return null;

var startCell = gridMap.GetCell(start);
var endCell = gridMap.GetCell(end);

// Goal needs to be valid and walkable
if (!CanMove(startCell, endCell)) return null;
```

startCell과 endCell은 처음 유효성 검증에서 선언된 뒤, 다시 사용되지 않는다. 
이 부분이 내부 함수로 독립되어도 아무런 문제가 되지 않는다. 

```csharp
private static bool IsValidPath(GridMap gridMap, Vector2Int startPos, Vector2Int endPos)
{
    if (gridMap == null) return false;
    var startCell = gridMap.GetCell(startPos);
    var endCell = gridMap.GetCell(endPos);
    return CanMove(startCell, endCell);
}
```

따라서 이와 같이 함수로 은닉화 시킬 수 있다. 
이 함수는 시작지점과 끝 지점을 보고 지점설정이 올바른지 확인하는 목적만 가지고 있다.


```csharp
// 가장 낮은 F 비용을 가진 노드 찾기
var currentNode = openList.OrderBy(n => n.F).ThenBy(n => n.H).First();
```

이 부분은 긴 람다식을 주석으로 대체해 설명하고 있다. 
소프트웨어 공학에서는 코드가 언제든지 수정될 수 있기에, 주석은 최소화해서 적는게 좋다. 
주석은 IDE에서 리팩토리로 이름을 바꿔도 남아있는 경우가 많고, 업데이트를 꾸준히 하지 않으면 오히려 독이된다.
함수 자체가 문서가 되도록 이름을 지어서 처리하자.

```csharp
private static Node GetBestNode(this List<Node> list) =>
            list.OrderBy(n => n.F).ThenBy(n => n.H).First();
```

```csharp
var currentNode = open.GetBestNode();
```

open의 가장 좋은 노드를 가져온다. 라는 의미전달이 강하게 되고 있다.
가장 좋은 노드가 뭘 의미 하는걸까 궁금하면, 클릭하고 private 함수 한 줄 읽어보면 된다. 


```csharp
foreach (var neighborPos in GetNeighbors(currentPos))
{
    if (closedSet.Contains(neighborPos)) continue;

    var neighborCell = gridMap.GetCell(neighborPos);

    if (!CanMove(currentCell, neighborCell)) continue;

    var newMovementCostToNeighbor = currentNode.G + neighborCell.MoveCost; 
    var neighborNode = openList.FirstOrDefault(n => n.Position == neighborPos);
    
    if (neighborNode == null || newMovementCostToNeighbor < neighborNode.G)
    {
        if (neighborNode == null)
        {
            neighborNode = new Node(neighborPos);
            neighborNode.H = GetDistance(neighborPos, end);
            openList.Add(neighborNode);
        }
        
        neighborNode.G = newMovementCostToNeighbor;
        neighborNode.Parent = currentNode;
    }
}
```

코드를 작성하면서 가장 주의해야 하는 부분은 이런 반복문이다.
반복문이 길수록 머리 속에서 반복문을 시뮬레이션하기 어렵다. 
또한 대부분 반복문은 독립적으로 돌아가야 하기에, 함수로 은닉화 할 수 있다.


```csharp
private static void UpdateNeighbor(List<Node> open, Node currentNode, Cell neighborCell, Vector2Int end)
{
    var newCost = currentNode.Cost + neighborCell.MoveCost;
    var neighborNode = open.FirstOrDefault(n => n.Position == neighborCell.Position);

    if (neighborNode != null && newCost >= neighborNode.Cost) return;
            
    neighborNode ??= CreateNode(open, neighborCell.Position, end);
    neighborNode.Cost = newCost;
    neighborNode.Parent = currentNode;
}

private static Node CreateNode(List<Node> open, Vector2Int pos, Vector2Int end)
{
    var node = new Node(pos) { H = GetDistance(pos, end)};
    open.Add(node);
    return node;
}
```

이와 같은 함수로 깔끔하게 분리할 수 있다. 
UpdateNeighbor는 이웃 셀을 읽고 새 코스트와 기존 노드를 비교한다. 없으면 생성한다.
이런 식으로 생각하면 하나의 목적을 가진 함수는 아니지만, 이웃을 갱신한다 라는 하나의 목적을 가지고 있다.

코드 내부에서는 중간에 early-return을 통해서 코드블럭을 치웠다.

또한 이웃 노드를 만드는 코드를 CreateNode 함수로 대체하였다.
CreateNode는 노드를 만드는 목적을 가진 함수로 작동한다.

이를 통해 ?? 표현식을 사용해 코드를 더욱 깔끔하게 유지할 수 있다. 


```csharp
...
openList.Remove(currentNode);
closedSet.Add(currentPos);

foreach (var neighborPos in GetNeighbors(currentPos))
{
    if (closedSet.Contains(neighborPos)) continue;
    ...
}
...
```

그 외에 추가로 나는 문장으로 자연스럽게 읽히는 것을 좋아해서 위의 표현식을 싫어한다.
노드를 방문했을 때 행동과 방문을 체크하는 부분이라는 것을 알기 위해 머리 속에서 해석을 두 단계로 돌리기 때문이다.

```csharp

private static void VisitNode(List<Node> open, HashSet<Vector2Int> closed, Node node)
{
    open.Remove(node);
    closed.Add(node.Position);
}

private static bool IsVisited(HashSet<Vector2Int> closed, Vector2Int pos) =>
    closed.Contains(pos);
```

이렇게 VisitNode와 IsVisited로 바꿔주었다. 
하지만 이건 어느정도 선택에 달린 문제다. 특히 Contains를 한번에 이해못하는 사람은 거의 없다.

하지만 VsitNode를 작성했으니, IsVisited를 넣는게 자연스럽다.


```csharp
public static List<Vector2Int> FindPath(GridMap gridMap, Vector2Int startPos, Vector2Int endPos)
{
    if (!IsValidPath(gridMap, startPos, endPos)) return null;

    var open = new List<Node> {new Node(startPos)};
    var closed = new HashSet<Vector2Int>();

    while (open.Count > 0)
    {
        var currentNode = open.GetBestNode();
        var currentPos = currentNode.Position;
        var currentCell = gridMap.GetCell(currentPos);
        
        if (currentPos == endPos) return RetracePath(startPos, currentNode);

        VisitNode(open, closed, currentNode);

        foreach (var neighborPos in GetNeighbors(currentPos))
        {
            if (IsVisited(closed, neighborPos)) continue;

            var neighborCell = gridMap.GetCell(neighborPos);
            
            if (!CanMove(currentCell, neighborCell)) continue;

            UpdateNeighbor(open, currentNode, neighborCell, endPos);
        }
    }

    return null;
}
```

최종적으로 아름다운 함수가 탄생하였다. 타당한 경로인지 먼저 확인하고, A* 알고리즘이 돌아간다.

foreach문은 간단하게 읽힌다. 방문 여부를 확인하고, 그 셀을 갈 수 있는지 여부를 확인한 뒤, 이웃을 갱신한다!
아까 50줄에서 이를 이해하는데 걸리는 속도와 비교해보면 차원이 다르다. 

그렇게 완성된 최종 코드는 밑과 같다. 

<details>
<summary>After</summary>

```csharp
using System.Collections.Generic;
using System.Linq;
using UnityEngine;

namespace Data.Map
{
    public static class Pathfinder
    {
        private class Node
        {
            public Vector2Int Position;
            public Node Parent;
            public int Cost;
            public int H;
            public int F => Cost + H;

            public Node(Vector2Int position)
            {
                Position = position;
            }
        }

        public static List<Vector2Int> FindPath(GridMap gridMap, Vector2Int startPos, Vector2Int endPos)
        {
            if (!IsValidPath(gridMap, startPos, endPos)) return null;

            var open = new List<Node> {new Node(startPos)};
            var closed = new HashSet<Vector2Int>();

            while (open.Count > 0)
            {
                var currentNode = open.GetBestNode();
                var currentPos = currentNode.Position;
                var currentCell = gridMap.GetCell(currentPos);
                
                if (currentPos == endPos) return RetracePath(startPos, currentNode);

                VisitNode(open, closed, currentNode);

                foreach (var neighborPos in GetNeighbors(currentPos))
                {
                    if (IsVisited(closed, neighborPos)) continue;

                    var neighborCell = gridMap.GetCell(neighborPos);
                    
                    if (!CanMove(currentCell, neighborCell)) continue;

                    UpdateNeighbor(open, currentNode, neighborCell, endPos);
                }
            }

            return null;
        }

        private static bool IsValidPath(GridMap gridMap, Vector2Int startPos, Vector2Int endPos)
        {
            if (gridMap == null) return false;
            var startCell = gridMap.GetCell(startPos);
            var endCell = gridMap.GetCell(endPos);
            return CanMove(startCell, endCell);
        }

        private static Node GetBestNode(this List<Node> list) =>
            list.OrderBy(n => n.F).ThenBy(n => n.H).First();

        private static void VisitNode(List<Node> open, HashSet<Vector2Int> closed, Node node)
        {
            open.Remove(node);
            closed.Add(node.Position);
        }

        private static bool IsVisited(HashSet<Vector2Int> closed, Vector2Int pos) =>
            closed.Contains(pos);
        
        private static void UpdateNeighbor(List<Node> open, Node currentNode, Cell neighborCell, Vector2Int end)
        {
            var newCost = currentNode.Cost + neighborCell.MoveCost;
            var neighborNode = open.FirstOrDefault(n => n.Position == neighborCell.Position);

            if (neighborNode != null && newCost >= neighborNode.Cost) return;
                    
            neighborNode ??= CreateNode(open, neighborCell.Position, end);
            neighborNode.Cost = newCost;
            neighborNode.Parent = currentNode;
        }

        private static Node CreateNode(List<Node> open, Vector2Int pos, Vector2Int end)
        {
            var node = new Node(pos) { H = GetDistance(pos, end)};
            open.Add(node);
            return node;
        }

        public static bool CanMove(Cell fromCell, Cell toCell)
        {
            if (fromCell == null || toCell == null) return false;
            if (fromCell.NavigationID <= 0 || toCell.NavigationID <= 0) return false;
            if (fromCell.NavigationID != toCell.NavigationID) return false;
            return true;
        }

        private static List<Vector2Int> RetracePath(Vector2Int start, Node endNode)
        {
            var path = new List<Vector2Int>();
            var currentNode = endNode;

            while (currentNode.Position != start)
            {
                path.Add(currentNode.Position);
                currentNode = currentNode.Parent;
            }
            path.Reverse();
            return path;
        }

        private static int GetDistance(Vector2Int a, Vector2Int b)
        {
            var distance = a - b;
            return 10 * (Mathf.Abs(distance.x) + Mathf.Abs(distance.y));
        }

        private static IEnumerable<Vector2Int> GetNeighbors(Vector2Int pos)
        {
            yield return new Vector2Int(pos.x + 1, pos.y);
            yield return new Vector2Int(pos.x - 1, pos.y);
            yield return new Vector2Int(pos.x, pos.y + 1);
            yield return new Vector2Int(pos.x, pos.y - 1);
        }

        public static string PathToString(List<Vector2Int> path)
        {
            if (path == null || path.Count == 0) return "No Path";
            return string.Join(" -> ", path.Select(p => $"({p.x},{p.y})"));
        }
    }
}
```

</details>
