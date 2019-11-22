---
title: Java 设计模式-23.Interpreter模式
date: 2018-09-10 14:45:49
tags: [Java,设计模式]
categories: Java 设计模式
---

在《图解设计模式》一书中，Interpreter模式被归入了第10部分[用类来表现]()。在GoF原书中，Interpreter模式则被归入了[行为型设计模式]()。

<!-- more -->

# 综述

现今主流的编程语言依然都属于第三代编程语言，这些语言的文法接近人类的自然语言，对程序员极其友好。然而实际执行程序的机器依然只能识别第一代编程语言(也就是一串1和0组成的流啦)，这就导致了，无论使用何种第三代编程语言，是Java也好，还是C++也罢，虽然具体的实现细节依语言不同而千差万别，但终归都需要进行一种操作：将程序员编写的高级语言代码翻译为机器能够识别的01码。

下面我们就以Java为例，来相对详细的说说这个过程。

和大多直接由高级语言翻译(编译执行与解释执行只是手段，其本质目的都还是翻译)为机器语言不同的是，Java将它内部的翻译过程又分为了两个阶段(当然，对外部使用者，也就是实际执行机器码的机器而言，这个过程是黑盒的)：首先会从程序员编写的，人类能够识别的Java语言翻译为.class文件(编译期)。而后再将.class文件翻译为机器能识别的机器码。

简单来说，程序员编写的Java语言遵循着一套语法(不妨标记为语法1)，机器识别机器码时遵循着另一套语法(语法3)。在此之外，Java又在其内部搞出了一个.class文件(语法2)。这样翻译链的顺序就变为了

```
语法1 --> 语法2 --> 语法3
```

之所以要这么搞，是为了实现Java语言的语言无关性。详见[Java 基础-技术体系](/2017/10/16/Java 基础-技术体系/)。

且不说翻译了几次，单说翻译行为本身，它的作用大致可归为以下两点：

1. 虽然翻译本身增加了额外的开销，却使得程序员得以用更加类似自然语言的语法编写程序，极大的降低了学习成本，同时提高了代码编写的便利性与可维护性。
2. 完成了各级语言间的解耦。以语法1 --> 语法2的过程为例。Java程序员只需要知道Java语言的语法，而它的翻译者(我们称其为编译器)在此基础上还需要知道.class文件的语法。这意味着，无论是语法1亦或是语法2发生了何种变化，只要另一种语法尚能实现相同的功能，那么对彼此而言，这种变化就是透明的：它会被翻译者消化掉。从这种意义上来讲，翻译者颇有些[2.Adapter模式]()中的适配器的意思。

将这种思想进一步扩展，得到的就是Interpreter模式。

主流的高级编程语言都是通用的，虽然各有擅长的点，但基本都能胜任绝大多数场景。而这种通用的另一面就是不够特化：高级语言语法的设计者当然不可能因为某个需求就修改语法。不过，高级语言的设计者虽然不能这么做，但需求的设计者却可以这样做：也就是说，我们为某个需求，或是某一类需求，定制一种"迷你语言"。如果我们将其称之为"语法0"的话，那么上文的翻译链就变为了：

```
语法0 --> 语法1 --> 语法2 --> 语法3
```

如果想要这么做的话，我们当然也可以在语法0前面加上个"语法-1"，使得翻译链无限的向上堆叠上去，只不过这会让程序变得复杂，而这种复杂通常是没什么意义的，所以通常都不会这么做。

具体到这个应用场景，我们再来说下引入"语法0"的好处：

1. 较之高级语言，迷你语言的语法简单，且为需求高度定制。使得代码(此时程序员写的当然就是迷你语言的代码啦)的书写更为容易，目的性也更强。
2. 在这个场景下，高级语言的语法一般不会发生变化，迷你语言的语法通常也不会发生变化。发生变化的基本只会是使用迷你语言编写的代码。

