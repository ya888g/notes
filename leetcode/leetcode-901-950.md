# LeetCode 901 - 950

### 901. Online Stock Span

Write a class `StockSpanner` which collects daily price quotes for some stock, and returns the *span* of that stock's price for the current day.

The span of the stock's price today is defined as the maximum number of consecutive days (starting from today and going backwards) for which the price of the stock was less than or equal to today's price.

For example, if the price of a stock over the next 7 days were `[100, 80, 60, 70, 60, 75, 85]`, then the stock spans would be `[1, 1, 1, 2, 1, 4, 6]`.

Solution: stack

```cpp
class StockSpanner {
public:
    vector<pair<int, int>> history;
    int cnt;
    StockSpanner() {
        history.clear();
        cnt = 0;
    }
    
    int next(int price) {
        ++cnt;
        while (!history.empty() && history.back().first <= price) history.pop_back();
        int ret = history.empty()? cnt: cnt - history.back().second;
        history.push_back(make_pair(price, cnt));
        return ret;
    }
};
```

### 902. Numbers At Most N Given Digit Set

We have a **sorted** set of digits `D`, a non-empty subset of `{'1','2','3','4','5','6','7','8','9'}`.  Now, we write numbers using these digits, using each digit as many times as we want. Return the number of positive integers that can be written (using the digits of `D`) that are less than or equal to `N`.

Example:

```
Input: D = ["1","3","5","7"], N = 100
Output: 20 ([1, 3, 5, 7, 11, 13, 15, 17, 31, 33, 35, 37, 51, 53, 55, 57, 71, 73, 75, 77])
```

Solution: 先算比N少一位数字的时候所有的可能性，即sum(power(D.size(), i) for i in range(1, N))，然后再从第一位到最后一位考虑可能的组合情况，一定要背

```cpp
int helper(vector<char> vocab, string s) {
   int cnt = 0, nv = vocab.size(), ns = s.size();
   if (!ns) return 1;
   for (int i = 0; i < nv; ++i) {
      if (vocab[i] > s[0]) break;
      else if (vocab[i] < s[0]) cnt += pow(nv, ns - 1);
      else if (vocab[i] == s[0]) {
         s.erase(0, 1);
         cnt += helper(vocab, s);
         break;
      }
   }
   return cnt;
}
int atMostNGivenDigitSet (vector<string>& D, int N) {
    int n = to_string(N).size(), d = D.size(), ans = 0;
    // step 1: count digits less than n length
    for (int i = 1; i < n; ++i) ans += pow(d, i);
    // step 2: recursively count possible combinations
    vector<char> vocab(d);
    for (int i = 0; i < d; ++i) vocab[i] = D[i][0];
    ans += helper(vocab, to_string(N));
    return ans;
}
```

### 904. Fruit Into Baskets

In a row of trees, the `i`-th tree produces fruit with type `tree[i]`.

You **start at any tree of your choice**, then repeatedly perform the following steps:

1. Add one piece of fruit from this tree to your baskets.  If you cannot, stop.
2. Move to the next tree to the right of the current tree.  If there is no tree to the right, stop.

Note that you do not have any choice after the initial choice of starting tree: you must perform step 1, then step 2, then back to step 1, then step 2, and so on until you stop.

You have two baskets, and each basket can carry any quantity of fruit, but you want each basket to only carry one type of fruit each. What is the total amount of fruit you can collect with this procedure?

Example:

```
Input: [0,1,2,2]
Output: 3 (We can collect [1,2,2]. If we started at the first tree, we would only collect [0, 1].)
```

Solution: 遍历一遍即可

```cpp
int totalFruit(vector<int>& tree) {
    int basket1 = -1, basket2 = -1;
    int begin = 0, end = 0;
    int n = tree.size(), res = -1;
    for (end = 0; end < n; end++) {
        if (basket1 == -1) { basket1 = tree[end]; continue;}
        if (basket1 == tree[end]) continue;
        if (basket2 == -1) { basket2 = tree[end]; continue;}
        if (basket2 == tree[end]) continue;
        res = max(res, end - begin);
        begin = end - 1;
        while (begin >= 1 && tree[begin] == tree[begin-1]) --begin;
        basket1 = tree[begin];
        basket2 = tree[end];
    }
    res = max(res, end - begin);
    return res;
}
```

### 905. Sort Array By Parity

Given an array `A` of non-negative integers, return an array consisting of all the even elements of `A`, followed by all the odd elements of `A`. You may return any answer array that satisfies this condition.

Example:

```
Input: [3,1,2,4]
Output: [2,4,3,1]
```

Solution: 遍历一遍即可，也可以用类似quick sort的遍历法实现in-place，一定要背

```cpp
vector<int> sortArrayByParity(vector<int>& A) {
    int i = 0, j = A.size() - 1;
    while (i < j) {
        while (A[i] % 2 == 0 && i < j) ++i;
        while (A[j] % 2 == 1 && i < j) --j;
        if (i < j) swap(A[i], A[j]);
    }
    return A;
}
```

### 907. Sum of Subarray Minimums

Given an array of integers `A`, find the sum of `min(B)`, where `B` ranges over every (contiguous) subarray of `A`. Since the answer may be large, **return the answer modulo 10^9 + 7.**

Example:

```
Input: [3,1,2,4]
Output: 17 (Subarrays are [3], [1], [2], [4], [3,1], [1,2], [2,4], [3,1,2], [1,2,4], [3,1,2,4]. 
Minimums are 3, 1, 2, 4, 1, 1, 2, 1, 1, 1.  Sum is 17.)
```

Solution: 两个stack，每个分别表示到目前位置为止左右的<最小值，连续长度>，这样问题可以转化为对每个位置坐左右延展，一定要背

```cpp
int sumSubarrayMins(vector<int> A) {
    int res = 0, n = A.size(), mod = 1e9 + 7;
    vector<int> left(n), right(n);
    stack<pair<int, int>> s1, s2;
    for (int i = 0; i < n; ++i) {
        int count = 1;
        while (!s1.empty() && s1.top().first > A[i]) {
            count += s1.top().second;
            s1.pop();
        }
        s1.push({A[i], count});
        left[i] = count;
    }
    for (int i = n - 1; i >= 0; --i) {
        int count = 1;
        while (!s2.empty() && s2.top().first >= A[i]) {
            count += s2.top().second;
            s2.pop();
        }
        s2.push({A[i], count});
        right[i] = count;
    }
    for (int i = 0; i < n; ++i)
        res = (res + A[i] * left[i] * right[i]) % mod;
    return res;
}
```

