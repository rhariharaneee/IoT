from base64 import b64encode, b64decode
from hashlib import sha256
from urllib import quote_plus, urlencode
from hmac import HMAC
import requests
import json
import os
import time
 
# Temperature Sensor
BASE_DIR = '/sys/bus/w1/devices/'
SENSOR_DEVICE_ID = 'YOUR_DEVICE_ID'
DEVICE_FILE = BASE_DIR + SENSOR_DEVICE_ID + '/w1_slave'

# Azure IoT Hub
URI = 'YOUR_IOT_HUB_NAME.azure-devices.net'
KEY = 'YOUR_IOT_HUB_PRIMARY_KEY'
IOT_DEVICE_ID = 'YOUR_REGISTED_IOT_DEVICE_ID'
POLICY = 'iothubowner'

def generate_sas_token():
    expiry=3600
    ttl = time.time() + expiry
    sign_key = "%s\n%d" % ((quote_plus(URI)), int(ttl))
    signature = b64encode(HMAC(b64decode(KEY), sign_key, sha256).digest())

    rawtoken = {
        'sr' :  URI,
        'sig': signature,
        'se' : str(int(ttl))
    }

    rawtoken['skn'] = POLICY

    return 'SharedAccessSignature ' + urlencode(rawtoken)

def read_temp_raw():
    f = open(DEVICE_FILE, 'r')
    lines = f.readlines()
    f.close()
    return lines

def read_temp():
    lines = read_temp_raw()
    while lines[0].strip()[-3:] != 'YES':
        time.sleep(0.2)
        lines = read_temp_raw()
    equals_pos = lines[1].find('t=')
    if equals_pos != -1:
        temp_string = lines[1][equals_pos+2:]
        temp_c = float(temp_string) / 1000.0
        return temp_c

def send_message(token, message):
	url = 'https://{0}/devices/{1}/messages/events?api-version=2016-11-14'.format(URI, IOT_DEVICE_ID)
    headers = {
        "Content-Type": "application/json",
        "Authorization": token
    }
    data = json.dumps(message)
    print data
    response = requests.post(url, data=data, headers=headers)

if __name__ == '__main__':
    # 1. Enable Temperature Sensor
    os.system('modprobe w1-gpio')
    os.system('modprobe w1-therm')

    # 2. Generate SAS Token
    token = generate_sas_token()

    # 3. Send Temperature to IoT Hub
    while True:
        temp = read_temp() 
        message = { "temp": str(temp) }
        send_message(token, message)
        time.sleep(1)
