
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


# 回溯


组合问题:N个数里面按一定规则找出k个数的集合 
切割问题:一个字符串按一定规则有几种切割方式 
子集问题:一个N个数的集合里有多少符合条件的子集 
排列问题:N个数按一定规则全排列，有几种排列方式 
棋盘问题:N皇后，解数独等等
![[截屏2023-03-17 22.31.52.png]]




组合是不强调元素顺序的，排列是强调元素顺序。

![[Pasted image 20230317203038.png]]



## 组合问题

如果是同一个集合里面找数据，每次都需要startIndex，如果是不同集合里面找数据，就是不需要startIndex。

### 含有k个元素的组合

给定两个整数k和n，返回1~n中可能k个元素的组合

思路：
![[截屏2023-03-17 20.36.58.png]]

每次从集合中选取元素，可选择的范围随着选择的进行而收缩，调整可选择的范围，就是要 靠startIndex。


```java
List<List<Integer>> ans = new ArrayList<>();  
  
public List<List<Integer>> combine(int n, int k) {  
    // 直接从1 开始，之后的数据  
    back(n, k, new ArrayList<>(), 1);  
    return ans;  
}  
  
public void back(int n, int k, List<Integer> tmpRes, int start) {  
    if (tmpRes.size() == k) {  
        ans.add(new ArrayList<>(tmpRes));  
        return;    }  
  
    for (int i = start; i <= n; i++) {  
        tmpRes.add(i);  
        back(n, k, tmpRes, i + 1);  
        tmpRes.remove(tmpRes.size() - 1);  
    }  
}

```

### 组合总和3

题目：
找出所有相加之和为 n 的 k 个数的组合，且满足下列条件：
	只使用数字1到9
	每个数字 最多使用一次 
返回 所有可能的有效组合的列表 。该列表不能包含相同的组合两次，组合可以以任何顺序返回。

思路：
targetSum(int)目标和，也就是题目中的n。
k(int)就是题目中要求k个数的集合。 
sum(int)为已经收集的元素的总和，也就是path里元素的总和。
startIndex(int)为下一层for循环搜索的起始位置。


![[截屏2023-03-17 21.00.39.png]]


```java
List<List<Integer>> ans = new ArrayList<>();  
public List<List<Integer>> combinationSum3(int k, int n) {  
    // k 作为树的深度，1到9作为树的宽度  
    backed(k, n, 0, 1, new ArrayList<>());  
    return ans;  
}  
public void backed(int k, int n, int sum, int begin, List<Integer> tmp) {  
    if (tmp.size() == k) { // 在往下走就没有意义了。  
        if (sum == n) ans.add(new ArrayList<>(tmp));  
        return;    }  
  
    for (int i = begin; i <= 9; i++) {  
        sum += i;  
        tmp.add(i);  
        backed(k, n, sum, i + 1, tmp);  
        sum -= i;  
        tmp.remove(tmp.size() - 1);  
    }  
}
```



### 组合总和

1）数字无重复
2）同一个数字可以重复使用


```java

List<List<Integer>> ans = new ArrayList<>();  
public List<List<Integer>> combinationSum(int[] candidates, int target) {  
    backed(candidates, target, new ArrayList<>(), 0, 0);  
    return ans;  
}  
public void backed(int[] candidates, int target, List<Integer> tmp, int sum, int startIndex) {  
  
    if (sum == target) {  
        ans.add(new ArrayList<>(tmp));  
        return;    }  
  
    if (sum > target) {  
        return;  
    }  
    for (int i = startIndex; i <= candidates.length; i++) {  
        int num = candidates[i];  
        sum += num;  
        tmp.add(num);  
        backed(candidates, target, tmp, sum, i); // 还是选择i保证可以重复使用 
        sum -= num;  
        tmp.remove(tmp.size() - 1);  
    }  
  
}
```

### 组合总和2

给定一个组数据，和目标值target 找到数据加起来等于target的组合

1）数字有重复
2）同一个数字只能使用一次

强调一下，树层去重的话，需要对数组排序!

![[截屏2023-03-17 21.24.15.png]]


1）回溯三部曲中需要加入一个used数组
![[截屏2023-03-17 21.47.59.png]]

