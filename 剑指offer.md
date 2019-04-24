# **机器人的运动范围**
地上有一个m行和n列的方格。一个机器人从坐标0,0的格子开始移动，每一次只能向左，右，上，下四个方向移动一格，但是不能进入行坐标和列坐标的数位之和大于k的格子。 例如，当k为18时，机器人能够进入方格（35,37），因为3+5+3+7 = 18。但是，它不能进入方格（35,38），因为3+5+3+8 = 19。请问该机器人能够达到多少个格子？

##### **链接：**[牛客网][1]

##### **思路：**

- 回溯法：
- 从（0,0）开始，没走成功一步标记当前位置为true,然后向当前四个方向探索，返回1+4个方向的探索值。
- 探索时判断当前节点是否可达时的标准被：
    - 当前节点在矩阵内
    - 当前节点未被访问过
    - 当前节点满足limit限制

##### **代码：**
```
public class Solution {
    public int movingCount(int threshold, int rows, int cols)
    {
        boolean[][] flag = new boolean[rows][cols];
        return move(threshold,rows,cols,0,0,flag);
    }
    
    public int move(int threshold,int rows,int cols,int r,int c,boolean[][] flag){
        
        if(r < 0 || r >= rows || c < 0 || c >= cols || flag[r][c] || numSum(r) + numSum(c) > threshold){
            return 0;
        }
        flag[r][c] = true;
        return move(threshold,rows,cols,r-1,c,flag) 
            + move(threshold,rows,cols,r+1,c,flag)
            + move(threshold,rows,cols,r,c-1,flag)
            + move(threshold,rows,cols,r,c+1,flag)
            + 1;
    }
    
    public int numSum(int x){
        int c = 0;
        while(x / 10 != 0){
            c += x % 10;
            x = x / 10;
        }
        return c + x;
    }
}
```

# **矩阵中的路径**
请设计一个函数，用来判断在一个矩阵中是否存在一条包含某字符串所有字符的路径。路径可以从矩阵中的任意一个格子开始，每一步可以在矩阵中向左，向右，向上，向下移动一个格子。如果一条路径经过了矩阵中的某一个格子，则之后不能再次进入这个格子。 例如 a b c e s f c s a d e e 这样的3 X 4 矩阵中包含一条字符串"bcced"的路径，但是矩阵中不包含"abcb"路径，因为字符串的第一个字符b占据了矩阵中的第一行第二个格子之后，路径不能再次进入该格子。

##### **链接：**[牛客网][2]

##### **思路：**
- 思路同上题差不多，修改一个条件：matrix[cur] != str[index]
##### **代码：**

```
public class Solution {
    public boolean hasPath(char[] matrix, int rows, int cols, char[] str)
    {
        for(int i = 0;i < rows;i++){
            for(int j = 0;j < cols;j++){
                boolean[] flag = new boolean[rows * cols];
                if(helper(matrix,rows,cols,i,j,str,0,flag))
                    return true;
            }
        }
        return false;
    }

    public boolean helper(char[] matrix,int rows,int cols,int r,int c,char[] str,int index,boolean[] flag){
        int cur = r * cols + c;
        if(r < 0 || r >= rows || c < 0 || c >= cols || flag[cur] || matrix[cur] != str[index]){
            return false;
        }
        flag[cur] = true;
        if(index == str.length - 1){
            return true;
        }
        index++;
        if(helper(matrix,rows,cols,r-1,c,str,index,flag)
            || helper(matrix,rows,cols,r+1,c,str,index,flag)
            || helper(matrix,rows,cols,r,c-1,str,index,flag)
            || helper(matrix,rows,cols,r,c+1,str,index,flag)){
            return true;
        }
        flag[cur] = false;
        return false;
        
    }

}
```

# **滑动窗口的最大值**
给定一个数组和滑动窗口的大小，找出所有滑动窗口里数值的最大值。例如，如果输入数组{2,3,4,2,6,2,5,1}及滑动窗口的大小3，那么一共存在6个滑动窗口，他们的最大值分别为{4,4,6,6,6,5}； 针对数组{2,3,4,2,6,2,5,1}的滑动窗口有以下6个： {[2,3,4],2,6,2,5,1}， {2,[3,4,2],6,2,5,1}， {2,3,[4,2,6],2,5,1}， {2,3,4,[2,6,2],5,1}， {2,3,4,2,[6,2,5],1}， {2,3,4,2,6,[2,5,1]}。

