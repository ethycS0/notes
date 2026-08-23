Previous: [[Remove Linked List Elements (E)]] | Next: [[Palindrome Linked List (E)]]

You are given the head of a singly linked list and two integers left and right where left <= right, reverse the nodes of the list from position left to position right (1-indexed), and return the reversed list.

## Solution

So this requires reversing a very specific location onwards. Remember to focus on the edge cases. Otherwise this quesiton is quite simple. Use a dummy node. Iterate till you get to left. Here initialize curr to be prev->next and then start the head-insertion technique.

Example:


0 -> 1 -> 2 -> 3 -> 4 -> 5  left = 2 right = 4


Dummy Node:

-1 -> 0 -> 1 -> 2 -> 3 -> 4 -> 5 ->
 P

Get to left:

-1 -> 0 -> 1 -> 2 -> 3 -> 4 -> 5 ->
      P    C

Now flip time:

Iter 1:

-1 -> 0 -> 1 -> 2 -> 3 -> 4 -> 5 ->
      P    C    T
	  
-1 -> 0 -> 1 -> 3 -> 4 -> 5 ->
      P    C    
	  
2 -> 1 -> 3 -> 4 -> 5
T

-1 -> 0 -> 2 -> 1 -> 3 -> 4 -> 5 ->
      P         C 

Iter 2:

-1 -> 0 -> 2 -> 1 -> 3 -> 4 -> 5 ->
      P         C    T
	  
-1 -> 0 -> 2 -> 1 -> 4 -> 5 ->
      P         C
                   
3 -> 2 -> 1 -> 4 -> 5 ->
T

-1 -> 0 -> 3 -> 2 -> 1 -> 4 -> 5 ->
P              C

Loop terminates (right - left = 2 iterations complete).

Final State:
-1 -> 0 -> 3 -> 2 -> 1 -> 4 -> 5 ->

```C++
class Solution {
public:
    ListNode* reverseBetween(ListNode* head, int left, int right) {
        ListNode dummy(0, head);
        ListNode* prev = &dummy;

        for (int i = 0; i < left - 1; ++i) {
            prev = prev->next;
        }

        ListNode* curr = prev->next;
        for (int i = 0; i < right - left; ++i) {
            ListNode* temp = curr->next;
            curr->next = temp->next;
            temp->next = prev->next;
            prev->next = temp;
        }

        return dummy.next;
    }
}
```