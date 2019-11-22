---
title: Java 设计模式-16.Mediator模式
date: 2018-08-23 18:55:49
tags: [Java,设计模式]
categories: Java 设计模式
---

在《图解设计模式》一书中，Mediator模式被归入了第7部分[简单化]()。在GoF原书中，Mediator模式则被归入了[行为型设计模式]()。

<!-- more -->

# 综述

假设有这样一个开发小组，组内共有10位程序员，每人负责一个模块，共同开发一个项目。既然同属一个项目，各模块自然需要协同工作。通常来说，每个模块都需对接多个其他的模块，显然这违反了最少知识法则(迪米特法则)，使得各模块间耦合得比较严重。不仅如此，每个程序员都会自然而然的更偏向于自己负责的模块，例如要添加一个需求时，我们很可能会听到如下对话：

小张："这个需求不能放在我这做，因为这会导致我这的时间复杂度大幅提升。这是不能接受的，小王你负责的那个模块也能做吧。"

小王："这可不行，放在我这边的话我的功能层次结构就乱了，从逻辑上来讲就不该我做。"

小张："那用户响应慢10倍？这谁能接受？"

小王："实际功能问题能和架构问题比吗？架构乱了还怎么维护？而且性能这种东西总是可以通过代码优化提升的，你代码写的有问题吧！"

小张："you can you up啊，什么玩意就说提升就提升。。。"

就这样，每个人都考虑着自己模块的局部最优，进而指挥别人，导致整个项目组的气氛一点也不好，同时项目进度也会非常缓慢。

针对这个问题，一个有效的解决方案就是再为这个小组配备一个"调停者"。调停者并不负责具体的模块，也并不会细致的研究每个模块的技术细节，但是他会比较全面的了解所有模块的功能特点，从而起到调停小组中各组员的作用。

现在，小组中的各组员之间不会互相沟通，说得极端一些，他们都不需要知道彼此的存在(此时就遵循最少知识法则了)。在遇到问题时，他们都会把自身的看法告诉给调停者，调停者在汇聚所有人的想法后，会站在项目整体的角度上权衡利弊，选择出最合理的方案，并将该方案告知所有组员，即便违背了组员局部最优的利益，组员也会无条件的遵循调停者的指示。

由此演化而来的设计模式就是Mediator模式。其中Mediator是"调停者"的意思，自然就是Mediator模式中最核心的角色了。此外，继承上文的例子，该模式的另一个重要角色被称为Colleague，也就是"组员"。很显然，组员会有很多个，而调停者通常只会有1位。

在此我们可以再举一个更常见的例子：电脑。一台电脑若要正常工作，需要它的所有组件：CPU，内存，显卡，各种IO设备等协同工作，而且这种协同关系往往是很复杂的。对此，我们并没有将所有原件错综复杂的连接在一起，而是准备了一张主板，主板并不承担具体的功能，它的作用就是让所有的原件插到自己身上。这样某原件会通过主板向外界传递信息，同时也会从主板那里接到指令。这样整个系统的关系就从各器件间错综复杂的勾连进化到了大家都只和主板交互。显然，在这个例子中，主板就是调停者，而各器件则是组员。

下面我们不妨考虑一种极端的情况：假设有n个Colleague，它们两两之间都需要双向通信，那么共需要通路n(n-1)条。而引入Mediator模式后，同样的需求下只需要2n的通路，通路的复杂度相当于降低了一个量级。在n越大，连接关系越复杂时，Mediator模式对通路的优化效果越大。

如果我们站在一个更大的视角上来看待问题的话，调停者担任的其实是统筹全局的角色。在过去已经介绍过的设计模式中，类似作用的角色其实很常见了，例如[3.Template Method模式]()中的AbstractClass(抽象类)，[4.Factory Method模式]()中的Creator(创建者)，[7.Builder模式]()中的Designer(设计者)等。

# 示例程序

下面我们来看一个应用了Mediator模式的示例程序。这是一个使用Java AWT制作的登录窗：

![0.jpg](/images/blog_pic/Java 设计模式/16Mediator模式/0.jpg)

作为一个登录框，我们希望它具备如下功能：