### 908. Smallest Range I

Given an array `A` of integers, for each integer `A[i]` we may choose any `x` with `-K <= x <= K`, and add `x` to `A[i]`. After this process, we have some array `B`. Return the smallest possible difference between the maximum value of `B` and the minimum value of `B`.

Example:

```
Input: A = [1,3,6], K = 3
Output: 0 (B = [3,3,3] or B = [4,4,4])
```

Solution: 遍历一遍找最大最小值即可

```cpp
int smallestRangeI(vector<int>& A, int K) {
    int max = A[0], min = A[0];
    for (int i = 1; i < A.size(); ++i) {
        if (A[i] < min) min = A[i];
        if (A[i] > max) max = A[i];
    }
    return (max - min) > 2 * K ? (max - min - 2 * K) : 0;      
}
```

### 909. Snakes and Ladders

On an N x N `board`, the numbers from `1` to `N*N` are written *boustrophedonically* **starting from the bottom left of the board**, and alternating direction each row.  You start on square `1` of the board (which is always in the last row and first column). For example, for a 6 x 6 board, the numbers are written as follows:

![](../.gitbook/assets3/snakes.png)

 Each move, starting from square `x`, consists of the following:

- You choose a destination square ``S``with number ``x+1``, ``x+2``, ``x+3``, ``x+4``, ``x+5`` or ``x+6``, ,, provided this number is ``<= N*N``. (This choice simulates the result of a standard 6-sided die roll: i.e., there are always **at most 6 destinations, regardless of the size of the board**.)
- If `S` has a snake or ladder, you move to the destination of that snake or ladder.  Otherwise, you move to `S`.

A board square on row `r` and column `c` has a "snake or ladder" if `board[r][c] != -1`.  The destination of that snake or ladder is `board[r][c]`.

Note that you only take a snake or ladder at most once per move: if the destination to a snake or ladder is the start of another snake or ladder, you do **not** continue moving.  (For example, if the board is `[[4,-1],[-1,3]]`, and on the first move your destination square is `2`, then you finish your first move at `3`, because you do **not** continue moving to `4`.)

Return the least number of moves required to reach square N*N.  If it is not possible, return `-1`.

Example:

```
Input: [
    [-1,-1,-1,-1,-1,-1],
    [-1,-1,-1,-1,-1,-1],
    [-1,-1,-1,-1,-1,-1],
    [-1,35,-1,-1,13,-1],
    [-1,-1,-1,-1,-1,-1],
    [-1,15,-1,-1,-1,-1]
]
Output: 4 (At the beginning, you start at square 1 [at row 5, column 0].
You decide to move to square 2, and must take the ladder to square 15.
You then decide to move to square 17 (row 3, column 5), and must take the snake to square 13.
You then decide to move to square 14, and must take the ladder to square 35.
You then decide to move to square 36, ending the game.
It can be shown that you need at least 4 moves to reach the N*N-th square, so the answer is 4.)
```

Solution: bfs

```cpp
pair<int, int> translate(int i, int n){
    int r = ceil(i * 1.0 / n);
    bool lr = ((r % 2) == 1);
    int c = (i % n) == 0? n - 1: (i % n) - 1;
    if (!lr) c = n - 1 - c;
    return make_pair(n - r, c);
}

int snakesAndLadders(vector<vector<int>>& board) {
    int n = board.size();
    queue<int> poses;
    vector<bool> vis(n*n + 1, false);
    poses.push(1);
    int qsize, cnt = 0;
    while (!poses.empty()){
        ++cnt;
        qsize = poses.size();
        while (qsize--) {
            int front = poses.front();
            poses.pop();
            vis[front] = true;
            for (int i = front + 1; i <= front + 6; ++i) {
                if (i == n*n) return cnt;
                auto new_pos = translate(i, n);
                int new_val = board[new_pos.first][new_pos.second];
                int next_pos = new_val == -1? i: new_val;
                if (vis[next_pos]) continue;
                if (next_pos == n*n) return cnt;
                poses.push(next_pos);
            }
        }
    }
    return -1;
}
```

### 910. Smallest Range II

Given an array `A` of integers, for each integer `A[i]` we need to choose **either x = -K or x = K**, and add `x` to `A[i]`. After this process, we have some array `B`. Return the smallest possible difference between the maximum value of `B` and the minimum value of `B`.

Example:

```
Input: A = [1,3,6], K = 3
Output: 3 (B = [4,6,3])
```

Solution: sort一遍，然后遍历算最小差，一定要背

```cpp
int smallestRangeII(vector<int>& A, int K) {
    sort(A.begin(), A.end());
    int res = A[A.size() - 1] - A[0];
    int left = A[0] + K, right = A[A.size() - 1] - K;
    for (int i = 0; i < A.size() - 1; i++) {
        int maxi = max(A[i] + K, right), mini = min(left, A[i + 1] - K);
        res = min(res, maxi - mini);
    }
    return res;
}
```

### 911. Online Election

In an election, the `i`-th vote was cast for `persons[i]` at time `times[i]`. Now, we would like to implement the following query function: `TopVotedCandidate.q(int t)` will return the number of the person that was leading the election at time `t`.   Votes cast at time `t` will count towards our query.  In the case of a tie, the most recent vote (among tied candidates) wins.

Example:

```
Input: ["TopVotedCandidate","q","q","q","q","q","q"], [[[0,1,1,0,0,1,0],[0,5,10,15,20,25,30]],[3],[12],[25],[15],[24],[8]]
Output: [null,0,1,1,0,0,1]
(At time 3, the votes are [0], and 0 is leading.
At time 12, the votes are [0,1,1], and 1 is leading.
At time 25, the votes are [0,1,1,0,0,1], and 1 is leading (as ties go to the most recent vote.)
This continues for 3 more queries at time 15, 24, and 8.)
```

