# **贪心**
### [45.Jump Game II][1]
Given an array of non-negative integers, you are initially positioned at the first index of the array.

Each element in the array represents your maximum jump length at that position.

Your goal is to reach the last index in the minimum number of jumps.

```
Input: [2,3,1,1,4]
Output: 2
Explanation: The minimum number of jumps to reach the last index is 2.
    Jump 1 step from index 0 to 1, then 3 steps to the last index.
```

```
class Solution {
    public int jump(int[] nums) {
        if(nums.length <= 1){
            return 0;
        }
        int i = 0,max = 0,step = 0,index = 0;
        while(i < nums.length){
            if(i + nums[i] >= nums.length - 1){
                step++;
                return step;
            }
            max = 0;  //记录当前步数的最大值
            index = i + 1;   //记录当前的索引位置，每次都必须走一步
            //贪心：当前步数最优解（在这个位置下一步能够跳到最远）
            for(int j = i + 1 ; j< nums.length && j - i <= nums[i];j++){     
                if(max < nums[j] + j){
                    max = nums[j] + j;
                    index = j;
                }
            }
            i = index;
            step++;
        }
        return step;
    }
}
```
# **双指针**

### [19.Remove Nth Node From End of List][2]

Given a linked list, remove the n-th node from the end of list and return its head.

```
Given linked list: 1->2->3->4->5, and n = 2.

After removing the second node from the end, the linked list becomes 1->2->3->5.
```

```
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
class Solution {
    public ListNode removeNthFromEnd(ListNode head, int n) {
        ListNode res = new ListNode(-1);
        res.next = head;
        ListNode fast = res;
        ListNode slow = res;
        for(int i = 0;i<=n;i++){    //快指针比慢指针先移动n+1步,以保持两个指针两个n位
            fast = fast.next;
        }
        while(fast != null){        //两个指针同时移动知道快指针到达末尾，此时慢指针位置位于n+1，正好能够跳过下一节点的位置
            fast = fast.next;
            slow = slow.next;
        }
        slow.next = slow.next.next;
        return res.next;
        
    }
}
```

# **运算**
### **寻找只出现一个的数字**
在数组中，除了某个数字x之外，其他数字都出现了3次。
请给出最快的方法找出x，时间复杂度为O(n)，空间复杂度为O(1)。

//分析: 分别统计每一个数字32bit出现1的次数, 然后将所有数字对应bit的次数相加,
// 得到的次数对3取余,出现3次的数字都在对3取余的过程中抵消掉了,剩下的次数即为x各位出现1的次数.
//推广: 所有其他数字出现N(N>=2)次, 而一个数字出现1次都可以用这种解法来推导出这个出现1次的数字.

```
请输入代码
```

# **链表**
### [25. Reverse Nodes in k-Group][3]

Given a linked list, reverse the nodes of a linked list k at a time and return its modified list.

k is a positive integer and is less than or equal to the length of the linked list. If the number of nodes is not a multiple of k then left-out nodes in the end should remain as it is.

```
Given this linked list: 1->2->3->4->5

For k = 2, you should return: 2->1->4->3->5

For k = 3, you should return: 3->2->1->4->5
```

```
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
class Solution {
    public ListNode reverseKGroup(ListNode head, int k) {
        ListNode cur = head;
        int count = 0;
        while(cur != null && count != k){
            cur = cur.next;
            count++;
        }
        if(count == k){
            cur = reverseKGroup(cur,k);
            while(count-- != k){
                ListNode tmp = head.next;
                head.next = cur;
                cur = head;
                head = tmp;
            }
            head = cur;
        }
        return head;
    }
}
```
### [445.Add Two Number II][4]
You are given two **non-empty** linked lists representing two non-negative integers. The most significant digit comes first and each of their nodes contain a single digit. Add the two numbers and return it as a linked list.

You may assume the two numbers do not contain any leading zero, except the number 0 itself.

**Follow up:**
What if you cannot modify the input lists? In other words, reversing the lists is not allowed.

```
Input: (7 -> 2 -> 4 -> 3) + (5 -> 6 -> 4)
Output: 7 -> 8 -> 0 -> 7
```

