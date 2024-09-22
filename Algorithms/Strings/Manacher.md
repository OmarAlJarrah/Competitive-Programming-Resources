# Manacher's Algorithm

Manacher's algorithm is used to find the longest palindrome substring. The algorithm works by utlizing the symetric property of palindromes into use. The algorithm works in a dyncamic programming style where previously computed values are used to avoid re-computation of new values.

### Palindromes are Symetric
Palindromes are symetric at their center. In other words, the left side of a palindrome's center is equal to the right side of a palindrome's center. Consider the palindrome string `"aabaa"`. The center of the palindrome is `"b"`, and it's left and right sides respectively are `"aa"` and `"aa"`.

## Problem Statement
Given a string `text`, find the longest palindrome substring in text. Notice that all of `text` can be palinrome.

## Naive Approach: Center Expansion
One way to find the longest palindrome substring is by attempting to find all palindrome substrings and taking the longest. One of the most straightforward ways to find palindrome substrings is the center **expansion method**.

The **center expansion** method assumes a character as the center of a palindrome, then starts expanding left and right while validating equality. One might also need to care for cases where the center of a palindrome can be two characters (or empty character) such as the palindrome `"abba"` where there is no single leter as a center.

#### Implementation
```python
def expand(left: int, right: int) -> str:
    while left >= 0 and right < len(text) and text[left] == text[right]: # O(n) time
        left -= 1
        right += 1
    return text[left+1:right]

def find_longest_palindrome_substring(text: str) -> str:
    result = ""
    for index in range(len(text)): # O(n) time
        substring1 = expand(index, index) # O(n^2) time
        if len(substring1) > len(result):
            result = substring1
        substring2 = expand(index, index+1)
        if len(substring2) > len(result):
            result = substring2
    return result
```

#### Complexity
* **Time:O($n^2$)**
* **Time:O(1)**

## Optimization: Manacher's Algorithm
Manacher's Algorithm extends the idea of center extansion by utilizing the fact that strings are symetric. We know that the right side of a palindrome is equal to the left side of it. Based on that fact, the expansion method at the left side of a palindrome center should also infer the result of expansion in the mirroring side of right of the palindrome center.

For example, given the string `"abcccxccc"`, if we traverse characters from left to right, and invoke the expansion method on each character, we will find that the slice of indices `2:4` inclusive is a palindrome (`palindrome-1`). Then and as we go, we will find that the slice of indices `2:8` inclusive is a palindrome (`palindrome-2`).

Since we know that `palindrome-1` is a substring on the left to the center of `palindrome-2`, we know for a fact that there exists a palindrome substring on the right side of the center of `palindrome-2`.

Manacher's algorithm works by utilizing an array `dp`, where for each `index` of the string, `dp[index]` is equal to the length of palindrome substring with its center as the character at `index`.

#### Note
There are two implementations for the Manacher's algorithm for even and odd length palindromes, but for simplicity, we will sacrific some memory in order to make all palindromes of odd length. We will a character that could never exist in the string at the star, end, and between all characters of the strings.

### Steps
#### 0) Initialize `PalindromeMetadata` class. \[Optional\]
We will initialize a class `PalindromeMetadata` that would help us make our code more readable.
```python
from dataclasses import dataclass

@dataclass
class PalindromeMetadata:
    center: int
    radius: int

    def rightmost_index(self):
        return self.center + self.radius
```

#### 1) Preprocessing & Initializations
+ We will add a character that can never exist in the string at the start, end and between the characters of the string. For example, if our string is `aabb`, and our character of choice is `#`, our new string will be `#a#a#b#b#`. Size of the new string will always be `2*n + 1`, where `n` is the length of the original string.
```python
processed: str = f"#{'#'.join(string)}#"
```

+ Initialize `dp` array
```python
dp: list[int] = [0] * len(processed)
```
+ Initialize starting `PalindromeMetadata` instance
```python
palindrome: PalindromeMetadata = PalindromeMetadata(0, 0)
```
+ Index-based traversal of the string
```python
for index in range(len(processed)):
    ...
```
#### 2) Mirror Calculation
The `mirror` value of a substring on the left side of a palindrome's center is the symetric value on the right side of the palindrome's center. The mirror value as an index, can be calculated with the equation `2*center + 1`.

This is true because for a mirror value on the rigth side of a palindrome's center, we know that it is `X` distance away from the center. `X` is the same value of the distance of the left side substring from the center. `X` can be calulated by the simple equation `index - center`. So, we now can make the assumption that:
+ `left_index - center == right_index + center` is `true`.
+ `left_index == right_index + center + center` (added `+ center` to both sides).
+ `left_index == right_index + 2*center` is true

```python
mirror: int = 2 * palindrome.center - index
```

