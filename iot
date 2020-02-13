from PyQt5 import QtWidgets, QtCore
from PyQt5.QtWidgets import QApplication
from PyQt5.QtCore import pyqtSignal
from bluetooth import RFCOMM, BluetoothSocket
from bluetooth.btcommon import BluetoothError
from PyQt5.QtGui import QPen, QColor, QTextCursor
from datetime import datetime
from time import sleep
import time
import mainwindow
import pyqtgraph as pg
import sys
from numpy import array

colorsTheme = {
    "idle_color": "rgb(255,193,7)",
    "disabled_color": "rgb(229,57,53)",
    "enabled_color": "rgb(100,228,8)",
}

colorsDict = {
    "white": "rgb(255, 255, 255)",
    "lavender": "rgb(158, 109, 207)",
    "lightLavender": "rgb(179, 147, 210)",
    "bgGrey": "rgb(53, 53, 53)",
    "buttonGrey": "rgb(63, 63, 63)",
    "widgetGrey": "rgb(68, 68, 68)",
    "errorRed": "rgb(200, 16, 16)",
    "okLabelGreen": "rgb(0, 200, 83);",
    "warnLabelYellow": "rgb(255, 196, 0);",
    "denyLabelRed": "rgb(229, 57, 53);"
}

esp_dict = {
    "esp_32_0": "30:AE:A4:73:D6:BA",
    "esp_32_1":  "30:AE:A4:74:39:42",
    "esp_32_2": "30:AE:A4:74:A5:BE",
}

PORT = 1


