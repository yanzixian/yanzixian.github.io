#### 146、LRU缓存机制

##### 题目描述

```text
运用你所掌握的数据结构，设计和实现一个  LRU (最近最少使用) 缓存机制。它应该支持以下操作： 获取数据 get 和 写入数据 put 。

获取数据 get(key) - 如果密钥 (key) 存在于缓存中，则获取密钥的值（总是正数），否则返回 -1。

```

##### 示例

```java
LRUCache cache = new LRUCache( 2 /* 缓存容量 */ );

cache.put(1, 1);
cache.put(2, 2);
cache.get(1);       // 返回  1
cache.put(3, 3);    // 该操作会使得密钥 2 作废
cache.get(2);       // 返回 -1 (未找到)
cache.put(4, 4);    // 该操作会使得密钥 1 作废
cache.get(1);       // 返回 -1 (未找到)
cache.get(3);       // 返回  3
cache.get(4);       // 返回  4
```

##### 思路

```reStructuredText
利用一个HashMap和双向列表实现
双向列表表头为最近使用的节点，表尾为最近未使用的节点
每对一个节点进行操作，均将其位置确定为表头，容量不够时则删除表尾的节点
```

##### 代码

```java
public class LRUCache {

    class LinkedNode {
        int key;
        int value;
        LinkedNode pre;
        LinkedNode next;

        public LinkedNode() {
        }

        public LinkedNode(int _key, int _value) {
            key = _key;
            value = _value;
        }
    }

    HashMap<Integer, LinkedNode> hashMap = new HashMap<>();
    //最大容量
    int capacity;
    //已使用容量
    int size = 0;
    LinkedNode head, tail;

    public LRUCache(int capacity) {
        this.capacity = capacity;
        this.size = 0;
        //创建头节点与尾节点
        head = new LinkedNode();
        tail = new LinkedNode();
        head.next = tail;
        tail.pre = head;
    }

    public int get(int key) {
        LinkedNode node = hashMap.get(key);
        if (node == null) {
            return -1;
        }
        //从原位置删除node节点
        node.next.pre = node.pre;
        node.pre.next = node.next;

        //将node节点添加到头部
        node.next = head.next;
        node.pre = head;

        head.next.pre = node;
        head.next = node;

        return node.value;
    }

    public void put(int key, int value) {
        LinkedNode node1 = hashMap.get(key);
        if (node1 == null) {
            //节点不存在

            if (size >= capacity) {
                //删除最后一个节点
                hashMap.remove(tail.pre.key);
                tail.pre.pre.next = tail;
                tail.pre = tail.pre.pre;

            }
            LinkedNode node = new LinkedNode(key, value);
            hashMap.put(key, node);
            //将node节点添加到头部
            node.next = head.next;
            node.pre = head;
            head.next.pre = node;
            head.next = node;

            size++;

        } else {
            //节点存在，更改value值
            node1.value = value;
            //从原位置删除node节点
            node1.next.pre = node1.pre;
            node1.pre.next = node1.next;

            //将node1节点添加到头部
            node1.next = head.next;
            node1.pre = head;
            head.next.pre = node1;
            head.next = node1;

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

#### 287、寻找重复数

##### 题目描述

给定一个包含 n + 1 个整数的数组 nums，其数字都在 1 到 n 之间（包括 1 和 n），可知至少存在一个重复的整数。假设只有一个重复的整数，找出这个重复的数。

示例 1:

输入: [1,3,4,2,2]
输出: 2

示例 2:

输入: [3,1,3,4,2]
输出: 3

说明：

    不能更改原数组（假设数组是只读的）。
    只能使用额外的 O(1) 的空间。
    时间复杂度小于 O(n2) 。
    数组中只有一个重复的数字，但它可能不止重复出现一次。

##### 方法一

设置标志数组，但空间复杂度为O(n)，不符合题目要求

```java
public int findDuplicate(int[] nums) {
        int[] flags=new int[nums.length];
        Arrays.fill(flags,0);

        for(int i:nums){
            if(flags[i]==0){
                flags[i]=1;
            }else{
                return i;
            }

        }
        return -1;
    }
```

##### 方法二

对数组进行排序，后比较相邻两数是否相等

对原数组进行了改变，不符合题目要求；内存消耗较大

```java
public int findDuplicate(int[] nums) {
        Arrays.sort(nums);
        for(int i=0;i<nums.length;i++){
            if(nums[i]==nums[i+1]){
                return nums[i];
            }
        }
        return -1;
    }