Solution: 预处理一下即可，不要想复杂：对于每个时间算目前最高频vote，然后额外存一下时间，用来二分查找pos，一定要背

```cpp
class TopVotedCandidate {
public:
    vector<int> ordered_persons, sorted_times;
    int n;
    TopVotedCandidate(vector<int> persons, vector<int> times) {
        n = persons.size();
        vector<int> bins(n, 0), wins(n, 0);
        int curmax = 0, curmaxpos = 0;
        for (int i = 0; i < n; ++i){
            ++bins[persons[i]];
            if (bins[persons[i]] >= curmax) {
                curmaxpos = persons[i];
                curmax = bins[persons[i]];
            }
            wins[i] = curmaxpos;
        }
        sorted_times = times;
        ordered_persons = wins;
    }
    
    int q(int t) {
        int pos = lower_bound(sorted_times.begin(), sorted_times.end(), t) - sorted_times.begin();
        if (sorted_times[pos] > t || pos == n) --pos;
        return ordered_persons[pos];
    }
};
```

### 912. Sort an Array

Given an array of integers `nums`, sort the array in ascending order.

Example:

```
Input: [5,2,3,1]
Output: [1,2,3,5]
```

Solution: 原题可能想着可以用桶排序解决小整数，但实际上直接排序即可

```cpp
vector<int> sortArray(vector<int>& nums) {
    vector<int> ret(nums);
    sort(ret.begin(), ret.end());
    return ret;
}
```

### 914. X of a Kind in a Deck of Cards

In a deck of cards, each card has an integer written on it.

Return `true` if and only if you can choose `X >= 2` such that it is possible to split the entire deck into 1 or more groups of cards, where:

- Each group has exactly `X` cards.
- All the cards in each group have the same integer.

Example:

```
Input: [1,1,2,2,2,2]
Output: true ([1,1],[2,2],[2,2])
```

Solution: hashmap + gcd

```cpp
int gcd (const int & a, const int & b) {
    return !b ? a: gcd(b, a % b);
}
bool hasGroupsSizeX(vector<int>& deck) {        
    unordered_map<int, int> count;
    int res = 0;
    for (const int & i: deck) ++count[i];
    for (const auto & i: count) res = gcd(i.second, res);
    return res > 1;
}
```

### 915. Partition Array into Disjoint Intervals

Given an array `A`, partition it into two (contiguous) subarrays `left` and `right` so that:

- Every element in `left` is less than or equal to every element in `right`.
- `left` and `right` are non-empty.
- `left` has the smallest possible size.

Return the **length** of `left` after such a partitioning.  It is guaranteed that such a partitioning exists.

Example:

```
Input: [5,0,3,8,6]
Output: 3 (left = [5,0,3], right = [8,6])
```

Solution: 左右两端遍历一遍，然后比较即可。一般来说需要考虑左右关系的题都可以左边右边各来一遍，如907，一定要理解和背

```cpp
int partitionDisjoint(vector<int>& A) {
    vector<int> maxs(A.size()), mins(A.size());
    maxs[0] = A[0], mins[A.size()-1] = A.back();
    for (int i = 1; i < A.size(); ++i) maxs[i] = max(maxs[i-1], A[i]);   
    for (int i = A.size() - 2; i >= 0; --i) mins[i] = min(mins[i+1], A[i]);
    for (int i = 0; i < A.size() -1 ; ++i)  if (maxs[i] <= mins[i+1]) return i + 1;
    return -1;
}
```

### 916. Word Subsets

Given two arrays `A` and `B` of lowercase words, we say that word `b` is a subset of word `a` if every letter in `b` occurs in `a`, **including multiplicity**.  For example, `"wrr"` is a subset of `"warrior"`, but is not a subset of `"world"`.

Now say a word `a` from `A` is *universal* if for every `b` in `B`, `b` is a subset of `a`. Return a list of all universal words in `A`.  You can return the words in any order

Example:

```
Input: A = ["amazon","apple","facebook","google","leetcode"], B = ["ec","oc","ceo"]
Output: ["facebook","leetcode"]
```

Solution: 遍历求所有B词的max频率是否能满足就可以了

```cpp
vector<string> wordSubsets(vector<string>& A, vector<string>& B) {
    vector<vector<int>> adict(A.size(), vector<int>(26, 0));
    vector<int> bvec(26, 0), blocal(26, 0);
    for (int i = 0; i < B.size(); ++i) {
        blocal = vector<int>(26, 0);
        for (int j = 0; j < B[i].length(); ++j) ++blocal[B[i][j] - 'a'];
        for (int j = 0; j < 26; ++j) bvec[j] = max(bvec[j], blocal[j]);
    }

    vector<string> ret;
    for (auto s: A) {
        vector<int> avec(26, 0);
        for (int j = 0; j < s.length(); ++j) ++avec[s[j] - 'a'];
        bool met = true;
        for (int j = 0; j < 26 && met; ++j) {
            if (bvec[j] > avec[j]) {
                met = false;
                break;
            }
        }
        if (met) ret.push_back(s);
    }
    return ret;
}
```

### 917. Reverse Only Letters

Given a string `S`, return the "reversed" string where all characters that are not a letter stay in the same place, and all letters reverse their positions.

Example:

```
Input: "Test1ng-Leet=code-Q!"
Output: "Qedo1ct-eeLg=ntse-T!"
```

Solution: 双指针交换

```cpp
bool isLetter(char c) {
    return (c - 'a' >= 0 && 'z' - c >= 0) || (c - 'A' >= 0 && 'Z' - c >= 0);
}
string reverseOnlyLetters(string S) {
    int n = S.length();
    if (n <= 1) return S;
    int i = 0, j = n - 1;
    while (i < j) {
        while(i < j && !isLetter(S[i])) ++i;
        while(i < j && !isLetter(S[j])) --j;
        if (i == j) break;
        swap(S[i++], S[j--]);
    }
    return S;
}
```

### 918. Maximum Sum Circular Subarray

