### java swing总结

### 摘要

swing的组件介绍与常用代码demo.没有最好的技术，只有在特定场景下最合适的技术



### 关键字

swing组件学习，swing UI美化



### 1、swing使用体验

刚开始出于好奇心想体验下java开发的桌面程序，网上搜索了一把，主流的就swing和javafx。一番体验和对比。最终还是倒向了swing.对于一个后端开发来说，swing绝对是完胜。理解好几个重要的概念，界面的事情交给框架和三方。

下面从几个维度对这两个技术简单的对比一下：

|      | swing |   javafx   |
| ---- | ---- | ---- |
|  界面  |  界面一般  | 理论上完全可以自定义，因为支持css |
|  技术新颖  |  接近20年的历史  | 5年左右的历史 |
|  技术成熟度  |  久经考验，文档丰富  | 尚在孵化中 |
|  市面接纳度  |  老牌的都在用  | 没见到几个产品再用 |

### 2、swing UI优化

swing默认的ui的确有点难看，但是它提供了一个lookandfeel来支持ui的优化。

lookandfeel 通俗的说，这就是皮肤；从功能上说。这是一种批量管理 Swing 控件外观的机制；

在启动你的程序前：

UIManager.setLookAndFeel(…);

就可以

假设是在程序已经启动之后再换 LookAndFeel，那在上面那句之后，建议再加上：

SwingUtilities.updateComponentTreeUI(…);

### 3、swing的的核心介绍

 为了有效地使用 Swing 组件，必须了解 Swing 包的层次结构和继承关系，其中比较重要的类是 Component 类、Container 类和 JComponent 类。

```java
  Java.lang.Object 类 --> Java.awt.Component 类 --> Java.awt.Container 类 --> Java.swing.JComponent 类 
```

  在 Swing 组件中大多数 GUI 组件都是 Component 类的直接子类或间接子类，JComponent 类是 Swing 组件各种特性的存放位置，这些组件的特性包括设定组件边界、GUI 组件自动滚动等。

  在 Swing 组件中最重要的父类是 Container 类，而 Container 类有两个最重要的子类，分别为 java.awt.Window 与 java.awt.Frame ，除了以往的 AWT 类组件会继承这两个类之外，现在的 Swing 组件也扩展了这两个类。顶层父类是 Component 类与 Container 类，所以 Java 关于窗口组件的编写，都与组件以及容器的概念相关联。

  1.3 常用的 Swing 组件概述

| 组件名称       | 定义                                           |
| -------------- | ---------------------------------------------- |
| JPanel       | 代表 容器      |
| JButton        | 代表 Swing 按钮，按钮可以带一些图片或文字      |
| JCheckBox      | 代表 Swing 中的复选框组件                      |
| JComBox        | 代表下拉列表框，可以在下拉显示区域显示多个选项 |
| JFrame         | 代表 Swing 的框架类                            |
| JDialog        | 代表 Swing 版本的对话框                        |
| JLabel         | 代表 Swing 中的标签组件                        |
| JRadioButton   | 代表 Swing 的单选按钮                          |
| JList          | 代表能够在用户界面中显示一系列条目的组件       |
| JTextField     | 代表文本框                                     |
| JPasswordField | 代表密码框                                     |
| JTextArea      | 代表 Swing 中的文本区域                        |
| JOptionPane    | 代表 Swing 中的一些对话框                      |

**所有的布局问题可以通过嵌套Jpanel来解决**

### 4、swing的常见问题

+ 界面卡顿，多线程解决
+ 界面难看，更换皮肤

### 5、可复用的代码段

#### 5.1、加载皮肤

```
/**
     * 初始化look and feel
     */
    public static void initTheme() {

        try {
            switch (App.config.getTheme()) {
                case "BeautyEye":
                    BeautyEyeLNFHelper.launchBeautyEyeLNF();
                    UIManager.put("RootPane.setupButtonVisible", false);
                    break;
                case "系统默认":
                    UIManager.setLookAndFeel(UIManager.getSystemLookAndFeelClassName());
                    break;
                case "weblaf":
                case "Darcula(推荐)":
                default:
                    if (SystemUtil.isLinuxOs()) {
                        UIManager.setLookAndFeel(UIManager.getSystemLookAndFeelClassName());
                    } else {
                        UIManager.setLookAndFeel("com.bulenkov.darcula.DarculaLaf");
                    }
            }
        } catch (Exception e) {
            logger.error(e);
        }
    }
```



#### 5.2、不同的layout

+ 1、 边界布局（BorderLayout）

+ 2、流式布局（FlowLayout）

+ 3、网格布局（GridLayout）

+ 4、盒子布局（BoxLaYout）

+ 5、空布局（null）

还有其他两种布局，分别是GridBagLayout（网格包布局）、CardLayout（卡片布局）

注意：JFrame和JDialog默认布局为BorderLayout，JPanel和Applet默认布局为FlowLayout

#### 5.3、boder

