import spidev
import RPi.GPIO as GPIO
import time
import requests
import smtplib
from email.mime.text import MIMEText

# ---------------- TELEGRAM SETUP ----------------
BOT_TOKEN = "8272130963:AAHfDXkGzB-1NzSnAD4-K7w4dzPhTAjrzYo"
CHAT_ID = "5912875888"

def send_telegram(msg):
    url = f"https://api.telegram.org/bot{BOT_TOKEN}/sendMessage"
    payload = {"chat_id": CHAT_ID, "text": msg}
    try:
        requests.post(url, data=payload, timeout=3)
        print("📩 Telegram alert sent")
    except:
        print("❌ Telegram send failed")

# ---------------- EMAIL SETTINGS ----------------
SENDER_EMAIL = "navyashree.cs.nc@cambridge.edu.in"
APP_PASSWORD = "aakniixsqzxrglzy"
RECEIVER_EMAIL = "shubda.cs.nc@cambridge.edu.in"

def send_email():
    try:
        subject = "⚠️ ALERT: Floor Pressure Detected"
        body = "Human foot pressure detected on the digital floor mat.\nPlease check immediately."

        msg = MIMEText(body)
        msg["Subject"] = subject
        msg["From"] = SENDER_EMAIL
        msg["To"] = RECEIVER_EMAIL

        server = smtplib.SMTP("smtp.gmail.com", 587)
        server.starttls()
        server.login(SENDER_EMAIL, APP_PASSWORD)
        server.sendmail(SENDER_EMAIL, RECEIVER_EMAIL, msg.as_string())
        server.quit()

        print("📧 Email alert sent")

    except Exception as e:
        print("❌ Email failed:", e)

# ---------------- GPIO SETUP ----------------
BUZZER_PIN = 18
GPIO.setmode(GPIO.BCM)
GPIO.setup(BUZZER_PIN, GPIO.OUT)
GPIO.output(BUZZER_PIN, GPIO.LOW)

# ---------------- SPI SETUP ----------------
spi = spidev.SpiDev()
spi.open(0, 0)
spi.max_speed_hz = 1350000

# ---------------- MCP3008 READ ----------------
def read_adc(channel):
    adc = spi.xfer2([1, (8 + channel) << 4, 0])
    return ((adc[1] & 3) << 8) | adc[2]

# ---------------- SETTINGS ----------------
LOCKED_THRESHOLD = 100
COOLDOWN = 10   # seconds (avoid spam alerts)

last_trigger = 0

print("🟢 Digital Floor Mat Security System Started")
send_telegram("🟢 Floor Security System Started")

try:
    while True:
        raw_adc = read_adc(0)

        if raw_adc >= LOCKED_THRESHOLD and (time.time() - last_trigger) > COOLDOWN:

            print(f"⚠️ Pressure detected! ADC: {raw_adc}")
            GPIO.output(BUZZER_PIN, GPIO.HIGH)

            alert = f"⚠️ ALERT!\nPressure detected!\nADC Value: {raw_adc}"

            send_telegram(alert)
            send_email()

            time.sleep(0.5)
            GPIO.output(BUZZER_PIN, GPIO.LOW)
            last_trigger = time.time()

        time.sleep(0.05)

except KeyboardInterrupt:
    print("\n🔴 System stopped")

finally:
    GPIO.cleanup()
    spi.close()
    send_telegram("🔴 Floor Security System Stopped")