#### **链接：**[牛客网][3]

#### **思路：**
- 通过维护一个双端队列，队列的第一个值保存最大值，当每次窗口滑动：
    - 1、判断队列第一个值是否失效。
    - 2、从队尾开始比较，移出队列所有比当前值小的值。
#### **代码：**

```
import java.util.*;
public class Solution {
    public ArrayList<Integer> maxInWindows(int [] num, int size)
    {
        ArrayList<Integer> list = new ArrayList<>();
        if(size == 0){
            return list;
        }
        ArrayDeque<Integer> deque = new ArrayDeque<>();
        int begin = 0;
        for(int i = 0;i<num.length;i++){
            begin = i - size + 1;
            if(deque.isEmpty()){
                deque.add(i);
            }
            else if(begin > deque.peekFirst()){
                deque.pollFirst();
            }
            while((!deque.isEmpty()) && num[deque.peekLast()] <= num[i]){
                deque.pollLast();
            }
            deque.add(i);
            if(begin >= 0){
                list.add(num[deque.peekFirst()]);
            }
        }
        return list;
    }
}
```

# **数据流中的中位数**
如何得到一个数据流中的中位数？如果从数据流中读出奇数个数值，那么中位数就是所有数值排序之后位于中间的数值。如果从数据流中读出偶数个数值，那么中位数就是所有数值排序之后中间两个数的平均值。我们使用Insert()方法读取数据流，使用GetMedian()方法获取当前读取数据的中位数。

**链接：**[牛客网][4]

**思路：**
- 方法一：通过二分查找，维护一个递增集合List
- 方法二：维护两个堆，大根堆放大的那部分数，小根堆放小的那部分数据。

**代码：**

```
import java.util.ArrayList;
public class Solution {
 
    ArrayList<Integer> list = new ArrayList<Integer>();
    int size = 0;
    public void Insert(Integer num) {
        int lo = 0,ro = list.size()-1,mid = 0;
        while(lo <= ro){
            mid = (lo + ro) / 2;
            if(num < list.get(mid)){
                ro = mid - 1;
            }else if(num > list.get(mid)){
                lo = mid +1;
            }
        }
        list.add(mid,num);
        size++;
    }
 
    public Double GetMedian() {
        int cur = size / 2;
        if(size % 2 == 0){
            return (new Double(list.get(cur)) + new Double(list.get(cur-1))) / 2;
        }else{
            return new Double(list.get(cur));
        }
    }
}
```
```
import java.util.PriorityQueue;
import java.util.Comparator;
public class Solution {

    int count = 0;
    private PriorityQueue<Integer> minHeap = new PriorityQueue<>();
    private PriorityQueue<Integer> maxHeap = new PriorityQueue<>(15,new Comparator<Integer>(){
        public int compare(Integer o1,Integer o2){
            return o2 - o1;
        }
    });
    public void Insert(Integer num) {
        if(count % 2 == 0){
            maxHeap.offer(num);
            int cur = maxHeap.poll();
            minHeap.offer(cur);
        }else{
            minHeap.offer(num);
            int cur = minHeap.poll();
            maxHeap.offer(cur);
        }
        count++;
    }

    public Double GetMedian() {
        if(count % 2 == 0){
            return new Double(minHeap.peek() + maxHeap.peek()) /2 ;
        }else{
            return new Double(minHeap.peek());
        }
    }
}
```

# **二叉搜索树的第k个节点**
给定一棵二叉搜索树，请找出其中的第k小的结点。例如， （5，3，7，2，4，6，8）    中，按结点数值大小顺序第三小结点的值为4。

**链接：**[牛客网][5]

**思路：**

- 利用二叉搜索树的中序遍历得到从小到大的顺序，得到第k个值。

**代码：**