如果要使用Interpreter模式编写程序的话，难点通常并不是在于编写迷你语言本身，而是在于编写"语法0 --> 语法1"的翻译器。因为迷你语言完全是程序员根据需求生造出来的，只有他自己才知道语法，因此也只有他本身才能编写翻译器。这也是Interpreter模式被称为"解释器模式"的原因所在。通常翻译器我们都会用"语法1"，也就是作为基底的高级语言来开发。

# 控制人物移动的迷你语言

下面我们就针对"控制人物移动"这个需求来创建一门全新的迷你语言吧！

首先介绍下本文用于作为行走角色的妹子，来自东方Project的[伊吹萃香](https://baike.baidu.com/item/%E4%BC%8A%E5%90%B9%E8%90%83%E9%A6%99/8771754?fr=aladdin)：

![0.jpg](/images/blog_pic/Java 设计模式/23Interpreter模式/0.jpg)

萃香好可爱啊~融化了。

这是用到的行走图素材，也是萌萌哒：

![1.png](/images/blog_pic/Java 设计模式/23Interpreter模式/1.png)

按需求，我们可以向人物下达如下3种指令：

- 前进1个长度单位(go)
- 右转(right)
- 左转(left)

其中左转与右转是严格意义上的原地转向。在此之上，为了构成一个相对复杂的语言，我们引入了新的指令：

- 重复(repeat)

这可以让人物重复一定次数的指令集合。至此，我们已得到构成本文迷你语言的全部指令。

要想被称之为一门语言，光有指令(相当于自然语言中的单词)是不够的，我们还需要规定一个语法将它们组合起来。对于本文的迷你语言，我们先来看一段最简单的代码：

```
program
end
```

我们规定，这门迷你语言必须以program开头，后面跟随着指令集合(command list)。command list必须以end结尾。如果command list为空，那么就会出现上文中program后面直接跟着一个end的情况啦。显然，上述代码不会产生任何实际的效果。

程序可以采用换行，空格，tab等任何主流的分隔符，这意味上述代码我们也可以写成这样：

```
program end
```
不过为了便于阅读，对于书写任意语言而言，适当的缩进都是很必要的。

然后我们再来看一段稍微复杂点的代码：

```
program
  go go right
  go go right
  go go right
  go go right
end
```

程序会按指令的书写顺序执行它们。执行这段指令，可以输出：

![2.jpg](/images/blog_pic/Java 设计模式/23Interpreter模式/2.jpg)

该代码会让人物走一个正方形，而后回到初始的位置和朝向。为了在静态的图片中表现出代码的执行过程，我们绘制了人物移动的轨迹。

如果仔细看一下上述代码的话，会发现我们将：

```
go go right
```

这个代码片段重复写了4遍。显然这是很low的行为。因此就需要repeat指令登场啦：

```
program
  repeat 4 go go right end
end
```

repeat指令的语法规则为：首先是作为关键字的repeat，然后是表示循环次数的数字。最后是实际被循环的command list。执行该代码后输出与前图相同。

我们可以再来走一个更复杂的轨迹：

```
program
  repeat 1000
    repeat 4
      repeat 3 go right go left end
      right
    end
  end
end
```

这段代码会重复一个相对复杂的轨迹1000次，贴几张移动过程中的截图吧：

![3.jpg](/images/blog_pic/Java 设计模式/23Interpreter模式/3.jpg)

![4.jpg](/images/blog_pic/Java 设计模式/23Interpreter模式/4.jpg)

![5.jpg](/images/blog_pic/Java 设计模式/23Interpreter模式/5.jpg)

![6.jpg](/images/blog_pic/Java 设计模式/23Interpreter模式/6.jpg)

![7.jpg](/images/blog_pic/Java 设计模式/23Interpreter模式/7.jpg)

# 示例程序

迷你语言的语法结构至此就算是结束了，该语言非常简单，简单到只有顺序和循环结构，连分支结构都没有。不过用来作说明已经是足够了。此外，上文我们是以自然语言在进行描述，也就是说给产品经理听的。而技术人员如果要将该语法落实到代码中，还需要用更为准确的数学语言来描述才可以。

本文采取的描述方法为EBNF(Extended Backus–Naur Form，即扩展的巴科斯-瑙尔范式)。顾名思义，它是对BNF(Backus-Naur Form，巴科斯范式)的扩展。我们先给出全部的语法描述：

```
<program> ::= program <command list>
<command list> ::= <command>* end
<command> ::= <repeat command> | <primitive command>
<repeat command> ::= repeat <number> <command list>
<primitive command> ::= go | left | right
```

较之自然语言，是不是简洁了许多呢(事实上，不仅仅是看起来简洁了，也更为精确了)？下面就来逐句分析一下吧！

```
<program> ::= program <command list>
```

首先是第一句，这个::=我们简单的理解为赋值就可以啦！该句用于描述&lt;program&gt;这个标签，或者更确切的说，是构成语法树的&lt;program&gt;结点。按照描述，&lt;program&gt;节点就是program关键字后跟上一个&lt;command list&gt;。

至此&lt;program&gt;算是描述完了，但是我们却无法真正能说完全了解它，program作为一个普通的字符串没什么可说的，不过这个&lt;command list&gt;是什么呢？这就好比表达式y=x+3，如果我们不知道x是什么，自然也不能知道y是什么。

不过不要急，第二句就是在描述&lt;command list&gt;啦：

```
<command list> ::= <command>* end
```

哦，原来&lt;command list&gt;是0到多个(描述范式中*所代表的含义，熟悉正则的朋友们应该会感到很亲切吧)&lt;command&gt;节点后再加上end关键字。那么这个&lt;command&gt;是什么呢：

```
<command> ::= <repeat command> | <primitive command>
```

范式告诉我们，&lt;command&gt;是一个&lt;repeat command&gt;结点或是一个&lt;primitive command&gt;结点。

而&lt;repeat command&gt;：

```
<repeat command> ::= repeat <number> <command list>
```

它是repeat关键字后跟随一个&lt;number&gt;结点，再跟随一个前文介绍过的&lt;command list&gt;结点。从而形成了递归结构。

最后是&lt;primitive command&gt;：

```
<primitive command> ::= go | left | right
```

它是go,left,right这3个关键字中的某一个。在编程领域，像&lt;primitive command&gt;这样描述它的信息中不包含其他结点的，或者更直观的说，我们可以仅仅通过描述语句本身就完全弄明白含义的结点，被称为终结符表达式(Terminal Expression)。与之相对的，像&lt;command list&gt;，&lt;command&gt;等需要进一步展开的结点则被称为非终结符表达式(Nonterminal Expression)。所谓递归，其实就是因非终结符表达式而起，最后收束于终结符表达式。

细心的朋友想必可以发现，我们还剩一个&lt;number&gt;结点没有描述。其实是应该写的，它应该是一个大于等于0的整数，是一个终结符表达式。只不过它的描述较为复杂，这里便省略了。

完成了语法的数学化描述后，下一步就是根据该描述将迷你语言的代码依据语法规则翻译为语法树，并存入宿主语言(本文使用Java)中，这一步被称为解析。而后再执行语法树中存储的语义，得到输出。这么说还是会有一些抽象，还是让我们赶紧来看具体的代码吧。

本程序中的所有代码将被统一置于design23包下，结构如下：

![8.jpg](/images/blog_pic/Java 设计模式/23Interpreter模式/8.jpg)

迷你语言的示例程序使用前文介绍过的那个重复绘制1000次的复制图形。它被存储在名为code.txt的文件中。

**code.txt**

```
program
  repeat 1000
    repeat 4
      repeat 3 go right go left end
      right
    end
  end
end
```

然后我们介绍本文的重点，也就是model.parse包下的内容。这个包下的代码完成了前文说到的迷你语言的语法树的构建(即解析)以及执行。秉承Java语言"一切都是对象，对象就是一切"的思想，上文介绍的EBNF中的结点最终都被翻译为了类，这在后文的代码中将会有很明显的体现。

**Node类**

```
package design23.model.parse;

import design23.model.entity.GameMap;

public abstract class Node {

    public abstract void parse(Context context) throws ParseException;

    public abstract void exe(GameMap map);
}
```

Node类是所有结点的抽象父类。按照需求，它约束自身的子类必须实现两个基本的功能：parse() --> 解析 及 exe() --> 执行。很显然，这两个方法的调用是有先后顺序的，即我们必须先完成解析，形成语法树，才能执行。

在解析方法parse()的方法定义中，我们抛出了自定义异常ParseException：

**ParseException类**

```
package design23.model.parse;

public class ParseException extends Exception {

    private static final long serialVersionUID = 1L;

    ParseException(String msg) {
        super(msg);
    }
}
```

这只是一个很简单的异常类，就不多做赘述了。

**ProgramNode类**

```
package design23.model.parse;

import design23.model.entity.GameMap;

/**
 * <program> ::= program <command list>
 */
public class ProgramNode extends Node {

    private CommandListNode commandListNode;

    private static final String PROGRAM_STR = "program";

    @Override
    public void parse(Context context) throws ParseException {
        context.skip(ProgramNode.PROGRAM_STR);
        this.commandListNode = new CommandListNode();
        this.commandListNode.parse(context);
    }

    @Override
    public String toString() {
        return "[" + ProgramNode.PROGRAM_STR + " " + this.commandListNode + "]";
    }

    @Override
    public void exe(GameMap map) {
        this.commandListNode.exe(map);
    }
}
```

终于到EBNF中介绍的真正的结点了。很显然，ProgramNode对应&lt;program&gt;，这在类首的注释中也有所体现。我们会为每一个结点都加上类似的类首注释，以表明它代表的是EBNF中的哪个结点，因此后文在介绍其他结点时就不会再显式的点出这种对应关系了。

关于将EBNF翻译为实际的类，有一个很重要的点就是"不要做多余的事"。简单来说，在我们编写ProgramNode时，能参照的就仅仅只是:

```
<program> ::= program <command list>
```

其他的一切都不需要知道。也不要耍小聪明做一些EBNF中没有的事，如果实在是想加新功能，也需要修改EBNF，而不是在翻译出的代码中自作主张。语法的设计在EBNF中已全部完成了，从EBNF到具体的代码仅仅只需要机械的翻译。

因此，这些结点的代码本身其实没什么好说的，后续结点的代码我们将快速的贴出。

**CommandListNode类**

```
package design23.model.parse;

import java.util.ArrayList;
import java.util.List;

import design23.model.entity.GameMap;

/**
 * <command list> ::= <command>* end
 */
public class CommandListNode extends Node {

    private List<CommandNode> commandList = new ArrayList<CommandNode>();

    @Override
    public void parse(Context context) throws ParseException {
        while (true) {
            String current = context.peek();
            String endstr = "end";
            if (null == current) throw new ParseException("Missing '" + endstr + "'");
            if (endstr.equals(current)) {
                context.skip(endstr);
                break;
            }
            CommandNode commandNode = new CommandNode();
            this.commandList.add(commandNode);
            commandNode.parse(context);
        }
    }

    @Override
    public String toString() {
        return this.commandList.toString();
    }

    @Override
    public void exe(GameMap map) {
        for (CommandNode commandNode : this.commandList)
            commandNode.exe(map);
    }
}
```

**CommandNode类**

```
package design23.model.parse;

import design23.model.entity.GameMap;

/**
 * <command> ::= <repeat command> | <primitive command>
 */
public class CommandNode extends Node {

    private CommandNode commandNode;

    protected static final String REPEAT_STR = "repeat";

    @Override
    public void parse(Context context) throws ParseException {
        this.commandNode = CommandNode.REPEAT_STR.equals(context.peek()) ?
                           new RepeatCommandNode() :
                           new PrimitiveCommandNode();
        this.commandNode.parse(context);

    }

    @Override
    public String toString() {
        return this.commandNode.toString();
    }

    @Override
    public void exe(GameMap map) {
        this.commandNode.exe(map);
    }
}
```

**RepeatCommandNode类**

```
package design23.model.parse;

import design23.model.entity.GameMap;

/**
 * <repeat command> ::= repeat <number> <command list>
 */
public class RepeatCommandNode extends CommandNode {

    private int number;

    private CommandListNode commandListNode;

    @Override
    public void parse(Context context) throws ParseException {
        context.skip(CommandNode.REPEAT_STR);
        try {
            this.number = Integer.parseInt(context.peek());
        } catch (NumberFormatException e) {
            throw new ParseException("fail to parse " + CommandNode.REPEAT_STR + " times." + e);
        }
        context.next();
        this.commandListNode = new CommandListNode();
        this.commandListNode.parse(context);
    }

    @Override
    public String toString() {
        return "[" + CommandNode.REPEAT_STR + " " + this.number + " " + this.commandListNode + "]";
    }

    @Override
    public void exe(GameMap map) {
        for (int i = 0; i < this.number; i++)
            this.commandListNode.exe(map);
    }
}
```

**PrimitiveCommandNode类**

```
package design23.model.parse;

import design23.model.entity.GameMap;
import design23.model.entity.Grid;

/**
 * <primitive command> ::= go | left | right
 */
public class PrimitiveCommandNode extends CommandNode {

    private String name;

    public static final String GO = "go";

    public static final String LEFT = "left";

    public static final String RIGHT = "right";

    @Override
    public void parse(Context context) throws ParseException {
        this.name = context.peek();
        context.skip(this.name);
        if (!PrimitiveCommandNode.GO.equals(this.name) &&
            !PrimitiveCommandNode.LEFT.equals(this.name) &&
            !PrimitiveCommandNode.RIGHT.equals(this.name))
            throw new ParseException("command " + this.name + " is undefined");
    }

    @Override
    public String toString() {
        return this.name;
    }

    @Override
    public void exe(GameMap map) {
        int maxR = map.getGrids().length;
        int maxC = map.getGrids()[0].length;
        int focusR = map.getFocusR();
        int focusC = map.getFocusC();
        Grid focusGrid = map.getGrids()[focusR][focusC];
        int direction = focusGrid.getDirection();
        if (PrimitiveCommandNode.GO.equals(this.name)) {
            int newFocusR = focusR;
            int newFocusC = focusC;
            switch (direction) {
            case 0:
                if (focusR + 1 == maxR) break;
                newFocusR = focusR + 1;
                break;
            case 1:
                if (focusC == 0) break;
                newFocusC = focusC - 1;
                break;
            case 2:
                if (focusC + 1 == maxC) break;
                newFocusC = focusC + 1;
                break;
            case 3:
                if (focusR == 0) break;
                newFocusR = focusR - 1;
            }
            // 模拟走路动作
            for (int i = 0; i < map.getStageCount(); i++) {
                try {
                    Thread.sleep(300);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                focusGrid.setStage((focusGrid.getStage() + 1) % map.getStageCount());
                if (i == map.getStageCount() / 2) {
                    // 设置新的焦点格
                    map.getGrids()[newFocusR][newFocusC].setDirection(direction);
                    map.getGrids()[newFocusR][newFocusC].setStage(focusGrid.getStage());
                    map.getGrids()[newFocusR][newFocusC].setTimes(map.getGrids()[newFocusR][newFocusC].getTimes() + 1);
                    map.setFocusR(newFocusR);
                    map.setFocusC(newFocusC);
                    // 设置原来的焦点格
                    focusGrid.setDirection(-1);
                    focusGrid.setStage(-1);
                }
            }
        } else if (PrimitiveCommandNode.LEFT.equals(this.name)) {
            switch (direction) {
            case 0:
                focusGrid.setDirection(2);
                break;
            case 1:
                focusGrid.setDirection(0);
                break;
            case 2:
                focusGrid.setDirection(3);
                break;
            case 3:
                focusGrid.setDirection(1);
            }
        } else {
            switch (direction) {
            case 0:
                focusGrid.setDirection(1);
                break;
            case 1:
                focusGrid.setDirection(3);
                break;
            case 2:
                focusGrid.setDirection(0);
                break;
            case 3:
                focusGrid.setDirection(2);
            }
        }
    }
}
```

至此，所有的结点我们都介绍完了。不过，为了能让翻译器接收到源码，我们还写了Context类：

**Context类**

```
package design23.model.parse;

import java.util.StringTokenizer;

public class Context {

    private StringTokenizer st;

    private String current;

    public Context(String text) {
        this.st = new StringTokenizer(text);
        this.next();
    }

    String peek() {
        return this.current;
    }

    String next() {
        this.current = this.st.hasMoreTokens() ?
                       this.st.nextToken() :
                       null;
        return this.current;
    }

    void skip(String token) throws ParseException {
        if (!token.equals(this.current))
            throw new ParseException("need " + token + ", but " + this.current + " is found");
        this.next();
    }
}
```

Context类的功能很简单，它将源码按分割符分割为一个个单词，然后作为解析器会逐个读取单词，而后根据单词生成新的结点，并将结点挂载到语法树合适的位置上。随着语法树的构建，Context中的文本信息会越来越少，直至读完。

至此，本文要介绍的核心功能就已经描述完了。后文要介绍的代码都只是为了得到一个可视化的结果。

首先来看model.entity包，这个包下存储了程序要展示的一些实体。一如前文中的截图看到的，我们希望游戏最终能实现RPG制作大师做出的游戏的效果。因此我们需要一张存储所有要素的游戏地图，该地图是由一个个"小格子"组成的。

**GameMap类**

```
package design23.model.entity;

public class GameMap {

    private Grid[][] grids;

    private int focusR;

    private int focusC;

    private int stageCount;

    /**
     * @param row 地图上小格子的行数
     * @param column 地图上小格子的列数
     * @param stageCount 人物的行走状态
     * @param focusR 初始时人物所在的小格子的行号
     * @param focusC 初始时人物所在的小格子的列号
     * @param initDirection 初始时人物的朝向 
     */
    public GameMap(int row, int column, int stageCount, int focusR, int focusC, int direction) {
        this.stageCount = stageCount;
        this.grids = new Grid[row][column];
        for (int r = 0; r < row; r++)
            for (int c = 0; c < column; c++)
                this.grids[r][c] = new Grid();
        this.focusR = focusR;
        this.focusC = focusC;
        this.grids[this.focusR][this.focusC].setDirection(direction);
        this.grids[this.focusR][this.focusC].setStage(1);
        this.grids[this.focusR][this.focusC].setTimes(1);
    }

    public Grid[][] getGrids() {
        return grids;
    }

    public int getFocusR() {
        return focusR;
    }

    public void setFocusR(int focusR) {
        this.focusR = focusR;
    }

    public int getFocusC() {
        return focusC;
    }

    public void setFocusC(int focusC) {
        this.focusC = focusC;
    }

    public int getStageCount() {
        return stageCount;
    }
}
```

**Grid类**

```
package design23.model.entity;

public class Grid {

    private int direction = -1;

    private int stage = -1;

    private int times;

    public int getDirection() {
        return direction;
    }

    public void setDirection(int direction) {
        this.direction = direction;
    }

    public int getStage() {
        return stage;
    }

    public void setStage(int stage) {
        this.stage = stage;
    }

    public int getTimes() {
        return times;
    }

    public void setTimes(int times) {
        this.times = times;
    }
}
```

调用了GameMap的构造函数后，我们便构建了这样的一张地图：

- row行column列
- 初始时人物所在的格子位置：grids[this.focusR][this.focusC]
- 只会有一个格子处于焦点状态。未处于焦点状态的格子的direction和stage均默认为-1。
- Grid的times属性是指人物来到该格子上的次数，绘制轨迹用。

这里需要简单介绍下direction与stage的含义。其中direction表示人物当前的朝向。而stage则表示行走状态，在人物从1个格子移动到另一个格子的过程中，我们一共会变化出3张图片，从而模拟出"人物移动的动作"。

这两个字段实际都是服务于图character.png：

![9.png](/images/blog_pic/Java 设计模式/23Interpreter模式/9.png)

纵向来看，0-3分别代表下左右上四个朝向。而横向的0-2则代表三个行走状态：站立不动是中间的那张状态1。当人物从一个格子移动到另一个格子上时，发生的变化为：状态1 --> 状态2 --> 状态0 --> 状态1。

然后我们提供了将parse包及entity包整合起来并对外(指得就是View啦)提供功能接口的Model类：

**Model类**

```
package design23.model;

import java.io.BufferedReader;
import java.io.File;
import java.io.FileInputStream;
import java.io.InputStreamReader;

import org.apache.commons.lang.StringUtils;

import design23.model.entity.GameMap;
import design23.model.parse.Context;
import design23.model.parse.ParseException;
import design23.model.parse.ProgramNode;

public class Model {

    private GameMap map;

    private ProgramNode root = new ProgramNode();

    public void initMap(int row, int column, int stageCount, int focusR, int focusC, int direction) {
        this.map = new GameMap(row, column, stageCount, focusR, focusC, direction);
    }

    public void initInterpreter(String path) {
        try {
            this.root.parse(new Context(this.loadCode(path)));
        } catch (ParseException e) {
            e.printStackTrace();
        }
        System.out.println(this.root);
    }

    public void exe() {
        new Thread() {
            @Override
            public void run() {
                Model.this.root.exe(Model.this.map);
            }
        }.start();
    }

    public GameMap getMap() {
        return map;
    }

    private String loadCode(String path) {
        StringBuilder sb = new StringBuilder();
        FileInputStream fi = null;
        InputStreamReader ir = null;
        BufferedReader br = null;
        try {
            fi = new FileInputStream(new File(path));
            ir = new InputStreamReader(fi, "UTF-8");
            br = new BufferedReader(ir);
            String lineTxt = null;
            while (!StringUtils.isBlank((lineTxt = br.readLine())))
                sb.append(lineTxt).append(" ");
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            try {
                br.close();
                ir.close();
                fi.close();
            } catch (Exception ef) {
                ef.printStackTrace();
            }
        }
        return sb.toString();
    }
}
```

然后是作为GUI显示的View类，本文使用Java AWT：

**View类**

```
package design23.view;

import java.awt.Color;
import java.awt.Frame;
import java.awt.Graphics;
import java.awt.Image;
import java.awt.event.WindowAdapter;
import java.awt.event.WindowEvent;
import java.awt.image.BufferedImage;
import java.io.IOException;

import javax.imageio.ImageIO;

import design23.model.Model;
import design23.model.entity.Grid;

public class View extends Frame {

    private static final long serialVersionUID = 1L;

    private int gridSize = 32;

    private int widebar = 31;

    private int narrowbar = 6;

    private Model model = new Model();

    private BufferedImage img = this.loadImg("character.png");

    public void launchFrame() {
        int gridRow = 7;
        int gridColumn = 7;
        this.model.initMap(gridRow, gridColumn, 3, 3, 0, 3);
        this.model.initInterpreter("src/main/java/design23/code.txt");
        super.setLocation(800, 300);
        super.setSize(this.narrowbar + this.gridSize * gridColumn + this.narrowbar, this.widebar + this.gridSize * gridRow + this.narrowbar);
        new Thread(this.new RepaintRunnable()).start();
        super.addWindowListener(
            new WindowAdapter() {
                @Override
                public void windowClosing(WindowEvent e) {
                    System.exit(0);
                }
            }
        );
        this.model.exe();
        super.setVisible(true);
    }

    @Override
    public void paint(Graphics g) {
        Grid[][] grids = this.model.getMap().getGrids();
        for (int r = 0; r < grids.length; r++) {
            for (int c = 0; c < grids[0].length; c++) {
                Grid grid = grids[r][c];
                int dx1 = this.narrowbar + c * this.gridSize;
                int dy1 = this.widebar + r * this.gridSize;
                if (grid.getTimes() > 0) {
                    Color tempColor = g.getColor();
                    g.setColor(Color.GRAY);
                    g.fillRect(dx1, dy1, this.gridSize, this.gridSize);
                    g.setColor(tempColor);
                }
                if (grid.getDirection() >= 0) {
                    int direction = grid.getDirection();
                    int stage = grid.getStage();
                    g.drawImage(this.img,
                                dx1,
                                dy1,
                                this.narrowbar + c * this.gridSize + this.gridSize,
                                this.widebar + r * this.gridSize + this.gridSize,
                                stage * this.gridSize,
                                direction * this.gridSize,
                                stage * this.gridSize + this.gridSize,
                                direction * this.gridSize + this.gridSize,
                                null);
                }
            }
        }
    }

    private class RepaintRunnable implements Runnable {
        @Override
        public void run() {
            while (true) {
                try {
                    View.this.repaint();
                    Thread.sleep(10);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    @Override
    public void update(Graphics g) {
        Image bImage = super.createImage(this.getWidth(), this.getHeight());
        Graphics bg = bImage.getGraphics();
        this.paint(bg);
        bg.dispose();
        g.drawImage(bImage, 0, 0, this);
    }

    private BufferedImage loadImg(String name) {
        BufferedImage bImage = null;
        try {
            bImage = ImageIO.read(View.class.getClassLoader().getResource("design23/" + name));
        } catch (IOException e) {
            e.printStackTrace();
        }
        return bImage;
    }
}
```

在View的launchFrame()中，我们先调用model的initMap()初始化了地图，而后又调用model的initInterpreter()初始化了语法树。而后在需要的时候，也就是launchFrame()的最后，我们调用model的exe()方法来执行语法树。该方法会启一个新的线程，按设定的代码移动人物(其实就是改变地图中属性的值)。而View剩余要做的，就是不断的刷新并显示地图，将翻译器更新后的数据显示出来。

最后给出Main.java。执行该类后，即可得到结果。

```
package design23;

import design23.view.View;

public class Main {

    public static void main(String[] args) {
        new View().launchFrame();        
    }
}
```

需要注意的是，因为我们在Model初始化语法树时将语法树打印出来了。因此除了UI外，我们还会在控制台中得到这样的一句输出：

```
[program [[repeat 1000 [[repeat 4 [[repeat 3 [go, right, go, left]], right]]]]]]
```

大家可以对照前文的EBNF加深理解。

# 登场角色

上面的示例程序介绍了Interpreter模式的Java实现，下面咱们试着跳出语言层面，抽象出Interpreter模式中登场的角色。

**AbstractExpression(抽象表达式)**

定义结点的基本约束。在示例程序中，由Node类扮演这个角色。

**TerminalExpression(终结符表达式)**

即EBNF中的终结符表达式，在示例程序中，由PrimitiveCommandNode类扮演这个角色。

**NonterminalExpression(非终结符表达式)**

即EBNF中的非终结符表达式，在示例程序中，由ProgramNode类，CommandListNode类，CommandNode类，RepeatCommandNode类联袂扮演这个角色。

**Context(上下文)**

在示例程序中，由Context类扮演这个角色。

# 迷你语言可以有哪些？

其实说白了，只要功能允许，也就是语言A中所表述的实体在语言B中实际存在，那么就可以使用Interpreter模式将语言A至于语言B之上。下面列出的是几种常用和热门的。

**批处理语言**

也就是所谓的第二代语言。这种语言基本就是单词的拼接，能表达的含义远没有第三代语言多。但是在某些本来就不需要那么多含义的场景下，使用批处理语言会使程序"更纯粹"，反而有利于开发和维护。本文示例程序中设计的迷你语言即属于批处理语言。

**正则**

大多数主流语言都支持对正则语法的解析。实现细节虽然千差万别，但核心思想基本不出本文藩篱。

**自然语言**

自然语言的语义识别和语法树的构建是人工智能的关键技术之一。