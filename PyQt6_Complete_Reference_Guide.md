# PyQt6 Complete Reference Guide

A detailed, example-driven walkthrough of the PyQt6 library, organized as both a **tutorial** (read top to bottom to learn) and a **reference** (jump to any section to look something up).

---

## Table of Contents

1. [Introduction & Installation](#1-introduction--installation)
2. [Core Architecture](#2-core-architecture)
3. [Signals and Slots](#3-signals-and-slots)
4. [QtCore Module](#4-qtcore-module)
5. [QtWidgets Module — Windows & Containers](#5-qtwidgets-module--windows--containers)
6. [Layouts](#6-layouts)
7. [Common Widgets Reference](#7-common-widgets-reference)
8. [Dialogs](#8-dialogs)
9. [Menus, Toolbars, Actions & Status Bar](#9-menus-toolbars-actions--status-bar)
10. [Model/View Architecture](#10-modelview-architecture)
11. [QtGui Module](#11-qtgui-module)
12. [Painting & Custom Widgets](#12-painting--custom-widgets)
13. [Event Handling](#13-event-handling)
14. [Threading & Concurrency](#14-threading--concurrency)
15. [Styling (Qt Style Sheets)](#15-styling-qt-style-sheets)
16. [Networking (QtNetwork)](#16-networking-qtnetwork)
17. [Files, Settings & Resources](#17-files-settings--resources)
18. [Packaging & Deployment](#18-packaging--deployment)
19. [Quick Reference Tables](#19-quick-reference-tables)

---

## 1. Introduction & Installation

**PyQt6** is a set of Python bindings for **Qt6**, the C++ cross-platform application framework maintained by The Qt Company. It lets you build native-looking desktop GUIs (Windows, macOS, Linux) as well as some mobile/embedded targets, using Python instead of C++.

PyQt6 is developed by **Riverbank Computing** and is dual-licensed (GPL v3 / commercial). The alternative binding, **PySide6**, is Qt's own official binding (LGPL) and is API-compatible for ~95% of use cases — most code in this guide works with only import-line changes.

### Installation

```bash
pip install PyQt6
pip install PyQt6-tools        # Qt Designer and other dev tools (optional)
pip install PyQt6-Charts       # optional: charting widgets
pip install PyQt6-WebEngine    # optional: embedded Chromium browser widget
```

### Minimal application skeleton

```python
import sys
from PyQt6.QtWidgets import QApplication, QWidget

app = QWidget()  # placeholder, see below for a real widget
```

A real minimal app:

```python
import sys
from PyQt6.QtWidgets import QApplication, QMainWindow, QLabel

class MainWindow(QMainWindow):
    def __init__(self):
        super().__init__()
        self.setWindowTitle("My App")
        self.setCentralWidget(QLabel("Hello, PyQt6!"))

app = QApplication(sys.argv)
window = MainWindow()
window.show()
sys.exit(app.exec())
```

Key points from this skeleton:
- Exactly **one** `QApplication` must exist per process, created before any widgets.
- `sys.argv` is passed so Qt can parse its own command-line arguments (e.g. `-style`, `-platform`).
- `app.exec()` starts the **event loop** (note: `exec()`, not `exec_()` — the trailing underscore was needed in PyQt5 for Python 2 compatibility and has been dropped in PyQt6).
- `sys.exit(...)` ensures the process exit code from Qt propagates to the OS.

### PyQt6 module layout

PyQt6 splits Qt's functionality into submodules, each imported separately:

| Module | Contents |
|---|---|
| `PyQt6.QtCore` | Non-GUI core classes: event loop, signals/slots, threading, files, timers, data structures |
| `PyQt6.QtWidgets` | Classic widget-based UI: buttons, layouts, windows, dialogs |
| `PyQt6.QtGui` | Painting, fonts, colors, images, icons, clipboard, drag-and-drop, window/surface classes |
| `PyQt6.QtNetwork` | Sockets, HTTP requests |
| `PyQt6.QtSql` | Database access (SQLite, MySQL, PostgreSQL, ODBC...) |
| `PyQt6.QtMultimedia` | Audio/video playback and capture |
| `PyQt6.QtWebEngineWidgets` | Embedded Chromium browser widget |
| `PyQt6.QtCharts` | Charting widgets (separate install) |
| `PyQt6.QtSvg` | SVG rendering |
| `PyQt6.QtPrintSupport` | Printing and print preview |
| `PyQt6.QtTest` | Unit-testing helpers for GUI apps |
| `PyQt6.uic` | Loading `.ui` files produced by Qt Designer |

---

## 2. Core Architecture

### The event loop

Qt applications are **event-driven**. After setup, the program enters `app.exec()`, which blocks and processes an internal queue of events: mouse clicks, key presses, paint requests, timers, network replies, custom signals, etc. Nothing after `app.exec()` in `main()` runs until the loop ends (normally via closing the last window or calling `QApplication.quit()`).

### QObject — the base of (almost) everything

`QObject` (from `QtCore`) is the base class for nearly every non-visual and visual Qt class (`QWidget` inherits from it). It provides:

- **Object trees / parent-child ownership** — when you give an object a parent (`QLabel(parent=self)` or `widget.setParent(x)`), Qt automatically deletes children when the parent is deleted. This replaces manual memory management.
- **Signals and slots** (see next section).
- **Properties** accessible via `object.property("name")` / `setProperty()`, useful for QSS styling and dynamic attributes.
- **Object naming**: `setObjectName()` / `objectName()` — used by style sheets and `findChild()`.
- **Event filtering**: `installEventFilter()`.

Key `QObject` methods:

| Method | Purpose |
|---|---|
| `setParent(parent)` | Reparent the object |
| `parent()` | Get parent |
| `children()` | List direct children |
| `findChild(Type, name)` | Find a descendant by type/name |
| `findChildren(Type, name)` | Find all matching descendants |
| `setObjectName(str)` / `objectName()` | Name used in QSS selectors and debugging |
| `deleteLater()` | Schedule safe deletion once control returns to the event loop |
| `blockSignals(bool)` | Temporarily suppress signal emission |
| `installEventFilter(obj)` | Intercept another object's events |
| `connect()` / `disconnect()` | Signal/slot wiring (usually called on the signal, not the object) |
| `setProperty(name, value)` / `property(name)` | Dynamic properties |
| `timerEvent(event)` | Override to receive `startTimer()` ticks |
| `moveToThread(thread)` | Move object's affinity to another `QThread` |

### QApplication

`QApplication` (QtWidgets) manages the GUI application's control flow and main settings. There's also `QGuiApplication` (QtGui, for non-widget Qt apps) and `QCoreApplication` (QtCore, non-GUI).

| Method / Property | Purpose |
|---|---|
| `QApplication(sys.argv)` | Constructor; parses Qt's CLI args |
| `exec()` | Start the event loop (blocks) |
| `quit()` / `exit(code)` | Stop the event loop |
| `instance()` | Static: get the running `QApplication` |
| `setStyle(name)` | Apply a QStyle (`"Fusion"`, `"Windows"`, etc.) |
| `setStyleSheet(str)` | Apply global QSS |
| `setApplicationName/setOrganizationName/setApplicationVersion` | Metadata used by `QSettings`, About dialogs |
| `setWindowIcon(QIcon)` | Default icon for all windows |
| `setFont(QFont)` | Default application font |
| `screens()` / `primaryScreen()` | Multi-monitor info |
| `clipboard()` | Access `QClipboard` |
| `processEvents()` | Manually pump pending events (use sparingly, e.g. to keep UI responsive during a long loop) |
| `aboutToQuit` (signal) | Emitted just before the app exits — good place for cleanup |

---

## 3. Signals and Slots

Signals and slots are Qt's mechanism for communication between objects, replacing raw callback registration. A **signal** is emitted when something happens; a **slot** is a callable that runs in response. Any Python function/method can act as a slot; you can also declare custom signals.

### Connecting

```python
from PyQt6.QtWidgets import QPushButton

button = QPushButton("Click me")
button.clicked.connect(lambda: print("Clicked!"))
button.clicked.connect(self.on_click)   # bound method also works
```

### Disconnecting

```python
button.clicked.disconnect(self.on_click)   # remove one connection
button.clicked.disconnect()                # remove all connections to this signal
```

### Custom signals — `pyqtSignal`

Signals must be declared as **class attributes** of a `QObject` subclass:

```python
from PyQt6.QtCore import QObject, pyqtSignal

class Worker(QObject):
    progress = pyqtSignal(int)              # emits an int
    finished = pyqtSignal()                 # no arguments
    result_ready = pyqtSignal(str, dict)    # multiple arguments

    def run(self):
        for i in range(101):
            self.progress.emit(i)
        self.result_ready.emit("done", {"count": 101})
        self.finished.emit()
```

Connect them like built-in signals:

```python
worker = Worker()
worker.progress.connect(lambda pct: print(f"{pct}%"))
```

### Slots — `pyqtSlot`

Decorating a method with `@pyqtSlot(...)` is optional but recommended: it lets Qt use a faster C++ dispatch path, improves type-checking, and disambiguates overloaded signals.

```python
from PyQt6.QtCore import pyqtSlot

class Receiver(QObject):
    @pyqtSlot(int)
    def on_progress(self, value):
        print("progress:", value)
```

### Connection types

`connect()` accepts a `type=` keyword using the `Qt.ConnectionType` enum:

| Type | Behavior |
|---|---|
| `Qt.ConnectionType.AutoConnection` (default) | Direct if same thread, queued if cross-thread |
| `Qt.ConnectionType.DirectConnection` | Slot runs immediately, in emitter's thread |
| `Qt.ConnectionType.QueuedConnection` | Slot runs later via the receiver's event loop (needed cross-thread) |
| `Qt.ConnectionType.BlockingQueuedConnection` | Like queued, but emitter blocks until slot finishes (cross-thread only, deadlock risk if misused) |
| `Qt.ConnectionType.UniqueConnection` | Fails silently if an identical connection already exists |

### Other signal-related tools

- `QObject.sender()` — inside a slot, returns which object emitted the signal that triggered it (useful for shared slots across multiple widgets).
- `signalsBlocked()` / `blockSignals(True/False)` — temporarily disable emission (e.g. while programmatically updating a widget you don't want to trigger its own handler).
- Signals can be connected to other signals directly: `signalA.connect(signalB)`.

---

## 4. QtCore Module

`QtCore` provides non-visual infrastructure. Selected essentials:

### QTimer

```python
from PyQt6.QtCore import QTimer

timer = QTimer()
timer.setInterval(1000)          # milliseconds
timer.timeout.connect(tick_fn)
timer.start()                    # or timer.start(1000)
timer.stop()
timer.setSingleShot(True)        # fire once only

QTimer.singleShot(2000, callback)  # convenience one-shot, no object needed
```

### QThread

Covered in depth in [Section 14](#14-threading--concurrency).

### Basic value/data classes

| Class | Purpose |
|---|---|
| `QDate`, `QTime`, `QDateTime` | Date/time values, arithmetic, formatting (`toString()`, `currentDateTime()`) |
| `QPoint`, `QPointF` | 2D integer/float point |
| `QSize`, `QSizeF` | Width/height pair |
| `QRect`, `QRectF` | Rectangle geometry (`x()`, `y()`, `width()`, `height()`, `contains()`, `intersects()`) |
| `QLine`, `QLineF` | Line segment |
| `QUrl` | URL parsing/building |
| `QVariant` | (Largely implicit in PyQt — Python objects auto-convert) |
| `QByteArray` | Raw byte buffer, useful with sockets/files |
| `QRegularExpression` | Modern regex engine (preferred over legacy `QRegExp`, which is removed in Qt6) |

### File & path handling

| Class | Purpose |
|---|---|
| `QDir` | Directory listing, navigation, filters |
| `QFile` | Low-level file I/O (`open()`, `read()`, `write()`, `close()`) |
| `QFileInfo` | Metadata about a file/path (size, permissions, timestamps) |
| `QStandardPaths` | Cross-platform standard locations (`writableLocation(QStandardPaths.StandardLocation.AppDataLocation)`, etc.) |
| `QSettings` | Persistent key/value config, see [Section 17](#17-files-settings--resources) |

### Model helpers (used with Model/View — see Section 10)

`QAbstractItemModel`, `QAbstractListModel`, `QAbstractTableModel`, `QSortFilterProxyModel`, `QModelIndex`, `QPersistentModelIndex`.

### Miscellaneous but frequently used

| Class / function | Purpose |
|---|---|
| `QObject.tr(text)` | Marks a string for translation (Qt Linguist workflow) |
| `QLocale` | Locale-aware number/date formatting |
| `QMimeData` | Payload for clipboard/drag-and-drop |
| `QCoreApplication.translate()` | Translation lookup |
| `QEventLoop` | A standalone, nestable event loop (useful for blocking-style waits) |
| `Qt` namespace | Global enums: `Qt.AlignmentFlag`, `Qt.Orientation`, `Qt.Key`, `Qt.MouseButton`, `Qt.ItemDataRole`, `Qt.CheckState`, etc. |

> **Qt6 enum note:** In Qt6/PyQt6, enums are scoped (e.g. `Qt.AlignmentFlag.AlignCenter` instead of PyQt5's `Qt.AlignCenter`). This is one of the most common porting gotchas from PyQt5.

---

## 5. QtWidgets Module — Windows & Containers

### QWidget

The base class of every visual element. Key members:

| Method | Purpose |
|---|---|
| `show()` / `hide()` / `close()` | Visibility control |
| `setVisible(bool)` | Programmatic show/hide |
| `resize(w, h)` / `size()` | Widget dimensions |
| `move(x, y)` / `pos()` | Position relative to parent |
| `setGeometry(x, y, w, h)` | Position + size together |
| `setFixedSize(w, h)` / `setMinimumSize()` / `setMaximumSize()` | Size constraints |
| `setLayout(layout)` | Attach a layout manager |
| `setWindowTitle(str)` | Titlebar text (top-level widgets) |
| `setWindowIcon(QIcon)` | Titlebar/taskbar icon |
| `setEnabled(bool)` / `isEnabled()` | Enable/disable interaction |
| `setStyleSheet(str)` | Per-widget QSS |
| `setToolTip(str)` | Hover tooltip |
| `setFocus()` / `hasFocus()` | Keyboard focus |
| `setFocusPolicy(Qt.FocusPolicy...)` | How the widget accepts focus |
| `update()` / `repaint()` | Request/force a redraw |
| `setCursor(Qt.CursorShape...)` | Mouse cursor over the widget |
| `setSizePolicy(horizontal, vertical)` | How the widget grows/shrinks in layouts |
| `mapToGlobal()` / `mapFromGlobal()` | Coordinate conversions |
| `raise_()` / `lower()` | Stacking order |
| `setAttribute(Qt.WidgetAttribute...)` | Fine-grained behavior flags |
| `setWindowFlags(Qt.WindowType...)` | Window decorations/behavior (frameless, always-on-top, etc.) |

Common overridable event handlers (see [Section 13](#13-event-handling)): `paintEvent`, `resizeEvent`, `mousePressEvent`, `keyPressEvent`, `closeEvent`.

### QMainWindow

A ready-made application window skeleton with slots for a menu bar, toolbars, dock widgets, a status bar, and one central widget.

```python
from PyQt6.QtWidgets import QMainWindow, QLabel, QDockWidget

class MainWindow(QMainWindow):
    def __init__(self):
        super().__init__()
        self.setCentralWidget(QLabel("Main content"))
        self.setMenuBar(...)          # or use menuBar() to get the default one
        self.addToolBar(...)
        self.setStatusBar(...)
        dock = QDockWidget("Panel", self)
        self.addDockWidget(Qt.DockWidgetArea.LeftDockWidgetArea, dock)
```

Key `QMainWindow` methods: `setCentralWidget()`, `menuBar()`, `addToolBar()`, `statusBar()`, `addDockWidget()`, `removeToolBar()`, `setUnifiedTitleAndToolBarOnMac()`.

### QDialog

Base class for modal/modeless popup windows. See [Section 8](#8-dialogs).

### Container / grouping widgets

| Widget | Purpose |
|---|---|
| `QWidget` | Plain generic container |
| `QFrame` | Adds a border/frame style (`setFrameShape`, `setFrameShadow`) |
| `QGroupBox` | Titled box, often around a set of related controls |
| `QTabWidget` | Multiple pages selected via tabs (`addTab()`, `setCurrentIndex()`) |
| `QStackedWidget` | Multiple pages, one visible at a time, switched programmatically (`setCurrentIndex()`) |
| `QScrollArea` | Makes an oversized child widget scrollable |
| `QSplitter` | Resizable divider between two or more widgets |
| `QToolBox` | Vertically stacked collapsible sections |
| `QMdiArea` / `QMdiSubWindow` | Multiple-document interface (internal child windows) |

---

## 6. Layouts

Layouts automatically size and position child widgets, adjusting on window resize, font/DPI changes, and content changes — the recommended alternative to manual `setGeometry()` calls.

### QVBoxLayout / QHBoxLayout

Stack widgets vertically or horizontally.

```python
from PyQt6.QtWidgets import QVBoxLayout, QHBoxLayout, QWidget, QPushButton

layout = QVBoxLayout()
layout.addWidget(QPushButton("One"))
layout.addWidget(QPushButton("Two"))
layout.addStretch()              # flexible empty space
layout.addSpacing(20)            # fixed gap
container = QWidget()
container.setLayout(layout)
```

### QGridLayout

Row/column placement:

```python
from PyQt6.QtWidgets import QGridLayout

grid = QGridLayout()
grid.addWidget(QPushButton("1,1"), 0, 0)
grid.addWidget(QPushButton("1,2 span"), 0, 1, 1, 2)   # rowSpan=1, colSpan=2
grid.setColumnStretch(0, 1)
grid.setRowMinimumHeight(0, 40)
```

### QFormLayout

Two-column "label: field" layout common in settings screens:

```python
from PyQt6.QtWidgets import QFormLayout, QLineEdit

form = QFormLayout()
form.addRow("Name:", QLineEdit())
form.addRow("Email:", QLineEdit())
```

### Layout method reference

| Method | Purpose |
|---|---|
| `addWidget(widget, [stretch], [alignment])` | Insert a widget |
| `addLayout(layout)` | Nest another layout |
| `addStretch(factor)` | Insert expanding empty space |
| `addSpacing(px)` | Insert fixed empty space |
| `setSpacing(px)` | Gap between items |
| `setContentsMargins(l, t, r, b)` | Outer padding |
| `setAlignment(widget, Qt.AlignmentFlag)` | Align a specific item |
| `removeWidget(widget)` | Detach (does not delete) a widget |
| `count()` / `itemAt(i)` | Introspect layout contents |

---

## 7. Common Widgets Reference

### Buttons

| Widget | Key signals | Key methods |
|---|---|---|
| `QPushButton` | `clicked`, `pressed`, `released`, `toggled` | `setText()`, `setIcon()`, `setCheckable(bool)`, `setDefault(bool)` |
| `QToolButton` | `clicked` | `setToolButtonStyle()`, `setPopupMode()`, `setMenu()` |
| `QRadioButton` | `toggled` | `setChecked()`, `isChecked()` (auto-exclusive within parent) |
| `QCheckBox` | `stateChanged` / `checkStateChanged` (Qt6) | `setChecked()`, `isChecked()`, `setTristate(bool)` |

### Text input

| Widget | Key signals | Key methods |
|---|---|---|
| `QLineEdit` | `textChanged`, `textEdited`, `returnPressed`, `editingFinished` | `text()`, `setText()`, `setPlaceholderText()`, `setEchoMode(QLineEdit.EchoMode.Password)`, `setValidator()`, `setMaxLength()` |
| `QTextEdit` | `textChanged` | `toPlainText()`, `setPlainText()`, `toHtml()`, `setHtml()`, `append()` (rich text) |
| `QPlainTextEdit` | `textChanged` | Like `QTextEdit` but optimized for large plain text (e.g. logs, code) |
| `QSpinBox` | `valueChanged` | `setRange(min, max)`, `setSingleStep()`, `value()`, `setSuffix()` |
| `QDoubleSpinBox` | `valueChanged` | Same as `QSpinBox` with `setDecimals()` |

### Display widgets

| Widget | Purpose | Key methods |
|---|---|---|
| `QLabel` | Static text or image | `setText()`, `setPixmap()`, `setAlignment()`, `setWordWrap(True)` |
| `QProgressBar` | Progress indicator | `setRange(0, 100)`, `setValue(n)`, `setTextVisible(bool)` |
| `QLCDNumber` | Digital-style numeric display | `display(value)` |

### Selection widgets

| Widget | Key signals | Key methods |
|---|---|---|
| `QComboBox` | `currentIndexChanged`, `currentTextChanged` | `addItem()`, `addItems([...])`, `currentText()`, `setEditable(bool)` |
| `QSlider` | `valueChanged`, `sliderMoved` | `setRange()`, `setValue()`, `setOrientation(Qt.Orientation.Horizontal)`, `setTickPosition()` |
| `QDial` | `valueChanged` | Same pattern as `QSlider`, circular control |
| `QDateEdit` / `QTimeEdit` / `QDateTimeEdit` | `dateChanged` / `timeChanged` / `dateTimeChanged` | `setCalendarPopup(True)`, `setDisplayFormat()` |
| `QCalendarWidget` | `selectionChanged`, `clicked` | `selectedDate()`, `setSelectedDate()` |

### Item-based list/table widgets (simple, non-model-backed)

| Widget | Key signals | Key methods |
|---|---|---|
| `QListWidget` | `itemClicked`, `currentItemChanged` | `addItem()`, `addItems()`, `item(row)`, `currentItem()`, `takeItem()` |
| `QTableWidget` | `cellClicked`, `itemChanged` | `setRowCount()`, `setColumnCount()`, `setItem(r, c, QTableWidgetItem)`, `setHorizontalHeaderLabels()` |
| `QTreeWidget` | `itemClicked`, `itemExpanded` | `addTopLevelItem()`, `setHeaderLabels()`, `QTreeWidgetItem(parent, [values])` |

> For large or dynamic datasets, prefer the **Model/View** classes (`QListView`, `QTableView`, `QTreeView` + a model) over the `*Widget` convenience classes — see [Section 10](#10-modelview-architecture).

### Other useful widgets

| Widget | Purpose |
|---|---|
| `QScrollBar` | Standalone scrollbar (usually managed automatically by scroll areas/views) |
| `QSplashScreen` | Splash image shown while the app loads |
| `QSystemTrayIcon` | Icon + menu in the OS system tray |
| `QGraphicsView` / `QGraphicsScene` / `QGraphicsItem` | 2D canvas for custom, interactive, scalable graphics (games, diagrams) |
| `QCompleter` | Autocomplete popup attached to a `QLineEdit` |
| `QUndoStack` / `QUndoCommand` | Built-in undo/redo framework |

---

## 8. Dialogs

`QDialog` is the base class; PyQt6 also ships several ready-made dialogs.

### QDialog basics

```python
from PyQt6.QtWidgets import QDialog, QVBoxLayout, QDialogButtonBox

class MyDialog(QDialog):
    def __init__(self, parent=None):
        super().__init__(parent)
        self.setWindowTitle("Settings")
        layout = QVBoxLayout(self)
        buttons = QDialogButtonBox(
            QDialogButtonBox.StandardButton.Ok | QDialogButtonBox.StandardButton.Cancel
        )
        buttons.accepted.connect(self.accept)
        buttons.rejected.connect(self.reject)
        layout.addWidget(buttons)

dlg = MyDialog()
if dlg.exec() == QDialog.DialogCode.Accepted:
    print("User clicked OK")
```

| Method | Purpose |
|---|---|
| `exec()` | Show modally, block until closed, returns `DialogCode` |
| `show()` | Show modelessly (non-blocking) |
| `accept()` / `reject()` | Close with Accepted/Rejected result |
| `setModal(bool)` | Modal vs modeless |
| `result()` | Get the last result code |

### Built-in dialog helpers (static convenience methods)

| Class | Example call |
|---|---|
| `QMessageBox` | `QMessageBox.information(parent, "Title", "Text")`, `.warning(...)`, `.critical(...)`, `.question(...)` |
| `QFileDialog` | `QFileDialog.getOpenFileName(parent, "Open", dir, "Images (*.png *.jpg)")`, `.getSaveFileName()`, `.getExistingDirectory()`, `.getOpenFileNames()` |
| `QColorDialog` | `QColorDialog.getColor(initial, parent)` |
| `QFontDialog` | `QFontDialog.getFont(initial, parent)` |
| `QInputDialog` | `QInputDialog.getText()`, `.getInt()`, `.getDouble()`, `.getItem()` |
| `QProgressDialog` | Cancelable progress popup for long tasks |
| `QErrorMessage` | Shows an error, remembers "don't show again" per message |

---

## 9. Menus, Toolbars, Actions & Status Bar

### QAction

`QAction` represents a user-triggerable command that can appear simultaneously in a menu, a toolbar, and be bound to a shortcut — defined once, used everywhere. (Note: In PyQt6, `QAction` lives in **`QtGui`**, unlike PyQt5 where it was in `QtWidgets`.)

```python
from PyQt6.QtGui import QAction, QIcon
from PyQt6.QtCore import QSize

action = QAction(QIcon("open.png"), "&Open...", parent)
action.setShortcut("Ctrl+O")
action.setStatusTip("Open a file")
action.triggered.connect(open_file)
action.setCheckable(True)
```

### Menus

```python
menu_bar = self.menuBar()          # QMainWindow convenience
file_menu = menu_bar.addMenu("&File")
file_menu.addAction(action)
file_menu.addSeparator()
submenu = file_menu.addMenu("Recent Files")
```

`QMenu` methods: `addAction()`, `addMenu()`, `addSeparator()`, `exec(pos)` (show as a context menu), `popup()`.

### Context menus

```python
widget.setContextMenuPolicy(Qt.ContextMenuPolicy.CustomContextMenu)
widget.customContextMenuRequested.connect(show_context_menu)

def show_context_menu(pos):
    menu = QMenu()
    menu.addAction("Copy")
    menu.exec(widget.mapToGlobal(pos))
```

### Toolbars

```python
toolbar = self.addToolBar("Main")
toolbar.addAction(action)
toolbar.setIconSize(QSize(24, 24))
toolbar.setMovable(False)
```

### Status bar

```python
self.setStatusBar(QStatusBar())
self.statusBar().showMessage("Ready", 3000)   # 3-second timeout
self.statusBar().addPermanentWidget(QLabel("v1.0"))
```

---

## 10. Model/View Architecture

For anything beyond simple static lists, Qt separates **data (model)** from **presentation (view)**, optionally through a **delegate** that customizes rendering/editing of individual cells.

### The pieces

| Piece | Role |
|---|---|
| `QAbstractItemModel` | Base class for any model (tree-like data) |
| `QAbstractListModel` | Simplified base for flat, 1-D lists |
| `QAbstractTableModel` | Simplified base for 2-D tabular data |
| `QStandardItemModel` + `QStandardItem` | Ready-made, in-memory generic model — no subclassing needed |
| `QSortFilterProxyModel` | Wraps another model to add sorting/filtering without touching the source data |
| `QListView`, `QTableView`, `QTreeView`, `QColumnView` | Views that render any compatible model |
| `QStyledItemDelegate` | Customizes how cells are drawn/edited |

### Custom table model example

```python
from PyQt6.QtCore import QAbstractTableModel, Qt, QModelIndex

class PandasLikeModel(QAbstractTableModel):
    def __init__(self, data):
        super().__init__()
        self._data = data   # list of lists

    def rowCount(self, parent=QModelIndex()):
        return len(self._data)

    def columnCount(self, parent=QModelIndex()):
        return len(self._data[0]) if self._data else 0

    def data(self, index, role=Qt.ItemDataRole.DisplayRole):
        if role == Qt.ItemDataRole.DisplayRole:
            return str(self._data[index.row()][index.column()])

    def headerData(self, section, orientation, role=Qt.ItemDataRole.DisplayRole):
        if role == Qt.ItemDataRole.DisplayRole and orientation == Qt.Orientation.Horizontal:
            return f"Col {section}"
```

```python
from PyQt6.QtWidgets import QTableView

view = QTableView()
view.setModel(PandasLikeModel([[1, 2], [3, 4]]))
```

### Quick, no-subclass option: QStandardItemModel

```python
from PyQt6.QtGui import QStandardItemModel, QStandardItem

model = QStandardItemModel(0, 2)
model.setHorizontalHeaderLabels(["Name", "Age"])
model.appendRow([QStandardItem("Alice"), QStandardItem("30")])

view = QTableView()
view.setModel(model)
```

### Key model roles (`Qt.ItemDataRole`)

`DisplayRole`, `EditRole`, `DecorationRole` (icon), `ToolTipRole`, `BackgroundRole`, `ForegroundRole`, `CheckStateRole`, `UserRole` (start of custom, app-defined roles).

---

## 11. QtGui Module

`QtGui` sits between `QtCore` and `QtWidgets`: low-level GUI functionality without being tied to widgets specifically (also used by Qt Quick).

### Fonts & colors

```python
from PyQt6.QtGui import QFont, QColor, QPalette

font = QFont("Arial", 14, QFont.Weight.Bold)
widget.setFont(font)

color = QColor(255, 0, 0)          # or QColor("#ff0000")
palette = widget.palette()
palette.setColor(QPalette.ColorRole.Window, color)
widget.setPalette(palette)
```

### Icons & images

| Class | Purpose |
|---|---|
| `QIcon` | Scalable, state-aware icon (normal/disabled/active, multiple resolutions) |
| `QPixmap` | Off-screen raster image optimized for on-screen display |
| `QImage` | Raster image optimized for pixel-level manipulation/I/O |
| `QMovie` | Animated image playback (e.g. GIF) |
| `QPainterPath` | Vector path for complex shapes |

```python
icon = QIcon("logo.png")
pixmap = QPixmap("photo.jpg").scaled(200, 200, Qt.AspectRatioMode.KeepAspectRatio)
label.setPixmap(pixmap)
```

### Clipboard & drag-and-drop

```python
from PyQt6.QtWidgets import QApplication
QApplication.clipboard().setText("copied text")
text = QApplication.clipboard().text()
```

`QDrag`, `QMimeData`, and the widget events `dragEnterEvent`, `dragMoveEvent`, `dropEvent` implement drag-and-drop (see [Section 13](#13-event-handling)).

### QAction, QKeySequence, QShortcut

- `QAction` — see [Section 9](#9-menus-toolbars-actions--status-bar).
- `QKeySequence("Ctrl+Shift+S")` — cross-platform shortcut representation.
- `QShortcut(QKeySequence("Ctrl+Q"), widget, callback)` — a shortcut not tied to a menu action.

### Validators

| Class | Purpose |
|---|---|
| `QIntValidator` | Restrict a `QLineEdit` to integers in a range |
| `QDoubleValidator` | Restrict to floating-point numbers |
| `QRegularExpressionValidator` | Restrict via a regex pattern |

```python
from PyQt6.QtGui import QIntValidator
line_edit.setValidator(QIntValidator(0, 100))
```

---

## 12. Painting & Custom Widgets

To draw custom content, subclass `QWidget` and override `paintEvent`, using `QPainter`.

```python
from PyQt6.QtWidgets import QWidget
from PyQt6.QtGui import QPainter, QPen, QBrush, QColor
from PyQt6.QtCore import Qt, QRect

class CustomWidget(QWidget):
    def paintEvent(self, event):
        painter = QPainter(self)
        painter.setRenderHint(QPainter.RenderHint.Antialiasing)
        painter.setPen(QPen(QColor("black"), 2))
        painter.setBrush(QBrush(QColor("skyblue")))
        painter.drawRect(10, 10, 100, 60)
        painter.drawEllipse(120, 10, 80, 80)
        painter.drawText(QRect(0, 100, 200, 30), Qt.AlignmentFlag.AlignCenter, "Custom draw")
```

### QPainter drawing methods

`drawLine()`, `drawRect()`, `drawRoundedRect()`, `drawEllipse()`, `drawArc()`, `drawPolygon()`, `drawPath(QPainterPath)`, `drawText()`, `drawPixmap()`, `drawImage()`, `fillRect()`, `setOpacity()`, `translate()`, `rotate()`, `scale()`, `save()` / `restore()` (state stack for transforms).

### QGraphicsView framework (for many interactive items)

```python
from PyQt6.QtWidgets import QGraphicsView, QGraphicsScene, QGraphicsEllipseItem

scene = QGraphicsScene()
scene.addItem(QGraphicsEllipseItem(0, 0, 50, 50))
view = QGraphicsView(scene)
```

`QGraphicsItem` subclasses (`QGraphicsRectItem`, `QGraphicsPixmapItem`, `QGraphicsTextItem`, ...) support independent selection, movement, rotation, and z-ordering — ideal for diagram editors, node graphs, or simple 2D games.

---

## 13. Event Handling

Every `QWidget` can override event handler methods; the default implementation typically just calls the base class (important to preserve default behavior via `super().eventName(event)`).

### Common event handlers

| Method | Fires on |
|---|---|
| `mousePressEvent(event)` / `mouseReleaseEvent` / `mouseMoveEvent` / `mouseDoubleClickEvent` | Mouse interaction |
| `keyPressEvent(event)` / `keyReleaseEvent` | Keyboard input |
| `paintEvent(event)` | Widget needs redrawing |
| `resizeEvent(event)` | Widget resized |
| `closeEvent(event)` | Window close requested (`event.accept()`/`ignore()` to control it) |
| `showEvent` / `hideEvent` | Visibility changes |
| `enterEvent` / `leaveEvent` | Mouse enters/leaves widget bounds |
| `focusInEvent` / `focusOutEvent` | Keyboard focus changes |
| `wheelEvent(event)` | Mouse wheel scroll |
| `dragEnterEvent` / `dragMoveEvent` / `dropEvent` / `dragLeaveEvent` | Drag-and-drop |
| `contextMenuEvent(event)` | Right-click (alternative to the `customContextMenuRequested` signal approach) |

### Example: capturing a key press

```python
from PyQt6.QtCore import Qt

class MyWidget(QWidget):
    def keyPressEvent(self, event):
        if event.key() == Qt.Key.Key_Escape:
            self.close()
        else:
            super().keyPressEvent(event)
```

### Event filters (intercept another object's events)

```python
class ClickFilter(QObject):
    def eventFilter(self, obj, event):
        if event.type() == QEvent.Type.MouseButtonPress:
            print("Clicked inside", obj)
        return super().eventFilter(obj, event)   # False = let event continue

widget.installEventFilter(click_filter_instance)
```

### Event propagation

Events flow through: **event filters on the object** → the object's own handler (e.g. `mousePressEvent`) → optionally propagated to the parent if `event.ignore()` is called (for some event types). Call `event.accept()` to mark it fully handled.

---

## 14. Threading & Concurrency

Qt's GUI is **not thread-safe** — only the main thread may touch widgets. Long-running work must happen off the main thread, then communicate results back via signals (which are automatically queued across threads).

### Worker-object pattern (recommended)

```python
from PyQt6.QtCore import QObject, QThread, pyqtSignal

class Worker(QObject):
    finished = pyqtSignal()
    progress = pyqtSignal(int)

    def run(self):
        for i in range(100):
            # ... do work ...
            self.progress.emit(i)
        self.finished.emit()

class MainWindow(QMainWindow):
    def start_work(self):
        self.thread = QThread()
        self.worker = Worker()
        self.worker.moveToThread(self.thread)

        self.thread.started.connect(self.worker.run)
        self.worker.finished.connect(self.thread.quit)
        self.worker.finished.connect(self.worker.deleteLater)
        self.thread.finished.connect(self.thread.deleteLater)
        self.worker.progress.connect(self.update_progress_bar)

        self.thread.start()
```

This pattern (subclass `QObject`, not `QThread`, and use `moveToThread`) is preferred over subclassing `QThread` directly, because it keeps the worker's slots executing on the *worker* thread while staying flexible.

### Other concurrency tools

| Tool | Purpose |
|---|---|
| `QThreadPool` + `QRunnable` | Run many short-lived tasks in a managed pool (`QThreadPool.globalInstance().start(runnable)`) |
| `QtConcurrent` (via `concurrent.futures`-like helpers, limited in PyQt) | Higher-level parallel map/filter (more commonly used from C++; in Python, `concurrent.futures` or `multiprocessing` combined with signals is often simpler) |
| `QMutex`, `QMutexLocker`, `QSemaphore`, `QWaitCondition` | Low-level synchronization primitives |
| `QThread.sleep()/msleep()/usleep()` | Static sleep helpers |

### Rules of thumb

1. Never call widget methods from a non-GUI thread — emit a signal instead and let a main-thread slot update the UI.
2. Use `QueuedConnection` (the default for cross-thread signals) so slots run safely in the receiver's thread.
3. Avoid `time.sleep()` in the GUI thread — it freezes the whole UI; use `QTimer` or a worker thread instead.

---

## 15. Styling (Qt Style Sheets)

QSS is a CSS-like syntax for styling widgets.

```python
widget.setStyleSheet("""
    QPushButton {
        background-color: #2d89ef;
        color: white;
        border-radius: 6px;
        padding: 6px 12px;
    }
    QPushButton:hover {
        background-color: #1b5fbd;
    }
    QPushButton:pressed {
        background-color: #123f80;
    }
    QLineEdit {
        border: 1px solid #ccc;
        border-radius: 4px;
    }
""")
```

- Selectors can target a class (`QPushButton`), an object name (`#myButton`), or a property (`QPushButton[flat="true"]`).
- Pseudo-states: `:hover`, `:pressed`, `:disabled`, `:checked`, `:focus`.
- Can be applied globally (`QApplication.setStyleSheet()`) or per-widget.
- `QApplication.setStyle("Fusion")` changes the underlying native look-and-feel engine, often combined with QSS for a consistent cross-platform look.

---

## 16. Networking (QtNetwork)

| Class | Purpose |
|---|---|
| `QNetworkAccessManager` | Issues HTTP(S) requests, manages replies asynchronously |
| `QNetworkRequest` | Describes a request (URL, headers) |
| `QNetworkReply` | The (async) response object — a `QIODevice` you read once `finished` fires |
| `QTcpSocket` / `QTcpServer` | Raw TCP client/server sockets |
| `QUdpSocket` | UDP datagrams |
| `QSslSocket` | TLS-secured TCP |

```python
from PyQt6.QtNetwork import QNetworkAccessManager, QNetworkRequest
from PyQt6.QtCore import QUrl

manager = QNetworkAccessManager()

def handle_response(reply):
    print(reply.readAll().data().decode())

manager.finished.connect(handle_response)
manager.get(QNetworkRequest(QUrl("https://example.com")))
```

---

## 17. Files, Settings & Resources

### QSettings — persistent application settings

Stores key/value pairs in the platform-appropriate backend (Windows Registry, macOS plist, INI file on Linux).

```python
from PyQt6.QtCore import QSettings

settings = QSettings("MyCompany", "MyApp")
settings.setValue("window/width", 800)
width = settings.value("window/width", 640, type=int)   # default + type coercion
settings.sync()   # force write to disk
```

### Qt Resource System (`.qrc`)

Bundle images/icons/fonts into the executable via a `.qrc` XML file compiled with `pyrcc6` (or, in newer toolchains, embedded directly), then reference them with `:/prefix/name.png` paths instead of filesystem paths — useful for single-file distributable apps.

### Loading Designer `.ui` files

```python
from PyQt6 import uic
window = uic.loadUi("mainwindow.ui")
```

Or convert once to Python with the `pyuic6` command-line tool:

```bash
pyuic6 mainwindow.ui -o ui_mainwindow.py
```

---

## 18. Packaging & Deployment

PyQt6 apps are typically frozen into standalone executables with:

- **PyInstaller** — `pyinstaller --onefile --windowed main.py` (most common, cross-platform)
- **cx_Freeze** — alternative freezer
- **Nuitka** — compiles to C for smaller/faster binaries
- **fbs (fman build system)** — historically popular for PyQt5, less actively maintained for PyQt6

Common gotchas: remember to include Qt plugin directories (platform plugins, image format plugins) if using `--onefile` mode and things fail to launch; test the frozen build on a clean machine/VM before shipping.

---

## 19. Quick Reference Tables

### Essential imports cheat-sheet

```python
from PyQt6.QtWidgets import (
    QApplication, QMainWindow, QWidget, QDialog, QLabel, QPushButton,
    QLineEdit, QTextEdit, QComboBox, QCheckBox, QRadioButton, QSlider,
    QSpinBox, QVBoxLayout, QHBoxLayout, QGridLayout, QFormLayout,
    QTabWidget, QStackedWidget, QListWidget, QTableWidget, QTreeWidget,
    QMessageBox, QFileDialog, QMenu, QStatusBar, QToolBar,
)
from PyQt6.QtGui import (
    QIcon, QPixmap, QFont, QColor, QPainter, QAction, QKeySequence,
    QStandardItemModel, QStandardItem,
)
from PyQt6.QtCore import (
    Qt, QTimer, QThread, QObject, pyqtSignal, pyqtSlot, QSize, QRect,
    QSettings, QUrl, QAbstractTableModel, QModelIndex,
)
```

### PyQt5 → PyQt6 migration notes

| PyQt5 | PyQt6 |
|---|---|
| `app.exec_()` | `app.exec()` |
| `Qt.AlignCenter` | `Qt.AlignmentFlag.AlignCenter` |
| `QAction` from `QtWidgets` | `QAction` from `QtGui` |
| Flat enums everywhere (`Qt.Key_Escape`) | Scoped enums (`Qt.Key.Key_Escape`) |
| `QRegExp` | `QRegularExpression` |
| `QDesktopWidget` | `QScreen` (via `QApplication.primaryScreen()`) |
| High-DPI attributes needed manual opt-in | High-DPI scaling on by default |

### Widget class hierarchy (simplified)

```
QObject
 └── QWidget
      ├── QFrame
      │    ├── QLabel
      │    ├── QLineEdit
      │    ├── QAbstractScrollArea
      │    │    ├── QAbstractItemView
      │    │    │    ├── QListView / QListWidget
      │    │    │    ├── QTableView / QTableWidget
      │    │    │    └── QTreeView / QTreeWidget
      │    │    ├── QTextEdit / QPlainTextEdit
      │    │    └── QScrollArea
      │    └── QGroupBox
      ├── QAbstractButton
      │    ├── QPushButton
      │    ├── QCheckBox
      │    ├── QRadioButton
      │    └── QToolButton
      ├── QDialog
      │    ├── QMessageBox
      │    ├── QFileDialog
      │    ├── QColorDialog
      │    └── QFontDialog
      └── QMainWindow
```

### Naming conventions in Qt/PyQt

- Classes: `QPascalCase` (Qt built-ins), your own subclasses can use plain `PascalCase`.
- Methods/signals/slots: `camelCase`.
- Enums (Qt6): `Namespace.EnumClass.MemberName`, e.g. `Qt.Orientation.Horizontal`.

---

### Where to go next

- Official docs: Qt for Python style docs are C++-oriented but map almost 1:1 to PyQt naming — use them alongside PyQt6's own reference at Riverbank's site.
- Practice by rebuilding a small tool (a to-do list, a unit converter, or a simple text editor) touching layouts, signals/slots, a dialog, and QSettings — that combination covers the majority of real-world app patterns.