```
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
class Solution {
    public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
        Stack<Integer> stack1 = new Stack<>();
        Stack<Integer> stack2 = new Stack<>();
        while(l1 != null){
            stack1.add(l1.val);
            l1 = l1.next;
        }
        while(l2 != null){
            stack2.add(l2.val);
            l2 = l2.next;
        }
        ListNode head = new ListNode(-1);
        int count = 0;
        while(!stack1.isEmpty() || !stack2.isEmpty() || count != 0){
            int x = stack1.isEmpty() ? 0 : stack1.pop();
            int y = stack2.isEmpty() ? 0 : stack2.pop();
            int sum = x + y + count;
            ListNode node = new ListNode(sum % 10);
            node.next = head.next;
            head.next = node;
            count = sum / 10;
        }
        return head.next;
    }
}
```
### [234.Palindrome Linked List][5]
Given a singly linked list, determine if it is a palindrome.

```
Input: 1->2
Output: false

Input: 1->2->2->1
Output: true
```

```
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
class Solution {
    public boolean isPalindrome(ListNode head) {
        if(head == null || head.next == null){
            return true;
        }
        ListNode fast = head;
        ListNode slow = head;
        while(fast != null && fast.next != null){
            fast = fast.next.next;
            slow = slow.next;
        }
        if(fast != null){           //偶数节点
            slow = slow.next;
        }
        cut(head,slow);
        
        return isEquals(head,revert(slow));
    }
    
    public void cut(ListNode head,ListNode cur){
        while(head.next != cur){
            head = head.next;
        }
        head.next = null;
    }
    
    public ListNode revert(ListNode node){
        ListNode pre = null;
        while(node != null){
            ListNode next = node.next;
            node.next = pre;
            pre = node;
            node = next;
        }
        return pre;
    }   
    public boolean isEquals(ListNode node1,ListNode node2){
        while(node1 != null && node2 != null){
            if(node1.val != node2.val){
                return false;
            }
            node1 = node1.next;
            node2 = node2.next;
        }
        return true;
    }
}
```
# **二叉树**
### [104. Maximum Depth of Binary Tree][6]
Given a binary tree, find its maximum depth.

The maximum depth is the number of nodes along the longest path from the root node down to the farthest leaf node.

Note: A leaf is a node with no children.

Example:

Given binary tree `[3,9,20,null,null,15,7],`

        3
       / \
      9  20
        /  \
       15   7

return its depth = 3.

```
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    public int maxDepth(TreeNode root) {
        if(root == null){
            return 0;
        }
        return Math.max(maxDepth(root.left),maxDepth(root.right)) + 1;
    }
}
```

### [110. Balanced Binary Tree][7]
Given a binary tree, determine if it is height-balanced.

For this problem, a height-balanced binary tree is defined as:

a binary tree in which the depth of the two subtrees of every node never differ by more than 1.
Example 1:

Given the following tree [3,9,20,null,null,15,7]:

        3
       / \
      9  20
        /  \
       15   7

Return true.

Example 2:

Given the following tree [1,2,2,3,3,null,null,4,4]:

           1
          / \
         2   2
        / \
       3   3
      / \
     4   4

Return false.

```
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    boolean result = true;
    public boolean isBalanced(TreeNode root) {
        getDepth(root);
        return result;
    }
    
    public int getDepth(TreeNode root){
        if(root == null){
            return 0;
        }
        int left = getDepth(root.left);
        int right = getDepth(root.right);
        if(Math.abs(left-right) > 1){
            result = false;
        }
        return Math.max(left,right)+1;
    }
}
```
### [543. Diameter of Binary Tree][8]
Given a binary tree, you need to compute the length of the diameter of the tree. The diameter of a binary tree is the length of the longest path between any two nodes in a tree. This path may or may not pass through the root.

Example:
Given a binary tree 

          1
         / \
        2   3
       / \     
      4   5    

Return 3, which is the length of the path [4,2,1,3] or [5,2,1,3].

Note: The length of path between two nodes is represented by the number of edges between them.

