
## 一、Trie树的意义

Trie树（又称字典树、前缀树）的核心思想是用路径替代字符串的完整存储，通过类树结构存储前缀元素的操作，将对前缀遍历检索过程改变成对路径是否存在的判断，将原在列表中检索字符串前缀时间复杂度O(n* m)限定为O(n)。


### 适用场景

1. 前缀匹配：如查找所有以 `aa` 开头的字符串（日志检索、自动补全）；
    
2. 精准字符串查重：如判断 `aab` 是否已存入（词库校验、敏感词过滤）；
    
3. 公共前缀共享：如 `aab` 和 `aabb` 共享 `aab` 路径，大幅节省存储空间。
    

## 二、二叉树版Trie的设计与实现

### 1. 基础结构：节点类定义

每个节点是路径的“中转站”，需包含三个核心属性：

```Python
class TrieNode:
    def __init__(self):
        self.left = None   # 左子节点 → 对应字符a（往左走）
        self.right = None  # 右子节点 → 对应字符b（往右走）
        self.is_end = False  # 标记是否为某字符串的终止节点
```

- `left/right`：仅维护两个子节点，贴合“a/b”的限定场景，比通用Trie树（字典存储子节点）更轻量化；
    
- `is_end`：是区分“完整字符串”和“前缀”的关键——比如 `aab` 的最后一个节点标记为 `True`，而其前缀 `aa` 对应的节点标记为 `False`。
    

### 2. Trie树核心操作实现

#### （1）初始化：根节点是路径的起点

根节点不对应任何字符，仅作为所有路径的“入口”：

```Python
class Trie:
    def __init__(self):
        self.root = TrieNode()  # 根节点为空，无字符关联
```

#### （2）插入操作：构建字符路径并标记终止节点

插入逻辑的核心是“按字符遍历，无节点则创建，最后标记终止”：

```Python
def insert(self, s: str) -> None:
    current = self.root  # 从根节点开始遍历
    for char in s:
        if char == 'a':
            # a对应左子节点，不存在则创建
            if not current.left:
                current.left = TrieNode()
            current = current.left  # 移动到左子节点
        elif char == 'b':
            # b对应右子节点，不存在则创建
            if not current.right:
                current.right = TrieNode()
            current = current.right  # 移动到右子节点
    # 遍历完字符串，标记最后一个节点为终止节点
    current.is_end = True
```

**示例**：插入 `aab` 时，路径为「根→左→左→右」，最后一个“右节点”的 `is_end = True`；插入 `aabb` 时，复用「根→左→左→右」路径，新增「右节点」并标记 `is_end = True`。

#### （3）查找操作：验证路径存在且终止节点标记为True

查找的核心是“路径不中断 + 最后节点是终止节点”，避免把“前缀”误判为“完整字符串”：

```Python
def search(self, s: str) -> bool:
    current = self.root
    for char in s:
        if char == 'a':
            if not current.left:  # 路径中断，字符串不存在
                return False
            current = current.left
        elif char == 'b':
            if not current.right:  # 路径中断，字符串不存在
                return False
            current = current.right
    # 路径存在，需验证是否为完整字符串（而非前缀）
    return current.is_end
```

**示例**：查找 `aa` 时，路径「根→左→左」存在，但最后节点 `is_end = False`，返回 `False`；查找 `aab` 时，路径存在且最后节点 `is_end = True`，返回 `True`。

#### （4）前缀检查：仅验证路径存在即可

前缀检查无需关注终止节点，只需确认路径能完整遍历：

```Python
def startsWith(self, prefix: str) -> bool:
    current = self.root
    for char in prefix:
        if char == 'a':
            if not current.left:
                return False
            current = current.left
        elif char == 'b':
            if not current.right:
                return False
            current = current.right
    # 遍历完前缀未中断，说明前缀存在
    return True
```

**示例**：检查前缀 `aa` 时，路径「根→左→左」存在，直接返回 `True`。

### 3. 完整可运行代码

```Python
class TrieNode:
    def __init__(self):
        self.left = None   # 对应字符a
        self.right = None  # 对应字符b
        self.is_end = False

class Trie:
    def __init__(self):
        self.root = TrieNode()

    def insert(self, s: str) -> None:
        current = self.root
        for char in s:
            if char == 'a':
                if not current.left:
                    current.left = TrieNode()
                current = current.left
            elif char == 'b':
                if not current.right:
                    current.right = TrieNode()
                current = current.right
        current.is_end = True

    def search(self, s: str) -> bool:
        current = self.root
        for char in s:
            if char == 'a':
                if not current.left:
                    return False
                current = current.left
            elif char == 'b':
                if not current.right:
                    return False
                current = current.right
        return current.is_end

    def startsWith(self, prefix: str) -> bool:
        current = self.root
        for char in prefix:
            if char == 'a':
                if not current.left:
                    return False
                current = current.left
            elif char == 'b':
                if not current.right:
                    return False
                current = current.right
        return True
```

## 三、使用示例：验证核心功能

```Python
# 初始化Trie树
trie = Trie()

# 插入字符串
trie.insert("aab")   # 路径：根→左→左→右（is_end=True）
trie.insert("aabb")  # 路径：根→左→左→右→右（is_end=True）
trie.insert("ab")    # 路径：根→左→右（is_end=True）

# 测试search操作
print(trie.search("aab"))    # True（完整字符串存在）
print(trie.search("aa"))     # False（仅为前缀，无终止标记）
print(trie.search("aabb"))   # True（完整字符串存在）
print(trie.search("abc"))    # False（路径中断）

# 测试startsWith操作
print(trie.startsWith("aa"))   # True（前缀存在）
print(trie.startsWith("aab"))  # True（前缀存在）
print(trie.startsWith("ac"))   # False（路径中断）
```

**输出结果**：

```Plain
True
False
True
False
True
True
False
```



### 关键作用

- **时间效率**：`insert`/`search`/`startsWith` 时间复杂度均为 `O(n)`（`n` 为字符串长度），与存储的字符串数量无关，比哈希表更适合批量字符串匹配；
    
- **空间效率**：公共前缀共享节点（如 `aab` 和 `aabb` 共享前3个节点），字符重复度越高，空间优势越明显。

 

## 五、学习总结

1. Trie树的核心是利用路径节点存储前缀
    
2. end是判别“完整字符串”和“前缀”的核心，创建判定标准应当跟随正常的搜索流程；
    
3. Trie树在“前缀匹配”“字符串检索”等场景下效率高于暴力。