```
/*
public class TreeNode {
    int val = 0;
    TreeNode left = null;
    TreeNode right = null;

    public TreeNode(int val) {
        this.val = val;

    }

}
*/
public class Solution {
    int count = 0;
    TreeNode KthNode(TreeNode pRoot, int k)
    {
        if(pRoot != null){
            TreeNode node = KthNode(pRoot.left,k);
            if(node != null){
                return node;
            }
            if(++count == k){
                return pRoot;
            }
            node = KthNode(pRoot.right,k);
            if(node != null){
                return node;
            }
        }
        return null;
    }


}
```

# **序列化二叉树**
请实现两个函数，分别用来序列化和反序列化二叉树

**链接：**[牛客网][6]

**思路：**
- 对于序列化：使用前序遍历，递归的将二叉树的值转化为字符，并且在每次二叉树的结点不为空时，在转化val所得的字符之后添加一个' , '作为分割。对于空节点则以 '#' 代替。
- 对于反序列化：按照前序顺序，递归的使用字符串中的字符创建一个二叉树.
**代码：**

```
/*
public class TreeNode {
    int val = 0;
    TreeNode left = null;
    TreeNode right = null;

    public TreeNode(int val) {
        this.val = val;

    }

}
*/
public class Solution {
    int index = -1;
    String Serialize(TreeNode root) {
        StringBuilder sb = new StringBuilder();
        if(root == null){
            sb.append("#,");
            return sb.toString();
        }
        sb.append(root.val).append(",");
        sb.append(Serialize(root.left));
        sb.append(Serialize(root.right));
        return sb.toString();
  }
    TreeNode Deserialize(String str) {
       index++;
       if(index == str.length()){
           return null;
       }
       String[] strr = str.split(",");
        TreeNode node = null;
       if(!strr[index].equals("#")){
           node = new TreeNode(Integer.parseInt(strr[index]));
           node.left = Deserialize(str);
           node.right = Deserialize(str);
       }
       return node;
  }
}
```
# **把二叉树打印成多行**
从上到下按层打印二叉树，同一层结点从左至右输出。每一层输出一行。

**链接：**[牛客网][7]

**思路：**
- 按层次输出二叉树
- 访问根节点，并将根节点入队。
- 当队列不空的时候，重复以下操作。
    - 1、弹出一个元素。作为当前的根节点。
    - 2、如果根节点有左孩子，访问左孩子，并将左孩子入队。
    - 3、如果根节点有右孩子，访问右孩子，并将右孩子入队。
**代码：**

```
import java.util.ArrayList;
import java.util.Queue;
import java.util.LinkedList;


/*
public class TreeNode {
    int val = 0;
    TreeNode left = null;
    TreeNode right = null;

    public TreeNode(int val) {
        this.val = val;

    }

}
*/
public class Solution {
    ArrayList<ArrayList<Integer> > Print(TreeNode pRoot) {
        ArrayList<ArrayList<Integer>> result = new ArrayList<>();
        if(pRoot == null){
            return result;
        }
        ArrayList<Integer> list = new ArrayList<>();
        Queue<TreeNode> queue = new LinkedList<>();
        int start = 0,end = 1;
        queue.add(pRoot);
        while(!queue.isEmpty()){
            start++;
            TreeNode node = queue.remove();
            list.add(node.val);
            if(node.left != null){
                queue.add(node.left);
            }
            if(node.right != null){
                queue.add(node.right);
            }
            if(start == end){
                start = 0;
                end = queue.size();
                result.add(list);
                list = new ArrayList<>();
            }
        }
        return result;
    }
    
}
```
# **按之字形顺序打印二叉树**
请实现一个函数按照之字形打印二叉树，即第一行按照从左到右的顺序打印，第二层按照从右至左的顺序打印，第三行按照从左到右的顺序打印，其他行以此类推。

**链接：**[牛客网][8]

**思路：**
- 思路同上题差不多。
**代码：**