Given a **circular array** **C** of integers represented by `A`, find the maximum possible sum of a non-empty subarray of **C**. Here, a *circular array* means the end of the array connects to the beginning of the array. Also, a subarray may only include each element of the fixed buffer `A` at most once.

Example:

```
Input: [3,-1,2,-1]
Output: 4 (Subarray [2,-1,3] has maximum sum 2 + (-1) + 3 = 4)
```

Solution: 重复一遍+滑动窗口，或者记录最大和以及最小和连续串，结果为最大和连续串，和总和减去最小和连续串（即circular的最大和连续串）两者的最大值。这里有一个小技巧，算当前最大（小）和积分的时候，可以用当前总和减去当前最小（大）和积分，一定要背

```cpp
int maxSubarraySumCircular(vector<int>& A) {
    int total = 0, cur_max_sum = 0, cur_min_sum = 0, max_subsum = INT_MIN, min_subsum = INT_MAX;
    for (int i = 0; i < A.size(); i++){
        total += A[i];
        min_subsum = min(min_subsum, total - cur_max_sum);
        max_subsum = max(max_subsum, total - cur_min_sum);
        cur_max_sum = max(cur_max_sum, total);
        cur_min_sum = min(cur_min_sum, total);
    }
    return max_subsum > 0 ? max(max_subsum, total - min_subsum) : max_subsum;
}
```

### 919. Complete Binary Tree Inserter

A *complete* binary tree is a binary tree in which every level, except possibly the last, is completely filled, and all nodes are as far left as possible.

Write a data structure `CBTInserter` that is initialized with a complete binary tree and supports the following operations:

- `CBTInserter(TreeNode root)` initializes the data structure on a given tree with head node `root`;
- `CBTInserter.insert(int v)` will insert a `TreeNode` into the tree with value `node.val = v` so that the tree remains complete, **and returns the value of the parent of the inserted TreeNode**;
- `CBTInserter.get_root()` will return the head node of the tree.

Example:

```
Input: inputs = ["CBTInserter","insert","insert","get_root"], inputs = [[[1,2,3,4,5,6]],[7],[8],[]]
Output: [null,3,4,[1,2,3,4,5,6,7,8]]
```

Solution: 相比于用node的奇偶来遍历，不如直接把node存在一个vector里，每次找好父亲位置直接push_back，一定要背

```cpp
class CBTInserter {
private:
    vector<TreeNode*> trees = {NULL};
public:
    CBTInserter(TreeNode* root) {
        if (!root) return;
        queue<TreeNode*> q;
        q.push(root);
        while (!q.empty()) {
            auto node = q.front();
            q.pop();
            trees.push_back(node);
            if (node->left) q.push(node->left);
            if (node->right) q.push(node->right);
        }
    }
    
    int insert(int v) {
        int id = trees.size(), parent = id / 2;
        TreeNode* node = new TreeNode(v);
        trees.push_back(node);
        if (id % 2 == 0) trees[parent]->left = node;
        else trees[parent]->right = node;
        return trees[parent]->val;
    }
    
    TreeNode* get_root() {
        return trees[1];
    }
};
```

### 921. Minimum Add to Make Parentheses Valid

Given a string `S` of `'('` and `')'` parentheses, we add the minimum number of parentheses ( `'('` or `')'`, and in any positions ) so that the resulting parentheses string is valid.

Given a parentheses string, return the minimum number of parentheses we must add to make the resulting string valid.

Example:

```
Input: "()))(("
Output: 4 ("((()))(())")
```

Solution: stack，但是可以用变量代替stack，一定要背

```cpp
int minAddToMakeValid(string S) {
    int n = S.length();
    if (n <= 1) return n;
    int left = 0, right = 0;
    for (auto c: S) {
        if (c == '(') ++left;
        else if (left) --left;
        else ++right;
    }
    return left + right;
}
```

### 922. Sort Array By Parity II

Given an array `A` of non-negative integers, half of the integers in A are odd, and half of the integers are even. Sort the array so that whenever `A[i]` is odd, `i` is odd; and whenever `A[i]` is even, `i` is even. You may return any answer array that satisfies this condition.

Example:

```
Input: [4,2,5,7]
Output: [4,5,2,7] or [4,7,2,5] or [2,5,4,7] or [2,7,4,5] 
```

Solution: 正常遍历

```cpp
vector<int> sortArrayByParityII(vector<int>& A) {
    int n = A.size();
    if (n < 2) return vector<int>(A);
    vector<int> odds, evens, ret;
    for (auto i: A) {
        if (i % 2) odds.push_back(i);
        else evens.push_back(i);
    }
    for (int i = 0; i < n/2; ++i){
        ret.push_back(evens.back());
        ret.push_back(odds.back());
        evens.pop_back();
        odds.pop_back();
    }      
    return ret;
}
```

### 923. 3Sum With Multiplicity

Given an integer array `A`, and an integer `target`, return the number of tuples `i, j, k`  such that `i < j < k`and `A[i] + A[j] + A[k] == target`. As the answer can be very large, return it **modulo 10^9 + 7**.  Each number is within range``0 <= A[i] <= 100``. 

Example:

```
Input: A = [1,1,2,2,2,2], target = 5
Output: 12 (A[i] = 1, A[j] = A[k] = 2 occurs 12 times)
```

Solution: 统计次数，之后和3Sum相同，需要注意组合数的计算方法

```cpp
int threeSumMulti(vector<int>& A, int target) {
    vector<long long> stats(101, 0);
    for (int n : A) ++stats[n];
    int M = 1e9 + 7;
    long long result = 0;
    for (int i = 0; i <= 100; ++i) {
        for (int j = i; j <= 100; ++j) {
            int k = target - i -j;
            if (k < 0 || k > 100) continue;
            if (i == j && j == k) result = (result + stats[i]*(stats[i]-1)*(stats[i]-2)/6) % M;
            else if (i == j) result = (result + stats[k]*stats[i]*(stats[i]-1)/2) % M;
            else if(j < k) result = (result + stats[i]*stats[j]*stats[k]) % M;
        }
    }
    return result;
}
```

### 925. Long Pressed Name

