# data-base-serialport
import serial
from time import sleep
from tkinter import *
import mysql.connector

# Connect to serial port
ser = serial.Serial('/dev/ttyACM0', 9600)

config = {
    'user': 'root',
    'password': '78523',
    'host': '127.0.0.1',
    'database': 'Smart_Car_Parking',
    'raise_on_warnings': True
}
# Connect to database and change status
database = mysql.connector.connect(**config)
cursor = database.cursor()
cursor2 = database.cursor()

sp_query = "SELECT * FROM Parks where park_id=1"
entrance_query = "SELECT * FROM Users where user_tag_id = %s and is_inside = 0"
exit_query = "SELECT * FROM Users where user_tag_id = %s and is_inside = 1"
update_query = "Update Users set is_inside = 1 where user_tag_id = %s"
exitupdate_query = "Update Users set is_inside = 0 where user_tag_id = %s"
form_query = "SELECT * FROM Parks where park_id = 1"

while ser.is_open:
    incomingByte = ser.readline().decode()
    if incomingByte.startswith('From en'):
        print("---------------------------")
        print(incomingByte.strip())
    elif incomingByte.startswith('>'):
        id = incomingByte.strip().replace('>', '')
        print(id.strip())
        try:
            cursor.execute(entrance_query, (id.strip(),))
            if cursor.rowcount == 0:
                print("DO NOT HAVE ACCESS")
                ser.write("0".encode())
            else:
                result = cursor.fetchone()
                UserID = result[1]
                UserName = result[2]
                UserSurname = result[3]
                print(UserID + " "+UserName + " " + UserSurname + " Welcome.")
                ser.write("1".encode())
                cursor.execute(update_query, (UserID,))
                database.commit()
                #database.close()

        except:
            print("ERROR: DO NOT HAVE ACCESS")

    elif incomingByte.startswith('<'):
        id = incomingByte.strip().replace('<', '')
        print(id.strip())
        try:
            cursor.execute(exit_query, (id.strip(),))
            if cursor.rowcount == 0:
                print("DO NOT HAVE ACCESS")
                ser.write("0".encode())
            else:
                result = cursor.fetchone()
                UserID = result[1]
                UserName = result[2]
                UserSurname = result[3]
                print(UserID + " " + UserName + " " + UserSurname + " GoodBye.")
                ser.write("1".encode())
                cursor.execute(exitupdate_query, (UserID,))
                database.commit()
                # database.close()

        except:
            print("ERROR: DO NOT HAVE ACCESS")

    sleep(.1)