- 任何时候，取消(Cancel)按钮均会放开，点击该按钮会导致登录窗关闭，其效果和点击右上角的叉相同。
- 第一行是一个二选一，我们可以选择采用游客(Guest)模式或是登陆(Login)模式登入。
- 如果选择游客模式，用户名(Username)及密码(Password)文本框均会置灰。同时放开登陆(OK)按钮，点击该按钮后会直接以游客身份登入，登录框关闭。
- 如果选择登陆模式，用户名文本框会被放开，若此时用户名文本框中没有文字，则密码文本框及登陆按钮均会置灰。只有当用户名文本框中有文字后，密码文本框才会被放开，此时若密码文本框中没有文字，则登陆按钮依然置灰，反之，登陆按钮放开。此时若点击登陆按钮，若用户名密码匹配，则会以该身份登入，登录框关闭。反之，不做任何操作(会在控制台打印一句提示语句)。

如果是这样掰开了揉碎了仔细分析的话，小小的登陆框的逻辑其实还是挺复杂。通常我们会将每个组件都声明为一个实例：

- Guest选框
- Login选框
- Username文本框
- Password文本框
- OK按钮
- Cancel按钮

按照上文的需求，这些实例之间的关系错综复杂(这么说有点过了哈，稍微夸张一下)，此时就可以使用Mediator模式啦。很显然，我们应该将这些组件视为Colleague。而Mediator则让承载这些组件的Frame担任。如此看来，Java AWT中的Frame的作用其实就是相当于计算机中的主板，本身不承载具体的功能，而是负责组件的安插及沟通。

说了这么多需求，下面就来看看具体的代码吧。首先是类图：

![1.jpg](/images/blog_pic/Java 设计模式/16Mediator模式/1.jpg)

本程序中的所有代码将被统一置于design16包下，结构如下：

![2.jpg](/images/blog_pic/Java 设计模式/16Mediator模式/2.jpg)

其中Main.java是测试代码，并没有出现在类图中。

下面将逐个贴出每个类的源码。

**Mediator接口**

```
package design16;

public interface Mediator {

    void colleagueChanged(Colleague colleague);
}
```

该接口内部只有一个方法，是供Colleague调用告知自身情况的。依本示例的需求，Colleague会把自身作为参数传递给调停者。实际应用时该方法可以是任何形式，甚至可以不是一个方法而是一组方法，只要目的不变即可。

**Colleague接口**

```
package design16;

public interface Colleague {

    void setColleagueEnabled(boolean enabled);
}
```

该接口内部同样只有一个方法，是供Mediator调用向各组员下达命令的。同上文的colleagueChanged()方法，该方法的组织形式依然多种多样，只要达到目的即可。

至此我们可以稍微总结一下了。Mediator作为调停者，提供了一个供组员调用报告自身情况的方法。同样Colleague作为组员，也提供了一个供调停者调用下达命令的方法。这样才能保证调停者与组员间信息沟通的通畅。在实际应用中，我们自然可以依据需要为Mediator或Colleague添加更多的代码，也可以改变colleagueChanged()及setColleagueEnabled()这两个方法的形式，不过这两个方法所代表的功能却是构成Mediator模式的基本要素，是一定要有的。

**ColleagueButton类**

```
package design16;

import java.awt.Button;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;

public class ColleagueButton extends Button implements ActionListener, Colleague {

    private static final long serialVersionUID = 1L;

    private Mediator mediator;

    public ColleagueButton(Mediator mediator, String caption) {
        super(caption);
        this.mediator = mediator;
    }

    @Override
    public void setColleagueEnabled(boolean enabled) {
        this.setEnabled(enabled);
    }

    @Override
    public void actionPerformed(ActionEvent e) {
        this.mediator.colleagueChanged(this);
    }
}
```

具体的组员之一，前文需求中的OK按钮及Cancel按钮就是它的实例。

既然Mediator设置了一个colleagueChanged()方法供组员调用，那么这里隐含的前提条件就是组员要知道自己的调停者是谁才行。因此ColleagueButton中添加了mediator字段，用以记录自身的调停者。本程序是以构造函数传入调停者的，实际使用时自然也可以在其他时机传入，只要用的时候有就行。

其实不仅仅ColleagueButton，在后文中我们会看到，所有的组员都需要做类似的操作。很显然，这是重复代码。那么既然如此，我们为什么不将mediator提到更高的层级呢？其原因就在于，Java是单继承的，这就使得Colleague只能被声明为接口，无法承载记录mediator的功能。

**ColleagueCheckbox类**