Your friend is typing his `name` into a keyboard.  Sometimes, when typing a character, the key might get *long pressed*, and the character will be typed 1 or more times. You examine the `typed` characters of the keyboard.  Return `True` if it is possible that it was your friends name, with some characters (possibly none) being long pressed.

Example:

```
Input: name = "alex", typed = "aaleex"
Output: true
```

Solution: 正常遍历

```cpp
bool isLongPressedName(string name, string typed) {
    int na = name.size(), nb = typed.size();
    if (nb < na) return false;
    if (nb == na) return name == typed;
    int i = 0, j = 0, prev;
    while (j < nb) {
        if (name[i] != typed[j]) {
            if (i != 0 && prev == typed[j]) ++j;
            else return false;
        } else {
            prev = name[i];
            ++i;
            ++j;
        }
    }
    return i == na;
}
```

### 926. Flip String to Monotone Increasing

A string of `'0'`s and `'1'`s is *monotone increasing* if it consists of some number of `'0'`s (possibly 0), followed by some number of `'1'`s (also possibly 0.) We are given a string `S` of `'0'`s and `'1'`s, and we may flip any `'0'` to a `'1'` or a `'1'` to a `'0'`. Return the minimum number of flips to make `S` monotone increasing.

Example:

```
Input: "010110"
Output: 2 ("011111" or "000111")
```

Solution: 可以左右各一遍brute force，也可以一遍完成

```cpp
int minFlipsMonoIncr(string s) {
    int zero = 0, one = 0, n = s.length();
    for(int i = 0; i < n; ++i) {
        if (s[i] == '0') {
            one = 1 + min(zero, one);
        } else {
            one = min(zero, one);
            ++zero;
        }
    }
    return min(zero, one);
}
```

### 929. Unique Email Addresses

Every email consists of a local name and a domain name, separated by the @ sign. For example, in `alice@leetcode.com`, `alice` is the local name, and `leetcode.com` is the domain name. Besides lowercase letters, these emails may contain `'.'`s or `'+'`s.

If you add periods (`'.'`) between some characters in the **local name** part of an email address, mail sent there will be forwarded to the same address without dots in the local name.  For example, `"alice.z@leetcode.com"` and `"alicez@leetcode.com"` forward to the same email address.

If you add a plus (`'+'`) in the **local name**, everything after the first plus sign will be **ignored**. This allows certain emails to be filtered, for example `m.y+name@email.com` will be forwarded to `my@email.com`. It is possible to use both of these rules at the same time.

Given a list of `emails`, we send one email to each address in the list.  How many different addresses actually receive mails? 

Example:

```
Input: [
  "test.email+alex@leetcode.com",
  "test.e.mail+bob.cathy@leetcode.com",
  "testemail+david@lee.tcode.com"
]
Output: 2
```

Solution: hashset

```cpp
int numUniqueEmails(vector<string>& emails) {
    unordered_set<string> hashset;
    for (auto & s : emails) {
        string temp;
        for (int i = 0; i < s.length(); ++i) {
            if (s[i] == '@') {
                temp += s.substr(i);
                break;
            }
            if (s[i] == '+') {
                while (s[i+1] != '@') ++i;
            } else if (s[i] != '.') {
                temp += s[i];
            }
        }
        hashset.insert(temp);
    }
    return hashset.size();
}
```

### 930. Binary Subarrays With Sum

In an array `A` of `0`s and `1`s, how many **non-empty** subarrays have sum `S`?

Example:

```
Input: A = [1,0,1,0,1], S = 2
Output: 4 ([1,0,1], [1,0,1,0], [0,1,0,1], [1,0,1])
```

Solution: 双指针或者积分和统计频率，这里注意积分和的问题如果需要位置则可以记录<积分和，位置>，如果需要频率则可以记录<积分和, 频率>，一定要背

```cpp
int numSubarraysWithSum(vector<int>& A, int S) {
    vector<int> prefixSum(A.size() + 1, 0);
    int sum = 0;
    int result = 0;
    for (int i = 0; i < A.size(); ++i) {
        sum += A[i];
        int target = sum - S;
        if (sum == S) ++result;
        if (target >= 0 && target < A.size()) {
            result += prefixSum[target];        
        }
        ++prefixSum[sum];
    }
    return result;
}
```

### 931. Minimum Falling Path Sum

Given a **square** array of integers `A`, we want the **minimum** sum of a *falling path* through `A`. A falling path starts at any element in the first row, and chooses one element from each row.  The next row's choice must be in a column that is different from the previous row's column by at most one.

Example:

```
Input: [
    [1,2,3],
    [4,5,6],
    [7,8,9]
]
Output: 12 ([1,4,7])
```

Solution: dp

```cpp
int minFallingPathSum(vector<vector<int>>& A) {
    int m = A.size(), n = A[0].size(), sum = INT_MAX;
    vector<vector<int>> dp(m, vector<int>(n));
    for (int j = 0; j < n; ++j) dp[0][j] = A[0][j];
    for (int i = 1; i < m; ++i) {
        dp[i][0] = min(dp[i-1][0], dp[i-1][1]) + A[i][0];
        dp[i][n-1] = min(dp[i-1][n-2], dp[i-1][n-1]) + A[i][n-1];
        for (int j = 1; j < n - 1; ++j)
            dp[i][j] = min(min(dp[i-1][j-1], dp[i-1][j]), dp[i-1][j+1])+ A[i][j];
    }
    for (auto i: dp[m-1]) sum = min(sum, i);
    return sum;
}
```

### 932. Beautiful Array

For some fixed `N`, an array `A` is *beautiful* if it is a permutation of the integers `1, 2, ..., N`, such that for every `i < j`, there is **no** `k` with `i < k < j` such that `A[k] * 2 = A[i] + A[j]`. Given `N`, return **any** beautiful array `A`. It is guaranteed that one exists.

Example:

```
Input: 5
Output: [3,1,2,5,4]
```

Solution: 数学题

```cpp
vector<int> beautifulArray(int N) {
    vector<int> res{1};
    while (res.size() < N) {
        vector<int> tmp;
        for (int & i: res) if (i * 2 - 1 <= N) tmp.push_back(i * 2 - 1);
        for (int & i: res) if (i * 2 <= N) tmp.push_back(i * 2);
        res.swap(tmp);
    }
    return res;
}
```

### 933. Number of Recent Calls

Write a class `RecentCounter` to count recent requests. It has only one method: `ping(int t)`, where t represents some time in milliseconds. Return the number of `ping`s that have been made from 3000 milliseconds ago until now. Any ping with time in `[t - 3000, t]` will count, including the current ping. It is guaranteed that every call to `ping` uses a strictly larger value of `t` than before.

Example:

```
Input: inputs = ["RecentCounter","ping","ping","ping","ping"], inputs = [[],[1],[100],[3001],[3002]]
Output: [null,1,2,3,3]
```

Solution: deque

```cpp
class RecentCounter {
public:
    RecentCounter() {}
    int ping(int t) {
        while (!dq.empty() && dq.front() + 3000 < t) dq.pop_front();
        dq.push_back(t);
        return dq.size();
    }
private:
    deque<int> dq;
};
```

### 934. Shortest Bridge

In a given 2D binary array `A`, there are two islands.  (An island is a 4-directionally connected group of `1`s not connected to any other 1s.) Now, we may change `0`s to `1`s so as to connect the two islands together to form 1 island. Return the smallest number of `0`s that must be flipped.  (It is guaranteed that the answer is at least 1.)

Example:

```
Input: [
    [1,1,1,1,1],
    [1,0,0,0,1],
    [1,0,1,0,1],
    [1,0,0,0,1],
    [1,1,1,1,1]
]
Output: 1
```

Solution: 先dfs找到一个岛，然后bfs找最短距离

```cpp
void dfs(queue<pair<int, int>>& points, vector<vector<int>>& A, int m, int n, int i, int j) {
    if (i < 0 || j < 0 || i == m || j == n || A[i][j] == 2) return;
    if (A[i][j] == 0) {
        points.push({i, j});
        return;
    }
    A[i][j] = 2;
    dfs(points, A, m, n, i-1, j);
    dfs(points, A, m, n, i+1, j);
    dfs(points, A, m, n, i, j-1);
    dfs(points, A, m, n, i, j+1);
}

int shortestBridge(vector<vector<int>>& A) {
    int m = A.size(), n = A[0].size();
    queue<pair<int, int>> points;
    bool flipped = false;
    for (int i = 0; i < m; ++i) {
        if (flipped) break;
        for (int j = 0; j < n; ++j) {
            if (A[i][j] == 1) {
                dfs(points, A, m, n, i, j);
                flipped = true;
                break;
            }
        }

    }
    int i, j, idx, jdy;
    int dx[4] = {-1, 1, 0, 0}, dy[4] = {0, 0, 1, -1};
    int level = 0;
    while (!points.empty()){
        ++level;
        int n_points = points.size();
        while (n_points--) {
            auto point = points.front();
            points.pop();
            i = point.first, j = point.second;
            for (int k = 0; k < 4; ++k) {
                idx = i + dx[k], jdy = j + dy[k];
                if (idx >= 0 && jdy >= 0 && idx < m && jdy < n) {
                    if (A[idx][jdy] == 2) continue;
                    if (A[idx][jdy] == 1) return level;
                    points.push({idx, jdy});
                    A[idx][jdy] = 2;
                }
            }
        }
    }
    return 0;
}
```

### 935. Knight Dialer

A chess knight can move as indicated in the chess diagram below:

![](../.gitbook/assets3/knight_keypad.png)

This time, we place our chess knight on any numbered key of a phone pad (indicated above), and the knight makes `N-1` hops.  Each hop must be from one key to another numbered key. Each time it lands on a key (including the initial placement of the knight), it presses the number of that key, pressing `N` digits total.

How many distinct numbers can you dial in this manner? Since the answer may be large, **output the answer modulo 10^9 + 7**.

Example:

```
Input: 3
Output: 46
```

Solution: 这种state machine转移状态的题都可以用dp，一定要背

```cpp
const int MOD = 1e9 + 7;
int knightDialer( int N ){
    vector<long> cur(10, 1), next(10, 1);
    for (int i = 2; i <= N; ++i){
        next[0] = (cur[4] + cur[6]) % MOD;
        next[1] = (cur[6] + cur[8]) % MOD;
        next[2] = (cur[7] + cur[9]) % MOD;
        next[3] = (cur[4] + cur[8]) % MOD;
        next[4] = (cur[0] + cur[3] + cur[9]) % MOD;
        next[5] = 0;
        next[6] = (cur[0] + cur[1] + cur[7]) % MOD;
        next[7] = (cur[2] + cur[6]) % MOD;
        next[8] = (cur[1] + cur[3]) % MOD;
        next[9] = (cur[2] + cur[4]) % MOD;
        cur.swap(next);
    }
    return accumulate(cur.begin(), cur.end(), 0L) % MOD;
}
```

### 937. Reorder Log Files

You have an array of `logs`.  Each log is a space delimited string of words. For each log, the first word in each log is an alphanumeric *identifier*.  Then, either:

- Each word after the identifier will consist only of lowercase letters, or;
- Each word after the identifier will consist only of digits.

We will call these two varieties of logs *letter-logs* and *digit-logs*.  It is guaranteed that each log has at least one word after its identifier. Reorder the logs so that all of the letter-logs come before any digit-log.  The letter-logs are ordered lexicographically ignoring identifier, with the identifier used in case of ties.  The digit-logs should be put in their original order. Return the final order of the logs.

Example:

```
Input: ["a1 9 2 3 1","g1 act car","zo4 4 7","ab1 off key dog","a8 act zoo"]
Output: ["g1 act car","a8 act zoo","ab1 off key dog","a1 9 2 3 1","zo4 4 7"]
```

Solution: brute force，注意在sort的时候，可以把identifier和后面内容分成pair的两部分，就可以不用map了

```cpp
vector<string> reorderLogFiles(vector<string>& logs) {
    vector<string> digit_logs, res;
    vector<pair<string, string>> letter_logs;
    for (const string & s: logs) {
        int pos = s.find(" ") + 1;
        if (s[pos] - '0' >= 0 && s[pos] - '0' <= 9) {
            digit_logs.push_back(s);
        } else letter_logs.push_back({s.substr(pos), s.substr(0, pos)});
    }
    sort(letter_logs.begin(), letter_logs.end());
    for (auto p: letter_logs) {
        res.emplace_back(p.second + p.first);
    }
    res.insert(res.end(), digit_logs.begin(), digit_logs.end());
    return res;
}
```

### 938. Range Sum of BST

Given the `root` node of a binary search tree, return the sum of values of all nodes with value between `L` and `R`(inclusive). The binary search tree is guaranteed to have unique values.

Example:

```
Input: root = [10,5,15,3,7,13,18,1,null,6], L = 6, R = 10
Output: 23
```

Solution: divide，比中序遍历要快，一定要背

```cpp
int rangeSumBST(TreeNode* root, int L, int R) {
    if (!root) return 0;
    if (root->val > R) return rangeSumBST(root->left, L, R);
    if (root->val < L) return rangeSumBST(root->right, L, R);
    return root->val + rangeSumBST(root->left, L, R) + rangeSumBST(root->right, L, R);
}
```

### 939. Minimum Area Rectangle

Given a set of points in the xy-plane, determine the minimum area of a rectangle formed from these points, with sides parallel to the x and y axes. If there isn't any rectangle, return 0.

Example:

```
Input: [[1,1],[1,3],[3,1],[3,3],[2,2]]
Output: 4
```

Solution: 先按照行列存一个map<row, vector\<col\>>，然后sort vector使得每行的col index按顺序排列；之后对row做两个for loop，loop内对两个row从左到右分别找col相等的时候的所有位置，就可以算最小面积了，一定要背

```cpp
int minAreaRect(vector<vector<int>>& points) {
    map<int, vector<int>> rows;
    for (auto& p : points) rows[p[0]].push_back(p[1]);        
    for (auto& it : rows) sort(it.second.begin(), it.second.end());        
    int ans = INT_MAX;
    for (auto& it1 : rows) {
        if (it1.second.size() <= 1) continue;
        for (auto& it2 : rows) {
            if (it2.first <= it1.first) continue;
            if (it2.second.size() <= 1) continue;                
            int i = 0, j = 0, last = -1;
            while (i < it1.second.size() && j < it2.second.size()) {
                if (it1.second[i] < it2.second[j]) {
                    ++i;
                } else if (it1.second[i] > it2.second[j]) {
                    ++j;
                } else {
                    if (last >= 0) ans = min(ans, (it2.first - it1.first) * (it1.second[i] - last));
                    last = it1.second[i];
                    ++i;
                    ++j;
                }
            }
        }
    }
    return ans == INT_MAX? 0: ans;
}
```

### 940. Distinct Subsequences II

Given a string `S`, count the number of distinct, non-empty subsequences of `S` . Since the result may be large, **return the answer modulo 10^9 + 7**.

Example:

```
Input: "abc"
Output: 7 (["a", "b", "c", "ab", "ac", "bc", "abc"])
```

Solution: 用一个dp记录每个字符最后出现的位置，另一个dp算subsequence和，一定要理解和背

```cpp
int distinctSubseqII(string S) {
    int n = S.length(); 
    vector<int> last(26, -1); 
    vector<long long> dp(n+1, 1);
    int pos, mod = 1e9 + 7;
    for (int i = 1; i <= n; ++i)  { 
        pos = S[i-1] - 'a';
        dp[i] = 2 * dp[i-1]; 
        if (last[pos] != -1) dp[i] -= dp[last[pos]]; 
        if (dp[i] < 0) dp[i] += mod;
        dp[i] %= mod;
        last[pos] = i-1; 
    } 
    return dp[n] - 1; 
}
```

### 941. Valid Mountain Array

Given an array `A` of integers, return `true` if and only if it is a *valid mountain array*.

Recall that A is a mountain array if and only if `A.length >= 3` and there exists some ``0 < i < A.length - 1`` such that ``A[0] < A[1] < ... A[i-1] < A[i]`` and ``A[i] > A[i+1] > ... > A[B.length - 1]``.

Example:

```
Input: [0,3,2,1]
Output: true
```

Solution: 遍历一遍就好

```cpp
bool validMountainArray(vector<int>& A) {
    int n = A.size();
    if (n < 3) return false;
    int i = 0;
    while (i < n - 1 && A[i] < A[i+1]) ++i;
    if (i == n - 1 || i == 0) return false;
    while (i < n - 1 && A[i] > A[i+1]) ++i;
    return i == n - 1;
}
```

### 942. DI String Match

Given a string `S` that **only** contains "I" (increase) or "D" (decrease), let `N = S.length`. Return **any** permutation `A` of `[0, 1, ..., N]` such that for all `i = 0, ..., N-1`:

- If `S[i] == "I"`, then `A[i] < A[i+1]`
- If `S[i] == "D"`, then `A[i] > A[i+1]`

Example:

```
Input: "DDI"
Output: [3,2,0,1]
```

Solution: 遍历一遍就好，十分巧妙，一定要背

```cpp
vector<int> diStringMatch(string S) {
    int n = S.length();
    int maxint = n, minint = 0;
    vector<int> res;
    for (int i = 0; i < n; ++i) {
        if (S[i] == 'I') res.push_back(minint++);
        else res.push_back(maxint--);
    }
    res.push_back(maxint);
    return res;
}
```

### 944. Delete Columns to Make Sorted

We are given an array `A` of `N` lowercase letter strings, all of the same length. Now, we may choose any set of deletion indices, and for each string, we delete all the characters in those indices. Suppose we chose a set of deletion indices `D` such that after deletions, each remaining column in A is in **non-decreasing** sorted order. Return the minimum possible value of `D.length`.

Example:

```
Input: ["cba","daf","ghi"]
Output: 1 (D = {1}, remaining columns are ["c","d","g"] and ["a","f","i"])
```

Solution: 按列遍历即可

```cpp
int minDeletionSize(vector<string>& A) {
    int res = 0;
    for (int j = 0; j < A[0].size(); ++j) {
        for (int i = 0; i < A.size() - 1; ++i) {
            if (A[i][j] > A[i+1][j]) {
                ++res;
                break;
            }
        }
    }
    return res;
}
```

### 945. Minimum Increment to Make Array Unique

Given an array of integers A, a *move* consists of choosing any `A[i]`, and incrementing it by `1`. Return the least number of moves to make every value in `A` unique.

Example:

```
Input: [3,2,1,2,1,7]
Output: 6 (After 6 moves, the array could be [3, 4, 1, 2, 5, 7])
```

Example: 排序后遍历一遍，处理十分巧妙，一定要背

```cpp
int minIncrementForUnique(vector<int>& A, int res = 0) {
    sort(A.begin(), A.end());
    for (int i = 1; i < A.size(); ++i) {
        res += max(0, A[i-1] - A[i] + 1);
        A[i] += max(0, A[i-1] - A[i] + 1);
    }
    return res;
}
```

### 946. Validate Stack Sequences

Given two sequences `pushed` and `popped` **with distinct values **and same length, return `true` if and only if this could have been the result of a sequence of push and pop operations on an initially empty stack. 

Example:

```
Input: pushed = [1,2,3,4,5], popped = [4,5,3,2,1]
Output: true
```

Solution: stack + 贪心

```cpp
bool validateStackSequences(vector<int>& pushed, vector<int>& popped) {
    int count = 0;
    stack<int> st;
    for (const int & i: pushed) {
        st.push(i);
        while (!st.empty() && st.top() == popped[count]) {
            st.pop();
            ++count;
        }
    }
    return st.empty();
}
```

### 947. Most Stones Removed with Same Row or Column

On a 2D plane, we place stones at some integer coordinate points.  Each coordinate point may have at most one stone. A *move* consists of removing a stone that shares a column or row with another stone on the grid. What is the largest possible number of moves we can make?

Example:

```
Input: stones = [[0,0],[0,1],[1,0],[1,2],[2,1],[2,2]]
Output: 5
```

Solution: 转化成并查集问题，一定要背

```cpp
int removeStones(vector<vector<int>>& stones) {
    for (int i = 0; i < stones.size(); ++i)
        uni(stones[i][0], ~stones[i][1]);
    return stones.size() - islands;
}

unordered_map<int, int> f;
int islands = 0;

int find(int x) {
    if (!f.count(x)) f[x] = x, ++islands;
    if (x != f[x]) f[x] = find(f[x]);
    return f[x];
}

void uni(int x, int y) {
    x = find(x), y = find(y);
    if (x != y) f[x] = y, --islands;
}
```

### 948. Bag of Tokens

You have an initial power `P`, an initial score of `0` points, and a bag of tokens. Each token can be used at most once, has a value `token[i]`, and has potentially two ways to use it.

- If we have at least `token[i]` power, we may play the token face up, losing `token[i]` power, and gaining `1` point.
- If we have at least `1` point, we may play the token face down, gaining `token[i]` power, and losing `1` point.

Return the largest number of points we can have after playing any number of tokens.

Example:

```
Input: tokens = [100,200,300,400], P = 200
Output: 2
```

Solution: 双指针

```cpp
int bagOfTokensScore(vector<int>& tokens, int P) {
    int current_point = 0, max_point = 0, i = 0, j = tokens.size() - 1;
    sort(tokens.begin(), tokens.end());
    while (i <= j && current_point >= 0) {
        if (tokens[i] <= P) {
            P -= tokens[i++];
            max_point = max(max_point, ++current_point);
        } else {
            P += tokens[j--];
            --current_point;
        }
    }
    return max_point;
}
```

### 949. Largest Time for Given Digits

Given an array of 4 digits, return the largest 24 hour time that can be made. If no valid time can be made, return an empty string.

Example:

```
Input: [1,2,3,4]
Output: "23:41"
```

Solution: next permutation

```cpp
string largestTimeFromDigits(vector<int>& A) {
    string res = "";
    sort(A.begin(), A.end());
    do {
        if((A[0] == 2 && A[1] <= 3 || A[0] < 2) && A[2] <= 5) {
            string temp = to_string(A[0]) + to_string(A[1]) + ":" + to_string(A[2]) + to_string(A[3]);
            if (temp > res) res = temp;
        }       
    } while (next_permutation(A.begin(), A.end()));
    return res;
}
```

### 950. Reveal Cards In Increasing Order

In a deck of cards, every card has a unique integer.  You can order the deck in any order you want.

Initially, all the cards start face down (unrevealed) in one deck. Now, you do the following steps repeatedly, until all cards are revealed:

1. Take the top card of the deck, reveal it, and take it out of the deck.
2. If there are still cards in the deck, put the next top card of the deck at the bottom of the deck.
3. If there are still unrevealed cards, go back to step 1.  Otherwise, stop.

Return an ordering of the deck that would reveal the cards in **increasing order.** The first entry in the answer is considered to be the top of the deck.

Example:

```
Input: [17,13,11,2,3,5,7]
Output: [2,13,3,11,5,17,7] (We get the deck in the order [17,13,11,2,3,5,7] and reorder it to  [2,13,3,11,5,17,7], where 2 is the top of the deck.
We reveal 2, and move 13 to the bottom.  The deck is now [3,11,5,17,7,13].
We reveal 3, and move 11 to the bottom.  The deck is now [5,17,7,13,11].
We reveal 5, and move 17 to the bottom.  The deck is now [7,13,11,17].
We reveal 7, and move 13 to the bottom.  The deck is now [11,17,13].
We reveal 11, and move 17 to the bottom.  The deck is now [13,17].
We reveal 13, and move 17 to the bottom.  The deck is now [17].
We reveal 17.
Since all the cards revealed are in increasing order, the answer is correct.)
```

Solution: 倒着处理一遍即可，因为涉及到前后端操作，用deque最方便

```cpp
vector<int> deckRevealedIncreasing(vector<int>& deck) {
    deque<int> dq;
    vector<int> sorted(deck);
    sort(sorted.begin(), sorted.end(), greater<int>());
    for (const int& i: sorted) {
        if (dq.empty()) dq.push_back(i);
        else {
            dq.push_front(dq.back());
            dq.pop_back();
            dq.push_front(i);
        }
    }
    return vector<int>(dq.begin(), dq.end());
}
```