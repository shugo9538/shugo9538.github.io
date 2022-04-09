---

title: Valid Parentheses 
date: 2022-04-09
categories: [leet code]  
tags: ['java', 'leet code']  
use_math: true

---

***
# 문제
Given a string s containing just the characters '(', ')', '{', '}', '[' and ']', determine if the input string is valid.

An input string is valid if:

Open brackets must be closed by the same type of brackets.
Open brackets must be closed in the correct order.


Example 1:
```scss
Input: s = "()"
Output: true
```
Example 2:
```scss
Input: s = "()[]{}"
Output: true
```
Example 3:
```scss
Input: s = "(]"
Output: false
```


Constraints:

$1 <= s.length <= 104$
$s consists of parentheses only '()[]{}'.$

# 코드

```java
class Solution {
    public boolean isValid(String s) {
        //
        /**
         * 1. 순서, 쌍이 정확히 있어야 함
         *
         */
        boolean flag = true;

        List<String> opener = new ArrayList<>();
        opener.add("(");
        opener.add("{");
        opener.add("[");

        Stack<String> stack = new Stack<>();

        for (int i = 0 ; i < s.length() ; i++) {
            //
            if (opener.contains(String.valueOf(s.charAt(i)))) {
                stack.push(String.valueOf(s.charAt(i)));
            } else {
                //
                if (!stack.isEmpty()) {
                    switch (stack.peek()) {
                        case "(":
                            if (s.charAt(i) == ')') stack.pop();
                            else flag = false;
                            break;
                        case "{":
                            if (s.charAt(i) == '}') stack.pop();
                            else flag = false;
                            break;
                        case "[":
                            if (s.charAt(i) == ']') stack.pop();
                            else flag = false;
                            break;
                        default:
                            flag = false;
                            break;
                    }
                } else {
                    flag = false;
                }
            }

            if (!flag) break;
        }

        if (stack.isEmpty()) return flag;
        else return false;
    }
}
```

# 문제 해설
코드는 좀 더 정리할 수 있을 것 같고 개선의 여지도 많이 보인다.
문제를 푸는데 30분이나 걸렸으니 좀 더 빠르게 풀 수 있도록 연습 많이하자...
