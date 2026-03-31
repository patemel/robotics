# robotics
έργο ότι νάνε 
from machine import Pin, PWM
import time

SENSORS
left = Pin(16, Pin.IN)
center = Pin(17, Pin.IN)
right = Pin(18, Pin.IN)

MOTORS
in1 = Pin(6, Pin.OUT)
in2 = Pin(7, Pin.OUT)
in3 = Pin(8, Pin.OUT)
in4 = Pin(9, Pin.OUT)
ena = PWM(Pin(10))
enb = PWM(Pin(11))
ena.freq(1000)
enb.freq(1000)

ΡΥΘΜΙΣΕΙΣ
FAST = 45000      # βασική ταχύτητα
TURN = 25000      # στροφή
STOP = 0          # σταμάτα

BLACK_LINE = 0    # μαύρη γραμμή = 0

def go_forward():
    in1.value(1); in2.value(0)
    in3.value(1); in4.value(0)

def motor_stop():
    in1.value(0); in2.value(0)
    in3.value(0); in4.value(0)
    ena.duty_u16(0)
    enb.duty_u16(0)

def set_speed(left_speed, right_speed):
    ena.duty_u16(left_speed)
    enb.duty_u16(right_speed)

print("🏁 LINE FOLLOWER vFINAL")
print("Μαύρη γραμμή = [010]")
print("Αποθήκευσε ως main.py για μπαταρία!")

while True:
    L = left.value()
    C = center.value()
    R = right.value()

DEBUG (αφαίρεσε # για να κρύψεις)
    print(f"[{L}{C}{R}]", end='\r')

    go_forward()

ΛΟΓΙΚΗ
    if C == BLACK_LINE:              # ΚΕΝΤΡΟ στη γραμμή
        set_speed(FAST, FAST)

    elif L == BLACK_LINE:            # Γραμμή ΑΡΙΣΤΕΡΑ → στρίψε ΔΕΞΙΑ
        set_speed(TURN, FAST)

    elif R == BLACK_LINE:            # Γραμμή ΔΕΞΙΑ → στρίψε ΑΡΙΣΤΕΡΑ
        set_speed(FAST, TURN)

    elif L == BLACK_LINE and R == BLACK_LINE:  # ΣΤΑΥΡΟΣ
        set_speed(FAST, FAST)

    else:                            # ΧΑΜΕΝΟ (άσπρο/αέρας)
        set_speed(STOP, STOP)

    time.sleep(0.005)