```

##### 方法三（官方）

二分查找

###### 思路

> 定义 cnt[i]表示 nums[]数组中小于等于 i的数有多少个，假设我们重复的数是 target，那么 [1,target−1]里的所有数满足`cnt[i]≤i`，[target,n] 里的所有数满足 `cnt[i]>i`，具有单调性
>

这里需要明白，对于一个简单连续的的序列1，2，3，4 ...，有`cnt[i]=i`

若该序列中出现缺失，1，2，4，5...，则有`cnt[i]<i`

若该序列中出现重复，1，2，3，3，4，5...，3出现重复，则对于`i<3`，有`cnt[i]<=i`成立；对于`i>=3`，均有`cnt[i]>i`成立；

这样就转化为寻找一个满足条件的数target的问题，条件为：对于则对于`i<target`，有`cnt[i]<=i`成立；对于`i>=target`，均有`cnt[i]>i`成立；使用二分查找寻找该数

###### 算法

```java
public int findDuplicate(int[] nums) {
   int n= nums.length;
   int l=1,h=n-1,mid=0;
   int ans=-1;
   while(l<=h){
       mid=(l+h)>>1;
       int count=0;
       for(int i=0;i<n;i++){
           //计算cnt[mid]
           if(nums[i]<=mid) {
               count++;
           }
       }
       //对于数字mid，如果cnt[mid]<=mid，则mid位于[l,target-1)中，故需要从后半部分寻找target
       if(count<=mid){
           l=mid+1;
       }else{
           //如果cnt[mid]>mid，则mid位于[target,h)中，即target位于[l,mid]中，故需要从前半部分寻找target
           h=mid-1;
           ans=mid;
       }
   }
   return ans;
}
```

#### 974、和可被 K 整除的子数组

##### 题目描述

```
给定一个整数数组 A，返回其中元素之和可被 K 整除的（连续、非空）子数组的数目。

 

示例：

