---
title: Java 算法题-翻转单链表
date: 2017-10-11 19:06:36
tags: [算法,Java]
categories: Java 算法题
---

<!-- more -->

```
public class Test {

    /**
     * 递归
     */
    public static Node reverse(Node node) {
        if (null == node || null == node.next) return node;
        Node head = reverse(node.next);
        node.next.next = node;
        node.next = null;
        return head;
    }

    /**
     * 遍历
     */
    public static Node reverse2(Node node) {
        if (null == node || null == node.next) return node;
           Node p = node;    // 前一个结点
           Node c = node.next;    // 当前结点
           Node n = c;    // 后一个结点
           while (null != c) {
               n = c.next;
               c.next = p;
               p = c;
               c = n;
           }
           node.next = null;
           return p;
    }

    public static void main(String[] args) {
        Node head = Node.createList(new int[] {1, 2, 3, 4, 5});
        head.show();
        System.out.println();
        reverse2(head).show();
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