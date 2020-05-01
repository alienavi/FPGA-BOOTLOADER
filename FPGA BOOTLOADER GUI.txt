from PyQt5.QtWidgets import QWidget, QApplication, QPushButton, QLabel, QLineEdit, QComboBox, QProgressBar, QStatusBar, \
    QMainWindow, QFileDialog, QMessageBox
from PyQt5.QtGui import QIcon
from PyQt5.QtCore import pyqtSlot
import sys, serial, glob, os


class Gui(QWidget):
    def __init__(self):

        super().__init__()
        # self.ports=self.serial_ports()
        self.UI()

    def UI(self):

        comLabel=QLabel("COM PORT",self)
        comLabel.setGeometry(155,0, 70, 23)

        # Comport ComboBox object is created
        self.comport = QComboBox(self)
        # Below function gets called and COM Ports are shown
        self.on_click_refresh()

        # REFRESH PUSH BUTTON FOR REFRESHING COM PORT VALUES

        refreshPort = QPushButton("refresh", self)
        refreshPort.setGeometry(80, 0, 70, 23)
        refreshPort.clicked.connect(self.on_click_refresh)

        # SELECT FILE BUTTON

        selectFileButton = QPushButton("Select File", self)
        selectFileButton.setGeometry(0, 50, 100, 25)
        selectFileButton.clicked.connect(self.selectFile)

        # Upload File Directory Edit Box

        self.uploadfileEdit = QLineEdit(self)
        self.uploadfileEdit.setText("")
        self.uploadfileEdit.setGeometry(150, 50, 500, 25)

        # Program FPGA Button

        proFpgaButton = QPushButton("ProgramFPGA", self)
        proFpgaButton.setGeometry(0, 100, 100, 25)
        proFpgaButton.clicked.connect(self.upload)  # Upload Function gets called => Nothing in that function

        #Progress Bar Widget which shows progress of upload

        self.pbar = QProgressBar(self)
        self.pbar.setGeometry(150, 100, 540, 25)

        # Exit Button to Close Application
        exitBootloader = QPushButton("Exit Bootloader", self)
        exitBootloader.setGeometry(0, 150, 100, 25)
        exitBootloader.clicked.connect(QApplication.instance().quit)

        # Label which tells you upload time and Size
        self.textSizeTime = "UploadTime: None | FileSize: None"
        self.label = QLabel(self.textSizeTime, self)
        self.label.setGeometry(400, 160, 230, 20)

        # Window geometry and application name
        self.setGeometry(300, 300, 700, 200)
        self.setWindowTitle('FPGA BootLoader')
        self.setWindowIcon(QIcon('cpu_icon.png'))
        self.show()


    # Close event => once close button is pressesd
    def closeEvent(self, event): # Overided method

        reply = QMessageBox.question(self, 'Message',
                                     "Are you sure to quit?", QMessageBox.Yes |
                                     QMessageBox.No, QMessageBox.No)

        if reply == QMessageBox.Yes:
            event.accept()
        else:
            event.ignore()


    # Select File function to select file from directory
    @pyqtSlot()
    def selectFile(self):
        try:
            print('PyQt5 button click')
            fname = QFileDialog.getOpenFileName(self, 'Open File', 'c\\', 'JTAG Files (*.jpg .*png)')
            self.uploadfileEdit.setText(fname[0])

            ### FOR CHECKING FILE SIZE
            file_stat = os.stat(fname[0])
            sizeInMb = file_stat.st_size / (1024 * 1024)
            size = round(sizeInMb, 3)
            self.label.setText("UploadTime: None | FileSize: " + str(size) + "MB")  # Placing File size on Label Widget
        except OSError as e:
            print(e.errno)


    @pyqtSlot() # Refresh Button Method to refresh COM Port
    def on_click_refresh(self):
        self.p = self.serial_ports()
        self.comport.clear()
        for i in self.p:
            self.comport.addItem(i)

    def serial_ports(self): # Method which finds out the available ports and returns a list
        """ Lists serial port names

            :raises EnvironmentError:
                On unsupported or unknown platforms
            :returns:
                A list of the serial ports available on the system
        """
        if sys.platform.startswith('win'):
            ports = ['COM%s' % (i + 1) for i in range(256)]
        elif sys.platform.startswith('linux') or sys.platform.startswith('cygwin'):
            # this excludes your current terminal "/dev/tty"
            ports = glob.glob('/dev/tty[A-Za-z]*')
        elif sys.platform.startswith('darwin'):
            ports = glob.glob('/dev/tty.*')
        else:
            raise EnvironmentError('Unsupported platform')

        self.result = []
        for port in ports:
            try:
                s = serial.Serial(port)
                s.close()
                self.result.append(port)
            except (OSError, serial.SerialException):
                pass
        return self.result

    def upload(self):
        if self.uploadfileEdit.text()=="":
            QMessageBox.about(self, "Message", "Select File First !")
        else:
            for i in range(101):
                self.pbar.setValue(i)
            QMessageBox.about(self, "Message", "Upload Complete !")



if __name__ == '__main__':
    app = QApplication(sys.argv)
    ex = Gui()
    sys.exit(app.exec_())