used主要用来去重的，去重的是同一层中已经取过重复的数值了，

```
1)if candidate[i]==candidate[i-1]&&used[i-1]=true 
  说明同一个树枝上，同一个节点上，candidate[i-1]已经使用过
2)if candidate[i]==candidate[i-1]&&used[i-1]=false 
   说明这个节点上，并没有使用相同的值，那么同一层中肯定有个节点是使用了，所以直接跳过就可

```

```java
List<List<Integer>> ans = new ArrayList<>();  
boolean[] used;  
  
public List<List<Integer>> combinationSum2(int[] candidates, int target) {  
    used = new boolean[candidates.length];  
    // 首先将数组排序  
    Arrays.sort(candidates);  
    back(candidates, target, 0, new ArrayList<>(), 0);  
    return ans;  
}  
public void back(int[] nums, int target, int sum, List<Integer> tmp, int index) {  
    if (sum == target) {  
        ans.add(new ArrayList<>(tmp));  
        return;    }  
    if (sum > target) {  
        return;  
    }  
  
    for (int i = index; i < nums.length; i++) {  
        // 说明前一个相同的数字在本次没有被使用，那么在同一个层的前一个肯定被使用，就会出现  
        if (i > 0 && nums[i] == nums[i - 1] && used[i - 1] == false) {  
            continue;  
        }  
        used[i] = true;  
        int num = nums[i];  
        sum += num;  
        tmp.add(num);  
        back(nums, target, sum, tmp, i + 1);  
        used[i] = false;  
        sum -= nums[i];  
        tmp.remove(tmp.size() - 1);  
    }  
  
}
```


## 分割问题


### 分割回文串

给定一个字符串 `s` ，请将 `s` 分割成一些子串，使每个子串都是 **回文串** ，返回 s 所有可能的分割方案。

**回文串** 是正着读和反着读都一样的字符串。

思路：

![[截屏2023-03-17 22.09.30.png]]

根据上述图：
在处理组合问题的时候，递归参数需要传入startIndex，表示下一轮递归遍历的起始位置， 这个startIndex就是切割线。

```java
List<List<String>> ans = new ArrayList<>();  
  
public String[][] partition(String s) {  
    back(s, 0, new ArrayList<>());  
    String[][] res = new String[ans.size()][];  
  
    for (int i = 0; i < ans.size(); i++) {  
        List<String> list = ans.get(i);  
        res[i] = list.toArray(new String[list.size()]);  
    }  
    return res;  
}  
  
public void back(String s, int index, List<String> tmp) {  
    if (index == s.length()) {  
        ans.add(new ArrayList<>(tmp));  
        return;    }  
  
    for (int i = index; i < s.length(); i++) {  
        // 判断从index，到i截取的字符串是否是回文  
        String value = s.substring(index, i + 1);  
        if (!valid(value)) continue;  
        tmp.add(value);  
        back(s, i + 1, tmp);  
        tmp.remove(tmp.size() - 1);  
    }  
}  
  
  
public boolean valid(String s) {  
    int left = 0, right = s.length() - 1;  
    while (left <= right) {  
        if (s.charAt(left) != s.charAt(right))  
            return false;  
        left++;  
        right--;  
    }  
    return true;  
}
```


### 分割ip

## 子集问题

### #### [所有子集](https://leetcode.cn/problems/TVdhkn/)

1）无重复值

组合是找叶子节点，子集是找到树上的每个节点。
其实子集也是一种组合问题，因为它的集合是无序的，子集{1,2} 和 子集{2,1}是一样的。


那么既然是无序，取过的元素不会重复取，写回溯算法的时候，for就要从startIndex开始，而不是从0开始!
![[截屏2023-03-17 22.49.29.png]]

```java

  
List<List<Integer>> ans = new ArrayList<>();  
  
public List<List<Integer>> subsets(int[] nums) {  
    back(nums, 0, new ArrayList<>());  
    return ans;  
}  
  
public void back(int[] nums, int index, List<Integer> temp) {  
    ans.add(new ArrayList<>(temp));  // 遍历到树上的每个节点都要加入结果集中
    if (index >= nums.length) return;  
  
    for (int i = index; i < nums.length; i++) {  
        int num = nums[i];  
        temp.add(num);  
        back(nums, i + 1, temp);  
        temp.remove(temp.size() - 1);  
    }  
}
```