class MainWindow(QtWidgets.QMainWindow, mainwindow.Ui_MainWindow):
    def __init__(self):
        pg.setConfigOption("foreground", "k")
        super().__init__()
        self.setupUi(self)
        self.init_gui()

        self.esp_32_0_receiver = DataReceiver("30:AE:A4:73:D6:BA")
        self.esp_32_1_receiver = DataReceiver("30:AE:A4:74:39:42")
        self.esp_32_2_receiver = DataReceiver("30:AE:A4:74:A5:BE")

        self.esp_32_0_receiver.consoleOutText.connect(self.update_console)
        self.esp_32_1_receiver.consoleOutText.connect(self.update_console)
        self.esp_32_2_receiver.consoleOutText.connect(self.update_console)

        self.esp_32_0_receiver.receivedData.connect(self.update_top_plot)
        self.esp_32_1_receiver.receivedData.connect(self.update_bottom_plot)
        self.esp_32_1_receiver.receivedData.connect(self.update_top_plot)
        self.esp_32_2_receiver.receivedData.connect(self.update_top_plot)

        self.dallas_temp_curve = self.top_plot.plot(
            name="dallas_temp",
            pen=self.set_pen(QColor(255,116,0))
            # pen=self.set_pen(QColor(255,0,0))
        )
        self.sound_curve = self.top_plot.plot(
            name="sound",
            pen=self.set_pen(QColor(245, 0, 29))
            # pen=self.set_pen(QColor(255, 0, 0))
        )
        self.photoresistor_curve = self.top_plot.plot(
            name='photoresistor',
            pen=self.set_pen(QColor(0, 255, 0))
            # pen=self.set_pen(QColor(255, 0, 0))
        )
        self.dht_temp_curve = self.bottom_plot.plot(
            name="dht_temp",
            pen=self.set_pen(QColor(173,0,159))
            # pen=self.set_pen(QColor(255, 0, 0))
        )
        self.humidity_curve = self.bottom_plot.plot(
            name="humidity",
            pen=self.set_pen(QColor(233,251,0))
            # pen=self.set_pen(QColor(255, 0, 0))
        )

        self.startButton.clicked.connect(self.start_receiving)
        self.stopButton.clicked.connect(self.stop_receiving)

        self.startButton.setStyleSheet(
            f"""
            #startButton {{
                background-color: rgb(51,67,107);
                border-radius: 5px;
                color: rgb(231,233,220);
             }}
             #startButton:pressed {{
                 background-color: {colorsDict["lavender"]};
             }}
             """)

        self.stopButton.setStyleSheet(
            f"""
            #stopButton {{
                background-color: grey;
                border-radius: 5px;
                color: rgb(231,233,220);
             }}
             #stopButton:pressed {{
                 background-color: {colorsDict["lavender"]};
             }}
             """)

    @staticmethod
    def set_pen(color, width=0.75):
        plot_pen = QPen(color)
        plot_pen.setWidth(width)
        return plot_pen

    def init_gui(self):
        self.dallas_temp_plot_color.setStyleSheet(
            "border: none;\n"
            "background-color: rgb(255,116,0);")
        self.dht_temp_plot_color.setStyleSheet(
            "border: none;\n"
            "background-color: rgb(173,0,159);"
        )
        self.humid_plot_color.setStyleSheet(
            "border: none;\n"
            "background-color: rgb(233,251,0);"
        )
        self.sound_plot_color.setStyleSheet(
            "border: none;\n"
            "background-color: rgb(245, 0, 29);"
        )
        self.light_plot_color.setStyleSheet(
            "border: none;\n"
            "background-color: rgb(0, 255, 0);"
        )
        pg.setConfigOption("foreground", QColor(231, 233, 220))
        self.top_plot.setBackground((63, 63, 63))
        self.top_plot.showGrid(x=True, y=True)
        self.top_plot.setTitle("Voltage")
        self.top_plot.getAxis("bottom").setPen(QPen(QColor(231,233,220)))
        self.top_plot.getAxis("left").setPen(QPen(QColor(231, 233, 220)))
        self.top_plot.setRange(rect=None,
                               xRange=[0, 10],
                               yRange=[0, 3.5],
                               padding=0.0,
                               update=True,
                               disableAutoRange=True
                               )

        self.bottom_plot.setBackground((63, 63, 63))
        self.bottom_plot.showGrid(x=True, y=True)
        self.bottom_plot.setTitle("Temperature and humidity")
        self.bottom_plot.getAxis("bottom").setPen(QPen(QColor(231,233,220)))
        self.bottom_plot.getAxis("left").setPen(QPen(QColor(231, 233, 220)))
        self.bottom_plot.setRange(rect=None,
                                  xRange=[0, 10],
                                  yRange=[0, 30],
                                  padding=0.0,
                                  update=True,
                                  disableAutoRange=True
                                  )

    def start_receiving(self):
        self.startButton.setDisabled(True)
        self.startButton.setStyleSheet(
            f"""
            #startButton {{
                background-color: grey;
                border-radius: 5px;
             }}
             #startButton:pressed {{
                 background-color: {colorsDict["lavender"]};
             }}
             """)
        self.stopButton.setStyleSheet(
            f"""
            #stopButton {{
                background-color: rgb(51,67,107);
                border-radius: 5px;
             }}
             #stopButton:pressed {{
                 background-color: {colorsDict["lavender"]};
             }}
             """)
        self.esp_32_0_receiver.start()
        self.esp_32_1_receiver.start()
        self.esp_32_2_receiver.start()

    def stop_receiving(self):
        self.update_console("[ {} ]: Closing connection to {}\n".format(
            datetime.now().strftime("%Y-%m-%d %H:%M:%S"),
            self.esp_32_0_receiver.MAC))
        self.esp_32_0_receiver.socket.close()
        self.update_console("[ {} ]: Connection closed!\n".format(datetime.now().strftime("%Y-%m-%d %H:%M:%S")))
        self.update_console("[ {} ]: Closing connection to {}\n".format(
            datetime.now().strftime("%Y-%m-%d %H:%M:%S"),
            self.esp_32_1_receiver.MAC))
        self.esp_32_1_receiver.socket.close()
        self.update_console("[ {} ]: Connection closed!\n".format(datetime.now().strftime("%Y-%m-%d %H:%M:%S")))
        self.update_console("[ {} ]: Closing connection to {}\n".format(
            datetime.now().strftime("%Y-%m-%d %H:%M:%S"),
            self.esp_32_2_receiver.MAC))
        self.esp_32_2_receiver.socket.close()
        self.update_console("[ {} ]: Connection closed!\n".format(datetime.now().strftime("%Y-%m-%d %H:%M:%S")))
        self.startButton.setStyleSheet(
            f"""
            #startButton {{
                background-color: rgb(51,67,107);
                border-radius: 5px;
             }}
             #startButton:pressed {{
                 background-color: {colorsDict["lavender"]};
             }}
             """)
        self.stopButton.setStyleSheet(
            f"""
            #stopButton {{
                background-color: grey;
                border-radius: 5px;
             }}
             #stopButton:pressed {{
                 background-color: {colorsDict["lavender"]};
             }}
             """)
        self.startButton.setDisabled(False)

    def update_top_plot(self, data):
        y_values = data.get(1)
        x_values = data.get(2)
        sender_name = data.get(0)
        values_to_display = array(list(zip(x_values, y_values)))
        self.top_plot.getAxis("bottom").setPen(QPen(QColor(231, 233, 220)))
        self.top_plot.getAxis("left").setPen(QPen(QColor(231, 233, 220)))
        if len(x_values) < 10:
            self.top_plot.setRange(
                rect=None,
                xRange=[0, 10],
                yRange=[0, 3.5],
                padding=0.0,
                update=True,
                disableAutoRange=True
            )
        else:

            self.top_plot.setRange(
                rect=None,
                xRange=[x_values[-10] - 1, x_values[-1]],
                yRange=[0, 3.5],
                padding=0.0,
                update=True,
                disableAutoRange=True
            )
        if sender_name == "30:AE:A4:73:D6:BA":
            self.sound_curve.setData(values_to_display)
            self.sound_sensor_value.setText("{} V".format(str(y_values[-1])))
        elif sender_name =="30:AE:A4:74:39:42":
            self.dallas_temp_curve.setData(values_to_display)
            self.dallas_temp_sensor_value.setText("{} V".format(str(y_values[-1])))
        elif sender_name == "30:AE:A4:74:A5:BE":
            self.photoresistor_curve.setData(values_to_display)
            self.light_sensor_value.setText("{} V".format(str(y_values[-1])))

    def update_bottom_plot(self, data):
        sender_name = data.get(0)
        if sender_name == "30:AE:A4:74:39:42":
            t_y_values = data.get(3)[0]
            h_y_values = data.get(3)[1]
            x_values = data.get(2)
            t_values_to_display = array(list(zip(x_values, t_y_values)))
            h_values_to_display = array(list(zip(x_values, h_y_values)))
            self.bottom_plot.getAxis("bottom").setPen(QPen(QColor(231, 233, 220)))
            self.bottom_plot.getAxis("left").setPen(QPen(QColor(231, 233, 220)))
            if len(x_values) < 10:
                self.bottom_plot.setRange(
                    rect=None,
                    xRange=[0, 10],
                    yRange=[0, 30],
                    padding=0.0,
                    update=True,
                    disableAutoRange=False
                )
            else:
                self.bottom_plot.setRange(
                    rect=None,
                    xRange=[x_values[-10] - 1, x_values[-1]],
                    yRange=[0, 30],
                    padding=0.0,
                    update=True,
                    disableAutoRange=False
                )
            self.dht_temp_curve.setData(t_values_to_display)
            try:
                self.dht_temp_sensor_value.setText("{} C".format(str(t_y_values[-1])))
            except IndexError:
                pass
            self.humidity_curve.setData(h_values_to_display)
            try:
                self.dht_humid_sensor_value.setText("{}%".format(str(h_y_values[-1])))
            except IndexError:
                pass

    def update_console(self, msg):
        curr_text = self.textEdit.toPlainText()
        self.textEdit.setText(curr_text + msg)
        self.textEdit.moveCursor(QTextCursor.End)


