---
title: "leetcode 刷题笔记"
description: 
date: 2025-04-01T16:24:26Z
categories:
    - 数据结构与算法
math: true
---

## 动态规划（DP）

### 最长公共子序列

> dp[i][j] : text1[0:i] 和 text2[0:j] 的公共子序列的最大长度。

```java
int m = text1.length(), n = text2.length();
int[][] dp = new int[m + 1][n + 1];

for (int i = 1; i <= m; i++) {
    char c1 = text1.charAt(i - 1);
    for (int j = 1; j <= n; j++) {
        char c2 = text2.charAt(j - 1);
        if (c1 == c2) {
            dp[i][j] = dp[i - 1][j - 1] + 1;
        } else {
            dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1]);
        }
    }
}
```

### 最长递增子序列
> dp[i] ：以第 i 个数字结尾的最长递增子序列的长度，注意 nums[i] 必须被选取
```java
int[] dp = new int[n];
Arrays.fill(dp, 1);
int ans = -1;
for (int i = 0; i < n; i++) {
    for (int j = 0; j < i; j++) {
        if (nums[j] < nums[i]) {
            dp[i] = Math.max(dp[i], dp[j] + 1);
        }
    }
    ans = Math.max(ans, dp[i]);
}
```
### 背包问题
> dp[j] : 到目前元素 i 为止，背包剩余容量为 j 时的最大价值。

#### 0-1背包
求最大价值
```java
int[] dp = new int[capacity + 1];
for (int w: weights) {
    for (int j = capacity; j >= w; j--) {
        dp[j] = Math.max(dp[j], dp[j - w] + v);
    }
}
```
求方案数
```java
int[] dp = new int[capacity + 1];
dp[0] = 1;
for (int w: weights) {
    for (int j = capacity; j >= w; j--) {
        dp[j] += dp[j - w];
    }
}
```
求最大长度
```java
int[] dp = new int[capacity + 1];
Arrays.fill(dp, Integer.MIN_VALUE);
dp[0] = 0;
for (int w: weights) {
    for (int j = capacity; j >= w; j--) {
        dp[j] = Math.max(dp[j], dp[j - w] + 1);
    }
}
```
求最小长度
```java
int[] dp = new int[capacity + 1];
Arrays.fill(dp, Integer.MAX_VALUE / 2);
dp[0] = 0;
for (int w: weights) {
    for (int j = capacity; j >= w; j--) {
        dp[j] = Math.min(dp[j], dp[j - w] + 1);
    }
}
```

#### 完全背包
求最大价值
```java
int[] dp = new int[capacity + 1];
for (int w: weights) {
    for (int j = w; j <= capacity; j++) {
        dp[j] = Math.max(dp[j], dp[j - w] + v);
    }
}
```


### 状态机

> dp[i][j] 表示第 i 天结束之后，当前状态为 j 的累计最大收益。

```java
int[][] dp = new int[steps][stateCount];
initialize(dp[0]);  // 设定初始状态
for (int t = 1; t < steps; t++) { // 遍历时间
    for (int s = 0; s < stateCount; s++) { // 遍历状态
        dp[t][s] = transition(dp, t, s); // 计算状态转移
    }
}
```


## 二分查找
### 搜索确定值，闭区间
```java
public int bsearch(int[] nums, int target) {
    int left = 0;
    int right = nums.length - 1;

    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] == target) {
            return mid;
        } else if (nums[mid] < target) {
            left = mid + 1;
        } else if (nums[mid] > target) {
            right = mid - 1;
        }
    }
    return -1;
}
```
### 搜索左边界，左闭右开
```java
public int bsearch(int[] nums, int target) {
    int left = 0;
    int right = nums.length;

    while (left < right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] < target) {
            left = mid + 1;
        } else {
            right = mid;
        }
    }
    return left; // >=target 的最小值；
}
```
### 搜索右边界，左闭右开
```java
public int bsearch(int[] nums, int target) {
    int left = 0;
    int right = nums.length;

    while (left < right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] > target) {
            right = mid;
        } else {
            left = mid + 1;
        }
    }
    return left - 1; // <=target 的最大值
}
```

- 第 k 小：求最小的 x，满足 $\le x$ 的数至少有 k 个。
- 第 k 大：求最大的 x，满足 $\ge x$ 的数至少有 k 个。

## 回溯
选择 → 回溯 → 撤销选择

