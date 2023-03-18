
# DFS

## 岛屿问题

### 岛屿的最大面积

遇到1的时候就是面积，找到1连接的上下左右最大的

```java
int max = 0;  
int cur = 0;  
  
public int maxAreaOfIsland(int[][] grid) {  
    for (int i = 0; i < grid.length; i++) {  
        for (int j = 0; j < grid[0].length; j++) {  
            if (grid[i][j] == 1) {  
                cur = 0;  
                dfs(i, j, grid);  
                max = Math.max(max, cur);  
            }  
        }  
    }  
    return max;  
}  
  
public void dfs(int x, int y, int[][] grid) {  
    if (x < 0 || y < 0 || x >= grid.length || y >= grid[0].length || grid[x][y] == 0) return;  
  
    if (grid[x][y] == 1) {  
        grid[x][y] = 0;  
        cur++;  
    }  
    dfs(x - 1, y, grid);  
    dfs(x + 1, y, grid);  
    dfs(x, y - 1, grid);  
    dfs(x, y + 1, grid);  
}
```




## 括号生成

https://leetcode.cn/problems/generate-parentheses/

![[Pasted image 20230312215715.png]]

画图以后，可以分析出的结论：
- 当前左右括号都有大于 0个可以使用的时候，才产生分支；
- 产生左分支的时候，只看当前是否还有左括号可以使用；
- 产生右分支的时候，还受到左分支的限制，右边剩余可以使用的括号数量一定得在严格大于左边剩余的数量的时候，才可以产生分支；
- 在左边和右边剩余的括号数都等于 0 的时候结算

无需回溯，直接进行遍历
```java
public class Solution { 
    List<String> res = new ArrayList<>();  
    public List<String> generateParenthesis(int n) {  
        if (n == 0) {  
            return res;  
        }  
        // 执行深度优先遍历，搜索可能的结果  
        dfs("", n, n, res);  
        return res;  
    }  
  
    private void dfs(String curStr, int left, int right) {  
// 因为每一次尝试，都使用新的字符串变量，所以无需回溯 在递归终止的时候，直接把它添加到结果集即可
        if (left == 0 && right == 0) {  
            res.add(curStr);  
            return;        
        }  
        // 剪枝（如图，左括号可以使用的个数严格大于右括号可以使用的个数，才剪枝，注意这个细节）  
        if (left > right) {  
            return;  
        }  
  
        if (left > 0) {  
            dfs(curStr + "(", left - 1, right);  
        }  
  
        if (right > 0) {  
            dfs(curStr + ")", left, right - 1);  
        }  
    }  
}
```

