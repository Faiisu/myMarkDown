**Greedy Algorithm** for finding **Minimum Spanning Tree**

1. Sort Edges from low to high
2. Using [[Union-Find]] for finding circle
	- No circle : Add Node & weight
	- circle : Skip
3. continue to all node