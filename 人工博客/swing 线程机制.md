### swing UI编程指南

### 1、swing 线程机制

Swing程序往往包括了三种类型的线程，分别是：

+ 初始化线程（Initial Thread）

+ 事件调度线程（Event Dispatch Thread，EDT）

+ 任务线程（Work Thread）

 

每个程序都从main方法开始执行，该方法一般运行在初始化线程上，初始化线程主要负责启动程序的UI界面，一旦UI界面启动完毕，初始化线程的工作便宣告结束。

每个Swing程序都会有一个EDT，EDT主要负责绘制和更新界面，并响应用户输入。每个EDT都会负责管理一个事件队列（EventQueue），而用户每次对界面更新的请求（包括键盘鼠标事件等）都会排到事件队列中，然后等待EDT的处理。

工作线程主要负责执行和界面无直接关系的耗时任务和输入/输出密集型操作，也即任何高染或延迟UI事件的处理都应该由任务线程来完成。

 

### 2、Swing编程中的注意点

 

在编写Swing程序的时候，必须注意：

+ 不能从其他非EDT线程来访问UI组件和事件处理器，否则可能会使程序出现非线程安全问题。

+ 不能在EDT中执行耗时任务，这会使得GUI事件被阻塞在队列中而得不到处理，使程序失去响应性。

 

 

### 3、如何正确地启动UI界面

 

错误的启动UI界面的方法

```
public class MainFrame extends javax.swing.JFrame {
  // ...
  public static void main(String[] args) {
    new MainFrame().setVisible(true);
  }
}

// 正确的启动UI界面的方法
public class MainFrame extends javax.swing.JFrame {
  // ...
  public static void main(String[] args) {
    SwingUtilities.invokeLater(new Runnable() {
      public void run() {
        new MainFrame().setVisible(true);
      }
    });
  }
}
```
### 4、关于SwingUtilities类

SwingUtilities提供了最常用的invokeAndWait()方法和invokeLater()方法，其他线程通过这两个方法可以将代码放 到事件队列中，当EDT进入该代码块后，就开始执行，并对UI组件进行安全修改。这两个方法又有所区别，invokeLater()方法是异步的，即 EDT将将事件放到队列中就返回；而invokeAndWait()方法是同步的，即EDT将事件放到队列中等到其Runnable执行完毕才返回，所以 注意绝对不能使用EDT来调用invokeAndWait()方法，否则会导致死锁发生 。




如何改善Swing程序的响应性

 

在上一篇文章中，曾经说过不能在EDT中执行耗时的任务，主要的原因是因为当EDT在执行这些耗时任务的时候，不能及时更新UI界面，这个时候整个界面是 处于“卡住”状态的，不能接受任何事件（如键盘输入等）和描绘界面，容易给用户一种程序“死掉”的感觉，非常不友好，比如下面的代码是不受欢迎的。

```
button.addActionListener(new ActionListener() {
  public void actionPerformed(ActionEvent e) {
    writeHugeData();          // 执行耗时的写文件任务
    jLabel.setText("Writting data..."); // 试图立刻改变 jLabel 的内容
  }
});
```
这个时候我们应该很容易地想到使用一个新线程来处理writeHugeData()任务，即把耗时的任务从EDT中剥离出来，这个时候代码变成

```
button.addActionListener(new ActionListener() {
  public void actionPerformed(ActionEvent e) {
    // 启动一个新线程来处理耗时工作
    new Thread(new Runnable() {
      public void run() {
        writeHugeData();
        jLabel.setText("Writting data...");
      }
    }).start();
  }
}); 
```

上面的代码应该是很多人经常写的，但是 注意这里它违背了前面提到的一个注意事项：更新界面的操作必须由EDT中去完成 。也即上面修改jLabel状态的代码不能由新建的线程完成，否则程序随时存在着死锁的危险。然后，我们可以将上面的代码改成

```
button.addActionListener(new ActionListener() {
  public void actionPerformed(ActionEvent e) {
    new Thread(new Runnable() {
      public void run() {
        writeHugeData();
        // 将更新界面的代码放到事件队列中
        SwingUtilities.invokeLater(new Runnable() {
          public void run() {
            jLabel.setText("Writting data...");
          }
        });
      }
    }).start();
  }
});
```
这个时候已经能解决所有问题了，但是如果这样做的话每次更新界面都需要新建一个Runnable的实例，对程序的性能有点浪费。其实最佳的方法应该是让一 个线程做好准备，运行和等待完成任务，这个线程我们称其为“任务线程”。EDT将负责搜集事件，使用同步化将其传递给等待的任务线程，并通过等待/通知机 制来告诉任务线程有新的要求。当任务线程完成了请求后，它将通过invokeLater()方法将更新界面的代码交给EDT去处理，然后任务线程将返回并 继续等待下一个通知。也就是在这样的背景下，SwingWorker类应运而生。

 

 

关于SwingWorke类

 

SwingWorker类是在JavaSE6中才出现的，它的目的是为了简化程序员开发任务线程的工作，SwingWorker可以与各种UI组件在多线 程的环境下交互，而不用程序员去过多关注。一般使用SwingWorker的做法是创建一个SwingWorker的子类，然后重写其 doInBackground()、done()和process()方法来实现我们需要完成的功能。上面的代码可以继续修改为

```
button.addActionListener(new ActionListener() {
  public void actionPerformed(ActionEvent e) {
    new SwingWorker<Long, Void>() {
      protected Long doInBackground() {
        // 执行耗时的写文件任务
        return writeHugeData();
      }
      protected void done() {
        try {
          jLabel.setText("Writting data...");
        } catch (Exception e) {
          e.printStackTrace();
        }
      }
    }.execute();
  }
});
```



### 5、jTable的更新

```
DefaultTableModel tModel = new DefaultTableModel(Object[][] data, Object[] columnNames); 
// 应用model到table
table = new JTable(tModel);

// 点击搜索按钮触发事件，用setDataVector()方法来更新Model的数据
tModel.setDataVector(Object[][] data, Object[] columnNames);

tModel.setDataVector.fireTableStructureChanged();或jt.updateUI();//更新显示
```

