from machine import Pin, PWM
from utime import sleep_ms

# --- ΠΙΝΣ ΜΟΤΕΡ ---
ENA = PWM(Pin(19))
IN1 = Pin(6, Pin.OUT)
IN2 = Pin(7, Pin.OUT)

IN3 = Pin(8, Pin.OUT)
IN4 = Pin(9, Pin.OUT)
ENB = PWM(Pin(20))

ENA.freq(20000)
ENB.freq(20000)

# ΣΚΛΗΡΟ RESET ΣΤΑ ΜΟΤΕΡ ΓΙΑ ΝΑ ΜΗΝ ΤΡΕΧΟΥΝ ΜΟΝΑ ΤΟΥΣ
IN1.low()
IN2.low()
IN3.low()
IN4.low()
ENA.duty_u16(0)
ENB.duty_u16(0)

base_speed = 38000
turn_speed = 26000

# --- ΠΙΝΣ ΑΙΣΘΗΤΗΡΩΝ ---
LEFT = Pin(16, Pin.IN)
MID = Pin(14, Pin.IN)
RIGHT = Pin(13, Pin.IN)

def motor_speed(left, right):
    ENA.duty_u16(left)
    ENB.duty_u16(right)

# 🚀 ΚΙΝΗΣΗ ΜΠΡΟΣΤΑ (Κανονική κατεύθυνση)
def forward():
    motor_speed(base_speed, base_speed)
    IN1.low()    # Αριστερή ρόδα -> Μπροστά
    IN2.high()   
    IN3.low()    # Δεξιά ρόδα -> Μπροστά
    IN4.high()   

# ↪️ Ο ΔΕΞΙΟΣ βρήκε γραμμή -> Δουλεύει ΜΟΝΟ η ΑΡΙΣΤΕΡΗ ρόδα (Στροφή δεξιά)
def slight_left():
    motor_speed(base_speed, 0) # Μηδέν γκάζι στη δεξιά
    IN1.low()    # Αριστερή ρόδα -> Μπροστά
    IN2.low()   
    IN3.low()    # Δεξιά ρόδα -> Σβηστή
    IN4.high()    

# ↩️ Ο ΑΡΙΣΤΕΡΟΣ βρήκε γραμμή -> Δουλεύει ΜΟΝΟ η ΔΕΞΙΑ ρόδα (Στροφή αριστερά)
def slight_right():
    motor_speed(0, base_speed) # Μηδέν γκάζι στην αριστερή
    IN1.low()    # Αριστερή ρόδα -> Σβηστή
    IN2.high()    
    IN3.low()    # Δεξιά ρόδα -> Μπροστά
    IN4.low()
# 🛑 ΣΤΑΜΑΤΗΜΑ (Κλείνουν όλα)
def stop():
    motor_speed(0, 0)
    IN1.low()
    IN2.low()
    IN3.low()
    IN4.low()

# === ΕΓΚΕΦΑΛΟΣ (ΛΟΓΙΚΗ: 1 = ΜΑΥΡΟ, 0 = ΑΣΠΡΟ) ===
while True:
    L = LEFT.value()
    M = MID.value()
    R = RIGHT.value()

    # 1. Στο λευκό πλακάκι (Όλα μηδέν -> 0, 0, 0) -> ΤΩΡΑ ΣΤΑΜΑΤΑΕΙ ΚΑΝΟΝΙΚΑ
    if [L, M, R] == [0, 0, 0]:
        stop()

    # 2. Κέντρο (Μόνο ο μεσαίος βλέπει μαύρο -> 1) -> Πάει μπροστά
    elif [L, M, R] == [0, 1, 0]:
        forward()

    # 3. Η γραμμή έφυγε αριστερά (Αριστερός βρήκε μαύρο -> 1) -> Στρίψε δεξιά
    elif [L, M, R] == [1, 0, 0] or [L, M, R] == [1, 1, 0]:
        slight_right()   

    # 4. Η γραμμή έφυγε δεξιά (Δεξιός βρήκε μαύρο -> 1) -> Στρίψε αριστερά
    elif [L, M, R] == [0, 0, 1] or [L, M, R] == [0, 1, 1]:
        slight_left()    

    # 5. Αν βρεθεί ολόκληρο πάνω σε μαύρο (1, 1, 1) -> Προχώρα μπροστά
    elif [L, M, R] == [1, 1, 1]:
        forward()

    sleep_ms(10)
