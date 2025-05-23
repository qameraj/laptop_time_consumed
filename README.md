# laptop_time_consumed
This code we build to understand how much it is taking for user to be on laptop each day .
import time
import pandas as pd
import ctypes
import datetime

# Idle time threshold (in seconds)
IDLE_THRESHOLD = 60  # 1 min

# CSV log file
LOG_FILE = "usage_log.csv"

# Get idle time in seconds
def get_idle_duration():
    class LASTINPUTINFO(ctypes.Structure):
        _fields_ = [("cbSize", ctypes.c_uint), ("dwTime", ctypes.c_uint)]
    lii = LASTINPUTINFO()
    lii.cbSize = ctypes.sizeof(lii)
    ctypes.windll.user32.GetLastInputInfo(ctypes.byref(lii))
    millis = ctypes.windll.kernel32.GetTickCount() - lii.dwTime
    return millis / 1000.0

# Log start time
print("🟢 Tracking started...")
start_time = time.time()
active_minutes = 0

try:
    while True:
        idle = get_idle_duration()
        if idle < IDLE_THRESHOLD:
            active_minutes += 1
        time.sleep(60)  # Check every minute

        # Save every hour or stop with CTRL+C
        if active_minutes % 60 == 0:
            now = datetime.datetime.now()
            df = pd.DataFrame([[now.date(), active_minutes / 60]],
                              columns=["date", "hours"])
            df.to_csv(LOG_FILE, mode='a', header=not pd.read_csv(LOG_FILE).shape[0] if os.path.exists(LOG_FILE) else True, index=False)
except KeyboardInterrupt:
    print("\n⏹️ Tracking stopped.")
