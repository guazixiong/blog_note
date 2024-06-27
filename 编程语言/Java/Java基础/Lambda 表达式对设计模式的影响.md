# Lambda 表达式对设计模式的影响

## 命令式模式

在这个设计模式的实现过程中有如下⼏个⽐较᯿要的点；

1. 抽象命令类；声明执⾏命令的接⼝和⽅法
2. 具体的命令实现类；接⼝类的具体实现，可以是⼀组相似的⾏为逻辑
3. 实现者；也就是为命令做实现的具体实现类
4. 调⽤者；处理命令、实现的具体操作者，负责对外提供命令服务

### 工程结构

![1700311791051](Lambda%20%E8%A1%A8%E8%BE%BE%E5%BC%8F%E5%AF%B9%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F%E7%9A%84%E5%BD%B1%E5%93%8D.assets/1700311791051.webp)

**文本编辑器功能**

```java

/**
 * 文本工具编辑器(抽象命名类)
 */
public interface Editor {
  /**
   * 保存
   */
  public void save();

  /**
   * 打开
   */
  public void open();

  /**
   * 关闭
   */
  public void close();
}

/**
 * @description txt文本实现类(具体的命令实现类)
 */
public class TxtEditorImpl implements Editor {
  @Override
  public void save() {
    System.out.println("Txt文本保存");
  }

  @Override
  public void open() {
    System.out.println("Txt文本打开");
  }

  @Override
  public void close() {
    System.out.println("Txt文本关闭");
  }
}

/**
 * @description 文本编辑器实现类(具体的命令实现类)
 */
public class WordEditorImpl implements Editor {
  @Override
  public void save() {
    System.out.println("Word文件保存");
  }

  @Override
  public void open() {
    System.out.println("Word文件打开");
  }

  @Override
  public void close() {
    System.out.println("Word文件关闭");
  }
}
```

**用户行为**

```java
/**
 * @description 实现者公共接口
 */
public interface Action {
  public void perform();
}


/**
 * @description 保存行为(实现者)
 */
public class SaveActionImpl implements Action {
  private final Editor editor;
  public SaveActionImpl(Editor editor) {
    this.editor = editor;
  }

  @Override
  public void perform() {
    editor.save();
  }
}

/**
 * @description 打开行为(实现者)
 */
public class OpenActionImpl implements Action {
  private final Editor editor;
  public OpenActionImpl(Editor editor) {
    this.editor = editor;
  }

  @Override
  public void perform() {
    editor.open();
  }
}

/**
 * @description 关闭行为(实现者)
 */
public class CloseActionImpl implements Action {
  private final Editor editor;
  public CloseActionImpl(Editor editor) {
    this.editor = editor;
  }

  @Override
  public void perform() {
    editor.close();
  }
}
```

**调⽤者**

```java
/**
 * @description 调用者
 */
public class Macro {
  private final List<Action> actions;

  public Macro() {
    actions = new ArrayList<>();
  }

  public void record(Action action) {
    actions.add(action);
  }

  public void run() {
    actions.forEach(Action::perform);
  }
}
```

**测试类**

```java
public class Main {
  public static void main(String[] args) {
//    Editor editor = new TxtEditorImpl();
    Editor editor = new WordEditorImpl();
    Macro macro = new Macro();
    // 命令式模式
//    macro.record(new Save(editor));
//    macro.record(new Open(editor));
//    macro.record(new Close(editor));
//    macro.run();
    // lambda 改造命令式模式
    macro.record(editor::save);
    macro.record(editor::open);
    macro.record(editor::close);
    macro.run();
  }
}
```

