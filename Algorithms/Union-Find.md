The **Union-Find Algorithm**, also known as the **Disjoint Set Union (DSU)**

use for track of elements that are partitioned into **disjoint (non-overlapping) sets**. It provides two key operations:
- Find(x) -> Determines the **Leader**
- Union(x,y) -> Merges the sets X and Y
It is commonly used in **graph algorithms** like [[Kruskal's Algorithm]] for **Minimum Spanning Tree (MST)** and problems involving **connectivity queries**.

## Union By Rank/Size
- **Union By Rank**: ensures that the shorter tree is always merged under the taller tree.
- **Union By Size** merges the smaller set under the larger set to keep the tree as flat as possible.

**Cpp example**
```cpp
int parents[100010], Ranks[100010];

int find(int x, int y){
	if(parents[x] != x) return find(parents[x]);
	else return x;
}

void union(int x, int y){
	int pX = find(x);
	int pY = find(y);
	if(Ranks[pX] < Ranks[Py]) parents[pX] = pY
	else if( Ranks[Px] > Ranks[Py]) parents[pY] = pX;
	else{
		parent[rootY] = rootX;
		rank[rootX]++;
	}
}
```