```
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    int max = 0;
    public int diameterOfBinaryTree(TreeNode root) {
        getDepth(root);
        return max;
    }
    
    public int getDepth(TreeNode root){
        if(root == null){
            return 0;
        }
        int left = getDepth(root.left);
        int right = getDepth(root.right);
        max = max > left + right ? max : left + right;
        return Math.max(left,right) + 1;
    }
}
```
### [226. Invert Binary Tree][9]

Invert a binary tree.

Example:

Input:

         4
       /   \
      2     7
     / \   / \
    1   3 6   9

Output:

         4
       /   \
      7     2
     / \   / \
    9   6 3   1

```
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    public TreeNode invertTree(TreeNode root) {
        if(root == null){
            return null;
        }
        TreeNode left = root.left;
        root.left = invertTree(root.right);
        root.right = invertTree(left);
        return root;
    }
}
```





### [236.Lowest Common Ancestor of a Binary Tree][10]
Given a binary tree, find the lowest common ancestor (LCA) of two given nodes in the tree.

According to the definition of LCA on Wikipedia: “The lowest common ancestor is defined between two nodes p and q as the lowest node in T that has both p and q as descendants (where we allow a node to be a descendant of itself).”

Given the following binary tree:  root = [3,5,1,6,2,0,8,null,null,7,4]

            _______3______
           /              \
        ___5__          ___1__
       /      \        /      \
       6      _2       0       8
             /  \
             7   4

Example 1:

    Input: root = [3,5,1,6,2,0,8,null,null,7,4], p = 5, q = 1
    Output: 3
    Explanation: The LCA of of nodes 5 and 1 is 3.
```
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
        if(root == null || root == p || root == q)
            return root;
        TreeNode left = lowestCommonAncestor(root.left,p,q);
        TreeNode right = lowestCommonAncestor(root.right,p,q);
        return left == null ? right : right == null ? left : root; 
    }
}
```

# **DFS**
### [494.Target Sum][11]
You are given a list of non-negative integers, a1, a2, ..., an, and a target, S. Now you have 2 symbols + and -. For each integer, you should choose one from + and - as its new symbol.

Find out how many ways to assign symbols to make sum of integers equal to target S.

```
Input: nums is [1, 1, 1, 1, 1], S is 3. 
Output: 5
Explanation: 

-1+1+1+1+1 = 3
+1-1+1+1+1 = 3
+1+1-1+1+1 = 3
+1+1+1-1+1 = 3
+1+1+1+1-1 = 3

There are 5 ways to assign symbols to make the sum of nums be target 3.
```

```
class Solution {
    public int findTargetSumWays(int[] nums, int S) {
        return findTargetSumWays(nums,0,S);
    }
    
    public int findTargetSumWays(int[] nums,int start,int S){
        if(start == nums.length){
            return S == 0 ? 1 : 0;
        }
        return findTargetSumWays(nums,start+1,S+nums[start]) 
            + findTargetSumWays(nums,start+1,S-nums[start]);
    }
}        
```
该题另一解法01背包

# **回溯**
### [77.Combinations][12]
Given two integers n and k, return all possible combinations of k numbers out of 1 ... n.

```
Input: n = 4, k = 2
Output:
[
  [2,4],
  [3,4],
  [2,3],
  [1,2],
  [1,3],
  [1,4],
]
```

```
class Solution {
    public List<List<Integer>> combine(int n, int k) {
        List<List<Integer>> res = new ArrayList<>();
        List<Integer> list = new ArrayList<>();
        helper(n,k,1,res,list);
        return res;
    }
    
    public void helper(int n,int k,int start,List<List<Integer>> res,List<Integer> list){
        if(k == 0){
            res.add(new ArrayList<>(list));
            return;
        }
        for(int i = start;i<=n-k+1;i++){
            list.add(i);
            helper(n,k-1,i+1,res,list);
            list.remove(list.size()-1);
        }
    }
}
```

### [37.sudoku Solver][13]
A sudoku solution must satisfy all of the following rules:

Each of the digits 1-9 must occur exactly once in each row.
Each of the digits 1-9 must occur exactly once in each column.
Each of the the digits 1-9 must occur exactly once in each of the 9 3x3 sub-boxes of the grid.
Empty cells are indicated by the character '.'.