```
import java.util.ArrayList;
import java.util.Stack;
/*
public class TreeNode {
    int val = 0;
    TreeNode left = null;
    TreeNode right = null;

    public TreeNode(int val) {
        this.val = val;

    }

}
*/
public class Solution {
    public ArrayList<ArrayList<Integer> > Print(TreeNode pRoot) {
        ArrayList<ArrayList<Integer>> res = new ArrayList<>();
        if(pRoot == null){
            return res;
        }
        Stack<TreeNode> stack1 = new Stack<>();
        Stack<TreeNode> stack2 = new Stack<>();
        ArrayList<Integer> list = new ArrayList<>();
        stack1.add(pRoot);
        int start = 0,end = stack1.size(),size = 1;
        while(!stack1.isEmpty() || !stack2.isEmpty()){
            if(size % 2 == 1){
                TreeNode node = stack1.pop();
                list.add(node.val);
                if(node.left != null){
                    stack2.add(node.left);
                }
                if(node.right != null){
                    stack2.add(node.right);
                }
                start++;
                if(start == end){
                    start = 0;
                    end = stack2.size();
                    size++;
                    res.add(list);
                    list = new ArrayList<>();
                }
            }else{
                TreeNode node = stack2.pop();
                list.add(node.val);
                if(node.right != null){
                    stack1.add(node.right);
                }
                if(node.left != null){
                    stack1.add(node.left);
                }
                start++;
                if(start == end){
                    start = 0;
                    end = stack1.size();
                    size++;
                    res.add(list);
                    list = new ArrayList<>();
                }
            }
        }
        return res;
        
    }

}
```

# **对称的二叉树**
请实现一个函数，用来判断一颗二叉树是不是对称的。注意，如果一个二叉树同此二叉树的镜像是同样的，定义其为对称的。

**链接：**[牛客网][9]

**思路：**

- 首先判断左孩子是否等于右孩子
- 接着递归判断左孩子的**左右**孩子是否与右孩子的**右左**孩子对应相等。

**代码：**

```
/*
public class TreeNode {
    int val = 0;
    TreeNode left = null;
    TreeNode right = null;

    public TreeNode(int val) {
        this.val = val;

    }

}
*/
public class Solution {
    boolean isSymmetrical(TreeNode pRoot)
    {
        if(pRoot == null){
            return true;
        }
        return helper(pRoot.left,pRoot.right);
    }
   
    boolean helper(TreeNode node1,TreeNode node2){
        if(node1 == null)
            return node2 == null;
        if(node2 == null || node1.val != node2.val)
            return false;
        return helper(node1.left,node2.right) && helper(node1.right,node2.left);
    }
}
```

# **二叉树的下一个节点**
给定一个二叉树和其中的一个结点，请找出中序遍历顺序的下一个结点并且返回。注意，树中的结点不仅包含左右子结点，同时包含指向父结点的指针。

**链接：**[牛客网][11]

**思路：**

- 节点为空，则放回空。
- 如果二叉树的右孩子不为空，则设置一指针从右孩子出发，一直沿着左孩子节点指针找到叶子节点，返回。
- 节点不是根节点，如果节点是其父节点的左节点，则返回右节点；否则（节点是其父节点的右节点）则向上遍历其父节点的父节点，重复之前的判断。
**代码：**

```
/*
public class TreeLinkNode {
    int val;
    TreeLinkNode left = null;
    TreeLinkNode right = null;
    TreeLinkNode next = null;

    TreeLinkNode(int val) {
        this.val = val;
    }
}
*/
public class Solution {
    public TreeLinkNode GetNext(TreeLinkNode pNode)
    {
        if(pNode == null){
            return null;
        }
        if(pNode.right != null){
            TreeLinkNode node = pNode.right;
            while(node.left != null){
                node = node.left;
            }
            return node;
        }
        TreeLinkNode nNode = pNode.next;
        while(nNode != null){
            if(nNode.left == pNode){
                return nNode;
            }
            pNode = nNode;
            nNode = nNode.next;
        }
        return null;
    }
}
```

# **删除链表中重复的结点**
在一个排序的链表中，存在重复的结点，请删除该链表中重复的结点，重复的结点不保留，返回链表头指针。 例如，链表1->2->3->3->4->4->5 处理后为 1->2->5

**链接：**[牛客网][12]

**思路：**
- 新建一个头结点
- 如果当前节点的下一节点nNode等于下下节点nNode2，nNode2指向其下一节点即nNode2 = nNode2.next，重复判断直到nNode跟nNode2不相等，将当前节点下一节点指向nNode2，continue跳出当前循环。
- 否则cur指向下一节点。
**代码：**

