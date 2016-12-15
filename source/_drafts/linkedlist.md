---
title: linkedlist
tags: [algorithm]
---

## 链表是否有环，及其进入环的位置

## 删除链表中重复的元素

## 合并n个有序链表(分治)
```c++
class Solution {
public:
     ListNode *mergeKLists(vector<ListNode *> &lists) {
        if(lists.empty()) {
            return NULL;
        }

        int n = lists.size();
        while(n > 1) {
            int k = (n + 1) / 2;
            for(int i = 0; i < n / 2; i++) {
                //合并i和i + k的链表，并放到i位置
                lists[i] = merge2List(lists[i], lists[i + k]);
            }
            //下个循环只需要处理前k个链表了
            n = k;
        }
        return lists[0];
    }

    ListNode* merge2List(ListNode* n1, ListNode* n2) {
        ListNode dummy(0);
        ListNode* p = &dummy;
        while(n1 && n2) {
            if(n1->val < n2->val) {
                p->next = n1;
                n1 = n1->next;
            } else {
                p->next = n2;
                n2 = n2->next;
            }
            p = p->next;
        }

        if(n1) {
            p->next = n1;
        } else if(n2) {
            p->next = n2;
        }

        return dummy.next;
    }
};
```

## 排序(On(logn))
```c++
class Solution {
public:
   ListNode *sortList(ListNode *head) {
        if(head == NULL || head->next == NULL) {
            return head;
        }

        ListNode* fast = head;
        ListNode* slow = head;

        //快慢指针得到中间点
        while(fast->next && fast->next->next) {
            fast = fast->next->next;
            slow = slow->next;
        }

        //将链表拆成两半
        fast = slow->next;
        slow->next = NULL;

        //左右两半分别排序
        ListNode* p1 = sortList(head);
        ListNode* p2 = sortList(fast);

        //合并
        return merge(p1, p2);
    }

    ListNode *merge(ListNode* l1, ListNode* l2) {
        if(!l1) {
            return l2;
        } else if (!l2) {
            return l1;
        } else if (!l1 && !l2) {
            return NULL;
        }

        ListNode dummy(0);
        ListNode* p = &dummy;

        while(l1 && l2) {
            if(l1->val < l2->val) {
                p->next = l1;
                l1 = l1->next;
            } else {
                p->next = l2;
                l2 = l2->next;
            }

            p = p->next;
        }

        if(l1) {
            p->next = l1;
        } else if(l2){
            p->next = l2;
        }

        return dummy.next;
    }
};
```