![clipboard.png](/img/bVNaNS)


A sudoku puzzle...

![clipboard.png](/img/bVQIGP)


...and its solution numbers marked in red.

```
class Solution {
    public void solveSudoku(char[][] board) {
        helper(board,0,0);
    }
    
    public boolean helper(char[][] board,int row,int col){
        if(col == board[0].length){
            row++;
            col = 0;
        }
        if(row == board.length){
            return true;
        }
        if(board[row][col] != '.'){
            return helper(board,row,col+1);
        }
        for(char ch = '1';ch <= '9';ch++){
            if(invlid(board,row,col,ch)){
                board[row][col] = ch;
                if(helper(board,row,col+1)){
                    return true;
                }
                board[row][col] = '.';
            }
        }
        return false;
    }
    
    public boolean invlid(char[][] board,int row,int col,char ch){
        for(int i = 0;i<9;i++){
            if(board[row][i] == ch || board[i][col] == ch || board[row/3*3+i/3][col/3*3+i%3] == ch){
                return false;
            }
        }
        return true;
    }
}
```

### [51.N-Queens][14]
The n-queens puzzle is the problem of placing n queens on an n×n chessboard such that no two queens attack each other.



Given an integer n, return all distinct solutions to the n-queens puzzle.

Each solution contains a distinct board configuration of the n-queens' placement, where 'Q' and '.' both indicate a queen and an empty space respectively.

```
Input: 4
Output: [
 [".Q..",  // Solution 1
  "...Q",
  "Q...",
  "..Q."],

 ["..Q.",  // Solution 2
  "Q...",
  "...Q",
  ".Q.."]
]
Explanation: There exist two distinct solutions to the 4-queens puzzle as shown above.
```

```
class Solution {
    public List<List<String>> solveNQueens(int n) {
        boolean[] column = new boolean[n];
        boolean[] leftDiagonal = new boolean[2*n];
        boolean[] rightDiagonal = new boolean[2*n];
        List<List<String>> result = new ArrayList<>();
        helper(result,new ArrayList<>(),column,leftDiagonal,rightDiagonal,0);
        return result;
    }
    
    public void helper(List<List<String>> result,List<String> current,boolean[] column,
                       boolean[] leftDiagonal,boolean[] rightDiagonal,int rowIndex){
        int length = column.length;
        if(rowIndex == length){
            result.add(new ArrayList<>(current));
            return;
        }
        
        for(int i = 0;i<length;i++){
            if(column[i] || leftDiagonal[rowIndex+i] || rightDiagonal[rowIndex-i+length-1]){
                continue;
            }
            char[] ch = new char[length];
            Arrays.fill(ch,'.');
            ch[i] = 'Q';
            current.add(new String(ch));
            column[i] = true;
            leftDiagonal[rowIndex+i] = true;
            rightDiagonal[rowIndex-i+length-1] = true;
            helper(result,current,column,leftDiagonal,rightDiagonal,rowIndex+1);
            current.remove(current.size()-1);
            ch[i] = '.';
            column[i] = false;
            leftDiagonal[rowIndex+i] = false;
            rightDiagonal[rowIndex-i+length-1] = false;
        }
    }
}
```

# **动态规划**
### [198.House Robber][15]


You are a professional robber planning to rob houses along a street. Each house has a certain amount of money stashed, the only constraint stopping you from robbing each of them is that adjacent houses have security system connected and it will automatically contact the police if two adjacent houses were broken into on the same night.

Given a list of non-negative integers representing the amount of money of each house, determine the maximum amount of money you can rob tonight without alerting the police.
```
Input: [1,2,3,1]
Output: 4
Explanation: Rob house 1 (money = 1) and then rob house 3 (money = 3).
             Total amount you can rob = 1 + 3 = 4.
```           

```
class Solution {
    public int rob(int[] nums) {        //f(n) =max(f(n-1),f(n-2)+n)
        int cur1 = 0,cur2 = 0;
        for(int i = 0;i<nums.length;i++){
            int tmp = Math.max(cur2,cur1 + nums[i]);
            cur1 = cur2;
            cur2 = tmp;
        }
        return cur2;
    }
}
```
### [213.House Robber II][16]
You are a professional robber planning to rob houses along a street. Each house has a certain amount of money stashed, the only constraint stopping you from robbing each of them is that adjacent houses have security system connected and it will automatically contact the police if two adjacent houses were broken into on the same night.