### 子集2

1）有重复值
输入: [1,2,2]  
输出:  
[  
[2],  
[1],  
[1,2,2],  
[2,2],  
[1,2],  
[]  
]
```java
List<List<Integer>> result = new ArrayList<>();  
boolean[] used;  
public List<List<Integer>> subsetsWithDup(int[] nums) {  
    Arrays.sort(nums);  
    used = new boolean[nums.length];  
    back(nums, 0, new ArrayList<>());  
    return result;  
}  
  
private void back(int[] nums, int startIndex, List<Integer> temp) {  
    result.add(new ArrayList<>(temp));  
    for (int i = startIndex; i < nums.length; i++) {  
        if (i > 0 && nums[i] == nums[i - 1] && !used[i - 1]) {  
            continue;  
        }  
        temp.add(nums[i]);  
        used[i] = true;  
        back(nums, i + 1, temp);  
        used[i] = false;  
        temp.remove(temp.size() - 1);  
    }  
}
```

## 

解集 **不能** 包含重复的子集。你可以按 **任意顺序** 返回解集。

```java
// 需要设定一个size 根据长度进行
List<List<Integer>> ans = new ArrayList<>();  
public List<List<Integer>> subsets(int[] nums) {  
    for (int i = 0; i <= nums.length; i++) {  
        // 设定子集的size  
        DFS(nums, 0, i, new ArrayList<>());  
    }  
    return ans;  
}  
  
public void DFS(int[] nums, int begin, int size, List<Integer> tmp) {  
    if (tmp.size() == size) {  
        ans.add(new ArrayList<>(tmp));  
        return;    }  
    for (int i = begin; i < nums.length; i++) {  
        tmp.add(nums[i]);  
        DFS(nums, begin + 1, size, tmp);  
        tmp.remove(tmp.size() - 1);  
    }  
}
```

## 组合问题

### 没有重复元素的全排列

首先排列是有序的，也就是说[1,2] 和[2,1] 是两个集合，这和之前分析的子集以及组合所不 同的地方。![[截屏2023-03-17 23.16.56.png]]

1）为什么不需要使用startIndex
可以看出元素1在[1,2]中已经使用过了，但是在[2,1]中还要在使用一次1，所以处理排列问题 就不用使用startIndex了。

2）为什么需要used数组
![[截屏2023-03-17 23.18.19.png]]

3）什么时候终止？
叶子节点

```java

  
List<List<Integer>> ans = new ArrayList<>();  
boolean[] used;  
  
public List<List<Integer>> permute(int[] nums) {  
    used = new boolean[nums.length];  
    back(nums, new ArrayList<>());  
    return ans;  
}  
  
public void back(int[] nums, List<Integer> temp) {  
    if (temp.size() == nums.length) {  
        ans.add(new ArrayList<>(temp));  
        return;    }  
    for (int i = 0; i < nums.length; i++) {  
        if (used[i]) continue;  
        used[i] = true;  
        temp.add(nums[i]);  
        back(nums, temp);  
        used[i] = false;  
        temp.remove(temp.size() - 1);  
    }  
}
```

### 有重复元素的全排列

```java

List<List<Integer>> ans = new ArrayList<>();  
boolean[] used;  
  
public List<List<Integer>> permuteUnique(int[] nums) {  
    Arrays.sort(nums);  
    used = new boolean[nums.length];  
    back(nums, new ArrayList<>());  
    return ans;  
}  
  
public void back(int[] nums, ArrayList<Integer> tmp) {  
    if (nums.length == tmp.size()) {  
        ans.add(new ArrayList<>(tmp));  
        return;    }  
  
    for (int i = 0; i < nums.length; i++) {  
        if (used[i]) continue;  
        if (i > 0 && nums[i] == nums[i - 1] && !used[i - 1]) continue;  
        used[i] = true;  
        tmp.add(nums[i]);  
        back(nums, tmp);  
        used[i] = false;  
        tmp.remove(tmp.size() - 1);  
    }  
}
```