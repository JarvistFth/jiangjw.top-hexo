---
title: 【字符串】 - 1
date: 2021-07-18 23:02:32
tags: [leetcode,字符串]
categories: 
- leetcode
- 字符串
---
LeetCode 字符串相关。
<!---more--->

## 5. 最长回文子串
https://leetcode-cn.com/problems/longest-palindromic-substring/

中心扩展法，每次取一个字符作为回文串的中点，然后分别比较他们的左边字符串和右边字符串。



```C++
class Solution {
public:
    string longestPalindrome(string s) {
        if(s.empty()){
            return "";
        }

        int begin = 0;
        int maxlen = 0;

        for(int i=0; i<s.size(); i++){
            int l=i,r=i;
            while(l>=0 && r<s.size()){
                if(s[l] == s[r]){
                    if(r-l+1 > maxlen){
                        begin = l;
                        maxlen = r-l+1;
                    }
                    l--;r++;
                }else{
                    break;
                }
            }
            l=i;r=i+1;
            while(l>=0 && r<s.size()){
                if(s[l] == s[r]){
                    if(r-l+1 > maxlen){
                        begin = l;
                        maxlen = r-l+1;
                    }
                    l--;r++;
                }else{
                    break;
                }
            }
        }
        return s.substr(begin,maxlen);
    }
};
```

## 415. 字符串相加
https://leetcode-cn.com/problems/add-strings/

类似列竖式，从后面开始算，用一个变量记录进位；最后将计算结果反转。

```C++
class Solution {
public:
    string addStrings(string num1, string num2) {
        string ans;
        int carry = 0;

        int n1=num1.size(), n2=num2.size();
        int i=n1-1, j=n2-1;

        while(i>=0 || j>=0 || carry){
            int sum = carry;
            if(i>=0){
                sum += num1[i--] - '0';
            }
            if(j>=0){
                sum += num2[j--] - '0';
            }
            carry = sum/10;
            sum %= 10;
            ans.push_back(sum + '0');
        }
        reverse(ans.begin(),ans.end());
        return ans;
    }
};
```

## 1190. 反转每对括号间的子串
https://leetcode-cn.com/problems/reverse-substrings-between-each-pair-of-parentheses/

用一个栈记录每个左括号的下标；当遇到右括号的时候，将栈顶的左括号坐标弹出；
然后将左括号的坐标+1 - 当前右括号坐标-1的位置开始反转字符串。

最后输出的时候如果碰到括号就不输出。

```C++
class Solution {
public:
    string reverseParentheses(string s) {
        if(s.empty()){
            return "";
        }
        string ans;
        stack<int> idxstk;
        for(int i=0; i<s.size(); i++){
            char c = s[i];
            if(c == '('){
                idxstk.push(i);
            }
            else if(c == ')'){
                reverse(s.begin()+idxstk.top()+1,s.begin()+i);
                idxstk.pop();
            }
        }

        for(auto& c:s){
            if(c == '(' || c == ')'){
                continue;
            }
            ans += c;
        }
        return ans;

    }
};
```

## 20. 有效的括号
https://leetcode-cn.com/problems/valid-parentheses/

将遇到的左括号全部入栈；遇到右括号出栈。如果出栈的那个括号对应不上，就不是有效的括号。

```C++
class Solution {
public:
    bool isValid(string s) {
        if(s.empty()){
            return true;
        }

        stack<char> stack;

        for(auto c:s){
            if(c == '(' || c == '[' || c == '{'){
                stack.push(c);
            }else{
                if(stack.empty()){
                    return false;
                }
                char t = stack.top();
                if(c == ')'){
                    if(t != '('){
                        return false;
                    }else{
                        stack.pop();
                    }

                }else if(c == ']'){
                    if(t != '['){
                        return false;
                    }else{
                        stack.pop();
                    }
                }else if(c == '}'){
                    if(t != '{'){
                        return false;
                    }else{
                        stack.pop();
                    }
                }
            }
        }
        return stack.empty()?true:false;
    }
};
```

## 14. 最长公共前缀
https://leetcode-cn.com/problems/longest-common-prefix/