Given a list of non-negative integers representing the amount of money of each house, determine the maximum amount of money you can rob tonight without alerting the police.

```
Input: [1,2,3,1]
Output: 4
Explanation: Rob house 1 (money = 1) and then rob house 3 (money = 3).
             Total amount you can rob = 1 + 3 = 4.
```

```
class Solution {
    public int rob(int[] nums) {
        if(nums == null || nums.length == 0){
            return 0;
        }
        if(nums.length == 1){
            return nums[0];
        }
        return Math.max(rob(nums,0,nums.length -2),rob(nums,1,nums.length - 1));
    }
    public int rob(int[] nums ,int start , int end){
        int per1 = 0, per2 = 0, per3 = 0;
        for(int i = start;i<=end;i++){
            int cur = Math.max(per1,per2) + nums[i];
            per1 = per2;
            per2 = per3;
            per3 = cur;
        }
        return Math.max(per2,per3);
    }
}
```

### [64.Minimum Path Sum][17]
Given a m x n grid filled with non-negative numbers, find a path from top left to bottom right which minimizes the sum of all numbers along its path.

Note: You can only move either down or right at any point in time.

```
Input:
[
  [1,3,1],
  [1,5,1],
  [4,2,1]
]
Output: 7
Explanation: Because the path 1→3→1→1→1 minimizes the sum.
```

```
class Solution {
    public int minPathSum(int[][] grid) {
        int m = grid.length,n = grid[0].length;
        if(m == 0 || n == 0){
            return -1;
        }
        int[] dp = new int[n];
        
        for(int i = 0;i<m;i++){
            for(int j = 0;j<n;j++){
                if(j == 0){
                    dp[j] = dp[j] + grid[i][j];
                }else if(i == 0){
                    dp[j] = dp[j-1] + grid[i][j];
                }else{
                    dp[j] = Math.min(dp[j-1],dp[j]) + grid[i][j];
                }
            }
        }
        return dp[n-1];
    }
}   
```
### [413.Arithmetic Slices][18]


```
class Solution {
    public int numberOfArithmeticSlices(int[] A) {
        if(A == null || A.length == 0){
            return 0;
        }
        int[] dp = new int[A.length];
        for(int i = 2;i<A.length;i++){
            if(A[i] - A[i-1] == A[i-1] - A[i-2]){
                dp[i] = dp[i-1] + 1;
            }
        }
        int res = 0;        
        for(int i : dp){
            res += i;   
        }
        return res;
    }
}
```

### [343.Integer break][19]

Given a positive integer n, break it into the sum of at least two positive integers and maximize the product of those integers. Return the maximum product you can get.

```
Input: 2
Output: 1
Explanation: 2 = 1 + 1, 1 × 1 = 1.
```

```
class Solution {
    public int integerBreak(int n) {
        int[] dp = new int[n+1];
        for(int i = 2;i<=n;i++){
            for(int j = 1;j<=i-1;j++){
                dp[i] = Math.max(dp[i],Math.max(j*(i-j),j*dp[i-j]));
            }
        }
        return dp[n];
    }
}
                                
```
### [300.Longest Increasing Subsequeue][20]
Given an unsorted array of integers, find the length of longest increasing subsequence.

```
Input: [10,9,2,5,3,7,101,18]
Output: 4 
Explanation: The longest increasing subsequence is [2,3,7,101], therefore the length is 4. 
```

```
class Solution {
    public int lengthOfLIS(int[] nums) {
        
        int n = nums.length;
        int[] dp = new int[n];
        for(int i = 0;i<n;i++){
            int max = 1;
            for(int j = 0;j<i;j++){
                if(nums[i] > nums[j])
                    max = Math.max(max,dp[j]+1);
            }
            dp[i] = max;
        }

       // return Arrays.stream(dp).max().orElse(0);
        
        int res = 0;
        for(int d : dp){
            res = res > d ? res : d;
        }
        return res;
    }
}
```

