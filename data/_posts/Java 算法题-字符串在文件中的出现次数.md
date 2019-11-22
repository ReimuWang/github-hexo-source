---
title: Java 算法题-字符串在文件中的出现次数
date: 2017-10-12 15:51:36
tags: [算法,Java]
categories: Java 算法题
---

<!-- more -->

```
import java.io.BufferedReader;
import java.io.FileReader;

public class Test {

    public static int countWordInFile(String filename, String word) {
        int count = 0;
        try (FileReader fr = new FileReader(filename)) {
            try (BufferedReader br = new BufferedReader(fr)) {
                String line = null;
                while ((line = br.readLine()) != null) {
                    int index = -1;
                    while (line.length() >= word.length() && (index = line.indexOf(word)) >= 0) {
                        count++;
                        line = line.substring(index + word.length());
                    }
                }
            }
        } catch (Exception ex) {
            ex.printStackTrace();
        }
        return count;
    }

    public static void main(String[] args) {
        System.out.println(countWordInFile("e:\\blog\\test\\temp.txt", "public"));
    }
}
```