输入：A = [4,5,0,-2,-3,1], K = 5
输出：7
解释：
有 7 个子数组满足其元素之和可被 K = 5 整除：
[4, 5, 0, -2, -3, 1], [5], [5, 0], [5, 0, -2, -3], [0], [0, -2, -3], [-2, -3]
```

##### 方法一 前缀和（官方）

###### 思路

> 通常，涉及连续子数组问题的时候，我们使用前缀和来解决。
>
> 我们令 `P[i]=A[0]+A[1]+...+A[i]`。那么每个连续子数组的和 sum(i,j) 就可以写成 `P[j]−P[i−1]`（其中 0<i<j）的形式。此时，判断子数组的和能否被 K整除就等价于判断 `(P[j]−P[i−1]) mod K==0`，根据 同余定理，只要 `P[j] mod K==P[i−1] mod K`，就可以保证上面的等式成立。
>
> 因此我们可以考虑对数组进行遍历，在遍历同时统计答案。当我们遍历到第 i 个元素时，我们统计以 i 结尾的符合条件的子数组个数。我们可以维护一个以前缀和模 K 的值为键，出现次数为值的哈希表 record，在遍历的同时进行更新。这样在计算以 iii 结尾的符合条件的子数组个数时，根据上面的分析，答案即为`[0..i−1]` 中前缀和模 K 也为 `P[i] mod K` 的位置个数，即 `record[P[i] mod K`]
>

即利用同余定理，对于每一个出现的余数，计算具有相同余数的前缀和的个数；

从具有相同余数的前缀和中任意取出两个，它们的差（即相应子数组的和）一定被K整除；

###### 算法

```java
public int subarraysDivByK(int[] A, int K) {
    Map<Integer, Integer> map = new HashMap<>();
    int len = A.length;
    int sum = 0;
    int result = 0;
    //这里需要注意余数为0的情况，即前缀和本身就被K整除，则不需要非得和其他前缀和作差，来满足条件，因此需要初始化余数为0是的值为1
    map.put(0, 1);
    for (int i = 0; i < len; i++) {
        sum += A[i];
        int modulus = (sum % K + K) % K;
        int count = map.getOrDefault(modulus, 0);
        //每出现一个相同余数的前缀和，均与其余前缀和作差，得到的子数组个数为count
        result += count;
        map.put(modulus, count + 1);
    }
    return result;
}
```

#### 394、 字符串解码

##### 题目描述

> 给定一个经过编码的字符串，返回它解码后的字符串。
>
> 编码规则为: k[encoded_string]，表示其中方括号内部的 encoded_string 正好重复 k 次。注意 k 保证为正整数。
>
> 你可以认为输入字符串总是有效的；输入字符串中没有额外的空格，且输入的方括号总是符合格式要求的。
>
> 此外，你可以认为原始数据不包含数字，所有的数字只表示重复的次数 k ，例如不会出现像 3a 或 2[4] 的输入。
>
> 示例:
>
> s = "3[a]2[bc]", 返回 "aaabcbc".
> s = "3[a2[c]]", 返回 "accaccacc".
> s = "2[abc]3[cd]ef", 返回 "abcabccdcdcdef".

##### 方法一 栈操作（官方）

栈操作

###### 思路

本题中可能出现括号嵌套的情况，比如 `2[a2[bc]]`，这种情况下我们可以先转化成 2[abcbc]，在转化成 `abcbcabcbc`。我们可以把字母、数字和括号看成是独立的 TOKEN，并用栈来维护这些 TOKEN。具体的做法是，遍历这个栈：

    如果当前的字符为数位，解析出一个数字（连续的多个数位）并进栈
    如果当前的字符为字母或者左括号，直接进栈
    如果当前的字符为右括号，开始出栈，一直到左括号出栈，出栈序列反转后拼接成一个字符串，此时取出栈顶的数字（此时栈顶一定是数字，想想为什么？），就是这个字符串应该出现的次数，我们根据这个次数和字符串构造出新的字符串并进栈

最终将栈中的元素按照从栈底到栈顶的顺序拼接起来，就得到了答案。**注意：这里可以用不定长数组来模拟栈操作，方便从栈底向栈顶遍历**

###### 算法

```java
class Solution {
public:
    string getDigits(string &s, size_t &ptr) {
        string ret = "";
        while (isdigit(s[ptr])) {
            ret.push_back(s[ptr++]);
        }
        return ret;
    }

    string getString(vector <string> &v) {
        string ret;
        for (const auto &s: v) {
            ret += s;
        }
        return ret;
    }

    string decodeString(string s) {
        vector <string> stk;
        size_t ptr = 0;

        while (ptr < s.size()) {
            char cur = s[ptr];
            if (isdigit(cur)) {
                // 获取一个数字并进栈
                string digits = getDigits(s, ptr);
                stk.push_back(digits);
            } else if (isalpha(cur) || cur == '[') {
                // 获取一个字母并进栈
                stk.push_back(string(1, s[ptr++])); 
            } else {
                ++ptr;
                vector <string> sub;
                while (stk.back() != "[") {
                    sub.push_back(stk.back());
                    stk.pop_back();
                }
                reverse(sub.begin(), sub.end());
                // 左括号出栈
                stk.pop_back();
                // 此时栈顶为当前 sub 对应的字符串应该出现的次数
                int repTime = stoi(stk.back()); 
                stk.pop_back();
                string t, o = getString(sub);
                // 构造字符串
                while (repTime--) t += o; 
                // 将构造好的字符串入栈
                stk.push_back(t);
            }
        }

        return getString(stk);
    }
};

```

##### 方法二 递归（官方）

###### 思路

我们也可以用递归来解决这个问题，从左向右解析字符串：

    如果当前位置为数字位，那么后面一定包含一个用方括号表示的字符串，即属于这种情况：k[...]：
        我们可以先解析出一个数字，然后解析到了左括号，递归向下解析后面的内容，遇到对应的右括号就返回，此时我们可以根据解析出的数字 x 解析出的括号里的字符串 s′ 构造出一个新的字符串 x×s′′；
        我们把 k[...] 解析结束后，再次调用递归函数，解析右括号右边的内容。
    如果当前位置是字母位，那么我们直接解析当前这个字母，然后递归向下解析这个字母后面的内容。

###### 算法
```java
class Solution {
    String src;
    int ptr;
public String decodeString(String s) {
    src = s;
    ptr = 0;
    return getString();
}

	public String getString() {
    if (ptr == src.length() || src.charAt(ptr) == ']') {
        // String -> EPS
        return "";
    }

    char cur = src.charAt(ptr);
    int repTime = 1;
    String ret = "";

    if (Character.isDigit(cur)) {
        // String -> Digits [ String ] String
        // 解析 Digits
        repTime = getDigits(); 
        // 过滤左括号
        ++ptr;
        // 解析 String
        String str = getString(); 
        // 过滤右括号
        ++ptr;
        // 构造字符串
        while (repTime-- > 0) {
            ret += str;
        }
    } else if (Character.isLetter(cur)) {
        // String -> Char String
        // 解析 Char
        ret = String.valueOf(src.charAt(ptr++));
    }
    
    return ret + getString();
}

	public int getDigits() {
    int ret = 0;
    while (ptr < src.length() && Character.isDigit(src.charAt(ptr))) {
        ret = ret * 10 + src.charAt(ptr++) - '0';
    }
    return ret;
}

}
```

#### 198、打家劫舍

##### 题目描述

你是一个专业的小偷，计划偷窃沿街的房屋。每间房内都藏有一定的现金，影响你偷窃的唯一制约因素就是相邻的房屋装有相互连通的防盗系统，如果两间相邻的房屋在同一晚上被小偷闯入，系统会自动报警。

给定一个代表每个房屋存放金额的非负整数数组，计算你 不触动警报装置的情况下 ，一夜之内能够偷窃到的最高金额。

示例 1:

```
输入: [1,2,3,1]
输出: 4
解释: 偷窃 1 号房屋 (金额 = 1) ，然后偷窃 3 号房屋 (金额 = 3)。
     偷窃到的最高金额 = 1 + 3 = 4 。
```

示例 2:

```
输入: [2,7,9,3,1]
输出: 12
解释: 偷窃 1 号房屋 (金额 = 2), 偷窃 3 号房屋 (金额 = 9)，接着偷窃 5 号房屋 (金额 = 1)。
     偷窃到的最高金额 = 2 + 9 + 1 = 12 。


```

##### 方法一 动态规划

###### 思路

> 首先考虑最简单的情况。如果只有一间房屋，则偷窃该房屋，可以偷窃到最高总金额。如果只有两间房屋，则由于两间房屋相邻，不能同时偷窃，只能偷窃其中的一间房屋，因此选择其中金额较高的房屋进行偷窃，可以偷窃到最高总金额。
>
> 如果房屋数量大于两间，应该如何计算能够偷窃到的最高总金额呢？对于第 k (k>2)) 间房屋，有两个选项：
>
>     偷窃第 k间房屋，那么就不能偷窃第 k−1 间房屋，偷窃总金额为前 k−2 间房屋的最高总金额与第 kkk 间房屋的金额之和。
>    
>     不偷窃第 k 间房屋，偷窃总金额为前 k−1间房屋的最高总金额。
>
> 在两个选项中选择偷窃总金额较大的选项，该选项对应的偷窃总金额即为前 kkk 间房屋能偷窃到的最高总金额。
>
> 用 dp[i] 表示前 iii 间房屋能偷窃到的最高总金额，那么就有如下的状态转移方程：
>
> `dp[i]=max⁡(dp[i−2]+nums[i],dp[i−1])`
>
> 边界条件为：
>
> `dp[0]=nums[0]`只有一间房屋，则偷窃该房屋
>
> `dp[1]=max⁡(nums[0],nums[1])`只有两间房屋，选择其中金额较高的房屋进行偷窃
>
> 最终的答案即为 dp[n−1]，其中 n 是数组的长度。
>

###### 算法

```java
class Solution {
    public int rob(int[] nums) {
       int[] dp=new int[nums.length];
        int i=0;
        while(i<nums.length){
            if(i==0){
                dp[i]=nums[i];
            }
            if(i==1){
                dp[i]=Math.max(nums[0],nums[1]);
            }
            if(i>=2){
                dp[i]=Math.max(dp[i-2]+nums[i],dp[i-1]);
            }
            i++;
        }
        
        return nums.length>0?dp[nums.length-1]:0;
    }
}
```

##### 方法二 动态规划+滚动数组（官方）

###### 思路

> 接方法一，由代码执行过程可知，dp[i]只与dp[i-1]和dp[i-2]，有关，所以并不需要将dp数组的大小设置为和nums一样，使之大小为2即可

###### 算法

```java
if(nums==null||nums.length==0){
            return 0;
        }
        if(nums.length==1){
            return nums[0];
        }
        int[] dp=new int[3];
        int i=2;
        dp[0]=nums[0];
        dp[1]=Math.max(nums[0],nums[1]);
        while(i<nums.length){
            dp[2]=Math.max(dp[0]+nums[i],dp[1]);
            dp[0]=dp[1];
            dp[1]=dp[2];
            i++;
        }

        return dp[1];
```

#### 1431、拥有糖果最多的孩子

##### 题目描述

给你一个数组 candies 和一个整数 extraCandies ，其中 candies[i] 代表第 i 个孩子拥有的糖果数目。

对每一个孩子，检查是否存在一种方案，将额外的 extraCandies 个糖果分配给孩子们之后，此孩子有 最多 的糖果。注意，允许有多个孩子同时拥有 最多 的糖果数目。

示例 1：

```
输入：candies = [2,3,5,1,3], extraCandies = 3
输出：[true,true,true,false,true] 
解释：
孩子 1 有 2 个糖果，如果他得到所有额外的糖果（3个），那么他总共有 5 个糖果，他将成为拥有最多糖果的孩子。
孩子 2 有 3 个糖果，如果他得到至少 2 个额外糖果，那么他将成为拥有最多糖果的孩子。
孩子 3 有 5 个糖果，他已经是拥有最多糖果的孩子。
孩子 4 有 1 个糖果，即使他得到所有额外的糖果，他也只有 4 个糖果，无法成为拥有糖果最多的孩子。
孩子 5 有 3 个糖果，如果他得到至少 2 个额外糖果，那么他将成为拥有最多糖果的孩子。
```

示例 2：

```
输入：candies = [4,2,1,1,2], extraCandies = 1
输出：[true,false,false,false,false] 
解释：只有 1 个额外糖果，所以不管额外糖果给谁，只有孩子 1 可以成为拥有糖果最多的孩子。
```

示例 3：

```
输入：candies = [12,1,12], extraCandies = 10
输出：[true,false,true]
```

##### 思路

题目只需要判断是否存在一种方案满足条件，因此只需要探讨将所有的糖果分给一个孩子是否满足条件即可；

事实上，对于每一个小朋友，只要这个小朋友「拥有的糖果数目」加上「额外的糖果数目」大于等于所有小朋友拥有的糖果数目最大值，那么这个小朋友就可以拥有最多的糖果。

##### 算法

```java
class Solution {
    public List<Boolean> kidsWithCandies(int[] candies, int extraCandies) {
        int max=0;
        for(int i=0;i<candies.length;i++){
            max=Math.max(max,candies[i]);
        }
        List<Boolean> result=new ArrayList<>();
        for(int i=0;i<candies.length;i++){
            result.add(i,(candies[i]+extraCandies)>=max);
        }
        return result;
    }
}
```

#### 面试题64. 求1+2+…+n

##### 题目描述

求 `1+2+...+n` ，要求不能使用乘除法、for、while、if、else、switch、case等关键字及条件判断语句（A?B:C）。

示例 1：

```
输入: n = 3
输出: 6
```

示例 2：

```
输入: n = 9
输出: 45
```

限制：

    1 <= n <= 10000

##### 方法

###### 思路

由于不能使用循环、乘除，判断等，故常用的公式计算和迭代不能使用，可以考虑递归法，只需要考虑如何终止递归即可

可以使用逻辑运算符的性质来实现终止：

常见的逻辑运算符有三种，即 “`与 &&` ”，“`或 ∣∣` ”，“`非 !`” ；而其有重要的短路效应，如下所示：

```
if(A && B)  // 若 A 为 false ，则 B 的判断不会执行（即短路），直接判定 A && B 为 false

if(A || B) // 若 A 为 true ，则 B 的判断不会执行（即短路），直接判定 A || B 为 true
```

###### 算法

```java
public int sumNums(int n) {
		//n=0时递归停止
        boolean flag=n>0&&(n+=sumNums(n-1))>0;
        return n;
    }
```

#### 152. 乘积最大子数组

##### 题目描述

给你一个整数数组 nums ，请你找出数组中乘积最大的连续子数组（该子数组中至少包含一个数字），并返回该子数组所对应的乘积。

示例 1:

```
输入: [2,3,-2,4]
输出: 6
```


解释: 子数组 [2,3] 有最大乘积 6。

示例 2:

```
输入: [-2,0,-1]
输出: 0
解释: 结果不能为 2, 因为 [-2,-1] 不是子数组。
```

##### 方法一

暴力破解

###### 算法

```java
  public int maxProduct(int[] nums) {
        int temp=1;
        int result=Integer.MIN_VALUE;
        for(int i=0;i<nums.length;i++){
            for(int j=i;j<nums.length;j++){
                for(int k=i;k<=j;k++){
                    temp=nums[k]*temp;

                }
                result= Math.max(temp, result);
                temp=1;
            }
        }
    return result;
    }
```

##### 方法二 动态规划（官方）

###### 思路

> 如果我们用 fmax⁡(i) 开表示以第 i个元素结尾的乘积最大子数组的乘积，a表示输入参数 nums，那么根据「53. 最大子序和」的经验，我们很容易推导出这样的状态转移方程：
>
> `fmax⁡(i)=max⁡{f(i−1)×ai,ai}`
>
> 它表示以第 i 个元素结尾的乘积最大子数组的乘积可以考虑 ai 加入前面的 fmax⁡(i−1) 对应的一段，或者单独成为一段，这里两种情况下取最大值。求出所有的 fmax⁡(i)之后选取最大的一个作为答案。
>
> 可是在这里，这样做是错误的。为什么呢？
>
> 因为这里的定义并不满足「最优子结构」。具体地讲，如果 a={5,6,−3,4,−3}，那么此时 fmax⁡对应的序列是 {5,30,−3,4,−3}，按照前面的算法我们可以得到答案为 30，即前两个数的乘积，而实际上答案应该是全体数字的乘积。我们来想一想问题出在哪里呢？问题出在最后一个 −3 所对应的 fmax⁡ 的值既不是 −3，也不是 4×−3，而是 5×30×(−3)×4×(−3)。所以我们得到了一个结论：当前位置的最优解未必是由前一个位置的最优解转移得到的。
>
> 我们可以根据正负性进行分类讨论。
>
> 考虑当前位置如果是一个负数的话，那么我们希望以它前一个位置结尾的某个段的积也是个负数，这样就可以负负得正，并且我们希望这个积尽可能「负得更多」，即尽可能小。如果当前位置是一个正数的话，我们更希望以它前一个位置结尾的某个段的积也是个正数，并且希望它尽可能地大。于是这里我们可以再维护一个 fmin⁡(i)，它表示以第 i 个元素结尾的乘积最小子数组的乘积，那么我们可以得到这样的动态规划转移方程：
>
> `fmax⁡(i)=max⁡i=1n{fmax⁡(i−1)×ai,fmin⁡(i−1)×ai,ai｝`
>
> `fmin⁡(i)=min⁡i=1n{fmax⁡(i−1)×ai,fmin⁡(i−1)×ai,ai}` 
>
> 它代表第 iii 个元素结尾的乘积最大子数组的乘积 fmax⁡(i)，可以考虑把 ai 加入第 i−1 个元素结尾的乘积最大或最小的子数组的乘积中，二者加上 ai，三者取大，就是第 i个元素结尾的乘积最大子数组的乘积。第 i个元素结尾的乘积最小子数组的乘积 fmin⁡(i) 同理。
>

###### 算法

```java
public int maxProduct(int[] nums) {
        int[] max=new int[nums.length];
        int[] min=new int[nums.length];
        max[0]=nums[0];
        min[0]=nums[0];
        int result=nums[0];
        for(int i=1;i< nums.length;i++){
            max[i]=Math.max(max[i-1]*nums[i],Math.max(min[i-1]*nums[i],nums[i]));
            min[i]=Math.min(max[i-1]*nums[i],Math.min(min[i-1]*nums[i],nums[i]));
            result=Math.max(result,max[i]);
        }
        return result;
    }
```

进一步，在计算出之后的max[i]，min[i]时，可以根据滚动数组的思想，可以进一步压缩空间

```java
public int maxProduct(int[] nums) {
        int max=nums[0];
        int min=nums[0];
        int result=nums[0];
        for(int i=1;i< nums.length;i++){
            int tmp=Math.max(max*nums[i],Math.max(min*nums[i],nums[i]));
            int tmp1=Math.min(max*nums[i],Math.min(min*nums[i],nums[i]));
            max=tmp;
            min=tmp1;
            result=Math.max(result,max);
        }
        return result;
    }
```

#### 5、最长回文子串

##### 题目描述

给定一个字符串 s，找到 s 中最长的回文子串。你可以假设 s 的最大长度为 1000。

示例 1：

```
输入: "babad"
输出: "bab"
注意: "aba" 也是一个有效答案。
```

示例 2：

```
输入: "cbbd"
输出: "bb"
```

##### 方法一 暴力破解

###### 思路

对于每个子串P[i,j]，判断是否是回文串，找出最长的回文串输出即可

###### 算法

```java
class Solution {
    public String longestPalindrome(String s) {
        String result="";
        int l=Integer.MIN_VALUE;
        for(int i=0;i<s.length();i++){
            for(int j=i;j<=s.length();j++){
                if(isPalindrome(s.substring(i,j))){
                    int length = s.substring(i, j).length();

                    if(l < length){
                        l= length;
                        result=s.substring(i,j);
                    }
                }
            }
        }
        return result;
    }
    public Boolean isPalindrome(String s){
        Stack<Character> stack=new Stack<>();
        for(int i=0;i<s.length()/2;i++){
            if(s.charAt(i)!=s.charAt(s.length()-i-1)){
                return false;
            }

        }
        return true;

    }
}
```

##### 方法二 动态规划

###### 思路

> 对于一个子串而言，如果它是回文串，并且长度大于 2，那么将它首尾的两个字母去除之后，它仍然是个回文串。例如对于字符串 “ababa如果我们已经知道 “bab”是回文串，那么 “ababa” 一定是回文串，这是因为它的首尾两个字母都是 “a”。
>
> 根据这样的思路，我们就可以用动态规划的方法解决本题。我们用 P(i,j) 表示字符串 s 的第 i到 j 个字母组成的串（下文表示成 s[i:j]）是否为回文串：
>
> - P(i,j)=true,如果子串 Si…Sj 是回文串
> - P(i,j)=false,其它情况
>
> 这里的「其它情况」包含两种可能性：
>
> - s[i,j]本身不是一个回文串；      
> - i>j此时 s[i,j] 本身不合法。
> 
>
>那么我们就可以写出动态规划的状态转移方程：
> 
>`P(i,j)=P(i+1,j−1)∧(Si==Sj)`
> 
>也就是说，只有 s[i+1:j−1]是回文串，并且 s的第 i和 j 个字母相同时，s[i:j]才会是回文串。
> 
>上文的所有讨论是建立在子串长度大于 2 的前提之上的，我们还需要考虑动态规划中的边界条件，即子串的长度为 1 或 2。对于长度为 1 的子串，它显然是个回文串；对于长度为 2 的子串，只要它的两个字母相同，它就是一个回文串。因此我们就可以写出动态规划的边界条件：
> 
>- P(i,i)=true
> - P(i,i+1)=(Si==Si+1)

> 根据这个思路，我们就可以完成动态规划了，最终的答案即为所有 P(i,j)=true中 j−i+1（即子串长度）的最大值。注意：在状态转移方程中，我们是从长度较短的字符串向长度较长的字符串进行转移的，因此一定要注意动态规划的循环顺序。
>

###### 算法

```java
public String longestPalindrome(String s) {
        int len=s.length();
         if (len <=1) {
            return s;
        }
        boolean[][] p = new boolean[len][len];

        int min = 0, max = 0;
        int l = 0;
		//列优先
        for (int j = 0; j < len; j++) {
            for (int i = j; i>=0&&i<len; i--) {

                boolean b = s.charAt(i) == s.charAt(j);
                if (i == j) {
                    p[i][j] = true;
                } else if (i + 1 == j && b) {
                    p[i][j] = true;
                    if (j - i + 1 > l) {
                        l = j - i + 1;
                        max = j;
                        min = i;
                    }
                } else if (p[i + 1][j - 1] && b) {
                    p[i][j] = true;
                    if (j - i + 1 > l) {
                        l = j - i + 1;
                        max = j;
                        min = i;
                    }
                } else {
                    p[i][j] = false;
                }
            }

        }
        return s.substring(min, max + 1);
    }
```

###### 注：

这里对于boolean矩阵进行赋值时采用列优先的方式，若采用行优先的方式，计算`p[i][j]`时会出现`p[i+1][j-1]`尚未赋值的情况

同时可以看出，二维数组p只有上三角部分得到利用，因此可以进一步优化空间，但处理会变得复杂

#### 837、新21点

##### 题目描述

爱丽丝参与一个大致基于纸牌游戏 “21点” 规则的游戏，描述如下：

爱丽丝以 0 分开始，并在她的得分少于 K 分时抽取数字。 抽取时，她从 [1, W] 的范围中随机获得一个整数作为分数进行累计，其中 W 是整数。 每次抽取都是独立的，其结果具有相同的概率。

当爱丽丝获得不少于 K 分时，她就停止抽取数字。 爱丽丝的分数不超过 N 的概率是多少？

示例 1：

```
输入：N = 10, K = 1, W = 10
输出：1.00000
```


说明：爱丽丝得到一张卡，然后停止。

示例 2：

```
输入：N = 6, K = 1, W = 10
输出：0.60000
```

说明：爱丽丝得到一张卡，然后停止。
在 W = 10 的 6 种可能下，她的得分不超过 N = 6 分。

示例 3：

```
输入：N = 21, K = 17, W = 10
输出：0.73278
```

提示：

    0 <= K <= N <= 10000
    1 <= W <= 10000
    如果答案与正确答案的误差不超过 10^-5，则该答案将被视为正确答案通过。
    此问题的判断限制时间已经减少。

##### 方法一 动态规划（官方）

###### 思路

> 爱丽丝获胜的概率只和下一轮开始前的得分有关，因此根据得分计算概率。
>
> 令 dp[x] 表示从得分为x 的情况开始游戏并且获胜的概率，目标是求 dp[0] 的值。
>
> 根据规则，当分数达到或超过 K时游戏结束，游戏结束时，如果分数不超过 N则获胜，如果分数超过 N 则失败。因此当 K≤x≤min⁡(N,K+W−1) 时有 dp[x]=1，当 x>min⁡(N,K+W−1) 时有 dp[x]=0。
>
> 为什么分界线是 min⁡(N,K+W−1)\min(N, K+W-1)min(N,K+W−1)？首先，只有在分数不超过 N 时才算获胜；其次，可以达到的最大分数为 K+W−1，即在最后一次抽取数字之前的分数为 K−1，并且抽到了 W。
>
> 当 0≤x<K 时，如何计算 dp[x] 的值？注意到每次在范围 `[1,W]` 内随机抽取一个整数，且每个整数被抽取到的概率相等，因此可以得到如下状态转移方程：
>
> dp[x]=（dp[x+1]+dp[x+2]+⋯+dp[x+W]）/ W
>

首先，在分数x大于等于k时，抽取已经停止，如果分数x的值小于N并且没有超出取值范围，则一定会获胜，即dp[i]=1；但当分数x的值大于N或者超出分数可能的范围时，此时一定会失败，dp[i]=0；

当目前分数x<k时，考虑下一次抽取，下一次抽取的可能性有W种，下一次得分为x+1,x+2......x+W，共W种情况；

所以**当前得分赢的概率为：下一次抽取各个情况赢的概率求和取平均，即dp[x]=（dp[x+1]+dp[x+2]+⋯+dp[x+W]）/ W**

###### 算法一

```java
public double new21Game(int N, int K, int W) {
    	if(K==0){
            return 1.0;
        }
        int l = Math.max(N, K + W - 1);
        double[] dp = new double[l + 1];
        int j = Math.min(N, K + W - 1);
        for (int i = l; i >= 0; i--) {
            if (i >= K && i <= j) {
                dp[i] = 1;
            } else if (i > j) {
                dp[i] = 0;
            } else {
                double sum = 0;
                for (int k = 1; k <= W; k++) {
                    sum += dp[i + k];
                }
                dp[i] = sum / W;
            }
        }
        return dp[0];
    }

```

运行超时，需要进一步优化

注意到dp的相邻项关系：

![image-20200603170110728](https://gitee.com/yanzixian/picBed/raw/master/img202005/20200603170113.png)

其中`0≤x<K−1`

因此可以得到新的状态转移方程：

![image-20200603170220856](https://gitee.com/yanzixian/picBed/raw/master/img202006/20200603170222.png)

其中`0≤x<K−1`

注意到上述状态转移方程中 x的取值范围，当 x=K−1 时不适用。因此对于 dp[K−1]的值，需要通过

![image-20200603170559953](https://gitee.com/yanzixian/picBed/raw/master/img202006/20200603170601.png)

计算得到。注意到只有当 `K≤x≤min⁡(N,K+W−1)` 时才有 dp[x]=1，因此

![image-20200603170701979](https://gitee.com/yanzixian/picBed/raw/master/img202006/20200603170703.png)

可在 O(1) 时间内计算得到 dp[K−1] 的值。

对于 dp[K−2] 到 dp[0] 的值，则可通过新的状态转移方程得到

###### 算法二

```java
class Solution {
    public double new21Game(int N, int K, int W) {
        if (K == 0) {
            return 1.0;
        }
        double[] dp = new double[K + W];
        for (int i = K; i <= N && i < K + W; i++) {
            dp[i] = 1.0;
        }
        dp[K - 1] = 1.0 * Math.min(N - K + 1, W) / W;
        for (int i = K - 2; i >= 0; i--) {
            dp[i] = dp[i + 1] - (dp[i + W + 1] - dp[i + 1]) / W;
        }
        return dp[0];
    }
}


```

#### 238、除自身以外数组的乘积

##### 题目描述

给你一个长度为 n 的整数数组 nums，其中 n > 1，返回输出数组 output ，其中 output[i] 等于 nums 中除 nums[i] 之外其余各元素的乘积。

```
示例:

输入: [1,2,3,4]
输出: [24,12,8,6]
```

提示：题目数据保证数组之中任意元素的全部前缀元素和后缀（甚至是整个数组）的乘积都在 32 位整数范围内。

说明: 请不要使用除法，且在 O(n) 时间复杂度内完成此题。

##### 方法一 暴力求解

###### 思路

求出所有数的乘积，该乘积除以nums[i]即得ouput[i]，但该方法使用了除法，不符合题目要求

##### 方法二 动态规划

###### 思路

新建两个数组pre和post，pre[i]表示nums[i]的全部前缀元素乘积，post[i]表示nums[i]的全部后缀元素乘积；其中第一个元素的全部前缀元素乘积为1，同样，最后一个元素的全部后缀元素乘积为1

可以得到状态转移方程为：

```
pre[i]=pre[i-1]*nums[i],post[i]=post[i+1]*nums[i]，其中0<i<len-1
pre[0]=1,post[len-1]=1;
```

###### 算法

```java
public int[] productExceptSelf(int[] nums) {
    int len=nums.length;
    int[] pre=new int[len];
    int[] post=new int[len];
    int[] result=new int[len];
    for(int i=0;i<len;i++){
        if(i==0){
            pre[i]=1;
            post[len-1-i]=1;
        }else{
            pre[i]=pre[i-1]*nums[i-1];
            post[len-1-i]=post[len-i]*nums[len-i];
        }
    }
    for(int i=0;i<len;i++){
        result[i]=pre[i]*post[i];
       
    }
    return result;
}
```

##### 方法三 动态规划（官方）

###### 思路

为了进一步优化空间，实现题目要求的O(1)空间复杂度，可以舍弃存储前缀元素积和后缀元素积的数组

由于输出数组不算在空间复杂度内，那么我们可以将 L 或 R 数组用输出数组来计算。先把输出数组当作 L 数组来计算，然后再动态构造 R 数组得到结果。让我们来看看基于这个思想的算法

###### 算法

> 初始化 answer 数组，对于给定索引 i，answer[i] 代表的是 i 左侧所有数字的乘积。
> 构造方式与之前相同，只是我们试图节省空间，先把 answer 作为方法二的 L 数组。
> 这种方法的唯一变化就是我们没有构造 R 数组。而是用一个遍历来跟踪右边元素的乘积。并更新数组 `answer[i]=answer[i]∗R`。然后 RRR 更新为 `R=R∗nums[i]`，其中变量 R 表示的就是索引右侧数字的乘积。

```java
class Solution {
    public int[] productExceptSelf(int[] nums) {
        int length = nums.length;
        int[] answer = new int[length];

        // answer[i] 表示索引 i 左侧所有元素的乘积
        // 因为索引为 '0' 的元素左侧没有元素， 所以 answer[0] = 1
        answer[0] = 1;
        for (int i = 1; i < length; i++) {
            answer[i] = nums[i - 1] * answer[i - 1];
        }

        // R 为右侧所有元素的乘积
        // 刚开始右边没有元素，所以 R = 1
        int R = 1;
        for (int i = length - 1; i >= 0; i--) {
            // 对于索引 i，左边的乘积为 answer[i]，右边的乘积为 R
            answer[i] = answer[i] * R;
            // R 需要包含右边所有的乘积，所以计算下一个结果时需要将当前值乘到 R 上
            R *= nums[i];
        }
        return answer;
    }
}
```