### [583.Delete Operation For Two Strings][21]
 Given two words word1 and word2, find the minimum number of steps required to make word1 and word2 the same, where in each step you can delete one character in either string.
```
Input: "sea", "eat"
Output: 2
Explanation: You need one step to make "sea" to "ea" and another step to make "eat" to "ea".
```

```
//转换成求公共字符串问题
class Solution {
    public int minDistance(String word1, String word2) {
        
        int n = word1.length();
        int m = word2.length();
        int[][] dp = new int[n+1][m+1];
        for(int i = 1;i<=n;i++){
            for(int j = 1;j<=m;j++){
                if(word1.charAt(i-1) == word2.charAt(j-1)){
                    dp[i][j] = dp[i-1][j-1] + 1;
                }else{
                    dp[i][j] = Math.max(dp[i-1][j],dp[i][j-1]);
                }
            }
        }
        return n + m - dp[n][m] * 2;
    }
}
```


# **背包**
###[416.Partition Equal Subset Sum][22]
Given a non-empty array containing only positive integers, find if the array can be partitioned into two subsets such that the sum of elements in both subsets is equal.

Note:
Each of the array element will not exceed 100.
The array size will not exceed 200.

```
Input: [1, 5, 11, 5]

Output: true

Explanation: The array can be partitioned as [1, 5, 5] and [11].
```

```
//0-1背包
class solution{
    public boolean canPartition(int[] nums){
        int sum = 0;
        for(int num : nums){
            sum += num;
        }
        if(sum % 2 == 1){
            return false;
        }
        sum /= 2;
        int n = nums.length;
        boolean[][] dp = new boolean[n+1][sum+1];
        for(int i = 0;i<n+1;i++){
            dp[i][0] = true;
        }
        for(int i = 1;i<n+1;i++){
            for(int j = 1;j<sum+1;j++){
                if(j <= nums[i-1]){
                    dp[i][j] = dp[i-1][j] || dp[i-1][j-nums[i-1]];
                }
            }
        }
        return dp[n][sum];
    }
}
```

```
class Solution {
    public boolean canPartition(int[] nums) {
        int sum = 0;
        for(int num : nums){
            sum += num;
        }
        if(sum % 2 == 1){
            return false;
        }
        sum /= 2;
        boolean[] dp = new boolean[sum+1];
        dp[0] = true;
        for(int num : nums){            //每个背包只能放一次
            for(int i = sum;i>0;i--){   //从后往前，先计算do[i]在计算dp[i-1]   // for(int i = sum ;i>=num;i--)
                if(i >= num){
                    dp[i] = dp[i] || dp[i-num];
                }
            }
        }
        return dp[sum];
    }
}
```
### [494.Target Sum][23]

You are given a list of non-negative integers, a1, a2, ..., an, and a target, S. Now you have 2 symbols + and -. For each integer, you should choose one from + and - as its new symbol.

Find out how many ways to assign symbols to make sum of integers equal to target S.

```
Input: nums is [1, 1, 1, 1, 1], S is 3. 
Output: 5
Explanation: 

-1+1+1+1+1 = 3
+1-1+1+1+1 = 3
+1+1-1+1+1 = 3
+1+1+1-1+1 = 3
+1+1+1+1-1 = 3

There are 5 ways to assign symbols to make the sum of nums be target 3.
```



```
//0-1背包
class Solution {
    public int findTargetSumWays(int[] nums, int S) {
        //正数 + 正数 = 目标数 + 负数 + 正数
        //正数 = (目标数 + 总数)/2
        int sum = 0;
        for(int num : nums){
            sum += num;
        }
        if(sum < S || (sum + S) % 2 == 1){
            return 0;
        }
        int W = (sum + S) / 2;
        
        int[] dp = new int[W+1];
        dp[0] = 1;
        for(int num : nums){
            for(int i = W;i>=num;i--){
                dp[i] = dp[i] + dp[i-num];
            }
        }
        return dp[W];
    }
}
```
### [139.Word Break][24]
Given a non-empty string s and a dictionary wordDict containing a list of non-empty words, determine if s can be segmented into a space-separated sequence of one or more dictionary words.

