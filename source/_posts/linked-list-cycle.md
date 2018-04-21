title: 判断链表是否有环
date: 2015-4-15 22:05:43
tags: [技术]
---
``` text
Given a linked list, determine if it has a cycle in it.

Follow up:
Can you solve it without using extra space?
```

可以使用快慢指针判断，快指针每次走两步，慢指针每次走一步
如果有环的话快慢指针一定会相遇，注意判断指针是否为空
``` cpp
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
class Solution {
public:
    bool hasCycle(ListNode *head) {
        ListNode *fast, *slow;

        fast = slow = head;

        while (fast && slow) {
            fast = fast->next;

            if (fast == NULL) {
                return false;
            }

            fast = fast->next;
            slow = slow->next;

            if (fast == slow) {
                return true;
            }
        }

        return false;
    }
};
```
