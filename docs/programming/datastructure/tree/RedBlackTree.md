# Red Black Tree

## 개요

- 레드 블랙 트리 (Red Black Tree) 는 이진 탐색 트리 (Binary Search Tree) 의 일종
- 색 (color) 정보를 추가하여 최악의 경우에도 높이를 O(logn) 으로 보장
  
## 특징

- 노드는 빨강(Red) 또는 검정(Black) 색을 가진다
- 루트 노드는 항상 검정
- 모든 리프(nil) 노드는 검정
  - (구현상 null 포인터를 하나의 단일 sentinel nil 노드로 취급)
- 빨간 노드의 자식은 모두 검정
  - 빨강 노드가 연속으로 이어질 수 없음
- 어떤 노드에서든 그 노드로부터 리프(nil)까지 가는 경로상의 “검정 노드 수(black-height)”는 모두 동일

이 5가지 제약 덕분에, 루트에서 리프까지의 가장 긴 경로는 짧은 경로의 최대 두 배를 넘지 않아 h=O(logn)를 보장

## 원리

### 2.1 회전

```plain
      x                   y
     / \                 / \
    a   y     →         x   c
       / \             / \
      b   c           a   b
```

- 좌회전(Left-Rotate), 우회전(Right-Rotate)
- 서브트리의 구조만 바꿔주고, 이진 탐색 트리 속성(왼쪽 작음, 오른쪽 큼)은 유지
- 삽입/삭제 후 균형을 맞추기 위해 사용

### 2.2 재색칠

- 회전만으로 삽입/삭제 후 모든 속성을 맞추지 못할 때
- 부모·삼촌 노드 색을 검정으로 바꾸고, 조상(할아버지) 노드를 빨강으로 색칠한 뒤 위로 올라가 재조정

## 연산

### 삽입

- 일반 이진 탐색 트리처럼 새 노드를 빨강으로 삽입
- 부모가 검정이라면 완료
- 부모가 빨강인 경우, 할아버지·삼촌 노드 상태에 따라
  - 삼촌이 빨강: 부모·삼촌을 검정, 할아버지를 빨강으로 만들고, 할아버지를 새로운 삽입 노드로 설정해 반복
  - 삼촌이 검정: 회전 + 색 교환
    - 새 노드가 부모의 “내부(오른쪽-왼쪽)” 위치면, 부모 기준 회전 후 “외부” 위치로 만들어 처리
    - “외부” 위치이면, 할아버지 기준 회전 후 부모·할아버지 색 교환

```java
class RBNode<K,V> {
    K key; V value;
    RBNode<K,V> left, right, parent;
    boolean isRed;
}

void insert(K key, V value) {
    RBNode<K,V> z = new RBNode<>(key, value, true);
    // 1) BST 삽입
    RBNode<K,V> y = nil, x = root;
    while (x != nil) {
        y = x;
        x = (z.key.compareTo(x.key) < 0) ? x.left : x.right;
    }
    z.parent = y;
    if (y == nil) root = z;
    else if (z.key.compareTo(y.key) < 0) y.left = z;
    else y.right = z;
    // 2) 색 및 회전 보정
    insertFixup(z);
}

void insertFixup(RBNode<K,V> z) {
    while (z.parent.isRed) {
        if (z.parent == z.parent.parent.left) {
            RBNode<K,V> y = z.parent.parent.right; // 삼촌
            if (y.isRed) {
                // Case 1: 삼촌이 빨강
                z.parent.isRed = false;
                y.isRed = false;
                z.parent.parent.isRed = true;
                z = z.parent.parent;
            } else {
                if (z == z.parent.right) {
                    // Case 2: 내부형 삽입
                    z = z.parent;
                    leftRotate(z);
                }
                // Case 3: 외부형 삽입
                z.parent.isRed = false;
                z.parent.parent.isRed = true;
                rightRotate(z.parent.parent);
            }
        } else {
            // 대칭 처리 (오른쪽 부모 케이스)
            ...
        }
    }
    root.isRed = false; // 속성 2 보장
}
```

### 삭제

- 일반 BST 삭제: 삭제할 노드 z 선택
- 실제 제거되는 노드 y: 자식이 둘인 경우 후속 노드(successor)와 swap
- 삭제 전 y의 색을 yColor에 저장
- 자식 x를 y의 위치로 붙여넣음
- 만약 yColor가 BLACK이었다면, black-height가 하나 줄어들었기 때문에 deleteFixup(x)로 균형 복원

deleteFixup(x)는 “블랙 결핍(double-black)” 상태를 해소하기 위해

- 형제 노드(sibling) 색과 자식 색에 따라 회전·재색칠
- 경우를 나눠 총 4가지 케이스 처리

## 시간, 공간 복잡도

- 검색, 삽입, 삭제: 모두 O(logn)
- 공간: 노드당 추가로 색 비트를 하나 사용하므로 O(n)
