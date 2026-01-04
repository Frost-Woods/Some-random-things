### UF的使用
- 并查集选择以数组下标索引作为对不同元素节点的标记，以索引对应的数组内容标记该节点元素的根节点：
		其中Find方法通过读取数组内容追溯根节点，数组内容将指向值与当前索引内容相同的索引，直到对应值与索引本身相同时停止搜索，即找到根节点。
		而Union方法则是将多个树的根节点合并，使得树中元素能相互连通。

```python
#假设有一无向图test
class UnionFind: 
	def __init__(self, size): # 初始化：每个元素的父节点指向自己 
		self.parent = list(range(size)) 
	
	def find(self, x): 
		if self.parent[x] != x: # 递归查找根节点 
			return self.find(self.parent[x]) 
		return x 
	
	def union(self, x, y):
		root_x = self.find(x) 
		root_y = self.find(y) 
		if root_x != root_y: # 直接将一个根节点的父节点指向另一个根节点
			self.parent[root_y] = root_x 
	
	def connected(self, x, y):
		return self.find(x) == self.find(y)
```
### 缺点
- 最坏情况下find可能出现链式递归，导致时间复杂度达到O(n)
- union操作可能导致树高度过高，增加find的时间复杂度

### UFO思路
- 路径压缩：即在数组中直接存储索引对应根节点，而不是储存上一级节点。
```python
def find(self, x):
    # 先找到根节点
    root = x
    while self.parent[root] != root:
        root = self.parent[root]
    # 路径压缩：将x到根节点路径上的所有节点直接指向根
    while self.parent[x] != root:
        # 暂存当前节点的父节点
        next_parent = self.parent[x]
        self.parent[x] = root
        x = next_parent
    return root
```
- 按秩合并：按照一定标准执行union，使得合并树高度尽可能低。