### 元素无重，不可复选
组合
```java
class Solution {
    List<List<Integer>> res = new LinkedList<>();
    LinkedList<Integer> track = new LinkedList<>();

    public List<List<Integer>> combine(int n, int k) {
        backtrack(1, n, k);
        return res;
    }

    void backtrack(int start, int n, int k) {
        if (k == track.size()) {
            res.add(new LinkedList<>(track));
            return;
        }
        for (int i = start; i <= n; i++) {
            track.addLast(i);
            backtrack(i + 1, n, k);
            track.removeLast();
        }
    }
}
```
排列
```java
class Solution {
    List<List<Integer>> res = new LinkedList<>();
    LinkedList<Integer> track = new LinkedList<>();
    boolean[] used;

    public List<List<Integer>> permute(int[] nums) {
        used = new boolean[nums.length];
        backtrack(nums);
        return res;
    }

    void backtrack(int[] nums) {
        if (track.size() == nums.length) {
            res.add(new LinkedList(track));
            return;
        }

        for (int i = 0; i < nums.length; i++) {
            if (used[i]) continue;
            used[i] = true; track.addLast(nums[i]);
            backtrack(nums);
            track.removeLast(); used[i] = false;
        }
    }
}
```

### 元素可重，不可复选
组合
```java
class Solution {
    List<List<Integer>> res = new LinkedList<>();
    LinkedList<Integer> track = new LinkedList<>();
    int trackSum = 0;

    public List<List<Integer>> combine(int[] nums, int target) {
        Arrays.sort(nums);
        backtrack(nums, 0, target);
        return res;
    }

    void backtrack(int[] nums, int start, int target) {
        if (trackSum == target) {
            res.add(new LinkedList<>(track));
            return;
        }
        if (trackSum > target) {
            return;
        }
        for (int i = start; i < nums.length; i++) {
            if (i > start && nums[i] == nums[i - 1]) {
                continue;
            }
            track.add(nums[i]); trackSum += nums[i];
            backtrack(nums, i + 1, target);
            track.removeLast(); trackSum -= nums[i];
        }
    }
}
```
排列
```java
class Solution {
    List<List<Integer>> res = new LinkedList<>();
    LinkedList<Integer> track = new LinkedList<>();
    boolean[] used;

    public List<List<Integer>> permute(int[] nums) {
        Arrays.sort(nums);
        used = new boolean[nums.length];
        backtrack(nums);
        return res;
    }

    void backtrack(int[] nums) {
        if (track.size() == nums.length) {
            res.add(new LinkedList(track));
            return;
        }
        for (int i = 0; i < nums.length; i++) {
            if (used[i]) {
                continue;
            }
            if (i > 0 && nums[i] == nums[i - 1] && !used[i - 1]) {
                continue;
            }
            track.add(nums[i]);
            used[i] = true;
            backtrack(nums);
            track.removeLast();
            used[i] = false;
        }
    }
}
```
### 元素无重，可复选
- 组合：将backtrack(nums, i + 1)改为 backtrack(nums, i)
- 排列：直接删去boolean[] used数组和判断过程。


## 前缀和、差分、线段树
- 数组元素不变，求区间和 → 前缀和
- 数组区间变化，求单点值 → 差分
- 数组区间变化，求区间和 → 线段树
### 前缀和
一维前缀和
```java
class NumArray {
    private int[] preSum;

    public NumArray(int[] nums) {
        preSum = new int[nums.length + 1];
        for (int i = 1; i < preSum.length; i++) {
            preSum[i] = preSum[i - 1] + nums[i - 1];
        }
    }

    public int sumRange(int left, int right) {
        return preSum[right + 1] - preSum[left];
    }
}
```
二维前缀和
```java
class NumMatrix {
    private int[][] preSum;

    public NumMatrix(int[][] matrix) {
        int m = matrix.length, n = matrix[0].length;
        if (m == 0 || n == 0) return;
        preSum = new int[m + 1][n + 1];
        for (int i = 1; i <= m; i++) {
            for (int j = 1; j <= n; j++) {
                preSum[i][j] = preSum[i-1][j] + preSum[i][j-1] + matrix[i - 1][j - 1] - preSum[i-1][j-1];
            }
        }
    }

    public int sumRegion(int x1, int y1, int x2, int y2) {
        return preSum[x2+1][y2+1] - preSum[x1][y2+1] - preSum[x2+1][y1] + preSum[x1][y1];
    }
}
```