![border的使用](https://cdn.jsdelivr.net/gh/chen-xing/figure_bed/cdn/20210109214233847.png)

```
this.setBorder(BorderFactory.createEmptyBorder(20, 20, 20, 20));
this.setBorder(BorderFactory.createMatteBorder(1, 0, 0, 0, Color.lightGray));
```

#### 5.4、tab

```
public TabbedPaneDemo() {
        super(new GridLayout(1, 1));
        JTabbedPane tabbedPane = new JTabbedPane();
        ImageIcon icon = createImageIcon("tab.jp1g");
        JComponent panel1 = makeTextPanel("计算机名");
        tabbedPane.addTab("计算机名", icon, panel1, "Does nothing");
        tabbedPane.setMnemonicAt(0, KeyEvent.VK_1);
        JComponent panel2 = makeTextPanel("硬件");
        tabbedPane.addTab("硬件", icon, panel2, "Does twice as much nothing");
        tabbedPane.setMnemonicAt(1, KeyEvent.VK_2);
        JComponent panel3 = makeTextPanel("高级");
        tabbedPane.addTab("高级", icon, panel3, "Still does nothing");
        tabbedPane.setMnemonicAt(2, KeyEvent.VK_3);
        JComponent panel4 = makeTextPanel("系统保护");
        panel4.setPreferredSize(new Dimension(410, 50));
        tabbedPane.addTab("系统保护", icon, panel4, "Does nothing at all");
        tabbedPane.setMnemonicAt(3, KeyEvent.VK_4);
        add(tabbedPane);
    }
```

#### 5.5、tree

```
private JPanel createComponent()
    {
        JPanel panel=new JPanel();
        DefaultMutableTreeNode root=new DefaultMutableTreeNode("教师学历信息");
        String Teachers[][]=new String[3][];
        Teachers[0]=new String[]{"王鹏","李曼","韩小国","穆保龄","尚凌云","范超峰"};
        Teachers[1]=new String[]{"胡会强","张春辉","宋芳","阳芳","朱山根","张茜","宋媛媛"};
        Teachers[2]=new String[]{"刘丹","张小芳","刘华亮","聂来","吴琼"};
        String gradeNames[]={"硕士学历","博士学历","博士后学历"};
        DefaultMutableTreeNode node=null;
        DefaultMutableTreeNode childNode=null;
        int length=0;
        for(int i=0;i<3;i++)
        {
            length=Teachers[i].length;
            node=new DefaultMutableTreeNode(gradeNames[i]);
            for (int j=0;j<length;j++)
            {
                childNode=new DefaultMutableTreeNode(Teachers[i][j]);
                node.add(childNode);
            }
            root.add(node);
        }
        JTree tree=new JTree(root);
        panel.add(tree);
        panel.setVisible(true);
        return panel;
    }
```



#### 5.6、table

```
 public static void main(String[] agrs)
    {
        JFrame frame=new JFrame("学生成绩表");
        frame.setSize(500,200);
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        Container contentPane=frame.getContentPane();
        Object[][] tableDate=new Object[5][8];
        for(int i=0;i<5;i++)
        {
            tableDate[i][0]="1000"+i;
            for(int j=1;j<8;j++)
            {
                tableDate[i][j]=0;
            }
        }
        String[] name={"学号","软件工程","Java","网络","数据结构","数据库","总成绩","平均成绩"};
        JTable table=new JTable(tableDate,name);
        contentPane.add(new JScrollPane(table));
        frame.setVisible(true);
    }
```



#### 5.7、BOX

`BoxLayout`，箱式布局管理器。它把若干组件按水平或垂直方向依次排列放置。Swing 提供了一个实现了 BoxLayout 的容器组件`Box`。使用 Box 提供的静态方法，可快速创建水平/垂直箱容器(Box)，以及填充组件之间空隙的不可见组件。用水平箱和垂直箱的组合嵌套可实现类似于 GridBagLayout 的效果，但没那么复杂。

```
// 创建一个水平箱容器
Box hBox = Box.createHorizontalBox();
    
// 创建一个垂直箱容器
Box vBox = Box.createVerticalBox();
```

Box 内的组件之间默认没有空隙并居中，如果想在组件之间（或头部/尾部）添加空隙，可以在其中添加一个影响布局的不可见组件。Box 提供了三种用于填充空隙的不可见组件：glue、struts 和 rigidAreas。

创建 胶状（宽/高可伸缩）的不可见组件（glue）:

```
// 创建一个 水平方向胶状 的不可见组件，用于撑满水平方向剩余的空间（如果有多个该组件，则平分剩余空间）
Component hGlue = Box.createHorizontalGlue();

// 创建一个 垂直方向胶状 的不可见组件，用于撑满垂直方向剩余的空间（如果有多个该组件，则平分剩余空间）
Component vGlue = Box.createVerticalGlue();

// 创建一个 水平和垂直方向胶状 的不可见组件，用于撑满水平和垂直方向剩余的空间（如果有多个该组件，则平分剩余空间）
Component glue = Box.createGlue();
```



创建 固定宽度或高度 的不可见组件（struts）:

```
// 创建一个 固定宽度 的不可见组件（用于水平箱）
Component hStrut = Box.createHorizontalStrut(int width);

// 创建一个 固定高度 的不可见组件（用于垂直箱）
Component vStrut = Box.createVerticalStrut(int height);
```



创建 固定宽高 的不可见组件（rigidAreas）:

```
// 创建 固定宽高 的不可见组件
Component rigidArea = Box.createRigidArea(new Dimension(int width, int height));
```



### 6、启动方式

```
start javaw -jar rgtool-1.0-SNAPSHOT.jar
```

java和javaw命令对比

**相同点：**二者都是Java的虚拟机，用来执行Java程序

**区别：**

　　**1、 javaw.exe**运行程序时**不会输出控制台信息 (注**：“w”就是window的意思**)**。

使用案例 start.bat（y以下代码内容） 与 jar 包目录平级 

```
-- 未配置 环境变量的情况下
start  D:\pro\jre7\bin\javaw -jar  data.jar
-- 已配置 环境变量的情况下
start  javaw -jar  data.jar
```

 

　　**2、java.exe 会显示在控制台中输出信息，关闭窗口则程序停止。 \**
\****

使用案例

```
start  D:\pro\jre7\bin\java -jar  data.jar
```