Note:

The same word in the dictionary may be reused multiple times in the segmentation.
You may assume the dictionary does not contain duplicate words.

```
Input: s = "leetcode", wordDict = ["leet", "code"]
Output: true
Explanation: Return true because "leetcode" can be segmented as "leet code".
```

```
 //多重背包
class Solution {
    public boolean wordBreak(String s, List<String> wordDict) {
       
        int n = s.length();
        boolean[] dp = new boolean[n+1];
        dp[0] = true;
        for(int i = 1;i<=n;i++){
            for(String word : wordDict){
                int len = word.length();
                if(i >= len && word.equals(s.substring(i-len,i))){
                    dp[i] = dp[i] || dp[i-len];
                }
            }
        }
        return dp[n];
    }
}
```
### [437.Ones and Zeros][25]
In the computer world, use restricted resource you have to generate maximum benefit is what we always want to pursue.

For now, suppose you are a dominator of m 0s and n 1s respectively. On the other hand, there is an array with strings consisting of only 0s and 1s.

Now your task is to find the maximum number of strings that you can form with given m 0s and n 1s. Each 0 and 1 can be used at most once.

Note:
The given numbers of 0s and 1s will both not exceed 100
The size of given string array won't exceed 600.

```
Input: Array = {"10", "0001", "111001", "1", "0"}, m = 5, n = 3
Output: 4

Explanation: This are totally 4 strings can be formed by the using of 5 0s and 3 1s, which are “10,”0001”,”1”,”0”
```

```
//双维度0-1背包
class Solution {
    public int findMaxForm(String[] strs, int m, int n) {
        int[][] dp = new int[m+1][n+1];
    //    dp[0][0] = 1;
        for(String str : strs){
            int zero = 0;
            int one = 0;
            for(int i = 0;i<str.length();i++){
                if(str.charAt(i) == '0'){
                    zero++;
                }else{
                    one++;
                }
            }
            for(int i = m;i>=zero;i--){
                for(int j = n;j>=one;j--){
                    dp[i][j] = Math.max(dp[i][j] , dp[i-zero][j-one]+1);
                }
            }
        }
        return dp[m][n];
        
    }
}
```

### [322.Coin Change][26]


You are given coins of different denominations and a total amount of money amount. Write a function to compute the fewest number of coins that you need to make up that amount. If that amount of money cannot be made up by any combination of the coins, return -1.


```
Input: coins = [1, 2, 5], amount = 11
Output: 3 
Explanation: 11 = 5 + 5 + 1
```

```
//完全背包
class Solution {
    public int coinChange(int[] coins, int amount) {
        
        
        int[] dp = new int[amount+1];
        Arrays.fill(dp,amount+1);
        
        dp[0] = 0;
        for(int i = 0;i<amount+1;i++){
            for(int coin : coins){
                if(coin <= i)
                    dp[i] = Math.min(dp[i],dp[i-coin]+1);
            }
        }
        return dp[amount] > amount ? -1 : dp[amount];
    }
}
```
### [377.Combination Sum IV][27]
Given an integer array with all positive numbers and no duplicates, find the number of possible combinations that add up to a positive integer target.

```
nums = [1, 2, 3]
target = 4

The possible combination ways are:
(1, 1, 1, 1)
(1, 1, 2)
(1, 2, 1)
(1, 3)
(2, 1, 1)
(2, 2)
(3, 1)

Note that different sequences are counted as different combinations.

Therefore the output is 7.
```

```
//完全背包
class Solution {
    public int combinationSum4(int[] nums, int target) {
        
        int[] dp = new int[target+1];
        dp[0] = 1;
        for(int i = 1;i<=target;i++){
            for(int num : nums){
                if(i >= num){
                    dp[i] = dp[i] + dp[i-num];
                }
            }
        }
        return dp[target];
    }
}
```

# **lRU缓存算法**
## [146. LRU Cache][28]
You are given a list of non-negative integers, a1, a2, ..., an, and a target, S. Now you have 2 symbols + and -. For each integer, you should choose one from + and - as its new symbol.

Find out how many ways to assign symbols to make sum of integers equal to target S.