```
/*
 public class ListNode {
    int val;
    ListNode next = null;

    ListNode(int val) {
        this.val = val;
    }
}
*/
public class Solution {
    public ListNode deleteDuplication(ListNode pHead)
    {
        ListNode cur = new ListNode(-1);
        cur.next = pHead;
        ListNode res = cur;
        while(cur.next != null){
            ListNode nNode = cur.next;
            if(nNode.next != null){
                ListNode nNode2 = nNode.next;
                if(nNode2 != null && nNode.val == nNode2.val){
                    while(nNode2 != null && nNode.val == nNode2.val){
                        nNode2 = nNode2.next;
                    }
                    cur.next = nNode2;
                    continue;
                }
            }
            cur = nNode;
        }
        return res.next;
    }
}
```
# **字符流中第一个不重复的字符**
请实现一个函数用来找出字符流中第一个只出现一次的字符。例如，当从字符流中只读出前两个字符"go"时，第一个只出现一次的字符是"g"。当从该字符流中读出前六个字符“google"时，第一个只出现一次的字符是"l"。

**链接：**[牛客网][13]

**思路：**
- 用一快一慢的指针开始行走，直到两者相遇，则相遇点在环内，计算出环的节点数n。
- 让指针p1从起点先走n步，p2才开始从起点出发，两个指针相遇的点即为环的入口。
改进：
- 第一步同上。
- 让指针p1从起点出发，指针p2从相遇点出发，两者相遇点即为环的入口。证明如下图：

![clipboard.png](/img/bVbflt3)

**代码：**

```
/*
 public class ListNode {
    int val;
    ListNode next = null;

    ListNode(int val) {
        this.val = val;
    }
}
*/
public class Solution {

    public ListNode EntryNodeOfLoop(ListNode pHead)
    {
        if(pHead == null || pHead.next == null){
            return null;
        }
        ListNode slow = pHead.next;
        ListNode fast = pHead.next.next;
        while(slow != fast){
            if(slow.next != null && fast.next != null && fast.next.next != null){
                slow = slow.next;
                fast = fast.next.next;
            }else{
                return null;
            }
        }
        slow = pHead;
        while(slow != fast){
            slow = slow.next;
            fast = fast.next;
        }
        return slow;
    }
}
```

# **字符流中第一个不重复的字符**
请实现一个函数用来找出字符流中第一个只出现一次的字符。例如，当从字符流中只读出前两个字符"go"时，第一个只出现一次的字符是"g"。当从该字符流中读出前六个字符“google"时，第一个只出现一次的字符是"l"。

**链接：**[牛客网][13]

**思路：**

- 用一个128的数组来统计每次字符出现的字数
- 用一个队列，如果第一次遇到ch字符，则插入队列，其他情况不插入
- 求解第一个出现的字符，判断队首元素是否只出现一次，如果是直接返回，否则弹出重复该步骤
- 时间复杂度O(1)，空间复杂度O(n)

**代码：**

```
import java.util.Queue;
import java.util.LinkedList;
public class Solution {
    //Insert one char from stringstream
    int[] arr = new int[256];
    Queue<Character> queue = new LinkedList<>();
    public void Insert(char ch)
    {
        if(queue.contains(ch)){
            arr[ch]++;
        }else{
            queue.add(ch);
            arr[ch] = 1;
        }
    }
  //return the first appearence once char in current stringstream
    public char FirstAppearingOnce()
    {
        while(!queue.isEmpty() && arr[queue.peek()] > 1){
            queue.poll();
        }
        if(queue.isEmpty()){
            return '#';
        }
        return queue.peek();
    }
}
```

# **构建乘积数组**
给定一个数组A[0,1,...,n-1],请构建一个数组B[0,1,...,n-1],其中B中的元素B[i]=A[0]*A[1]*...*A[i-1]*A[i+1]*...*A[n-1]。不能使用除法。

**链接：**[牛客网][14]

**思路：**
- 先计算A[0]*...*A[i-1]这部分
- 在计算A[i+1]*...*A[n]这部分
**代码：**

