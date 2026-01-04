
#### 在匹配字符串时
- 暴力检索的指针会在无法匹配时回到 上一轮起始位置+1 的位置作为起点重新开始遍历匹配，使得时间复杂度达到O(n×m)
- kmp通过next数组（或PM组）存放已遍历的结果，记录已匹配的最长字符串的长度，使得指针在无法匹配时返回到已匹配字符串的下一位，避免重复遍历造成的复杂度增加，使得复杂度降低到O(n+m)


#### next数组
- next意指当字符串无法匹配时，指针下一轮重新开始匹配的指向
- next数组中存储了模式串每个位置的“最长相等前后缀长度”相关信息，用于指导匹配失败时模式串指针的回退位置
其中：
- next[0] = -1：标记模式串第0个字符的前缀为空
    
- next[i] = 0：表示模式串前i个字符（pattern[0..i-1]）没有长度≥1的相等前后缀（即“无前缀字符可匹配”）
    
- next[i] = k（k>0）：表示模式串前i个字符（pattern[0..i-1]）的“最长相等前后缀长度”为k

#### KMP算法运算示例
- 主串“ABABC...”，模式串“ABCABX”，当匹配到主串第4位（“B”）和模式串第4位（“B”）时成功，继续匹配主串第5位（“C”）和模式串第5位（“X”）失败。
- 此时next[5] = 2，说明模式串前5位（“ABCAB”）的最长相等前后缀是“AB”（长度2），因此模式串指针回退到2（指向“C”），主串指针继续留在5（指向“C”），直接比较两者即可，无需回退主串。

#### 代码实现

- 构建next数组
```python
def build_next(pattern):
    m = len(pattern)
    next_arr = [-1] * m
    i, j = 0, -1
    
    while i < m - 1:
        if j == -1 or pattern[i] == pattern[j]:
            i += 1
            j += 1
            next_arr[i] = j
        else:
            j = next_arr[j]
    
    return next_arr
```

- 进行kmp算法计算
```python
def kmp_search(main_str, pattern):
    n = len(main_str)
    m = len(pattern)
    
    if m == 0 or m > n:
        return -1
    
    next_arr = build_next(pattern)
    i = 0
    j = 0
    
    while i < n and j < m:
        if j == -1 or main_str[i] == pattern[j]:
            i += 1
            j += 1
        else:
            j = next_arr[j]
    
    if j == m:
        return i - m
    return -1
```
