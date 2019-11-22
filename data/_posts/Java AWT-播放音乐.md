---
title: Java AWT-播放音乐
date: 2018-05-20 16:31:55
tags: [Java,GUI,AWT]
categories: Java AWT
---

使用Maven管理jar包，首先在pom.xml中导入所需jar：

```
<dependency>
  <groupId>com.googlecode.soundlibs</groupId>
  <artifactId>jlayer</artifactId>
  <version>1.0.1.4</version>
</dependency>
```

<!-- more -->

然后准备一首mp3格式的音乐作为例子：

```
DiGiTAL WiNG - 恋ノ蟲.mp3
```

下面给出使用示例：

```
import java.io.BufferedInputStream;
import java.io.File;
import java.io.FileInputStream;
import java.io.IOException;

import javazoom.jl.decoder.JavaLayerException;
import javazoom.jl.player.Player;

public class Test {

    public static void main(String[] args) throws JavaLayerException, IOException {
        String path = new StringBuilder("D:").append(File.separator)
                                             .append("DiGiTAL WiNG - 恋ノ蟲.mp3")
                                             .toString();
        File file = new File(path);
        try (FileInputStream fis = new FileInputStream(file);) {
            try (BufferedInputStream bis = new BufferedInputStream(fis);) {
                Player player = new Player(bis);
                try {
                    player.play();
                } finally {
                    player.close();
                }
            }
        }
    }
}
```

歌曲播放完成后程序退出。