### 差分数组
```java
class Difference {
    private int[] diff;
    
    public Difference(int[] nums) {
        diff = new int[nums.length];
        diff[0] = nums[0];
        for (int i = 1; i < nums.length; i++) {
            diff[i] = nums[i] - nums[i - 1];
        }
    }

    public void increment(int i, int j, int val) {
        diff[i] += val;
        if (j + 1 < diff.length) {
            diff[j + 1] -= val;
        }
    }

    public int[] result() {
        int[] res = new int[diff.length];
        res[0] = diff[0];
        for (int i = 1; i < diff.length; i++) {
            res[i] = res[i - 1] + diff[i];
        }
        return res;
    }
}
```

## 滑动窗口
```java
void slidingWindow(String s) {
    // 用合适的数据结构记录窗口中的数据，根据具体场景变通
    // 比如说，我想记录窗口中元素出现的次数，就用 map
    // 如果我想记录窗口中的元素和，就可以只用一个 int
    Object window = ...
    
    int left = 0, right = 0;
    while (right < s.length()) {
        char c = s[right++];
        window.add(c)
        ...  // 进行窗口内数据的一系列更新
        
        // printf("window: [%d, %d)\n", left, right);
        
        while (left < right && window needs shrink) {
            char d = s[left++];
            window.remove(d)
            ... // 进行窗口内数据的一系列更新
        }
    }
}
```
### 求子数组个数
越长越合法
```java
int ans = 0;
int left = 0, right = 0, cnt = 0;
while(right < nums.length) {
    if(nums[right++] == target) cnt++;
    while(left < right && cnt >= k) {
        if(nums[left++] == target) cnt--;
    }
    ans += left;
}
return ans;
```

越短越合法

```java
...
    ans += right - left;
...
```

恰好型滑动窗口：

例如，要计算有多少个元素和恰好等于 k 的子数组，可以把问题变成：
- 计算有多少个元素和 $\ge k$ 的子数组。
- 计算有多少个元素和 > k，也就是 $\ge k + 1$ 的子数组。
答案就是元素和 $\ge k$的子数组个数，减去元素和$\ge k+1$的子数组个数。这里把 > 转换成 $\ge$，从而可以把滑窗逻辑封装成一个函数 f()，然后用 f(k) - f(k + 1)计算，无需编写两份滑窗代码。

## 单调栈
求下一个更大元素
```java
public int[] nextLargerElement(int[] nums) {
    int n = nums.length;
    int[] res = new int[n];
    Arrays.fill(res, -1);
    Deque<Integer> s = new ArrayDeque<>();  // 这里放元素索引，而不是元素
    for (int i = 0; i < n; i++) {
        while (!s.isEmpty() && nums[s.peek()] < nums[i]) {
            int prev = s.pop();
            res[prev] = i;
        }
        s.push(i); 
    }
    return res;
}
```

## 二叉树
### 普通二叉树
```java
void dfs(TreeNode node) {
    if (node == null) {
        return; // 递归的终止条件
    }
    // 前序：
    doSomething();
        
    // 递归调用左子树
    dfs(node.left);
    // 递归调用右子树
    dfs(node.right);
        
    // 后序
    doSomething();
}
```
### 二叉搜索树
```java
void BST(TreeNode root, int target) {
    if (root.val == target)
        // 找到目标，做点什么
    if (root.val < target) 
        BST(root.right, target);
    if (root.val > target)
        BST(root.left, target);
}
```

## 排序
### 快排
```java
class Solution {
    public int[] sortArray(int[] nums) {
        quicksort(nums, 0, nums.length - 1);
        return nums;
    }

    public void quicksort(int[] nums, int l, int r) {
        if (l >= r)
            return;
        int pivot = partition(nums, l, r);
        quicksort(nums, l, pivot);
        quicksort(nums, pivot + 1, r);
    }

    int partition(int[] nums, int l, int r) {
        int pivot = nums[l+(r-l)/2];
        int i = l - 1, j = r + 1;
        while (i < j) {
            do i++; while (nums[i] < pivot);
            do j--; while (nums[j] > pivot);
            if (i < j) swap(nums, i, j);
        }
        return j;
    }

    void swap(int[] nums, int i, int j) {
        int temp = nums[i];
        nums[i] = nums[j];
        nums[j] = temp;
    }
}
```