```C++
class Solution {
public:
    string longestCommonPrefix(vector<string>& strs) {
        if(strs.empty()){
            return "";
        }

        //从第一个字符串的第一个字符开始选前缀
        for(int i=0; i<strs[0].size(); i++){
            //枚举第二个开始的字符串
            for(int j=1; j<strs.size(); j++){
                //如果有一个字符串的字符前缀不相等，或者字符前缀的下标已经是某一个较短字符串的全部字符
                //就返回0~i-1的字符串
                if(strs[j][i] != strs[0][i] || i == strs[j].size()){
                    return strs[0].substr(0,i);
                }
            }
        }

        return strs[0];
    }
};
```

## 394. 字符串解码
https://leetcode-cn.com/problems/decode-string/

用两个栈分别记录数字和字符串；遇到左括号，将当前数字和当前的字符串分别入栈；

遇到右括号，取出最近的一个数字；然后将当前的字符串重复n次后，添加到最近的字符串上，也就是添加到字符串栈的栈顶，然后让栈顶作为当前的字符串。

```C++
class Solution {
public:
    string decodeString(string s) {
        stack<int> numStack;
        stack<string> strStack;


        int num = 0;
        string cur;
        string ans;
        for(auto c:s){
            if(c <= '9' && c >= '0'){
                num = num*10 + c - '0';
            }
            if(c == '['){
                numStack.push(num);
                strStack.push(cur);
                cur = "";
                num = 0;
            }else if(c == ']'){
                int n = numStack.top();
                numStack.pop();
                string tmp = cur;                
                for(int i=1; i<=n; i++){
                    strStack.top() += tmp;
                }
                cur = strStack.top();
                strStack.pop();
            }else if((c <= 'z' && c >= 'a') || (c <= 'Z' && c >= 'A')){
                cur += c;
            }
        }
        return cur;
    }
};
```

## 6. Z 字形变换
https://leetcode-cn.com/problems/zigzag-conversion/

用一个变量控制当前行这个变量的走向，当当前行是第一行或者是最后一行的时候，就让它开始往下/往上走。

```C++
class Solution {
public:
    string convert(string s, int numRows) {
        if(numRows <= 1){
            return s;
        }

        vector<string> temp(numRows);
        int currRow = 0;
        bool down = false;

        for(auto& c:s){
            temp[currRow] += c;
            if(currRow == 0 || currRow == numRows-1){
                down = !down;
            }
            currRow += down?1:-1;
        }

        string ans;
        for(auto& str:temp){
            ans += str;
        }
        return ans;
    }   
};
```

## 179. 最大数
https://leetcode-cn.com/problems/largest-number/

将数组里面的数字，转换成字符串，然后按照字符串的倒序进行排序。

注意最后如果字符串开头是0，后面就不要再加0了。

```c++
class Solution {
public:
    string largestNumber(vector<int>& nums) {
        
        vector<string> tmp(nums.size());
        for(int i=0; i<nums.size(); i++){
            tmp[i] = to_string(nums[i]);
        }



        sort(tmp.begin(),tmp.end(),[](const string& a, const string &b){
            return a+b>b+a;
        });
        string ans;
        for(auto& str:tmp){
            if(str[0] == '0' && ans[0] == '0'){
                continue;
            }
            ans += str;
        }

        
        return ans;


    //     auto func = [](const string& a, const string& b){
    //         return a+b>b+a;
    //     };

    //     multiset<string,decltype(func)> s(func);

    //     for(auto n:nums){
    //         s.insert(to_string(n));
    //     }
    //     string ans;
    //      for(auto& str:s){
    //         if(str[0] == '0' && ans[0] == '0'){
    //             continue;
    //         }
    //         ans += str;
    //     }

    //     return ans;
    // }
};
```

## 43. 字符串相乘
https://leetcode-cn.com/problems/multiply-strings/comments/

num1的第i位(高位从0开始)和num2的第j位相乘的结果在乘积中的位置是[i+j, i+j+1].

```c++
class Solution {
public:
    string multiply(string num1, string num2) {
        int n1 = num1.size(), n2 = num2.size();

        vector<int> result(n1+n2);

        for(int i=n1-1; i>=0; i--){
            for(int j=n2-1; j>=0; j--){
                int bitmul = (num1[i] - '0') * (num2[j] - '0');
                bitmul += result[i+j+1];
                result[i+j] += bitmul/10;
                result[i+j+1] = bitmul%10;
            }
        }

        string ans;
        int i=0;
        while(i<result.size()-1 && result[i] == 0){
            i++;
        }

        while(i<result.size()){
            ans.push_back(result[i++] + '0');
        }
        return ans;
    }
};
```