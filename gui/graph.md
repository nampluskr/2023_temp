### [graph.py]

```python
import sys
import numpy as np
import matplotlib.pyplot as plt

from matplotlib.backends.backend_qtagg import FigureCanvas
from PySide6.QtWidgets import QApplication, QMainWindow
from graph_ui import Ui_MainWindow


class MainWindow(QMainWindow):
    def __init__(self):
        super(MainWindow, self).__init__()
        self.ui = Ui_MainWindow()
        self.ui.setupUi(self)
        
        self.fig = plt.Figure()
        self.canvas = FigureCanvas(self.fig)
        self.ui.graph_layout.addWidget(self.canvas)

    def draw(self):        
        x = np.linspace(0, 2*np.pi, 101)
        y = np.sin(x)
        ax = self.fig.add_subplot(111)
        ax.plot(x, y)
        self.fig.tight_layout()
        self.canvas.draw()
        
    def clear(self):
        self.fig.clear()
        self.canvas.draw()


if __name__ == "__main__":

    app = QApplication(sys.argv)
    window = MainWindow()
    window.show()
    sys.exit(app.exec())
```

### [graph.ui]

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <class>MainWindow</class>
 <widget class="QMainWindow" name="MainWindow">
  <property name="geometry">
   <rect>
    <x>0</x>
    <y>0</y>
    <width>640</width>
    <height>480</height>
   </rect>
  </property>
  <property name="windowTitle">
   <string>MainWindow</string>
  </property>
  <widget class="QWidget" name="centralwidget">
   <widget class="QWidget" name="verticalLayoutWidget">
    <property name="geometry">
     <rect>
      <x>150</x>
      <y>10</y>
      <width>361</width>
      <height>411</height>
     </rect>
    </property>
    <layout class="QVBoxLayout" name="graph_layout"/>
   </widget>
   <widget class="QPushButton" name="pushButton">
    <property name="geometry">
     <rect>
      <x>20</x>
      <y>20</y>
      <width>75</width>
      <height>24</height>
     </rect>
    </property>
    <property name="text">
     <string>PushButton</string>
    </property>
   </widget>
   <widget class="QPushButton" name="pushButton_2">
    <property name="geometry">
     <rect>
      <x>20</x>
      <y>50</y>
      <width>75</width>
      <height>24</height>
     </rect>
    </property>
    <property name="text">
     <string>PushButton</string>
    </property>
   </widget>
   <widget class="QRadioButton" name="radioButton">
    <property name="geometry">
     <rect>
      <x>20</x>
      <y>90</y>
      <width>89</width>
      <height>20</height>
     </rect>
    </property>
    <property name="text">
     <string>RadioButton</string>
    </property>
   </widget>
  </widget>
  <widget class="QMenuBar" name="menubar">
   <property name="geometry">
    <rect>
     <x>0</x>
     <y>0</y>
     <width>640</width>
     <height>22</height>
    </rect>
   </property>
  </widget>
  <widget class="QStatusBar" name="statusbar"/>
 </widget>
 <resources/>
 <connections>
  <connection>
   <sender>pushButton</sender>
   <signal>clicked()</signal>
   <receiver>MainWindow</receiver>
   <slot>draw()</slot>
   <hints>
    <hint type="sourcelabel">
     <x>76</x>
     <y>52</y>
    </hint>
    <hint type="destinationlabel">
     <x>122</x>
     <y>48</y>
    </hint>
   </hints>
  </connection>
  <connection>
   <sender>pushButton_2</sender>
   <signal>clicked()</signal>
   <receiver>MainWindow</receiver>
   <slot>clear()</slot>
   <hints>
    <hint type="sourcelabel">
     <x>78</x>
     <y>82</y>
    </hint>
    <hint type="destinationlabel">
     <x>124</x>
     <y>83</y>
    </hint>
   </hints>
  </connection>
 </connections>
 <slots>
  <slot>draw()</slot>
  <slot>clear()</slot>
 </slots>
</ui>
```

### [QTreeView / QListView]

```python
import sys
import numpy as np
import matplotlib.pyplot as plt
import skimage

