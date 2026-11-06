# Java Swing GUI Programming: From Zero to Advanced

A hands-on, code-heavy guide. Every section builds on the last. **Type the code yourself, run it, break it, fix it.** That is the fastest way to learn Swing.

---

## Table of Contents

**Part 0: Getting Started**
0. [Prerequisites & Setup](#0-prerequisites--setup)

**Part 1: The Basics**
1. [Your First Window](#1-your-first-window)
2. [The Event Dispatch Thread (EDT)](#2-the-event-dispatch-thread-edt)
3. [Core Components](#3-core-components)
4. [Containers & Panels](#4-containers--panels)
5. [Layout Managers](#5-layout-managers)
6. [Event Handling](#6-event-handling)
7. [Dialogs](#7-dialogs)
8. [Menus, Toolbars & Popups](#8-menus-toolbars--popups)

**Part 2: Intermediate**
9. [Lists, Combo Boxes & Models](#9-lists-combo-boxes--models)
10. [Tables (JTable)](#10-tables-jtable)
11. [Trees (JTree)](#11-trees-jtree)
12. [Text Components in Depth](#12-text-components-in-depth)
13. [Borders, Icons & Images](#13-borders-icons--images)
14. [Look and Feel](#14-look-and-feel)
15. [Actions & Keyboard Shortcuts (Key Bindings)](#15-actions--keyboard-shortcuts-key-bindings)
16. [Input Validation](#16-input-validation)

**Part 3: Advanced**
17. [Custom Painting](#17-custom-painting)
18. [Animation with Timers](#18-animation-with-timers)
19. [Concurrency: SwingWorker](#19-concurrency-swingworker)
20. [Architecture: MVC & Observer](#20-architecture-mvc--observer)
21. [Building Custom Components](#21-building-custom-components)
22. [Undo / Redo](#22-undo--redo)
23. [Drag & Drop and Clipboard](#23-drag--drop-and-clipboard)
24. [Advanced Layouts: GridBagLayout, GroupLayout, MigLayout](#24-advanced-layouts)
25. [Internal Frames & Layered Panes](#25-internal-frames--layered-panes)
26. [Internationalization & Preferences](#26-internationalization--preferences)
27. [High-DPI, Accessibility & Performance](#27-high-dpi-accessibility--performance)
28. [Testing Swing Apps](#28-testing-swing-apps)
29. [Packaging & Distribution](#29-packaging--distribution)

**Part 4: Projects & Reference**
30. [Capstone Projects](#30-capstone-projects)
31. [Common Pitfalls](#31-common-pitfalls)
32. [Cheat Sheet](#32-cheat-sheet)
33. [Where to Go Next](#33-where-to-go-next)

---

# Part 0: Getting Started

## 0. Prerequisites & Setup

**You should know:** Java basics (classes, interfaces, inheritance, generics, lambdas, inner/anonymous classes, collections).

**You need:**
- JDK 17 or newer (JDK 21 recommended). Check with `java -version`.
- An editor/IDE: IntelliJ IDEA, Eclipse, NetBeans, or VS Code with the Java extension.

Swing is part of the JDK (`javax.swing` and `java.awt`). **No extra dependencies are needed** for most of this guide.

### What is Swing?

- **AWT** (Abstract Window Toolkit): the original Java GUI toolkit; uses native OS widgets ("heavyweight").
- **Swing**: built on top of AWT; components are drawn by Java itself ("lightweight"), so they look and behave the same on every platform. Class names start with `J` (`JButton`, `JFrame`).
- **Swing vs JavaFX**: JavaFX is the newer toolkit, but Swing is stable, ubiquitous in enterprise/desktop tools (IntelliJ itself is Swing-based), and needs no extra libraries.

### Compile & run from the command line

```bash
javac HelloSwing.java
java HelloSwing
```

With Java 11+ you can also run single-file programs directly:

```bash
java HelloSwing.java
```

### Package map (what to import)

| Package | Contents |
|---|---|
| `javax.swing` | Components: `JFrame`, `JButton`, `JTable`, ... |
| `javax.swing.event` | Swing-specific events: `ChangeListener`, `ListSelectionListener`, ... |
| `javax.swing.table` | Table models, renderers, editors |
| `javax.swing.tree` | Tree models, nodes |
| `javax.swing.text` | Text document model, styled text |
| `javax.swing.border` | Borders |
| `java.awt` | Layout managers, `Color`, `Font`, `Graphics`, `Dimension` |
| `java.awt.event` | `ActionListener`, `MouseListener`, `KeyListener`, ... |

---

# Part 1: The Basics

## 1. Your First Window

```java
import javax.swing.*;

public class HelloSwing {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Hello Swing");   // window with a title
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE); // quit app on close
            frame.setSize(400, 300);                    // width x height in pixels
            frame.setLocationRelativeTo(null);          // center on screen
            frame.add(new JLabel("Hello, World!", SwingConstants.CENTER));
            frame.setVisible(true);                     // ALWAYS last
        });
    }
}
```

### Line by line

| Code | Meaning |
|---|---|
| `SwingUtilities.invokeLater(...)` | Runs the code on the Event Dispatch Thread (next section). |
| `new JFrame("title")` | A top-level window with a title bar and borders. |
| `setDefaultCloseOperation` | What happens when the user clicks X. Options below. |
| `setSize(w, h)` | Fixed size. (Better: `pack()`, see below.) |
| `setLocationRelativeTo(null)` | Centers the window. |
| `add(component)` | Adds to the frame's *content pane*. |
| `setVisible(true)` | Shows the window. Do this **after** adding components. |

**Close operations:**

| Constant | Behavior |
|---|---|
| `JFrame.EXIT_ON_CLOSE` | Calls `System.exit(0)`. |
| `JFrame.DISPOSE_ON_CLOSE` | Closes this window; app ends when the last window is gone. |
| `JFrame.HIDE_ON_CLOSE` | Hides the window (default). |
| `JFrame.DO_NOTHING_ON_CLOSE` | Ignore; handle it yourself with a `WindowListener`. |

### `pack()` vs `setSize()`

`pack()` sizes the window to fit the *preferred sizes* of its contents. It's usually the better choice because it adapts to fonts, OS, and DPI.

```java
frame.pack();
frame.setLocationRelativeTo(null);
frame.setVisible(true);
```

### Object-oriented style (recommended)

Subclass `JFrame` or (even better) keep a `JFrame` as a field and build panels separately.

```java
import javax.swing.*;
import java.awt.*;

public class MainWindow extends JFrame {

    public MainWindow() {
        super("My App");
        setDefaultCloseOperation(EXIT_ON_CLOSE);

        JLabel label = new JLabel("Welcome!", SwingConstants.CENTER);
        label.setFont(label.getFont().deriveFont(Font.BOLD, 24f));
        label.setBorder(BorderFactory.createEmptyBorder(40, 80, 40, 80));
        add(label);

        pack();
        setLocationRelativeTo(null);
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> new MainWindow().setVisible(true));
    }
}
```

### Exercise 1
1. Change the title and size. Try each close operation.
2. Add an icon to the window: `frame.setIconImage(new ImageIcon("icon.png").getImage());`
3. Make the window non-resizable with `setResizable(false)`.

---

## 2. The Event Dispatch Thread (EDT)

This is the **single most important rule** in Swing.

> **All creation of and changes to Swing components must happen on the Event Dispatch Thread.**

Swing is **not thread-safe**. The EDT is a special thread that processes user events (clicks, keystrokes) and repaints. If you touch a component from another thread, you get weird, intermittent bugs. If you do slow work *on* the EDT, the UI freezes.

### Starting a GUI correctly

```java
public static void main(String[] args) {
    SwingUtilities.invokeLater(() -> {
        // create and show GUI here
    });
}
```

(`EventQueue.invokeLater` is equivalent.)

### Related helpers

```java
SwingUtilities.invokeLater(Runnable r);      // run on EDT later, don't wait
SwingUtilities.invokeAndWait(Runnable r);    // run on EDT and block until done (throws checked exceptions)
SwingUtilities.isEventDispatchThread();      // am I on the EDT?
```

### Demo: freezing the UI (don't do this)

```java
JButton button = new JButton("Do slow work");
button.addActionListener(e -> {
    try { Thread.sleep(5000); } catch (InterruptedException ex) { }  // EDT blocked!
    System.out.println("Done");
});
// During those 5 seconds the window can't repaint or respond.
```

The fix is background threads or `SwingWorker`, covered in [Section 19](#19-concurrency-swingworker). For now, remember: **event handlers must be quick.**

---

## 3. Core Components

A *component* is a visual element. All Swing components descend from `JComponent` (which extends `java.awt.Container` -> `Component`).

### A tour in one window

```java
import javax.swing.*;
import java.awt.*;

public class ComponentTour {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Component Tour");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JPanel panel = new JPanel(new GridLayout(0, 2, 8, 8)); // 2 columns, any rows
            panel.setBorder(BorderFactory.createEmptyBorder(12, 12, 12, 12));

            // JLabel
            panel.add(new JLabel("JLabel:"));
            panel.add(new JLabel("Some text"));

            // JButton
            panel.add(new JLabel("JButton:"));
            panel.add(new JButton("Click me"));

            // JTextField
            panel.add(new JLabel("JTextField:"));
            panel.add(new JTextField("editable", 15));

            // JPasswordField
            panel.add(new JLabel("JPasswordField:"));
            panel.add(new JPasswordField("secret"));

            // JCheckBox
            panel.add(new JLabel("JCheckBox:"));
            panel.add(new JCheckBox("I agree", true));

            // JRadioButton (grouped so only one can be chosen)
            panel.add(new JLabel("JRadioButton:"));
            JPanel radios = new JPanel(new FlowLayout(FlowLayout.LEFT, 0, 0));
            JRadioButton small = new JRadioButton("S");
            JRadioButton large = new JRadioButton("L", true);
            ButtonGroup group = new ButtonGroup();
            group.add(small);
            group.add(large);
            radios.add(small);
            radios.add(large);
            panel.add(radios);

            // JComboBox
            panel.add(new JLabel("JComboBox:"));
            panel.add(new JComboBox<>(new String[]{"Red", "Green", "Blue"}));

            // JSpinner
            panel.add(new JLabel("JSpinner:"));
            panel.add(new JSpinner(new SpinnerNumberModel(5, 0, 10, 1)));

            // JSlider
            panel.add(new JLabel("JSlider:"));
            JSlider slider = new JSlider(0, 100, 40);
            slider.setMajorTickSpacing(25);
            slider.setPaintTicks(true);
            slider.setPaintLabels(true);
            panel.add(slider);

            // JProgressBar
            panel.add(new JLabel("JProgressBar:"));
            JProgressBar bar = new JProgressBar(0, 100);
            bar.setValue(65);
            bar.setStringPainted(true);
            panel.add(bar);

            // JToggleButton
            panel.add(new JLabel("JToggleButton:"));
            panel.add(new JToggleButton("Toggle"));

            frame.add(panel);
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

### Component reference

| Component | Purpose | Key methods |
|---|---|---|
| `JLabel` | Text and/or icon, non-interactive | `setText`, `setIcon`, `setHorizontalAlignment`, `setLabelFor` |
| `JButton` | Push button | `addActionListener`, `setEnabled`, `setMnemonic`, `setToolTipText` |
| `JToggleButton` | Button that stays pressed | `isSelected`, `setSelected` |
| `JCheckBox` | On/off | `isSelected`, `addItemListener` |
| `JRadioButton` | One-of-many (use `ButtonGroup`) | `isSelected`, `setActionCommand` |
| `JTextField` | Single-line input | `getText`, `setText`, `setColumns`, `addActionListener` (Enter key) |
| `JPasswordField` | Masked input | `getPassword()` returns `char[]` |
| `JTextArea` | Multi-line plain text | `append`, `setLineWrap`, `setWrapStyleWord` |
| `JComboBox<E>` | Dropdown | `getSelectedItem`, `setSelectedIndex`, `setEditable` |
| `JList<E>` | Selectable list | `getSelectedValue`, `setSelectionMode` |
| `JSpinner` | Number/date/list stepper | `getValue`, `setValue` |
| `JSlider` | Pick a value from a range | `getValue`, `addChangeListener` |
| `JProgressBar` | Progress display | `setValue`, `setIndeterminate` |

### Common methods inherited by all components

```java
component.setEnabled(false);              // grey out / disable input
component.setVisible(false);              // hide
component.setToolTipText("Helpful hint"); // hover text
component.setFont(new Font("SansSerif", Font.BOLD, 14));
component.setForeground(Color.WHITE);     // text colour
component.setBackground(Color.DARK_GRAY); // needs setOpaque(true) on labels
component.setPreferredSize(new Dimension(200, 40)); // use sparingly; see layouts
component.setCursor(Cursor.getPredefinedCursor(Cursor.HAND_CURSOR));
component.requestFocusInWindow();
```

### Mnemonics

A mnemonic lets users press `Alt+<key>` to activate a control.

```java
JButton save = new JButton("Save");
save.setMnemonic('S');   // Alt+S, and the S is underlined
```

### JLabel with HTML

Most Swing text components render simple HTML if the text begins with `<html>`:

```java
JLabel rich = new JLabel("<html><b>Bold</b> and <i>italic</i><br>second line</html>");
```

### JTextArea inside a scroll pane

`JTextArea` does **not** scroll by itself. Wrap it:

```java
JTextArea area = new JTextArea(10, 40);
area.setLineWrap(true);
area.setWrapStyleWord(true);
JScrollPane scroll = new JScrollPane(area);
frame.add(scroll);
```

### Exercise 3
Build a "profile form" with a name field, a gender radio group, a country combo box, a newsletter checkbox, and a "Bio" text area in a scroll pane. Don't worry about layout yet; use a `GridLayout`.

---

## 4. Containers & Panels

Components live inside **containers**. Containers can hold other containers, forming a tree (the *containment hierarchy*).

```
JFrame
 └─ contentPane (JPanel)
     ├─ JPanel (toolbar)
     │    ├─ JButton
     │    └─ JButton
     └─ JScrollPane
          └─ JTextArea
```

### Top-level containers

| Class | Use |
|---|---|
| `JFrame` | Main application window |
| `JDialog` | Secondary window (modal or not) |
| `JWindow` | Undecorated window (splash screens) |
| `JApplet` | Obsolete, ignore |

### JPanel: the workhorse

A plain container, default layout is `FlowLayout`.

```java
JPanel panel = new JPanel();                       // FlowLayout
JPanel panel2 = new JPanel(new BorderLayout());    // pick a layout
panel.add(new JButton("A"));
panel.add(new JButton("B"));
```

### JScrollPane

```java
JScrollPane sp = new JScrollPane(bigComponent);
sp.setVerticalScrollBarPolicy(ScrollPaneConstants.VERTICAL_SCROLLBAR_AS_NEEDED);
sp.setHorizontalScrollBarPolicy(ScrollPaneConstants.HORIZONTAL_SCROLLBAR_NEVER);
sp.getVerticalScrollBar().setUnitIncrement(16); // smoother mouse-wheel scrolling
```

### JSplitPane: resizable divider

```java
JSplitPane split = new JSplitPane(
        JSplitPane.HORIZONTAL_SPLIT,
        new JScrollPane(new JList<>(new String[]{"One", "Two", "Three"})),
        new JScrollPane(new JTextArea()));
split.setDividerLocation(150);
split.setOneTouchExpandable(true);
split.setResizeWeight(0.0);   // extra space goes to the right component
```

### JTabbedPane

```java
JTabbedPane tabs = new JTabbedPane();
tabs.addTab("General", new JPanel());
tabs.addTab("Advanced", new JPanel());
tabs.setTabPlacement(JTabbedPane.TOP);     // TOP, BOTTOM, LEFT, RIGHT
tabs.setSelectedIndex(1);
tabs.addChangeListener(e -> System.out.println("Tab: " + tabs.getSelectedIndex()));
```

### Content pane, root pane, glass pane

A `JFrame` actually contains a `JRootPane`:

```
JRootPane
 ├─ glassPane        (transparent overlay covering everything)
 ├─ layeredPane
 │    ├─ menuBar
 │    └─ contentPane (what frame.add() uses)
```

```java
frame.getContentPane().setBackground(Color.WHITE);
frame.setJMenuBar(menuBar);        // sets the menu bar
frame.getRootPane().setDefaultButton(okButton); // Enter triggers this button
```

### Exercise 4
Build a window with a `JSplitPane`: a list on the left and a `JTabbedPane` on the right with 3 tabs.


---

## 5. Layout Managers

A **layout manager** decides the size and position of components inside a container. You almost never position things by hand; layouts adapt to window resizing, font changes, and different OSes.

```java
panel.setLayout(new BorderLayout());
// or pass it to the constructor
JPanel p = new JPanel(new GridLayout(2, 3));
```

### 5.1 FlowLayout (default for JPanel)

Places components left-to-right, wrapping to a new row when out of space. Like text in a paragraph.

```java
JPanel p = new JPanel(new FlowLayout(FlowLayout.LEFT, 10, 5)); // align, hgap, vgap
for (int i = 1; i <= 8; i++) p.add(new JButton("Button " + i));
```

Good for: rows of buttons.

### 5.2 BorderLayout (default for JFrame content pane)

Five regions: `NORTH`, `SOUTH`, `EAST`, `WEST`, `CENTER`. **CENTER takes all the leftover space.**

```java
JFrame frame = new JFrame("BorderLayout");
frame.setLayout(new BorderLayout(5, 5));
frame.add(new JButton("North"),  BorderLayout.NORTH);
frame.add(new JButton("South"),  BorderLayout.SOUTH);
frame.add(new JButton("West"),   BorderLayout.WEST);
frame.add(new JButton("East"),   BorderLayout.EAST);
frame.add(new JTextArea("Center"), BorderLayout.CENTER);
```

Good for: the overall skeleton of a window (toolbar on top, status bar on bottom, content in the middle).

Modern alias names: `PAGE_START`, `PAGE_END`, `LINE_START`, `LINE_END`, `CENTER` (respect right-to-left languages).

### 5.3 GridLayout

Equal-sized cells in a grid.

```java
JPanel keypad = new JPanel(new GridLayout(4, 3, 4, 4)); // rows, cols, hgap, vgap
String[] keys = {"1","2","3","4","5","6","7","8","9","*","0","#"};
for (String k : keys) keypad.add(new JButton(k));
```

Pass `0` for rows or cols to say "as many as needed".

### 5.4 BoxLayout

Stacks components in a single row or column, respecting their preferred/maximum sizes. Use `Box` helpers for spacing.

```java
JPanel column = new JPanel();
column.setLayout(new BoxLayout(column, BoxLayout.Y_AXIS));
column.add(new JButton("One"));
column.add(Box.createVerticalStrut(10));  // fixed 10px gap
column.add(new JButton("Two"));
column.add(Box.createVerticalGlue());     // flexible: pushes things apart
column.add(new JButton("Three"));
```

Alignment gotcha: components in a `BoxLayout` panel should share the same `alignmentX`:

```java
button.setAlignmentX(Component.LEFT_ALIGNMENT);
```

Also, text fields stretch to fill width. Cap it with:

```java
field.setMaximumSize(new Dimension(Integer.MAX_VALUE, field.getPreferredSize().height));
```

Shortcut: `Box.createVerticalBox()` and `Box.createHorizontalBox()` give you pre-configured boxes.

### 5.5 CardLayout

Shows one "card" (child) at a time. Great for wizards and screen switching.

```java
import javax.swing.*;
import java.awt.*;

public class CardDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("CardLayout");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            CardLayout cards = new CardLayout();
            JPanel deck = new JPanel(cards);
            deck.add(new JLabel("Page 1", SwingConstants.CENTER), "page1");
            deck.add(new JLabel("Page 2", SwingConstants.CENTER), "page2");
            deck.add(new JLabel("Page 3", SwingConstants.CENTER), "page3");

            JButton prev = new JButton("Previous");
            JButton next = new JButton("Next");
            prev.addActionListener(e -> cards.previous(deck));
            next.addActionListener(e -> cards.next(deck));

            JPanel nav = new JPanel();
            nav.add(prev);
            nav.add(next);

            frame.add(deck, BorderLayout.CENTER);
            frame.add(nav, BorderLayout.SOUTH);
            frame.setSize(300, 200);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

Jump directly: `cards.show(deck, "page2");`

### 5.6 Nesting layouts (the real technique)

Real UIs combine layouts by nesting panels. Example: a login form.

```java
import javax.swing.*;
import java.awt.*;

public class LoginForm extends JPanel {
    public LoginForm() {
        super(new BorderLayout(10, 10));
        setBorder(BorderFactory.createEmptyBorder(16, 16, 16, 16));

        JLabel title = new JLabel("Sign In", SwingConstants.CENTER);
        title.setFont(title.getFont().deriveFont(Font.BOLD, 20f));

        JPanel fields = new JPanel(new GridLayout(2, 2, 8, 8));
        fields.add(new JLabel("Username:"));
        fields.add(new JTextField(15));
        fields.add(new JLabel("Password:"));
        fields.add(new JPasswordField(15));

        JPanel buttons = new JPanel(new FlowLayout(FlowLayout.RIGHT));
        buttons.add(new JButton("Cancel"));
        buttons.add(new JButton("Login"));

        add(title, BorderLayout.NORTH);
        add(fields, BorderLayout.CENTER);
        add(buttons, BorderLayout.SOUTH);
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame f = new JFrame("Login");
            f.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            f.setContentPane(new LoginForm());
            f.pack();
            f.setLocationRelativeTo(null);
            f.setVisible(true);
        });
    }
}
```

### 5.7 The `null` layout (avoid)

`setLayout(null)` + `setBounds(x, y, w, h)` gives absolute positioning, but it breaks with resizing, different fonts, and different DPI. Use it only for quick experiments or custom drawing surfaces.

### Layout cheat summary

| Layout | Best for |
|---|---|
| `FlowLayout` | Button rows |
| `BorderLayout` | Main window skeleton |
| `GridLayout` | Uniform grids (keypads, calculators) |
| `BoxLayout` | Simple vertical/horizontal stacks |
| `CardLayout` | Switching between screens |
| `GridBagLayout` | Complex forms (see [Section 24](#24-advanced-layouts)) |
| `GroupLayout` | GUI builders (NetBeans) |

### How sizes work

Each component reports `getMinimumSize()`, `getPreferredSize()`, `getMaximumSize()`. Layouts consult these. Prefer overriding `getPreferredSize()` in custom components over calling `setPreferredSize()` from outside.

### Exercise 5
Recreate a simple calculator layout: a display text field (NORTH), a 4x4 `GridLayout` of buttons (CENTER).

---

## 6. Event Handling

Swing uses the **observer pattern**: components *fire* events; you register *listeners*.

```
User clicks button -> JButton creates ActionEvent -> calls actionPerformed() on every registered ActionListener
```

### 6.1 ActionListener (buttons, menu items, Enter in text fields)

Three equivalent styles:

```java
// 1. Anonymous inner class
button.addActionListener(new ActionListener() {
    @Override public void actionPerformed(ActionEvent e) {
        System.out.println("Clicked!");
    }
});

// 2. Lambda (ActionListener has one method)
button.addActionListener(e -> System.out.println("Clicked!"));

// 3. Method reference
button.addActionListener(this::onSave);
// ...
private void onSave(ActionEvent e) { /* ... */ }
```

Full example: a click counter.

```java
import javax.swing.*;
import java.awt.*;

public class ClickCounter {
    private int count = 0;

    public static void main(String[] args) {
        SwingUtilities.invokeLater(new ClickCounter()::build);
    }

    private void build() {
        JFrame frame = new JFrame("Counter");
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

        JLabel label = new JLabel("Clicks: 0", SwingConstants.CENTER);
        JButton button = new JButton("Click me");
        button.addActionListener(e -> label.setText("Clicks: " + (++count)));

        frame.add(label, BorderLayout.CENTER);
        frame.add(button, BorderLayout.SOUTH);
        frame.setSize(250, 150);
        frame.setLocationRelativeTo(null);
        frame.setVisible(true);
    }
}
```

### 6.2 One listener for many components

Use `getSource()` or `getActionCommand()`:

```java
ActionListener shared = e -> {
    String cmd = e.getActionCommand();      // button text by default
    switch (cmd) {
        case "Red"   -> panel.setBackground(Color.RED);
        case "Green" -> panel.setBackground(Color.GREEN);
        case "Blue"  -> panel.setBackground(Color.BLUE);
    }
};
for (String name : new String[]{"Red", "Green", "Blue"}) {
    JButton b = new JButton(name);
    b.addActionListener(shared);
    toolbar.add(b);
}
```

### 6.3 Text field: Enter key

```java
JTextField input = new JTextField(20);
input.addActionListener(e -> {                  // fires when user presses Enter
    System.out.println("You typed: " + input.getText());
    input.setText("");
});
```

### 6.4 Mouse events

```java
panel.addMouseListener(new MouseAdapter() {      // Adapter = empty implementations
    @Override public void mouseClicked(MouseEvent e) {
        System.out.println("Clicked at " + e.getX() + "," + e.getY()
                + " count=" + e.getClickCount());
        if (SwingUtilities.isRightMouseButton(e)) System.out.println("Right click");
    }
    @Override public void mousePressed(MouseEvent e)  { }
    @Override public void mouseReleased(MouseEvent e) { }
    @Override public void mouseEntered(MouseEvent e)  { panel.setBackground(Color.YELLOW); }
    @Override public void mouseExited(MouseEvent e)   { panel.setBackground(Color.WHITE); }
});

panel.addMouseMotionListener(new MouseMotionAdapter() {
    @Override public void mouseMoved(MouseEvent e)   { /* hovering */ }
    @Override public void mouseDragged(MouseEvent e) { /* button held while moving */ }
});

panel.addMouseWheelListener(e -> System.out.println("Wheel: " + e.getWheelRotation()));
```

Tip: `MouseAdapter` implements all three mouse listener interfaces, so one subclass can be registered with `addMouseListener`, `addMouseMotionListener`, and `addMouseWheelListener`.

### 6.5 Keyboard events

```java
textField.addKeyListener(new KeyAdapter() {
    @Override public void keyPressed(KeyEvent e) {
        if (e.getKeyCode() == KeyEvent.VK_ESCAPE) textField.setText("");
    }
    @Override public void keyTyped(KeyEvent e) {
        char c = e.getKeyChar();
        if (!Character.isDigit(c)) e.consume(); // reject non-digits
    }
});
```

For application shortcuts, **prefer Key Bindings** ([Section 15](#15-actions--keyboard-shortcuts-key-bindings)) over `KeyListener`; they're more reliable and focus-aware.

### 6.6 Window events

```java
frame.setDefaultCloseOperation(JFrame.DO_NOTHING_ON_CLOSE);
frame.addWindowListener(new WindowAdapter() {
    @Override public void windowClosing(WindowEvent e) {
        int choice = JOptionPane.showConfirmDialog(frame,
                "Really quit?", "Confirm", JOptionPane.YES_NO_OPTION);
        if (choice == JOptionPane.YES_OPTION) frame.dispose();
    }
});
```

### 6.7 Change, item, selection, and document events

| Listener | Fires from | Example |
|---|---|---|
| `ChangeListener` | `JSlider`, `JSpinner`, `JTabbedPane` | `slider.addChangeListener(e -> ...)` |
| `ItemListener` | `JCheckBox`, `JComboBox`, `JRadioButton` | `check.addItemListener(e -> ...)` |
| `ListSelectionListener` | `JList`, `JTable` selection | `list.addListSelectionListener(e -> ...)` |
| `DocumentListener` | Text content changes | see below |
| `PropertyChangeListener` | Any bound property | `comp.addPropertyChangeListener(...)` |
| `ComponentListener` | Resize/move/show/hide | `comp.addComponentListener(...)` |
| `FocusListener` | Focus gained/lost | `field.addFocusListener(...)` |

```java
// Slider -> label
JSlider slider = new JSlider(0, 100, 50);
JLabel value = new JLabel("50");
slider.addChangeListener(e -> {
    if (!slider.getValueIsAdjusting()) {      // only when the user lets go
        value.setText(String.valueOf(slider.getValue()));
    }
});

// Checkbox
check.addItemListener(e -> {
    boolean on = e.getStateChange() == ItemEvent.SELECTED;
    System.out.println("Checked: " + on);
});

// Combo box
combo.addActionListener(e -> System.out.println(combo.getSelectedItem()));

// Live text changes
import javax.swing.event.*;
field.getDocument().addDocumentListener(new DocumentListener() {
    @Override public void insertUpdate(DocumentEvent e)  { changed(); }
    @Override public void removeUpdate(DocumentEvent e)  { changed(); }
    @Override public void changedUpdate(DocumentEvent e) { changed(); } // style changes
    private void changed() { System.out.println("Now: " + field.getText()); }
});
```

> Never call `setText` on a field *inside* its own DocumentListener; it throws `IllegalStateException` (mutation not allowed during notification). Use `SwingUtilities.invokeLater` if you must.

### Exercise 6
Build a live text-length counter: a `JTextArea` and a label that shows the number of characters and words, updated through a `DocumentListener`.

---

## 7. Dialogs

### 7.1 JOptionPane (ready-made dialogs)

```java
// Message
JOptionPane.showMessageDialog(frame, "File saved.", "Info", JOptionPane.INFORMATION_MESSAGE);
// Other types: ERROR_MESSAGE, WARNING_MESSAGE, QUESTION_MESSAGE, PLAIN_MESSAGE

// Confirm
int result = JOptionPane.showConfirmDialog(frame, "Delete this file?", "Confirm",
        JOptionPane.YES_NO_CANCEL_OPTION, JOptionPane.WARNING_MESSAGE);
if (result == JOptionPane.YES_OPTION) { /* delete */ }

// Input (returns null if cancelled)
String name = JOptionPane.showInputDialog(frame, "Your name?");
if (name != null && !name.isBlank()) System.out.println("Hi " + name);

// Choose from options
String[] choices = {"Small", "Medium", "Large"};
Object pick = JOptionPane.showInputDialog(frame, "Size?", "Pick",
        JOptionPane.QUESTION_MESSAGE, null, choices, choices[1]);

// Custom buttons
String[] opts = {"Save", "Don't Save", "Cancel"};
int n = JOptionPane.showOptionDialog(frame, "Save changes?", "Unsaved",
        JOptionPane.DEFAULT_OPTION, JOptionPane.WARNING_MESSAGE, null, opts, opts[0]);
// n is the index of the chosen button, or CLOSED_OPTION (-1)
```

Pass the parent frame as the first argument so the dialog centers over it and is modal to it. (`null` centers on screen.)

### 7.2 JFileChooser

```java
JFileChooser chooser = new JFileChooser();
chooser.setCurrentDirectory(new java.io.File(System.getProperty("user.home")));
chooser.setFileFilter(new javax.swing.filechooser.FileNameExtensionFilter(
        "Text files (*.txt)", "txt"));

// Open
if (chooser.showOpenDialog(frame) == JFileChooser.APPROVE_OPTION) {
    java.io.File file = chooser.getSelectedFile();
    String text = java.nio.file.Files.readString(file.toPath());  // wrap in try/catch IOException
    textArea.setText(text);
}

// Save
if (chooser.showSaveDialog(frame) == JFileChooser.APPROVE_OPTION) {
    java.io.File file = chooser.getSelectedFile();
    java.nio.file.Files.writeString(file.toPath(), textArea.getText());
}

// Pick directories
chooser.setFileSelectionMode(JFileChooser.DIRECTORIES_ONLY);
// Multiple files
chooser.setMultiSelectionEnabled(true); // then getSelectedFiles()
```

### 7.3 JColorChooser

```java
Color c = JColorChooser.showDialog(frame, "Pick a colour", panel.getBackground());
if (c != null) panel.setBackground(c);   // null if cancelled
```

### 7.4 Custom dialogs with JDialog

```java
import javax.swing.*;
import java.awt.*;

public class SettingsDialog extends JDialog {
    private final JTextField nameField = new JTextField(15);
    private boolean confirmed = false;

    public SettingsDialog(Frame owner) {
        super(owner, "Settings", true);        // true = modal (blocks the owner)

        JPanel form = new JPanel(new FlowLayout());
        form.add(new JLabel("Name:"));
        form.add(nameField);

        JButton ok = new JButton("OK");
        JButton cancel = new JButton("Cancel");
        ok.addActionListener(e -> { confirmed = true; dispose(); });
        cancel.addActionListener(e -> dispose());

        JPanel buttons = new JPanel(new FlowLayout(FlowLayout.RIGHT));
        buttons.add(ok);
        buttons.add(cancel);

        add(form, BorderLayout.CENTER);
        add(buttons, BorderLayout.SOUTH);
        getRootPane().setDefaultButton(ok);    // Enter = OK

        // Escape closes the dialog
        getRootPane().registerKeyboardAction(e -> dispose(),
                KeyStroke.getKeyStroke("ESCAPE"), JComponent.WHEN_IN_FOCUSED_WINDOW);

        pack();
        setLocationRelativeTo(owner);
    }

    /** Shows the dialog and returns the entered name, or null if cancelled. */
    public String showDialog() {
        setVisible(true);                       // blocks here until disposed (modal)
        return confirmed ? nameField.getText() : null;
    }
}

// Usage:
String result = new SettingsDialog(frame).showDialog();
```

Modality types: `Dialog.ModalityType.APPLICATION_MODAL` blocks all windows of the app; `DOCUMENT_MODAL` blocks only the owner's document tree; `MODELESS` blocks nothing.

### 7.5 JOptionPane with a custom component

```java
JTextField user = new JTextField();
JPasswordField pass = new JPasswordField();
Object[] message = {"Username:", user, "Password:", pass};
int ok = JOptionPane.showConfirmDialog(frame, message, "Login", JOptionPane.OK_CANCEL_OPTION);
if (ok == JOptionPane.OK_OPTION) {
    System.out.println(user.getText() + " / " + new String(pass.getPassword()));
}
```

---

## 8. Menus, Toolbars & Popups

### 8.1 Menu bar

```java
JMenuBar menuBar = new JMenuBar();

JMenu fileMenu = new JMenu("File");
fileMenu.setMnemonic('F');

JMenuItem newItem = new JMenuItem("New");
newItem.setAccelerator(KeyStroke.getKeyStroke("control N"));
newItem.addActionListener(e -> newDocument());

JMenuItem openItem = new JMenuItem("Open...");
openItem.setAccelerator(KeyStroke.getKeyStroke("control O"));

JMenuItem exitItem = new JMenuItem("Exit");
exitItem.addActionListener(e -> System.exit(0));

fileMenu.add(newItem);
fileMenu.add(openItem);
fileMenu.addSeparator();
fileMenu.add(exitItem);

JMenu viewMenu = new JMenu("View");
JCheckBoxMenuItem statusBar = new JCheckBoxMenuItem("Status bar", true);
JRadioButtonMenuItem small = new JRadioButtonMenuItem("Small");
JRadioButtonMenuItem big = new JRadioButtonMenuItem("Big", true);
ButtonGroup sizes = new ButtonGroup();
sizes.add(small);
sizes.add(big);
viewMenu.add(statusBar);
viewMenu.addSeparator();
viewMenu.add(small);
viewMenu.add(big);

JMenu zoom = new JMenu("Zoom");           // submenu
zoom.add(new JMenuItem("In"));
zoom.add(new JMenuItem("Out"));
viewMenu.add(zoom);

menuBar.add(fileMenu);
menuBar.add(viewMenu);
frame.setJMenuBar(menuBar);
```

Cross-platform accelerators: use `Toolkit.getDefaultToolkit().getMenuShortcutKeyMaskEx()` instead of hard-coding Ctrl so macOS uses Cmd:

```java
int mask = Toolkit.getDefaultToolkit().getMenuShortcutKeyMaskEx();
item.setAccelerator(KeyStroke.getKeyStroke(KeyEvent.VK_S, mask));
```

### 8.2 Popup (context) menus

```java
JPopupMenu popup = new JPopupMenu();
popup.add(new JMenuItem("Cut"));
popup.add(new JMenuItem("Copy"));
popup.add(new JMenuItem("Paste"));

textArea.addMouseListener(new MouseAdapter() {
    @Override public void mousePressed(MouseEvent e)  { maybeShow(e); }
    @Override public void mouseReleased(MouseEvent e) { maybeShow(e); }
    private void maybeShow(MouseEvent e) {
        if (e.isPopupTrigger()) popup.show(e.getComponent(), e.getX(), e.getY());
    }
});
// Shortcut: textArea.setComponentPopupMenu(popup);
```

(Check `isPopupTrigger()` in *both* pressed and released; platforms differ.)

### 8.3 Toolbars

```java
JToolBar toolbar = new JToolBar("Tools");
toolbar.setFloatable(false);                       // lock in place
JButton b = new JButton(new ImageIcon("save.png")); // icon-only button
b.setToolTipText("Save");
toolbar.add(b);
toolbar.addSeparator();
toolbar.add(new JButton("Cut"));
frame.add(toolbar, BorderLayout.NORTH);
```

### 8.4 Status bar (a pattern, not a class)

```java
JLabel status = new JLabel(" Ready");
status.setBorder(BorderFactory.createEtchedBorder());
frame.add(status, BorderLayout.SOUTH);
```

### Mini project: tiny Notepad

```java
import javax.swing.*;
import java.awt.*;
import java.awt.event.KeyEvent;
import java.io.IOException;
import java.nio.file.*;

public class MiniNotepad extends JFrame {
    private final JTextArea area = new JTextArea();
    private Path current;

    public MiniNotepad() {
        super("Untitled - Mini Notepad");
        setDefaultCloseOperation(EXIT_ON_CLOSE);
        area.setFont(new Font(Font.MONOSPACED, Font.PLAIN, 14));
        add(new JScrollPane(area));
        setJMenuBar(buildMenu());
        setSize(600, 400);
        setLocationRelativeTo(null);
    }

    private JMenuBar buildMenu() {
        int mask = Toolkit.getDefaultToolkit().getMenuShortcutKeyMaskEx();
        JMenuItem open = new JMenuItem("Open...");
        open.setAccelerator(KeyStroke.getKeyStroke(KeyEvent.VK_O, mask));
        open.addActionListener(e -> open());

        JMenuItem save = new JMenuItem("Save");
        save.setAccelerator(KeyStroke.getKeyStroke(KeyEvent.VK_S, mask));
        save.addActionListener(e -> save());

        JMenuItem exit = new JMenuItem("Exit");
        exit.addActionListener(e -> dispose());

        JMenu file = new JMenu("File");
        file.add(open); file.add(save); file.addSeparator(); file.add(exit);
        JMenuBar bar = new JMenuBar();
        bar.add(file);
        return bar;
    }

    private void open() {
        JFileChooser fc = new JFileChooser();
        if (fc.showOpenDialog(this) != JFileChooser.APPROVE_OPTION) return;
        try {
            current = fc.getSelectedFile().toPath();
            area.setText(Files.readString(current));
            setTitle(current.getFileName() + " - Mini Notepad");
        } catch (IOException ex) {
            JOptionPane.showMessageDialog(this, ex.getMessage(), "Error", JOptionPane.ERROR_MESSAGE);
        }
    }

    private void save() {
        if (current == null) {
            JFileChooser fc = new JFileChooser();
            if (fc.showSaveDialog(this) != JFileChooser.APPROVE_OPTION) return;
            current = fc.getSelectedFile().toPath();
        }
        try {
            Files.writeString(current, area.getText());
            setTitle(current.getFileName() + " - Mini Notepad");
        } catch (IOException ex) {
            JOptionPane.showMessageDialog(this, ex.getMessage(), "Error", JOptionPane.ERROR_MESSAGE);
        }
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> new MiniNotepad().setVisible(true));
    }
}
```


---

# Part 2: Intermediate

## 9. Lists, Combo Boxes & Models

Swing's complex components follow **MVC-style separation**: the component (view + controller) shows data from a **model**. Understanding models is the key to intermediate Swing.

### 9.1 JList with DefaultListModel

```java
DefaultListModel<String> model = new DefaultListModel<>();
model.addElement("Apple");
model.addElement("Banana");
model.addElement("Cherry");

JList<String> list = new JList<>(model);
list.setSelectionMode(ListSelectionModel.SINGLE_SELECTION);
// others: SINGLE_INTERVAL_SELECTION, MULTIPLE_INTERVAL_SELECTION
list.setVisibleRowCount(6);

list.addListSelectionListener(e -> {
    if (!e.getValueIsAdjusting()) {            // ignore intermediate events
        System.out.println("Selected: " + list.getSelectedValue());
    }
});

// Double-click
list.addMouseListener(new MouseAdapter() {
    @Override public void mouseClicked(MouseEvent e) {
        if (e.getClickCount() == 2) {
            int index = list.locationToIndex(e.getPoint());
            System.out.println("Double-clicked " + model.get(index));
        }
    }
});

// Modify through the MODEL, not the list; the list updates automatically
model.addElement("Date");
model.remove(0);
model.set(1, "Blueberry");

frame.add(new JScrollPane(list));  // always put lists in a scroll pane
```

A **filter box** example: a text field that filters a list as you type.

```java
import javax.swing.*;
import javax.swing.event.*;
import java.awt.*;
import java.util.List;

public class FilterList extends JPanel {
    private final List<String> all = List.of("Alice", "Albert", "Bob", "Bella", "Carol", "Charlie");
    private final DefaultListModel<String> model = new DefaultListModel<>();

    public FilterList() {
        super(new BorderLayout(5, 5));
        JTextField search = new JTextField();
        JList<String> list = new JList<>(model);
        refill("");

        search.getDocument().addDocumentListener(new DocumentListener() {
            public void insertUpdate(DocumentEvent e)  { refill(search.getText()); }
            public void removeUpdate(DocumentEvent e)  { refill(search.getText()); }
            public void changedUpdate(DocumentEvent e) { }
        });

        add(search, BorderLayout.NORTH);
        add(new JScrollPane(list), BorderLayout.CENTER);
    }

    private void refill(String filter) {
        model.clear();
        all.stream()
           .filter(s -> s.toLowerCase().contains(filter.toLowerCase()))
           .forEach(model::addElement);
    }
}
```

### 9.2 Custom cell renderers

A **renderer** is a "stamp" that draws each cell. It is reused for every cell, so don't store state in it.

```java
class PersonRenderer extends DefaultListCellRenderer {
    @Override
    public Component getListCellRendererComponent(JList<?> list, Object value, int index,
                                                  boolean isSelected, boolean cellHasFocus) {
        JLabel label = (JLabel) super.getListCellRendererComponent(
                list, value, index, isSelected, cellHasFocus);
        if (value instanceof Person p) {
            label.setText(p.name() + "  (" + p.age() + ")");
            label.setIcon(p.age() >= 18 ? adultIcon : childIcon);
        }
        label.setBorder(BorderFactory.createEmptyBorder(4, 8, 4, 8));
        return label;
    }
}

record Person(String name, int age) { }

JList<Person> people = new JList<>(personModel);
people.setCellRenderer(new PersonRenderer());
```

### 9.3 JComboBox

```java
JComboBox<String> combo = new JComboBox<>(new String[]{"Java", "Kotlin", "Scala"});
combo.setSelectedItem("Kotlin");
combo.setEditable(true);                          // user can type their own
String chosen = (String) combo.getSelectedItem();

combo.addActionListener(e -> System.out.println("Chose " + combo.getSelectedItem()));

// Dynamic model
DefaultComboBoxModel<String> cm = new DefaultComboBoxModel<>();
cm.addElement("One");
cm.addElement("Two");
combo.setModel(cm);
```

Combo boxes can also use custom renderers (`setRenderer`) with the same pattern as lists.

### 9.4 Writing your own ListModel

```java
class RangeModel extends AbstractListModel<Integer> {
    private final int size;
    RangeModel(int size) { this.size = size; }
    @Override public int getSize() { return size; }
    @Override public Integer getElementAt(int index) { return index * index; } // computed lazily!
}
JList<Integer> huge = new JList<>(new RangeModel(1_000_000)); // no problem: nothing is stored
huge.setPrototypeCellValue(999999999);   // speeds up layout for huge lists
```

When your data changes, call `fireContentsChanged`, `fireIntervalAdded`, or `fireIntervalRemoved` so the view updates.

---

## 10. Tables (JTable)

`JTable` is one of Swing's most powerful components. It has three collaborators: **TableModel** (data), **TableCellRenderer** (display), **TableCellEditor** (editing).

### 10.1 The quickest table (don't stay here long)

```java
String[] columns = {"Name", "Age", "Active"};
Object[][] data = {
    {"Alice", 30, true},
    {"Bob",   25, false},
    {"Carol", 41, true},
};
JTable table = new JTable(data, columns);
frame.add(new JScrollPane(table));   // scroll pane is REQUIRED for column headers to show
```

Problem: every cell is editable and everything is treated as a `String`. Use a real model.

### 10.2 DefaultTableModel

```java
DefaultTableModel model = new DefaultTableModel(new Object[]{"Name", "Age"}, 0) {
    @Override public boolean isCellEditable(int row, int col) { return col == 1; }
    @Override public Class<?> getColumnClass(int col) {
        return col == 1 ? Integer.class : String.class;   // enables right-aligned numbers, proper sorting
    }
};
model.addRow(new Object[]{"Alice", 30});
model.addRow(new Object[]{"Bob", 25});
model.removeRow(0);
model.setValueAt(31, 0, 1);

JTable table = new JTable(model);
```

### 10.3 Custom AbstractTableModel (the professional way)

Back the table with your own domain objects.

```java
import javax.swing.table.AbstractTableModel;
import java.util.*;

record Employee(String name, String dept, double salary, boolean active) { }

class EmployeeTableModel extends AbstractTableModel {
    private final String[] cols = {"Name", "Department", "Salary", "Active"};
    private final List<Employee> rows = new ArrayList<>();

    @Override public int getRowCount()    { return rows.size(); }
    @Override public int getColumnCount() { return cols.length; }
    @Override public String getColumnName(int c) { return cols[c]; }

    @Override public Class<?> getColumnClass(int c) {
        return switch (c) {
            case 2 -> Double.class;
            case 3 -> Boolean.class;       // rendered as a checkbox automatically
            default -> String.class;
        };
    }

    @Override public Object getValueAt(int r, int c) {
        Employee e = rows.get(r);
        return switch (c) {
            case 0 -> e.name();
            case 1 -> e.dept();
            case 2 -> e.salary();
            case 3 -> e.active();
            default -> null;
        };
    }

    @Override public boolean isCellEditable(int r, int c) { return c == 2 || c == 3; }

    @Override public void setValueAt(Object v, int r, int c) {
        Employee e = rows.get(r);
        rows.set(r, switch (c) {
            case 2 -> new Employee(e.name(), e.dept(), (Double) v, e.active());
            case 3 -> new Employee(e.name(), e.dept(), e.salary(), (Boolean) v);
            default -> e;
        });
        fireTableCellUpdated(r, c);        // ALWAYS tell the table something changed
    }

    // Domain-level operations
    public void add(Employee e) {
        rows.add(e);
        fireTableRowsInserted(rows.size() - 1, rows.size() - 1);
    }
    public void remove(int row) {
        rows.remove(row);
        fireTableRowsDeleted(row, row);
    }
    public Employee get(int row) { return rows.get(row); }
}
```

### 10.4 Table configuration

```java
JTable table = new JTable(new EmployeeTableModel());

table.setAutoCreateRowSorter(true);                          // click headers to sort
table.setSelectionMode(ListSelectionModel.SINGLE_SELECTION);
table.setRowHeight(24);
table.setShowGrid(true);
table.setFillsViewportHeight(true);                          // fill empty area under rows
table.setAutoResizeMode(JTable.AUTO_RESIZE_ALL_COLUMNS);
table.getColumnModel().getColumn(0).setPreferredWidth(200);
table.getTableHeader().setReorderingAllowed(false);

// Selection
table.getSelectionModel().addListSelectionListener(e -> {
    if (e.getValueIsAdjusting()) return;
    int viewRow = table.getSelectedRow();
    if (viewRow < 0) return;
    int modelRow = table.convertRowIndexToModel(viewRow);    // IMPORTANT with sorting/filtering!
    System.out.println(((EmployeeTableModel) table.getModel()).get(modelRow));
});
```

> **Classic bug:** when sorting or filtering is enabled, view row indexes != model row indexes. Always convert with `convertRowIndexToModel`.

### 10.5 Sorting and filtering

```java
TableRowSorter<EmployeeTableModel> sorter = new TableRowSorter<>(model);
table.setRowSorter(sorter);

// Filter by text, case-insensitive, across all columns
searchField.getDocument().addDocumentListener(new SimpleDocumentListener(() -> {
    String text = searchField.getText();
    sorter.setRowFilter(text.isBlank() ? null
            : RowFilter.regexFilter("(?i)" + java.util.regex.Pattern.quote(text)));
}));

// Number filter: salary > 50000
sorter.setRowFilter(RowFilter.numberFilter(RowFilter.ComparisonType.AFTER, 50000, 2));

// Custom comparator for column 1
sorter.setComparator(1, String.CASE_INSENSITIVE_ORDER);
```

Helper to avoid boilerplate:

```java
class SimpleDocumentListener implements javax.swing.event.DocumentListener {
    private final Runnable r;
    SimpleDocumentListener(Runnable r) { this.r = r; }
    public void insertUpdate(javax.swing.event.DocumentEvent e)  { r.run(); }
    public void removeUpdate(javax.swing.event.DocumentEvent e)  { r.run(); }
    public void changedUpdate(javax.swing.event.DocumentEvent e) { r.run(); }
}
```

### 10.6 Custom renderer

```java
class SalaryRenderer extends DefaultTableCellRenderer {
    private final java.text.NumberFormat fmt = java.text.NumberFormat.getCurrencyInstance();

    @Override
    public Component getTableCellRendererComponent(JTable t, Object value, boolean selected,
                                                   boolean focus, int row, int col) {
        super.getTableCellRendererComponent(t, value, selected, focus, row, col);
        setText(fmt.format((Double) value));
        setHorizontalAlignment(SwingConstants.RIGHT);
        if (!selected) {
            setForeground((Double) value > 100_000 ? new Color(0, 130, 0) : Color.BLACK);
        }
        return this;
    }
}
table.getColumnModel().getColumn(2).setDefaultRenderer(Double.class, new SalaryRenderer());
// or: table.setDefaultRenderer(Double.class, new SalaryRenderer());
```

Striped rows (override `prepareRenderer`):

```java
JTable striped = new JTable(model) {
    @Override public Component prepareRenderer(TableCellRenderer r, int row, int col) {
        Component c = super.prepareRenderer(r, row, col);
        if (!isRowSelected(row)) c.setBackground(row % 2 == 0 ? Color.WHITE : new Color(245, 245, 250));
        return c;
    }
};
```

### 10.7 Custom editor: combo box in a cell

```java
JComboBox<String> deptCombo = new JComboBox<>(new String[]{"Engineering", "Sales", "HR"});
table.getColumnModel().getColumn(1).setCellEditor(new DefaultCellEditor(deptCombo));
```

### 10.8 Button in a table cell (renderer + editor)

```java
class ButtonColumn extends AbstractCellEditor implements TableCellRenderer, TableCellEditor {
    private final JButton renderButton = new JButton();
    private final JButton editButton = new JButton();
    private int editingRow;

    ButtonColumn(String text, java.util.function.IntConsumer onClick) {
        renderButton.setText(text);
        editButton.setText(text);
        editButton.addActionListener(e -> {
            fireEditingStopped();                   // end editing first
            onClick.accept(editingRow);
        });
    }
    @Override public Component getTableCellRendererComponent(JTable t, Object v, boolean s,
                                                             boolean f, int r, int c) {
        return renderButton;
    }
    @Override public Component getTableCellEditorComponent(JTable t, Object v, boolean s, int r, int c) {
        editingRow = t.convertRowIndexToModel(r);
        return editButton;
    }
    @Override public Object getCellEditorValue() { return null; }
}

table.getColumnModel().getColumn(4)
     .setCellRenderer(new ButtonColumn("Delete", row -> model.remove(row)));
table.getColumnModel().getColumn(4)
     .setCellEditor(new ButtonColumn("Delete", row -> model.remove(row)));
```

### 10.9 Complete example: Employee manager

```java
import javax.swing.*;
import javax.swing.table.*;
import java.awt.*;

public class EmployeeApp extends JFrame {
    private final EmployeeTableModel model = new EmployeeTableModel();
    private final JTable table = new JTable(model);

    public EmployeeApp() {
        super("Employees");
        setDefaultCloseOperation(EXIT_ON_CLOSE);

        model.add(new Employee("Alice", "Engineering", 120000, true));
        model.add(new Employee("Bob", "Sales", 70000, true));
        model.add(new Employee("Carol", "HR", 65000, false));

        table.setAutoCreateRowSorter(true);
        table.setRowHeight(24);
        table.setFillsViewportHeight(true);
        table.setDefaultRenderer(Double.class, new SalaryRenderer());

        JButton add = new JButton("Add");
        add.addActionListener(e -> {
            String name = JOptionPane.showInputDialog(this, "Name:");
            if (name != null && !name.isBlank())
                model.add(new Employee(name, "Unassigned", 50000, true));
        });
        JButton del = new JButton("Delete selected");
        del.addActionListener(e -> {
            int viewRow = table.getSelectedRow();
            if (viewRow >= 0) model.remove(table.convertRowIndexToModel(viewRow));
        });

        JPanel south = new JPanel(new FlowLayout(FlowLayout.RIGHT));
        south.add(add);
        south.add(del);

        add(new JScrollPane(table), BorderLayout.CENTER);
        add(south, BorderLayout.SOUTH);
        setSize(600, 350);
        setLocationRelativeTo(null);
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> new EmployeeApp().setVisible(true));
    }
}
// plus the Employee record, EmployeeTableModel and SalaryRenderer from above
```

### Exercise 10
Add a search box that filters by name, a "salary > X" spinner filter, and a footer label showing the total salary of the visible rows (hint: listen to `TableModelEvent`s and `RowSorterEvent`s).

---

## 11. Trees (JTree)

### 11.1 Basic tree

```java
DefaultMutableTreeNode root = new DefaultMutableTreeNode("Company");
DefaultMutableTreeNode eng = new DefaultMutableTreeNode("Engineering");
eng.add(new DefaultMutableTreeNode("Alice"));
eng.add(new DefaultMutableTreeNode("Bob"));
DefaultMutableTreeNode sales = new DefaultMutableTreeNode("Sales");
sales.add(new DefaultMutableTreeNode("Carol"));
root.add(eng);
root.add(sales);

JTree tree = new JTree(root);
tree.setRootVisible(true);
tree.setShowsRootHandles(true);
frame.add(new JScrollPane(tree));
```

Imports: `javax.swing.tree.*` and `javax.swing.event.*`.

### 11.2 Selection and expansion

```java
tree.getSelectionModel().setSelectionMode(TreeSelectionModel.SINGLE_TREE_SELECTION);
tree.addTreeSelectionListener(e -> {
    DefaultMutableTreeNode node = (DefaultMutableTreeNode) tree.getLastSelectedPathComponent();
    if (node == null) return;
    System.out.println("Selected: " + node.getUserObject() + (node.isLeaf() ? " (leaf)" : ""));
});

// Expand everything
for (int i = 0; i < tree.getRowCount(); i++) tree.expandRow(i);   // grows as rows expand

tree.expandPath(new TreePath(eng.getPath()));
tree.collapseRow(0);
```

### 11.3 Modifying a tree dynamically

Use the **tree model** so the view updates:

```java
DefaultTreeModel model = (DefaultTreeModel) tree.getModel();

DefaultMutableTreeNode parent = (DefaultMutableTreeNode) tree.getLastSelectedPathComponent();
DefaultMutableTreeNode child = new DefaultMutableTreeNode("New node");
model.insertNodeInto(child, parent, parent.getChildCount());
tree.scrollPathToVisible(new TreePath(child.getPath()));

// Remove
model.removeNodeFromParent(child);

// Rename: change user object then notify
child.setUserObject("Renamed");
model.nodeChanged(child);
```

### 11.4 Custom icons and renderer

```java
tree.setCellRenderer(new DefaultTreeCellRenderer() {
    @Override
    public Component getTreeCellRendererComponent(JTree t, Object value, boolean sel,
            boolean expanded, boolean leaf, int row, boolean focus) {
        super.getTreeCellRendererComponent(t, value, sel, expanded, leaf, row, focus);
        DefaultMutableTreeNode node = (DefaultMutableTreeNode) value;
        if (node.getUserObject() instanceof java.io.File f) {
            setText(f.getName().isEmpty() ? f.getPath() : f.getName());
        }
        return this;
    }
});
```

### 11.5 Lazy-loading file system tree

```java
class FileNode extends DefaultMutableTreeNode {
    private boolean loaded = false;
    FileNode(java.io.File f) { super(f); }
    @Override public boolean isLeaf() { return !((java.io.File) getUserObject()).isDirectory(); }
    void loadChildren(DefaultTreeModel model) {
        if (loaded) return;
        loaded = true;
        java.io.File[] files = ((java.io.File) getUserObject()).listFiles();
        if (files == null) return;
        java.util.Arrays.sort(files);
        for (java.io.File f : files) model.insertNodeInto(new FileNode(f), this, getChildCount());
    }
}

FileNode root = new FileNode(new java.io.File(System.getProperty("user.home")));
DefaultTreeModel model = new DefaultTreeModel(root);
JTree tree = new JTree(model);
root.loadChildren(model);

tree.addTreeWillExpandListener(new TreeWillExpandListener() {
    @Override public void treeWillExpand(TreeExpansionEvent e) {
        ((FileNode) e.getPath().getLastPathComponent()).loadChildren(model);
    }
    @Override public void treeWillCollapse(TreeExpansionEvent e) { }
});
```

(For big directories, do the `listFiles()` call in a `SwingWorker`; see Section 19.)

---

## 12. Text Components in Depth

### 12.1 The hierarchy

```
JTextComponent
 ├─ JTextField ── JPasswordField, JFormattedTextField
 ├─ JTextArea           (plain text)
 └─ JEditorPane         (HTML/RTF display)
      └─ JTextPane      (styled text, your own formatting)
```

All use a **Document** as the model. Everything (listeners, undo, filters) hooks into the document.

### 12.2 Common JTextComponent operations

```java
text.getText();  text.setText("...");
text.getSelectedText();  text.select(0, 5);  text.selectAll();
text.setCaretPosition(text.getDocument().getLength());  // move to end
text.cut(); text.copy(); text.paste();
text.setEditable(false);
text.getDocument().getLength();
```

JTextArea extras:

```java
area.append("log line\n");
area.insert("text", 0);
area.replaceRange("new", 0, 3);
area.setTabSize(4);
area.getLineCount();
area.getLineOfOffset(area.getCaretPosition());
```

Auto-scroll a log to the bottom:

```java
area.append(line + "\n");
area.setCaretPosition(area.getDocument().getLength());
```

### 12.3 JFormattedTextField

```java
// Number
JFormattedTextField amount = new JFormattedTextField(java.text.NumberFormat.getIntegerInstance());
amount.setValue(100);
amount.setColumns(10);
int n = ((Number) amount.getValue()).intValue();

// Date
JFormattedTextField date = new JFormattedTextField(new java.text.SimpleDateFormat("yyyy-MM-dd"));
date.setValue(new java.util.Date());

// Mask: phone number
import javax.swing.text.MaskFormatter;
MaskFormatter mask = new MaskFormatter("(###) ###-####");   // # = digit
mask.setPlaceholderCharacter('_');
JFormattedTextField phone = new JFormattedTextField(mask);

// React to a committed value
amount.addPropertyChangeListener("value", e -> System.out.println("New value " + e.getNewValue()));
```

### 12.4 DocumentFilter: restricting input

```java
import javax.swing.text.*;

class DigitsOnlyFilter extends DocumentFilter {
    private final int maxLength;
    DigitsOnlyFilter(int maxLength) { this.maxLength = maxLength; }

    @Override
    public void insertString(FilterBypass fb, int offset, String s, AttributeSet a) throws BadLocationException {
        replace(fb, offset, 0, s, a);
    }
    @Override
    public void replace(FilterBypass fb, int offset, int length, String s, AttributeSet a) throws BadLocationException {
        if (s == null) return;
        String digits = s.replaceAll("\\D", "");
        int newLength = fb.getDocument().getLength() - length + digits.length();
        if (newLength <= maxLength) super.replace(fb, offset, length, digits, a);
        else Toolkit.getDefaultToolkit().beep();
    }
}

((AbstractDocument) textField.getDocument()).setDocumentFilter(new DigitsOnlyFilter(5));
```

### 12.5 JTextPane: styled text

```java
JTextPane pane = new JTextPane();
StyledDocument doc = pane.getStyledDocument();

Style bold = pane.addStyle("bold", null);
StyleConstants.setBold(bold, true);

Style red = pane.addStyle("red", null);
StyleConstants.setForeground(red, Color.RED);
StyleConstants.setFontSize(red, 18);

try {
    doc.insertString(doc.getLength(), "Normal, ", null);
    doc.insertString(doc.getLength(), "bold, ", bold);
    doc.insertString(doc.getLength(), "big red.\n", red);
} catch (BadLocationException e) { e.printStackTrace(); }

// Style the current selection
SimpleAttributeSet attrs = new SimpleAttributeSet();
StyleConstants.setItalic(attrs, true);
doc.setCharacterAttributes(pane.getSelectionStart(),
        pane.getSelectionEnd() - pane.getSelectionStart(), attrs, false);

// Insert components or icons
pane.insertComponent(new JButton("Inline button"));
pane.insertIcon(new ImageIcon("smile.png"));
```

Built-in editor actions: `StyledEditorKit.BoldAction`, `ItalicAction`, `UnderlineAction`, etc. can be attached directly to buttons:

```java
JButton boldBtn = new JButton(new StyledEditorKit.BoldAction());
boldBtn.setText("B");
```

### 12.6 Syntax-highlight sketch

```java
private void highlight(JTextPane pane) {
    StyledDocument doc = pane.getStyledDocument();
    Style normal = pane.addStyle("n", null);
    Style keyword = pane.addStyle("k", null);
    StyleConstants.setForeground(keyword, new Color(0, 0, 160));
    StyleConstants.setBold(keyword, true);

    doc.setCharacterAttributes(0, doc.getLength(), normal, true);
    java.util.regex.Matcher m = java.util.regex.Pattern
            .compile("\\b(public|class|static|void|int|return|if|else|for|while)\\b")
            .matcher(pane.getText());
    while (m.find()) doc.setCharacterAttributes(m.start(), m.end() - m.start(), keyword, false);
}
```

Call it from a `DocumentListener` using `SwingUtilities.invokeLater` (never mutate inside the notification).

### 12.7 JEditorPane: displaying HTML

```java
JEditorPane html = new JEditorPane("text/html",
        "<html><body><h1>Hello</h1><p>This is <b>HTML 3.2</b> only; no modern CSS!</p></body></html>");
html.setEditable(false);
html.addHyperlinkListener(e -> {
    if (e.getEventType() == javax.swing.event.HyperlinkEvent.EventType.ACTIVATED) {
        try { Desktop.getDesktop().browse(e.getURL().toURI()); } catch (Exception ex) { }
    }
});
```

Swing's HTML support is old (HTML 3.2 plus limited CSS), so it suits help text or simple formatted messages, not browsers.

---

## 13. Borders, Icons & Images

### 13.1 Borders

```java
import javax.swing.border.*;

panel.setBorder(BorderFactory.createEmptyBorder(10, 10, 10, 10));   // padding (top, left, bottom, right)
panel.setBorder(BorderFactory.createLineBorder(Color.GRAY, 2));
panel.setBorder(BorderFactory.createTitledBorder("Options"));       // group box
panel.setBorder(BorderFactory.createEtchedBorder());
panel.setBorder(BorderFactory.createRaisedBevelBorder());
panel.setBorder(new LineBorder(Color.RED, 3, true));                // rounded

// Combine: outside, inside
panel.setBorder(BorderFactory.createCompoundBorder(
        BorderFactory.createTitledBorder("Details"),
        BorderFactory.createEmptyBorder(8, 8, 8, 8)));
```

Custom rounded border:

```java
class RoundedBorder extends AbstractBorder {
    private final int radius;
    RoundedBorder(int radius) { this.radius = radius; }
    @Override public void paintBorder(Component c, Graphics g, int x, int y, int w, int h) {
        Graphics2D g2 = (Graphics2D) g.create();
        g2.setRenderingHint(RenderingHints.KEY_ANTIALIASING, RenderingHints.VALUE_ANTIALIAS_ON);
        g2.setColor(Color.GRAY);
        g2.drawRoundRect(x, y, w - 1, h - 1, radius, radius);
        g2.dispose();
    }
    @Override public Insets getBorderInsets(Component c) { return new Insets(radius/2, radius/2, radius/2, radius/2); }
}
```

### 13.2 Icons

```java
// From a file path
ImageIcon icon = new ImageIcon("images/save.png");

// From the classpath (works inside a JAR!): put images in src/main/resources/images/
java.net.URL url = MyClass.class.getResource("/images/save.png");
ImageIcon icon2 = new ImageIcon(url);

button.setIcon(icon2);
button.setHorizontalTextPosition(SwingConstants.RIGHT);
label.setIcon(icon2);
```

**Always load via `getResource` for anything you ship.** Relative file paths break when the app is run from a different directory or packaged.

### 13.3 Scaling images

```java
Image scaled = icon.getImage().getScaledInstance(32, 32, Image.SCALE_SMOOTH);
ImageIcon small = new ImageIcon(scaled);
```

Better quality using `BufferedImage` and `Graphics2D`:

```java
static BufferedImage resize(BufferedImage src, int w, int h) {
    BufferedImage out = new BufferedImage(w, h, BufferedImage.TYPE_INT_ARGB);
    Graphics2D g = out.createGraphics();
    g.setRenderingHint(RenderingHints.KEY_INTERPOLATION, RenderingHints.VALUE_INTERPOLATION_BICUBIC);
    g.setRenderingHint(RenderingHints.KEY_RENDERING, RenderingHints.VALUE_RENDER_QUALITY);
    g.drawImage(src, 0, 0, w, h, null);
    g.dispose();
    return out;
}
```

### 13.4 Reading images (ImageIO)

```java
import javax.imageio.ImageIO;
BufferedImage img = ImageIO.read(MyClass.class.getResource("/images/photo.jpg")); // throws IOException
ImageIO.write(img, "png", new java.io.File("out.png"));
```

Supported by default: PNG, JPEG, GIF, BMP, WBMP.

### 13.5 Icon from drawing code (no files needed)

```java
class CircleIcon implements Icon {
    private final int size; private final Color color;
    CircleIcon(int size, Color color) { this.size = size; this.color = color; }
    public int getIconWidth()  { return size; }
    public int getIconHeight() { return size; }
    public void paintIcon(Component c, Graphics g, int x, int y) {
        Graphics2D g2 = (Graphics2D) g.create();
        g2.setRenderingHint(RenderingHints.KEY_ANTIALIASING, RenderingHints.VALUE_ANTIALIAS_ON);
        g2.setColor(color);
        g2.fillOval(x, y, size, size);
        g2.dispose();
    }
}
button.setIcon(new CircleIcon(12, Color.GREEN));
```

---

## 14. Look and Feel

The **Look and Feel (L&F)** controls how all components are drawn.

### 14.1 Built-in L&Fs

```java
// Must be set BEFORE creating components
try {
    UIManager.setLookAndFeel(UIManager.getSystemLookAndFeelClassName()); // native look
} catch (Exception e) { e.printStackTrace(); }

// Nimbus: modern-ish cross-platform
for (UIManager.LookAndFeelInfo info : UIManager.getInstalledLookAndFeels()) {
    if ("Nimbus".equals(info.getName())) {
        UIManager.setLookAndFeel(info.getClassName());
        break;
    }
}

// Default cross-platform ("Metal")
UIManager.setLookAndFeel(UIManager.getCrossPlatformLookAndFeelClassName());
```

Switch at runtime:

```java
UIManager.setLookAndFeel(className);
SwingUtilities.updateComponentTreeUI(frame);   // refresh existing components
frame.pack();
```

### 14.2 FlatLaf: the modern choice

[FlatLaf](https://www.formdev.com/flatlaf/) gives you a clean, modern flat look with light/dark themes, HiDPI support, and many tweaks.

Maven:

```xml
<dependency>
    <groupId>com.formdev</groupId>
    <artifactId>flatlaf</artifactId>
    <version>3.5.1</version>   <!-- check for the latest -->
</dependency>
```

Gradle: `implementation("com.formdev:flatlaf:3.5.1")`

```java
import com.formdev.flatlaf.FlatLightLaf;  // or FlatDarkLaf, FlatIntelliJLaf, FlatDarculaLaf

public static void main(String[] args) {
    FlatLightLaf.setup();                       // before creating any UI
    SwingUtilities.invokeLater(() -> new MainWindow().setVisible(true));
}

// Per-component styling via client properties
button.putClientProperty("JButton.buttonType", "roundRect");
textField.putClientProperty("JTextField.placeholderText", "Search...");
textField.putClientProperty("JTextField.showClearButton", true);
```

Toggle dark/light at runtime:

```java
FlatDarkLaf.setup();
FlatLaf.updateUI();
```

### 14.3 UIManager defaults

```java
UIManager.put("Button.font", new Font("Segoe UI", Font.PLAIN, 14));
UIManager.put("Panel.background", new Color(250, 250, 250));
UIManager.put("OptionPane.yesButtonText", "Yes please");
UIManager.put("Table.alternateRowColor", new Color(245, 245, 245));
```

Call these **before** creating components (or call `updateComponentTreeUI` afterward).

### 14.4 Cross-platform tips

- macOS: `System.setProperty("apple.laf.useScreenMenuBar", "true");` puts the menu in the system bar. Set it before any AWT use.
- Don't hardcode colors/fonts everywhere. Use `UIManager.getColor("Panel.background")` etc. so your app respects the active theme.

---

## 15. Actions & Keyboard Shortcuts (Key Bindings)

### 15.1 The Action pattern

A single `Action` object can power a menu item, a toolbar button, and a shortcut, keeping text/icon/enabled state in sync.

```java
class SaveAction extends AbstractAction {
    SaveAction() {
        super("Save", new ImageIcon(SaveAction.class.getResource("/images/save.png")));
        putValue(SHORT_DESCRIPTION, "Save the document");                // tooltip
        putValue(MNEMONIC_KEY, KeyEvent.VK_S);
        putValue(ACCELERATOR_KEY, KeyStroke.getKeyStroke(KeyEvent.VK_S,
                Toolkit.getDefaultToolkit().getMenuShortcutKeyMaskEx()));
    }
    @Override public void actionPerformed(ActionEvent e) { save(); }
}

Action save = new SaveAction();
fileMenu.add(new JMenuItem(save));       // menu item
toolbar.add(save);                       // toolbar button
save.setEnabled(false);                  // disables BOTH at once
```

Lambda-friendly helper:

```java
static Action action(String name, Runnable r) {
    return new AbstractAction(name) {
        @Override public void actionPerformed(ActionEvent e) { r.run(); }
    };
}
```

### 15.2 Key Bindings (InputMap + ActionMap)

Two maps: **InputMap** maps `KeyStroke` -> an action *name*; **ActionMap** maps name -> `Action`.

```java
JComponent comp = panel;
int when = JComponent.WHEN_IN_FOCUSED_WINDOW;   // works when anywhere in the window has focus

comp.getInputMap(when).put(KeyStroke.getKeyStroke("control S"), "save");
comp.getActionMap().put("save", save);

// More examples
comp.getInputMap(when).put(KeyStroke.getKeyStroke("ESCAPE"), "close");
comp.getInputMap(when).put(KeyStroke.getKeyStroke("F5"), "refresh");
comp.getInputMap(when).put(KeyStroke.getKeyStroke("shift alt D"), "debug");
comp.getInputMap(when).put(KeyStroke.getKeyStroke(KeyEvent.VK_LEFT, 0), "left");
comp.getInputMap(when).put(KeyStroke.getKeyStroke("released SPACE"), "spaceUp");
```

The three focus conditions:

| Condition | Active when |
|---|---|
| `WHEN_FOCUSED` | this component has focus |
| `WHEN_ANCESTOR_OF_FOCUSED_COMPONENT` | focus is in a child of this component |
| `WHEN_IN_FOCUSED_WINDOW` | anywhere in the window |

### 15.3 Why prefer Key Bindings over KeyListener?

- Don't require the component to be focused (with `WHEN_IN_FOCUSED_WINDOW`)
- Cooperates with the L&F (you can rebind or override existing shortcuts)
- Easy to enable/disable via `Action`
- No need to implement 3 methods or filter events

### 15.4 Game-style example: move a square with arrows

```java
import javax.swing.*;
import java.awt.*;
import java.awt.event.*;

public class ArrowMove extends JPanel {
    private int x = 100, y = 100;

    public ArrowMove() {
        setFocusable(true);
        bind("LEFT",  -10,   0);
        bind("RIGHT",  10,   0);
        bind("UP",      0, -10);
        bind("DOWN",    0,  10);
    }

    private void bind(String key, int dx, int dy) {
        getInputMap(WHEN_IN_FOCUSED_WINDOW).put(KeyStroke.getKeyStroke(key), key);
        getActionMap().put(key, new AbstractAction() {
            @Override public void actionPerformed(ActionEvent e) {
                x += dx; y += dy; repaint();
            }
        });
    }

    @Override protected void paintComponent(Graphics g) {
        super.paintComponent(g);
        g.setColor(Color.BLUE);
        g.fillRect(x, y, 30, 30);
    }

    @Override public Dimension getPreferredSize() { return new Dimension(400, 300); }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame f = new JFrame("Arrows");
            f.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            f.add(new ArrowMove());
            f.pack();
            f.setLocationRelativeTo(null);
            f.setVisible(true);
        });
    }
}
```

---

## 16. Input Validation

### 16.1 InputVerifier (validate on focus loss)

```java
JTextField ageField = new JTextField(5);
ageField.setInputVerifier(new InputVerifier() {
    @Override public boolean verify(JComponent input) {
        try {
            int age = Integer.parseInt(((JTextField) input).getText().trim());
            return age >= 0 && age <= 150;
        } catch (NumberFormatException e) {
            return false;
        }
    }
});
// Focus cannot leave the field while verify() returns false.
```

Too aggressive for many UIs. A friendlier approach is the next one.

### 16.2 Live validation with visual feedback

```java
private void validateEmail(JTextField field, JLabel error) {
    boolean ok = field.getText().matches("[\\w.+-]+@[\\w-]+\\.[\\w.]+");
    field.setBackground(ok || field.getText().isEmpty() ? UIManager.getColor("TextField.background")
                                                         : new Color(255, 220, 220));
    error.setText(ok || field.getText().isEmpty() ? " " : "Invalid email address");
}

emailField.getDocument().addDocumentListener(new SimpleDocumentListener(
        () -> validateEmail(emailField, errorLabel)));
```

### 16.3 Form-level validation pattern

```java
private boolean validateForm() {
    List<String> errors = new ArrayList<>();
    if (nameField.getText().isBlank())           errors.add("Name is required");
    if (!emailField.getText().contains("@"))     errors.add("Email looks invalid");
    if (passwordField.getPassword().length < 8)  errors.add("Password needs 8+ characters");

    if (!errors.isEmpty()) {
        JOptionPane.showMessageDialog(this, String.join("\n", errors),
                "Please fix the following", JOptionPane.WARNING_MESSAGE);
        return false;
    }
    return true;
}
submitButton.addActionListener(e -> { if (validateForm()) submit(); });
```

Enable/disable the submit button reactively:

```java
Runnable update = () -> submitButton.setEnabled(
        !nameField.getText().isBlank() && emailField.getText().contains("@"));
nameField.getDocument().addDocumentListener(new SimpleDocumentListener(update));
emailField.getDocument().addDocumentListener(new SimpleDocumentListener(update));
update.run();
```


---

# Part 3: Advanced

## 17. Custom Painting

### 17.1 How painting works

Swing repaints a component by calling `paintComponent(Graphics g)`. You never call it directly; you call `repaint()` and Swing schedules it on the EDT.

```
repaint()  ->  (EDT) paint()  ->  paintComponent()  ->  paintBorder()  ->  paintChildren()
```

**Rules:**
1. Override `paintComponent`, not `paint`.
2. Call `super.paintComponent(g)` first (it clears the background if the component is opaque).
3. Never keep or reuse the `Graphics` object after the method returns.
4. Never do slow work or change state in `paintComponent`. Compute elsewhere; paint just draws.
5. Cast to `Graphics2D` for the good API.

### 17.2 Hello, graphics

```java
import javax.swing.*;
import java.awt.*;
import java.awt.geom.*;

public class DrawingPanel extends JPanel {

    @Override
    protected void paintComponent(Graphics g) {
        super.paintComponent(g);
        Graphics2D g2 = (Graphics2D) g.create();   // copy so our changes don't leak
        try {
            g2.setRenderingHint(RenderingHints.KEY_ANTIALIASING, RenderingHints.VALUE_ANTIALIAS_ON);
            g2.setRenderingHint(RenderingHints.KEY_TEXT_ANTIALIASING, RenderingHints.VALUE_TEXT_ANTIALIAS_ON);

            // Lines and shapes
            g2.setColor(Color.RED);
            g2.setStroke(new BasicStroke(3f));
            g2.drawLine(10, 10, 200, 10);

            g2.setColor(new Color(0, 120, 215));
            g2.fillRect(10, 30, 100, 60);
            g2.setColor(Color.BLACK);
            g2.drawRect(10, 30, 100, 60);

            g2.fillOval(130, 30, 60, 60);
            g2.drawRoundRect(210, 30, 100, 60, 20, 20);

            // Polygon
            g2.fillPolygon(new int[]{340, 380, 300}, new int[]{30, 90, 90}, 3);

            // Gradient fill
            g2.setPaint(new GradientPaint(10, 110, Color.ORANGE, 310, 170, Color.MAGENTA));
            g2.fillRoundRect(10, 110, 300, 60, 20, 20);

            // Text
            g2.setColor(Color.DARK_GRAY);
            g2.setFont(new Font("SansSerif", Font.BOLD, 24));
            g2.drawString("Hello, Graphics2D", 10, 210);

            // Centered text using FontMetrics
            String s = "Centered";
            FontMetrics fm = g2.getFontMetrics();
            int tx = (getWidth() - fm.stringWidth(s)) / 2;
            int ty = (getHeight() + fm.getAscent() - fm.getDescent()) / 2;
            g2.drawString(s, tx, ty);

            // Shapes API (java.awt.geom) with transforms
            Shape star = new Rectangle2D.Double(-20, -20, 40, 40);
            g2.translate(350, 200);
            g2.rotate(Math.toRadians(45));
            g2.setColor(Color.GREEN.darker());
            g2.fill(star);
        } finally {
            g2.dispose();
        }
    }

    @Override public Dimension getPreferredSize() { return new Dimension(450, 280); }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame f = new JFrame("Graphics2D");
            f.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            f.add(new DrawingPanel());
            f.pack();
            f.setLocationRelativeTo(null);
            f.setVisible(true);
        });
    }
}
```

### 17.3 Graphics2D quick reference

| Concept | API |
|---|---|
| Color / paint | `setColor`, `setPaint(GradientPaint / TexturePaint)` |
| Line style | `setStroke(new BasicStroke(width, cap, join, miter, dashArray, phase))` |
| Transforms | `translate`, `rotate`, `scale`, `shear`, `setTransform`, `getTransform` |
| Clipping | `setClip(Shape)`, `clip(Shape)` |
| Transparency | `setComposite(AlphaComposite.getInstance(AlphaComposite.SRC_OVER, 0.5f))` |
| Shapes | `Line2D`, `Rectangle2D`, `RoundRectangle2D`, `Ellipse2D`, `Arc2D`, `Path2D`, `Area` |
| Shape ops | `Area a = new Area(s1); a.subtract(new Area(s2));` |
| Images | `drawImage(img, x, y, observer)` |
| Quality | `RenderingHints` (antialiasing, interpolation, stroke control) |

Dashed line:

```java
float[] dash = {8f, 4f};
g2.setStroke(new BasicStroke(2f, BasicStroke.CAP_BUTT, BasicStroke.JOIN_MITER, 10f, dash, 0f));
```

### 17.4 Interactive drawing: a mini paint program

```java
import javax.swing.*;
import java.awt.*;
import java.awt.event.*;
import java.awt.image.BufferedImage;

public class PaintPanel extends JPanel {
    private BufferedImage canvas;
    private Point last;
    private Color color = Color.BLACK;
    private int brush = 4;

    public PaintPanel() {
        setBackground(Color.WHITE);
        MouseAdapter ma = new MouseAdapter() {
            @Override public void mousePressed(MouseEvent e) { last = e.getPoint(); drawTo(e.getPoint()); }
            @Override public void mouseDragged(MouseEvent e) { drawTo(e.getPoint()); }
            @Override public void mouseReleased(MouseEvent e) { last = null; }
        };
        addMouseListener(ma);
        addMouseMotionListener(ma);
    }

    private void ensureCanvas() {
        if (canvas == null || canvas.getWidth() != getWidth() || canvas.getHeight() != getHeight()) {
            BufferedImage n = new BufferedImage(Math.max(1, getWidth()), Math.max(1, getHeight()),
                    BufferedImage.TYPE_INT_ARGB);
            Graphics2D g = n.createGraphics();
            g.setColor(Color.WHITE);
            g.fillRect(0, 0, n.getWidth(), n.getHeight());
            if (canvas != null) g.drawImage(canvas, 0, 0, null);   // keep old drawing on resize
            g.dispose();
            canvas = n;
        }
    }

    private void drawTo(Point p) {
        ensureCanvas();
        Graphics2D g = canvas.createGraphics();
        g.setRenderingHint(RenderingHints.KEY_ANTIALIASING, RenderingHints.VALUE_ANTIALIAS_ON);
        g.setColor(color);
        g.setStroke(new BasicStroke(brush, BasicStroke.CAP_ROUND, BasicStroke.JOIN_ROUND));
        g.drawLine(last.x, last.y, p.x, p.y);
        g.dispose();
        last = p;
        repaint();
    }

    public void setColor(Color c) { color = c; }
    public void setBrush(int size) { brush = size; }
    public void clear() { canvas = null; ensureCanvas(); repaint(); }
    public BufferedImage getImage() { ensureCanvas(); return canvas; }

    @Override protected void paintComponent(Graphics g) {
        super.paintComponent(g);
        ensureCanvas();
        g.drawImage(canvas, 0, 0, null);
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame f = new JFrame("Mini Paint");
            f.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            PaintPanel paint = new PaintPanel();

            JToolBar tb = new JToolBar();
            JButton colorBtn = new JButton("Colour...");
            colorBtn.addActionListener(e -> {
                Color c = JColorChooser.showDialog(f, "Colour", Color.BLACK);
                if (c != null) paint.setColor(c);
            });
            JSlider size = new JSlider(1, 40, 4);
            size.addChangeListener(e -> paint.setBrush(size.getValue()));
            JButton clear = new JButton("Clear");
            clear.addActionListener(e -> paint.clear());
            tb.add(colorBtn); tb.add(new JLabel(" Size ")); tb.add(size); tb.add(clear);

            f.add(tb, BorderLayout.NORTH);
            f.add(paint, BorderLayout.CENTER);
            f.setSize(800, 600);
            f.setLocationRelativeTo(null);
            f.setVisible(true);
        });
    }
}
```

The technique: draw into an off-screen `BufferedImage` (persistent state) and just blit it in `paintComponent`. This is **double buffering**, and Swing components already double-buffer for you, but keeping your own model of "what has been drawn" is the correct approach for drawing apps.

### 17.5 Painting vs. model: the vector approach

For editable graphics (shapes you can select/move), store *objects* and redraw them all:

```java
interface Shape2 { void draw(Graphics2D g); boolean contains(Point p); void moveBy(int dx, int dy); }

class Board extends JPanel {
    private final List<Shape2> shapes = new ArrayList<>();
    @Override protected void paintComponent(Graphics g) {
        super.paintComponent(g);
        Graphics2D g2 = (Graphics2D) g;
        g2.setRenderingHint(RenderingHints.KEY_ANTIALIASING, RenderingHints.VALUE_ANTIALIAS_ON);
        for (Shape2 s : shapes) s.draw(g2);
    }
}
```

### 17.6 Efficient repainting

```java
repaint();                          // whole component
repaint(x, y, w, h);                // only a region; much cheaper for big canvases
revalidate();                       // layout changed (components added/removed); then repaint()
```

Typical pair after adding/removing children:

```java
panel.add(newComp);
panel.revalidate();
panel.repaint();
```

In `paintComponent`, use `g.getClipBounds()` to skip drawing things outside the dirty area.

---

## 18. Animation with Timers

### 18.1 javax.swing.Timer

`javax.swing.Timer` fires `ActionEvent`s **on the EDT**, so it's safe to update components from it. (Don't use `java.util.Timer` for UI work.)

```java
Timer timer = new Timer(16, e -> {          // ~60 FPS
    update();
    repaint();
});
timer.start();
// timer.stop();  timer.setDelay(50);  timer.setRepeats(false) for one-shot
```

### 18.2 Bouncing ball

```java
import javax.swing.*;
import java.awt.*;

public class BouncingBall extends JPanel {
    private double x = 50, y = 50, vx = 180, vy = 140;      // pixels per second
    private final int size = 40;
    private long lastTime = System.nanoTime();

    public BouncingBall() {
        setBackground(Color.BLACK);
        new Timer(10, e -> step()).start();
    }

    private void step() {
        long now = System.nanoTime();
        double dt = (now - lastTime) / 1e9;                   // delta time keeps speed frame-rate independent
        lastTime = now;

        x += vx * dt;
        y += vy * dt;
        if (x < 0)                  { x = 0;                  vx = -vx; }
        if (y < 0)                  { y = 0;                  vy = -vy; }
        if (x + size > getWidth())  { x = getWidth() - size;  vx = -vx; }
        if (y + size > getHeight()) { y = getHeight() - size; vy = -vy; }
        repaint();
    }

    @Override protected void paintComponent(Graphics g) {
        super.paintComponent(g);
        Graphics2D g2 = (Graphics2D) g;
        g2.setRenderingHint(RenderingHints.KEY_ANTIALIASING, RenderingHints.VALUE_ANTIALIAS_ON);
        g2.setColor(Color.ORANGE);
        g2.fillOval((int) x, (int) y, size, size);
        Toolkit.getDefaultToolkit().sync();                   // smoother on Linux
    }

    @Override public Dimension getPreferredSize() { return new Dimension(500, 350); }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame f = new JFrame("Bouncing Ball");
            f.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            f.add(new BouncingBall());
            f.pack();
            f.setLocationRelativeTo(null);
            f.setVisible(true);
        });
    }
}
```

### 18.3 Easing / tweening for UI effects

Fade a component or panel slide:

```java
class Tween {
    static void animate(int durationMs, java.util.function.DoubleConsumer onFrame, Runnable onDone) {
        long start = System.currentTimeMillis();
        Timer t = new Timer(15, null);
        t.addActionListener(e -> {
            double p = Math.min(1.0, (System.currentTimeMillis() - start) / (double) durationMs);
            double eased = 1 - Math.pow(1 - p, 3);          // ease-out cubic
            onFrame.accept(eased);
            if (p >= 1.0) { t.stop(); if (onDone != null) onDone.run(); }
        });
        t.start();
    }
}

// Slide a panel's width from 0 to 250
Tween.animate(300, p -> {
    sidebar.setPreferredSize(new Dimension((int) (250 * p), 1));
    container.revalidate();
}, null);
```

### 18.4 Clock example (simple Timer use)

```java
JLabel clock = new JLabel();
Timer t = new Timer(1000, e -> clock.setText(java.time.LocalTime.now()
        .format(java.time.format.DateTimeFormatter.ofPattern("HH:mm:ss"))));
t.setInitialDelay(0);
t.start();
```

---

## 19. Concurrency: SwingWorker

### 19.1 The problem

- Long tasks on the EDT freeze the UI.
- Updating components from background threads is illegal.

`SwingWorker<T, V>` solves both:

| Method | Runs on | Purpose |
|---|---|---|
| `doInBackground()` | background thread | the slow work; returns `T` |
| `publish(V... chunks)` | called from background | send intermediate results |
| `process(List<V> chunks)` | **EDT** | receive published chunks, update UI |
| `done()` | **EDT** | called when finished; `get()` the result |
| `setProgress(int)` | called from background | fires `"progress"` property change on the EDT |
| `cancel(true)` | any | request cancellation |

### 19.2 Progress bar + cancel

```java
import javax.swing.*;
import java.awt.*;
import java.util.List;
import java.util.concurrent.ExecutionException;

public class WorkerDemo extends JFrame {
    private final JProgressBar bar = new JProgressBar(0, 100);
    private final JTextArea log = new JTextArea(8, 30);
    private final JButton start = new JButton("Start");
    private final JButton cancel = new JButton("Cancel");
    private SwingWorker<Integer, String> worker;

    public WorkerDemo() {
        super("SwingWorker");
        setDefaultCloseOperation(EXIT_ON_CLOSE);
        bar.setStringPainted(true);
        cancel.setEnabled(false);
        log.setEditable(false);

        start.addActionListener(e -> startWork());
        cancel.addActionListener(e -> worker.cancel(true));

        JPanel buttons = new JPanel();
        buttons.add(start);
        buttons.add(cancel);
        add(bar, BorderLayout.NORTH);
        add(new JScrollPane(log), BorderLayout.CENTER);
        add(buttons, BorderLayout.SOUTH);
        pack();
        setLocationRelativeTo(null);
    }

    private void startWork() {
        start.setEnabled(false);
        cancel.setEnabled(true);
        log.setText("");

        worker = new SwingWorker<>() {
            @Override protected Integer doInBackground() throws Exception {
                int total = 20;
                for (int i = 1; i <= total; i++) {
                    if (isCancelled()) return i;                 // cooperative cancellation
                    Thread.sleep(200);                           // simulate work (safe here: not on EDT)
                    publish("Processed item " + i);
                    setProgress(i * 100 / total);
                }
                return total;
            }
            @Override protected void process(List<String> chunks) {      // EDT
                chunks.forEach(s -> log.append(s + "\n"));
            }
            @Override protected void done() {                            // EDT
                start.setEnabled(true);
                cancel.setEnabled(false);
                if (isCancelled()) { log.append("Cancelled.\n"); return; }
                try {
                    log.append("Finished: " + get() + " items\n");
                } catch (InterruptedException | ExecutionException ex) {
                    log.append("Error: " + ex.getCause() + "\n");
                }
            }
        };
        worker.addPropertyChangeListener(evt -> {
            if ("progress".equals(evt.getPropertyName())) bar.setValue((Integer) evt.getNewValue());
        });
        worker.execute();
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> new WorkerDemo().setVisible(true));
    }
}
```

### 19.3 Loading data into a table in the background

```java
new SwingWorker<List<Employee>, Void>() {
    @Override protected List<Employee> doInBackground() throws Exception {
        return repository.loadAll();        // slow DB / network call
    }
    @Override protected void done() {
        try {
            get().forEach(tableModel::add);
        } catch (Exception ex) {
            JOptionPane.showMessageDialog(frame, "Load failed: " + ex.getCause().getMessage());
        } finally {
            spinner.setVisible(false);
        }
    }
}.execute();
spinner.setVisible(true);
```

### 19.4 Modern alternative: CompletableFuture + invokeLater

```java
java.util.concurrent.CompletableFuture
    .supplyAsync(() -> repository.loadAll())                         // background pool
    .thenAccept(list -> SwingUtilities.invokeLater(() -> {           // back to EDT
        list.forEach(tableModel::add);
    }))
    .exceptionally(ex -> {
        SwingUtilities.invokeLater(() -> showError(ex));
        return null;
    });
```

Both are fine. SwingWorker bundles progress, publish/process, and cancellation. `CompletableFuture` composes well for chains.

### 19.5 Indeterminate progress

```java
bar.setIndeterminate(true);    // spinning/bouncing bar when you don't know the duration
// ... when done:
bar.setIndeterminate(false);
```

### 19.6 Golden rules

1. Short tasks (< ~50 ms) can stay on the EDT. Anything involving I/O, network, DB, or heavy loops goes to a worker.
2. Never touch Swing components in `doInBackground()`.
3. Disable the triggering button while the task runs to prevent double-starts.
4. Handle exceptions from `get()`; wrap them in the `done()` method.
5. Support cancellation by checking `isCancelled()` or catching `InterruptedException`.

---

## 20. Architecture: MVC & Observer

As apps grow, a single class with all the logic becomes unmaintainable. Separate:

| Layer | Responsibility | Swing dependency |
|---|---|---|
| **Model** | Data and business rules | None (pure Java, easily unit-tested) |
| **View** | Components, layout, painting | Yes |
| **Controller / Presenter** | Reacts to user input, updates model, updates view | Yes |

### 20.1 Observable model with PropertyChangeSupport

```java
import java.beans.*;

public class CounterModel {
    private final PropertyChangeSupport pcs = new PropertyChangeSupport(this);
    private int value;

    public int getValue() { return value; }

    public void setValue(int newValue) {
        int old = this.value;
        this.value = newValue;
        pcs.firePropertyChange("value", old, newValue);    // only fires if old != new
    }
    public void increment() { setValue(value + 1); }
    public void reset()     { setValue(0); }

    public void addPropertyChangeListener(PropertyChangeListener l) { pcs.addPropertyChangeListener(l); }
    public void removePropertyChangeListener(PropertyChangeListener l) { pcs.removePropertyChangeListener(l); }
}
```

### 20.2 View

```java
import javax.swing.*;
import java.awt.*;

public class CounterView extends JPanel {
    private final JLabel label = new JLabel("0", SwingConstants.CENTER);
    private final JButton inc = new JButton("+1");
    private final JButton reset = new JButton("Reset");

    public CounterView() {
        super(new BorderLayout(8, 8));
        setBorder(BorderFactory.createEmptyBorder(12, 12, 12, 12));
        label.setFont(label.getFont().deriveFont(Font.BOLD, 40f));
        JPanel south = new JPanel();
        south.add(inc);
        south.add(reset);
        add(label, BorderLayout.CENTER);
        add(south, BorderLayout.SOUTH);
    }

    // The view exposes what controllers need, nothing more.
    public void setCount(int n)                       { label.setText(String.valueOf(n)); }
    public void onIncrement(Runnable r)               { inc.addActionListener(e -> r.run()); }
    public void onReset(Runnable r)                   { reset.addActionListener(e -> r.run()); }
}
```

### 20.3 Controller and wiring

```java
public class CounterController {
    public CounterController(CounterModel model, CounterView view) {
        view.onIncrement(model::increment);
        view.onReset(model::reset);
        model.addPropertyChangeListener(evt -> {
            if ("value".equals(evt.getPropertyName())) view.setCount((Integer) evt.getNewValue());
        });
        view.setCount(model.getValue());
    }
}

public class CounterApp {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            CounterModel model = new CounterModel();
            CounterView view = new CounterView();
            new CounterController(model, view);

            JFrame f = new JFrame("MVC Counter");
            f.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            f.setContentPane(view);
            f.pack();
            f.setLocationRelativeTo(null);
            f.setVisible(true);
        });
    }
}
```

Benefits: you can test `CounterModel` with JUnit without any GUI, swap views, and attach multiple views to the same model.

### 20.4 Passive view / MVP

For maximum testability, make the view an **interface** and the presenter hold all logic:

```java
interface LoginView {
    String username();
    char[] password();
    void showError(String msg);
    void showBusy(boolean busy);
    void close();
    void onLogin(Runnable r);
}

class LoginPresenter {
    LoginPresenter(LoginView view, AuthService auth) {
        view.onLogin(() -> {
            if (view.username().isBlank()) { view.showError("Username required"); return; }
            view.showBusy(true);
            CompletableFuture.supplyAsync(() -> auth.login(view.username(), view.password()))
                .thenAccept(ok -> SwingUtilities.invokeLater(() -> {
                    view.showBusy(false);
                    if (ok) view.close(); else view.showError("Bad credentials");
                }));
        });
    }
}
// Test: pass a fake LoginView implementation, no Swing at all.
```

### 20.5 A simple event bus (for loosely coupled screens)

```java
class EventBus {
    private final Map<Class<?>, List<Consumer<Object>>> handlers = new HashMap<>();
    @SuppressWarnings("unchecked")
    public <T> void subscribe(Class<T> type, Consumer<T> h) {
        handlers.computeIfAbsent(type, k -> new ArrayList<>()).add((Consumer<Object>) h);
    }
    public void publish(Object event) {
        handlers.getOrDefault(event.getClass(), List.of()).forEach(h -> h.accept(event));
    }
}
record DocumentSaved(java.nio.file.Path path) { }
bus.subscribe(DocumentSaved.class, e -> statusBar.setText("Saved " + e.path()));
bus.publish(new DocumentSaved(path));
```

(Publish from the EDT, or wrap handlers in `invokeLater`.)

---

## 21. Building Custom Components

### 21.1 Extend JComponent or JPanel

Example: a **toggle switch** (iOS-style).

```java
import javax.swing.*;
import java.awt.*;
import java.awt.event.*;

public class ToggleSwitch extends JComponent {
    private boolean selected;
    private float knobPos;                         // 0.0 (off) to 1.0 (on) for animation
    private Timer anim;

    public ToggleSwitch() {
        setPreferredSize(new Dimension(52, 28));
        setCursor(Cursor.getPredefinedCursor(Cursor.HAND_CURSOR));
        setFocusable(true);
        addMouseListener(new MouseAdapter() {
            @Override public void mouseClicked(MouseEvent e) { if (isEnabled()) setSelected(!selected); }
        });
        // Keyboard support through key binding
        getInputMap(WHEN_FOCUSED).put(KeyStroke.getKeyStroke("SPACE"), "toggle");
        getActionMap().put("toggle", new AbstractAction() {
            @Override public void actionPerformed(ActionEvent e) { setSelected(!selected); }
        });
    }

    public boolean isSelected() { return selected; }

    public void setSelected(boolean value) {
        if (value == selected) return;
        boolean old = selected;
        selected = value;
        animateKnob();
        firePropertyChange("selected", old, value);   // standard bound property: listeners can subscribe
    }

    private void animateKnob() {
        if (anim != null) anim.stop();
        float target = selected ? 1f : 0f;
        anim = new Timer(10, null);
        anim.addActionListener(e -> {
            knobPos += (target - knobPos) * 0.3f;
            if (Math.abs(target - knobPos) < 0.01f) { knobPos = target; anim.stop(); }
            repaint();
        });
        anim.start();
    }

    @Override protected void paintComponent(Graphics g) {
        Graphics2D g2 = (Graphics2D) g.create();
        g2.setRenderingHint(RenderingHints.KEY_ANTIALIASING, RenderingHints.VALUE_ANTIALIAS_ON);
        int w = getWidth(), h = getHeight();

        Color off = new Color(180, 180, 180), on = new Color(52, 199, 89);
        g2.setColor(blend(off, on, knobPos));
        g2.fillRoundRect(0, 0, w, h, h, h);

        int d = h - 6;
        int x = 3 + Math.round((w - d - 6) * knobPos);
        g2.setColor(Color.WHITE);
        g2.fillOval(x, 3, d, d);

        if (isFocusOwner()) {
            g2.setColor(new Color(0, 120, 215, 120));
            g2.setStroke(new BasicStroke(2f));
            g2.drawRoundRect(1, 1, w - 3, h - 3, h, h);
        }
        g2.dispose();
    }

    private static Color blend(Color a, Color b, float t) {
        return new Color(
                (int) (a.getRed()   + (b.getRed()   - a.getRed())   * t),
                (int) (a.getGreen() + (b.getGreen() - a.getGreen()) * t),
                (int) (a.getBlue()  + (b.getBlue()  - a.getBlue())  * t));
    }
}

// Usage
ToggleSwitch sw = new ToggleSwitch();
sw.addPropertyChangeListener("selected", e -> System.out.println("Now " + e.getNewValue()));
```

### 21.2 Checklist for a good custom component

- [ ] Overrides `getPreferredSize()` (and min/max if needed)
- [ ] Respects `isEnabled()` (draw greyed out)
- [ ] Handles focus and keyboard (key bindings), not just mouse
- [ ] Fires **bound properties** (`firePropertyChange`) or provides listener methods
- [ ] Uses `UIManager` colors/fonts where sensible
- [ ] Doesn't allocate large objects in `paintComponent`
- [ ] Takes `getInsets()` into account if it has a border
- [ ] Accessible: sets an accessible name (`getAccessibleContext().setAccessibleName(...)`)

### 21.3 Rounded button via subclass

```java
class RoundButton extends JButton {
    RoundButton(String text) {
        super(text);
        setContentAreaFilled(false);      // we paint the background ourselves
        setFocusPainted(false);
        setBorderPainted(false);
        setForeground(Color.WHITE);
        setCursor(Cursor.getPredefinedCursor(Cursor.HAND_CURSOR));
    }
    @Override protected void paintComponent(Graphics g) {
        Graphics2D g2 = (Graphics2D) g.create();
        g2.setRenderingHint(RenderingHints.KEY_ANTIALIASING, RenderingHints.VALUE_ANTIALIAS_ON);
        Color base = new Color(0, 120, 215);
        g2.setColor(getModel().isPressed() ? base.darker()
                  : getModel().isRollover() ? base.brighter() : base);
        g2.fillRoundRect(0, 0, getWidth(), getHeight(), 20, 20);
        g2.dispose();
        super.paintComponent(g);          // draws text/icon on top
    }
}
```

### 21.4 Composite components

Often you don't need to paint at all; compose existing ones:

```java
class LabeledField extends JPanel {
    private final JTextField field = new JTextField(15);
    LabeledField(String label) {
        super(new BorderLayout(6, 0));
        add(new JLabel(label), BorderLayout.WEST);
        add(field, BorderLayout.CENTER);
    }
    String getText() { return field.getText(); }
}
```

### 21.5 Overlay: the glass pane

Block interaction and show a "loading" overlay:

```java
JPanel overlay = new JPanel(new GridBagLayout()) {
    @Override protected void paintComponent(Graphics g) {
        g.setColor(new Color(0, 0, 0, 120));
        g.fillRect(0, 0, getWidth(), getHeight());
    }
};
overlay.setOpaque(false);
JProgressBar spinner = new JProgressBar();
spinner.setIndeterminate(true);
overlay.add(spinner);
overlay.addMouseListener(new MouseAdapter() { });        // swallow clicks
frame.setGlassPane(overlay);

overlay.setVisible(true);      // show
overlay.setVisible(false);     // hide
```

---

## 22. Undo / Redo

Swing text components support undo via `UndoManager`:

```java
import javax.swing.undo.*;

UndoManager undo = new UndoManager();
undo.setLimit(200);
textArea.getDocument().addUndoableEditListener(undo);

Action undoAction = new AbstractAction("Undo") {
    @Override public void actionPerformed(ActionEvent e) { if (undo.canUndo()) undo.undo(); }
};
Action redoAction = new AbstractAction("Redo") {
    @Override public void actionPerformed(ActionEvent e) { if (undo.canRedo()) undo.redo(); }
};

textArea.getInputMap().put(KeyStroke.getKeyStroke("control Z"), "undo");
textArea.getActionMap().put("undo", undoAction);
textArea.getInputMap().put(KeyStroke.getKeyStroke("control Y"), "redo");
textArea.getActionMap().put("redo", redoAction);
```

### Custom undo for your own model (Command pattern)

```java
class AddShapeEdit extends AbstractUndoableEdit {
    private final List<Shape> shapes; private final Shape shape;
    AddShapeEdit(List<Shape> shapes, Shape shape) { this.shapes = shapes; this.shape = shape; }

    @Override public void undo() throws CannotUndoException { super.undo(); shapes.remove(shape); }
    @Override public void redo() throws CannotRedoException { super.redo(); shapes.add(shape); }
    @Override public String getPresentationName() { return "Add shape"; }
}

// When the user adds a shape:
shapes.add(shape);
undoManager.addEdit(new AddShapeEdit(shapes, shape));
board.repaint();
```

Then keep the Undo/Redo menu items synced:

```java
undoItem.setEnabled(undoManager.canUndo());
undoItem.setText(undoManager.getUndoPresentationName());   // "Undo Add shape"
```

---

## 23. Drag & Drop and Clipboard

### 23.1 Clipboard

```java
import java.awt.datatransfer.*;

Clipboard cb = Toolkit.getDefaultToolkit().getSystemClipboard();

// Copy
cb.setContents(new StringSelection("Hello"), null);

// Paste
try {
    if (cb.isDataFlavorAvailable(DataFlavor.stringFlavor)) {
        String text = (String) cb.getData(DataFlavor.stringFlavor);
    }
} catch (Exception ex) { ex.printStackTrace(); }
```

### 23.2 Drop files onto a component (very common)

```java
import java.awt.datatransfer.DataFlavor;
import java.io.File;
import java.util.List;

JTextArea area = new JTextArea();
area.setTransferHandler(new TransferHandler() {
    @Override public boolean canImport(TransferSupport s) {
        return s.isDataFlavorSupported(DataFlavor.javaFileListFlavor);
    }
    @Override public boolean importData(TransferSupport s) {
        if (!canImport(s)) return false;
        try {
            @SuppressWarnings("unchecked")
            List<File> files = (List<File>) s.getTransferable().getTransferData(DataFlavor.javaFileListFlavor);
            for (File f : files) area.append(f.getAbsolutePath() + "\n");
            return true;
        } catch (Exception e) { return false; }
    }
});
```

### 23.3 Built-in drag support

Many components support text drag out of the box:

```java
textArea.setDragEnabled(true);
list.setDragEnabled(true);
list.setDropMode(DropMode.INSERT);
table.setDragEnabled(true);
```

### 23.4 Reordering a list by drag & drop

```java
class ListReorderHandler extends TransferHandler {
    private int fromIndex = -1;

    @Override public int getSourceActions(JComponent c) { return MOVE; }

    @Override protected Transferable createTransferable(JComponent c) {
        JList<?> list = (JList<?>) c;
        fromIndex = list.getSelectedIndex();
        return new StringSelection(String.valueOf(list.getSelectedValue()));
    }

    @Override public boolean canImport(TransferSupport s) {
        return s.isDrop() && s.isDataFlavorSupported(DataFlavor.stringFlavor);
    }

    @Override public boolean importData(TransferSupport s) {
        if (!canImport(s)) return false;
        JList.DropLocation dl = (JList.DropLocation) s.getDropLocation();
        @SuppressWarnings("unchecked")
        DefaultListModel<String> model = (DefaultListModel<String>) ((JList<String>) s.getComponent()).getModel();
        int to = dl.getIndex();
        try {
            String value = (String) s.getTransferable().getTransferData(DataFlavor.stringFlavor);
            if (to > fromIndex) to--;                 // adjust since we remove first
            model.remove(fromIndex);
            model.add(to, value);
            return true;
        } catch (Exception e) { return false; }
    }
}

list.setDragEnabled(true);
list.setDropMode(DropMode.INSERT);
list.setTransferHandler(new ListReorderHandler());
```

### 23.5 Dragging a component manually (free-form)

```java
class Draggable extends MouseAdapter {
    private Point offset;
    @Override public void mousePressed(MouseEvent e) { offset = e.getPoint(); }
    @Override public void mouseDragged(MouseEvent e) {
        Component c = e.getComponent();
        Point p = SwingUtilities.convertPoint(c, e.getPoint(), c.getParent());
        c.setLocation(p.x - offset.x, p.y - offset.y);   // parent must use a null layout
    }
}
// JLabel tile = ...; Draggable d = new Draggable(); tile.addMouseListener(d); tile.addMouseMotionListener(d);
```


---

## 24. Advanced Layouts

### 24.1 GridBagLayout: the powerful one

`GridBagLayout` places components in a grid where cells can span multiple rows/columns and have different sizes. It's verbose but flexible. Configure each component with `GridBagConstraints`.

| Field | Meaning |
|---|---|
| `gridx`, `gridy` | Cell column/row (0-based) |
| `gridwidth`, `gridheight` | Cells spanned (`REMAINDER` = to the end of row/column) |
| `weightx`, `weighty` | Share of **extra** space (0 = don't grow) |
| `fill` | `NONE`, `HORIZONTAL`, `VERTICAL`, `BOTH` |
| `anchor` | Where to put the component inside a larger cell (`WEST`, `NORTHEAST`, ...) |
| `insets` | External padding `new Insets(top, left, bottom, right)` |
| `ipadx`, `ipady` | Internal padding |

A realistic form:

```java
import javax.swing.*;
import java.awt.*;

public class GridBagForm extends JPanel {
    public GridBagForm() {
        super(new GridBagLayout());
        setBorder(BorderFactory.createEmptyBorder(12, 12, 12, 12));

        GridBagConstraints c = new GridBagConstraints();
        c.insets = new Insets(4, 4, 4, 4);

        // Row 0: label + text field
        c.gridx = 0; c.gridy = 0; c.anchor = GridBagConstraints.LINE_END; c.weightx = 0;
        add(new JLabel("Name:"), c);
        c.gridx = 1; c.anchor = GridBagConstraints.LINE_START; c.fill = GridBagConstraints.HORIZONTAL; c.weightx = 1;
        add(new JTextField(20), c);

        // Row 1
        c.gridx = 0; c.gridy = 1; c.fill = GridBagConstraints.NONE; c.anchor = GridBagConstraints.LINE_END; c.weightx = 0;
        add(new JLabel("Email:"), c);
        c.gridx = 1; c.fill = GridBagConstraints.HORIZONTAL; c.anchor = GridBagConstraints.LINE_START; c.weightx = 1;
        add(new JTextField(20), c);

        // Row 2: notes label anchored to the top
        c.gridx = 0; c.gridy = 2; c.fill = GridBagConstraints.NONE; c.anchor = GridBagConstraints.FIRST_LINE_END; c.weightx = 0;
        add(new JLabel("Notes:"), c);
        // Text area grows in both directions
        c.gridx = 1; c.fill = GridBagConstraints.BOTH; c.weightx = 1; c.weighty = 1; c.anchor = GridBagConstraints.LINE_START;
        add(new JScrollPane(new JTextArea(5, 20)), c);

        // Row 3: buttons spanning both columns, right aligned
        c.gridx = 0; c.gridy = 3; c.gridwidth = 2; c.fill = GridBagConstraints.NONE; c.weighty = 0;
        c.anchor = GridBagConstraints.LINE_END;
        JPanel buttons = new JPanel(new FlowLayout(FlowLayout.RIGHT, 6, 0));
        buttons.add(new JButton("Cancel"));
        buttons.add(new JButton("Save"));
        add(buttons, c);
    }
}
```

A helper to cut the boilerplate:

```java
static GridBagConstraints gbc(int x, int y, double wx, double wy, int fill) {
    GridBagConstraints c = new GridBagConstraints();
    c.gridx = x; c.gridy = y; c.weightx = wx; c.weighty = wy; c.fill = fill;
    c.insets = new Insets(4, 4, 4, 4);
    c.anchor = GridBagConstraints.WEST;
    return c;
}
panel.add(new JLabel("Name:"), gbc(0, 0, 0, 0, GridBagConstraints.NONE));
panel.add(nameField,           gbc(1, 0, 1, 0, GridBagConstraints.HORIZONTAL));
```

### 24.2 GroupLayout

Used heavily by NetBeans' Matisse GUI builder. You define horizontal and vertical groups independently (sequential vs parallel).

```java
GroupLayout layout = new GroupLayout(panel);
panel.setLayout(layout);
layout.setAutoCreateGaps(true);
layout.setAutoCreateContainerGaps(true);

JLabel l1 = new JLabel("Name:"), l2 = new JLabel("Email:");
JTextField f1 = new JTextField(), f2 = new JTextField();

layout.setHorizontalGroup(layout.createSequentialGroup()
    .addGroup(layout.createParallelGroup(GroupLayout.Alignment.TRAILING).addComponent(l1).addComponent(l2))
    .addGroup(layout.createParallelGroup().addComponent(f1).addComponent(f2)));

layout.setVerticalGroup(layout.createSequentialGroup()
    .addGroup(layout.createParallelGroup(GroupLayout.Alignment.BASELINE).addComponent(l1).addComponent(f1))
    .addGroup(layout.createParallelGroup(GroupLayout.Alignment.BASELINE).addComponent(l2).addComponent(f2)));
```

Verbose to write by hand; handy to read when you inherit NetBeans code.

### 24.3 MigLayout (third-party, very productive)

```xml
<dependency>
    <groupId>com.miglayout</groupId>
    <artifactId>miglayout-swing</artifactId>
    <version>11.4.2</version>
</dependency>
```

```java
import net.miginfocom.swing.MigLayout;

JPanel p = new JPanel(new MigLayout("fillx, insets 12", "[right][grow, fill]", "[]8[]8[grow, fill]"));
p.add(new JLabel("Name:"));
p.add(new JTextField(), "wrap");                 // wrap = next row
p.add(new JLabel("Email:"));
p.add(new JTextField(), "wrap");
p.add(new JLabel("Notes:"), "top");
p.add(new JScrollPane(new JTextArea()), "wrap, push");
p.add(new JButton("Save"), "span 2, split 2, align right");
p.add(new JButton("Cancel"));
```

Compare this to the ~25-line GridBagLayout version. Other options: **JGoodies FormLayout**, **DesignGrid**.

### 24.4 Responsive tips

- Use `JScrollPane` for content that may overflow.
- Use `weightx/weighty` (GridBag) or `grow/push` (Mig) for areas that should expand.
- Use `Box.createGlue()`/`createHorizontalGlue()` to push toolbar items to the edge.
- Let components pick their own size; avoid `setPreferredSize` except on custom drawing areas.
- Set `setMinimumSize` on the frame so users can't shrink it into nonsense: `frame.setMinimumSize(new Dimension(500, 400));`

### 24.5 Writing a custom LayoutManager

```java
class WrapLayout implements LayoutManager {   // stacks components vertically with fixed gap
    private final int gap;
    WrapLayout(int gap) { this.gap = gap; }
    @Override public void addLayoutComponent(String name, Component comp) { }
    @Override public void removeLayoutComponent(Component comp) { }

    @Override public Dimension preferredLayoutSize(Container parent) {
        Insets in = parent.getInsets();
        int w = 0, h = in.top + in.bottom;
        for (Component c : parent.getComponents()) {
            if (!c.isVisible()) continue;
            Dimension d = c.getPreferredSize();
            w = Math.max(w, d.width);
            h += d.height + gap;
        }
        return new Dimension(w + in.left + in.right, h);
    }
    @Override public Dimension minimumLayoutSize(Container parent) { return preferredLayoutSize(parent); }

    @Override public void layoutContainer(Container parent) {
        Insets in = parent.getInsets();
        int y = in.top;
        int width = parent.getWidth() - in.left - in.right;
        for (Component c : parent.getComponents()) {
            if (!c.isVisible()) continue;
            int h = c.getPreferredSize().height;
            c.setBounds(in.left, y, width, h);
            y += h + gap;
        }
    }
}
```

---

## 25. Internal Frames & Layered Panes

### 25.1 MDI with JDesktopPane

```java
JDesktopPane desktop = new JDesktopPane();
desktop.setBackground(Color.GRAY);

JInternalFrame doc = new JInternalFrame("Document 1", true, true, true, true);
// resizable, closable, maximizable, iconifiable
doc.add(new JScrollPane(new JTextArea()));
doc.setSize(300, 200);
doc.setLocation(30, 30);
doc.setVisible(true);
desktop.add(doc);
try { doc.setSelected(true); } catch (java.beans.PropertyVetoException ignore) { }

frame.setContentPane(desktop);
```

Useful when users need many documents visible inside one window. In modern apps, tabs are usually preferred.

### 25.2 JLayeredPane: stacking components

```java
JLayeredPane layers = new JLayeredPane();
layers.setPreferredSize(new Dimension(300, 300));

JPanel bottom = new JPanel(); bottom.setBackground(Color.BLUE);  bottom.setBounds(20, 20, 150, 150);
JPanel middle = new JPanel(); middle.setBackground(Color.GREEN); middle.setBounds(60, 60, 150, 150);
JPanel top    = new JPanel(); top.setBackground(Color.RED);      top.setBounds(100, 100, 150, 150);

layers.add(bottom, JLayeredPane.DEFAULT_LAYER);
layers.add(middle, JLayeredPane.PALETTE_LAYER);
layers.add(top,    JLayeredPane.MODAL_LAYER);
layers.moveToFront(bottom);
```

### 25.3 Tooltips & custom popups (Popup / JWindow)

```java
// Custom tooltip content
JButton b = new JButton("Hover me") {
    @Override public JToolTip createToolTip() {
        JToolTip tip = super.createToolTip();
        tip.setBackground(Color.YELLOW);
        return tip;
    }
};
b.setToolTipText("<html><b>Rich</b> tooltip<br>with HTML</html>");
ToolTipManager.sharedInstance().setInitialDelay(300);

// Auto-complete style popup
Popup popup = PopupFactory.getSharedInstance().getPopup(owner, content, screenX, screenY);
popup.show();
// popup.hide();
```

---

## 26. Internationalization & Preferences

### 26.1 ResourceBundle

`src/main/resources/messages.properties`:

```properties
app.title=My Application
menu.file=File
button.save=Save
greeting=Hello, {0}!
```

`messages_fr.properties`:

```properties
app.title=Mon application
menu.file=Fichier
button.save=Enregistrer
greeting=Bonjour, {0} !
```

```java
import java.util.*;
import java.text.MessageFormat;

ResourceBundle msgs = ResourceBundle.getBundle("messages", Locale.getDefault());
JFrame frame = new JFrame(msgs.getString("app.title"));
JButton save = new JButton(msgs.getString("button.save"));
String hello = MessageFormat.format(msgs.getString("greeting"), userName);
```

Tips: never concatenate translated strings; use `MessageFormat`. Use `NumberFormat`/`DateTimeFormatter` with a locale. Use `BorderLayout.LINE_START` etc. and `applyComponentOrientation(ComponentOrientation.getOrientation(locale))` for right-to-left support. Use UTF-8 `.properties` files on Java 9+.

### 26.2 Persisting settings with Preferences

```java
import java.util.prefs.Preferences;

Preferences prefs = Preferences.userNodeForPackage(MyApp.class);

// Restore window state on start
frame.setBounds(prefs.getInt("x", 100), prefs.getInt("y", 100),
                prefs.getInt("w", 800),  prefs.getInt("h", 600));
String lastDir = prefs.get("lastDir", System.getProperty("user.home"));

// Save on close
frame.addWindowListener(new WindowAdapter() {
    @Override public void windowClosing(WindowEvent e) {
        Rectangle b = frame.getBounds();
        prefs.putInt("x", b.x); prefs.putInt("y", b.y);
        prefs.putInt("w", b.width); prefs.putInt("h", b.height);
    }
});
```

For richer config use a JSON/properties file in `~/.myapp/` (`System.getProperty("user.home")`).

---

## 27. High-DPI, Accessibility & Performance

### 27.1 High-DPI

Modern JDKs (9+) scale Swing automatically on HiDPI displays. To keep things crisp:

- Use vector icons (SVG via FlatLaf's `FlatSVGIcon`) or provide `@2x` images.
- Size things relative to the font: `int unit = getFontMetrics(getFont()).getHeight();`
- Never hard-code pixel layouts; let layout managers adapt.

FlatLaf SVG icons:

```java
// com.formdev:flatlaf-extras
import com.formdev.flatlaf.extras.FlatSVGIcon;
button.setIcon(new FlatSVGIcon("icons/save.svg", 16, 16));
```

Multi-resolution icons in JDK:

```java
java.awt.image.BaseMultiResolutionImage mr = new java.awt.image.BaseMultiResolutionImage(img1x, img2x);
label.setIcon(new ImageIcon(mr));
```

### 27.2 Accessibility

Screen readers use the Java Accessibility API.

```java
button.getAccessibleContext().setAccessibleName("Save document");
button.getAccessibleContext().setAccessibleDescription("Saves the current document to disk");
label.setLabelFor(textField);        // associates label & field
button.setMnemonic('S');             // keyboard access
```

Checklist:
- Everything reachable by keyboard (Tab order sensible, mnemonics, shortcuts)
- Visible focus indicators
- Don't communicate by color alone
- Sufficient contrast
- Tooltips for icon-only buttons
- Let font sizes scale

Custom focus traversal order:

```java
frame.setFocusTraversalPolicy(new FocusTraversalPolicy() { /* implement 5 methods */ });
// or simpler: setFocusable(false) on things that shouldn't take focus
```

### 27.3 Performance

| Problem | Fix |
|---|---|
| UI freezes | Move work off EDT (`SwingWorker`) |
| Slow big tables/lists | Use a custom model that computes lazily; `setPrototypeCellValue`; fixed row height; avoid heavy renderers |
| Slow renderers | Renderer is called many times per second; no allocations, no I/O, no `new ImageIcon(...)`; cache icons |
| Flickering when resizing | Avoid `setOpaque(false)` unless needed; draw in `paintComponent`, not `paint` |
| Many repaints | Repaint only dirty regions: `repaint(x, y, w, h)` |
| Many updates in a row | Batch updates, then fire a single `fireTableDataChanged()` |
| Too many components | Draw items yourself (custom painting) instead of creating 10,000 `JLabel`s |
| Huge text | Use `Document` operations in bulk; disable undo during load; load in chunks |
| Startup time | Show a splash, create heavy panels lazily (create tab contents on first select) |

Bulk-adding rows efficiently:

```java
// Bad: fires an event per row
for (Employee e : list) model.add(e);

// Good: one event
public void addAll(List<Employee> list) {
    int first = rows.size();
    rows.addAll(list);
    fireTableRowsInserted(first, rows.size() - 1);
}
```

Detect EDT violations / slow EDT while developing: use a tool like the [FEST/AssertJ Swing `FailOnThreadViolationRepaintManager`](https://assertj.github.io/doc/) or log long event handlers yourself:

```java
long t = System.nanoTime();
// ... handler body ...
long ms = (System.nanoTime() - t) / 1_000_000;
if (ms > 50) System.err.println("Slow EDT handler: " + ms + " ms");
```

### 27.4 Memory leaks

- Remove listeners from long-lived models when a view is disposed (`model.removePropertyChangeListener(...)`).
- `dispose()` dialogs and frames you no longer need.
- Stop `Timer`s when their component is removed (`removeNotify()` is a good hook).

```java
@Override public void removeNotify() {
    timer.stop();
    super.removeNotify();
}
```

---

## 28. Testing Swing Apps

### 28.1 Test the model and logic (easy, high value)

If you've separated the model/presenter (Section 20), most of your logic is plain JUnit:

```java
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;

class CounterModelTest {
    @Test void incrementFiresEvent() {
        CounterModel m = new CounterModel();
        int[] seen = {-1};
        m.addPropertyChangeListener(e -> seen[0] = (Integer) e.getNewValue());
        m.increment();
        assertEquals(1, m.getValue());
        assertEquals(1, seen[0]);
    }
}
```

### 28.2 Test the UI with AssertJ Swing

```xml
<dependency>
    <groupId>org.assertj</groupId>
    <artifactId>assertj-swing-junit</artifactId>
    <version>3.17.1</version>
    <scope>test</scope>
</dependency>
```

```java
import org.assertj.swing.edt.GuiActionRunner;
import org.assertj.swing.fixture.FrameFixture;
import org.junit.jupiter.api.*;

class LoginFormTest {
    private FrameFixture window;

    @BeforeEach void setUp() {
        JFrame frame = GuiActionRunner.execute(() -> {
            JFrame f = new JFrame();
            f.setContentPane(new LoginForm());
            f.pack();
            return f;
        });
        window = new FrameFixture(frame);
        window.show();
    }
    @AfterEach void tearDown() { window.cleanUp(); }

    @Test void buttonsExist() {
        window.button(org.assertj.swing.core.matcher.JButtonMatcher.withText("Login")).requireEnabled();
    }
}
```

Tip: give components names to find them reliably: `usernameField.setName("username");` then `window.textBox("username").enterText("alice");`

Headless caveat: Swing UI tests need a display (use `xvfb-run` on Linux CI).

---

## 29. Packaging & Distribution

### 29.1 Project structure (Maven)

```
myapp/
 ├─ pom.xml
 └─ src/
     ├─ main/
     │   ├─ java/com/example/myapp/Main.java
     │   └─ resources/images/save.png
     └─ test/java/...
```

### 29.2 Runnable JAR with Maven

```xml
<build>
  <plugins>
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-jar-plugin</artifactId>
      <version>3.4.1</version>
      <configuration>
        <archive>
          <manifest>
            <mainClass>com.example.myapp.Main</mainClass>
          </manifest>
        </archive>
      </configuration>
    </plugin>
    <!-- bundle dependencies (FlatLaf, etc.) into one "fat" jar -->
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-shade-plugin</artifactId>
      <version>3.6.0</version>
      <executions>
        <execution><phase>package</phase><goals><goal>shade</goal></goals></execution>
      </executions>
    </plugin>
  </plugins>
</build>
```

```bash
mvn clean package
java -jar target/myapp-1.0.jar
```

### 29.3 Without a build tool

```bash
javac -d out src/com/example/myapp/*.java
echo "Main-Class: com.example.myapp.Main" > manifest.txt
jar cfm myapp.jar manifest.txt -C out .
java -jar myapp.jar
```

(Copy resources into `out/` too, or they won't be inside the jar.)

### 29.4 Native installers with jpackage (JDK 14+)

```bash
jpackage \
  --name MyApp \
  --input target \
  --main-jar myapp-1.0.jar \
  --main-class com.example.myapp.Main \
  --type app-image          # or: exe / msi (Windows), dmg / pkg (macOS), deb / rpm (Linux)
  --icon icon.ico \
  --java-options "-Xmx512m"
```

This bundles a private JRE so users don't need Java installed. You must build on each target OS.

### 29.5 Useful JVM flags

```bash
-Dswing.aatext=true                  # antialiased text on older setups
-Dsun.java2d.uiScale=1.5             # force a UI scale factor
-Dapple.awt.application.name=MyApp   # macOS dock/menu name
-Xdock:name=MyApp                    # older macOS
```

---

# Part 4: Projects & Reference

## 30. Capstone Projects

Work through these in order. Each pulls in a group of skills. Try building each **without** looking at hints first.

### Project 1: Calculator (Basics)
**Skills:** `GridLayout`, `BorderLayout`, `ActionListener`, shared listeners.
Features: digits, `+ - × ÷`, `=`, `C`, decimal point, keyboard input via key bindings, error handling for divide-by-zero.

Starter skeleton:

```java
import javax.swing.*;
import java.awt.*;

public class Calculator extends JFrame {
    private final JTextField display = new JTextField("0");
    private double acc = 0;
    private String op = null;
    private boolean startNew = true;

    public Calculator() {
        super("Calculator");
        setDefaultCloseOperation(EXIT_ON_CLOSE);
        display.setEditable(false);
        display.setHorizontalAlignment(JTextField.RIGHT);
        display.setFont(new Font(Font.MONOSPACED, Font.BOLD, 28));
        add(display, BorderLayout.NORTH);

        String[] keys = {"7","8","9","/", "4","5","6","*", "1","2","3","-", "0",".","=","+"};
        JPanel pad = new JPanel(new GridLayout(4, 4, 4, 4));
        pad.setBorder(BorderFactory.createEmptyBorder(6, 6, 6, 6));
        for (String k : keys) {
            JButton b = new JButton(k);
            b.setFont(b.getFont().deriveFont(18f));
            b.addActionListener(e -> press(k));
            pad.add(b);
        }
        add(pad, BorderLayout.CENTER);

        JButton clear = new JButton("C");
        clear.addActionListener(e -> { acc = 0; op = null; startNew = true; display.setText("0"); });
        add(clear, BorderLayout.SOUTH);

        pack();
        setLocationRelativeTo(null);
    }

    private void press(String k) {
        if (k.matches("[0-9.]")) {
            display.setText(startNew ? k : display.getText() + k);
            startNew = false;
        } else {
            double cur = Double.parseDouble(display.getText());
            if (op != null) cur = apply(acc, cur, op);
            display.setText(format(cur));
            acc = cur;
            op = k.equals("=") ? null : k;
            startNew = true;
        }
    }

    private double apply(double a, double b, String op) {
        return switch (op) {
            case "+" -> a + b;
            case "-" -> a - b;
            case "*" -> a * b;
            case "/" -> b == 0 ? Double.NaN : a / b;
            default -> b;
        };
    }

    private String format(double d) {
        if (Double.isNaN(d)) return "Error";
        return d == Math.rint(d) ? String.valueOf((long) d) : String.valueOf(d);
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> new Calculator().setVisible(true));
    }
}
```
Extensions: operator precedence (use the shunting-yard algorithm), history panel, memory keys, a scientific mode in a `CardLayout`.

### Project 2: To-Do List (Intermediate)
**Skills:** `JList` + `DefaultListModel`, custom renderer (checkbox), dialogs, `JFileChooser`, persistence.
Features: add/edit/delete tasks, mark done (strike-through), filter (All / Active / Done), drag to reorder, save/load to a JSON or text file, remember window size via `Preferences`.

### Project 3: Contact Manager (Tables, MVC)
**Skills:** `AbstractTableModel`, `TableRowSorter`, search, form dialog, validation, MVC separation.
Features: CRUD on contacts, live search filter, validated edit dialog, export CSV, undo delete, SQLite persistence via JDBC (loaded in a `SwingWorker`).

### Project 4: Text/Code Editor (Text components, Actions)
**Skills:** `JTextPane`, `DocumentListener`, `UndoManager`, `Action`s, tabs, find & replace, `JFileChooser`.
Features: multiple files in `JTabbedPane` (with close buttons), line numbers (a `JPanel` set as the scroll pane's row header), find/replace dialog with highlighting (`Highlighter`), syntax highlighting, dirty-flag title (`*`), recent files menu.

Line-number gutter hint:

```java
class LineNumbers extends JComponent {
    private final JTextComponent text;
    LineNumbers(JTextComponent text) {
        this.text = text;
        text.getDocument().addDocumentListener(new SimpleDocumentListener(this::repaint));
        text.addCaretListener(e -> repaint());
    }
    @Override public Dimension getPreferredSize() { return new Dimension(40, text.getHeight()); }
    @Override protected void paintComponent(Graphics g) {
        g.setColor(new Color(240, 240, 240)); g.fillRect(0, 0, getWidth(), getHeight());
        g.setColor(Color.GRAY); g.setFont(text.getFont());
        FontMetrics fm = g.getFontMetrics();
        int lineHeight = fm.getHeight();
        int lines = text.getDocument().getDefaultRootElement().getElementCount();
        for (int i = 1; i <= lines; i++) g.drawString(String.valueOf(i), 4, i * lineHeight - fm.getDescent());
    }
}
scrollPane.setRowHeaderView(new LineNumbers(textPane));
```

### Project 5: Snake / Pong / Tetris (Painting, Timers, Key Bindings)
**Skills:** `paintComponent`, `Timer`, key bindings, game loop, collision detection.
Features: score, pause, speed increase, high scores saved to `Preferences`. Keep game state in a pure-Java model class that is testable without Swing.

### Project 6: Image Viewer / Editor (Images, SwingWorker)
**Skills:** `ImageIO`, `BufferedImage`, scaling, `JScrollPane`, background loading, `BufferedImageOp`.
Features: open folder, thumbnails grid loaded in the background, zoom (`Graphics2D.scale`), rotate/flip, grayscale/blur filters, save as, drag-and-drop to open.

Grayscale filter:

```java
static BufferedImage gray(BufferedImage src) {
    BufferedImage out = new BufferedImage(src.getWidth(), src.getHeight(), BufferedImage.TYPE_BYTE_GRAY);
    Graphics2D g = out.createGraphics();
    g.drawImage(src, 0, 0, null);
    g.dispose();
    return out;
}
```

### Project 7: Dashboard / Chart App (Custom components, Animation)
**Skills:** custom painting, timers, `SwingWorker`, layout composition.
Features: live-updating line chart drawn by hand, gauge component, bar chart, tooltips on hover showing nearest data point, resizable layout, dark/light theme toggle (FlatLaf).

### Project 8 (Capstone): Full Desktop Application
Pick one: **Note-taking app** (tree of notebooks + rich text), **Expense tracker** (tables + charts + DB), **Chat client** (sockets + `SwingWorker` + `JTextPane` bubbles), or **File manager** (lazy `JTree` + `JTable` + drag & drop).
Requirements: MVC/MVP structure, unit-tested model, background I/O, FlatLaf theme, key bindings & actions, preferences, packaged with `jpackage`.

---

## 31. Common Pitfalls

| Pitfall | Why it happens | Fix |
|---|---|---|
| Window shows nothing / blank | Called `setVisible(true)` before adding components, or forgot `revalidate()/repaint()` after dynamic changes | Add components first; after dynamic changes call `revalidate(); repaint();` |
| UI freezes | Long task on the EDT | `SwingWorker` / `CompletableFuture` |
| Random exceptions/odd rendering | Touching components from non-EDT threads | `SwingUtilities.invokeLater` |
| `JTextArea` doesn't scroll | Not inside a `JScrollPane` | Wrap it |
| Table headers missing | `JTable` not inside a `JScrollPane` | Wrap it |
| Component not visible in BorderLayout | Two components added to the same region (second replaces first) | Wrap them in a `JPanel` |
| Component too big/small | Calling `setSize` under a layout manager | Control via preferred size, constraints, or layout choice |
| Changes to model don't show | Modified backing list without `fireXxx` | Fire the appropriate event; or use `Default*Model` |
| Wrong row after sorting | Using view index as model index | `convertRowIndexToModel` |
| Custom drawing leaves artifacts | Forgot `super.paintComponent(g)` | Call it first |
| Painting state leaks into other components | Changed the passed-in `Graphics` (color, transform) | `Graphics2D g2 = (Graphics2D) g.create(); ... g2.dispose();` |
| Images not found after packaging | Loaded with file paths | `getResource("/images/x.png")` |
| `setBackground` has no effect on JLabel | Labels are non-opaque | `label.setOpaque(true)` |
| Key shortcuts don't work | `KeyListener` + no focus | Use Key Bindings with `WHEN_IN_FOCUSED_WINDOW` |
| Dialog appears behind / in wrong place | Parent `null` | Pass the owner frame |
| `IllegalStateException: Attempt to mutate in notification` | Changing the document inside a `DocumentListener` | `SwingUtilities.invokeLater` or use a `DocumentFilter` |
| Look & Feel not applied | Set after components were created | Set first, or `SwingUtilities.updateComponentTreeUI` |
| JVM doesn't exit | Non-daemon threads/timers running or `HIDE_ON_CLOSE` default | Stop threads; use `EXIT_ON_CLOSE` / `dispose()` correctly |
| Mixing AWT heavyweight and Swing lightweight | Overlap order problems | Use Swing components only |
| `ActionListener` fires twice | Added the same listener twice (e.g. in a method called repeatedly) | Add listeners once in the constructor |
| Focus problems | `requestFocus()` before the window is visible | `requestFocusInWindow()` after `setVisible(true)` or use `invokeLater` |
| Modal dialog blocks code unexpectedly | `setVisible(true)` on modal dialogs blocks until closed | Expected: read results after the call returns |

---

## 32. Cheat Sheet

### Skeleton of every Swing app

```java
public class App {
    public static void main(String[] args) {
        // 1. (optional) Look & Feel
        // 2. Start on the EDT
        SwingUtilities.invokeLater(() -> {
            JFrame f = new JFrame("Title");
            f.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            f.setContentPane(new MainPanel());
            f.setJMenuBar(buildMenuBar());
            f.pack();
            f.setLocationRelativeTo(null);
            f.setVisible(true);
        });
    }
}
```

### Which component?

| I need... | Use |
|---|---|
| Static text/image | `JLabel` |
| Click action | `JButton` |
| On/off | `JCheckBox`, `JToggleButton` |
| One of few | `JRadioButton` + `ButtonGroup` |
| One of many | `JComboBox`, `JList` |
| Short text input | `JTextField` |
| Number input | `JSpinner`, `JFormattedTextField`, `JSlider` |
| Long text | `JTextArea` in `JScrollPane` |
| Styled text | `JTextPane` |
| Tabular data | `JTable` |
| Hierarchical data | `JTree` |
| Progress | `JProgressBar` |
| File selection | `JFileChooser` |
| Colour selection | `JColorChooser` |
| Simple message/confirm | `JOptionPane` |
| Resizable split | `JSplitPane` |
| Multi-page | `JTabbedPane` / `CardLayout` |
| Custom drawing | `JPanel` + `paintComponent` |

### Which layout?

| I need... | Use |
|---|---|
| Window skeleton | `BorderLayout` |
| Row of buttons | `FlowLayout` (right-aligned for dialogs) |
| Uniform grid | `GridLayout` |
| Vertical stack | `BoxLayout(Y_AXIS)` |
| Form (label/field rows) | `GridBagLayout` or MigLayout |
| Screen switcher | `CardLayout` |

### Which listener?

| Event | Listener / API |
|---|---|
| Button / menu / Enter | `ActionListener` |
| Mouse | `MouseAdapter` |
| Key shortcuts | **Key Bindings** (InputMap/ActionMap) |
| Text changes | `DocumentListener` |
| Slider/spinner/tab | `ChangeListener` |
| List/table selection | `ListSelectionListener` |
| Tree selection | `TreeSelectionListener` |
| Window close | `WindowListener` |
| Property change | `PropertyChangeListener` |
| Resize | `ComponentListener` |

### Threading in 4 lines

```java
SwingUtilities.invokeLater(() -> updateUi());            // from any thread -> EDT
new SwingWorker<R, V>() { /* background + done() */ }.execute();   // long tasks
new Timer(delay, e -> tick()).start();                   // periodic UI updates (EDT)
if (SwingUtilities.isEventDispatchThread()) { ... }      // check
```

### Painting in 5 lines

```java
@Override protected void paintComponent(Graphics g) {
    super.paintComponent(g);
    Graphics2D g2 = (Graphics2D) g.create();
    g2.setRenderingHint(RenderingHints.KEY_ANTIALIASING, RenderingHints.VALUE_ANTIALIAS_ON);
    /* draw */  g2.dispose();
}
```

### After dynamic changes

```java
container.add(x); container.remove(y);
container.revalidate();   // recompute layout
container.repaint();      // redraw
```

---

## 33. Where to Go Next

### Learning path summary

| Stage | You can... |
|---|---|
| Basics (1-8) | Build forms, dialogs, menus, simple tools |
| Intermediate (9-16) | Build data-driven apps with tables, trees, validation, themes |
| Advanced (17-29) | Draw custom UIs, animate, use threads safely, structure, test and ship apps |
| Projects (30) | Prove it to yourself |

### Recommended resources

- **Oracle Java Tutorials: Creating a GUI with Swing** (the official, free, very complete reference) - `docs.oracle.com/javase/tutorial/uiswing/`
- **Javadoc** for `javax.swing`: always the final authority.
- **FlatLaf docs and demo** (modern look, theming, extras) - `formdev.com/flatlaf`
- **MigLayout quick start** - `miglayout.com`
- **Book: "Core Java, Volume I"** (Horstmann), with solid Swing chapters.
- **Book: "Filthy Rich Clients"** (Haase & Guy) for advanced effects and animation.
- **Book: "Swing Hacks"** for tricks and techniques.
- Read the source of open-source Swing apps: IntelliJ Community Edition, jEdit, DBeaver (SWT), JabRef (JavaFX now), Apache JMeter.

### Libraries worth knowing

| Library | Use |
|---|---|
| FlatLaf | Modern L&F and themes |
| MigLayout | Productive layouts |
| SwingX / `swingx` | Extra components (date picker, task pane, search field) |
| JFreeChart | Charts |
| RSyntaxTextArea | Code editor component with syntax highlighting |
| Jackson / Gson | JSON for persistence |
| AssertJ Swing | UI tests |
| Apache Batik / JSVG | SVG rendering |
| LGoodDatePicker | Date/time pickers |

### Keep growing

1. Build something *you* want to use.
2. Read other people's Swing code and refactor it toward MVC.
3. Profile and improve a slow screen.
4. Explore JavaFX afterwards; many concepts (properties, layout, event threads) transfer directly.

Happy coding!
