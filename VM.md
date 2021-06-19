## stoi
```
https://leetcode.com/problems/string-to-integer-atoi/
Implement the myAtoi(string s) function, which converts a string to a 32-bit signed integer (similar to C/C++'s atoi function).

The algorithm for myAtoi(string s) is as follows:

Read in and ignore any leading whitespace.
Check if the next character (if not already at the end of the string) is '-' or '+'. Read this character in if it is either. This determines if the final result is negative or positive respectively. Assume the result is positive if neither is present.
Read in next the characters until the next non-digit charcter or the end of the input is reached. The rest of the string is ignored.
Convert these digits into an integer (i.e. "123" -> 123, "0032" -> 32). If no digits were read, then the integer is 0. Change the sign as necessary (from step 2).
If the integer is out of the 32-bit signed integer range [-231, 231 - 1], then clamp the integer so that it remains in the range. Specifically, integers less than -231 should be clamped to -231, and integers greater than 231 - 1 should be clamped to 231 - 1.
Return the integer as the final result.
Note:

Only the space character ' ' is considered a whitespace character.
Do not ignore any characters other than the leading whitespace or the rest of the string after the digits.
 

Example 1:

Input: s = "42"
Output: 42
Explanation: The underlined characters are what is read in, the caret is the current reader position.
Step 1: "42" (no characters read because there is no leading whitespace)
         ^
Step 2: "42" (no characters read because there is neither a '-' nor '+')
         ^
Step 3: "42" ("42" is read in)
           ^
The parsed integer is 42.
Since 42 is in the range [-231, 231 - 1], the final result is 42.

```
### 解题步骤
+ 1）忽略最前面的空格符号:  "  
```
-42"->(-42)
```
+ 2)检查第一个字符是否为'-': 
++ 2.1） 如果是，就设置负数标志为true： 
```
“-42” -> (42)
```
+ 2.2)  如果不是，就判断第一个字符是否为'+',如果是就将下标加1； 
``` 
"+1"->(1); 
```
+ 3）循环读取剩下的字符，直到下一个非数字字符或者到字符串的结尾。剩下的字符忽略。
```
"words and 987" -> (0)
"-+12" -> (0)
"12jiui88"->(12)
```
+  4) 如果最终的结果大于int32的最大值或者是小于int32的最小值，返回INT_MAX或者INT_MIN
```
"-91283472332" -> (-2147483648)
```
```cpp
class Solution {
public:
    int myAtoi(string s) {
        long long res = 0;
        int idx = 0;
        for(; idx < s.size(); idx++){
            if(s[idx]!= ' '){
                break;
            }
        }
        bool negative = false;
        if(s[idx] == '-'){
            negative = true;
            idx++;
        }else if(s[idx] == '+'){
            idx++;
        }
        for(; idx < s.size(); idx++){
            if(s[idx] >= '0' && s[idx] <= '9'){
                res = res*10 + (s[idx] - '0');
                if(negative == false && res >= static_cast<long long>(INT_MAX)){
                    return INT_MAX;
                }
                if(negative == true && -res <= static_cast<long long>(INT_MIN)){
                    return INT_MIN;
                }
            }else{
                break;
            }      
        }
        
        return negative == true? int(-res) : int(res);
    }
};
```
## reverse Linked List
```
https://leetcode.com/problems/reverse-linked-list/
Given the head of a singly linked list, reverse the list, and return the reversed list.

Example 1:
Input: head = [1,2,3,4,5]
Output: [5,4,3,2,1]

Example 2:
Input: head = [1,2]
Output: [2,1]

Example 3:
Input: head = []
Output: []

```
### 解题思路
+ 首先定义要返回的新链表为空： tail=nullptr;
+ 从头到尾遍历要逆转的链表，将每一个节点查到新链表的头部。head->next = tail, tail = head;
```cpp
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode() : val(0), next(nullptr) {}
 *     ListNode(int x) : val(x), next(nullptr) {}
 *     ListNode(int x, ListNode *next) : val(x), next(next) {}
 * };
 */
class Solution {
public:
    ListNode* reverseList(ListNode* head) {
        
        ListNode* tail = nullptr;
        
        while(head != nullptr){
            //一定要用一个临时值先保存旧链表的下一个要处理的节点
            auto next = head->next; 
            head->next = tail;
            tail = head;
            head = next;
        }
        
        return tail;
    }
};
```

## remove-nth-node-from-end-of-list
```
https://leetcode.com/problems/remove-nth-node-from-end-of-list/

Given the head of a linked list, remove the nth node from the end of the list and return its head.

Example 1:
Input: head = [1,2,3,4,5], n = 2
Output: [1,2,3,5]

Example 2:
Input: head = [1], n = 1
Output: []

Example 3:
Input: head = [1,2], n = 1
Output: [1]
```
### 解题思路
+ Topic: 快慢指针
+ 为了处理删除链表头指针的情况，在链表前面加一个假的节点dummyHead；
+ 先让快指针pFast走n步，中间如果快指针为空（说明n比链表长度大）直接返回原链表；
+ 让慢指针pSlow指向假的头加点，快慢指针同时后移直到pFast==nullptr;此时pSlow->next就是要删除的节点。
+ 用临时变量pDel保存要删除的节点pDel=pSlow->next; 更改pSlow的next的值，pSlow->next=pSlow->next->next; 
```cpp
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode() : val(0), next(nullptr) {}
 *     ListNode(int x) : val(x), next(nullptr) {}
 *     ListNode(int x, ListNode *next) : val(x), next(next) {}
 * };
 */
class Solution {
public:
    ListNode* removeNthFromEnd(ListNode* head, int n) {
        ListNode dummyHead(0, head);
        
        ListNode* pFast = head;
        for(int i = 0; i < n; i++){
            if(pFast == nullptr){
                return head;
            }
            pFast = pFast->next;
        }
        
        ListNode* pSlow = &dummyHead;
        while(pFast != nullptr){
            pFast = pFast->next;
            pSlow = pSlow->next;
        }
        
        ListNode* pDel = pSlow->next;
        pSlow->next = pSlow->next->next;
        delete pDel;
        
        return dummyHead.next;
    }
};
```