```
package design16;

import java.awt.Checkbox;
import java.awt.CheckboxGroup;
import java.awt.event.ItemEvent;
import java.awt.event.ItemListener;

public class ColleagueCheckbox extends Checkbox implements ItemListener, Colleague {

    private static final long serialVersionUID = 1L;

    private Mediator mediator;

    public ColleagueCheckbox(Mediator mediator, String caption, CheckboxGroup group, boolean state) {
        super(caption, group, state);
        this.mediator = mediator;
    }

    @Override
    public void setColleagueEnabled(boolean enabled) {
        this.setEnabled(enabled);
    }

    @Override
    public void itemStateChanged(ItemEvent e) {
        this.mediator.colleagueChanged(this);
    }
}
```

具体的组员之一，前文需求中的Guest选框及Login选框就是它的实例。

**ColleagueTextField类**

```
package design16;

import java.awt.Color;
import java.awt.TextField;
import java.awt.event.TextEvent;
import java.awt.event.TextListener;

public class ColleagueTextField extends TextField implements TextListener, Colleague {

    private static final long serialVersionUID = 1L;

    private Mediator mediator;

    public ColleagueTextField(Mediator mediator, String text, int columns) {
        super(text, columns);
        this.mediator = mediator;
    }

    @Override
    public void setColleagueEnabled(boolean enabled) {
        this.setEnabled(enabled);
        this.setBackground(enabled ? Color.WHITE : Color.LIGHT_GRAY);
    }

    @Override
    public void textValueChanged(TextEvent e) {
        this.mediator.colleagueChanged(this);
    }
}
```

具体的组员之一，前文需求中的Username文本框及Password文本框就是它的实例。

**LoginFrame类**

```
package design16;

import java.awt.CheckboxGroup;
import java.awt.Color;
import java.awt.Frame;
import java.awt.GridLayout;
import java.awt.Label;
import java.awt.event.WindowAdapter;
import java.awt.event.WindowEvent;
import java.util.HashMap;
import java.util.Map;

public class LoginFrame extends Frame implements Mediator {

    private static final long serialVersionUID = 1L;

    private static Map<String, String> USERS = new HashMap<String, String>();

    private ColleagueCheckbox checkGuest;

    private ColleagueCheckbox checkLogin;

    private ColleagueTextField textUser;

    private ColleagueTextField textPassword;

    private ColleagueButton buttonOK;

    private ColleagueButton buttonCancel;

    static {
        LoginFrame.USERS.put("reimu", "reimuwang");
    }

    public LoginFrame() {
        super("登陆窗口");
        this.setBackground(Color.LIGHT_GRAY);
        this.setLayout(new GridLayout(4, 2));
        // 生成并初始化设置各组件
        CheckboxGroup group = new CheckboxGroup();
        this.checkGuest = new ColleagueCheckbox(this, "Guest", group, true);
        this.checkLogin = new ColleagueCheckbox(this, "Login", group, false);
        int textTotalLength = 10;
        this.textUser = new ColleagueTextField(this, "", textTotalLength);
        this.textUser.setColleagueEnabled(false);
        this.textPassword = new ColleagueTextField(this, "", textTotalLength);
        this.textPassword.setEchoChar('*');
        this.textPassword.setColleagueEnabled(false);
        this.buttonOK = new ColleagueButton(this, "OK");
        this.buttonCancel = new ColleagueButton(this, "Cancel");
        // 设置Listener
        this.checkGuest.addItemListener(this.checkGuest);
        this.checkLogin.addItemListener(this.checkLogin);
        this.textUser.addTextListener(this.textUser);
        this.textPassword.addTextListener(this.textPassword);
        this.buttonOK.addActionListener(this.buttonOK);
        this.buttonCancel.addActionListener(this.buttonCancel);
        // 调停者记录自身需要管理哪些组员
        this.add(this.checkGuest);
        this.add(this.checkLogin);
        this.add(new Label("Username:"));
        this.add(this.textUser);
        this.add(new Label("Password:"));
        this.add(this.textPassword);
        this.add(this.buttonOK);
        this.add(this.buttonCancel);
        // 设置点击关闭时需执行的操作
        super.addWindowListener(
            new WindowAdapter() {
                @Override
                public void windowClosing(WindowEvent e) {
                    System.exit(0);
                }
            }
        );
        // 显示
        this.pack();
        this.setVisible(true);
    }

    @Override
    public void colleagueChanged(Colleague colleague) {
        String mode = this.checkGuest.getState() ? "游客模式" : "登陆模式";
        if (colleague == this.checkGuest) {
            System.out.println("切换至游客模式，用户名输入框被关闭，密码输入框被关闭");
            this.textUser.setColleagueEnabled(false);
            this.textPassword.setColleagueEnabled(false);
            return;
        }
        if (colleague == this.checkLogin) {
            System.out.print("切换至登陆模式，用户名输入框被放开。");
            this.textUser.setColleagueEnabled(true);
            this.controlUser();
            System.out.println();
            return;
        }
        if (colleague == this.textUser) {
            System.out.print(mode + "下向用户名输入框中键入文字。");    // 其实只可能是登陆模式，姑且记录下
            this.controlUser();
            System.out.println();
            return;
        }
        if (colleague == this.textPassword) {
            System.out.print(mode + "下向密码输入框中键入文字。");    // 其实只可能是登陆模式，姑且记录下
            this.controlPassword();
            System.out.println();
            return;
        }
        if (colleague == this.buttonOK) {
            System.out.print(mode + "下OK按钮被按下。");
            if (this.checkGuest.getState()) {
                System.out.println("以游客的身份进入成功，窗体退出");
                System.exit(0);
            } else {
                if (this.textPassword.getText().equals(LoginFrame.USERS.get(this.textUser.getText()))) {
                    System.out.println("登陆成功，窗体退出");
                    System.exit(0);
                } else
                    System.out.println("用户名与密码不比配");
            }
            return;
        }
        if (colleague == this.buttonCancel) {
            System.out.println("取消按钮被按下，窗体退出");
            System.exit(0);
        }
        System.out.println(colleague);
    }

    private void controlUser() {
        if (this.textUser.getText().length() > 0) {
            System.out.print("用户名输入框中有文字，密码输入框被放开。");
            this.textPassword.setColleagueEnabled(true);
            this.controlPassword();
        } else {
            System.out.print("用户名输入框中没有文字，密码输入框，OK按钮关闭。");
            this.textPassword.setColleagueEnabled(false);
            this.buttonOK.setColleagueEnabled(false);
        }        
    }

    private void controlPassword() {
        if (this.textPassword.getText().length() > 0) {
            System.out.print("密码输入框中有文字，OK按钮被放开。");
            this.buttonOK.setColleagueEnabled(true);
        } else {
            System.out.print("密码输入框中没有文字，OK按钮关闭。");
            this.buttonOK.setColleagueEnabled(false);
        }
    }
}
```

