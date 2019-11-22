---
title: Java 算法题-查找链表中的倒数第k个结点
date: 2017-10-12 10:54:36
tags: [算法,Java]
categories: Java 算法题
---

<!-- more -->

```
public class Test {

    public static Node reGetKthNode(Node head, int k) {
        if (k <= 0) throw new IllegalArgumentException("k <= 0");
        if (null == head) return null;
        Node p = head;
        Node q = head;
        // p移动k-1次。即保证p指在正数第k个结点上
        for (int i = 1; i < k; i++) {
            if (null == p) break;
            p = p.next;
        }
        // 链表没有正数第k个结点，自然不会有倒数第k个结点
        if (null == p) return null;
        // 记忆时，可依k=1的特例记忆
        while (null != p.next) {
            p = p.next;
            q = q.next;
        }
        return q;
    }

    public static void main(String[] args) {
        Node head = Node.createList(new int[] {1, 2, 3, 4, 5});
        System.out.println(reGetKthNode(head, 5).data);
    }
}

class Node {

    public int data;

    public Node next;

    public static Node createList(int[] a) {
        Node head = new Node();
        Node temp = head;
        for (int i = 0; i < a.length; i++) {
            temp.data = a[i];
            if (i == a.length - 1) {
                temp.next = null;
            } else {
                temp.next = new Node();
                temp = temp.next;
            }
        }
        return head;
    }

    public void show() {
        Node temp = this;
        while (null != temp) {
            System.out.print(temp.data + "\t");
            temp = temp.next;
        }
    }
}
```