```
import java.util.ArrayList;
public class Solution {
    public int[] multiply(int[] A) {
        int length = A.length;
        int[] B = new int[length];
        B[0] = 1;
        int tmp = 1;
        for(int i = 1;i<length;i++){
            B[i] = B[i-1]*A[i-1];
        }
        for(int i = length - 2;i>=0;i--){
            tmp = A[i+1] * tmp;
            B[i] = B[i] * tmp;
        }
        return B;
    }
}
```
# **数组中重复的数字**
在一个长度为n的数组里的所有数字都在0到n-1的范围内。 数组中某些数字是重复的，但不知道有几个数字是重复的。也不知道每个数字重复几次。请找出数组中任意一个重复的数字。 例如，如果输入长度为7的数组{2,3,1,0,2,5,3}，那么对应的输出是第一个重复的数字2。

**链接：**[牛客网][15]

**思路：**
- 因为所有数字都小于n,所以可以利用现有数组设置标记，当一个数字被访问过后，可以设置对应位上的数+n，之后遇到相同的数时，会发现对应作为上的数已经大于等于n了，那么直接返回这个数即可。
**代码：**

```
public class Solution {
    // Parameters:
    //    numbers:     an array of integers
    //    length:      the length of array numbers
    //    duplication: (Output) the duplicated number in the array number,length of duplication array is 1,so using duplication[0] = ? in implementation;
    //                  Here duplication like pointor in C/C++, duplication[0] equal *duplication in C/C++
    //    这里要特别注意~返回任意重复的一个，赋值duplication[0]
    // Return value:       true if the input is valid, and there are some duplications in the array number
    //                     otherwise false
    public boolean duplicate(int numbers[],int length,int [] duplication) {
         
        for(int i = 0;i<length;i++){
            int tmp = numbers[i];
            tmp = tmp < length ? tmp : tmp - length;
            if(numbers[tmp] >= length){
                duplication[0] = tmp;
                return true;
            }else{
                numbers[tmp] += length;
            }
        }
        return false;
    }
}
```
# **把字符串转化为整数**
将一个字符串转换成一个整数(实现Integer.valueOf(string)的功能，但是string不符合数字要求时返回0)，要求不能使用字符串转换整数的库函数。 数值为0或者字符串不是一个合法的数值则返回0。

**链接：**[牛客网][16]

**思路：**

**代码：**

```
public class Solution {
    public int StrToInt(String str) {
        int length = str.length();
        if(length == 0){
            return 0;
        }
        int flag = 0,res = 0;
        if(str.charAt(0) == '-'){
            flag = 1;
        }
        
        for(int i = flag;i<length;i++){
            if(str.charAt(i) == '+'){
                continue;
            }
            if(str.charAt(i) < '0' || str.charAt(i) > '9'){
                return 0;
            }
            res = res * 10 + str.charAt(i) - '0';
        }
        return flag == 1 ? -1 * res : res;
    }
}
```
# **不用加减乘除做加法**
写一个函数，求两个整数之和，要求在函数体内不得使用+、-、*、/四则运算符号。

**链接：**[牛客网][17]

**思路：**

- 首先看十进制是如何做的： 5+7=12，三步走
    - 第一步：相加各位的值，不算进位，得到2。
    - 第二步：计算进位值，得到10. 如果这一步的进位值为0，那么第一步得到的值就是最终结果。
    - 第三步：重复上述两步，只是相加的值变成上述两步的得到的结果2和10，得到12。

- 同样我们可以用三步走的方式计算二进制值相加： 5-101，7-111 第一步：相加各位的值，不算进位，得到010，二进制每位相加就相当于各位做异或操作，101^111。

    - 第二步：计算进位值，得到1010，相当于各位做与操作得到101，再向左移一位得到1010，(101&111)<<1。

    - 第三步重复上述两步， 各位相加 010^1010=1000，进位值为100=(010&1010)<<1。    
     继续重复上述两步：1000^100 = 1100，进位值为0，跳出循环，1100为最终结果
**代码**:

```
public class Solution {
    public int Add(int num1,int num2) {
        while(num2 != 0){
            int tmp = num1 ^ num2;
            num2 = (num1 & num2) << 1;
            num1 = tmp;
        }
        return num1;
    }
}
```

# **求1+2+3+...+n**
求1+2+3+...+n，要求不能使用乘除法、for、while、if、else、switch、case等关键字及条件判断语句（A?B:C）。

**链接：**[牛客网][18]