终于来到最重要的Mediator了。很明显，因为程序主体的控制逻辑都在调停者中，因此它的代码看起来比组员要长得多，也复杂得多。这其实可以看作Mediator模式的一个特征，由于Mediator角色需要掌控全局，因此控制逻辑都会集中到Mediator中，这会导致Mediator相对来说不易维护，同时更容易产生bug。不过这其实是可以接受的，正所谓两害相权取其轻，如果我们将控制逻辑分散在各个组员中，那么一旦产生bug，调试起来将更为困难，因为我们不得不梳理分散在各处的代码。在编程领域，我们将这种情况称为"分散灾难"。

类似于我们在讨论组员时分析过的，既然组员提供了setColleagueEnabled()方法供调停者调用，那么调停者就一定要知道它管理的组员有哪些才行。本示例直接将各组员作为调停者的字段记录在调停者中了，这算是最见到的做法了，具体使用时依情况不同也可以采用其他方式。

另外，在本示例中，组员的生成也是由调停者完成的，这是由Java AWT的特性(通常我们在用Java AWT写控件时，组件都是直接在面板内生成的)决定的，实际上，组员的生成及设置并不归调停者管。更常见的例子时，调停者与组员的生成都是独立的，二者需要做的仅仅是完成绑定关系。

**Main类**

```
package design16;

public class Main {

    public static void main(String[] args) {
        new LoginFrame();
    }
}
```

该类的代码很简单，执行该类后，即可显示窗体，并按需求约束的那样工作。

# 登场角色

上面的示例程序介绍了Mediator模式的Java实现，下面咱们试着跳出语言层面，抽象出Mediator模式中登场的角色。

**Mediator(调停者)**

在示例程序中，由Mediator接口扮演这个角色，该角色会提供一个渠道供Colleague报告自身的情况。

**ConcreteMediator(具体的调停者)**

在示例程序中，由LoginFrame类扮演这个角色。通常，该角色只会有一个。

**Colleague(同事)**

在示例程序中，由Colleague接口扮演这个角色，该角色会提供一个渠道接收Mediator的指令。

**ConcreteColleague(具体的同事)**

在示例程序中，由ColleagueButton，ColleagueCheckbox，ColleagueTextField联袂扮演这个角色。通常，该角色不止一个。

下面是抽象后，无关语言的类图：

![3.jpg](/images/blog_pic/Java 设计模式/16Mediator模式/3.jpg)