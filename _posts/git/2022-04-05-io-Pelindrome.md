---

title: Pelindrome 
date: 2022-04-04
categories: [leet code]  
tags: ['java', 'leet code']  
use_math: true

---

***
# 문제

Given an integer x, return true if x is palindrome integer.

An integer is a palindrome when it reads the same backward as forward.

For example, 121 is a palindrome while 123 is not.


Example 1:

Input: x = 121
Output: true
Explanation: 121 reads as 121 from left to right and from right to left.
Example 2:

Input: x = -121
Output: false
Explanation: From left to right, it reads -121. From right to left, it becomes 121-. Therefore it is not a palindrome.
Example 3:

Input: x = 10
Output: false
Explanation: Reads 01 from right to left. Therefore it is not a palindrome.


Constraints:

$-2^31 <= x <= 2^31 - 1$

# 코드

```java
class Solution {
   //
   public boolean isPalindrome(int x) {
      boolean result  = false;
      // 1. int -> String
      String strX = String.valueOf(x);

      // 2. String = char[]
      int len = strX.length() - 1;

      for (int i = 0 ; i <= len ; i++) {
         if (strX.charAt(i) == strX.charAt(len - i)) {
            result = false;
         } else {
            break;
         }

         /**
          *  121: length = 3
          *  len = 2
          *  i++ 0 ~ 2
          */
         if (i >= len - i) {
            //
            result = true;
            break;
         }
      }

      return result;
   }
}
```

# 문제 해설

ps. 스터디로 진행하면서 하루 한 문제 풀기시작
