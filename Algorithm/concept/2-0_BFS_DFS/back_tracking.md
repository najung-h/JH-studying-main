## ✅ 백트래킹(Backtracking)

---

백트래킹은
 **해를 찾기 위해 모든 가능한 경우의 수를 탐색하되, 탐색 과정에서 유망하지 않은 경우(해가 될 수 없는 경우)는 더 이상 진행하지 않고 되돌아가 다른 경로를 시도하는 기법**이다.

------

## 핵심 포인트

1. **탐색 + 되돌아가기**
   - 어떤 선택을 했다가, 더 이상 진행할 수 없거나 목적을 달성하지 못하면 그 선택을 취소(undo)하고 직전 상태로 돌아감.
2. **재귀(DFS)와 함께 자주 사용**
   - 보통 DFS와 결합되어, 탐색하면서 `visited`, `path` 같은 상태를 기록했다가 되돌리는 방식으로 구현됨.
3. **효율성**
   - 단순한 brute force(모든 경우 다 시도)와 달리, **불필요한 경로는 조기에 잘라내어 탐색량을 줄이는 것**이 특징.

------

## 대표 예시 문제

- **N-Queens 문제**: 체스판에 퀸을 놓을 때, 불가능한 배치라면 더 탐색하지 않고 바로 되돌아감.
- **미로 찾기**: 막힌 길을 만나면 되돌아가 다른 길 탐색.
- **부분집합/순열 생성**: 특정 조건에 맞지 않는 경우는 더 깊이 가지 않고 되돌아감.

------

📌 한 줄 요약:
 **백트래킹은 “가능한 해를 찾는 과정에서, 해가 될 수 없는 경로는 즉시 되돌아가며 탐색하는 알고리즘 기법”이다.**



---

## 📊 DFS 단순 탐색 vs 백트래킹 탐색

| 구분             | DFS 단순 탐색                                    | DFS + 백트래킹 탐색                                 |
| ---------------- | ------------------------------------------------ | --------------------------------------------------- |
| **목적**         | 한 번 방문한 노드는 다시 방문하지 않고 경로 탐색 | 모든 가능한 경로 탐색 + 불가능한 경로는 조기에 포기 |
| **visited 처리** | 방문한 노드는 끝까지 유지 (`remove` 안 함)       | 탐색이 끝나면 방문 기록을 되돌림 (`remove`)         |
| **경로 관리**    | 경로를 한 번만 기록                              | 경로에 추가 → 탐색 끝나면 제거 (복원)               |
| **탐색 결과**    | 특정 경로 하나(또는 도달 여부) 확인              | 가능한 모든 경로·조합·배치 탐색                     |
| **활용 예시**    | 그래프 연결 여부 확인, 최단 거리 BFS 대체        | N-Queens, 미로 찾기, 순열·조합 생성                 |
| **비유**         | “한 번 간 길은 다시 안 간다”                     | “길을 가다가 막히면 돌아와서 다른 길을 다시 간다”   |

------



예시 그래프 1 

```txt
1
├─ 2
└─ 3
```



## 1️⃣ DFS 단순 탐색 (visited만 유지)

```python
def dfs_simple(x, visited, path):
    visited.add(x)
    path.append(x)

    for nxt in graph[x]:
        if nxt not in visited:
            dfs_simple(nxt, visited, path)

    # ❌ visited와 path를 복원하지 않음
    # 한 번 지나간 노드는 다른 경로에서 못 씀

graph = {1:[2,3], 2:[], 3:[]}
dfs_simple(1, set(), [])
# 결과: 1 → 2 만 탐색, 1 → 3 은 못 탐색
```

------



## 2️⃣ DFS + 백트래킹 (되돌리기)

핵심은 **자기 재귀 호출(스택 프레임)이 끝날 때**, 즉 **자신이 맡은 모든 자식 탐색이 끝난 직후**에 `visited.remove(x)`와 `path.pop()`이 실행된다는 점이야. 그래서 이 두 줄은 **함수의 “퇴장 훅(post-order)”** 같은 거라고 보면 돼.

```python
def dfs_backtracking(x, visited, path):
    visited.add(x)
    path.append(x)

    for nxt in graph[x]:
        if nxt not in visited:
            dfs_backtracking(nxt, visited, path)

    # ✅ 탐색 끝난 후 되돌리기
    visited.remove(x)
    path.pop()
    '''왜 이 시점에서 되돌리는 거냐면, '''

graph = {1:[2,3], 2:[], 3:[]}
dfs_backtracking(1, set(), [])
# 결과: 1 → 2, 1 → 3 두 경로 모두 탐색
```

