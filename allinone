import datetime
import time
import signal
import RPi.GPIO as GPIO
from RPLCD.i2c import CharLCD
import Adafruit_DHT
import speedtest
import requests

# Replace 0x27 with the I2C address of your LCD module
lcd = CharLCD('PCF8574', 0x27)

# Replace 'YOUR_API_KEY' with your OpenWeatherMap API key
API_KEY = 'e2692449d4657dd06c99cd150c33c675'
CITY = 'Beirut, Lebanon'  # Replace with your city and country

# Set up GPIO for LED backlight control
LED_PIN = 18
GPIO.setmode(GPIO.BCM)
GPIO.setup(LED_PIN, GPIO.OUT)

def display_time_date():
    # Get the current time and date
    current_time = datetime.datetime.now().strftime("%I:%M:%S %p")
    current_date = datetime.datetime.now().strftime("%d-%m-%Y")

    # Display the time and date on the LCD
    lcd.clear()
    lcd.write_string(f"Time: {current_time}")
    lcd.crlf()  # Move to the second line
    lcd.write_string(f"Date: {current_date}")

def read_temp_humidity():
    # Replace DHT22_PIN with the GPIO pin connected to the DHT22 sensor
    DHT22_PIN = 4
    # Read temperature and humidity from DHT22 sensor
    humidity, temperature = Adafruit_DHT.read_retry(Adafruit_DHT.DHT22, DHT22_PIN)
    return humidity, temperature

def display_temp_humidity(temp, humidity):
    # Display temperature and humidity on the LCD
    lcd.clear()
    lcd.write_string(f"Temp: {temp:.2f} C")
    lcd.crlf()  # Move to the second line
    lcd.write_string(f"Humi: {humidity:.2f} %")

def get_weather_data():
    try:
        url = f'https://api.openweathermap.org/data/2.5/weather?q={CITY}&appid={API_KEY}&units=metric'
        response = requests.get(url)
        data = response.json()

        # Extract weather information from the API response
        temperature = data['main']['temp']
        humidity = data['main']['humidity']
        weather_description = data['weather'][0]['description']

        return temperature, humidity, weather_description
    except requests.RequestException as e:
        print(f"Error fetching weather data: {e}")
        return None

def display_weather(temperature, humidity, weather_description):
    # Display weather information on the LCD
    lcd.clear()
    lcd.write_string(f"Rite Lebanon Beirut")
    lcd.crlf()  # Move to the second line
    lcd.write_string(f"T: {temperature:.1f} C, H: {humidity}%")
    lcd.crlf()  # Move to the third line
    lcd.write_string(f"Description: {weather_description}")

def measure_internet_speed():
    # Create a Speedtest object
    st = speedtest.Speedtest()

    # Find the best server automatically
    st.get_best_server()

    # Get download and upload speeds in bits per second (bps)
    download_speed = st.download()
    upload_speed = st.upload()

    # Convert speeds to Megabits per second (Mbps)
    download_speed_mbps = download_speed / 1_000_000
    upload_speed_mbps = upload_speed / 1_000_000

    return download_speed_mbps, upload_speed_mbps

def display_internet_speed(download_speed, upload_speed):
    # Display download and upload speeds on the LCD
    lcd.clear()
    lcd.write_string(f"Download: {download_speed:.2f} Mbps")
    lcd.crlf()  # Move to the second line
    lcd.write_string(f"Upload: {upload_speed:.2f} Mbps")

def cleanup_handler(signum, frame):
    lcd.clear()  # Clear the LCD before exiting
    lcd.write_string("Exiting...")
    time.sleep(2)  # Wait for 2 seconds before clearing the screen
    lcd.clear()  # Clear the screen
    GPIO.output(LED_PIN, GPIO.LOW)  # Turn off the backlight before exiting
    exit(0)

# Register the cleanup handler to handle Ctrl+C
signal.signal(signal.SIGINT, cleanup_handler)

try:
    while True:
        # Display time and date for 20 seconds
        display_time_date()
        GPIO.output(LED_PIN, GPIO.HIGH)  # Turn on the backlight
        time.sleep(20)

        # Get the current temperature and humidity
        humidity, temperature = read_temp_humidity()

        # Display temperature and humidity for 20 seconds
        display_temp_humidity(temperature, humidity)
        GPIO.output(LED_PIN, GPIO.HIGH)  # Turn on the backlight
        time.sleep(20)

        # Measure internet speed
        download_speed, upload_speed = measure_internet_speed()

        # Display internet speed for 20 seconds
        display_internet_speed(download_speed, upload_speed)
        GPIO.output(LED_PIN, GPIO.HIGH)  # Turn on the backlight
        time.sleep(20)

        # Get weather data and display it
        weather_data = get_weather_data()
        if weather_data:
            temperature, humidity, weather_description = weather_data
            display_weather(temperature, humidity, weather_description)
            GPIO.output(LED_PIN, GPIO.HIGH)  # Turn on the backlight
            time.sleep(20)

except KeyboardInterrupt:
    # When the program is interrupted with Ctrl+C, the cleanup_handler will be called to clear the LCD and turn off the backlight.
    pass