**思路：**
- 递归加&&运算短路机制
**代码：**

```
public class Solution {
    public int Sum_Solution(int n) {
        int res = n;
        boolean flag = (res > 0) && (res += Sum_Solution(n-1)) > 0;
        return res;
    }
}
```

# **扑克牌顺序**
LL今天心情特别好,因为他去买了一副扑克牌,发现里面居然有2个大王,2个小王(一副牌原本是54张^_^)...他随机从中抽出了5张牌,想测测自己的手气,看看能不能抽到顺子,如果抽到的话,他决定去买体育彩票,嘿嘿！！“红心A,黑桃3,小王,大王,方片5”,“Oh My God!”不是顺子.....LL不高兴了,他想了想,决定大\小 王可以看成任何数字,并且A看作1,J为11,Q为12,K为13。上面的5张牌就可以变成“1,2,3,4,5”(大小王分别看作2和4),“So Lucky!”。LL决定去买体育彩票啦。 现在,要求你使用这幅牌模拟上面的过程,然后告诉我们LL的运气如何， 如果牌能组成顺子就输出true，否则就输出false。为了方便起见,你可以认为大小王是0。

**链接：**[牛客网][19]

**思路：**
- 先统计出大小王（0）的数量，然后将数组排序，比较相邻两个数之间的差值总和与大小王数量的高低。
**代码：**

```
import java.util.Arrays;
public class Solution {
    public boolean isContinuous(int [] numbers) {
        int length = numbers.length;
        if(length == 0){
            return false;
        }
        int count = 0;
        for(int i = 0;i<length;i++){
            if(numbers[i] == 0){
                count++;
            }
        }
        Arrays.sort(numbers);
        for(int i = count;i<length-1;i++){
            if(numbers[i+1] == numbers[i]){
                return false;
            }
            if(numbers[i+1] - numbers[i] > 1){
                count -= numbers[i+1] - numbers[i] - 1;
            }
            if(count < 0){
                return false;
            }
        }
        return true;
    }
}
```

# **反转单词顺序列**
牛客最近来了一个新员工Fish，每天早晨总是会拿着一本英文杂志，写些句子在本子上。同事Cat对Fish写的内容颇感兴趣，有一天他向Fish借来翻看，但却读不懂它的意思。例如，“student. a am I”。后来才意识到，这家伙原来把句子单词的顺序翻转了，正确的句子应该是“I am a student.”。Cat对一一的翻转这些单词顺序可不在行，你能帮助他么？

**链接：**[牛客网][20]

**思路：**
- 以" " 空格分割
- 通过进行StringBuilder进行拼接
- 扩展，当所有的字符都翻转时，可先通过翻转全部字符，再翻转单词
**代码：**

```
public class Solution {
    public String ReverseSentence(String str) {
        if(str.trim().equals("")){
            return str;
        }
        String[] strr = str.split(" ");
        StringBuilder sb = new StringBuilder();
        for(int i = 0;i<strr.length;i++){
            sb.append(strr[strr.length - 1 - i]).append(" ");
        }
        return sb.substring(0,sb.length()-1).toString();
    }
}
```

# **左旋装字符串**
汇编语言中有一种移位指令叫做循环左移（ROL），现在有个简单的任务，就是用字符串模拟这个指令的运算结果。对于一个给定的字符序列S，请你把其循环左移K位后的序列输出。例如，字符序列S=”abcXYZdef”,要求输出循环左移3位后的结果，即“XYZdefabc”。是不是很简单？OK，搞定它！

**链接：**[牛客网][21]

**思路：**
- 见代码
- 另一种思路：将数组分为A、B两部分，先各翻转A、B，最后翻转数组即可。
**代码：**

```
public class Solution {
    public String LeftRotateString(String str,int n) {
        if(str.trim().equals("")){
            return str;
        }
        String cur = str + str;
        return cur.substring(n,str.length() + n);
    }
}
```

# **和为S的两个数字**
输入一个递增排序的数组和一个数字S，在数组中查找两个数，使得他们的和正好是S，如果有多对数字的和等于S，输出两个数的乘积最小的。

**链接：**[牛客网][22]

