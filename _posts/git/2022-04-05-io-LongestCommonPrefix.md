---

title: Pelindrome 
date: 2022-04-04
categories: [leet code]  
tags: ['java', 'leet code']  
use_math: true

---

***
# 문제
Write a function to find the longest common prefix string amongst an array of strings.

If there is no common prefix, return an empty string "".



Example 1:
```scss
Input: strs = ["flower","flow","flight"]
Output: "fl"
```
Example 2:
```scss
Input: strs = ["dog","racecar","car"]
Output: ""
Explanation: There is no common prefix among the input strings.
```

Constraints:

```scss
1 <= strs.length <= 200
0 <= strs[i].length <= 200
strs[i] consists of only lower-case English letters.
```

# 코드

```java
class Solution {
    public String longestCommonPrefix(String[] strs) {
        //
        StringBuilder sb = new StringBuilder();

        for (int i = 0 ; i < strs.length ; i++) {
            if (sb.toString().equals("") && i == 0) {
                sb.append(strs[i]);
            }

            String tmp = sb.toString();

            StringBuilder sb2 = new StringBuilder();

            int len = Math.min(tmp.length(), strs[i].length());

            for (int j = 0; j < len; j++) {
                if (tmp.charAt(j) == strs[i].charAt(j)) {
                    sb2.append(strs[i].charAt(j));
                } else {
                    break;
                }
            }

            sb.setLength(0);
            sb.append(sb2);
        }

        return sb.toString();
    }
}
```

# 문제 해설

ps. 길이, 메소드, 스태틱 필드 주의
