---

title: RomanToInt 
date: 2022-04-07
categories: [leet code]  
tags: ['java', 'leet code']  
use_math: true

---

***
# 문제
Roman numerals are represented by seven different symbols: I, V, X, L, C, D and M.

```scss
Symbol       Value
I             1
V             5
X             10
L             50
C             100
D             500
M             1000
```
For example, 2 is written as II in Roman numeral, just two one's added together. 12 is written as XII, which is simply X + II. The number 27 is written as XXVII, which is XX + V + II.

Roman numerals are usually written largest to smallest from left to right. However, the numeral for four is not IIII. Instead, the number four is written as IV. Because the one is before the five we subtract it making four. The same principle applies to the number nine, which is written as IX. There are six instances where subtraction is used:

```scss
I can be placed before V (5) and X (10) to make 4 and 9.
X can be placed before L (50) and C (100) to make 40 and 90.
C can be placed before D (500) and M (1000) to make 400 and 900.
```
Given a roman numeral, convert it to an integer.



Example 1:
```scss
Input: s = "III"
Output: 3
Explanation: III = 3.
```
Example 2:
```scss
Input: s = "LVIII"
Output: 58
Explanation: L = 50, V= 5, III = 3.
```
Example 3:
```scss
Input: s = "MCMXCIV"
Output: 1994
Explanation: M = 1000, CM = 900, XC = 90 and IV = 4.
```


Constraints:

$1 <= s.length <= 15$
$s contains only the characters ('I', 'V', 'X', 'L', 'C', 'D', 'M').$
$It is guaranteed that s is a valid roman numeral in the range [1, 3999].$

# 코드

```java
class Solution {
    public int romanToInt(String s) {
        //
        String tmp = s;

        int result = 0;
        Map<String, Integer> mapper1 = new HashMap<>();
        mapper1.put("IV", 4);
        mapper1.put("IX", 9);
        mapper1.put("XL", 40);
        mapper1.put("XC", 90);
        mapper1.put("CD", 400);
        mapper1.put("CM", 900);

        for (Object str : mapper1.keySet().toArray()) {
            if (tmp.contains((String) str)) {
                result += mapper1.get((String) str);
                tmp = tmp.replace((String) str, "");
            }
        }

        Map<String, Integer> mapper2 = new HashMap<>();
        mapper2.put("I", 1);
        mapper2.put("V", 5);
        mapper2.put("X", 10);
        mapper2.put("L", 50);
        mapper2.put("C", 100);
        mapper2.put("D", 500);
        mapper2.put("M", 1000);

        for (int i = 0 ; i < tmp.length() ; i++) {
            //
            result += mapper2.get(String.valueOf(tmp.charAt(i)));
        }

        return result;
    }
}
```

# 문제 해설