**思路：**
设头尾两个指针，小于目标数则头指针右移，大于则尾指针左移，相等则为目标数，第一次遇到（间隔最远）则为乘积最小组合。
**代码：**

```
import java.util.ArrayList;
public class Solution {
    public ArrayList<Integer> FindNumbersWithSum(int [] array,int sum) {
        ArrayList<Integer> result = new ArrayList<Integer>();
        if(array == null || array.length == 0 || sum == 0){
            return result;
        }
        int start = 0,end = array.length-1;
        while(start <= end){
            int curSum = array[start] + array[end];
            if(curSum == sum){
                result.add(array[start]);
                result.add(array[end]);
                break;
            }
            else if(curSum < sum){
                start++;
            }
            else{
                end--;
            }
            
        }
        return result;
    }
}
```


  [1]: https://www.nowcoder.com/practice/6e5207314b5241fb83f2329e89fdecc8?tpId=13&tqId=11219&tPage=4&rp=4&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking
  [2]: https://www.nowcoder.com/practice/c61c6999eecb4b8f88a98f66b273a3cc?tpId=13&tqId=11218&rp=4&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking
  [3]: https://www.nowcoder.com/practice/1624bc35a45c42c0bc17d17fa0cba788?tpId=13&tqId=11217&rp=4&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking
  [4]: https://www.nowcoder.com/practice/9be0172896bd43948f8a32fb954e1be1?tpId=13&tqId=11216&rp=4&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking
  [5]: https://www.nowcoder.com/practice/ef068f602dde4d28aab2b210e859150a?tpId=13&tqId=11215&rp=4&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking
  [6]: https://www.nowcoder.com/practice/cf7e25aa97c04cc1a68c8f040e71fb84?tpId=13&tqId=11214&rp=4&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking
  [7]: https://www.nowcoder.com/practice/445c44d982d04483b04a54f298796288?tpId=13&tqId=11213&rp=4&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking
  [8]: https://www.nowcoder.com/practice/91b69814117f4e8097390d107d2efbe0?tpId=13&tqId=11212&rp=4&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking
  [9]: https://www.nowcoder.com/practice/ff05d44dfdb04e1d83bdbdab320efbcb?tpId=13&tqId=11211&rp=4&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking
  [10]: https://www.nowcoder.com/practice/ff05d44dfdb04e1d83bdbdab320efbcb?tpId=13&tqId=11211&rp=4&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking
  [11]: https://www.nowcoder.com/practice/9023a0c988684a53960365b889ceaf5e?tpId=13&tqId=11210&rp=4&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking
  [12]: https://www.nowcoder.com/practice/fc533c45b73a41b0b44ccba763f866ef?tpId=13&tqId=11209&rp=4&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking
  [13]: https://www.nowcoder.com/practice/00de97733b8e4f97a3fb5c680ee10720?tpId=13&tqId=11207&rp=4&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking
  [14]: https://www.nowcoder.com/practice/94a4d381a68b47b7a8bed86f2975db46?tpId=13&tqId=11204&rp=4&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking
  [15]: https://www.nowcoder.com/practice/623a5ac0ea5b4e5f95552655361ae0a8?tpId=13&tqId=11203&rp=4&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking
  [16]: https://www.nowcoder.com/practice/1277c681251b4372bdef344468e4f26e?tpId=13&tqId=11202&rp=4&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking
  [17]: https://www.nowcoder.com/practice/59ac416b4b944300b617d4f7f111b215?tpId=13&tqId=11201&rp=4&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking
  [18]: https://www.nowcoder.com/practice/7a0da8fc483247ff8800059e12d7caf1?tpId=13&tqId=11200&rp=4&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking
  [19]: https://www.nowcoder.com/practice/762836f4d43d43ca9deb273b3de8e1f4?tpId=13&tqId=11198&rp=4&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking
  [20]: https://www.nowcoder.com/practice/3194a4f4cf814f63919d0790578d51f3?tpId=13&tqId=11197&rp=4&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking
  [21]: https://www.nowcoder.com/practice/12d959b108cb42b1ab72cef4d36af5ec?tpId=13&tqId=11196&rp=4&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking
  [22]: https://www.nowcoder.com/practice/390da4f7a00f44bea7c2f3d19491311b?tpId=13&tqId=11195&rp=4&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking
