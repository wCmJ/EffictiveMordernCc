## 11.3
### 4

### 10

### 23

### 25

### LFU
```cpp
class LeastFrequentlyUsed {
public:
  LeastFrequentlyUsed(int capacity) : capacity_(capacity) {}
  int get(int key) {
    // if existed return the value of key, otherwise return -1
    if (key_it_value_cnt.count(key) == 0) {
      return -1;
    }
    int cnt = key_it_value_cnt[key].second.second;
    auto it = key_it_value_cnt[key].first;
    cnt_keys_[cnt].erase(it);
    cnt_keys_[cnt + 1].push_front(key);
    key_it_value_cnt[key].first = cnt_keys_[cnt + 1].begin();
    key_it_value_cnt[key].second.second = cnt + 1;
  }
  void put(int key, int value) {
    // if key existed then modify the value of it, if not existed, insert
    // (key, value) to container. when container is full remove the least
    // frequently item
    // TODO: corner case
    if (capacity_ == 0) {
      return;
    }
    if (key_it_value_cnt.count(key) == 0) {
      // not existed
      if (key_it_value_cnt.size() >= capacity_) {
        for (auto &node : cnt_keys_) {
          if (!node.second.empty()) {
            int del_key = node.second.back();
            key_it_value_cnt.erase(del_key);
            node.second.pop_back();
            break;
          }
        }
      }
      cnt_keys_[1].push_front(key);
      key_it_value_cnt[key].first = cnt_keys_[1].begin();
      key_it_value_cnt[key].second.first = value;
      key_it_value_cnt[key].second.second = 1;
    } else {
      get(key);
      key_it_value_cnt[key].second.first = value;
    }
  }

private:
  map<int, list<int>> cnt_keys_;
  unordered_map<int, std::pair<list<int>::iterator, std::pair<int, int>>>
      key_it_value_cnt;
  int capacity_;
};
```

### LRU
```cpp
class LeastRecentlyUsed {
public:
  LeastRecentlyUsed(int capacity) : capacity_(capacity) {}
  int get(int key) {
    if (key_it_value_.count(key) == 0) {
      return -1;
    }
    keys_.erase(key_it_value_[key].first);
    keys_.push_front(key);
    key_it_value_[key].first = keys_.begin();
    return key_it_value_[key].second;
  }

  void put(int key, int value) {
    if (0 == capacity_) {
      return;
    }

    if (-1 != get(key)) {
      key_it_value_[key].second = value;
      return;
    }

    if (key_it_value_.size() == capacity_) {
      key_it_value_.erase(keys_.back());
      keys_.pop_back();
    }
    keys_.push_front(key);
    key_it_value_[key].first = keys_.begin();
    key_it_value_[key].second = value;
  }

private:
  list<int> keys_;
  unordered_map<int, std::pair<list<int>::iterator, int>> key_it_value_;
  int capacity_;
};


```
## 11.4
### 30
```cpp
// 串联所有单词的子串
// time: O(n*m), space: O(m)
void CheckIndexValid(const string &s, int i,
                     unordered_map<string, int> word_cnt, vector<int> &ans,
                     int word_len, int word_len_sum) {
  int start = i, end = i, count = word_cnt.size();
  while (start + word_len_sum <= s.size()) {
    if (end - start == word_len_sum) {
      if (count == 0) {
        ans.push_back(start);
      }
      string st = s.substr(start, word_len);
      start += word_len;
      if (word_cnt.count(st) && ++word_cnt[st] == 1) {
        count++;
      }
    } else {
      while (end - start < word_len_sum) {
        string tmp = s.substr(end, word_len);
        end += word_len;
        if (word_cnt.count(tmp) == 0) {
          while (start != end) {
            string st = s.substr(start, word_len);
            start += word_len;
            if (word_cnt.count(st) && ++word_cnt[st] == 1) {
              ++count;
            }
          }
          break;
        } else {
          while (word_cnt[tmp] == 0) {
            string st = s.substr(start, word_len);
            start += word_len;
            if (word_cnt.count(st) && ++word_cnt[st] == 1) {
              ++count;
            }
            if (st == tmp) {
              break;
            }
          }
          if (--word_cnt[tmp] == 0) {
            --count;
          }
        }
      }
    }
  }
}

vector<int> findSubstring(string s, vector<string> &words) {
  unordered_map<string, int> word_cnt;
  int word_len = 0, word_len_sum = 0;
  vector<int> ans;
  for (auto &word : words) {
    word_cnt[word]++;
    word_len = word.size();
    word_len_sum += word_len;
  }
  for (int i = 0; i < word_len; ++i) {
    CheckIndexValid(s, i, word_cnt, ans, word_len, word_len_sum);
  }
  return ans;
}
```
### 32
```cpp
// 最长有效括号
// time: O(n), space: O(n)
int lc_32_longestValidParentheses(string s) {
  int ans = 0, len = s.size();
  vector<int> dp(len, 0);
  for (int i = 1; i < len; ++i) {
    if (s[i] == ')') {
      int last = i - 1 - dp[i - 1];
      if (last >= 0 && s[last] == '(') {
        dp[i] = 2 + dp[i - 1] + (last - 1 >= 0 ? dp[last - 1] : 0);
      }
      ans = max(ans, dp[i]);
    }
  }
  return ans;
}
```

### 37
```cpp
// 解数独
// time: 9^n space: O(1)
bool rowValid(vector<vector<char>> &board, int i, int j) {
  for (int k = 0; k < 9; ++k) {
    if (k != j && board[i][k] == board[i][j]) {
      return false;
    }
  }
  return true;
}

bool colValid(vector<vector<char>> &board, int i, int j) {
  for (int k = 0; k < 9; ++k) {
    if (k != i && board[k][j] == board[i][j]) {
      return false;
    }
  }
  return true;
}

bool boxValid(vector<vector<char>> &board, int i, int j) {
  int si = 3 * (i / 3), sj = 3 * (j / 3);
  for (int r = si; r < si + 3; ++r) {
    for (int c = sj; c < sj + 3; ++c) {
      if (r != i && c != j && board[r][c] == board[i][j]) {
        return false;
      }
    }
  }
  return true;
}

std::pair<int, int> solveSudokuNext(vector<vector<char>> &board, int i, int j) {
  std::pair<int, int> ans{0, 0};
  for (int r = i; r < 9; ++r) {
    for (int c = 0; c < 9; ++c) {
      if (board[r][c] == '.') {
        ans.first = r;
        ans.second = c;
        return ans;
      }
    }
  }
  return ans;
}

bool solveSudokuImpl(vector<vector<char>> &board, int i, int j) {
  for (char c = '1'; c <= '9'; ++c) {
    board[i][j] = c;
    if (rowValid(board, i, j) && colValid(board, i, j) &&
        boxValid(board, i, j)) {
      auto next = solveSudokuNext(board, i, j);
      if ((next.first == 0 && next.second == 0) ||
          solveSudokuImpl(board, next.first, next.second)) {
        return true;
      }
    }
  }
  board[i][j] = '.';
  return false;
}

void lc_37_solveSudoku(vector<vector<char>> &board) {
  // 9 x 9
  for (int i = 0; i < 9; ++i) {
    for (int j = 0; j < 9; ++j) {
      if (board[i][j] == '.') {
        solveSudokuImpl(board, i, j);
        break;
      }
    }
  }
}
```
### 41
```cpp
// 第一个缺失的正数
// time: O(n), space: O(1)
int lc_41_firstMissingPositive(vector<int> &nums) {
  int len = nums.size();
  for (int i = 0; i < len;) {
    if (nums[i] != i + 1 && nums[i] >= 1 && nums[i] <= len &&
        nums[nums[i] - 1] != nums[i]) {
      swap(nums[i], nums[nums[i] - 1]);
    } else {
      ++i;
    }
  }
  int ans = len + 1;
  for (int i = 0; i < len; ++i) {
    if (nums[i] != i + 1) {
      ans = i + 1;
      break;
    }
  }
  return ans;
}
```

### 44
```cpp
// 通配符匹配：？any single char，* any string
// time: O(n*m) space: O(n*m)
bool lc_44_isMatch(string s, string p) {
  if (p.empty())
    return s.empty();
  int ls = s.size(), lp = p.size();
  vector<vector<int>> dp(ls + 1, vector<int>(lp + 1, 0));
  dp[0][0] = true;
  for (int i = 0; i < lp; ++i) {
    if (p[i] == '*') {
      dp[0][i + 1] = dp[0][i];
    } else {
      break;
    }
  }

  for (int i = 0; i < ls; ++i) {
    for (int j = 0; j < lp; ++j) {
      if (s[i] == p[j] || '?' == p[j]) {
        dp[i + 1][j + 1] = dp[i][j];
      } else {
        if ('*' == p[j]) {
          dp[i + 1][j + 1] = dp[i + 1][j] || dp[i][j] || dp[i][j + 1];
        }
      }
    }
  }
  return dp[ls][lp];
}
```
### 51
```cpp
// N皇后
// time: O(n^n) space: O(n)
int solveNQueensIndex(const string &s) {
  int ans = -1;
  for (int i = 0; i < s.size(); ++i) {
    if (s[i] == 'Q') {
      ans = i;
      break;
    }
  }
  return ans;
}

bool solveNQueensValid(vector<string> &loc, int i) {
  int li = solveNQueensIndex(loc[i]);
  for (int s = 0; s < i; ++s) {
    int ls = solveNQueensIndex(loc[s]);
    if (ls == li || (abs(ls - li) == abs(i - s))) {
      return false;
    }
  }
  return true;
}

void solveNQueensImpl(vector<string> &loc, int i, vector<vector<string>> &ans) {
  if (i >= loc.size()) {
    ans.push_back(loc);
    return;
  }
  for (int l = 0; l < loc[i].size(); ++l) {
    loc[i][l] = 'Q';
    if (solveNQueensValid(loc, i)) {
      solveNQueensImpl(loc, i + 1, ans);
    }
    loc[i][l] = '.';
  }
}

vector<vector<string>> lc_51_solveNQueens(int n) {
  vector<vector<string>> ans;
  vector<string> loc(n, string(n, '.'));
  solveNQueensImpl(loc, 0, ans);
  return ans;
}
```
### 52
```cpp
// N皇后2
// time: O(n^n) space: O(n)
bool QValid(vector<int> &Q, int index){
  for(int i = 0;i<index;++i){
    if(Q[index] == Q[i] || (abs(Q[index] - Q[i]) == abs(index - i))){
      return false;
    }
  }
  return true;
}


void dfs(vector<int> &Q, int index, int &ans){
  if(index >= Q.size()){
    ++ans;
    return;
  }
  for(int v = 1;v<=Q.size();++v){
    Q[index] = v;
    if(QValid(Q, index)){
      dfs(Q, index + 1, ans);
    }
  }
  Q[index] = 0;
}


int totalNQueens(int n) {
  int ans = 0;
  vector<int> Q(n, 0);
  dfs(Q, 0, ans);
  return ans;
}
```
### 60
```cpp
// 排列序列，1 <= n <= 9, 1 <= k <= n!
// time: O(n^2) space: O(n)
int oneToN(int n){
  int ans = 1;
  for(int i = 2;i<=n;++i){
    ans *= i;
  }
  return ans;
}

string getPermutation(int n, int k) {
  string ans(n, ' ');
  vector<int> used(n + 1, false);
  for(int i = 0;i<n;++i){
    int count = oneToN(n - 1 - i);
    int times = (k - 1)/count + 1;
    int val = 0;
    for(int j = 1;j< n + 1;++j){
      if(used[j] == 0){
        times--;
      }
      if(times == 0){
        used[j] = true;
        ans[i] = (char)('0' + j);
        break;
      }
    }
    k -= count * ((k - 1)/count);
  }
  return ans;
}

```
### 65
```cpp
// 有效数字按顺序
// 1. 小数或整数；2. 一个'e'或'E'，后面跟着一个整数
/* 小数分为以下几个部分：
  1. 一个符号字符（可选）；
  2. 下述格式之一:
    - 至少一个数字，后面跟着一个'.'
    - 至少一位数字，后面跟着一个点 '.' ，后面再跟着至少一位数字
    - 一个点 '.' ，后面跟着至少一位数字
*/
/* 整数按顺序可以分为以下几个部分：
  1. 一个符号字符（'+' 或 '-'）可选
  2. 至少一位数字
*/
/*
部分有效数字列举如下：
["2", "0089", "-0.1", "+3.14", "4.", "-.9", "2e10", "-90E3", "3e+7", "+6e-1", "53.5e93", "-123.456e789"]

部分无效数字列举如下：
["abc", "1a", "1e", "e3", "99e2.5", "--6", "-+3", "95a54e53"]
*/


bool isInt(const string &s, int start, int end) {
  if (start > end)
    return false;
  int dot_count = 0, num_count = 0;
  if (s[start] == '+' || s[start] == '-') {
    ++start;
  }
  while (start <= end) {
    if (s[start] >= '0' && s[start] <= '9') {
      ++num_count;
    } else {
      return false;
    }
    ++start;
  }
  return num_count > 0;
}

bool isFloat(const string &s, int start, int end) {
  if (start > end)
    return false;
  if (s[start] == '+' || s[start] == '-') {
    ++start;
  }
  int dot_count = 0, num_count = 0;
  while (start <= end) {
    if (s[start] == '.') {
      ++dot_count;
    } else if (s[start] >= '0' && s[start] <= '9') {
      ++num_count;
    } else {
      return false;
    }
    ++start;
  }
  return dot_count == 1 && num_count > 0;
}




bool isNumber(string s) {
  // check invalid character
  int ls = s.size();
  std::pair<int,int> e_times_firstLoc{0, -1};
  for(int i = 0;i<ls;++i){
    if(s[i] == 'e' || s[i] == 'E'){
      e_times_firstLoc.first++;
      e_times_firstLoc.second = i;
    }
    else if(s[i] != '+' && s[i] != '-' && s[i] != '.' && (s[i] < '0' || s[i] > '9')){
      return false;
    }
  }
  if(e_times_firstLoc.first > 1){
    return false;
  }
  else if(e_times_firstLoc.first == 1){
    return (isInt(s, 0, e_times_firstLoc.second - 1) || isFloat(s, 0, e_times_firstLoc.second - 1)) && isInt(s, e_times_firstLoc.second + 1, ls - 1);
  }
  return isInt(s, 0, ls - 1) || isFloat(s, 0, ls - 1);
}
```

### 68
```cpp
// 文本左右对齐
vector<string> fullJustify(vector<string> &words, int maxWidth) {
    // 11.4
    {
      vector<string> ans;
      for (int i = 0; i < words.size();) {
        int end = i, cur_len = 0, word_lens = 0;
        while (end < words.size()) {
          if (words[end].size() + cur_len <= maxWidth) {
            cur_len += words[end].size() + 1;
            word_lens += words[end].size();
            ++end;
          } else {
            break;
          }
        }
        if (cur_len == 0) {
          return vector<string>();
        }
        std::cout << "end: " << end << std::endl;
        int word_cnt = end - i;
        int space_cnt = maxWidth - word_lens;
        if (word_cnt == 1) {
          ans.push_back(words[i] + string(space_cnt, ' '));
        } else {
          if (end == words.size()) {
            string tmp;
            for (int j = i; j < end - 1; ++j) {
              tmp += words[j] + " ";
            }
            tmp += words[end - 1];
            if (maxWidth > tmp.size()) {
              tmp += string(maxWidth - tmp.size(), ' ');
            }
            ans.push_back(tmp);
          } else {
            int common_cnt = space_cnt / (word_cnt - 1);
            int left_cnt = space_cnt % (word_cnt - 1);
            string tmp;
            for (int j = i; j < end - 1; ++j) {
              tmp += words[j] + (common_cnt > 0 ? string(common_cnt, ' ') : "");
              tmp += (j - i + 1 <= left_cnt ? " " : "");
            }
            tmp += words[end - 1];
            ans.push_back(tmp);
          }
        }
        i = end;
      }
      return ans;
    }
  }





```
## 11.5
### 72
```cpp
// insert, delete, replace
int minDistance(string word1, string word2) {
  int l1 = word1.size(), l2 = word2.size();
  if(l1 == 0)return l2;
  if(l2 == 0)return l1;
  vector<vector<int>> dp(l1 + 1, vector<int>(l2 + 1, 0));
  for(int i = 0;i<l2;++i){
    dp[0][i + 1] = dp[0][i] + 1;
  }
  for(int i = 0;i<l1;++i){
    dp[i+1][0] = dp[i][0] + 1;
  }
  
  for(int i = 0;i < l1;++i){
    for(int j = 0;j < l2;++j){
      if(word1[i] == word2[j]){
        dp[i+1][j+1] = dp[i][j];
      }
      else{
        dp[i+1][j+1] = min(dp[i][j], min(dp[i][j+1], dp[i+1][j])) + 1;
      }
    }
  }
  return dp[l1][l2];
}

```

### 76
```cpp
// 最小覆盖子串
string minWindow(string s, string t) {
  int ls = s.size(), lt = t.size();
  if(lt > ls)return "";
  unordered_map<char, int> cs;
  for(auto c: t){
    cs[c]++;
  }
  int count = cs.size(), start = 0, end = 0;
  int ans_start = -1, ans_len = 0;
  while(end < ls){
    if(cs.count(s[end]) && --cs[s[end]] == 0){
      --count;
    }
    ++end;
    while(count == 0 && start < end){
      if(ans_len == 0 || end - start < ans_len){
        ans_start = start;
        ans_len = end - start;
      }
      
      if(cs.count(s[start]) && ++cs[s[start]] == 1){
        ++count;
      }
      ++start;
    }
  }
  return ans_len == 0 ? "" : s.substr(ans_start, ans_len);
}

```

### 84
```cpp
// 柱状图中最大的矩形
int largestRectangleArea(vector<int>& heights) {
  int len = heights.size();
  if(len == 0)return 0;
  stack<int> inc;
  int ans = 0;
  for(int i = 0;i<len;++i){
    while(!inc.empty() && heights[inc.top()] >= heights[i]){
      int h = inc.top();
      inc.pop();
      int left = inc.empty() ? -1 : inc.top();
      ans = max(ans, heights[h] * (i - left - 1));
    }
    inc.push(i);
  }
  while(!inc.empty()){
    int h = inc.top();
    inc.pop();
    int left = inc.empty() ? -1 : inc.top();
    ans = max(ans, heights[h] * (len - left - 1));  
  }  
  return ans;
}

```
### 85
```cpp
// 最大矩形
int maximalRectangle(vector<vector<char>>& matrix) {
  // method2, use dp
  { 
    if(matrix.size() == 0 || matrix[0].size() == 0){
      return 0;
    }
    int m = matrix.size(), n = matrix[0].size();
    int ans = 0;
    vector<vector<std::pair<int,int>>> dp(m, vector<std::pair<int,int>>(n, {0,0}));
    for(int i = 0;i<m;++i){
      for(int j = 0;j<n;++j){
        if(matrix[i][j] == '1'){
          int left = 0, up = 0;
          if(i > 0 && j > 0){
            left = min(dp[i-1][j].first - 1, min(dp[i][j-1].first, dp[i-1][j-1].first));
            up = min(dp[i][j-1].second - 1, min(dp[i-1][j-1].second, dp[i-1][j].second));
          }
          else if(i > 0){
            up = dp[i-1][j].second;
          }
          else if(j > 0){
            left = dp[i][j-1].first;
          }
          dp[i][j].first = 1 + left;
          dp[i][j].second = 1 + right;          
        }
      }    
    }
  
  }

  // method1, optimize to 84
  if(matrix.size() == 0 || matrix[0].size() == 0){
    return 0;
  }
  int m = matrix.size(), n = matrix[0].size();
  int ans = 0;
  vector<int> row_val(n, 0);
  for(int i = 0;i<m;++i){
    for(int j = 0;j<n;++j){
      if(matrix[i][j] == '0'){
        row_val[j] = 0;
      }
      else{
        row_val[j]++;
      }
    }
    ans = max(ans, largestRectangleArea(row_val));
  }
  return ans;
}

```
### 87
```cpp
// 扰乱字符串
bool isSimilar(const string &s1, const string &s2) {
  int cnt[26] = {0};
  for(auto c: s1){
    cnt[(int)(c - 'a')]++;
  }
  for(auto c: s2){
    if(--cnt[(int)(c - 'a')] < 0){
      return false;
    }
  }
  return true;
}

bool isScramble(string s1, string s2) {
  // 11.5
  {
    int len = s1.size();
    vector<vector<vector<int>>> dp(len + 1, vector<vector<int>>(len, vector<int>(len, false)));
    for(int i = 0;i<len;++i){
      for(int j = 0;j<len;++j){
        dp[0][i][j] = true;
      }
    }
    for(int l = 1;l<=len;++l){
      for(int i = 0;i<=len-l;++i){
        for(int j = 0;j<=len-l;++j){
          if(s1.substr(i, l) == s2.substr(j, l)){
            dp[l][i][j] = true;
          }
          else{
            for(int tl = 1;tl < l;++tl){
              dp[l][i][j] = dp[l][i][j] || (dp[tl][i][j] && dp[l - tl][i + tl][j + tl]);
              dp[l][i][j] = dp[l][i][j] || (dp[tl][i][j + l - tl] && dp[l - tl][i + tl][j]);
            }          
          }
        }
      }
    }
    return dp[len][0][0];
  }


  // TLE
  if(s1.size() != s2.size())return false;
  if(s1 == s2)return true;
  int len = s1.size();
  for(int l = 1;l<len;++l){
    string s1_left = s1.substr(0, l), s1_right = s1.substr(l);
    string s2_left = s2.substr(0, l), s2_right = s2.substr(l);
    if(isSimilar(s1_left, s2_left) && isSimilar(s1_right, s2_right) && isScramble(s1_left, s2_left) && isScramble(s1_right, s2_right)){
      return true;
    }
    s2_right = s2.substr(0, len - l);
    s2_left = s2.substr(len - l);
    if(isSimilar(s1_left, s2_left) && isSimilar(s1_right, s2_right) && isScramble(s1_left, s2_left) && isScramble(s1_right, s2_right)){
      return true;
    }    
  }
  return false;
}
```

## 11.6
### 115
```cpp
// 不同的子序列
// key-point：dp，结果溢出
int numDistinct(string s, string t) {
    int ls = s.size(), lt = t.size();
    vector<vector<long long>> dp(lt + 1, vector<long long>(ls + 1, 0));
    //init
    for(int i = 0;i<=ls;++i){
        dp[0][i] = 1;
    }
    for(int i = 0;i<lt;++i){
        for(int j = i;j<ls;++j){
            dp[i + 1][j + 1] = dp[i + 1][j];
            if(t[i] == s[j] && dp[i+1][j+1] + dp[i][j] < INT_MAX){
                dp[i+1][j+1] += dp[i][j];
            }
        }
    }
    return dp[lt][ls];
}
```

## 11.15
### 求解子数组和为奇数的个数
```cpp
int numberOfOddSubarray(vector<int> &nums){
  //method1
  {
    int presum = 0, odd_count = 0, even_count = 1;
    for(auto n: nums){
      presum += n;
      if(presum % 2 == 0){
        ++even_count;
      }
      else{
        ++odd_count;
      }
    }
    return even_count * odd_count;  
  }
  
  // method2
  {
    int len = nums.size();
    vector<int> odd(len, 0), even(len, 0);
    if(nums[0] % 2 == 0){
      even[0] = 1;
    }
    else{
      odd[0] = 1;
    }
    int ans = odd[0];
    for(int i = 1;i<len;++i){
      if(nums[i] % 2 == 0){
        even[i] = even[i-1] + 1;
        odd[i] = odd[i-1];
      }
      else{
        even[i] = odd[i-1];
        odd[i] = even[i-1] + 1;
      }
      ans += odd[i];
    }      
  }
}

```

## Graph
### 课程表 bool & vector<int>
```cpp
bool canFinish(int numCourses, vector<vector<int>>& prerequisites) {
  /*
    1. 构建邻接矩阵，表明被依赖关系
    2. 计算入度，即依赖于其他节点的数目
    3. 获取入度为0的key，同时更新该key被依赖的节点的入度，直到不存在入度为0的key为止  
  */
  vector<vector<int>> beDepends(numCourses);
  vector<int> degree(numCourses, 0);
  for(auto &node: prerequisites){
    beDepends[node[1]].push_back(node[0]);
    degree[node[0]]++;
  }
  deque<int> finished;
  for(int i = 0;i<numCourses;++i){
    if(degree[i]==0){
      finished.push_back(i);
    }
  }
  int ans = 0;
  while(!finished.empty()){
    int len = finished.size();
    ans += len;
    for(int i = 0;i<len;++i){
      int key = finished.front();
      finished.pop_front();
      //ans.push_back(key);
      for(auto n: beDepends[key]){
        if(--degree[n] == 0){
          finished.push_back(n);
        }
      }
    }  
  }
  return ans == numCourses;  
}  
```

### 重新安排行程
```cpp
// 第一次使用map时，会忽略重复的ticket，改成multimap  
bool dfs(unordered_map<string, multimap<string, bool>> &start_end, const string& loc, vector<string> &ans, int total){
  if(ans.size() == total){
    return true;
  }
  for(auto &node: start_end[loc]){
    if(!node.second){
      node.second = true;
      ans.push_back(node.first);
      if(dfs(start_end, node.first, ans, total)){
        return true;
      }
      node.second = false;
      ans.pop_back();
    }
  }
  return ans.size() == total;  
}  
  
vector<string> findItinerary(vector<vector<string>> &tickets){
  unordered_map<string, multimap<string, bool>> start_ends;
  for(auto &t: tickets){
    start_ends[t[0]].insert({t[1], false});
  }
  vector<string> ans{"JFK"};
  dfs(start_ends, "JFK", ans, tickets.size() + 1);
  return ans;  
}  
  

```


## 149. 直线上最多的点数
```cpp
  
  
  
  
  
  
```  

































