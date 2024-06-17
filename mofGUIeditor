import sys
import re
import os
from PyQt5.QtWidgets import QApplication, QMainWindow, QTextEdit, QFileDialog, QAction, QVBoxLayout, QWidget, QCompleter, QTextBrowser, QLineEdit, QMessageBox
from PyQt5.QtGui import QIcon, QColor, QTextCharFormat, QFont, QTextCursor, QSyntaxHighlighter
from PyQt5.QtCore import Qt, QStringListModel

class PythonHighlighter(QSyntaxHighlighter):
    def __init__(self, document):
        super().__init__(document)
        
        self.highlightingRules = []
        
        # Keywords
        keywordFormat = QTextCharFormat()
        keywordFormat.setForeground(QColor("blue"))
        keywordFormat.setFontWeight(QFont.Bold)
        keywords = [
            "and", "as", "assert", "break", "class", "continue", "def", "del",
            "elif", "else", "except", "False", "finally", "for", "from", "global",
            "if", "import", "in", "is", "lambda", "None", "nonlocal", "not", "or",
            "pass", "raise", "return", "True", "try", "while", "with", "yield"
        ]
        for word in keywords:
            pattern = re.compile(r'\b' + word + r'\b')
            self.highlightingRules.append((pattern, keywordFormat))
        
        # Single-line comments
        singleLineCommentFormat = QTextCharFormat()
        singleLineCommentFormat.setForeground(QColor("darkGreen"))
        pattern = re.compile(r'#.*')
        self.highlightingRules.append((pattern, singleLineCommentFormat))
        
        # Strings
        stringFormat = QTextCharFormat()
        stringFormat.setForeground(QColor("magenta"))
        pattern = re.compile(r'"[^"\\]*(\\.[^"\\]*)*"')
        self.highlightingRules.append((pattern, stringFormat))
        pattern = re.compile(r"'[^'\\]*(\\.[^'\\]*)*'")
        self.highlightingRules.append((pattern, stringFormat))
        
        # Functions
        functionFormat = QTextCharFormat()
        functionFormat.setFontItalic(True)
        functionFormat.setForeground(QColor("darkRed"))
        pattern = re.compile(r'\b[A-Za-z0-9_]+(?=\()')
        self.highlightingRules.append((pattern, functionFormat))
    
    def highlightBlock(self, text):
        for pattern, format in self.highlightingRules:
            for match in pattern.finditer(text):
                self.setFormat(match.start(), match.end() - match.start(), format)
        
        self.setCurrentBlockState(0)

class CodeEditor(QMainWindow):
    def __init__(self):
        super().__init__()

        self.initUI()
    
    def initUI(self):
        # Set up the main window
        self.setWindowTitle("Mof Code Editor")
        self.setGeometry(100, 100, 800, 600)
        
        # Create the text editor
        self.textEdit = CodeEditorWidget(self)
        self.highlighter = PythonHighlighter(self.textEdit.document())
        
        # Create console area
        self.consoleWidget = ConsoleWidget(self)
        
        # Arrange central widget layout
        layout = QVBoxLayout()
        layout.addWidget(self.textEdit)
        layout.addWidget(self.consoleWidget)
        
        central_widget = QWidget()
        central_widget.setLayout(layout)
        self.setCentralWidget(central_widget)
        
        # Set up the menu
        self.createMenu()
        
        # Show the main window
        self.show()
    
    def createMenu(self):
        # Create the menu bar
        menuBar = self.menuBar()
        
        # Create file menu
        fileMenu = menuBar.addMenu("File")
        
        # Create actions
        openFile = QAction(QIcon(), "Open", self)
        openFile.setShortcut("Ctrl+O")
        openFile.setStatusTip("Open a file")
        openFile.triggered.connect(self.openFile)
        
        saveFile = QAction(QIcon(), "Save", self)
        saveFile.setShortcut("Ctrl+S")
        saveFile.setStatusTip("Save the file")
        saveFile.triggered.connect(self.saveFile)
        
        # Add actions to file menu
        fileMenu.addAction(openFile)
        fileMenu.addAction(saveFile)
    
    def openFile(self):
        # Open file dialog
        options = QFileDialog.Option()
        fileName, _ = QFileDialog.getOpenFileName(self, "Open File", "", "All Files (*);;Text Files (*.txt);;Python Files (*.py)", options=options)
        if fileName:
            try:
                with open(fileName, 'r') as file:
                    self.textEdit.setText(file.read())
            except Exception as e:
                QMessageBox.critical(self, "Error", f"Failed to open file:\n{str(e)}")
    
    def saveFile(self):
        # Save file dialog
        options = QFileDialog.Option()
        fileName, _ = QFileDialog.getSaveFileName(self, "Save File", "", "All Files (*);;Text Files (*.txt);;Python Files (*.py)", options=options)
        if fileName:
            try:
                with open(fileName, 'w') as file:
                    file.write(self.textEdit.toPlainText())
            except Exception as e:
                QMessageBox.critical(self, "Error", f"Failed to save file:\n{str(e)}")

class CodeEditorWidget(QTextEdit):
    def __init__(self, parent=None):
        super().__init__(parent)
        
        # Setup the completer
        self.completer = QCompleter(self)
        self.completer.setWidget(self)
        self.completer.setCompletionMode(QCompleter.PopupCompletion)
        self.completer.setCaseSensitivity(Qt.CaseInsensitive)
        
        self.model = QStringListModel()
        self.completer.setModel(self.model)
        
        self.keywords = [
            "and", "as", "assert", "break", "class", "continue", "def", "del",
            "elif", "else", "except", "False", "finally", "for", "from", "global",
            "if", "import", "in", "is", "lambda", "None", "nonlocal", "not", "or",
            "pass", "raise", "return", "True", "try", "while", "with", "yield"
        ]
        
        self.model.setStringList(self.keywords)
    
    def keyPressEvent(self, event):
        if event.key() == Qt.Key_Return or event.key() == Qt.Key_Enter:
            self.insertPlainText("\n" + self.autoIndentation())
        elif event.key() == Qt.Key_V and event.modifiers() & Qt.ControlModifier:  # Handle Ctrl+V paste
            mimeData = QApplication.clipboard().mimeData()
            if mimeData.hasText():
                self.insertPlainText(mimeData.text())
        else:
            super().keyPressEvent(event)
    
    def autoIndentation(self):
        cursor = self.textCursor()
        cursor.movePosition(QTextCursor.StartOfLine, QTextCursor.KeepAnchor)
        text = cursor.selectedText()
        indentation = ""
        for char in text:
            if char == ' ' or char == '\t':
                indentation += char
            else:
                break
        
        if re.match(r'.*:\s*$', text):
            indentation += "    "
        
        return indentation

class ConsoleWidget(QWidget):
    def __init__(self, parent=None):
        super().__init__(parent)
        
        layout = QVBoxLayout()
        
        self.consoleOutput = QTextBrowser()
        layout.addWidget(self.consoleOutput)
        
        self.inputField = QLineEdit()
        self.inputField.returnPressed.connect(self.executeCommand)
        layout.addWidget(self.inputField)
        
        self.setLayout(layout)
    
    def executeCommand(self):
        command = self.inputField.text()
        self.consoleOutput.append(f"> {command}")
        self.inputField.clear()
        
        # Execute command using os.system
        output = os.popen(command).read()
        self.consoleOutput.append(output)

if __name__ == "__main__":
    app = QApplication(sys.argv)
    editor = CodeEditor()
    sys.exit(app.exec_())