# from PyQt5.QtWidgets import QMainWindow
from PyQt5.QtWidgets import QApplication, QWidget, QVBoxLayout, QHBoxLayout
from PyQt5.QtWidgets import QTreeView, QListView, QLineEdit, QPushButton, QLabel
from PyQt5.QtWidgets import QFileSystemModel
from PyQt5.QtGui import QIcon
from PyQt5.QtCore import QDir
import matplotlib.pyplot as plt
# from matplotlib.backends.backend_qt5agg import FigureCanvasQTAgg as FigureCanvas
from matplotlib.backends.backend_qtagg import FigureCanvas

class ApplicationWindow(QWidget):
    def __init__(self):
        super().__init__()
        self.init()

    def init(self):
        self.setGeometry(200, 100, 1200, 800)
        self.setWindowTitle("2D Data Analyzer ver. 0.1")
        self.setWindowIcon(QIcon('detective2.png'))
        # self.statusBar().showMessage('Ready')

        root_path = r"E:\data_pe\2d_data"

        # Left Layout
        
        self.label = QLabel("Data Path:", self)
        self.lineEdit = QLineEdit()
        self.lineEdit.setText(root_path)
        self.pushButton1 = QPushButton("Plot Graph")
        self.pushButton1.clicked.connect(self.plot_graph)
        self.pushButton2 = QPushButton("Show Image")
        self.pushButton2.clicked.connect(self.show_image)

        self.treeview = QTreeView()
        self.listview = QListView()

        layout1 = QVBoxLayout()
        layout1.addWidget(self.label)
        layout1.addWidget(self.lineEdit)
        layout1.addWidget(self.pushButton1)
        layout1.addWidget(self.pushButton2)
        layout1.addWidget(self.treeview)
        layout1.addWidget(self.listview)
        layout1.addStretch(1)

        ## folders
        self.dirModel = QFileSystemModel()
        self.dirModel.setRootPath(root_path)
        self.dirModel.setFilter(QDir.NoDotAndDotDot | QDir.AllDirs)

        self.treeview.setModel(self.dirModel)
        self.treeview.setAlternatingRowColors(True)
        self.treeview.setRootIndex(self.dirModel.index(root_path))
        for i in range(1, self.treeview.model().columnCount()):
            self.treeview.header().hideSection(i)
        self.treeview.clicked.connect(self.select_folder)

        ## files
        self.fileModel = QFileSystemModel()
        self.fileModel.setFilter(QDir.NoDotAndDotDot | QDir.Files)

        self.listview.setModel(self.fileModel)
        self.listview.setAlternatingRowColors(True)
        self.listview.setRootIndex(self.fileModel.index(root_path))
        # self.listview.doubleClicked.connect(self.select_file)
        # self.listview.clicked.connect(self.select_file)
        self.listview.activated.connect(self.select_file)

        # Right Layout
        self.fig = plt.Figure()
        self.canvas = FigureCanvas(self.fig)
        layout2 = QVBoxLayout()
        layout2.addWidget(self.canvas)

        # App Layout
        layout = QHBoxLayout()
        layout.addLayout(layout1)
        layout.addLayout(layout2)
        layout.setStretchFactor(layout1, 0)
        layout.setStretchFactor(layout2, 1)

        self.setLayout(layout)

    def select_folder(self, index):
        path = self.dirModel.fileInfo(index).absoluteFilePath()
        self.listview.setRootIndex(self.fileModel.setRootPath(path))

    def select_file(self, index):
        path = self.dirModel.fileInfo(index).absoluteFilePath()
        if path.endswith(".mim"):
            img = skimage.io.imread(path)
            self.fig.clear()
            ax = self.fig.add_subplot()
            ax.imshow(img, cmap='gray')
            self.canvas.draw()    

    def pushButtonClicked(self):
        print(self.lineEdit.text())

    def plot_graph(self):
        self.fig.clear()
        ax = self.fig.add_subplot()
        t = np.linspace(0, 10, 101)
        ax.plot(t, np.sin(t))
        self.canvas.draw()

    def show_image(self):
        self.fig.clear()
        ax = self.fig.add_subplot()
        img = skimage.data.astronaut()
        ax.imshow(img)
        self.canvas.draw()

if __name__ == "__main__":
    app = QApplication.instance()
    if not app:
        app = QApplication(sys.argv)
        
    win = ApplicationWindow()
    win.show()
    win.activateWindow()
    win.raise_()
    app.exec()
```