** 참고, 왜 루프 바깥에서 지울까?

```python
def dfs_backtracking(x, visited, path):
    visited.add(x)      # [입장] x 방문표시
    path.append(x)      # [입장] 경로에 x 기록

    for nxt in graph[x]:
        if nxt not in visited:
            dfs_backtracking(nxt, visited, path)  # 자식에게 맡김
            # (여기 시점에서는 "nxt가 스스로" visited/path를 되돌리고 반환함)

    visited.remove(x)   # [퇴장] x에 대한 모든 자식 탐색 끝 → x 흔적 제거
    path.pop()          # [퇴장] 경로에서 x 제거
```

- 각 **자식 호출**은 **자기 자신**을 들어갈 때 add/append 하고, **자기 자신**이 끝날 때 remove/pop 해.
- 그러고 부모로 돌아오면, 부모는 **여전히 x가 방문 중(visited에 있음)**인 상태에서 다음 형제 `nxt`를 본다.
  - 이것이 안전한 이유: **아래로 내려간 재귀가 부모(x)를 다시 밟는 것을 방지**하려면, 부모가 탐색 중에는 `visited`에 **남아 있어야** 해.
- 부모의 **모든 자식**을 끝까지 처리하고 나서야, 비로소 **부모 자신(x)** 을 지워도 돼.
   → 그래서 `visited.remove(x)`/`path.pop()`이 **for 루프 바깥(끝난 뒤)**에 있다.

> 만약 `visited.remove(x)`를 루프 안(자식 하나 끝날 때마다)에서 해버리면,
>  아직 다른 자식(다른 `nxt`)도 남았는데 **부모 x가 미방문 상태로 바뀌어** 아래 단계에서 x로 **역회귀(사이클)**가 생길 수 있어.
>  즉, **부모는 모든 자식 처리가 끝날 때** 내려놓는 게 맞다.

---



## 다이아몬드 그래프에서 차이 확인

```
1
├─ 2
└─ 3
 \ /
  4   (goal)
```

- 단순 DFS(전역 visited 유지, 복원 X): `4`를 한 번 방문하고 나면 **다른 경로의 4**로는 못 감 → **경로 하나만** 나옴
- 백트래킹(visited 복원): `4` 재사용 가능(경로마다 독립적으로 방문) → **모든 경로**가 나옴

```python
graph = {1:[2,3], 2:[4], 3:[4], 4:[]}
start, goal = 1, 4

# 1) 단순 DFS: visited 복원 없음 → 경로 일부 누락
def dfs_simple(u, goal, visited, path):
    visited.add(u)
    path.append(u)
    if u == goal:
        print("[단순DFS] 경로:", path)
        return
    for v in graph[u]:
        if v not in visited:
            # path는 복원 안 하므로, 안전하게 복사해서 내려감(실행 오류 방지용)
            dfs_simple(v, goal, visited, path[:])

# 2) 백트래킹 DFS: visited/path 복원 → 모든 경로 출력
def dfs_backtrack(u, goal, visited, path):
    visited.add(u)
    path.append(u)
    if u == goal:
        print("[백트래킹] 경로:", path)
    else:
        for v in graph[u]:
            if v not in visited:
                dfs_backtrack(v, goal, visited, path)
    # 되돌리기(핵심)
    visited.remove(u)
    path.pop()

print("=== 단순 DFS 결과 ===")
dfs_simple(start, goal, set(), [])

print("\n=== 백트래킹 DFS 결과 ===")
dfs_backtrack(start, goal, set(), [])
```

### 예상 출력(설명)

```diff
=== 단순 DFS 결과 ===
[단순DFS] 경로: [1, 2, 4]
# 4가 이미 visited이므로 [1, 3, 4]는 안 나옴

=== 백트래킹 DFS 결과 ===
[백트래킹] 경로: [1, 2, 4]
[백트래킹] 경로: [1, 3, 4]
```

------

## 한 줄 요약

- **백트래킹 = DFS + 상태 되돌리기**로, **불가능/비유망한 분기 즉시 포기**하고 **경로마다 독립적인 방문 상태**를 유지해 **모든 해/경로**를 탐색합니다.