---
title: 全排列问题(1)
date: 2015-08-11T14:52:17+08:00
lastmod: 2015-08-12
# draft: true
tags: [
    "Data Structure",
    "DFS"
]
categories: [
    "Development",
    "Algorithm"
]
---

全排列问题，指假如给定字符串，输出所有子串可能的排列的问题。解法比较多，递归非递归都可以。

> 深度优先搜素（Depth First Search,DFS)
> 搜索算法的一种。是沿着树的深度遍历树的节点，尽可能深的搜索树的分支。当节点 v 的所在边都己被探寻过，搜索将回溯到发现节点 v 的那条边的起始节点。这一过程一直进行到已发现从源节点可达的所有节点为止。如果还存在未被发现的节点，则选择其中一个作为源节点并重复以上过程，整个进程反复进行直到所有节点都被访问为止。属于盲目搜索。

DFS 的基本模型：

```c++
void dfs(int step)
{
    判断边界
    尝试每一种可能 for(i=1;i<=n;i++) {
        继续下一步 dfs(step+1);
    }
    return;
}
```

关键代码：

```c++
bool dfs(string &bunch, int bLen, string str)
{
    if (bLen == str.length()) {
            //
    }

    string::size_type i;
    for (i = 0; i < bLen; ++i) {
        if (!vis[i]) {
            vis[i] = true;
            str += bunch[i];
            if (dfs(bunch, bLen, str)) {
                return true;
            }
            vis[i] = false;
            str = str.substr(0, str.length() - 1);
        }
    }   // End of for
    return false;
}
```

---

全排列问题还有很多种变形问题，比如涉及去重等等。后面再具体补充。