```
Input: nums is [1, 1, 1, 1, 1], S is 3. 
Output: 5
Explanation: 

-1+1+1+1+1 = 3
+1-1+1+1+1 = 3
+1+1-1+1+1 = 3
+1+1+1-1+1 = 3
+1+1+1+1-1 = 3

There are 5 ways to assign symbols to make the sum of nums be target 3.
```

```
class LRUCache {
    private class Node{
		int key;
		int value;
		Node pre;
		Node next;
	}	
	private HashMap<Integer, Node> cache = new HashMap<>();
	int count;
	int capacity;
	private Node head,tail;
	public void addNode(Node node){
		node.pre = head;
		node.next = head.next;
		
		head.next.pre = node;
		head.next = node;
	}
	
	public void removeNode(Node node){
		Node pre = node.pre;
		Node next = node.next;
		
		pre.next = next;
		next.pre = pre;
	}
	
	public void removeToHead(Node node){
		this.removeNode(node);
		this.addNode(node);
	}
	
	public Node popTail(){
		Node res = tail.pre;
		this.removeNode(res);
		return res;
	}
	
	public LRUCache(int capacity) {
		this.count = 0;
		this.capacity = capacity;
		
		head = new Node();
		head.pre = null;
		
		tail = new Node();
		tail.next = null;
        
        head.next = tail;
        tail.pre = head;
	}
	
	public int get(int key){
		Node node = cache.get(key);
		if(node == null){
			return -1;
		}
		this.removeToHead(node);
		return node.value;
	}
	
	public void put(int key,int value){
		Node node = cache.get(key);
		if(node == null){
			node = new Node();
			node.key = key;
			node.value = value;
			this.cache.put(key, node);
			this.addNode(node);
			++count;
			if(count > capacity){
				Node popTail = this.popTail();
				cache.remove(popTail.key);
				--count;
			}
		}else{
			node.value = value;
			removeToHead(node);
		}
	}
}


/**
 * Your LRUCache object will be instantiated and called as such:
 * LRUCache obj = new LRUCache(capacity);
 * int param_1 = obj.get(key);
 * obj.put(key,value);
 */

```


  [1]: https://leetcode.com/problems/jump-game-ii/description/
  [2]: https://leetcode.com/problems/remove-nth-node-from-end-of-list/description/
  [3]: https://leetcode.com/problems/reverse-nodes-in-k-group/description/
  [4]: https://leetcode.com/problems/add-two-numbers-ii/description/
  [5]: https://leetcode.com/problems/palindrome-linked-list/description/
  [6]: https://leetcode.com/problems/maximum-depth-of-binary-tree/description/
  [7]: https://leetcode.com/problems/balanced-binary-tree/description/
  [8]: https://leetcode.com/problems/diameter-of-binary-tree/description/
  [9]: https://leetcode.com/problems/invert-binary-tree/description/
  [10]: https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-tree/description/
  [11]: https://leetcode.com/problems/target-sum/description/
  [12]: https://leetcode.com/problems/combinations/description/
  [13]: https://leetcode.com/problems/sudoku-solver/description/
  [14]: https://leetcode.com/problems/n-queens/description/
  [15]: https://leetcode.com/problems/house-robber/description/
  [16]: https://leetcode.com/problems/house-robber-ii/description/
  [17]: https://leetcode.com/problems/minimum-path-sum/description/
  [18]: https://leetcode.com/problems/minimum-path-sum/description/
  [19]: https://leetcode.com/problems/integer-break/description/
  [20]: https://leetcode.com/problems/longest-increasing-subsequence/description/
  [21]: https://leetcode.com/problems/delete-operation-for-two-strings/description/
  [22]: https://leetcode.com/problems/partition-equal-subset-sum/description/
  [23]: https://leetcode.com/problems/target-sum/description/
  [24]: https://leetcode.com/problems/word-break/description/
  [25]: https://leetcode.com/problems/ones-and-zeroes/description/
  [26]: https://leetcode.com/problems/coin-change/description/
  [27]: https://leetcode.com/problems/combination-sum-iv/description/
  [28]: https://leetcode.com/problems/lru-cache/description/