class SendingData:
    """
    This class represents a data structure to send in to plot.
    """
    def __init__(self, MAC):
        self.MAC = MAC
        self.timeList = []
        self.data_to_send = (
            self.MAC,          # ESP MAC address; [0]
            [],                        # voltage measure ESP data series; [1]
            self.timeList,   # time since measurement started data; [2]
            [
                [],                    # DHT11 temperature data series; [3][0]
                []                     # DHT11 humidity data series; [3][1]
            ]
        )

    def get(self, i):
        return self.data_to_send[i]

    def updateSendingData(self, voltageData, measureOneData, measureTwoData, timeData):
        try:
            self.data_to_send[1].append(float(voltageData))
            self.data_to_send[2].append(int(timeData))
            self.data_to_send[3][0].append(float(measureOneData))
            self.data_to_send[3][1].append(float(measureTwoData))
        except Exception:
            pass


class DataReceiver(QtCore.QThread):
    receivedData = pyqtSignal(SendingData)
    consoleOutText = pyqtSignal(str)

    def __init__(self, mac):
        super(DataReceiver, self).__init__()
        self.MAC = mac
        self.message = ""
        self.data = ""
        self.socket = BluetoothSocket(RFCOMM)
        self.dallas_temp_data = []
        self.dht_temp_data = []
        self.dht_humid_data = []
        self.sound_data = []
        self.time_list = []
        self.startTime = 0
        self.data_to_send = SendingData(self.MAC)

    def run(self):
        self.startTime = int(time.time())
        try:
            print("Connecting to {}".format(self.MAC))
            self.consoleOutText.emit("[ {} ]: Connecting to {}\n".format(
                datetime.now().strftime("%Y-%m-%d %H:%M:%S"),
                self.MAC))
            self.socket.connect((self.MAC, PORT))
            print("Connected!")
            self.consoleOutText.emit("[ {} ]: Connected!\n".format(datetime.now().strftime("%Y-%m-%d %H:%M:%S")))
        except BluetoothError:
            self.consoleOutText.emit(
                "[ {} ]: An error occured while connecting to {} because of bt.btcommon.BluetoothError\n".format(
                    datetime.now().strftime("%Y-%m-%d %H:%M:%S"),
                    self.MAC
                )
            )
        except OSError:
            self.consoleOutText.emit(
                "[ {} ]: An error occured while connecting to {} because of OSError. Check Bluetooth setting.\n".format(
                    datetime.now().strftime("%Y-%m-%d %H:%M:%S"),
                    self.MAC
                )
            )

        try:
            while True:
                self.data = self.socket.recv(1024).decode().split("\n")[0]
                if "7P5uk6fbZWTkXxr67P5uk6fbZWTkXxr6;" in self.data:
                    self.data = self.data.split(";")[1:]
                    self.data_to_send.updateSendingData(
                        self.data[0],
                        self.data[2],
                        self.data[4],
                        int(time.time()) - self.startTime
                    )
                    print(
                        "[{}]: {} : {}".format(
                                datetime.now().strftime("%Y-%m-%d %H:%M:%S"),
                                self.MAC,
                                self.data
                                ))
                    self.consoleOutText.emit(
                        "[{}]: {} : {}\n".format(
                                datetime.now().strftime("%Y-%m-%d %H:%M:%S"),
                                self.MAC,
                                self.data
                                )
                        )
                    self.receivedData.emit(self.data_to_send)
                    sleep(0.85)
        except KeyboardInterrupt:
                self.consoleOutText.emit("[ {} ]: Closing connection to {}\n".format(
                    datetime.now().strftime("%Y-%m-%d %H:%M:%S"),
                    self.MAC))
                self.socket.close()
                self.consoleOutText.emit("[ {} ]: Connection closed!\n".format(
                    datetime.now().strftime("%Y-%m-%d %H:%M:%S"))
                )
        except SystemExit:
            self.consoleOutText.emit("[ {} ]: Closing connection to {}\n".format(
                datetime.now().strftime("%Y-%m-%d %H:%M:%S"),
                self.MAC)
            )
            self. socket.close()
            self.consoleOutText.emit("[ {} ]: Connection closed!\n".format(
                datetime.now().strftime("%Y-%m-%d %H:%M:%S"))
            )


def __run__(window_object):
    app = QApplication([])
    window = window_object()
    center_point = QtWidgets.QDesktopWidget().availableGeometry().center()
    half_window_size = QtCore.QPoint(750 / 2, (250 + 38) / 2)
    window.resize(750, 400)
    window.move(center_point - half_window_size)
    window.show()
    sys.exit(app.exec_())

__run__(MainWindow)
